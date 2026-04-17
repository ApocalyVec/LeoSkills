---
name: model-experiment
description: "Use when starting, monitoring, debugging, iterating on, or restarting an ML experiment. Covers the full lifecycle: data inspection → pre-flight checklist → launch → live monitoring → crash triage → diagnosis → fix → re-run. Framework-agnostic (PyTorch, JAX, TensorFlow). Triggers on: launching training, investigating crashes, interpreting loss curves, diagnosing mode collapse/OOM/data errors, deciding whether to kill a run, or iterating on model performance."
---

# Model Experiment — Full Lifecycle

**Related skills:** For quick monitoring checks during active training, see `monitor-training-runs`. For GPU memory/OOM debugging, see `memory-optimization`.

## Overview

A training run is a multi-hour commitment. This skill defines the full lifecycle: data inspection → pre-flight → launch → monitor → diagnose → fix → re-run. Architecture and loss changes require user approval.

**Key principle:** The prep phase IS the experiment. GPU-hours wasted on misconfigured data or wrong loss weights cost more than careful review. Slow is smooth; smooth is fast.

---

## Phase 0: Data Inspection — BEFORE ANYTHING ELSE

**Show the user the input data. This is non-negotiable.**

Before any training run, generate and present:

1. **Per-feature statistics table**: min, max, mean, std for **every** input feature across **every** modality (market features, macro features, event features, text features, etc.). One table per modality. No feature may be omitted.
2. **Scale check**: flag features with std ratio > 100x (normalization needed). Show a ranked list of all features by std to visualize the spread.
3. **Class balance**: for classification targets, show positive/negative/neutral split. For regression targets, show distribution stats (mean, std, skew, min, max, percentiles). Show **before and after** any label transformations (e.g., rank normalization).
4. **Sample count**: total samples, per-group coverage, temporal coverage. Include: samples-per-ticker distribution (mean/median/min/max), samples-per-date distribution, year-by-year breakdown, train/val split sizes, and which tickers were skipped and why.
5. **Token/modality breakdown**: if multi-modal, show count and dimension per source type (mean/median/min/max tokens per sample). Include the encoder each modality routes through and its input→output dimensions.
6. **Missing data**: what percentage of samples have each modality. Show a bar chart or visual indicator for each. Flag any modality below 50% coverage.
7. **Sample examples**: show 3-5 actual input samples with their targets. Include ALL fields: raw feature values, target values at each horizon, macro snapshot, event distances, and modality availability flags.

**If any of these look wrong, STOP and fix before training.** Most model failures trace to data issues (Andrew Ng's data-centric AI principle).

---

## Phase 1: Experiment Pre-Flight Report (EPR)

**Every run requires a pre-flight report saved to a Markdown file. No exceptions.**

### Output format

Save the complete report (Phase 0 + Phase 1 combined) as a Markdown file in the project's experiment tracking location (e.g., `docs/experiments/`, `experiment_logs/`, or the project's knowledge base). The filename should include the experiment number and date, e.g., `experiment_6_preflight_2026-04-17.md`. The report must be a **complete standalone document** — someone reading it with no context should understand exactly what data goes in, what model processes it, how training is configured, why this experiment exists, and what success looks like.

### EPR contents

The report must include ALL of the following. No section may be omitted or abbreviated.

