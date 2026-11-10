# Agent Rules: Comprehensive Research Paper Summarization

## Trigger & Objective
When invoked, the agent reads each `.txt` document from the `./text/` directory, performs a thorough, evidence-based scientific extraction across preprocessing, modeling, validation, and performance metrics, and writes an in-depth summary markdown file to `./Summary/<filename>.md`.

The extraction rules are tailored to handle diverse EEG machine learning literature, including classical detection, preictal prediction, continuous probabilistic forecasting, neonatal analysis, and patient-independent deep architectures.

---

## Output Document Structure

Each output file in `Summary/<filename>.md` must strictly follow this structure:
1. **YAML Front Matter** (Structured metadata block for automated table generation)
2. `# General sumary`
3. `# Data sumary`
4. `# Conclusions`

---

## 1. YAML Front Matter (Fixed Schema)

Every summary file must begin with a YAML front matter block adhering to the schema below. If any field is not mentioned in the source text, write `"Not reported"`. Do not leave fields empty or fabricate values.

```yaml
---
title: "Exact Paper Title"
authors:
  - "Author 1"
  - "Author 2"
year: 2024
paper_type: "proposal" # proposal | performance analysis | review | benchmark | clinical study
task: "prediction"     # detection | prediction | forecasting | multi-class | performance analysis | review
population: "adult"    # neonatal | paediatric | adult | mixed | not reported
clinical_setting: "presurgical monitoring" # presurgical monitoring | NICU | ambulatory | ultra-long-term implanted | etc.
datasets:
  - name: "EPILEPSIAE"
    patients_stated: 40
    patients_inconsistency: "None" # e.g. "Text states 24 cases representing 23 individuals" or "None"
    total_recording_hours: 5484.0
    seizures_total: 224
    seizures_train: 120
    seizures_test: 104
montage:
  channels_count: 19
  channels_harmonisation: "Uniform 19 channels (10-20 system)" # e.g. "Varies per fold (16-23)", "None"
  referencing: "bipolar" # bipolar | referential | mixed | not reported
segmentation:
  window_length_seconds: 5.0
  overlap_ratio_or_seconds: "0s (non-overlapping)"
  labeling_rule: "Window within SPH+SOP assigned preictal"
features:
  feature_type: "handcrafted" # handcrafted | learned | hybrid | not applicable
  feature_summary: "59 univariate linear features per channel"
artifact_handling: "CNN-based automatic removal" # none applied | CNN-based | bandpass/thresholding only | manual expert | not reported
classifier:
  paradigm: "patient-specific" # patient-specific | patient-independent (LOSO/LOPO) | mixed | not applicable
  model_family: "Logistic Regression, SVM Ensemble, SNN Ensemble"
  ensemble_voting: "15-model majority voting for SVM and SNN" # single model | majority voting | soft voting | not applicable
  class_balancing: "class weights (LogReg), systematic random undersampling (SVM/SNN)" # none | SMOTE | class weights | random undersampling | balanced batching | negative-capping
  post_processing: "Firing Power (moving average) + risk thresholds (0.3 / 0.7)" # none | firing power | temporal smoothing | soft voting | not applicable
leakage_audit:
  patient_disjoint: "not applicable (patient-specific)" # yes | no | unclear | not applicable | not stated
  balancing_timing: "after split (safe)" # after split (safe) | before split (leakage) | unclear | not stated
  tuning_isolation: "isolated to training seizures" # isolated to train/val | touched test data | not stated
key_metrics:
  metric_level: "event-level" # window-level | episode-level | event-level | patient-level
  primary_results:
    - name: "SS (Seizure Sensitivity)"
      standard_name: "sensitivity"
      value: "0.28 ± 0.37"
      unit: "fraction"
      source: "Table 1"
    - name: "FPR/h"
      standard_name: "FPR/h"
      value: "0.36 ± 0.40"
      unit: "events/hour"
      source: "Table 1"
---
```

---

## 2. Section `# General sumary`

### Paper Identification & Metadata
- **Title, Authors, Affiliations, Publication Venue, Year, DOI/URL**.
- **Paper Type**: Classify explicitly into `proposal`, `performance analysis`, `review`, `benchmark`, or `clinical study`.
- **Task**: Specify the exact problem formulation:
  - `detection`: Retrospective identification of ictal events in ongoing EEG.
  - `prediction`: Raising a binary alarm in advance during the preictal period within defined SPH (Seizure Prediction Horizon) and SOP (Seizure Occurrence Period).
  - `forecasting`: Continuous probabilistic estimation of seizure likelihood across risk levels (e.g., low, moderate, high risk).
  - `multi-class`: Classification between multiple seizure types or distinct electrographic patterns.
  - `performance analysis`: Dissecting failure modes, error sources, or artifact impacts of an existing algorithm without introducing a new classifier.
  - `review`: Survey of state-of-the-art literature.

