---
title: "Runtime_stage_only_Dockerfile"
date: 2026-07-21
---

# Decoupling LLM Scaler Docker Builds with Pre-Built Wheel Artifacts

My Docker image optimization work for LLM Scaler has now been **merged into the opensource codebase**.

[Feat/dockerfile scripts cleanup](https://github.com/intel/llm-scaler/pull/540)

The multi-stage Dockerfile significantly reduced the runtime image size by keeping compilers, build dependencies, and intermediate files out of the final image. However, the build stage still had an important operational cost: compiling the Python wheels could take **up to an hour**.

That wait time became a problem for internal consumers of the image, particularly teams that need fast iteration or frequent rebuilds. The next improvement is therefore not primarily about image size—it is about separating the expensive compilation work from routine runtime-image builds.

The below work saved the Docker image build time significantly, from **81mins -> 12mins for vllm**.

## The Problem: Wheel Compilation Dominates Build Time

The optimized multi-stage Dockerfile has two main responsibilities:

1. A **build stage** installs build dependencies and compiles the required wheels.
2. A **runtime stage** installs those wheels into the minimal XPU runtime environment.

Although only the runtime stage is shipped, every full Docker build may still need to execute the expensive build stage. Compiling wheels for components such as SGLang and XPU kernels can take close to an hour, even when the final runtime image itself is relatively straightforward to assemble.

This creates an unnecessary coupling:

- A runtime-image change can trigger a costly wheel rebuild.
- Teams consuming the runtime image must wait for compilation even if the application dependencies have not changed.
- The build output is not yet treated as a reusable artifact.

## The New Approach: Publish the Build Stage as an Artifact Image

The solution is to turn the build stage into a standalone, reusable Docker image.

Instead of always running a complete multi-stage build, the wheel-compilation stage will be built and published independently. This image is intentionally a **dummy build-stage-only image**: it is not intended to run the LLM Scaler service. Its purpose is to hold the compiled wheel artifacts produced by the builder environment.

Conceptually, the workflow becomes:

```text
Source + build dependencies
        │
        ▼
Build-stage-only image
(compiles and contains wheels)
        │
        ▼
Extract or copy wheel artifacts
        │
        ▼
Runtime-only Dockerfile
(installs wheels into the production runtime image)
```

This makes the expensive compilation step explicit, cacheable, and independently publishable.

## Step 1: Build a Dedicated Build-Stage Image

The first artifact is an image containing only the build environment and its generated wheels.

This image includes the dependencies required to compile the packages, as well as the resulting wheel files in a known directory such as:

```text
/tmp/wheels/
```

It may be larger than the production image and can include compilers, headers, and other build tooling. That is acceptable because it is an intermediate artifact, not the image deployed to users.

The important contract is simple:

> The build-stage image produces a complete, versioned set of wheels that can be installed by the runtime image without rebuilding from source.

## Step 2: Extract the Wheels

Once the build-stage image is available, the wheels can be retrieved from it as build artifacts.

For example, the wheel directory can be copied from a built container:

```bash
docker create --name llm-scaler-wheel-artifact llm-scaler-build:latest
docker cp llm-scaler-wheel-artifact:/tmp/wheels ./wheels
docker rm llm-scaler-wheel-artifact
```

The extracted wheel set can then be:

- archived and stored in the internal artifact system,
- attached to a release pipeline,
- published as a versioned build artifact, or
- copied directly into a runtime-image build context.

Versioning the wheel artifact is important. The runtime image must use wheels produced from the corresponding source revision and compatible dependency set.

## Step 3: Build the Runtime Image from Pre-Built Wheels

The production Dockerfile can now be simplified to a single runtime stage.

Rather than cloning repositories or compiling packages, it starts from the XPU runtime base image, copies in the pre-built wheels, installs them, and removes the temporary wheel directory in the same layer.

```dockerfile
FROM <pytorch-xpu-runtime-base>

ENV PIP_NO_CACHE_DIR=1 \
    VIRTUAL_ENV=/opt/venv \
    PATH="/opt/venv/bin:${PATH}"
    ...

COPY ./vllm-engine/wheels/ /tmp/wheels/

RUN python3 -m venv "${VIRTUAL_ENV}" \
    && pip install --upgrade pip \
    && pip install /tmp/wheels/*.whl \
    &&
    ...
    ...
    ...
    && rm -rf /tmp/wheels \
    && find "${VIRTUAL_ENV}" -type d -name "__pycache__" -prune -exec rm -rf {} + \
    && find "${VIRTUAL_ENV}" -type f -name "*.pyc" -delete \
    && rm -rf /root/.cache /var/lib/apt/lists/*
```

This keeps the runtime Dockerfile single-stage from the perspective of its own build logic: it consumes a ready-made dependency artifact rather than performing compilation itself.

## Benefits

### Faster Runtime Builds

The largest benefit is that routine runtime-image builds no longer wait for wheel compilation. If the wheels have not changed, a runtime-image update can proceed directly from the published wheel artifact.

### Better CI Pipeline Separation

The pipeline can now distinguish between two kinds of changes:

| Change type | Required work |
|---|---|
| Python package, kernel, or dependency-source change | Rebuild and publish wheels, then build runtime image |
| Runtime Dockerfile, configuration, or application-only change | Reuse existing wheels and rebuild only the runtime image |

This removes the need to pay the compilation cost for every image rebuild.

### Reusable, Traceable Artifacts

The wheel bundle becomes a first-class artifact. It can be versioned, promoted through environments, and reused by multiple image builds without repeating the same compilation work.

### Retained Runtime Image Benefits

This approach preserves the earlier optimization work:

- the final image remains free of compilers and build toolchains;
- runtime dependencies remain explicit;
- pip caches, bytecode, and temporary files can still be removed in the same image layer;
- the final image continues to install from wheels rather than compiling packages in production.

## Trade-Offs and Operational Considerations

Separating wheel production from runtime-image assembly introduces an artifact-management responsibility.

The build pipeline must ensure that:

- wheels are tagged with the source revision or release version that produced them;
- the runtime base image is compatible with the wheels;
- the artifact repository retains the required wheel versions;
- security scanning covers both the builder artifact and the final runtime image;
- a wheel rebuild is triggered whenever relevant native dependencies, build inputs, or package sources change.

The additional pipeline structure is worthwhile because it changes a repeated one-hour compile step into a reusable, versioned dependency artifact.

## Conclusion

The original multi-stage Dockerfile optimization addressed runtime image size by ensuring that build tooling never shipped to production. This follow-up improves developer and internal consumer experience by ensuring that expensive compilation does not occur during every runtime-image build.

By publishing a dedicated build-stage image, extracting its wheels, and building the runtime image directly from those pre-built artifacts, the LLM Scaler pipeline can retain a compact production image while substantially reducing routine build latency.

Note that all the above work done is internal only, and may be published later.