1. **Dataset manifest** — n_samples, n_tickers, n_dates, date range, train/val split, class distribution before and after any transforms, feature dimensions per modality, mean/max sequence length, modality coverage percentages
2. **Model config** — full architecture description, every hyperparameter (d_model, n_layers, n_heads, head_dim, dropout, etc.), total parameter count, per-component parameter breakdown with percentages, which components are frozen, description of each prediction head (input, activation, output)
3. **Training hyperparameters** — optimizer (type + all settings), learning rate (peak + schedule + warmup steps), weight decay, batch size (physical + gradient accumulation + effective), gradient clipping value, max epochs, early stopping config, total estimated steps. Loss function: full specification including every component, every weight, every gamma/epsilon, per-horizon configuration if applicable.
4. **GPU plan & batch optimization** — GPU model + VRAM, CUDA version, driver version. VRAM estimate broken down: model params, optimizer states, activations, loss overhead. Utilization percentage. Recommendation on batch size given available VRAM. **This must be done BEFORE the smoke test** — the smoke test validates the chosen batch size.
5. **Hypothesis** — what specific question this experiment answers, what root cause it addresses, what changes were made and why, explicit success/failure criteria with numeric thresholds, failure modes to watch for during training
6. **Previous experiments** — table of all prior runs with: experiment number, config summary, primary metric result, val loss, one-line finding. Include baselines.
7. **Complete change log** — every change vs the previous experiment, categorized (DATA / MODEL / TRAIN), with before→after values where applicable
8. **GPU batch size optimization results** — Binary search for max batch size: start large, halve on OOM. Report actual GPU memory at the chosen batch size via `torch.cuda.max_memory_allocated()`. Target >50% VRAM utilization. If model is small relative to GPU, increase batch size or use gradient accumulation. **This step determines the final batch size used in the smoke test.**
9. **Smoke test results** — 50-step run at the optimized batch size: loss values at step 0/25/50, gradient norm at step 50, actual GPU memory usage, step time, whether loss is finite and trending downward. This is the final gate before full training launch.

### GPU Utilization Check (execute steps 4 and 8 above)

**Always maximize batch size for available VRAM.** The procedure is:

1. Estimate memory: `batch_size × seq_len × d_model × 4 bytes × ~3 (activations + gradients + optimizer)`
2. Start with a large batch size, binary search down if OOM
3. Use gradient accumulation for larger effective batch sizes
4. Report actual GPU memory after first batch: `nvidia-smi` or `torch.cuda.max_memory_allocated()`
5. If GPU utilization < 50%, the batch size is too small
6. **Target:** effective_batch_size ≥ 128-256 for stable transformer training
7. Once optimal batch size is found, proceed to smoke test with that batch size

### Go/No-Go checklist
- [ ] Data statistics reviewed — no scale imbalances > 100x (or normalizer in place)
- [ ] GPU batch size optimized — using >50% of available VRAM
- [ ] Smoke test passed — loss finite and decreasing at step 50
- [ ] Hypothesis documented — clear success criteria with numeric thresholds
- [ ] Previous experiment results noted — what changed from last run
- [ ] All changes categorized and documented in change log

---

## Phase 2: Launch + Monitoring

### MONITORING IS NON-NEGOTIABLE

**Runs WILL crash or degrade without warning. If you are not monitoring, you will not know.**

After every launch:
1. Check GPU usage after 60 seconds: `nvidia-smi`
2. Verify first epoch completes: check log for loss values
3. Monitor every 15-30 minutes during active training

### What to Log (TensorBoard or equivalent)

**Per step:** train loss, direction/classification accuracy, gradient norm (total)
**Per epoch:** val loss, val accuracy, learning rate, gate statistics (if gated architecture)
**Every N steps:** per-layer gradient norms, per-layer parameter norms

### How to Report Monitoring Results

When checking on a training run, report BOTH absolute values AND trends for every monitored quantity. A single snapshot is not enough — the user needs to see whether things are converging, diverging, or stuck.

For each metric, report:
1. **Current value** (latest step/epoch)
2. **Starting value** (step 0 or epoch 0)
3. **Trend direction**: converging (toward expected value), diverging (away from it), flat (not moving), or oscillating
4. **Rate of change**: fast, slow, or stalled — e.g., "decreased 3% over 150 steps" vs "decreased 0.01% over 150 steps (effectively frozen)"
5. **Flag anomalies**: any non-monotonic behavior, sudden jumps, values that should be changing but aren't

**Trend analysis is especially important for:**
- **Learned loss weights** (e.g., Kendall sigmas): Are they differentiating between tasks, or frozen at init?
- **Gate statistics**: Are gates opening/closing to specialize, or stuck at initialization?
- **Per-component parameter norms**: Is every component receiving gradients and updating, or are some frozen?
- **Per-horizon losses**: Are all horizons improving equally, or is one dominating/stuck?
- **Gradient norms**: Stabilizing (good), growing (potential explosion), or shrinking (vanishing)?

