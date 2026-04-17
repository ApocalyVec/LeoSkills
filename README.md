# LeoSkills

Reusable Claude Code skills. Framework-agnostic guidance for ML development, presentations, and more.

## ML Skills

| Skill | When to Use | Description |
|-------|------------|-------------|
| `model-experiment` | Starting/iterating experiments | Full lifecycle: data inspection → pre-flight → launch → monitor → diagnose → fix → re-run |
| `monitor-training-runs` | During active training | Quick health checks, abnormal value reference, trend analysis reporting |
| `memory-optimization` | OOM errors, batch size tuning | GPU memory profiling, gradient checkpointing, mixed precision |

These three skills cross-reference each other. `model-experiment` is the orchestrator; `monitor-training-runs` and `memory-optimization` are quick-reference cards invoked during specific situations.

## Presentation Skills

| Skill | When to Use | Description |
|-------|------------|-------------|
| `presentations` | Building conference slides | python-pptx slide generation, academic conventions, HCI storytelling |
| `presentation-animations` | Animating slide content | Manim-based data figure animations, transitions |

## Usage

Copy individual skill folders to `~/.claude/skills/` or symlink them:

```bash
ln -s /path/to/LeoSkills/model-experiment ~/.claude/skills/model-experiment
```
