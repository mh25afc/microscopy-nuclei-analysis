# Hybrid Pipeline for Nuclei Microscopy Image Analysis
 A hybrid biomedical image-analysis pipeline that combines a local vision-language model (VLM), classical image processing, and a U-Net segmentation network, run end-to-end on a fluorescence microscopy nuclei-segmentation dataset.

Pipeline: **raw image → segmentation → quantitative region features → structured JSON record → short narrative**

Full analysis, figures, and discussion are in report.

---

## Repository structure

```
.
├── README.md
├── requirements.txt
├── Assignment3_Report.pdf          # final report
├── notebook/
│   └── assignment_3.ipynb  # full end-to-end Colab notebook
├── outputs/
│   ├── eda_samples.png
│   ├── eda_histogram.png
│   ├── task2_pipeline.png
│   ├── task3_training_curves.png
│   ├── task3_predictions.png
│   ├── robustness_visual.png
│   ├── task4_results.csv
│   └── robustness_comparison.csv

```

## Dataset

[`Nickolay-K/Assingnment-3-dataset`](https://github.com/Nickolay-K/Assingnment-3-dataset) fluorescence microscopy nuclei images, 256×256 RGB, with paired binary masks.

| Split | Images |
|---|---|
| train | 80 |
| val | 20 |
| test | 12 |
| test_corrupted | 4 (blur / low-contrast variants, used for the robustness extension) |

## Model substitution note

The assignment specifies `llama3.2-vision`. This model's `mllama` architecture was not supported by the Ollama runtime in any of three independently tested environments (Google Colab via install script, Google Colab via direct GitHub release, and a local Windows CPU install) all raised `unknown model architecture: 'mllama'`. **`llava` was substituted** for all VLM steps to keep the pipeline fully local and reproducible. This is discussed in the report (Section 1 and Question 5).

## How to run

Designed to run top-to-bottom in **Google Colab** (GPU runtime recommended for Task 3 training).

1. **Clone this repository**
   ```bash
   git clone https://github.com/mh25afc/microscopy-nuclei-analysis.git
   cd microscopy-nuclei-analysis
   ```

2. **Clone the dataset**
   ```bash
   git clone https://github.com/Nickolay-K/Assingnment-3-dataset.git
   ```

3. **Install Ollama and pull the VLM**
   ```bash
   curl -fsSL https://ollama.com/install.sh | sh
   ollama serve &
   pip install -q ollama
   ollama pull llava
   ```

4. **Install Python dependencies**
   ```bash
   pip install -r requirements.txt
   ```

5. **Open and run** `notebook/assignment3_pipeline.ipynb` top to bottom (upload it to Colab, or open directly via `File → Open notebook → GitHub` and paste this repo's URL). Cells are ordered by task:
   - **Task 1** : grayscale/resize, EDA, naive vs. optimised VLM prompt, non-determinism check
   - **Task 2** : Otsu threshold + morphology + `regionprops_table`, numbers-first LLM interpretation
   - **Task 3** : U-Net definition, training loop (30 epochs, BCE+Dice loss), Dice/IoU evaluation, prediction visualisation
   - **Task 4** : full hybrid pipeline on the test set, aggregated to `task4_results.csv`
   - **Extension** :robustness test on `test_corrupted/` images, `robustness_comparison.csv`

6. Set `Runtime → Change runtime type → T4 GPU` before running Task 3/4 for reasonable training/inference time.

## Reproducing report figures/numbers exactly

All figures and tables in the report are saved directly from the notebook (`plt.savefig(...)` calls before each `plt.show()`), and the two result tables are exported as CSV (`task4_results.csv`, `robustness_comparison.csv`). Re-running the notebook end-to-end will regenerate all of these; due to LLM sampling non-determinism (see Task 1), exact VLM wording may vary slightly between runs, but the U-Net metrics and classical-feature numbers are deterministic given the same trained weights.