Example of good monitoring report:
```
Gradient norm: 26.0 → 2.85 over 150 steps (CONVERGING, stabilizing)
Gate L0 sigmoid: 0.5005 → 0.5005 over 150 steps (FLAT — expected during warmup, flag if still flat at epoch 5)
Kendall sigma H=1: 1.000 → 1.002 over 150 steps (FLAT — not yet differentiating)
Regime head param norm: 0.518 → 0.518 over 150 steps (FROZEN — no gradient flow, investigate)
```

### Alert Conditions

| Condition | Action |
|-----------|--------|
| Loss is NaN or Inf | **Stop immediately** — check data pipeline, reduce LR |
| Gradient norm > 100 | Gradient explosion — reduce LR, increase clipping |
| Gradient norm < 1e-7 after epoch 2 | Gradient vanishing — check stop-gradient, increase LR |
| Gradient norm trend: growing epoch over epoch | Approaching instability — reduce LR or increase clipping |
| Metric stuck at same value for 3+ epochs | Mode collapse — see Diagnosis section |
| Learned weights (sigmas, gates) unchanged after 3+ epochs | Component not receiving gradients or LR too low for that component |
| Any parameter norm frozen (0% change) after warmup | That component's gradients are blocked — check stop-gradient, detach, or architecture |
| Per-task losses diverging (one improving, another worsening) | Task conflict — check loss weighting, consider gradient surgery |
| GPU memory drops to ~0 | Process crashed — check logs for traceback |
| No log output for > 2× epoch time | Process hung or JIT compiling — check process status |

### When to Kill Early

**Kill immediately:** NaN loss, confirmed data pipeline bug, process crashed
**Kill after 500 steps of investigation:** Loss spike that doesn't recover, metric stuck at init value
**Let it run:** Slow plateau during LR annealing, single-step spikes that self-correct

---

## Phase 3: Diagnosis — When Things Go Wrong

### Decision Framework: What to Change

**Priority order** (data-centric AI principle):

1. **Fix data first** — normalization, missing modalities, label quality, leakage. Largest gains per effort.
2. **Change training procedure second** — loss function, LR, batch size, regularization. Cheaper than architecture.
3. **Change the model last** — only when you can diagnose a specific bottleneck.

**One change at a time.** If you change both loss and data, you can't know which helped.

### Common Failure Modes

| Symptom | Check First | Likely Fix |
|---------|------------|-----------|
| Metric stuck at base rate (majority class) | Class balance, loss function, gradient flow to prediction head | Focal loss, class weighting, separate head warmup, higher prediction head LR |
| Loss decreases but metric doesn't | A different loss component dominates (e.g., variance head) | Increase weight on stuck metric's loss term |
| Train improves, val doesn't | Overfitting | More data, dropout, augmentation, smaller model |
| Both plateau high | Underfitting | Better features, larger model, more data |
| Loss oscillates | LR too high, batch too small | Reduce LR, increase batch, gradient clipping |
| All outputs identical | Dead neurons or collapsed representation | Check init, reduce LR, inspect activations |
| Direction accuracy stuck at exactly majority% | BCE local minimum at base rate | Focal loss (gamma=2+), or train head separately with higher LR first |

### Crash Triage Order (fastest to slowest)

```
1. Infrastructure (GPU/disk/memory) → 2. Data pipeline → 3. Model forward pass → 4. Training loop
```

**NEVER restart without identifying root cause.** Restarting a broken config will just crash again.

---

## Phase 4: Architecture Change Gate

**STOP. Discuss with the user before implementing any of:**

- Adding or removing model components
- Changing loss formulation
- Changing optimizer or LR schedule
- Changing data mixing or modality inclusion
- Changing evaluation metrics

Present: (1) what the change is, (2) why, (3) tradeoff, (4) what tests verify it.