### Population & Clinical Setting
- **Subject Population**: `neonatal` (term or preterm), `paediatric`, `adult`, or `mixed`. Note age ranges and demographics.
- **Clinical Environment**:
  - `presurgical monitoring`: In-hospital video-EEG monitoring with anti-epileptic drug (AED) tapering/withdrawal and sleep deprivation (elevated seizure rates, hyper-excitable brain state).
  - `NICU`: Neonatal intensive care unit (cooling/hypothermia, high environmental and physiological artifacts, mechanical ventilation).
  - `ambulatory / out-of-hospital`: Wearable or portable monitoring during daily activities.
  - `ultra-long-term implanted`: Sub-scalp or intracranial electrodes recorded over months/years.

### Dataset & Recording Overview
- **Dataset Names**: Specify public datasets (e.g., CHB-MIT, EPILEPSIAE, TUSZ, BONN, FRE) or private in-house hospital cohorts.
- **Data Volume**:
  - Total duration: Total recording hours and days.
  - Total seizures count.
  - Partition counts: Number of training seizures vs. testing seizures; training hours vs. testing hours.
- **Montage & Referencing**:
  - Number of recording electrodes and channels.
  - Montage scheme: Bipolar (e.g., double-banana), referential (e.g., linked ears, common average), or mixed.
  - Channel Harmonisation: Whether a uniform channel subset was enforced or if channel counts varied across patients/folds (e.g., 16 to 23 channels depending on patient).
- **Dataset Version & Subject Inconsistencies**:
  - Note both stated counts and any textual or clinical inconsistencies (e.g., "Text states 24 cases, but case 24 is unknown/unpaired, representing 23 distinct individuals", or "mixing 22 and 24 patients across Sections 5 and 7").
- **Exclusions**:
  - Document all discarded records, patients, or channels and the explicit reasons given (e.g., excluded due to montage discrepancy, excessive NaN elements, absent seizure onset logs, or technical disconnects).

---

## 3. Section `# Data sumary`

*Applicability Rule*: If the paper is a pure performance analysis, clinical survey, or review that does not develop or evaluate a feature/classification pipeline, mark subsections as:  
`Not applicable (paper type: <paper_type> - reason why)` rather than forcing fabricated content.

### Data Preprocessing & Segmentation
- **Segment / Epoch Structure**:
  - Window length (e.g., 1 s, 5 s, 60 s).
  - Overlap / hop size (e.g., non-overlapping, 50% overlap, 1-second shift).
  - Label Assignment Logic: How window-level labels are determined (e.g., window inherits episode label if any seizure overlap exists, single 60s block extracted from midpoint of seizure, or boundary trimming). Note the implications for data leakage or metric bias.
- **Artifact Handling**:
  - State the exact artifact mitigation approach used (e.g., CNN-based deep learning artifact cleaner, independent component analysis (ICA), bandpass + energy thresholding, expert visual rejection).
  - *Absence Requirement*: If no artifact gating or cleaning is performed, explicitly write:  
    `None applied (the study does not employ an artifact-gating or artifact-removal module)`.
- **Filtering & Frequency Bands**:
  - High-pass, low-pass, notch, or bandpass specifications (e.g., 0.5–30 Hz, 50/60 Hz notch).
  - Explicit frequency bands analyzed (Delta, Theta, Alpha, Beta, Gamma sub-bands).
- **Missing Data & Anomaly Handling**:
  - Replacement methods for NaNs, zeros, or disconnected leads (e.g., mean interpolation over adjacent windows, zero-order hold, segment deletion).

### Feature Extraction & Dimensionality Reduction
- **Feature Categories**:
  - *Handcrafted Time-Domain*: Statistical moments (mean, variance, skewness, kurtosis), Hjörth parameters (activity, mobility, complexity), decorrelation time, zero-crossings.
  - *Handcrafted Frequency-Domain*: Power spectral density (PSD), relative subband powers, peak frequency, median frequency, spectral edge frequency/power, power ratios.
  - *Time-Frequency & Wavelet*: Discrete Wavelet Transform (DWT) detail subband energies, Empirical Mode Decomposition (EMD) / Intrinsic Mode Functions (IMF) statistics.
  - *Nonlinear & Dynamical*: Sample entropy, spectral entropy, approximate entropy, correlation dimension, Lyapunov exponents.
  - *Learned & Non-Handcrafted Representations*: Variational Autoencoder (VAE) latent space embeddings and reconstruction error, Common Spatial Patterns (CSP), raw time-series embeddings, learned Convolutional Neural Network (CNN) feature maps.
