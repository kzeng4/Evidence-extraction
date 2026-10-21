# CONSORT item extraction — v34 checkpoint (train-only prototype)

Sentence-level labelling of randomized-trial articles with CONSORT 2010 checklist items (34 items, multi-label). The system combines an LLM extractor, a supervised PubMedBERT classifier and a classifier-triaged LLM verifier.

| path | what it is |
|---|---|
| **`CONSORT_v34_end_to_end.ipynb`** | The whole process in one self-contained notebook: Step 1 LLM extractor, enriched features, residual classifier, triage, verifier with corpus evidence, optional prompt development, validation selection, worked examples, baselines with error accounting, a single test run, and a prototype leakage check. It includes a glossary of all terms. |
| `inputs/` | The six input files the notebook reads (data, split, annotation guideline, train-only prototype, CONSORT checklist). Nothing else is needed. |
| `outputs/` | Created by the notebook (git-ignored): Step 1 records, embeddings, classifier runs, verifier decisions, baselines, selection, test lock. |

## Pipeline

1. **LLM extractor** (v26 pipeline, `gpt-5.6-terra`): router → article memory → section extractors. Each extracted (item, sentence) pair comes with a one-sentence rationale. The prototype is rebuilt from train articles only.
2. **Enriched input:** sentence + context, plus the LLM pseudo-labels of the sentence, its paragraph and its neighbours, CONSORT-graph cross-references, and the extractor's rationale.
3. **Classifier:** PubMedBERT with an LLM-prior residual head, `logit = prior(LLM label) + correction(x)`. Trained on train articles only.
4. **Triage:** (sentence, item) pairs where the LLM and the classifier's top-3 disagree.
5. **Verifier:** one LLM call per paragraph. It gets the extractor's guidance, corpus evidence from train gold (annotated examples, near-misses, item statistics) and the classifier probability, and decides keep/remove or add/reject.

## Results (validation, 10 articles)

| system | P | R | F1 |
|---|---|---|---|
| LLM only (checklist definitions) | 0.596 | 0.578 | 0.587 |
| PubMedBERT text-only | 0.737 | 0.697 | 0.716 |
| Step 1 extractor | 0.690 | 0.821 | 0.750 |
| Step 1 + classifier decides | 0.722 | 0.805 | 0.761 |
| **Step 1 + verifier** | 0.736 | 0.795 | **0.765** |

Test (10 articles) is evaluated once, in Part H.

## Running

```
pip install "dspy==3.1.3" torch transformers scikit-learn pandas numpy tqdm
export OPENAI_API_KEY=...                       # or type it when prompted
export PUBMEDBERT_PATH=/path/to/local/model     # optional, for machines without Hugging Face access
jupyter lab CONSORT_v34_end_to_end.ipynb        # run all
```

The notebook computes everything from `inputs/`. Every stage saves its results in `outputs/` and skips work that is already there, so a run can be interrupted and resumed. Classifier runs store their config and are re-used only if the config matches. Training needs a GPU: 11 fine-tuning runs × 5 epochs.

**About the saved outputs.** The outputs saved in this notebook were produced by this code with the original experiment's Step 1 records, classifier probabilities and verifier decisions placed in `outputs/`. In that state the run made no LLM calls and did no training, and it reproduces the experiment's numbers exactly. A fresh run will differ somewhat: the LLM runs at temperature 1, and GPU training is not bit-exact. The graph-evidence text also differs in about 2% of sentences, where two graph edges tie on similarity. The experiment broke those ties by set order; this notebook sorts them.
