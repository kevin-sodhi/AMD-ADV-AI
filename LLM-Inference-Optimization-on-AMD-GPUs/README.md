# LLM Inference Optimization on AMD GPUs

Hands-on exploration of large language model inference and GPU kernel optimization on AMD Instinct hardware using **ROCm, AMD ATOM, vLLM, AITER, and PyTorch**.

The project explores two related ML systems problems:

1. Serving and benchmarking a large open-weight LLM with optimized inference techniques.
2. Iteratively optimizing a GEMM kernel for AMD MI300X GPUs and analyzing its performance using a roofline model.

## Overview

Modern LLM performance depends on more than the model itself. Serving architecture, attention implementations, KV-cache configuration, speculative decoding, memory movement, and accelerator-specific kernels can all significantly affect inference performance.

This project gave me hands-on experience exploring these layers on AMD GPUs, from serving an 80B-parameter mixture-of-experts model to working with low-level GEMM kernel optimizations.

## Tech Stack

- **AMD ROCm**
- **AMD ATOM**
- **AMD AITER**
- **AMD Instinct MI300X**
- **vLLM**
- **PyTorch**
- **Qwen3-Next-80B-A3B-Instruct-FP8**
- **Python / Jupyter**
- **HIP / GPU kernels**

## Part 1 — LLM Inference & Benchmarking

I served:

`Qwen/Qwen3-Next-80B-A3B-Instruct-FP8`

using AMD ATOM and explored **Multi-Token Prediction (MTP)** for accelerated inference.

The notebook includes infrastructure for launching and monitoring the inference server and benchmarking it under configurable workloads.

Benchmark parameters include:

- Input sequence length
- Output sequence length
- Number of prompts
- Request concurrency
- Request rate
- Dataset configuration

The goal is to examine how inference configuration affects serving performance rather than treating the model server as a black box.

### Performance Areas Explored

- Inference throughput
- Request latency
- Concurrent request handling
- Multi-Token Prediction
- FP8 inference
- GPU memory utilization
- KV-cache configuration
- Asynchronous scheduling
- CUDA/HIP graph execution

## Part 2 — vLLM + ATOM

I also configured the model through the ATOM/vLLM serving path with several performance-oriented settings, including:

- FP8 KV cache
- asynchronous scheduling
- optimized model loading
- graph capture/compilation
- high GPU-memory utilization
- large-context configuration

This helped me explore how model-serving frameworks expose hardware and runtime optimizations that affect LLM inference behavior.

## Part 3 — GEMM Kernel Optimization on MI300X

The second part moves below the model-serving layer into GPU kernel performance.

Starting from a deliberately inefficient GEMM implementation, I explored an optimization path targeting AMD MI300X.

The baseline kernel intentionally lacks several important GPU optimizations:

- shared-memory tiling
- vectorized memory access
- wavefront-aware execution
- prefetching / double buffering

The optimization sequence explored:

```text
Naive GEMM
    ↓
CUDA → HIP port
    ↓
Shared-memory tiling
    ↓
Vectorized loads
    ↓
MI300X wavefront-aware tuning
    ↓
Double buffering / prefetching
```

This exercise helped connect high-level LLM inference performance with the lower-level compute and memory behavior of GPU kernels.

## Roofline Analysis

Kernel benchmark results can be analyzed using a roofline model.

The notebook plots measured GEMM performance against the approximate compute and memory limits of the MI300X:

- Peak FP performance
- Memory bandwidth
- Arithmetic intensity
- Observed TFLOP/s

This provides a way to reason about whether a kernel is primarily **compute-bound or memory-bound**, rather than evaluating optimization only through raw execution time.

## What I Learned

The main takeaway from this project was that LLM inference performance is a full-stack systems problem.

Performance depends on interactions between:

```text
Model Architecture
        ↓
Inference / Serving Runtime
        ↓
Attention + KV Cache
        ↓
GPU Kernels
        ↓
Memory Hierarchy
        ↓
Accelerator Hardware
```

Working through both the model-serving and kernel layers gave me a better understanding of where inference bottlenecks can originate and why optimization requires measuring the system at multiple levels.

It also motivated my interest in **ML systems research**, particularly LLM inference, accelerator-aware optimization, benchmarking, and understanding performance trade-offs across models, serving frameworks, and hardware.

## Repository Contents

`atom_llm_demo.ipynb` — Main notebook containing the inference-serving, benchmarking, and kernel-optimization workflow.

The notebook covers:

- environment and ROCm validation
- ATOM model serving
- Qwen3-Next-80B-A3B inference
- MTP configuration
- configurable inference benchmarking
- ATOM/vLLM serving
- GEMM optimization workflow
- MI300X roofline analysis

## Background

This work was completed as part of hands-on experimentation with AMD's AI and ROCm ecosystem. The repository preserves my implementation, experiments, and notes as I developed a deeper understanding of LLM inference and GPU performance engineering.

## Areas I'm Exploring Next

I'm particularly interested in continuing to explore:

- KV-cache management and memory efficiency
- continuous batching and scheduling
- speculative decoding
- attention kernel optimization
- inference performance across different serving backends
- accelerator-aware LLM serving
- reproducible benchmarking of LLM systems
