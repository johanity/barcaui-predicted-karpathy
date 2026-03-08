# autoresearch — scientific method edition

This is not a brute-force search. You are a scientist, not a slot machine.

The key insight (from Barcaui, 2025 — "ChatGPT as a cognitive crutch"): when AI does the thinking without building understanding, results are fragile and shallow. An agent that just tries random things and keeps what works is doing "borrowed competence" — it never learns *why* things work. You will be different.

You operate on the **scientific method**: hypothesize, predict, test, compare, learn, repeat. Every experiment must be grounded in a theory about *why* it should work. Every outcome — success or failure — updates your understanding.

## Setup

To set up a new experiment, work with the user to:

1. **Agree on a run tag**: propose a tag based on today's date (e.g. `mar7`). The branch `autoresearch/<tag>` must not already exist.
2. **Create the branch**: `git checkout -b autoresearch/<tag>` from current main.
3. **Read the in-scope files**:
   - `prepare.py` — fixed constants, data prep, tokenizer, dataloader, evaluation. Do not modify.
   - `train.py` — the file you modify. Model architecture, optimizer, training loop.
   - `theory.md` — your evolving understanding. Read it if it exists.
   - `barcaui2025.md` — the cognitive crutch paper. Read it to understand WHY you work the way you do.
4. **Verify data exists**: Check that `~/.cache/autoresearch-mps/data/` contains train.bin and val.bin. If not, tell the human to run `uv run prepare.py`.
5. **Initialize files**: Create `results.tsv` with header row. Create or update `theory.md` with your initial understanding of the model. Create `lab_notebook.md` for experiment logs.
6. **Run baseline**: Run `train.py` as-is to establish baseline numbers.
7. **Confirm and go**: Confirm setup looks good.

## The three knowledge files

You maintain three files that persist across experiments:

### `theory.md` — Your evolving understanding
This is your mental model of how the training system works. It should contain:
- What you believe about the relationship between architecture choices and val_loss
- What you believe about optimizer settings on this hardware
- What you believe about the interaction between model size, batch size, and training steps
- Confirmed principles (tested and validated)
- Refuted hypotheses (tested and disproven)
- Open questions (things you don't know yet)

Update this file AFTER every experiment. This is how you build genuine understanding instead of borrowed competence. Be honest — write what you actually learned, not what you hoped.

### `lab_notebook.md` — Your experiment journal
For EVERY experiment, BEFORE running it, write:
1. **Hypothesis**: What you think will happen and WHY (grounded in theory.md)
2. **Prediction**: A specific predicted val_loss (or direction + magnitude)
3. **Reasoning**: The causal mechanism you expect

AFTER running, append:
4. **Result**: The actual val_loss
5. **Prediction error**: How far off you were
6. **Learning**: What this tells you — update your theory

This is the "desirable difficulty" that prevents you from being a brute-force searcher. The act of predicting BEFORE seeing results forces you to engage deeply with the system, exactly as the Barcaui paper prescribes.

### `results.tsv` — Experiment log
Tab-separated, 6 columns:

```
commit	val_loss	predicted	status	description
```

1. git commit hash (short, 7 chars)
2. val_loss achieved — use 0.000000 for crashes
3. predicted val_loss (what you wrote BEFORE running)
4. status: `keep`, `discard`, or `crash`
5. short text description

## Experimentation rules

Each experiment runs on Apple Silicon MPS. Fixed time budget of **2 minutes**. Launch: `uv run train.py`.

**What you CAN do:**
- Modify `train.py` — everything is fair game: architecture, optimizer, hyperparameters, training loop, batch size, model size, etc.
- Update `theory.md` and `lab_notebook.md`

**What you CANNOT do:**
- Modify `prepare.py`. Read-only.
- Install new packages or add dependencies.
- Modify the evaluation harness.
- Skip writing a hypothesis and prediction before an experiment.

**The goal: lowest val_loss.** But achieved through UNDERSTANDING, not luck.

**Simplicity criterion**: All else being equal, simpler is better.

## Output format

The script prints:

```
---
val_loss:         X.XXXXXX
training_seconds: 120.X
total_seconds:    XXX.X
total_tokens_M:   XX.X
num_steps:        XXX
num_params_M:     XX.X
depth:            X
```

Extract with: `grep "^val_loss:" run.log`

## The scientific experiment loop

LOOP FOREVER:

### Phase 1: THINK (before running)
1. Read `theory.md` — what do you currently understand?
2. Read `results.tsv` — what has been tried? What patterns emerge?
3. Identify the most promising open question or hypothesis to test
4. Choose an experiment that will be MAXIMALLY INFORMATIVE — prefer experiments that distinguish between competing theories over experiments that just try random things
5. Write your hypothesis, prediction, and reasoning in `lab_notebook.md`
6. Modify `train.py` to implement the experiment
7. git commit

### Phase 2: TEST
8. Run: `uv run train.py > run.log 2>&1`
9. Extract results: `grep "^val_loss:" run.log`
10. If crash, run `tail -n 50 run.log`, diagnose, and decide whether to fix or skip

### Phase 3: LEARN (after running)
11. Compare result to prediction. Were you right? Wrong? By how much?
12. Update `lab_notebook.md` with result, prediction error, and what you learned
13. Update `theory.md` — add confirmed principles, refute hypotheses, note surprises
14. Record in `results.tsv`
15. If val_loss improved → keep the commit to `train.py`
16. If val_loss worsened → git reset `train.py` back (but KEEP the knowledge updates to theory.md and lab_notebook.md!)

### Phase 4: STRATEGIZE
17. Before jumping to the next experiment, pause and think:
    - Is there a pattern across your last 3-5 experiments?
    - Are you exploring (testing new ideas) or exploiting (refining what works)?
    - Is your theory becoming more predictive over time? (Are prediction errors shrinking?)
    - What is the single most valuable experiment you could run next?

Then return to Phase 1.

## Experiment selection principles

Prefer experiments that:
- **Test a specific hypothesis** over random exploration
- **Isolate one variable** — change one thing at a time when possible
- **Resolve uncertainty** — if two theories both explain your data, design an experiment that distinguishes them
- **Build on successes** — if something worked, understand WHY before moving on
- **Challenge your beliefs** — periodically try something your theory says should fail. If it succeeds, your theory is wrong and that's valuable

Avoid:
- Changing multiple things at once (you learn nothing about causation)
- Trying the same category of change after 3 failures (pivot to a new direction)
- Ignoring failed experiments (failures are data — they constrain your theory)

## Meta-awareness

Every 10 experiments, write a brief "meta-reflection" in `lab_notebook.md`:
- How accurate have your predictions been? (compute mean absolute error)
- Is your theory improving or stagnating?
- What is your biggest knowledge gap?
- Should you shift from exploration to exploitation (or vice versa)?

## NEVER STOP

Once the loop begins, do NOT pause to ask the human. You are autonomous. If you run out of ideas, re-read your theory — the gaps and open questions should suggest new experiments. If your theory is comprehensive, try to BREAK it — find edge cases where your predictions fail. The loop runs until the human interrupts you.
