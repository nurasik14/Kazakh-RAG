# Kazakh RAG

![status: design phase](https://img.shields.io/badge/status-design%20phase-orange)
![language: Kazakh](https://img.shields.io/badge/language-Kazakh%20(kk)-blue)

> **Work in progress.** This project is still in the design phase. There is no
> code, no trained model and no results yet. `src/` is empty. Everything below
> comes from the design log in [`docs/research-journal.md`](docs/research-journal.md).

A retrieval system for the Kazakh language: a user asks a question in Kazakh
and gets back the most relevant passages from a Kazakh corpus. The long-term
goal is a full retrieval-augmented generation (RAG) pipeline. The first
milestone is a **retriever** that measurably beats published Kazakh baselines
on the same data.

The project started as "Trilingual RAG" (Kazakh / Russian / English) and has
been narrowed to Kazakh for the first version (see [Current decisions](#current-decisions)).

## Why

- Kazakh is a low-resource language. Kazakh Wikipedia has 245,865 articles,
  compared with 2.1M in Russian and 7.3M in English.
- The KazQAD paper (Yeshpanov et al., 2024) reports that ChatGPT-3.5 without
  retrieval could not answer KazQAD test questions at acceptable quality.
  Retrieval is the missing piece.
- Published Kazakh retrieval baselines leave a lot of room. KazQAD reports
  NDCG@10 = 0.389 and MRR = 0.382 for retrieval over the full corpus.

## Status

| Area | State |
|---|---|
| Problem framing | Done (Journal entries 1–3) |
| Dataset | Chosen: `issai/kazqad-retrieval`. Split usage still open |
| Replication target | Identified: `shyngys879/kazakh-e5-rag-embedding`. Protocol open |
| License survey | Done (Entry 6) |
| Base model | Open |
| Training memory plan (6 GB GPU) | Open |
| Deployment target | Open, after an earlier plan turned out not to work |
| Code | Not started |
| Results | None yet |

## Current decisions

Tags follow the journal: **Decided**, **Tentative**, **Open**, **Corrected**.

**Decided**
- v1 is a **retriever only**: it returns ranked passages. A generation layer
  (full RAG) may be added after I see real retrieval results.
- Deployment happens only **after a measured improvement** over baselines.
- Any comparison with other systems must use the same corpus, the same queries
  and the same metric. Otherwise it isn't a comparison.
- Approach: replicate one published model's results first, then improve on
  them. Build new data or metrics only if that turns out to be necessary.

**Tentative**
- Kazakh only for v1. This trades coverage for one language to evaluate, a
  smaller index and existing labelled data.

**Corrected along the way**
- Hugging Face ZeroGPU works only with Gradio Spaces, not Docker Spaces, and
  creating Gradio or Docker Spaces needs a PRO plan. The original "Docker +
  ZeroGPU" deployment plan is dropped.
- VRAM is fixed physical memory. This machine has an RTX 4050 Laptop GPU with
  6 GB, and CUDA does not spill into system RAM, so training has to fit in 6 GB.
- A CC BY-SA license is *more* restrictive than MIT or Apache-2.0 (share-alike),
  so it is not a reason to prefer a model.

## Data

[`issai/kazqad-retrieval`](https://huggingface.co/datasets/issai/kazqad-retrieval)
(CC BY-SA 4.0, Yeshpanov et al., LREC-COLING 2024):

- **Corpus:** about 800k Kazakh Wikipedia passages (dump of 2023-01-01), 137 MB gzipped.
- **Queries:** train, validation and test splits, with positive and negative passages.
  - Train questions are machine-translated Natural Questions.
  - Validation and test questions come from the real Kazakh UNT exam.

## Candidate models

All of these can be fine-tuned and published without contacting the author
(full table in Journal entry 6):

| Model | License | Params |
|---|---|---:|
| `intfloat/multilingual-e5-small` / `-base` / `-large` | MIT | 118M / 278M / 560M |
| `Nurlykhan/kazembed-v5` | Apache-2.0 | 278M |
| `sultanbi/e5-base-kazakh` | MIT | 278M |
| `Alibaba-NLP/gte-multilingual-base` | Apache-2.0 | 305M |
| `sentence-transformers/LaBSE` | Apache-2.0 | 471M |
| `BAAI/bge-m3` | MIT | ~568M |
| `Snowflake/snowflake-arctic-embed-l-v2.0` | Apache-2.0 | 568M |
| `Qwen/Qwen3-Embedding-0.6B` | Apache-2.0 | 596M |

Published Kazakh fine-tunes to compare against:
- `shyngys879/kazakh-e5-rag-embedding`: the replication target. The repository
  has weights only, with no evaluation code, and its license is "other" with no
  terms stated.
- `Tim2190/granite-278m-kk`: CC BY-SA 4.0. Its evaluation scripts are open,
  its training data was de-duplicated against the benchmarks, and it reports
  significance tests.

## TODO

Open questions from the journal. These need answers before any code is written.

- [ ] Language scope: stage-by-stage analysis of where extra languages add work
      (chunking, keyword index, embeddings, answer language), plus the tokenizer
      experiment.
- [ ] How each split (train / validation / test) is used, and why train-split
      scores may mislead.
- [ ] Size estimates: corpus, evaluation queries, training pairs.
- [ ] The metric and baselines that decide what "better" means.
- [ ] Replication protocol: comparability of 100-candidate MRR vs full-corpus
      MRR, missing candidate lists, a leakage check, the closest benchmark to
      real search use, and a tolerance for "replicated".
- [ ] Base model choice, justified by evidence other than the license.
- [ ] Training memory estimate for a 278M model on 6 GB, and a technique if it
      doesn't fit.
- [ ] Index size: 800k passages × embedding dimension × float32.
- [ ] Deployment target, and where the one-time embedding of the corpus runs.

Possible next steps once those are settled:

- [ ] Set up a Python environment (uv) and download KazQAD.
- [ ] Write an evaluation script that every model goes through unchanged.
- [ ] Run baselines on the validation split, and freeze the test split until the end.
- [ ] Attempt the replication and record whether it matches.
- [ ] Fine-tune the chosen base model and compare it on the same protocol.
- [ ] Test significance (for example, a paired bootstrap) before claiming an improvement.

## Future work

- **Generation layer:** turn the retriever into full RAG (retrieve, then
  answer), evaluated with answer metrics (EM / F1) separately from retrieval
  metrics.
- **Hybrid retrieval:** combine keyword matching (BM25) with dense embeddings.
- **Reranker:** add a cross-encoder stage on top of the retriever.
- **Trilingual extension:** add Russian and English, since much
  Kazakhstan-related content exists only in Russian.
- **Deployment:** a public demo once there is a measured improvement.
- **Release:** model and evaluation code on Hugging Face, with dataset
  attribution and the license stated.

## Project structure

```
kazakh_rag/
├── README.md
├── docs/
│   └── research-journal.md   # design log: reasoning, evidence, decisions
└── src/                      # empty, code not started
```

## References

See the [References section of the journal](docs/research-journal.md#references).
Key sources: KazQAD (arXiv:2404.04487), BEIR (arXiv:2104.08663), DPR
(arXiv:2004.04906), BGE M3 (arXiv:2402.03216), and Petrov et al. 2023 on
tokenizer unfairness (arXiv:2305.15425).

## License

Not chosen yet. Note that KazQAD is CC BY-SA 4.0, and any model trained on it
should attribute the dataset and state its license.
