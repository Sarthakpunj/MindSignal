<p align="center">
  <h1 align="center">MindSignal</h1>
  <p align="center">
    <strong>AI-Powered Depression Screening from Video Interviews</strong>
  </p>
  <p align="center">
    <em>Multimodal behavioural biomarker extraction + classification proof-of-concept</em>
  </p>
  <p align="center">
    <a href="#quick-start">Quick Start</a> •
    <a href="#how-it-works">How It Works</a> •
    <a href="#results">Results</a> •
    <a href="#notebooks">Notebooks</a> •
    <a href="#references">References</a>
  </p>
</p>

---

## What is MindSignal?

MindSignal detects vocal and facial biomarkers of depression from video interviews — **without requiring any patient self-report**. Current clinical screening relies on the PHQ-9 questionnaire, which has a fundamental flaw: depression itself impairs a patient's ability to accurately report their own symptoms [1]. MindSignal bypasses this by extracting objective, measurable signals directly from speech and facial behaviour.

This repository contains two Google Colab notebooks that together form the proof-of-concept:

| Notebook | Purpose | What it does |
|----------|---------|--------------|
| **`multimodal_pipeline.ipynb`** | Feature Extraction Pipeline | Extracts audio, visual, and deep speech features from any video interview |
| **`ravdess_classifier.ipynb`** | Trained Classifier + Demo | Trains a depression signal classifier on RAVDESS and deploys a Gradio web app |

> **Note:** This is a UCL MSc research proof-of-concept, not a clinical diagnostic tool. See [Ethical Considerations](#ethical-considerations).

---

## Quick Start

Both notebooks run entirely in **Google Colab** — no local setup needed.

### Notebook 1 — Multimodal Pipeline

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/)

```
1. Open 1_multimodal_pipeline.ipynb in Colab
2. Run Cell 1 → Restart Runtime
3. Run all remaining cells
4. Outputs saved to Google Drive: /DepressionDetection/TestRun/outputs/
```

### Notebook 2 — RAVDESS Classifier + Gradio Demo

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/)

```
1. Open 2_ravdess_classifier.ipynb in Colab
2. Run Cell 1 → Restart Runtime
3. Run Cells 2–5 to download data, extract features, train, and evaluate
4. Run Cell 6 to launch the Gradio demo (generates a public share link)
```

---

## How It Works

MindSignal analyses three parallel streams of behavioural data:

```
                        ┌──────────────────────┐
                        │   Audio Stream        │
                        │   librosa / YIN pitch │──────┐
                        │   52 acoustic features│      │
┌──────────────┐        └──────────────────────┘      │    ┌────────────────────────┐
│              │        ┌──────────────────────┐      ├───→│ Feature Concatenation   │
│  Clinical    │───────→│   Visual Stream       │──────┤    │         +               │
│  Video       │        │   dlib 68-pt landmarks│      │    │ Random Forest Classifier│
│  Interview   │        │   Facial action units │      │    │ (300 trees, balanced)   │
│              │        └──────────────────────┘      │    └───────────┬────────────┘
└──────────────┘        ┌──────────────────────┐      │                │
                        │   Deep AI Stream      │──────┘        ┌──────┴──────┐
                        │   Wav2Vec2-base-960h  │               │  Risk Score │
                        │   768-dim embeddings  │               │  PHQ-8 Est. │
                        └──────────────────────┘               │  Explainer  │
                                                               └─────────────┘
```

### Audio Features (52 total)
- **MFCCs** (13 coefficients × mean/std = 26 features) — vocal tract shape over time
- **Pitch** (YIN algorithm) — mean, std, range, voiced fraction
- **Energy** — RMS mean/std/range, pause count, silence fraction
- **Spectral** — centroid, rolloff, bandwidth, zero-crossing rate
- **Chroma** — harmonic content, mean and std

### Visual Features (dlib 68-point landmarks)
- **Eye Aspect Ratio (EAR)** — reduced in depression (fatigue, low affect)
- **Mouth openness** — reduced in depression (flat affect)
- **Brow furrow height** — lowered brows linked to depression
- **Head pitch/yaw** — downward gaze linked to depression

### Deep Speech Representations
- **Wav2Vec2-base-960h** — 768-dimensional embeddings per 10-second segment
- Captures paralinguistic patterns invisible to handcrafted features
- PCA-reduced for downstream classification

---

## Results

Trained on **RAVDESS** [2] — 1,440 emotional speech recordings from 24 actors.

| Metric | Value |
|--------|-------|
| **Test Accuracy** | 84.9% |
| **F1 Score** | 0.772 |
| **5-Fold CV Accuracy** | 84.9% ± 2% |
| **ROC-AUC** | Reported in notebook |
| **Dataset Size** | 1,440 samples |

**Label mapping:** Sad / Neutral / Calm → Depressive vocal signal · Happy / Angry / Fearful / Disgusted / Surprised → Non-depressive signal. This mapping is consistent with clinical literature on flat affect as a biomarker of depression [3][4].

---

## Repository Structure

```
MindSignal/
├── README.md                        ← You are here
├── LICENSE
├── .gitignore
├── multimodal_pipeline.ipynb  ← Full extraction pipeline (audio + visual + Wav2Vec2)
│── ravdess_classifier.ipynb   ← Train classifier + Gradio demo app
```

---

## Notebooks

### `multimodal_pipeline.ipynb` — Feature Extraction Pipeline

