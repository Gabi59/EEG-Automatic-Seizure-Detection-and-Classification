---
title: "In-depth performance analysis of an EEG based neonatal seizure detection algorithm"
authors:
  - "S. Mathieson"
  - "J. Rennie"
  - "V. Livingstone"
  - "A. Temko"
  - "E. Low"
  - "R. M. Pressler"
  - "G. B. Boylan"
year: 2016
paper_type: "performance analysis"
task: "performance analysis"
population: "neonatal"
clinical_setting: "NICU (Neonatal Intensive Care Unit with therapeutic hypothermia)"
datasets:
  - name: "London and Cork Neonatal Cohort"
    patients_stated: 20 # 10 with seizures, 10 seizure-free
    patients_inconsistency: "None"
    total_recording_hours: 1262.9 # Mean 63.1 hours per infant
    seizures_total: 421 # 419 confirmed by second clinical neurophysiologist (99.76% consensus)
    seizures_train: "Not applicable (evaluates pre-trained ANSeR alpha detector)"
    seizures_test: 421
montage:
  channels_count: 9
  channels_harmonisation: "Uniform modified neonatal 10:20 system (F4, F3, T4, T3, C4, C3, CZ, O2, O1)"
  referencing: "bipolar"
segmentation:
  window_length_seconds: 8.0 # ANSeR detector epoch length
  overlap_ratio_or_seconds: "overlapping epochs"
  labeling_rule: "Event-based detection (seizure designated if probability trace breeches sensitivity threshold on any channel)"
features:
  feature_type: "not applicable"
  feature_summary: "Not applicable (paper type: performance analysis - evaluates pre-existing 55 ANSeR features and scores 10 electrophysiological criteria per seizure)"
artifact_handling: "bandpass/thresholding only" # Evaluates impact of unrejected artifacts: respiration (34.7-64.6%), sweat (4.1-19.9%), movement, ECG/pulse, rhythmic quiet sleep
classifier:
  paradigm: "not applicable"
  model_family: "Pre-trained SVM (ANSeR alpha version); analyzed via mixed-effects logistic regression"
  ensemble_voting: "not applicable"
  class_balancing: "not applicable"
  post_processing: "Sigmoid probability conversion + moving average filter + adjustable threshold (0.4, 0.5, 0.6)"
leakage_audit:
  patient_disjoint: "yes" # Evaluated out-of-sample on 20 newly recorded neonates
  balancing_timing: "not applicable"
  tuning_isolation: "not applicable"
key_metrics:
  metric_level: "event-level"
  primary_results:
    - name: "Seizure Detection Rate (Threshold 0.4)"
      standard_name: "sensitivity"
      value: "60%"
      unit: "%"
      source: "Abstract & Section 3.2"
    - name: "False Detection Rate (Threshold 0.4)"
      standard_name: "FPR/h"
      value: "0.63"
      unit: "events/hour"
      source: "Section 3.2 & Table S1"
    - name: "Seizure Detection Rate (Threshold 0.5)"
      standard_name: "sensitivity"
      value: "54%"
      unit: "%"
      source: "Abstract & Section 3.2"
    - name: "False Detection Rate (Threshold 0.5)"
      standard_name: "FPR/h"
      value: "0.41"
      unit: "events/hour"
      source: "Section 3.2 & Table S1"
    - name: "Seizure Detection Rate (Threshold 0.6)"
      standard_name: "sensitivity"
      value: "45%"
      unit: "%"
      source: "Abstract & Section 3.2"
    - name: "False Detection Rate (Threshold 0.6)"
      standard_name: "FPR/h"
      value: "0.27"
      unit: "events/hour"
      source: "Section 3.2 & Table S1"
    - name: "Adaptive Beta Version AUC"
      standard_name: "AUC"
      value: "96.1% (vs. 93.4% alpha)"
      unit: "%"
      source: "Section 4 / Temko 2013"
    - name: "Adaptive Beta Version False Detection Rate"
      standard_name: "FPR/h"
      value: "0.24 FD/hr (vs. 0.42 FD/hr alpha)"
      unit: "events/hour"
      source: "Section 4 / Temko 2013"
---

# General sumary

