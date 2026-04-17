---
name: monitor-training-runs
description: "Use when monitoring active ML training runs — checks for crashed processes, abnormal loss values (NaN, inf, not decreasing), GPU memory drops, mode collapse, or gradient issues. Framework-agnostic (PyTorch, JAX). Use proactively during any session where training is running."
---

# Monitor Training Runs

## Quick Health Check

Run these in order. If any fail, investigate before continuing.

```bash
# 1. Process alive?
ps aux | grep "<training_script>" | grep -v grep | wc -l

# 2. GPU usage matches expected runs?
nvidia-smi --query-gpu=index,memory.used,utilization.gpu --format=csv,noheader

# 3. Recent log output (skip framework noise)
tail -5 <log_file> 2>/dev/null | grep -v "tensorflow\|FutureWarning\|UserWarning"

# 4. Latest metrics from TensorBoard
python -c "
from tensorboard.backend.event_processing.event_accumulator import EventAccumulator
import glob
for d in sorted(glob.glob('<log_dir>/*/'))[-3:]:
    ea = EventAccumulator(d, size_guidance={'scalars': 50}); ea.Reload()
    for t in [t for t in ea.Tags().get('scalars',[]) if 'loss' in t.lower()]:
        e = ea.Scalars(t)
        if e: print(f'{d.split(\"/\")[-2]} | {t}: {e[0].value:.4f}(s{e[0].step}) -> {e[-1].value:.4f}(s{e[-1].step})')
" 2>/dev/null
```

## Abnormal Value Reference

| Symptom | Likely Cause | Action |
|---------|-------------|--------|
| `loss = NaN/inf` | LR too high, bad batch, dtype overflow | Stop, check log, reduce LR or fix data |
| `loss = 0.0` from start | Trivial solution or empty batches | Check dataset size and content |
| Metric stuck at exact same value every epoch | Mode collapse — predicting majority class | See model-experiment Phase 3 diagnosis |
| GPU memory drops to ~0 | Process crashed | Check log for traceback, restart |
| No log updates > 2× epoch time | Hung or compiling | Check process status (`ps aux`) |
| Gradient norm spikes > 10× baseline | Precursor to loss explosion | Save checkpoint, watch next 10 steps |
| Throughput (steps/sec) drops > 20% | Data pipeline stall or memory pressure | Check disk I/O, DataLoader workers |

## What to Report

After each monitoring check, report:

1. **Alive**: yes/no + PID
2. **Epoch**: current / total
3. **Metrics**: latest train loss, val loss, primary metric (e.g., direction accuracy)
4. **Trend**: improving / plateaued / degrading
5. **GPU**: memory used / total, utilization %
6. **Anomalies**: any alerts from the table above

## Monitoring Schedule

- **First 5 minutes**: manual check — confirm startup, loss finite
- **Then**: every 15-30 minutes during active session
- **If unattended**: set up cron or `/loop 15m` for periodic checks

## Restart Protocol

**NEVER restart without diagnosing the crash:**

1. Read last 50 lines of log — find error class
2. Check if root cause is fixable (data bug? OOM? config error?)
3. Fix the root cause
4. Restart with **version increment** (v5 → v6)
5. Use `>>` not `>` for log redirection (preserve crash history)
6. Resume monitoring immediately after restart
