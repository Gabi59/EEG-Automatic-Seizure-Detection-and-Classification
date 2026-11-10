# Agent Rules: Research Open Problems Extraction

## Trigger & Objective
When invoked, the agent must inspect and read all `.txt` documents located in the `./text/` directory, analyze the scientific content of each paper, identify all stated and implied open research problems, and return a structured synthesis of the open challenges for each file.

---

## Processing Workflow

### 1. Document Discovery
- Scan the `./text/` directory for all files matching `*.txt`.
- Read and process each text file individually.
- If the `./text/` directory is missing or empty, notify the user and halt.

### 2. Open Problems Identification Taxonomy
For every paper, the agent must thoroughly search across all sections (especially *Introduction*, *Discussion*, *Results*, *Limitations*, and *Conclusions & Future Work*) to extract open problems across the following dimensions:

1. **Methodological & Algorithmic Limitations**:
   - Shortcomings of the chosen algorithms, models, or feature extraction methods.
   - Trade-offs between sensitivity, specificity, computational overhead, and detection delay.
   - Hyperparameter sensitivity and model complexity bottlenecks.

2. **Data & Validation Constraints**:
   - Constraints related to dataset size, patient cohort demographics, and class imbalance.
   - Artificiality of data segmentation (e.g., fixed 60-second segments vs. continuous long-term streaming).
   - Validity and potential bias of synthetic oversampling techniques (e.g., SMOTE) in biological signals.
   - Generalisability issues across different clinical environments and electrode setups.

3. **Detection vs. Prediction Gaps**:
   - Retrospective detection vs. early predictive forecasting (pre-ictal identification).
   - Granularity of classification (e.g., binary seizure detection vs. fine-grained multi-class seizure type classification).

4. **Clinical, Hardware & Operational Hurdles**:
   - Noise, electromyogenic/ocular artifacts, and sensor displacements in ambulatory/wearable devices.
   - Implementation challenges in real-time embedded hardware (low-power, edge computing).
   - Tele-EEG transmission security, communication bandwidth, and integration with hospital workflows (e.g., NHS clinical neurophysiology shortages).

5. **Explicit Author-Stated Future Work**:
   - Unresolved questions and subsequent research paths explicitly highlighted by the authors.

---

## Output Structure & Format

For each processed file, the agent must return:
- **Paper Title & File Reference**: Name of the source file and publication title.
- **Categorized Open Problems**: Bulleted analysis grouped by the taxonomy above.
- **Potential Research Opportunities**: Concrete ideas for how subsequent dissertation work or research projects can address these gaps.