- **Feature Selection & Optimization**:
  - Ranking and selection methods (ANOVA $F$-test, Random Forest feature importance, Principal Component Analysis (PCA), Linear Discriminant Analysis backward/forward search (LDA-b/f), Gram-Schmidt orthogonalization).
  - Number of selected features ($k$) and whether $k$ was tuned globally or per-patient.

### Classification Methodology
- **Classification Paradigm**: Explicitly identify whether the setup is:
  - `patient-specific`: Models trained and evaluated strictly on separate seizures from the same patient.
  - `patient-independent`: Models trained on a cohort of subjects and tested on unseen subjects (e.g., Leave-One-Subject-Out / LOSO, Leave-One-Patient-Out / LOPO).
  - `mixed / pooled`: Data points pooled across patients into common folds without strict subject-level isolation.
- **Model Architecture & Deep Learning Specifications**:
  - Model family (e.g., Logistic Regression, SVM with linear/RBF kernel, Shallow Neural Network, CNN, Transformer, LSTM, Random Forest).
  - Deep learning hyperparameters: Number of layers, filter sizes, dropout rates, optimizer (e.g., Adam), learning rate, batch size, epoch budget, and early stopping patience.
- **Ensemble & Voting Mechanics**:
  - Single model vs. ensemble configurations (e.g., 15 SVMs or 15 SNNs).
  - Decision aggregation: Hard majority voting, soft probability averaging, or bagging across randomly undersampled subsets.
- **Class Balancing & Sampling Strategy**:
  - Document the exact balancing technique:
    - Inverse class weighting ($w_i = N_{\text{total}} / (2 \cdot N_i)$).
    - Systematic random undersampling of the majority class across time.
    - Synthetic Minority Oversampling Technique (SMOTE).
    - Balanced mini-batch sampling during stochastic gradient descent.
    - Negative-capping with a fixed ratio (e.g., 3:1 or 10:1 non-seizure to seizure ratio).
    - None (trained on natural raw imbalance).
- **Validation Scheme & Splitting**:
  - Cross-validation protocol: Leave-One-Subject-Out (LOSO/LOPO), patient-disjoint $k$-fold, chronological seizure-based split (e.g., first 3 seizures for training/tuning, subsequent seizures for testing), Leave-One-Seizure-Out (LOOCV), or random pooled holdout.
- **Post-Processing & Alarm Logic**:
  - Temporal smoothing techniques: Moving average filter, Firing Power thresholding, minimum run length (e.g., requiring 3 consecutive positive windows).
  - Alarm constraints: Seizure Prediction Horizon (SPH), Seizure Occurrence Period (SOP), refractory lockout duration where new alarms are inhibited.
  - Continuous risk thresholding: Cutoffs for low, moderate, and high risk (e.g., 0.3 and 0.7).
- **Operating Threshold Calibration**:
  - State explicitly where decision thresholds were selected: on training data, cross-validation folds, or whether they were evaluated directly on test sets.

### Methodological Validity & Leakage Audit
The agent must objectively audit potential data leakage and experimental rigor based strictly on textual evidence:
1. **Patient-Disjoint Separation**:
   - Were the training and test sets strictly patient-disjoint (no data from test patients present in training)?
   - Quote or paraphrase the exact textual evidence. If ambiguous or pooled, write `"Not stated"` or describe the pooled setup.
2. **Balancing & Oversampling Timing**:
   - Was balancing (e.g., SMOTE, random undersampling) applied *strictly inside each training fold after the split*, or applied to the entire dataset *before splitting*?
   - Quote evidence. If the text does not specify, write `"Not stated"`.
3. **Feature Selection & Hyperparameter Tuning Isolation**:
   - Were normalization (e.g., z-score mean/std), feature selection, and hyperparameter tuning fitted exclusively on training data, or did they touch test folds?
   - Quote evidence or write `"Not stated"`.
4. **Strict Objectivity Constraint**:
   - Do not speculate or assume author intentions. Report exactly what the text states or fails to state.

---

## 4. Section `# Conclusions`