Minor fixes (collate bugs, shape mismatches, normalization) can be implemented directly.

---

## Phase 5: After Training — What to Show the User

1. **Results table**: this run vs previous runs vs baseline (same metrics, side by side)
2. **Loss curves**: train and val per epoch
3. **Metric trajectory**: primary metric per epoch (direction accuracy, IC, Sharpe)
4. **Sample predictions**: 5 best, 5 worst with context
5. **What changed**: one sentence on what was different from last run
6. **Next hypothesis**: what this result suggests trying next

---

## Phase 6: Fix → Test → Re-run

1. Identify root cause from diagnosis
2. Fix ONE thing
3. Run existing tests to confirm fix doesn't break anything
4. Update Experiment Log with: config, result, finding, next step
5. Launch new run with clear version increment
6. Resume monitoring immediately

---

## Experiment Tracking

Every run must be logged with:

| Field | Example |
|-------|---------|
| **Run #** | Experiment 5 |
| **Date** | 2026-04-16 |
| **Config** | d=256, 6L, 8H, lr=3e-4, focal(γ=2), batch=128 |
| **Context** | Market(60, normalized) + Macro(1) + Event(1-3) + Filing(0-10) |
| **Result** | dir_acc=56.6%, val_loss=-2.41, 49s/epoch |
| **Finding** | Multi-modal context didn't break mode collapse |
| **Next** | Try separate direction head LR or curriculum learning |

---

## Checkpoint Selection

Val loss alone is unreliable for model selection (arXiv:2410.05612, arXiv:2504.12491 show >50% error rate).

**Protocol:**
1. Save at least 3 recent checkpoints + periodic saves
2. Select by downstream task metric, not raw val loss
3. Never use the final checkpoint without comparing to 70-90% checkpoint
4. For general-purpose: the checkpoint at 70-90% of training usually outperforms the final one

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Launching without checking data distributions | Phase 0 — always inspect first |
| Launching without GPU memory check | Estimate VRAM, do smoke test |
| Skipping smoke test | Always 50-step test before full run |
| Debugging model without running existing tests | Check test suite first |
| Architecture changes without discussion | Always discuss — even "obvious" ones |
| Restarting with identical config after crash | Find root cause first |
| Overwriting logs on restart | Use `>>` not `>` to preserve crash history |
| Small batch size on large GPU | Maximize batch for available VRAM |
| Changing multiple things per experiment | One change at a time |
| Not showing the user data before training | Phase 0 is non-negotiable |

---

## Framework-Specific: JAX/XLA

If working in a JAX codebase, these additional considerations apply:

### JIT Compilation

| Situation | Expected Time | Action |
|-----------|:---:|--------|
| Small model, single GPU | 30-60s | Normal |
| Large model with tokenizers | 2-5 min | Normal, wait |
| pmap with conv tokenizers | Can exceed 4 hours | Switch to `jit+Mesh` sharding instead |
| JIT > 4 hours | Stuck | Set `XLA_FLAGS="--xla_gpu_strict_conv_algorithm_picker=false"` and restart |

### XLA-Specific OOM Signals

| Error Message | Cause | Fix |
|---------------|-------|-----|
| `RESOURCE_EXHAUSTED: Out of memory while trying to allocate X bytes` | GPU OOM | Reduce batch, enable remat |
| `MemoryError` in data loader | CPU RAM exhaustion (not GPU) | Reduce prefetch, check swap: `free -h` |
| `unbound axis name` | `jax.lax.all_gather` outside pmap | Check sharding flags |
| cuDNN workspace OOM (conv layers) | cuDNN algorithm search allocates workspace | Use `use_fft_conv=True` to route through cuFFT instead |

### JAX Gradient Checkpointing (Rematerialization)

```python
import jax
# Wrap expensive layers:
@jax.checkpoint
def expensive_layer(params, x):
    return model.apply(params, x)
```

Use `jax.checkpoint` (not `jax.remat` which is deprecated).

### pmap vs jit+Mesh