### Paper Identification & Metadata
- **Title**: In-depth performance analysis of an EEG based neonatal seizure detection algorithm
- **Authors**: S. Mathieson, J. Rennie, V. Livingstone, A. Temko, E. Low, R. M. Pressler, and G. B. Boylan
- **Affiliations**: Academic Research Department of Neonatology (University College London), Irish Centre for Fetal and Neonatal Translational Research / INFANT (University College Cork), and Great Ormond Street Hospital, UK & Ireland
- **Publication Venue**: *Clinical Neurophysiology*, Vol. 127, Issue 5, pp. 2246–2256, 2016, http://dx.doi.org/10.1016/j.clinph.2016.01.026
- **Paper Type**: `performance analysis`
- **Task**: `performance analysis` (systematic electrophysiological dissection of detected vs. missed seizures and detailed clinical categorization of false alarms in an automated neonatal detector)

### Population & Clinical Setting
- **Subject Population**: Term neonates with high-risk neurological conditions:
  - 10 neonates with electrographic seizures (diagnoses: HIE grades 2–3, stroke, intraparenchymal haemorrhage, subdural haemorrhage, meconium aspiration syndrome, septic emboli).
  - 10 seizure-free control neonates (diagnoses: HIE grades 1–2, birth asphyxia, meconium aspiration syndrome).
- **Clinical Environment**: Neonatal Intensive Care Units (NICUs) at University College Hospital London and University College Hospital Cork (infants monitored under therapeutic hypothermia/cooling for 72 hours).

### Dataset & Volume Overview
- **Data Volume**:
  - Total duration: **1,262.9 hours** of multi-channel continuous video-EEG (mean 63.1 hours per infant).
  - Seizures: **421 electrographic seizures** annotated by an electroencephalographer and verified by a blinded consultant neurophysiologist (419/421 verified, 99.76% consensus).
- **Montage & Recording Equipment**:
  - Recorded with NicoletOne monitors (CareFusion) at sampling rate 250 or 256 Hz with 0.5–70 Hz filter bandwidth.
  - Modified neonatal 10:20 bipolar montage across 9 electrodes: F4, F3, T4, T3, C4, C3, CZ, O2, O1.

---

# Data sumary

### Data Preprocessing & Annotation Scoring
- **Algorithmic Baseline**:
  - The paper does not propose a new classifier. It evaluates the alpha version of the **ANSeR** (Algorithm for Neonatal Seizure Recognition) detector (Temko et al. 2011), which extracts 55 features from 8-second overlapping EEG segments and classifies them via Support Vector Machine (SVM) into a probability trace.
- **Ten Seizure Assessment Criteria**:
  - Prior to analysis, all 421 seizures were scored by an expert across 10 criteria:
    1. *Peak amplitude* ($\mu\text{V}^2$, peak-to-trough at midpoint).
    2. *Rhythmicity score* (1 = significant dysrhythmia, 2 = minimal, 3 = highly rhythmic).
    3. *Background EEG score* (normal, moderate, severe abnormality).
    4. *Morphology at onset* (rhythmic delta [RDD], theta [RDT], alpha [RDA], sharp waves [SH], spike-wave complexes [SP+W]).
    5. *Morphology at peak*.
    6. *Seizure duration* (seconds).
    7. *Frequency variability* (standard deviation of discharge frequency across start, peak, end in Hz).
    8. *Morphology change from start to peak* (binary Y/N).
    9. *Number of EEG channels at onset*.
    10. *Number of EEG channels at peak*.
- **False Detection Categorization**:
  - Every false detection was classified into 9 categories: Respiration artifact, ECG/pulse artifact, chewing/sucking, bad/loose electrode, patient movement/handling, electronic equipment, sweat artifact, unclassified artifact, or no artifact identified (subdivided into rhythmic background, normal background, sharp waves, low amplitude).

### Feature Extraction & Selection
- `Not applicable (paper type: performance analysis)`: The study investigates the statistical predictors of detection within pre-extracted ANSeR outputs rather than engineering a new feature set.

### Classification Methodology
- `Not applicable (paper type: performance analysis)`: No new machine learning architecture is proposed.
- **Statistical Modeling**:
  - Univariate and mixed-effects multivariate logistic regression (infant ID modeled as random effect) evaluated which of the 10 seizure criteria independently predict automated detection.
  - Collinearity between channels-at-onset and channels-at-peak was resolved by backward stepwise deletion.