### Quantitative Metrics & Performance
- **Metric Level (Mandatory Specification)**:
  - Every metric must be explicitly labeled by its evaluation level:
    - `window-level`: Evaluation per discrete time epoch (e.g., 1s, 5s window).
    - `episode-level`: Evaluation grouped per contiguous preictal or ictal segment.
    - `event-level`: Clinical seizure detection rate (seizure correctly flagged if alarm occurs within SOP or ictal boundary).
    - `patient-level`: Proportion of patients successfully validated over chance.
  - *Context Warning*: Note that window-level specificity (e.g., 99.6%) and episode-level accuracy (e.g., 91.6%) are not mathematically comparable. Note if all-case accuracy is inflated by seizure-free control patients.
- **Metric Vocabulary & Standard Mapping**:
  - First, record each metric under the **exact name and unit used in the paper** (e.g., SS, FPR/h, TiW, BS, BSS, IoC, FD/h, FAR/h, AP, SDR).
  - Second, map to standard terms only if definitions strictly align:
    - *Sensitivity / Recall / Seizure Detection Rate (SDR)*
    - *Specificity*
    - *False Positive Rate per hour (FPR/h or FAR/h)*
    - *Area Under the ROC Curve (AUC)*
    - *Detection Latency / Delay (seconds)*
- **Data Formatting, Units & Uncertainty**:
  - Report mean $\pm$ standard deviation where available.
  - State whether values are expressed as fractions ($0.0 - 1.0$) or percentages ($0\% - 100\%$).
  - Provide per-patient ranges (min–max) if the paper provides patient breakdowns.
  - Cite the exact data source (e.g., Table 1, Table 6, Section 6.1).
- **Baseline vs. Best-Performing Configurations**:
  - Compare baseline models against the final proposed architecture.
- **Paper's Internal Related-Work Comparison**:
  - If the authors include an explicit comparative table benchmarking against previous literature (e.g., Costa Tables 2–3, Tiwary Tables 13–14, Ghosh Table 1, Mathieson Table 14), summarize or reproduce the main comparative findings.
- **Computational Runtime & Experimental Extras**:
  - Inference latency per window or execution time per patient (e.g., 116.7 ms per window, 7.34 s per patient).
  - Statistical significance validation (e.g., surrogate time-series testing, one-sample $t$-test, Wilcoxon signed-rank test).
  - Key ablation findings.

### Key Takeaways & Contributions
- Primary clinical, mathematical, or algorithmic contributions demonstrated by the study.

### Limitations & Gaps (Strictly Separated)
Divide the limitations subsection into two separate, unambiguous parts:
1. **(a) Author-Stated Limitations**:
   - Limitations, biases, or constraints explicitly acknowledged by the authors in the text (e.g., presurgical AED withdrawal bias, lack of ultra-long-term validation, lack of sleep staging).
2. **(b) Methodological Limitations (Summarizer's Note)**:
   - Objective architectural or methodological constraints evident from the study design (e.g., absence of external cross-hospital validation, single-center data, fixed risk thresholds, small cohort size, lack of temporal continuity in segmented blocks, unverified SMOTE electrodynamics).
   - Must be prefixed with `[Summarizer's Note]`.
3. **Future Work**:
   - Subsequent directions and open research lines proposed by the authors.

---

## 5. Operational Rules & Extraction Robustness

1. **Never Guess or Fabricate**:
   - If a numerical count, channel montage, hyperparameter, or metric is missing, write `"Not reported"`. Never interpolate values from unlabeled plots.
2. **Handle Text-Extraction Damage & OCR Glitches**:
   - Source `.txt` files often suffer from broken font ligatures (e.g., "Scientifc" instead of "Scientific", "Te" instead of "The", missing "fi", "fl", "ff").
   - Restore missing characters cleanly in the summary. Do not reproduce corrupted or illegible strings.
   - For garbled mathematical equations, restore readable standard mathematical notation or LaTeX where unambiguous; otherwise write `"Equation garbled in source text"`.
3. **Supplementary Material Caveat**:
   - If the paper references external online supplementary materials (e.g., "Supplementary Tables S1–S4") that were not included in the source text, explicitly record:  
     `"Supplementary materials (e.g., Tables S1–S4) were not available in the extracted text."`
4. **Header Consistency**:
   - Maintain the exact top-level header names: `# General sumary`, `# Data sumary`, and `# Conclusions` so that downstream pipeline orchestrators can index them reliably.
5. **Execution Reporting & Diagnostics**:
   - At the conclusion of batch processing, log per-file diagnostics (e.g., `"BMRI2015-986736: Stated 22 subjects in Sec 5.1 vs 24 subjects in Sec 7"`, `"Mathieson 2016: Paper type performance analysis, classifier sections marked N/A"`).
