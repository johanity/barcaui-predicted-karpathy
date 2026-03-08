# Lab Notebook — Experiment Journal

*Every experiment gets a BEFORE (hypothesis + prediction) and AFTER (result + learning) entry.*

---

## Experiment 0: Baseline

**Date**: 2026-03-07
**Hypothesis**: N/A — establishing baseline
**Prediction**: N/A
**Result**: val_loss = 4.232615
**Details**: 158 steps, 14.2M params, 0.6M tokens, 120.5s training
**Learning**: Baseline established. 158 steps in 2 min with batch_size=8, seq_len=512. The model processes only 0.6M tokens total — very few. Key observation: at this training duration, we're heavily undertrained. The most impactful changes will likely be ones that either (a) increase tokens/sec throughput or (b) make each token count more via better optimization.

---

## Experiment 1: Higher learning rate (3e-4 → 1e-3)

**Date**: 2026-03-07
**Hypothesis**: With only 158 steps, the model is severely undertrained. A higher learning rate (1e-3 vs 3e-4) will push the model further down the loss landscape in the same number of steps. The cosine schedule will still decay it smoothly. The risk is instability, but with grad clipping at 1.0 and warmup, it should be fine.
**Prediction**: val_loss ≈ 3.8 (improvement of ~0.4, or ~10%)
**Reasoning**: 3x higher LR means roughly 3x larger parameter updates. With only 158 steps, we're far from any local minimum, so larger steps should help significantly. The warmup prevents early instability.
**Result**: val_loss = 4.126618 (156 steps)
**Prediction error**: Predicted 3.8, got 4.13. Off by 0.33. Overestimated the impact.
**Learning**: Higher LR helped (~0.1 improvement) but not as dramatically as expected. The model is learning-rate-sensitive but not as much as I assumed. Possible reasons: (1) cosine decay means the effective average LR isn't 3x higher, it decays; (2) with only 156 steps, the model architecture itself may be the bottleneck, not the optimization; (3) the warmup eats 50 of 156 steps — that's 32% of training! Warmup should be shorter. **Key insight: warmup_steps=50 is way too many for 156 total steps.** STATUS: KEEP

---

## Experiment 2: Reduce warmup (50 → 10 steps)

**Date**: 2026-03-07
**Hypothesis**: Warmup of 50 steps out of ~156 total means 32% of training is at sub-optimal LR. Reducing to 10 steps (6% of training) will let the model train at full LR for much longer, yielding better convergence.
**Prediction**: val_loss ≈ 3.95 (improvement of ~0.17). Being more conservative this time — learned from Exp 1 that I overestimate impact.
**Reasoning**: The model spends 40 extra steps at reduced LR with warmup=50. Those 40 steps at full LR should produce meaningful additional learning. But this is a smaller change than the LR itself, so expecting modest improvement.
**Result**: val_loss = 4.107551 (160 steps)
**Prediction error**: Predicted 3.95, got 4.11. Off by 0.16. Still overestimating, but closer than last time.
**Learning**: Reducing warmup helped modestly (4.127 → 4.108, delta=0.019). The effect is real but small — warmup wasn't the main bottleneck. With LR and warmup now tuned, the next bottleneck is likely either (a) model throughput (only 160 steps / 0.6M tokens in 2 min) or (b) model capacity. I should consider: can I get MORE steps by reducing model size? Or should I try a bigger LR still? STATUS: KEEP

---

## Experiment 3: Disable dropout (0.1 → 0.0)

**Date**: 2026-03-07
**Hypothesis**: With only 160 steps and 0.6M tokens, we are massively undertrained. Dropout is a regularization technique that prevents overfitting — but we're nowhere near overfitting. We're underfitting. Removing dropout should let the model learn faster from the limited data it sees.
**Prediction**: val_loss ≈ 4.00. Dropout removal should give a noticeable boost (~0.1) because every neuron is now active on every step, effectively increasing model capacity during training.
**Reasoning**: Dropout makes sense when you train for many epochs and risk overfitting. We see each token essentially once (0.6M tokens from 477M available = 0.1% of data). There's zero overfitting risk. Dropout is pure waste here.
**Result**: val_loss = 3.938537 (168 steps)
**Prediction error**: Predicted 4.00, got 3.94. Off by only 0.06! Best prediction yet.
**Learning**: HUGE win. Removing dropout gave delta=0.17, the biggest single improvement so far. Also got 8 more steps (168 vs 160) — dropout removal slightly speeds up computation. This confirms the "we're undertrained, not overtrained" theory. Also: my predictions are getting more accurate (errors: 0.33, 0.16, 0.06) — the theory is working! STATUS: KEEP

---

## Experiment 4: Smaller faster model (6 layers → 4 layers, 384 → 256 dim)

