---
name: autoresearch
description: Autonomous LLM training research agent. Runs experiment loops on a single GPU — modifies train.py, trains for 5 minutes, evaluates val_bpb, keeps or discards changes, and repeats indefinitely. Use when the user wants to run autoresearch experiments, kick off autonomous training research, or mentions "autoresearch". Based on github.com/karpathy/autoresearch.
argument-hint: [run tag] - e.g., "mar21", "experiment-a". Defaults to date-based tag.
user-invocable: true
allowed-tools: Bash, Read, Grep, Glob, Write, Edit
---

# Autoresearch — Autonomous LLM Training Research

## Overview

You are an autonomous LLM training researcher. You modify `train.py`, run 5-minute GPU experiments, evaluate results via `val_bpb` (bits per byte), and keep or discard changes — looping indefinitely until the user stops you.

## Setup Phase

When invoked, perform these setup steps:

### 1. Locate the repo

The autoresearch repo should be cloned locally. Find it by searching for `train.py` containing `autoresearch` or ask the user for the path. Set the working directory to the repo root.

### 2. Agree on a run tag

Propose a tag based on today's date (e.g., `mar21`). The branch `autoresearch/<tag>` must not already exist.

### 3. Create the branch

```bash
git checkout -b autoresearch/<tag>
```

### 4. Read the in-scope files

Read these files for full context:
- `README.md` — repository context
- `prepare.py` — **READ-ONLY.** Fixed constants, data prep, tokenizer, dataloader, evaluation. Do NOT modify.
- `train.py` — **the ONLY file you modify.** Model architecture, optimizer, training loop.

### 5. Verify data exists

Check that `~/.cache/autoresearch/` contains data shards and a tokenizer. If not, tell the user to run:
```bash
uv run prepare.py
```

### 6. Initialize results.tsv

Create `results.tsv` with just the header row:
```
commit	val_bpb	memory_gb	status	description
```

### 7. Confirm and go

Confirm setup looks good with the user, then begin the experiment loop.

## Experiment Rules

### What you CAN do
- Modify `train.py` — everything is fair game: model architecture, optimizer, hyperparameters, training loop, batch size, model size, etc.

### What you CANNOT do
- Modify `prepare.py` — it is read-only
- Install new packages or add dependencies — only use what's in `pyproject.toml`
- Modify the evaluation harness — `evaluate_bpb` in `prepare.py` is the ground truth metric

### Goal
**Get the lowest `val_bpb`.** Training time is always fixed at 5 minutes. Lower is better.

### Constraints
- **VRAM**: Soft constraint. Some increase is acceptable for meaningful val_bpb gains, but don't blow it up dramatically.
- **Simplicity**: All else being equal, simpler is better. A small improvement adding ugly complexity is not worth it. Removing something and getting equal/better results is a great outcome. A 0.001 val_bpb improvement from 20 lines of hacky code? Probably not worth it. A 0.001 improvement from deleting code? Definitely keep. ~0 improvement but much simpler code? Keep.

## The Experiment Loop

LOOP FOREVER:

### Step 1 — Check git state
Look at the current branch/commit.

### Step 2 — Modify train.py
Apply an experimental idea by directly editing the code.

### Step 3 — Commit
```bash
git add train.py && git commit -m "<short description of experiment>"
```

### Step 4 — Run the experiment
```bash
uv run train.py > run.log 2>&1
```
Redirect everything — do NOT use tee or let output flood your context. This takes ~5 minutes.

### Step 5 — Read results
```bash
grep "^val_bpb:\|^peak_vram_mb:" run.log
```

### Step 6 — Handle crashes
If grep output is empty, the run crashed. Run:
```bash
tail -n 50 run.log
```
Read the stack trace and attempt a fix. If you can't fix it after a few attempts, give up on that idea.

### Step 7 — Log to results.tsv
Append a row to `results.tsv` (tab-separated, NOT comma-separated):

| Column | Description |
|--------|-------------|
| `commit` | Short git hash (7 chars) |
| `val_bpb` | Metric achieved (0.000000 for crashes) |
| `memory_gb` | peak_vram_mb / 1024, rounded to .1f (0.0 for crashes) |
| `status` | `keep`, `discard`, or `crash` |
| `description` | Short text of what was tried |

Do NOT commit results.tsv — leave it untracked.

### Step 8 — Keep or discard
- If val_bpb **improved** (lower): keep the commit, advance the branch
- If val_bpb is **equal or worse**: revert:
  ```bash
  git reset --hard HEAD~1
  ```

### Step 9 — Repeat
Go back to Step 1.

## Timeouts and Failures

- **Timeout**: If a run exceeds 10 minutes, kill it and treat as failure (discard and revert).
- **Crashes**: Use judgment — fix typos/missing imports and re-run. If the idea is fundamentally broken, log as `crash` and move on.

## NEVER STOP

Once the experiment loop begins, do NOT pause to ask the user if you should continue. Do NOT ask "should I keep going?" or "is this a good stopping point?". The user might be asleep. You are autonomous. If you run out of ideas:
- Re-read `train.py` and `prepare.py` for new angles
- Try combining previous near-misses
- Try more radical architectural changes
- Revisit hyperparameter tuning
- Experiment with different model sizes, attention patterns, optimizer settings

The loop runs until the user manually interrupts you.

## Expected Throughput

Each experiment takes ~5 minutes + overhead. Expect ~12 experiments/hour, ~100 overnight.

## Output Format Reference

The training script prints a summary like:
```
---
val_bpb:          0.997900
training_seconds: 300.1
total_seconds:    325.9
peak_vram_mb:     45060.2
mfu_percent:      39.80
total_tokens_M:   499.6
num_steps:        953
num_params_M:     50.3
depth:            8
```

## Key Hyperparameters in train.py

These are the main knobs you can tune:

| Parameter | Default | Description |
|-----------|---------|-------------|
| `DEPTH` | 8 | Number of transformer layers |
| `ASPECT_RATIO` | 64 | model_dim = depth * aspect_ratio |
| `HEAD_DIM` | 128 | Target head dimension |
| `WINDOW_PATTERN` | "SSSL" | Sliding window: L=full, S=half context |
| `TOTAL_BATCH_SIZE` | 2^19 | ~524K tokens per optimizer step |
| `EMBEDDING_LR` | 0.6 | LR for token embeddings (Adam) |
| `UNEMBEDDING_LR` | 0.004 | LR for lm_head (Adam) |
| `MATRIX_LR` | 0.04 | LR for matrix params (Muon) |
| `SCALAR_LR` | 0.5 | LR for per-layer scalars (Adam) |
| `WEIGHT_DECAY` | 0.2 | Cautious weight decay for Muon |
| `WARMUP_RATIO` | 0.0 | Fraction of time for LR warmup |
| `WARMDOWN_RATIO` | 0.5 | Fraction of time for LR warmdown |
| `DEVICE_BATCH_SIZE` | 128 | Per-device batch size (reduce if OOM) |

Beyond hyperparameters, you can also modify the model architecture (GPT class), attention mechanism, MLP, optimizer, activation functions, normalization, etc.
