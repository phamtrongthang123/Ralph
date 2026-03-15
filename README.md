# Ralph — AI-Assisted Research Project

This project uses an LLM agent (Claude) to drive a research/programming workflow on any machines.

## Structure

| File | Purpose |
|------|---------|
| `spec.md` | Final goal, deployment constraints, git workflow rules |
| `implementation_plan.md` | Step-by-step task list; updated by the agent after each task |
| `prompt.md` | Agent operating instructions (study plan → task loop) |
| `reflect.md` | Retrospective notes, lessons learned |
| `ralph.sh` | Main entry-point / launcher script |
| `example_slurm.sh` | Reference SLURM job template |
| `thang_note.md` | Human-written Vietnamese notes for review & QA |

## How it works

1. The agent reads `spec.md` and any implementation plan.
2. It picks the highest-leverage task, runs it on a GPU node via `sbatch`, and iterates until the code runs end-to-end.
3. Every meaningful change is committed before moving to the next task.

## Headless vs interactive

Ralph is **headless** — it runs autonomously without a UI. If you want an interactive version, consider [karpathy/autoresearch](https://github.com/karpathy/autoresearch).

## Human oversight

Because the code is largely LLM-generated it can be hard to audit at a glance.
`thang_note.md` exists as a dedicated human-readable log (in Vietnamese) where the human operator tracks progress and runs QA checks against the generated code.
See that file for the current review status.
