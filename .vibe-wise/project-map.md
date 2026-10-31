# Project Map

## Purpose
A Kazakh retrieval system: Kazakh query -> ranked relevant passages. A possible later generation layer makes it full RAG. Must end as a real deployed system.

## Requirements
- Real deployed system; shows ML/AI, math, MLOps.
- v1 = retriever only (learner decided). Deploy only after a measured improvement over baselines.
- Comparisons use the same corpus, queries and metric.

## Components
None implemented. Repo: README.md (learner-written status/plan), docs/research-journal.md (design journal, AI-maintained at the learner's request), empty src/.

## Main Flow
Proposed: KazQAD corpus -> embedding model (base model ?) -> index (?) -> query embedding -> top-k passages -> evaluation (metric ?). Generation layer: future.

## Data and Trust Boundaries
Data: issai/kazqad-retrieval (CC BY-SA 4.0). Candidate models in the journal (Entries 6, 8, 11). The shyngys879 model's license is "other" with no terms.

## Build and Deployment
None yet. No .git directory in kazakh_rag (as of 2026-10-31); the old remote was https://github.com/nurasik14/trilingual_rag.git.
Deployment target: open (Docker Space needs PRO; ZeroGPU is Gradio-only; free accounts can host 2 ZeroGPU Gradio Spaces).
Hardware: RTX 4050 Laptop 6141 MiB VRAM, 15 GiB RAM.

## Unknowns
Base model; training memory settings; index type/size; metric and baselines; split usage; replication protocol; deployment target; stack details.
