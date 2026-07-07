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

Initially, each stage uses a **single `RUN` instruction** as the default strategy. In Docker's layer model, files deleted in a later `RUN` are hidden but still physically present in earlier layers. Consolidating into one `RUN` ensures that deletions—`__pycache__` directories, `.pyc` files, pip caches, and build temporaries—are applied within the same layer and never committed to the image.

Later, the **build stage** was split into multiple `RUN` instructions grouped by logical phase (dependency resolution, source build, cleanup). This trades a marginal layer overhead for meaningfully better cache reuse during iterative development and easier maintenance. The **run stage** retains a single `RUN` where layer minimisation matters most.

### How the Savings Break Down

Comparing the baseline `docker history` against the optimized one, the ~13 GB reduction comes from a few concentrated sources rather than broad trimming:

- **Dropping the `intel-deep-learning-essentials` meta-package (~8 GB).** The baseline pulled in the full oneAPI toolchain (compiler, MKL, DNNL, MPI, TBB, debugger) as a single ~7.96 GB layer. Because the build tooling now lives only in the `build` stage and the `run` stage starts from `pytorch+xpu`, this entire toolchain no longer ships in the final image.
- **Building from pre-built wheels instead of source (~4 GB across three layers).** The baseline ran `git clone` + compile for `sglang` (~1.41 GB), `sgl-kernel-xpu` (~1.49 GB), and a separate multi-GB torch/vision/audio + triton install (~11.4 GB). The optimized run stage installs from wheels (`pip3 install /tmp/wheels/*.whl`), collapsing the application and kernel install into a single cleaned ~2.81 GB layer and avoiding cloned repos and compiler intermediates.
- **Cache and bytecode purging within the same layer.** Setting `PIP_NO_CACHE_DIR=1` plus `find ... -name "*.pyc" -delete`, removing `__pycache__`, and `rm -rf /root/.cache /root/.cmake /var/tmp/* /var/lib/apt/lists/*` in the same `RUN` keeps build residue out of the committed layers entirely. In the baseline these caches were baked into the multi-GB install layers.

In short, most of the win is structural—**not shipping the build toolchain and not compiling inside the final image**—rather than removing runtime functionality.

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
| XPU driver deps | Implicit | Explicit |

### Layer-to-Layer Comparison

Layers are matched **by function**, not by identical commands: the baseline builds on a full oneAPI stack and compiles from source, while the optimized image builds on `pytorch+xpu` and installs pre-built wheels. Sizes are taken directly from each `docker history`.

| Functional layer | Baseline (before) | Optimized (after) | Δ | Why it changed |
|---|---|---|---|---|
| Base OS (Ubuntu 24.04 `ADD`) | 78.1 MB | 78.1 MB | ~0 | Same base rootfs |
| Base build/apt bootstrap (curl, ca-certs, gpg, software-properties) | 150 MB | 502 MB | +352 MB | Optimized front-loads build toolchain into the `build` stage |
| Intel GPU runtime repo + drivers | 375 MB | 371 MB | ~0 | Equivalent GPU runtime; roughly a wash |
| **oneAPI toolchain (`intel-deep-learning-essentials`)** | **7.96 GB** | **0** | **−7.96 GB** | Confined to `build` stage; never shipped in runtime image |
| Python + pip/venv setup | 52.2 MB | 76.7 MB + 18.7 MB | +43 MB | Optimized uses an isolated venv (`/opt/venv`) |
| **torch + torchvision + torchaudio + triton** | **11.4 GB** | **8.33 GB** | **−3.07 GB** | Wheel install vs. source; no build intermediates retained |
| **`sglang` (app)** | **1.41 GB** | see combined ↓ | — | Built from wheel instead of `git clone` + compile |
| **`sgl-kernel-xpu`** | **1.49 GB** | see combined ↓ | — | Built from wheel instead of `git clone` + compile |
| **App + kernel + deps + in-layer cleanup (combined)** | (1.41 + 1.49 GB above) | **2.81 GB** | **−0.09 GB** and caches purged | Collapsed into one cleaned `RUN`; `.pyc`/`__pycache__`/caches removed in-layer |
| multi-arc installer + locale-gen + extra apt runtime | 1.69 GB + 3.07 MB + 135 MB | — | **−~1.83 GB** | Not required in the runtime image |
| App source copy (`COPY . .`) | n/a | 1.86 MB | +1.86 MB | Ships app code directly instead of cloning at build time |
| **Total image** | **~25.0 GB** | **~12.2 GB** | **−12.8 GB (51%)** | Structural: no toolchain shipped, no in-image compilation |

> Note: baseline `sglang` and `sgl-kernel-xpu` are two separate source-build layers (1.41 GB + 1.49 GB); in the optimized image they are installed from wheels and folded into the single 2.81 GB run-stage layer alongside dependency install and cleanup, so they map many-to-one.

---

## Status

The image is undergoing extended testing on [`lalapotter/llm-scaler@sgl-bmg`](https://github.com/lalapotter/llm-scaler/tree/sgl-bmg) before the branch is proposed for merge into `intel/llm-scaler`. The `--strip-debug` finding is documented as a known non-starter for future reference.

---

*Part of ongoing infrastructure work on [`intel/llm-scaler`](https://github.com/intel/llm-scaler).*