Demonstrates the full multimodal extraction pipeline on a sample video:

| Cell | What it does |
|------|-------------|
| 1 | Install dependencies (dlib, librosa, Wav2Vec2, OpenCV) |
| 2 | Mount Google Drive |
| 3 | Download dlib 68-point landmark model |
| 4 | Download sample video via yt-dlp |
| 5 | Trim + re-encode to H.264 (OpenCV compatibility fix) |
| 6 | Import libraries + verify setup |
| 7 | Load audio, plot waveform + mel spectrogram |
| 8 | Extract 52 handcrafted audio features |
| 9 | Plot audio features over time (pitch, energy, pauses, MFCCs) |
| 10 | Extract Wav2Vec2 deep embeddings (768-dim per segment) |
| 11 | Visualise embeddings (heatmap + PCA) |
| 12 | Test dlib face detection on single frame |
| 13 | Process full video — extract facial features per frame |
| 14 | Show sample annotated frames |
| 15 | Plot facial features over time (EAR, mouth, brow, head pose) |
| 16 | Aggregate to interview-level statistics |

**Outputs:** 5 visualisation plots + 4 feature files saved to Google Drive.

### `ravdess_classifier.ipynb` — Classifier + Gradio Demo

Downloads RAVDESS, trains a classifier, and launches a web demo:

| Cell | What it does |
|------|-------------|
| 1 | Install dependencies |
| 2 | Download RAVDESS from Zenodo (~200MB, 1,440 WAV files) |
| 3 | Parse emotion labels + extract 52 audio features per file |
| 4 | Train Random Forest (300 trees, balanced weights), 5-fold CV |
| 5 | Visualise: confusion matrix, feature importances, CV scores |
| 6 | Launch Gradio demo — upload any audio/video → get risk score |

**Gradio demo features:**
- Accepts audio (WAV, MP3) or video (MP4, MOV) uploads
- Real-time risk gauge with severity indicator
- Estimated PHQ-8 equivalent score
- Ranked feature importance explanation
- Built-in disclaimer and clinical context

---

## Clinical Basis

Each feature stream maps to established biomarkers of depression:

| Signal | Clinical Evidence |
|--------|------------------|
| Pitch monotony + reduced vocal energy | Psychomotor retardation in MDD [3] |
| Increased pause frequency | Slowed speech processing [4] |
| Reduced Eye Aspect Ratio | Fatigue and low affect [5] |
| Lowered brow position | Facial action unit changes in depression [5] |
| Deep speech embedding shifts | Paralinguistic feature changes [6] |

---

## Limitations

- **RAVDESS uses acted speech** — not clinically depressed individuals. Clinical validation requires the DAIC-WOZ dataset [7] (189 real clinical interviews with PHQ-8 labels).
- **Actors are predominantly North American** — cross-cultural validation is needed before any deployment.
- **Emotion-to-depression mapping is a proxy** — while consistent with the literature, it is not equivalent to clinical diagnosis.

---

## Ethical Considerations

- MindSignal is a **research proof-of-concept**, not a clinical diagnostic tool.
- Risk scores are designed for **clinician review only** — never shown to patients directly.
- The system is positioned as **decision-support**, not a replacement for clinical judgement.
- Video and audio data should be processed in real time and discarded — no storage.
- Bias testing across gender, age, and ethnicity is required before any deployment.

---

## Tech Stack

| Component | Library | Version |
|-----------|---------|---------|
| Audio feature extraction | librosa | 0.10+ |
| Pitch tracking | librosa YIN | — |
| Face detection + landmarks | dlib | 19.24+ |
| Deep speech embeddings | Wav2Vec2 (HuggingFace) | facebook/wav2vec2-base-960h |
| Classification | scikit-learn (Random Forest) | 1.3+ |
| Video processing | OpenCV | 4.8+ |
| Demo interface | Gradio | 4.0+ |
| Compute | Google Colab | GPU (T4) recommended |

---

## References

[1] M. G. Hunt et al., "Self-Report Bias and Underreporting of Depression on the BDI-II," *J. Personality Assessment*, vol. 80, no. 1, pp. 26–30, 2003.

[2] S. R. Livingstone and F. A. Russo, "The Ryerson Audio-Visual Database of Emotional Speech and Song (RAVDESS)," *PLoS ONE*, vol. 13, no. 5, e0196391, 2018. [doi:10.1371/journal.pone.0196391](https://doi.org/10.1371/journal.pone.0196391)

[3] C. Sobin and H. A. Sackeim, "Psychomotor symptoms of depression," *Am. J. Psychiatry*, vol. 154, no. 1, pp. 4–17, 1997.

[4] J. C. Mundt et al., "Vocal Acoustic Biomarkers of Depression Severity and Treatment Response," *Biological Psychiatry*, vol. 72, no. 7, pp. 580–587, 2012.

[5] J. F. Cohn et al., "Detecting Depression from Facial Actions and Vocal Prosody," in *Proc. ACII*, 2009.

[6] A. Baevski et al., "wav2vec 2.0: A Framework for Self-Supervised Learning of Speech Representations," *NeurIPS*, 2020.

[7] J. Gratch et al., "The Distress Analysis Interview Corpus (DAIC-WOZ)," in *Proc. LREC*, 2014.

---

## License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.

---

<p align="center">
  <strong>MindSignal</strong> — UCL MSc Applied AI · COMP0039 Technology Entrepreneurship · March 2026
</p>
