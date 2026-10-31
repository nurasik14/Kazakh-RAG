# Research Journal: Kazakh Retrieval System

This is a log of the design reasoning behind this project. Each entry gives the
question, my reasoning, the evidence I checked, the corrections that came out of
review, and the outcome. Decisions are tagged:

- **Decided**: I chose this.
- **Tentative**: I'm leaning this way but haven't committed.
- **Open**: not yet answered.
- **Corrected**: my earlier assumption was wrong. The correction is given.

All external facts were checked on the date of the entry. The source is given
next to each one.

**Queries.** Every question or instruction I gave the AI assistant is quoted
word for word (typos included) as a numbered *Query* in the entry it belongs
to, and all of them are listed in order in the [Query log](#query-log) at the
end. The journal is updated after every exchange.

**Note on dates.** Session 1 ran across 2026-10-30 and 2026-10-31. Entries
from that session are labelled with both dates.

---

## Entry 0 — Session 1 (2026-10-30/31): Project setup and learning method

> **Query 1:** "we are building triangular RAG pipeline that at the end of the project must be real deployed system. That should showcase my understanding of ML/AI, Math, and MLOps. Before anything lets go to the trilingual_rag folder"
> **Query 2:** "On top of that I must understand every system design, decision and every math and concepts. If I dont know something AI should not tell the answer, but propose the readings and I should deduce the solution if not AI sends back or hints a bit"
> **Query 3:** "Also, dont forget about active recall. Try to question in the interval manner already learned concepts, maybe lightly, if wrong again came back to it"
> **Query 4:** "Lets continue"

**Note.** Query 1 says "triangular"; it means *trilingual* (kk/ru/en).

**Setup.**
- Goal: a deployed retrieval / RAG system that shows understanding of ML/AI,
  the math behind it, and MLOps.
- Learning method (my requirements):
  - I must understand every design decision, every concept and all the math.
  - When I don't know something, the AI suggests readings instead of the
    answer. I work out the solution. If I'm wrong, it sends it back with a
    small hint.
  - Active recall: concepts I have already demonstrated are quizzed again at
    spaced intervals (1 → 3 → 7 → 14 → 30 days). A wrong answer resets the
    interval.
  - I write most of the code; the AI reviews.
- Agreed exceptions: plain facts that can't be deduced (library syntax, flags,
  versions, prices) are stated directly with a source. Bugs are flagged with
  where to look, not the fix, except for security or data-loss risks.

---

## Entry 1 — Session 1 (2026-10-30/31): Problem framing

> **Query 5:** "What do you think about making Kazakh google? So people googling in kazakh could get relevant info accurately? I would build it using RAG, keyword matching and etc. Hybrid retrieval deployed system. It must retrieves data from the kazakh info database."

**Question.** Who is the system for, and what does it do?

**My reasoning.** A "Kazakh Google": people search in Kazakh and get accurate,
relevant information. It would be built with hybrid retrieval (keyword matching
plus RAG) over a Kazakh information database, and deployed as a real system.

**Review / critique.**
1. Real web search starts with crawling, which is a project in its own right. A
   fixed corpus keeps the scope manageable.
2. "Kazakh info database" doesn't exist as one thing. A concrete corpus has to be
   chosen.
3. "Accurately" needs a metric and labelled data. Without them it's a claim, not
   a result.
4. The repo is called *trilingual*, but this framing only mentions Kazakh.

**Evidence.**
- Kazakh Wikipedia: 245,865 articles, about 41.5M words (kk.wikipedia.org API).
- KazQAD (Yeshpanov et al., LREC-COLING 2024, arXiv:2404.04487): about 800k
  Kazakh Wikipedia passages, about 6k questions, about 12k passage-level
  relevance judgements, CC BY-SA 4.0. The paper reports that ChatGPT-3.5,
  answering without retrieval (closed-book), could not answer KazQAD test
  questions at acceptable quality.

**Outcome.** Refined into Entries 2–3.

---

## Entry 2 — Session 1 (2026-10-30/31): Language scope (Kazakh-only vs trilingual)

> **Query 6:** "Is making trilingual RAG pipeline haarder than single language?"

**Question.** Is a trilingual (kk/ru/en) pipeline harder than a single-language
one, and is it worth it?

