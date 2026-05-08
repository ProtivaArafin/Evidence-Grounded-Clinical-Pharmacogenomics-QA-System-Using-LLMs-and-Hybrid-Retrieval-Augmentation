# Evidence-Grounded Clinical Pharmacogenomics Question Answering System

### Using Large Language Models and Hybrid Retrieval Augmentation

> \\\*\\\*MSc Thesis — Lakehead University, 2026\\\*\\\*
> \\\*\\\*Author:\\\*\\\* Protiva Arafin | \\\[LinkedIn](https://www.linkedin.com/in/protiva-arafin-036838286/) | \\\[GitHub](https://github.com/ProtivaArafin)
> \\\*\\\*Supervisor:\\\*\\\* Dr. Abedalrhman Alkhateeb
> \\\*\\\*Status:\\\*\\\* 📄 Paper under review — core implementation available

\---

## Overview

This project presents a **hybrid Retrieval-Augmented Generation (RAG) framework** for clinical pharmacogenomics question answering. It combines fine-tuned large language models (LLaMA 3.1 and Qwen3) with a multi-stage retrieval pipeline to generate accurate, evidence-grounded responses to gene-drug interaction queries.

Pharmacogenomics (PGx) studies how a person's genetic makeup affects their response to drugs. Providing reliable, evidence-based answers to PGx queries is critical for clinical decision support, yet existing LLMs hallucinate or lack domain-specific grounding. This system addresses that gap.

\---

## Key Results

|Model|Accuracy|Precision|Recall|F1-Score|
|-|-|-|-|-|
|Base LLaMA|0.435|0.36|0.44|0.33|
|LoRA LLaMA|0.556|0.25|0.26|0.21|
|**RAG + LoRA LLaMA** ✅|**0.91**|**0.83**|**0.91**|**0.87**|
|Base Qwen|0.91|0.92|0.90|0.91|
|LoRA Qwen|0.32|0.46|0.25|0.19|
|RAG + LoRA Qwen|0.88|0.72|0.75|0.72|

> ✅ \\\*\\\*RAG + LoRA LLaMA\\\*\\\* selected as the best final model — highest accuracy, fairest class-wise performance, and most clinically reliable responses.

\---

## System Architecture

```
User Query
    │
    ▼
┌─────────────────────────────────┐
│   STAGE 1: Lexical Pre-Filtering │
│   Pattern matching on JSONL      │
│   (genes, drugs, alleles)        │
└────────────────┬────────────────┘
                 │
                 ▼
┌─────────────────────────────────┐
│   STAGE 2: Semantic Re-Ranking   │
│   Sentence embeddings +          │
│   Cosine similarity scoring      │
└────────────────┬────────────────┘
                 │
                 ▼
┌─────────────────────────────────┐
│   STAGE 3: ClinPGx Integration   │
│   Semantic retrieval of          │
│   clinical guideline text        │
└────────────────┬────────────────┘
                 │
                 ▼
┌─────────────────────────────────┐
│   STAGE 4: Prompt Construction   │
│   Evidence fusion from           │
│   CPIC + ClinPGx sources         │
└────────────────┬────────────────┘
                 │
                 ▼
┌─────────────────────────────────┐
│   STAGE 5: Response Generation   │
│   Fine-tuned LLaMA / Qwen        │
│   with LoRA adapters             │
└────────────────┬────────────────┘
                 │
                 ▼
         Evidence-Grounded
           Clinical Answer
```

\---

## Data Sources

### CPIC (Clinical Pharmacogenetics Implementation Consortium)

* Structured relational data exported from PostgreSQL in CSV format
* Merged using **PySpark** and **DuckDB** with inner joins
* Preprocessed into JSONL format for LLM fine-tuning
* 74,565 training samples after cleaning and normalization

### ClinPGx

* Unstructured clinical guideline annotations collected via semi-automated web extraction
* Segmented into 500-character chunks with 100-character overlap (sliding window)
* Used for semantic retrieval during inference

\---

## Models

### LLaMA 3.1-8B (Meta)

Fine-tuned using **LoRA (Low-Rank Adaptation)** for parameter-efficient training:

|Parameter|Value|
|-|-|
|Fine-Tuning Method|LoRA|
|Target Layers|q\_proj, k\_proj, v\_proj, o\_proj|
|Rank (r)|16|
|Alpha|32|
|Dropout|0.05|
|Training Samples|74,565|
|Max Sequence Length|2048 tokens|
|Training Steps|1000|
|Batch Size|1 (gradient accumulation: 8 steps)|

### Qwen3-8B

* \~8.2 billion parameters, 36 transformer layers
* 32 query heads / 8 key-value heads
* Context length: 32,768 tokens (extendable to 131,072 via YaRN)
* Same LoRA fine-tuning configuration applied

\---

## Evaluation Methods

**Classification Metrics:** Accuracy · Precision · Recall · F1-Score

**Text-Based Metrics:** BERTScore · Cosine Similarity · LLM-as-a-Judge

**Manual Evaluation:** Human review across 6 criteria — Accuracy, Relevance, Completeness, Clarity, Hallucination Mitigation, RAG Relevancy

\---

## Tech Stack

* **Language:** Python
* **LLM Framework:** Hugging Face Transformers
* **Fine-Tuning:** LoRA via PEFT
* **Data Processing:** PySpark, DuckDB, Pandas
* **Embeddings:** Sentence Transformers
* **Compute:** HPC systems (Graham, Nibi — Compute Canada)
* **Data Format:** JSONL, CSV, PostgreSQL

\---

## Repository Structure

```
├── data\\\_preprocessing/
│   ├── cpic\\\_preprocessing.py        # CPIC data cleaning \\\& JSONL conversion
│   └── clinpgx\\\_preprocessing.py     # ClinPGx extraction \\\& chunking
├── fine\\\_tuning/
│   ├── llama\\\_lora\\\_finetune.py       # LLaMA LoRA fine-tuning pipeline
│   └── qwen\\\_lora\\\_finetune.py        # Qwen LoRA fine-tuning pipeline
├── rag\\\_pipeline/
│   ├── lexical\\\_filter.py            # Stage 1: keyword-based pre-filtering
│   ├── semantic\\\_reranker.py         # Stage 2: embedding + cosine re-ranking
│   ├── clinpgx\\\_retriever.py         # Stage 3: ClinPGx semantic retrieval
│   └── prompt\\\_builder.py            # Stage 4: evidence fusion \\\& prompt construction
├── evaluation/
│   ├── classification\\\_metrics.py    # Accuracy, Precision, Recall, F1
│   ├── text\\\_metrics.py              # BERTScore, Cosine Similarity
│   └── llm\\\_judge.py                 # LLM-as-a-Judge evaluation
└── README.md
```

> ⚠️ \\\*\\\*Note:\\\*\\\* This repository contains core implementation components. Full training data, model weights, and complete pipeline are withheld pending paper publication.

\---

## Citation

If you reference this work, please cite:

```
Arafin, P. (2026). Evidence-Grounded Clinical Pharmacogenomics Question Answering
System Using Large Language Models and Hybrid Retrieval Augmentation.
MSc Thesis, Lakehead University. (Paper under review)
```

\---

## Contact

**Protiva Arafin**
📧 parafin1@lakeheadu.ca
🔗 [LinkedIn](https://www.linkedin.com/in/protiva-arafin-036838286/)
💻 [GitHub](https://github.com/ProtivaArafin)

\---

*This work was conducted at Lakehead University under the supervision of Dr. Abedalrhman Alkhateeb.*

