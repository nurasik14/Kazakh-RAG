# Learning Progress

Recreated on 2026-10-31 from session evidence (the original was lost when the folder was renamed). The full narrative with verbatim queries is in docs/research-journal.md.

## Review queue
Rules (shown to the learner):
- Add a concept only after the learner has shown they understand it (their own correct reasoning).
- Intervals: 1 day -> 3 -> 7 -> 14 -> 30 days. Correct: move to the next interval. Partly right: repeat the same interval. Wrong: hint, reset to 1 day, mark it in Needs reinforcement.
- At most 1-2 per session, at the start or at a natural pause; ask the learner to apply the concept, not recite it.

| Concept | Learned | Last reviewed | Interval | Next due | History |
|---|---|---|---|---|---|
| Mixed-precision Adam model-state memory (6+8+4 = 18 B/param) | 2026-10-31 | - | 1 day | 2026-11-01 | derived from the HF anatomy doc |

## Problem framing and scope
- Learner proposed a "Kazakh Google" with hybrid retrieval, then narrowed it to a Kazakh retriever compared with others on the same data.
- Decided: v1 is a retriever only; generation maybe later; deploy after a measured improvement.
- Tentative: Kazakh-only v1 (folder renamed kazakh_rag).
- Skipped reasoning: what to compare against / which metric; the stage-by-stage language analysis.

## Data and evaluation
- Dataset: issai/kazqad-retrieval (train = MT NQ; val/test = UNT). Split usage, the three dataset sizes, and the metric are still open (readings given).
- Replication target: shyngys879/kazakh-e5-rag-embedding. Five protocol questions are open (journal Entry 5).

## Licenses and deployment
- Corrected: CC BY-SA is more restrictive, not less.
- Corrected: Docker on HF needs PRO; ZeroGPU is Gradio-only. The deployment target is open.

## GPU memory
- Corrected: VRAM is fixed (6141 MiB measured).
- Demonstrated: 18 B/param for model states.
- Sent back: the spike estimate scaled by params; activations depend on batch/seq/depth/hidden. GiB vs GB. Margin too thin.

## Base model
- Qwen3 dropped (size class). Granite chosen first "because CC BY-SA" (corrected), then "because lightweight 0.5 GB" (sent back: same 278M params, only the F16 vs F32 file differs).

## Pending decision
Decision: Base model choice and training memory, second attempt
Stage: awaiting reasoning
Asked: (1) choose a base model among the four 278M candidates with a reason that holds when training memory is identical; (2) redo the memory estimate with activations (which settings drive it; values that fit in 6 GiB), then measure with torch.cuda.max_memory_allocated.
Also open: the tokenizer experiment and index-size calculation (readings in journal Entry 13); split usage; metric/baselines; replication protocol; deployment target.