| Approach | When to Use | Gotcha |
|----------|------------|--------|
| `jax.pmap` | Legacy, simple data parallelism | Can cause >4hr JIT with conv layers; cuDNN autotuner iterates per-device |
| `jax.jit` + `jax.sharding.Mesh` | Preferred for multi-GPU | Faster JIT, explicit sharding control, compatible with FSDP |

If a run takes >4 hours to JIT compile under pmap, switch to jit+Mesh.

### JAX-Specific Debugging

```python
# Debug NaN in loss:
jax.debug.print("loss={}", total_loss)
jax.debug.print("grad_norm={}", jax.tree_util.tree_reduce(
    lambda a, b: a + b,
    jax.tree_util.tree_map(lambda g: jnp.linalg.norm(g)**2, grads)
)**0.5)

# Check for inf/nan in any tree:
jax.tree_util.tree_map(lambda x: jnp.any(jnp.isnan(x)), params)
```

### JAX Learning Rate Notes

| Parameter | Recommended | Literature |
|-----------|------------|-----------|
| Peak LR | 1e-4 | MAE ViT-2B uses 1e-4; conservative but stable |
| Warmup steps | 1000-2000 | ~2-5% of training steps |
| AdamW β₂ | 0.99 | Default 0.999 still warming up at step 1000; lower to 0.95 if spikes at steps 200-800 |

### Per-Modality Loss Balance (Multi-Modal JAX)

In multi-modal training, one modality can dominate gradients (e.g., EEG MSE >> wearable MSE). Check at step 500:

| Condition | Action |
|-----------|--------|
| One modality loss >50× others | Add inverse-loss weighting: `w_i = 1/loss_i(step=0)` |
| Any modality loss = 0.0 from step 1 | That modality has no data in batch — check dataset config |
| Alignment loss oscillates without trend | Log positive/negative logit means; reset alignment head if bias drifts |

---

## Framework-Specific: PyTorch

If working in a PyTorch codebase, these additional considerations apply:

### Mixed Precision

```python
# bf16 on Ampere+ GPUs (A100, L40S, H100):
from torch.cuda.amp import autocast, GradScaler
scaler = GradScaler()
with autocast(dtype=torch.bfloat16):
    output = model(input)
    loss = criterion(output, target)
scaler.scale(loss).backward()
scaler.step(optimizer)
scaler.update()
```

**Keep in fp32:** LayerNorm, softmax, loss computation, variance/uncertainty heads.

### Flash Attention

```python
# PyTorch 2.0+: automatic dispatch
output = F.scaled_dot_product_attention(query, key, value, attn_mask=mask)
# Uses FlashAttention-2 when available (Ampere+ GPUs)
```

### Gradient Checkpointing

```python
from torch.utils.checkpoint import checkpoint
# Wrap transformer layers:
output = checkpoint(self.cross_attention_layer, query, context, use_reentrant=False)
# Saves ~60% activation memory at ~30% speed cost
```

### DataLoader Best Practices

```python
DataLoader(
    dataset,
    batch_size=batch_size,
    shuffle=True,
    num_workers=4,           # match CPU cores
    pin_memory=True,         # faster GPU transfer
    persistent_workers=True, # avoid worker respawn overhead
    prefetch_factor=2,       # pre-load next batches
)
```

### Screen/tmux for Long Runs

Training processes die when SSH disconnects. Always use:
```bash
screen -dmS training bash -c 'python train.py > train.log 2>&1'
# Or:
tmux new -d -s training 'python train.py > train.log 2>&1'
```

### PyTorch-Specific Debugging

```python
# Check for NaN in any parameter:
for name, p in model.named_parameters():
    if torch.isnan(p).any():
        print(f"NaN in {name}")

# Gradient flow check:
for name, p in model.named_parameters():
    if p.grad is not None:
        print(f"{name}: grad_norm={p.grad.norm():.6f}")

# Memory profiling:
print(f"Allocated: {torch.cuda.memory_allocated()/1e9:.1f} GB")
print(f"Max allocated: {torch.cuda.max_memory_allocated()/1e9:.1f} GB")
```
