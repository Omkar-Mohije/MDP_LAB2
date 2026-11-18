# MMDP Lab Exam 2 — Autonomous Solution

**Student Name:** Omkar Mohije  
**Roll Number:** `MAI2026006`  
**Personal Parameter ($R$):** $R = 06$ (last two digits of roll number)  
**STFT Window Size (`n_fft`):**  
$$\text{n\_fft} = 512 + 64 \times (R \bmod 8) = 512 + 64 \times (6 \bmod 8) = 512 + 384 = 896$$
$$\text{hop\_length} = \frac{\text{n\_fft}}{4} = 224$$

---

## 📌 Repository Deliverables

| File | Description |
| :--- | :--- |
| [`Lab_Exam_2_MAI2026006.ipynb`](./Lab_Exam_2_MAI2026006.ipynb) | Fully executed end-to-end Jupyter Notebook with all outputs & plots embedded |
| [`restored.png`](./restored.png) / [`restored_MAI2026006.png`](./restored_MAI2026006.png) | Inpainted image restoring the $512 \times 512$ watermarked image (`img_05.jpg`) |
| [`result.json`](./result.json) / [`result_MAI2026006.json`](./result_MAI2026006.json) | Structured submission payload conforming to evaluation specifications |
| [`execution_MAI2026006.log`](./execution_MAI2026006.log) | Complete execution log with timestamps and assertion verifications |
| [`spectrogram_comparison_MAI2026006.png`](./spectrogram_comparison_MAI2026006.png) | Before/after spectrograms using personal parameter $\text{n\_fft} = 896$ |
| [`vocalisation_events_MAI2026006.png`](./vocalisation_events_MAI2026006.png) | Temporal power envelope with segmented animal vocalisation events |
| [`watermark_scores_MAI2026006.png`](./watermark_scores_MAI2026006.png) | Per-image forensic anomaly scores across `image_pool/` |
| [`watermark_restoration_MAI2026006.png`](./watermark_restoration_MAI2026006.png) | $2 \times 2$ grid showing detection, crop, binary mask, and restored image |

---

## 🔬 Task Summaries & Verified Metrics

### Task 1: Audio Analysis Without a Pretrained Classifier
- **Stationary Interference Tones:** Exact sinusoidal peaks at `[80.0, 120.0, 5200.0, 6000.0]` Hz (100% temporal persistence, coefficient of variation $< 0.25$).
- **Noise Floor:** Broadband noise floor estimated at $-58.20\text{ dBFS}$ per STFT bin ($-31.02\text{ dBFS}$ broadband RMS).
- **Wiener Filtering & Spectral Subtraction:** Estimating stationary noise PSD eliminates both the 4 sinusoidal tones and background hiss simultaneously (no notch filter or hard gate). Estimated SNR improves from $-12.84\text{ dB}$ (raw) to $+11.42\text{ dB}$ (denoised) ($+24.26\text{ dB}$ boost).
- **Animal Vocalisations:** Segments 6 distinct multi-harmonic vocalisation events with an overall median pYIN $f_0 \approx 1839.29\text{ Hz}$ and median spectral centroid $\approx 2182.45\text{ Hz}$.
- **Animal Identification:** The high fundamental frequency, high spectral centroid ($1800\text{--}2600\text{ Hz}$), and multi-part crowing cadence uniquely match the **rooster** (*Gallus gallus domesticus*, Paragraph 1 of `knowledge_base.txt`), ruling out low-frequency cattle mooing, sheep bleating, dog barking, and cat purring/meowing.

### Task 2: Image Forensics & Watermark Removal
- **Forensic Detection:** 1D vertical morphological top-hat (`MORPH_RECT (1, 19)`) combined with horizontal gradient anisotropy (`|dx| >> |dy|`) detects the faint horizontal text line near the bottom of **`img_05.jpg`** (forensic anomaly score: $12.84$, compared to all other images $< 2.80$).
- **Localisation:** Watermark bounding box localised to `(x1, y1, x2, y2) = [18, 468, 229, 486]`, revealing the text `ROLL-05-HASH-50982`.
- **Inpainting:** A tight, dilated binary stroke mask inside the bounding box inpainted via Telea / Dirichlet Laplacian harmonic reconstruction restores the natural ground texture at exact $512 \times 512$ resolution.

### Task 3: Cross-Modal Text Retrieval
- **Unimodal Text Ranking:** Paragraph 1 achieves 1-based rank **`1`** across TF-IDF, BM25, and Sentence-Embedding (`all-MiniLM-L6-v2`).
- **Zero-Shot Multimodal Fusion:** Zero-shot CLIP (`openai/clip-vit-base-patch32` on `img_05.jpg`) and CLAP (`laion/clap-htsat-unfused` on denoised audio at $48\text{ kHz}$) fused with Reciprocal Rank Fusion (RRF) yield `fused_paragraph_index = 1` (0-based).
- **Extracted Vocalisations:** `["crowing"]` (from *"loud, high-pitched crowing vocalizations"*).
- **Cat Substring Trap:** A plain substring search (`"cat" in text.lower()`) falsely returns Paragraph 1 (Rooster) due to `"domesti-cat-ed"`, as well as matching `"Cattle"`, `"communicate"`, and `"location"` across Paragraphs 9, 10, and 11, instead of isolating Paragraph 3 (*Felis catus*).

---

## 📄 Submission JSON Payload (`result.json`)

```json
{
  "roll": "MAI2026006",
  "n_fft": 896,
  "tones_hz": [
    80.0,
    120.0,
    5200.0,
    6000.0
  ],
  "num_events": 6,
  "median_f0_hz": 1839.29,
  "watermarked_image": "img_05.jpg",
  "watermark_bbox": [
    18,
    468,
    229,
    486
  ],
  "paragraph_ranks": {
    "tfidf": 1,
    "bm25": 1,
    "embedding": 1
  },
  "fused_paragraph_index": 1,
  "vocalisations": [
    "crowing"
  ]
}
```
