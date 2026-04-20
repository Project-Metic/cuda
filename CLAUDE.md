# CLAUDE.md — cuda

CUDA-accelerated analysis kernels for Project-Metic. GPU-accelerated implementations of
binary analysis passes, symbolic analysis, and proof obligation pre-processing that benefit
from parallel execution on NVIDIA hardware.

## What This Repo Is

CUDA kernel implementations for computationally intensive Metic analysis passes.
These kernels are called from the Python and Rust layers when GPU acceleration is available.

## Key Invariants

- **CUDA kernels are acceleration paths, not the authoritative implementation.** Every CUDA
  kernel must have a CPU-equivalent reference implementation. The reference implementation
  is the authoritative correctness baseline; CUDA results must match it within numerical
  precision bounds.
- **Never use CUDA results in proof certificates without CPU cross-validation.** GPU floating-
  point arithmetic can diverge from CPU. Any result that flows into a verification certificate
  must be cross-validated on CPU before the certificate is issued.
- **Mock mode required for CI.** Tests must run in mock/CPU mode when CUDA is unavailable.
  Never gate CI on GPU availability.

## Dev Commands

```bash
# Build CUDA kernels (requires CUDA toolkit)
mkdir build && cd build
cmake .. -DCUDA_ENABLED=ON
make -j$(nproc)

# Run tests (CPU mode if no GPU)
pytest tests/ -v
# or: ctest --output-on-failure
```

## Tech Stack

- CUDA C++, Python (ctypes or pybind11 bindings), CMake

## Integration with Project-Metic

- Invoked by `metic-api` and `project-metic-mono` analysis passes when GPU is available
- Falls back to CPU reference implementation when CUDA is unavailable
- GPU-accelerated taint analysis and symbolic execution passes

## What NOT to Do

- Do not use CUDA kernel output in certificates without CPU cross-validation
- Do not require GPU for CI (always provide CPU fallback)
- Do not remove CPU reference implementations when adding CUDA acceleration
