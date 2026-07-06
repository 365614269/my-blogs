---
title: "Debugging a tool_choice = required EngineCore Crash on Intel XPU"

date: 2026-07-06
---

*Issue investigation on [`intel/llm-scaler/issues/493`](https://github.com/intel/llm-scaler/issues/493)*

---

## The Report

A user (`@diablo581`) filed a bug: any chat completion request with `tool_choice: "required"` reliably crashed the vLLM EngineCore process with:

```
terminate called after throwing an instance of 'sycl::_V1::exception'
  what():  No device of requested type available.
ERROR [core_client.py:610] Engine core proc EngineCore_DP0 died unexpectedly
```

The identical request with `tool_choice: "auto"` succeeded every time. The minimal reproducer was a single zero-parameter tool and a one-word prompt—ruling out memory pressure, context length, or payload complexity as causes.

**Environment:** Intel Arc Pro B70 (Battlemage), Bazzite (Fedora Atomic), `intel/llm-scaler-vllm:0.14.0-b8.3.1`.

---

## Initial Hypothesis: Host Driver Mismatch

The first and most natural hypothesis was a host-side GPU driver/runtime issue. The container accesses the GPU via `--device=/dev/dri`, so the host kernel driver, firmware, and Level-Zero runtime are all in the path. A `sycl::exception: No device of requested type available` is a device-enumeration failure—exactly what a driver incompatibility would produce.

To test this, I reproduced the setup on two B-series cards under a known-good environment:

| Card | Device ID | OS | NEO compute-runtime | Level-Zero | Result |
|---|---|---|---|---|---|
| Arc Pro B60 | 0xe211 | Ubuntu 24.04 (PPA) | 26.09.37435.12 | 1.14.37435+12 | ✅ No crash |
| Arc Pro B70 | 0xe223 | Ubuntu 24.04 (PPA) | 26.09.37435.12 | 1.14.37435+12 | ✅ No crash |

`tool_choice: "required"` returned a valid tool call on both cards. The image digest matched the reporter's exactly (`sha256:4304f64f2a9a38994cf1e824d8fa0c61769c87c8b7ecfd2965d9a01b46c8bc8a`).

This ruled out:
- The image, vLLM code, or grammar/structured-output path
- The GPU model (B60 vs. B70)
- Single-card isolation (tested with `ZE_AFFINITY_MASK`)
- OOM or resource exhaustion (the error is a device query failure, not an allocation failure)

The reporter's host showed `intel-opencl-26.18.38308.1` (Fedora/fc44)—a different runtime version from the confirmed-working `26.09.37435.12`. Driver mismatch remained the leading hypothesis, and the suggested fix was to upgrade the host compute-runtime.

---

## The Picture Changes

After requesting container-side diagnostics from both the crashing and non-crashing containers, the reporter shared a critical comparison:

| Container | Level-Zero runtime | torch | Crashes? |
|---|---|---|---|
| `intel/llm-scaler-vllm:0.14.0-b8.3.1` | `1.14.37435+12` | `2.10.0+xpu` | ✅ Yes |
| `intel/vllm:0.10.2-xpu` | `1.6.34666+3` | `2.8.0+xpu` | ❌ No |

Both containers saw the GPU correctly. Device count was 1, `xpu.is_available()` was `True`, and the GPU PCI ID `[0xe223]` was visible in both. This eliminated generic device-invisibility as the root cause.

The crash was now clearly **specific to the `llm-scaler` image's runtime stack**—a different Level-Zero version and a newer torch XPU build—rather than a host-only driver issue. The most likely mechanism: `tool_choice: "required"` forces the structured-decoding/grammar path, which may invoke a SYCL device capability query that the newer runtime combination handles differently. `tool_choice: "auto"` keeps a free-text branch and skips that path entirely.

---

## Outcome

The crash was not reproduced on available hardware. A colleague (`@Wesley-Du`) confirmed the same on a separate setup, with the note that response behavior differed between `"required"` and `"auto"` as expected by design—but no EngineCore crash occurred.

The issue remains **open for reproduction in an environment closer to the reporter's** (Bazzite/Fedora Atomic, fc44, `intel-opencl-26.18.38308.x`). The investigation narrowed the root cause to a runtime-stack compatibility issue between `intel/llm-scaler-vllm:0.14.0-b8.3.1` and that specific host OS/driver combination, rather than anything in the image's application logic.

---

## What This Investigation Established

- `tool_choice: "required"` works correctly on B60 and B70 under Ubuntu 24.04 with NEO `26.09.37435.12`
- The crash is not reproducible from the image side alone—it requires the reporter's specific host runtime
- The `sycl: No device of requested type available` error on the structured-output path is the most specific lead for anyone attempting to reproduce in a Fedora/Bazzite environment
- A runtime-version bisect between Level-Zero `1.6.34666+3` (no crash) and `1.14.37435+12` (crash, reporter's stack) is the most direct path to root cause

---

*Part of ongoing support and debugging work on [`intel/llm-scaler`](https://github.com/intel/llm-scaler).*