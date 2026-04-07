# Clinical Text Summarization

An end-to-end NLP pipeline that **automatically summarizes clinical medical documents** using a T5-Large transformer model. The system extracts text from PDF clinical documents, preprocesses it with spaCy, applies a hierarchical abstractive summarization strategy, and evaluates output quality with ROUGE metrics.

---

## Why This Matters

Clinical documents — drug labels, vaccine information sheets, prescribing guides — are often 10-20+ pages of dense medical text. Clinicians, pharmacists, and researchers need to extract key information quickly. This pipeline compresses lengthy clinical PDFs into concise, accurate summaries in seconds, preserving critical details like dosage, contraindications, and adverse reactions.

---

## Pipeline Architecture

```
PDF Document
     │
     ▼
┌─────────────────┐
│  Text Extraction │  ← Apache Tika parses raw text from PDF
└────────┬────────┘
         ▼
┌─────────────────┐
│  Text Cleaning   │  ← Regex-based normalization (whitespace, newlines)
└────────┬────────┘
         ▼
┌─────────────────┐
│  NLP Preprocessing│ ← spaCy en_core_web_lg: sentence segmentation + lemmatization
└────────┬────────┘
         ▼
┌─────────────────┐
│  Smart Chunking  │  ← Token-aware splitting (512 token limit per chunk)
└────────┬────────┘
         ▼
┌─────────────────┐
│  Hierarchical    │  ← Stage 1: Summarize each chunk (max 50 tokens)
│  Summarization   │  ← Stage 2: Combine + re-summarize into final output (max 100 tokens)
│  (T5-Large)      │
└────────┬────────┘
         ▼
┌─────────────────┐
│  Post-Processing │  ← Domain-specific noise removal
└────────┬────────┘
         ▼
┌─────────────────┐
│  ROUGE Evaluation│  ← Precision, Recall, F1 across ROUGE-1, ROUGE-2, ROUGE-L
└────────┬────────┘
         ▼
   Final Summary + Metrics Visualization
```

---

## Tech Stack

| Technology | Purpose |
|---|---|
| **T5-Large** (770M params) | Abstractive summarization — encoder-decoder transformer via Hugging Face |
| **spaCy** (`en_core_web_lg`) | NLP preprocessing — sentence tokenization, lemmatization |
| **Apache Tika** | PDF text extraction from clinical documents |
| **rouge_score** | ROUGE-1, ROUGE-2, ROUGE-L evaluation (precision, recall, F1) |
| **Plotly Express** | Interactive visualization of evaluation metrics |
| **Pandas** | Structured results management |
| **Python 3** | Core language |

---

## How It Works

### 1. PDF Text Extraction
Clinical documents are ingested as PDFs and parsed using Apache Tika, which handles complex PDF layouts including multi-column text, headers, and footnotes.

### 2. NLP Preprocessing
The extracted text is processed through spaCy's large English model (`en_core_web_lg`) for:
- **Sentence segmentation** — splitting the document into individual sentences
- **Lemmatization** — normalizing word forms (e.g., "administered" → "administer") for cleaner input

### 3. Token-Aware Chunking
Clinical documents exceed transformer token limits. The chunking algorithm:
- Tracks token count per chunk using the T5 tokenizer
- Splits at sentence boundaries (never mid-sentence) to preserve meaning
- Respects the 512-token context window of the model

### 4. Hierarchical Abstractive Summarization
A **two-stage summarization** strategy using T5-Large:

| Stage | Input | Output | Max Tokens |
|---|---|---|---|
| Chunk-level | Individual text chunks | Per-chunk summaries | 50 |
| Document-level | Combined chunk summaries | Final clinical summary | 100 |

This approach handles documents of arbitrary length while maintaining coherence in the final output.

### 5. Evaluation with ROUGE Metrics
Generated summaries are scored against human-written reference summaries:

| Metric | What It Measures |
|---|---|
| **ROUGE-1** | Unigram overlap — captures key term retention |
| **ROUGE-2** | Bigram overlap — captures phrase-level accuracy |
| **ROUGE-L** | Longest common subsequence — captures structural similarity |

Results are visualized as interactive Plotly bar charts comparing precision, recall, and F1 across all metrics.

---

## Demo: Clinical Document Summarization

**Input:** ENGERIX-B vaccine prescribing information (multi-page PDF covering indications, dosage, contraindications, adverse reactions, and special populations)

**Output:** A concise summary capturing the essential clinical information — vaccine purpose, administration schedule, contraindications, common adverse reactions, and special population considerations — compressed from pages of text into a single readable paragraph.

---

## Project Structure

```
clinical_text_summarization/
├── clinical_text_summarization.ipynb   # Full pipeline: extraction → summarization → evaluation
├── ENGERIX-B.pdf                       # Sample clinical document (Hepatitis B vaccine)
└── README.md
```

---

## Getting Started

```bash
# Clone the repository
git clone https://github.com/ams11271/clinical_text_summarization.git
cd clinical_text_summarization

# Install dependencies
pip install spacy tika rouge_score transformers plotly pandas
python -m spacy download en_core_web_lg

# Launch the notebook
jupyter notebook clinical_text_summarization.ipynb
```

> **Note:** T5-Large requires ~3GB of memory. A GPU-enabled environment (Google Colab, etc.) is recommended for faster inference.

---

## Key Design Decisions

| Decision | Rationale |
|---|---|
| **T5-Large** over smaller variants | Clinical text requires high fidelity — larger models better preserve medical terminology and factual accuracy |
| **Hierarchical summarization** over single-pass | Clinical documents exceed token limits; two-stage approach maintains coherence across long documents |
| **Sentence-boundary chunking** over fixed-length splits | Preserves semantic completeness — never cuts a clinical instruction mid-sentence |
| **Deterministic generation** (`do_sample=False`) | Ensures reproducible outputs — critical for clinical applications where consistency matters |

---

## Real-World Applications

- **Clinical decision support** — quickly surface key drug information at point of care
- **Pharmacovigilance** — summarize adverse event reports for rapid review
- **Regulatory document review** — compress FDA drug labels and safety communications
- **Medical literature synthesis** — distill lengthy research papers into key findings
- **EHR integration** — generate patient-facing summaries of treatment documentation

---

## About

Demonstrates applied NLP and transformer-based deep learning for healthcare — from raw document ingestion through model inference to quantitative evaluation. Built with a focus on the unique challenges of clinical text: domain-specific terminology, strict accuracy requirements, and long-document handling.
