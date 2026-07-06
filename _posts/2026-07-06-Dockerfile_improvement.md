---
title: "Dockerfile_improvement"
date: 2026-07-06
---
# Dockerfile Optimization for LLM Scaler: 25 GB → 12.2 GB

*Work in progress on [`lalapotter/llm-scaler@sgl-bmg`](https://github.com/lalapotter/llm-scaler/tree/sgl-bmg), targeting merge into [`intel/llm-scaler`](https://github.com/intel/llm-scaler)*

---

## Motivation

The original `llm-scaler` Docker image weighed in at **25 GB**—unwieldy for CI pipelines, registry storage, and deployment iteration. The goal was to reduce image size significantly without breaking the Intel XPU runtime environment, which carries non-trivial dependency constraints.

Result: **12.2 GB**, a **51% reduction**, with no functional regressions.

---

## What Changed

### Multi-Stage Build with Purpose-Fit Base Images

The single-stage image was restructured into two stages:

| Stage | Base Image | Purpose |
|---|---|---|
| `build` | `omix` | Compile dependencies, install packages, run cleanup |
| `run` | `pytorch+xpu` | Minimal runtime, no build tooling |

Only the artifacts needed at runtime are copied from `build` into `run`, shedding compilers, headers, and intermediate caches that previously bloated the final image.

### Layer Consolidation and Cleanup

Each stage uses a **single `RUN` instruction** as the default strategy. In Docker's layer model, files deleted in a later `RUN` are hidden but still physically present in earlier layers. Consolidating into one `RUN` ensures that deletions—`__pycache__` directories, `.pyc` files, pip caches, and build temporaries—are applied within the same layer and never committed to the image.

The **build stage** was subsequently split into multiple `RUN` instructions grouped by logical phase (dependency resolution, source build, cleanup). This trades a marginal layer overhead for meaningfully better cache reuse during iterative development and easier maintenance. The **run stage** retains a single `RUN` where layer minimisation matters most.

### Runtime Driver Dependencies

Switching the run-stage base to `pytorch+xpu` (rather than inheriting from the original monolithic image) exposed a missing dependency: Intel XPU **runtime drivers** were not pre-installed in that base. These were added explicitly to the run stage. This is the kind of implicit dependency that monolithic images tend to hide and multi-stage builds force into the open.

### Stripping Debug Symbols: A Dead End

`--strip-debug` applied to `.so` files is a common size-reduction technique and was tested here. It caused **runtime errors** and was reverted. This is consistent with shared libraries that embed debug metadata used at runtime—not merely for debugging—such as certain Intel oneAPI and MKL components. The approach is not safe for this image without per-library audit.

---

## Results

| Metric | Before | After |
|---|---|---|
| Image size | 25.0 GB | 12.2 GB |
| Reduction | — | **51%** |
| Runtime errors | None | None |
| XPU driver deps | Implicit | Explicit |

---

## Status

The image is undergoing extended testing on [`lalapotter/llm-scaler@sgl-bmg`](https://github.com/lalapotter/llm-scaler/tree/sgl-bmg) before the branch is proposed for merge into `intel/llm-scaler`. The `--strip-debug` finding is documented as a known non-starter for future reference.

---

*Part of ongoing infrastructure work on [`intel/llm-scaler`](https://github.com/intel/llm-scaler).*