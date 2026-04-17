---
name: memory-optimization
description: "Use when diagnosing GPU OOM errors, optimizing batch sizes, or reducing memory footprint of ML training. Covers memory estimation, gradient checkpointing, mixed precision, and batch size binary search. Framework-agnostic base skill — extend with project-specific overlays."
---

# Memory Optimization

**Related skills:** For the full experiment lifecycle, see `model-experiment`. For monitoring active runs, see `monitor-training-runs`.

## Overview

GPU memory is the primary constraint for training large models. This skill covers: estimating memory requirements, finding maximum batch size, reducing memory via gradient checkpointing and mixed precision, and diagnosing OOM errors.

## Memory Estimation Formula

```
Total GPU memory ≈ Model params (fp32/fp16)
                 + Optimizer states (2× params for Adam)
                 + Activations (batch × seq_len × d_model × n_layers × ~2)
                 + Gradient accumulation buffer
                 + Framework overhead (~500 MB)
```

**Quick estimate for transformers:**
```python
params_mb = n_params * 4 / 1e6        # fp32
optimizer_mb = params_mb * 2           # Adam m + v
activations_mb = batch_size * seq_len * d_model * n_layers * 2 * 4 / 1e6
total_mb = params_mb + optimizer_mb + activations_mb + 500
```

## Finding Maximum Batch Size

**Binary search approach:**

1. Start with `batch_size = available_vram_gb * 128` (rough heuristic)
2. Run 5 training steps
3. If OOM: halve batch size
4. If succeeds: note GPU memory used, try 1.5× batch size
5. Target: 70-85% of GPU VRAM used (leave headroom for spikes)

```bash
# After first training step:
nvidia-smi --query-gpu=memory.used,memory.total --format=csv,noheader
# OR in PyTorch:
python -c "import torch; print(f'{torch.cuda.max_memory_allocated()/1e9:.1f} GB')"
```

**Use gradient accumulation** when batch size is memory-limited:
```python
effective_batch = batch_size * grad_accum_steps
# Target: effective_batch >= 128 for stable transformer training
```

## Memory Reduction Techniques

### 1. Mixed Precision (bf16/fp16)

Halves memory for activations and most parameters.

**PyTorch:**
```python
from torch.cuda.amp import autocast, GradScaler
scaler = GradScaler()
with autocast(dtype=torch.bfloat16):
    output = model(input)
    loss = criterion(output, target)
scaler.scale(loss).backward()
scaler.step(optimizer)
scaler.update()
```

**Keep in fp32:** LayerNorm, softmax, loss computation, variance heads.

### 2. Gradient Checkpointing

Trade compute for memory — recompute activations during backward instead of storing them.

**PyTorch:**
```python
from torch.utils.checkpoint import checkpoint
# Wrap expensive layers:
output = checkpoint(self.cross_attention_layer, query, context, use_reentrant=False)
```

**Saves:** ~60% of activation memory at cost of ~30% slower training.

### 3. Optimizer Memory

| Optimizer | Memory per param | Notes |
|-----------|-----------------|-------|
| AdamW | 8 bytes (2× fp32 states) | Default choice |
| SGD | 4 bytes (1× momentum) | Less memory, slower convergence |
| 8-bit Adam | 2 bytes (quantized states) | `bitsandbytes` library |
| Adafactor | ~4 bytes | Factored second moments |

### 4. Activation Offloading

Move activations to CPU during forward, reload during backward. Last resort — very slow.

## OOM Debugging Checklist

When you get `CUDA OutOfMemoryError`:

1. **What's the batch size?** Reduce by 50% and retry
2. **Is mixed precision enabled?** Enable bf16
3. **Is gradient checkpointing on?** Enable for transformer layers
4. **Are there memory leaks?** Check for tensors retained between steps: `.detach()` missing on stored values
5. **Is the context length too long?** Cap sequence length and pad
6. **Are frozen model params on GPU?** Move to CPU if not needed for forward
7. **Is there a rogue process?** `nvidia-smi` — kill anything unexpected

## Project-Specific Overlays

This is the generic base skill. Create project-specific overlays that extend it with:
- Concrete paths (venv, data, results)
- GPU assignments
- Validated batch sizes per model configuration
- Known bottleneck tensors