**Date**: 2026-03-07
**Hypothesis**: We're seeing only 168 steps in 2 min. A smaller model (4 layers, 256 dim, ~4M params) should run much faster, getting perhaps 2-3x more steps. More steps = more tokens seen = potentially better final loss despite less model capacity. This tests: is throughput or capacity the current bottleneck?
**Prediction**: val_loss ≈ 4.1. I think the model will be too small to beat 3.94 — capacity matters. But this experiment is INFORMATIVE: if it wins, throughput is the bottleneck. If it loses, capacity is. Either answer is valuable.
**Reasoning**: There's a fundamental tradeoff — bigger models learn more per step but take fewer steps. At our extreme (168 steps), we might be on the wrong side. But 256 dim with 4 heads is quite small. I expect capacity to win here, making this a "diagnostic" experiment.
**Result**: val_loss = 3.384197 (275 steps, 4.2M params)
**Prediction error**: Predicted 4.1, got 3.38. Off by 0.72. MASSIVELY wrong — in the good direction!
**Learning**: THIS IS THE BIGGEST FINDING SO FAR. The smaller model absolutely crushed the larger one (3.38 vs 3.94). 275 steps vs 168 = 64% more steps. This conclusively proves: **at 2 minutes of MPS training, throughput dominates capacity.** Seeing more tokens matters far more than having more parameters. The 14.2M model was dramatically oversized for this time budget. The 4.2M model sees 1.1M tokens vs 0.7M tokens. My theory was wrong — I need to completely revise. STATUS: KEEP

---

## Experiment 5: Even smaller model (3 layers, 192 dim)

**Date**: 2026-03-07
**Hypothesis**: If 4 layers/256 dim beat 6/384 because of throughput, maybe 3 layers/192 dim will be even faster and beat it again. But there must be a floor — at some point the model is too stupid to learn even with more data. This experiment tests where that floor is.
**Prediction**: val_loss ≈ 3.5. I think this will be WORSE than 3.38. Reasoning: 256 dim is already quite narrow. Going to 192 severely limits the representational capacity. The throughput gain (~30% more steps?) won't compensate. The sweet spot is probably around where we are.
**Reasoning**: We're testing the other side of the throughput-capacity tradeoff curve. The optimal point is somewhere between "too big" (14M) and "too small" (??). This helps locate it.
**Result**: val_loss = 3.274700 (344 steps, 1.8M params)
**Prediction error**: Predicted 3.5, got 3.27. Off by 0.23 — wrong direction! I thought it'd be worse but it was better.
**Learning**: I keep underestimating how much throughput matters. 344 steps vs 275 = 25% more. The 1.8M model is fast enough to still learn well. Score progression: 4.23 → 4.13 → 4.11 → 3.94 → 3.38 → 3.27. We've improved 22% from baseline! Open question: is there still more to gain by going even smaller, or have we found the sweet spot? Let me try one more reduction to find the floor. STATUS: KEEP

---

## Experiment 6: Tiny model (2 layers, 128 dim)

**Date**: 2026-03-07
**Hypothesis**: Continuing to shrink the model. 2 layers with 128 dim is very small (~0.5M params?). This SHOULD finally hit the capacity floor — a 2-layer transformer can only represent limited functions.
**Prediction**: val_loss ≈ 3.4. I think this will be WORSE than 3.27 — we'll finally see the capacity floor. 2 layers is probably too shallow for language modeling.
**Reasoning**: Each previous reduction helped because of throughput. But at 2 layers, the model can only do 2 rounds of attention+FFN. Language requires composing features hierarchically. 2 layers may not be enough for that, even with many steps.
**Result**: val_loss = 3.363086 (434 steps, 0.5M params)
**Prediction error**: Predicted 3.4, got 3.36. Off by only 0.04! Best prediction yet.
**Learning**: FOUND THE FLOOR. Despite 434 steps (26% more than exp 5), val_loss regressed from 3.27 to 3.36. The 0.5M model is too small — it can't represent the language patterns even with more data. **The sweet spot is around 1.8M-4.2M params (3-4 layers, 192-256 dim).** The optimal point for 2 min on MPS is experiment 5's config: 3 layers, 192 dim. Now I should go back to that config and try to improve it through OTHER means (LR, batch size, architecture tweaks). STATUS: DISCARD — revert train.py

---

## Experiment 7: Double LR (1e-3 → 2e-3)

**Hypothesis**: Brute-force found LR=0.002 helps. With our optimized 3-layer model, higher LR should push further in 344 steps.
**Prediction**: val_loss ≈ 3.15
**Result**: val_loss = 3.244042 (344 steps). Error: 0.09. KEEP.
**Learning**: LR=2e-3 helps. Now closer to brute-force best (3.216). LR is still on the improving side.

---

## Experiment 8: Higher LR (2e-3 → 3e-3)

**Hypothesis**: LR=2e-3 improved over 1e-3. Push further to 3e-3. There's a risk of instability but let's find the peak.
**Prediction**: val_loss ≈ 3.20 (small further gain, approaching diminishing returns)
