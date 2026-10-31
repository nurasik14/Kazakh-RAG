# Learner Profile

Learning mode: active
Onboarding: complete

## Project
Situation: New. Folder renamed trilingual_rag -> kazakh_rag on 2026-10-31. .git and the original .vibe-wise were missing after the rename; these notes were recreated from session evidence on 2026-10-31.
Building: A Kazakh retrieval system (v1 retriever only, possible later RAG), to end as a real deployed system
Codebase familiarity: Not applicable (new project)
Learning scope: Whole system (implied by the learner's requirement to understand everything)

## Experience
Overall programming: Intermediate (strongest in Python)
Stack familiarity: Self-described "complete noob" in ML retrieval (tokenizers, embeddings, vector indexes); needs more grounding

## Goals
Primary: A deployed Kazakh retrieval/RAG system that shows ML/AI, math and MLOps understanding
Capability goal: Not specified

## Preferences
Checkpoint frequency: Frequent
Question style: Open-ended
Implementation style: More hands-on (learner writes most code, AI reviews)
Teaching style (learner requirement): learner must understand every design decision, every math concept and every other concept. When the learner doesn't know something, AI does not give the answer. AI suggests readings, the learner works out the solution, and if they're wrong AI sends it back or gives a small hint (escalating gradually).
Active recall (learner requirement): quiz demonstrated concepts lightly at spaced intervals; a wrong answer brings it back sooner (see progress.md '## Review queue').
Journal (learner requirement): after every exchange, update docs/research-journal.md. Each entry quotes the learner's queries verbatim (also listed in the Query log), with the learner's reasoning, evidence, corrections and outcome.
Agreed exceptions: plain facts that can't be deduced (syntax, flags, versions, prices) are stated with sources; bugs are flagged with where to look, not the fix, except security/data-loss risks.

## Strong Concepts
- Training memory for model states: derived 18 bytes/param (mixed-precision Adam) from the HF anatomy doc and applied it to 278M params.

## Developing Concepts
- GPU memory: VRAM is fixed (corrected). Activation memory omitted; estimated it by scaling parameters (sent back).
- Model size vs file size: confused the stored dtype (F16 file) with model size.
- Model licenses: treated CC BY-SA as an advantage; share-alike vs permissive was explained.
- Comparison fairness: reasons about size class; same benchmark/protocol not yet articulated.

## Revisit
- Units: GB vs GiB.
