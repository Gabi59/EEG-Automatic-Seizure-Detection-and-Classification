# Master Pipeline Orchestrator

## Trigger & Objective
When invoked with commands such as *"Run orchestrator"*, *"Execute pipeline"*, or *"Process all papers"*, the agent acts as the master coordinator. It executes the end-to-end research document processing pipeline across all three specialized sub-agents in strict sequence.

---

## Workspace Architecture

```
d:\Dizertatie\
├── PDFs/              <-- Input: Raw research PDF documents
├── text/              <-- Output Stage 1: Extracted text files (*.txt)
├── Summary/           <-- Output Stage 2: Structured research summaries (*.md)
└── Open_Problems/     <-- Output Stage 3: Extracted open problems & challenges (*.md)
```

---

## Sequential Execution Pipeline

### Stage 1: Text Extraction & Normalization
* **Rule Reference**: [`text_agent.md`](file:///d:/Dizertatie/text_agent.md)
* **Input Directory**: `./PDFs/`
* **Output Directory**: `./text/`
* **Procedure**:
  1. Scan `PDFs/` for all `.pdf` documents.
  2. For each discovered file `PDFs/<name>.pdf`, check if `text/<name>.txt` exists.
  3. Extract full text across all pages preserving headings, tables, formulas, and references in UTF-8 format.
  4. Write the normalized text to `text/<name>.txt`.
* **Gate / Dependency Check**: Verify that `text/<name>.txt` exists and is non-empty before advancing to Stage 2 for that document.

---

### Stage 2: Scientific Document Summarization
* **Rule Reference**: [`sumary_rules.md`](file:///d:/Dizertatie/sumary_rules.md)
* **Input Directory**: `./text/`
* **Output Directory**: `./Summary/`
* **Procedure**:
  1. For each `text/<name>.txt`, check if `Summary/<name>.md` exists.
  2. Parse the text and generate a structured summary containing:
     - **YAML Front Matter**: Fixed-schema standardized metadata block (task, population, dataset, volume, montage, segmentation, features, classifier, leakage audit, key metrics).
     - `# General sumary`: Paper metadata, paper type, task, population/clinical setting, dataset volume, and exclusions.
     - `# Data sumary`: Preprocessing/segmentation, handcrafted vs. learned features, classification pipeline, and leakage/validity audit.
     - `# Conclusions`: Quantitative metrics (level-specified), related work comparisons, runtime, and strictly separated author-stated vs. observed limitations.
  3. Save the result to `Summary/<name>.md`.

---

### Stage 3: Open Problems & Challenges Extraction
* **Rule Reference**: [`open_problems.md`](file:///d:/Dizertatie/open_problems.md)
* **Input Directory**: `./text/`
* **Output Directory**: `./Open_Problems/`
* **Procedure**:
  1. For each `text/<name>.txt`, check if `Open_Problems/<name>.md` exists.
  2. Extract and synthesize open research questions categorized into:
     - Detection vs. Pre-ictal Prediction gaps.
     - Methodological & Algorithmic bottlenecks (spiking neural networks, deep learning, seizure subtypes).
     - Data & Validation constraints (class imbalance, continuous real-world streams, SMOTE validity).
     - Clinical, Hardware & Ambulatory hurdles (real-time artifacts, tele-EEG telemetry).
     - Explicit author-stated future directions.
     - Concrete research opportunities for subsequent dissertation research.
  3. Save the result to `Open_Problems/<name>.md`.

---

## Error Handling & Dependency Gates
- **Missing Directories**: Automatically create `text/`, `Summary/`, or `Open_Problems/` if they do not exist.
- **Incremental Processing**: If an output file for a given stage already exists and is up to date, the orchestrator skips re-generating it unless an explicit refresh is requested.
- **Failure Isolation**: If one PDF encounters an extraction error, log the failure and continue processing remaining documents.

---

## Consolidated Pipeline Report
At the conclusion of the run, the orchestrator must output a consolidated status table:

| Document Base Name | Stage 1 (Text Extracted) | Stage 2 (Summary Created) | Stage 3 (Open Problems Found) | Status |
| :--- | :---: | :---: | :---: | :---: |
| `example_paper` | ✅ `text/example_paper.txt` | ✅ `Summary/example_paper.md` | ✅ `Open_Problems/example_paper.md` | Complete |

