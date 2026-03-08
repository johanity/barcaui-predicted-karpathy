# Barcaui Predicted Karpathy

A study about students getting dumber from ChatGPT predicted exactly how an AI research agent would fail. One year before it was built.

The fix worked on AI too.

---

## The Story

**2025:** Barcaui runs an experiment on 120 college students. Half study with ChatGPT. Half study alone. 45 days later, the ChatGPT group scores **19% worse**. The reason? When AI does the thinking for you, you skip the struggle that actually builds knowledge. Barcaui calls this **"borrowed competence."** You feel smart, but you haven't learned anything.

**2026:** Karpathy releases [autoresearch](https://github.com/karpathy/autoresearch). An AI agent that autonomously improves ML training code. The loop is simple: try something, keep it if it works, discard it if it doesn't. No memory. No theory. No understanding of *why* things work or fail.

**The connection nobody made:** Karpathy's agent has the exact same disease as Barcaui's students. It gets results without building understanding. It repeats mistakes because it has no memory of what it already tried. It plateaus early because it never learned what actually matters.

**The fix:** Apply Barcaui's prescription to Karpathy's agent. Force it to hypothesize before every experiment, predict the outcome with a specific number, and maintain a written theory that persists even when experiments fail. The same "desirable difficulties" that help students learn.

**What happened:**

| | Karpathy Style Agent | Fixed Agent |
|---|---|---|
| Best result | 3.216 | **3.082** |
| Improvement from baseline | 23.5% | **27.2%** |
| Stopped improving at | Experiment 7 | **Still going at experiment 20** |
| Experiments wasted | 13 out of 20 | **2 out of 20** |
| Repeated proven bad ideas | 6+ times | **Never** |
| Knowledge produced | Nothing | **10 principles, 5 refuted hypotheses** |

The fix costs nothing. Thinking before acting adds zero compute. The training run takes 2 minutes. The hypothesis takes 5 seconds.

---

## The Key Insight

> **The same cognitive shortcut that makes students worse at learning makes AI agents worse at research.**

Barcaui showed that when students skip the struggle, their knowledge is shallow and decays fast.

This study showed that when AI agents skip the struggle, their research plateaus and they waste their budget repeating failures.

"Borrowed competence" is not a human bug. It is a universal property of any system that optimizes for outcomes without building understanding.

---

## The Experiment

Two agents ran on the same task: training a small GPT on TinyStories using an Apple M4 with a 2 minute time budget per run.

**Agent A (Karpathy Style):** Random mutation, keep if better, discard if worse. No memory of past experiments.

**Agent B (Scientific Method Loop):** Before each experiment, must write a hypothesis and predict the exact val_loss. After each experiment, must explain what it learned and update its theory. Failed experiments are reverted in code but **the knowledge stays forever**.

Both got 20 experiments. Same hardware. Same starting code. Same search space.

Agent A plateaued at experiment 7 and did nothing useful for the remaining 13 experiments.

Agent B was still finding improvements at experiment 18 and discovered entire categories of optimization (weight decay, optimizer betas) that Agent A never even tried.

---

## The Discovery That Changed Everything

At experiment 4, the agent predicted that a smaller model would perform **worse** (prediction: 4.1). Instead, it performed **dramatically better** (actual: 3.38).

This was the biggest prediction error of the entire run. And it was the single most valuable experiment.

Why? Because the agent was forced to explain *why* it was wrong. It realized that at 2 minute training budgets, **throughput matters more than model size**. A 1.8M parameter model that processes more data beats a 14.2M parameter model that barely trains.

In the Karpathy style loop, this experiment would just be "keep." The insight about throughput vs capacity would never exist. The agent would stumble forward without ever understanding why smaller worked better.

**Forced prediction turns every failure into a lesson. Without it, failures are just failures.**

---

## Repo Contents

| File | Description |
|---|---|
| [paper.pdf](paper.pdf) | Full research paper |
| [train.py](train.py) | Final optimized training script (1.8M param GPT, val_loss 3.082) |
| [prepare.py](prepare.py) | Data preparation and evaluation |
| [program.md](program.md) | The Scientific Method Loop protocol |
| [theory.md](theory.md) | The agent's final theory: 10 confirmed principles, 5 refuted hypotheses |
| [lab_notebook.md](lab_notebook.md) | Full experiment journal with predictions, results, and learnings |
| [results.tsv](results.tsv) | All 20 experiments from the scientific method agent |
| [bruteforce_results.tsv](bruteforce_results.tsv) | All 20 experiments from the Karpathy style agent |

---

## Run It Yourself

```bash
git clone https://github.com/johanity/barcaui-predicted-karpathy.git
cd barcaui-predicted-karpathy
python -m venv .venv && source .venv/bin/activate
pip install torch tiktoken requests datasets

# Prepare data (downloads TinyStories)
python prepare.py

# Run training (2 minutes on Apple Silicon)
python train.py
```

---

## The Punchline

Barcaui told us in 2025 that skipping the struggle makes you worse at learning.

Nobody realized he was talking about AI too.

---

## References

Barcaui, A. (2025). ChatGPT as a cognitive crutch: Evidence from a randomized controlled trial on knowledge retention. *Social Sciences & Humanities Open*, 12, 102287.

Karpathy, A. (2026). autoresearch. [github.com/karpathy/autoresearch](https://github.com/karpathy/autoresearch)

Bjork, E. L., & Bjork, R. A. (2011). Making things hard on yourself, but in a good way: Creating desirable difficulties to enhance learning.

---

## Citation

```
@article{bonilla2026borrowed,
  title={Barcaui Predicted Karpathy},
  author={Bonilla, Johan David},
  year={2026},
  note={github.com/johanity/barcaui-predicted-karpathy}
}
```
