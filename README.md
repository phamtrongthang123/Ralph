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

## Quick start

1. **Describe your goal** in `spec.md` — chat with an LLM to flesh out the main ideas first.
2. **Define implementation steps** in `spec.md` using checkbox-style tasks (`- [ ] ...`).
3. **Run Ralph:**
   ```bash
   bash ralph.sh
   ```

That's it. Ralph will:
- Retry automatically if you hit a rate limit (waits 3 hours then resumes).
- Run a reflection pass every N loops to combat context rot. Change the frequency by editing `REFLECT_EVERY` in `ralph.sh` (default: every 2 loops).

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

## FAQ

**Q: When do I need to use this Ralph loop?**

A: When you are absolutely sure you don't care about the code generated — as long as it works, you are happy, and no one will ever complain why the code is so over-engineered. Very useful when you have a repo you want to try but don't have time to set up the environment or run a few inference scripts, or even training. Run Ralph until it's done and you can come back to a working codebase whenever you want.

**Q: What if I don't have time but I need good working code?**

A: You either accept the fact that the code will not be perfect, or hire an actual developer who can actually write good code.

**Q: Will Ralph write production-quality code?**

A: No. Expect working code, not clean code. It may be verbose, inconsistently structured, or over-engineered. That is the tradeoff.

**Q: Can I run Ralph interactively to review each step?**

A: No — Ralph is fully headless and autonomous. If you want to review each decision, hire a developer or use an interactive agent instead.

**Q: What happens if something goes wrong mid-run?**

A: Ralph retries on rate limits automatically. For other failures, check the commit history and `reflect.md`. You can also interrupt with `Ctrl+C`, manually edit `spec.md` or `implementation_plan.md` to correct the course, then re-run `bash ralph.sh` to resume.

**Q: Can I see the current thought process?**

A: Yes — run `claude --resume`, which shows the history of all Claude sessions including the currently running `claude -p`. Pick the newest one with Ctrl-V and that is the current process.

## License
MIT