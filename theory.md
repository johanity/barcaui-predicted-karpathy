# Theory of Training — Final State After 20 Experiments

## Confirmed Principles

1. **Throughput > Capacity at short time budgets on MPS.** Smaller models that run more steps beat larger models. (Exps 4-6)
2. **Optimal model size: 1.8M params (3 layers, 192 dim).** Below this (0.5M) = capacity floor. Above (4.2M+) = throughput starved. (Exps 4-6)
3. **Dropout is pure waste when undertrained.** We see <0.2% of data per run. Removing it gave delta=0.17 and +5% speed. (Exp 3)
4. **LR=2e-3 is optimal for this config.** 1e-3 too slow, 3e-3 overshoots. (Exps 1,7,8)
5. **Warmup=5 is optimal.** Warmup=0 causes instability at LR=2e-3. Warmup=50 wastes 30%+ of training. (Exps 2,9,10)
6. **Batch_size=4 is optimal.** BS=2 too noisy (gradients), BS=8 too few steps, BS=16 way too few. (Exps 12-14)
7. **Weight_decay=0.01 beats 0.1 and 0.0.** Some regularization helps but 0.1 is too aggressive for short training. (Exps 16-17)
8. **Betas=(0.9, 0.99) is optimal.** Default (0.9,0.95) is too conservative for beta2. (0.9,0.999) is too slow to adapt. (Exps 18-19)
9. **MLP ratio 4x is correct for SwiGLU at this scale.** Reducing to 2.67x lost too much capacity. (Exp 15)
10. **Wider models (256 dim) don't compensate for throughput loss** even with optimized LR/warmup. (Exp 11)

## Refuted Hypotheses

- "Capacity matters more than throughput" — WRONG. At 2-min budget, throughput dominates. (Exp 4)
- "LR=3e-3 should improve over 2e-3" — WRONG. Overshoots. (Exp 8)
- "No warmup is fine with grad clipping" — WRONG. Causes instability at LR=2e-3. (Exp 10)
- "Batch_size=2 = more steps = better" — WRONG. Gradient noise dominates. (Exp 14)
- "Lower weight decay always better when undertrained" — WRONG. Some WD helps. (Exp 17)

## Meta-Observations

- Prediction errors over time: 0.33, 0.16, 0.06, 0.72, 0.23, 0.04, 0.09, 0.50, 0.05, 0.55, 0.40, 0.33, 0.10, 0.07, 0.11, 0.05, 0.07, 0.03, 0.14, 0.50
- Mean prediction error (first 10): 0.27
- Mean prediction error (last 10): 0.18
- Theory improved predictive accuracy by ~33% over the run

## Final Best Config
```
N_LAYER=3, N_HEAD=3, N_EMBD=192, DROPOUT=0.0
LR=2e-3, WD=0.01, BETAS=(0.9,0.99)
BATCH_SIZE=4, WARMUP=5
val_loss = 3.082 (27.2% improvement from baseline 4.233)
```