**Evidence.**
- Wikipedia article counts: kk 245,865 · ru 2,125,784 · en 7,257,382 (each
  language's Wikipedia API). Much Kazakhstan-related content exists only in
  Russian.

> **Query 7:** "Should I concentrate on Kazakh RAG or Trilingual RAG"

**Tradeoff identified.** Kazakh-only gives up coverage. In return it gives one
language to evaluate, a smaller index, and existing labelled data (KazQAD).
Trilingual gives up time and makes it harder to attribute evaluation results to
a cause, because every pipeline stage has to work in three languages. In return
it's more useful and a harder ML problem.

**Outcome.** **Tentative**: Kazakh-only for v1, implied by the later decisions
rather than stated directly. On 2026-10-31 the project folder was renamed from
`trilingual_rag` to `kazakh_rag`, and the README now says "Kazakh RAG".
**Open**: a stage-by-stage analysis of where extra languages add work (chunking,
keyword index, embeddings, answer language), and the tokenizer experiment
(Petrov et al. 2023, arXiv:2305.15425).

---

## Entry 3 — Session 1 (2026-10-30/31): System type and comparison

> **Query 8:** "What do you think about making a Kazakh documents and data retriever and comparing it to other RAG performances on the same data?"

**My reasoning.** Build a Kazakh document retriever, and compare it with other
systems on the same data.

**Review / critique.**
1. A comparison only means something if the corpus, the queries and the metric
   are identical.
2. Retrieval metrics (NDCG, MRR, Recall@k) and answer metrics (EM, F1) measure
   different things. I need to say which one I'm improving.
3. A benchmark result on its own isn't a deployed system.

**Evidence.** KazQAD's published baselines: retrieval NDCG@10 = 0.389,
MRR = 0.382. Full open-domain QA: EM = 17.8, F1 = 28.7.

> **Query 9:** "I think I will stick with retrieval model. After seeing real results maybe will connect the generation layer and make it full RAG. For now after endng my work with real improvment I will deploy the retrieval model with docker container on hugging face i think. Now lets get to the measuring and data finding"

**Decision.** **Decided**:
- v1 is a **retriever only**. It returns ranked passages.
- A generation layer (full RAG) may be added after I see real results.
- Deployment happens **after a measured improvement**.

**Open**: which baselines, and which metric decides "better" and why.
Reading: Manning et al., *Introduction to IR*, ch. 8; BEIR (arXiv:2104.08663).

---

## Entry 4 — Session 1 (2026-10-30/31): Dataset

**Evidence** (Hugging Face API):
- `issai/kazqad-retrieval`, CC BY-SA 4.0.
  - `corpus`: about 800k Kazakh Wikipedia passages (dump of 2023-01-01), one
    file of 137 MB gzipped.
  - `queries-and-passages`: train / validation / test splits. Each row has
    `question`, `positive_passages` and `negative_passages`.
  - **Train** questions are machine-translated Natural Questions. **Validation
    and test** questions come from the real Kazakh UNT exam.

> **Query 10:** "Can you suggest readding on picking the right dataset and estimate how much dataset size I should be aiming"

**Hint received.** "Dataset size" covers three independent quantities: corpus
size, number of evaluation queries, and number of training pairs.

**Open.**
- What each split will be used for.
- Why train-split scores may be misleading here, given that the splits come
  from different sources.
- A size estimate for each of the three quantities.

Readings: Datasheets for Datasets (arXiv:1803.09010); Voorhees & Buckley 2002
(doi:10.1145/564376.564432); Webber, Moffat & Zobel 2008
(doi:10.1145/1458082.1458158); Sakai 2016, "Topic set size design"
(doi:10.1007/s10791-015-9273-z); DPR (arXiv:2004.04906), the sample-efficiency
figure.

---

## Entry 5 — Session 1 (2026-10-30/31): Replication target

> **Query 11:** "I think first I will focus on replicating the results of the specific model. I chose the kazkah-5e-rag-embedding. And will try to improve based on their data and measuring scores. If necessary I will new dataset and new measurements"

**My reasoning.** Replicate one published model's results first, then improve
on them. Only build new data or metrics if that turns out to be necessary.

**Correction.** The model I named as "kazkah-5e-rag-embedding" is
`shyngys879/kazakh-e5-rag-embedding`.

**Evidence** (model card):
- Lineage: `intfloat/multilingual-e5-base` → `Nurlykhan/kazembed-v5` → this
  model. 278M parameters, 768-dimensional embeddings, `query:` / `passage:`
  prefixes.
- Its own benchmark, "OfficialKazQAD-HardTFIDF99": 1,929 KazQAD test queries,
  each ranked against 1 positive and 99 TF-IDF hard negatives. Scores:
  Hits@1 35.98%, Hits@5 72.47%, MRR 0.5189.
- On WikiFullCorpus it scores MRR 0.6689, below mE5-base (0.7290). The card
  itself lists this as a limitation.
- The repository contains weights only: no evaluation code and no candidate
  lists.
- License: "other", with no terms stated.

**Open questions for replication.**
1. Is a 100-candidate MRR comparable with KazQAD's MRR over the full corpus?
2. Without the candidate lists, how can a re-run be shown to be the same
   experiment?
3. The model was trained on KazQAD data and evaluated on KazQAD test queries.
   What has to be checked for leakage?
4. Which benchmark is closest to the real search use case?
5. What counts as "replicated" (what tolerance)?

---

## Entry 6 — Session 1 (2026-10-30/31): License survey

> **Query 12:** "Can you find other models that I can publish online without contacting the author. Would I still be able to compare its results to provided this 5e-rag-embedding model results"

**Question.** Which models can I fine-tune and publish without contacting the
author?

**Evidence** (Hugging Face model metadata):

| Model | License | Params |
|---|---|---:|
| Nurlykhan/kazembed-v5 | Apache-2.0 | 278M |
| intfloat/multilingual-e5-small / base / large / large-instruct | MIT | 118M / 278M / 560M / 560M |
| BAAI/bge-m3 | MIT | ~568M |
| Alibaba-NLP/gte-multilingual-base | Apache-2.0 | 305M |
| Snowflake/snowflake-arctic-embed-l-v2.0 | Apache-2.0 | 568M |
| sentence-transformers/LaBSE | Apache-2.0 | 471M |
| Qwen/Qwen3-Embedding-0.6B | Apache-2.0 | 596M |
| sultanbi/e5-base-kazakh | MIT | 278M |
| jinaai/jina-embeddings-v3 | CC BY-NC 4.0 (non-commercial) | 572M |
| google/embeddinggemma-300m | Gemma license, gated | 303M |

**Notes.**
- Citing a published number needs no license. Running or redistributing a
  model is a separate question.
- KazQAD is CC BY-SA 4.0 (share-alike). Whether a model trained on share-alike
  data must itself be share-alike is legally unsettled, as far as I know. The
  usual practice is to attribute the dataset and state its license. This is not
  legal advice.

---

## Entry 7 — Session 1 (2026-10-30/31): Deployment platform

> **Query 13:** "What about hugging face space? ZeroGPU (Dynamic GPU Allocation): Available for free to all authenticated users to run AI apps and demos powered by Nvidia RTX Pro 6000 Blackwell GPUs (with up to 96 GB VRAM). Daily Quota: Free accounts receive a daily GPU quota (typically around 3.5 to 5 minutes of active GPU usage per day, depending on account verification) which resets every 24 hours. Unauthenticated users receive about 2 minutes. What if I pick Qwen/Qwen3-Embedding-0.6B and compare it to previous kazakh-5e-rag-embedding and other models"

(The same query also proposes Qwen3; see Entry 8.)

**My reasoning.** Deploy the retriever as a Docker container on Hugging Face,
possibly using ZeroGPU.

**Corrected.** The HF docs (`spaces-zerogpu`, `spaces-overview`) say:
- ZeroGPU works **only with the Gradio SDK**, not with Docker Spaces.
- Gradio and Docker Spaces need a **PRO plan to create**, even on free CPU Basic
  hardware (2 vCPU, 16 GB RAM). Static Spaces are free.
- Free accounts in good standing (verified email, account older than 30 days)
  can host up to 2 ZeroGPU Gradio Spaces.
- Daily GPU quota: 2 min unauthenticated, 5 min for a free account, 40 min for
  PRO. Default 60 s per call. As far as I know, the quota is charged to the
  user calling the Space.

**Outcome.** **Open**: the deployment target needs a fresh decision.
**Open**: where the one-time embedding of the 800k-passage corpus happens,
versus per-query embedding at request time.

---

## Entry 8 — Session 1 (2026-10-30/31): Base model choice

**My reasoning, round 1** (Query 13). Use `Qwen/Qwen3-Embedding-0.6B` (Apache-2.0, 596M
parameters, 1024-dim with MRL down to 32, 32k context, instruction-aware, "100+
languages" with Kazakh not explicitly listed), and compare it with kazakh-e5 and
others.

> **Query 14:** "2) I can alocate more VRAM for GPU training. I think 12 GB will be enough? 3) Yeah I feel like it is unfair to compare to the model with double the params. Then So I found This three models that would be fair to compare between themselves: sultanbi/e5-base-kazakh; Nurlyhkan/kazembed-v5; shyngys879/kazakh-e5-rag-embedding; Tim2190/granite-278m-kk; Now I will choose most possibly the granite model because it is under CC BY-SA 4"

**My reasoning, round 2** (Query 14). Comparing against a model with twice the parameters
is unfair, so restrict the comparison to the same size class (278M):
`sultanbi/e5-base-kazakh`, `Nurlykhan/kazembed-v5`,
`shyngys879/kazakh-e5-rag-embedding` and `Tim2190/granite-278m-kk`. Lean toward
granite because it is CC BY-SA 4.0.

**Review / corrections.**
- Comparing within a size class is a valid axis: it isolates the method from
  scale. **But** that is a different kind of fairness from protocol fairness,
  which asks what must be identical for two scores to be comparable at all.
  Both are needed.
- **Corrected**: CC BY-SA is *more* restrictive than MIT or Apache-2.0, because
  share-alike means anything published that builds on it must also be CC BY-SA.
  It isn't a reason to prefer a model. Granite's base model
  (`ibm-granite/granite-embedding-278m-multilingual`) is Apache-2.0. The
  fine-tune is CC BY-SA because it was trained on KazQAD.

**Evidence to weigh** (granite model card):
- Open evaluation scripts at github.com/Tim2190/kazakh-granite-retriever
  (Apache-2.0).
- Training data de-duplicated against the benchmarks ("anti-leak").
- Paired-bootstrap significance tests (10k resamples).
- Reports a statistical tie with kazakh-e5 overall (p = 0.42).
- Training data: 57,369 synthetic pairs plus 3,733 KazQAD gold pairs.
  Loss: CachedMultipleNegativesRankingLoss. Max sequence length 512.

**Outcome.** **Open**: choose a base model and justify it with evidence other
than the license.

---

## Entry 9 — Session 1 (2026-10-30/31): GPU memory

**My reasoning** (Query 14, part 2). "I can allocate more VRAM for training; 12 GB will be
enough?"

**Corrected.** VRAM is fixed physical memory on the GPU and cannot be
increased. Measured on this machine:

```
$ nvidia-smi --query-gpu=name,memory.total --format=csv
NVIDIA GeForce RTX 4050 Laptop GPU, 6141 MiB
$ free -h
Mem: 15Gi total, ~6.7Gi available   # system RAM, not VRAM
```

On Linux, CUDA does not spill into system RAM by default. Running out of VRAM
gives a "CUDA out of memory" error.

**Open**: estimate the memory needed for full fine-tuning of a 278M model
(weights, gradients, optimizer state, activations), and whether it fits in
6 GB. If it doesn't, decide which technique to use. Readings: HF "Model training
anatomy", HF "Efficient training on a single GPU", ZeRO (arXiv:1910.02054)
§3.1.

---

## Entry 10 — Session 1 (2026-10-30/31): Writing this journal

> **Query 15:** "Write down this reasonning of mine with your corrections summarized as scientific experiment or journal inside some md file in the repo folder"
> **Query 17:** "Have you written this reasonning in the md file? Write every entry and query to you"

**Outcome.** This file was created as `docs/research-journal.md`. Query 17
added the convention of quoting every query (see the header).

---

## Entry 11 — Session 1 (2026-10-30/31): Base model choice, round 3

> **Query 16:** (quoted in full in the [Query log](#query-log); parts 1–3 are covered in Entries 11–13)

**My reasoning** (Query 16, part 1). Choose Granite because it is lightweight,
about 0.5 GB compared with the other models, which leaves more VRAM for
training.

**Evidence** (Hugging Face API, `safetensors` metadata):

| Model | Params | Stored as | File size |
|---|---:|---|---:|
| Tim2190/granite-278m-kk | 278,043,648 | F16 | 556 MB |
| shyngys879/kazakh-e5-rag-embedding | 278,043,648 | F32 | 1.11 GB |
| Nurlykhan/kazembed-v5 | 278,043,648 | F32 | 1.11 GB |
| sultanbi/e5-base-kazakh | 278,043,648 | F32 | 1.11 GB |

**Sent back.** All four have exactly the same number of parameters. The file
size differs only because of the storage data type: 2 bytes per number versus
4. My own bytes-per-parameter calculation in Entry 12 gives the same training
memory for all four, so file size isn't an argument for any of them.

**Outcome.** **Open**: choose again, using what actually differs between the
four (training data, available evaluation code, leakage handling, the base
model's license). See Entries 5 and 8.

---

## Entry 12 — Session 1 (2026-10-30/31): Training memory estimate, round 2

**My reasoning** (Query 16, part 2).
- Model states, following the HF "Model training anatomy" page, for
  mixed-precision training with Adam (assumed, since Granite's optimizer is
  unknown):
  - Weights: 6 bytes/param (fp16 copy + fp32 master copy) → 1.54
  - Adam optimizer states: 8 bytes/param → 2.05
  - Gradients: 4 bytes/param → 1.03
  - Total: 4.62
- "Spikes": HF's example uses 85 GB for a 4B model, of which about 67 GB is
  model states, leaving 18 GB of extra. Scaling by the same ratio:
  67 / 85 = 4.62 / x → x = 5.86. That leaves 1.24 for spikes and fits in
  6 GB.

**Review.**
- **Correct**: 18 bytes/param for mixed-precision Adam model states, and the
  multiplication 278M × 18 B. Assuming Adam was reasonable.
- **Units**: these figures are GiB (2^30 bytes), not GB (10^9 bytes). For
  example, 278M × 6 B = 1.67 GB = 1.55 GiB. The GPU is 6141 MiB ≈ 6.0 GiB.
  Use one unit throughout.
- **Margin**: even on this estimate, 5.86 of 6.0 leaves only 0.14 GiB, which
  isn't "safe".
- **Sent back**: the spike estimate. The HF example reads: "Training a 4B
  parameter model in mixed precision on a batch size of 16 requires roughly
  85GB." Scaling by parameter count assumes the extra memory grows with the
  number of parameters. The same page says activations "vary in size with
  batch size, sequence length, model depth, and hidden size". Activations
  were left out of the estimate.

**Outcome.** **Open**:
- Redo the estimate with activations included. Which settings do I control,
  and what did Granite use? (Its card says max_seq_len 512.)
- Then measure on the real GPU: one training step, then
  `torch.cuda.max_memory_allocated()`, and compare with the estimate.

---

## Entry 13 — Session 1 (2026-10-30/31): Learning materials for the tokenizer and index experiments

**Query** (Query 16, part 3): I'm new to tokenizers and embedding indexes and
asked what each experiment is for, plus materials.

**What each experiment is for.**
- *Tokenizer experiment.* A tokenizer cuts text into pieces (tokens) and maps
  each piece to an ID. The model only ever sees the IDs. The experiment
  answers:
  - (a) How many tokens does a Kazakh sentence cost compared with English or
    Russian?
  - (b) Do the pieces look like real parts of Kazakh words, or like
    meaningless fragments?
  - **Open**: what (a) means for the 512-token limit and for compute cost,
    and what (b) means for retrieval quality.
- *Index experiment.* An embedding model turns each passage into a vector
  (768 numbers for the 278M models). The index stores all passage vectors so
  that the nearest ones to a query vector can be found quickly. The
  experiment answers: how much memory do 800k vectors take, and does that fit
  on the search machine (15 GB of local RAM, or a 16 GB HF CPU Space)?
  - **Open**: the calculation itself.

**Readings, in order.**
1. HF LLM Course ch. 6, sections 1 and 5 (BPE):
   https://huggingface.co/learn/llm-course/chapter6/1
2. Karpathy, "Let's build the GPT Tokenizer" (first 30 minutes):
   https://www.youtube.com/watch?v=zduSFxRajkE
3. Petrov et al. 2023, arXiv:2305.15425
4. Then run the tokenizer experiment.
5. Vicki Boykis, "What are embeddings":
   https://vickiboykis.com/what_are_embeddings/
6. Jay Alammar, "The Illustrated Word2vec":
   https://jalammar.github.io/illustrated-word2vec/
7. SBERT, "Semantic search":
   https://sbert.net/examples/sentence_transformer/applications/semantic-search/README.html
8. Pinecone's Faiss tutorial:
   https://www.pinecone.io/learn/series/faiss/faiss-tutorial/
   Optional: the Faiss wiki "Faiss indexes", and the Faiss paper
   (arXiv:2401.08281).
9. Then do the index-size calculation.

---

## Open questions (consolidated)

- [ ] Language scope: stage-by-stage analysis, plus the tokenizer experiment.
- [ ] Split usage (train / validation / test), and why train scores may mislead.
- [ ] Size estimates: corpus, evaluation queries, training pairs.
- [ ] Metric and baselines that decide "better".
- [ ] Replication target and tolerance (Entry 5, questions 1–5).
- [ ] Base model choice, with evidence other than the license or file size (Entry 11).
- [ ] Training memory estimate including activations, then a measurement (Entry 12).
- [ ] Tokenizer experiment and what it means for this project (Entry 13).
- [ ] Index size: 800k passages × embedding dimension × float32.
- [ ] Deployment target.

## References

- Lewis et al. 2020. Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks. arXiv:2005.11401
- Yeshpanov et al. 2024. KazQAD. LREC-COLING 2024. arXiv:2404.04487
- Petrov et al. 2023. Language Model Tokenizers Introduce Unfairness Between Languages. arXiv:2305.15425
- Chen et al. 2024. BGE M3-Embedding. arXiv:2402.03216
- Thakur et al. 2021. BEIR. arXiv:2104.08663
- Manning, Raghavan & Schütze. Introduction to Information Retrieval, ch. 8. https://nlp.stanford.edu/IR-book/
- Gebru et al. 2018. Datasheets for Datasets. arXiv:1803.09010
- Voorhees & Buckley 2002. The effect of topic set size on retrieval experiment error. SIGIR. doi:10.1145/564376.564432
- Webber, Moffat & Zobel 2008. Statistical power in retrieval experimentation. CIKM. doi:10.1145/1458082.1458158
- Sakai 2016. Topic set size design. Information Retrieval Journal. doi:10.1007/s10791-015-9273-z
- Karpukhin et al. 2020. Dense Passage Retrieval. arXiv:2004.04906
- Rajbhandari et al. 2020. ZeRO. arXiv:1910.02054
- Hugging Face docs: spaces-zerogpu, spaces-overview, model_memory_anatomy, perf_train_gpu_one
- Hugging Face LLM Course, ch. 6 (tokenizers). https://huggingface.co/learn/llm-course/chapter6/1
- Karpathy, A. Let's build the GPT Tokenizer (video). https://www.youtube.com/watch?v=zduSFxRajkE
- Boykis, V. What are embeddings. https://vickiboykis.com/what_are_embeddings/
- Alammar, J. The Illustrated Word2vec. https://jalammar.github.io/illustrated-word2vec/
- Reimers & Gurevych. Sentence-Transformers docs, Semantic search. https://sbert.net
- Douze et al. 2024. The Faiss library. arXiv:2401.08281

## Query log

Every query to the AI assistant, in order. Q16 is quoted only here because of its length.

1. (Entry 0) "we are building triangular RAG pipeline that at the end of the project must be real deployed system. That should showcase my understanding of ML/AI, Math, and MLOps. Before anything lets go to the trilingual_rag folder"
2. (Entry 0) "On top of that I must understand every system design, decision and every math and concepts. If I dont know something AI should not tell the answer, but propose the readings and I should deduce the solution if not AI sends back or hints a bit"
3. (Entry 0) "Also, dont forget about active recall. Try to question in the interval manner already learned concepts, maybe lightly, if wrong again came back to it"
4. (Entry 0) "Lets continue"
5. (Entry 1) "What do you think about making Kazakh google? So people googling in kazakh could get relevant info accurately? I would build it using RAG, keyword matching and etc. Hybrid retrieval deployed system. It must retrieves data from the kazakh info database."
6. (Entry 2) "Is making trilingual RAG pipeline haarder than single language?"
7. (Entry 2) "Should I concentrate on Kazakh RAG or Trilingual RAG"
8. (Entry 3) "What do you think about making a Kazakh documents and data retriever and comparing it to other RAG performances on the same data?"
9. (Entry 3) "I think I will stick with retrieval model. After seeing real results maybe will connect the generation layer and make it full RAG. For now after endng my work with real improvment I will deploy the retrieval model with docker container on hugging face i think. Now lets get to the measuring and data finding"
10. (Entry 4) "Can you suggest readding on picking the right dataset and estimate how much dataset size I should be aiming"
11. (Entry 5) "I think first I will focus on replicating the results of the specific model. I chose the kazkah-5e-rag-embedding. And will try to improve based on their data and measuring scores. If necessary I will new dataset and new measurements"
12. (Entry 6) "Can you find other models that I can publish online without contacting the author. Would I still be able to compare its results to provided this 5e-rag-embedding model results"
13. (Entry 7, 8) "What about hugging face space? ZeroGPU (Dynamic GPU Allocation): Available for free to all authenticated users to run AI apps and demos powered by Nvidia RTX Pro 6000 Blackwell GPUs (with up to 96 GB VRAM). Daily Quota: Free accounts receive a daily GPU quota (typically around 3.5 to 5 minutes of active GPU usage per day, depending on account verification) which resets every 24 hours. Unauthenticated users receive about 2 minutes. What if I pick Qwen/Qwen3-Embedding-0.6B and compare it to previous kazakh-5e-rag-embedding and other models"
14. (Entry 8, 9) "2) I can alocate more VRAM for GPU training. I think 12 GB will be enough? 3) Yeah I feel like it is unfair to compare to the model with double the params. Then So I found This three models that would be fair to compare between themselves: sultanbi/e5-base-kazakh; Nurlyhkan/kazembed-v5; shyngys879/kazakh-e5-rag-embedding; Tim2190/granite-278m-kk; Now I will choose most possibly the granite model because it is under CC BY-SA 4"
15. (Entry 10) "Write down this reasonning of mine with your corrections summarized as scientific experiment or journal inside some md file in the repo folder"
16. (Entry 11, 12, 13) "1) I will choose Granite model. Because it is lightweight ~0.5 GB, compared to other models. Which leaves more VRAM for training. 2) According to my relative calculations Granite must fit inside 6 GB VRAM. From my HF readding tensors are stored in the model fp16 for each param for backward/forward pass, fp32 for each param to hold the weight values, which is 6 bytes per param => 1.54 GB. Next is optimizer. Because I could not find what optimizer Granite used I estimate it using Adam optimizer. Essentially, adam optimizer stores 8 bytes per parameter, which is 2.05 GB. Next is gradients, I took it as in HF too which is 4 bytes per param or 1.03GB. That builds up to 4.62GB. Next is spike estimation, I could not figure out how to calculate it so I used the relative calculation using the values from HF. In the HF memory allocation it says that using 85 GB memory is enough for running 4B parameter model. That 4B paramter model without spike have 67GB space allocation. That means 18GB is reserved for potential spikes. Now we divide 67GB / 85GB = 4.62GB / x => x = 5.86GB. That means we can safely reserve 1.24GB for spikes. But my GPU have 6GB of space, so it is enough to train the Granite in my system. 3) I am not familiar with tokenizer and index experiments. I am completely noob in this field and doing this project to learn, so could you pin point such stakes and give me material to read and learn."
17. (Entry 10) "Have you written this reasonning in the md file? Write every entry and query to you"
