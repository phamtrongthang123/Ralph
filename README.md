# Ralph — AI-Assisted Research Project

This project uses an LLM agent (Claude) to drive a research/programming workflow on any machines.

## Structure

| File | Purpose |
|------|---------|
| `spec.md` | Final goal, deployment constraints, git workflow rules. **Human-only** — the agent never edits this file |
| `implementation_plan.md` | Step-by-step task list; the only planning doc the agent edits |
| `prompt.md` | Agent operating instructions (study plan → task loop) |
| `reflect.md` | Instructions for periodic reflection passes (cleans up the plan, not code) |
| `ralph.sh` | Main entry-point / launcher script |
| `example_slurm.sh` | Reference SLURM job template |
| `thang_note.md` | Human-written Vietnamese notes for review & QA |

## The control model

Ralph separates **what** from **how**:

- **`spec.md`** (human-only) — You define the goal, constraints, and success criteria. The agent reads this but never modifies it.
- **`implementation_plan.md`** (agent-managed) — The agent breaks down the spec into tasks, tracks progress, and records lessons learned.

This gives you full control over direction while letting the agent handle execution autonomously. You can safely run full Opus without worrying about the agent drifting from your intent — if it notices something wrong with the spec, it flags it in the plan for you to address.

**Remember: Ralph is an offloading tool, not set-and-forget.** Check in periodically, review the plan, and update the spec when needed.

## Quick start

1. **Write your spec** in `spec.md` (see "Writing a good spec" below).
2. **Seed the plan** in `implementation_plan.md` with initial checkboxes, or leave it for the agent to populate.
3. **Run Ralph:**
   ```bash
   bash ralph.sh
   ```

That's it. Ralph will:
- Retry automatically if you hit a rate limit (waits 3 hours then resumes).
- Run a reflection pass every N loops to combat context rot. Change the frequency by editing `REFLECT_EVERY` in `ralph.sh` (default: every 2 loops).
- Stop after 6 loops. Change `count -ge 6` in `ralph.sh` to adjust.

## Writing a good spec

Before filling in `spec.md`, ask yourself these questions:

**Goal clarity**
- What does "done" look like?
- If the agent finishes and I check back tomorrow, what do I expect to see working?
- What is explicitly out of scope?

**Input / output**
- What data or resources does the agent start with? (repos, datasets, pretrained models, APIs)
- What artifacts should it produce? (trained model, predictions file, a script that runs end-to-end, plots)

**Constraints**
- What hardware is available? (GPUs, SLURM partitions, memory limits)
- Are there time constraints? (job walltime, deadline)
- Are there dependencies that must not change? (pinned library versions, specific model checkpoints)

**What the agent should NOT do**
- Are there files or directories it must not touch?
- Should it avoid certain approaches? (e.g., no rewriting the data loader, no changing the model architecture)

## How it works

1. The agent reads `spec.md` (human-defined, read-only) and `implementation_plan.md`.
2. It picks the highest-leverage task, executes it, and iterates until the code runs end-to-end.
3. Every meaningful change is committed before moving to the next task.
4. Every N loops, a reflection pass cleans up the plan to prevent context rot.
5. The agent updates `implementation_plan.md` with progress. If it spots spec issues, it flags them in the plan for you.

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

A: Ralph retries on rate limits automatically and stops after a set number of loops. For other failures, check the commit history and the plan. You can also interrupt with `Ctrl+C`, update `spec.md` to correct direction, then re-run `bash ralph.sh` to resume. The agent will pick up from the current state of `implementation_plan.md`.

**Q: Can I see the current thought process?**

A: Yes — run `claude --resume`, which shows the history of all Claude sessions including the currently running `claude -p`. Pick the newest one with Ctrl-V and that is the current process.

## License
MIT