- **Evaluated Sensitivity Thresholds**:
  - Threshold 0.4 (most sensitive), 0.5 (medium), and 0.6 (least sensitive).

### Methodological Validity & Leakage Audit
- **Patient-Disjoint Status**: Yes. Evaluated out-of-sample on 20 term neonates whose data was not used in training the original alpha ANSeR SVM.
- **Balancing Timing**: `Not applicable`.
- **Tuning Isolation**: `Not applicable`.

---

# Conclusions

### Quantitative Metrics & Performance
- **Seizure Detection vs. Threshold (Event-Level, Section 3.2)**:
  - *Threshold 0.4*: Detected **60%** of seizures; False Detection Rate = **0.63 FD/hr**.
  - *Threshold 0.5*: Detected **54%** of seizures; False Detection Rate = **0.41 FD/hr**.
  - *Threshold 0.6*: Detected **45%** of seizures; False Detection Rate = **0.27 FD/hr**.
- **Multivariate Predictors of Automated Detection (Table 3)**:
  - Four features were statistically significant independent predictors across all thresholds:
    1. *Peak Amplitude*: Odds ratio (OR) = **1.01 to 1.02 per $\mu\text{V}$** ($p < 0.001$).
    2. *Seizure Duration*: OR = **1.02 per second** ($p < 0.001$).
    3. *Rhythmicity*: Highly rhythmic seizures had OR = **3.03 to 4.96** compared to dysrhythmic seizures ($p \le 0.015$).
    4. *Number of Channels at Peak*: OR = **1.35 to 1.46 per channel** ($p \le 0.011$).
- **Quantification of False Detections (Table 4)**:
  - *Respiration Artifact*: Single largest cause across all operating points (**34.7%** at threshold 0.4, **47.9%** at 0.5, **64.6%** at 0.6) due to highly rhythmic chest wall movements mimicking seizure waveforms.
  - *Rhythmic Background EEG*: Accounted for **13.5% to 27.6%** of false detections (predominantly during quiet sleep/slow-wave sleep with rhythmic delta activity).
  - *Sweat Artifact*: Accounted for **4.1% to 19.9%** (drops sharply at higher thresholds).
  - *ECG / Pulse Artifact*: Accounted for **4.2% to 12.5%** (pulsatile vessels adjacent to electrodes).
- **Direct Impact on Algorithmic Improvement (The ANSeR Beta Version)**:
  - Observation that respiration, pulse, and sweat cause prolonged elevated baseline probabilities led to the **adaptive baseline correction** modification in the ANSeR Beta version.
  - Adaptive baseline increased ROC AUC from **93.4% to 96.1%** and reduced false alarms from **0.42 to 0.24 FD/hr** while maintaining 70% seizure burden capture (Temko et al. 2013).

### Key Takeaways & Contributions
- Provides the first systematic neurophysiological audit of automated neonatal seizure detection, demonstrating that missed seizures are predominantly brief (<30s), low amplitude, dysrhythmic, and focal (single channel).
- Reveals extreme patient clustering of false alarms: **83.5% of respiration false detections occurred in a single infant (Patient 2)**, and **69.8% of sweat false alarms occurred in Patient 15**, proving that aggregate population metrics mask extreme patient-level variation.

### Limitations & Gaps
- **(a) Author-Stated Limitations**:
  - 8-second analysis epochs in the alpha detector fail to capture multi-minute temporal evolution (e.g., transition from rhythmic delta to spike-and-wave).
  - Clinical alarms are subject to alarm fatigue in NICUs; acceptable false alarm rates can only be established in live bedside clinical trials.
- **(b) Methodological Limitations [Summarizer's Note]**:
  - Sample Size: 20 infants (10 with seizures) drawn from a single clinical trial cohort; patient clustering heavily biases aggregate artifact percentages.
  - Event Definition Asymmetry: A 5-minute generalized seizure detected for only 30 seconds on a single channel is counted as "detected," which inflates event sensitivity relative to actual duration-weighted seizure burden.
- **Future Work**:
  - Testing the beta version on unedited EEGs, assessing anticonvulsant medication impacts on seizure morphology, and implementing real-time adaptive patient thresholding.
