# FINAL GOAL
Final goal here

## Git Workflow
- **Commit after every meaningful change** (new script, code modification, config update). Don't batch multiple unrelated changes into one commit.
- Write concise commit messages describing what changed and why.
- Commit before starting a new task so work is never lost if the agent crashes.
- Do NOT commit large generated files (checkpoints, predictions JSONs, plots). Only commit code, configs, and documentation.

## Deployment Constraints
- Agent on login node, GPU work via `sbatch`
- **SLURM partition**: `condo` with `--constraint=aimrc&4a100` or `--constraint=csce&4a100`
  - `condo` with `--constraint=aimrc&4a100` — nodes c2003, c2004 (4×A100)
  - `condo` with `--constraint=csce&4a100` — nodes c2101, c2102, c2103 (4×A100)
  - `qgpu72` — nodes c1901-c1904 (4×A100, public, ExclusiveUser)
  - `agpu72` — nodes c1907-c2109 (1×A100, public, for evals)
  - **ALWAYS exclude**: `--exclude=c2001,c2002,c2105` — c2001/c2002 are faulty, c2105 is harris (no access)
  - If using bare `--constraint="4a100"` on condo, the `--exclude` list above is **mandatory**
  - **Before submitting**: check which GPU nodes are idle:
    ```bash
    sinfo -p agpu72,qgpu72,condo -N --format="%N %P %G %f %T" | grep -v "0gpu"
    ```
  - **Never wait on pending jobs.** If `squeue --me` shows a job stuck with `(Resources)` or `(Priority)` for more than a few minutes, cancel it and resubmit to a partition that has idle nodes. The whole point of listing multiple partitions is so you always have somewhere to run — don't block on one partition when another is free.
- Max 72 hours per job
- Always set `export TMPDIR="$HOME/.tmp" && mkdir -p "$TMPDIR"`