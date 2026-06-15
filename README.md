# 🧠 Gemma 4 E4B Compression for Edge Deployment

> **UNESCO Resilient AI Challenge 2026** — Image-to-Text Category  
> Team Visca Barca · Deep Learning MCTA 4363 · SEM 2 2025/2026

[![Challenge](https://img.shields.io/badge/Challenge-Resilient%20AI%202026-teal)]()
[![HuggingFace](https://img.shields.io/badge/🤗%20HuggingFace-Model%20Released-yellow)](https://huggingface.co/Defnotirf/gemma-4-E4B-hybrid-GGUF)
[![Best Result](https://img.shields.io/badge/VQA%20Accuracy-54%25-brightgreen)]()
[![Size](https://img.shields.io/badge/Model%20Size-5.88%20GB-blue)]()
[![Compression](https://img.shields.io/badge/Size%20Reduction-61%25-orange)]()

---

## 🎯 What We Did

Google's **Gemma 4 E4B** is a state-of-the-art multimodal vision-language model — but it requires **15 GB of VRAM** just to load, making it inaccessible on consumer hardware and energy-intensive in deployment.

We compressed it to **5.88 GB** while achieving **54% VQA accuracy — beating the unquantized baseline of 46%** — and submitted the result to the UNESCO Resilient AI Challenge 2026.

🤗 **Model on HuggingFace:** [Defnotirf/gemma-4-E4B-hybrid-GGUF](https://huggingface.co/Defnotirf/gemma-4-E4B-hybrid-GGUF)

---

## 🔍 The Core Discovery

We ran 8 systematic experiments across two compression families and found something unexpected:

> **Uniform quantization blinds the model. Importance-aware quantization restores its sight.**

| Method | VQA Accuracy | What happened |
|---|---|---|
| Unquantized baseline | 46% | Reference |
| NF4 4-bit BitsAndBytes | 10% ❌ | Model says *"no image provided"* |
| INT8 BitsAndBytes | 10% ❌ | Model says *"no clear human figure visible"* |
| Q4_K_M GGUF | 26% | Vision partially recovered |
| Q6_K GGUF | 32% | Better |
| Q8_0 GGUF | 34% | Good — but below baseline |
| Hybrid Q8+Q4_0 | 8% ❌ | Wrong quantization family for language |
| **Hybrid Q8_embd + Q4_K_M** | **54% ✅** | **Beats baseline** |

BitsAndBytes applies **uniform compression** to every layer equally — the vision encoder loses the precision it needs to extract spatial features from images. The model becomes effectively blind.

llama.cpp's GGUF format uses **importance-aware quantization** — it analyses weight activation magnitudes and protects high-sensitivity layers. The vision encoder survives. The model can see again.

---

## 🏗️ Hybrid Quantization Strategy

Our best model uses a selective precision approach:

```
Token Embeddings  →  Q8_0  (8-bit)   ← High precision, foundational
Output Norm       →  Q8_0  (8-bit)   ← Final layer, sensitivity preserved  
Attention Layers  →  Q4_K_M (4-bit)  ← Language reasoning, tolerates compression
FFN Layers        →  Q4_K_M (4-bit)  ← Pattern matching, robust to compression
Vision Projector  →  Separate Q8_0   ← Vision encoder fully preserved
```

This directly targets the failure mode we identified: protect what matters, compress what doesn't.

---

## 📊 Final Results

| Metric | Baseline | Our Best Model |
|---|---|---|
| VQA Accuracy | 46% | **54%** |
| Model Size | 15.0 GB | **5.88 GB** |
| Size Reduction | — | **61%** |
| Accuracy Retention | — | **117% of baseline** |
| Avg TTFT | 1.45s | 3.91s |
| CO₂/sample | 0.028g | 0.117g |

Evaluated on 50 stratified VQAv2 samples across three question types (yes/no, counting, open-ended) on NVIDIA RTX 5070 Ti 16 GB.

---

## 🧪 Experiment Log

| Exp | Method | VQA | Size | Status |
|---|---|---|---|---|
| 000 | Gemma 4 E4B unquantized | 46% | 15.0 GB | Baseline |
| 001 | NF4 4-bit BitsAndBytes | 10% | 9.3 GB | ❌ Vision destroyed |
| 002 | INT8 BitsAndBytes | 10% | 11.5 GB | ❌ Vision destroyed |
| 003 | Q4_K_M GGUF | 26% | 5.1 GB | Partial recovery |
| 004 | Q6_K GGUF | 32% | 5.9 GB | Better |
| 005 | Q8_0 GGUF | 34% | 7.6 GB | Good |
| 006 | Hybrid Q8+Q4_0 | 8% | 5.76 GB | ❌ Wrong language format |
| **007** | **Hybrid Q8_embd+Q4_K_M** | **54%** | **5.88 GB** | ✅ **Best — submitted** |

---

## 🚀 Run the Model

```bash
# Via HuggingFace (recommended)
llama-server -hf Defnotirf/gemma-4-E4B-hybrid-GGUF --n-gpu-layers 99

# Or manually
llama-server \
  -m gemma4-E4B-hybrid2.gguf \
  --mmproj gemma4-E4B-mmproj.gguf \
  --n-gpu-layers 99 \
  --port 8080
```

---

## 🔧 Tech Stack

- **Model:** Google Gemma 4 E4B (`google/gemma-4-E4B-it`)
- **Quantization:** llama.cpp GGUF (custom CUDA build for RTX 5070 Ti Blackwell sm_120a)
- **Inference:** llama-server / llama-mtmd-cli
- **Evaluation:** VQAv2 dataset, CodeCarbon energy tracking, fuzzy string scorer
- **Hardware:** NVIDIA RTX 5070 Ti 16 GB · Intel Core Ultra 7 265K · 125 GB RAM

---

## 📁 Repository Structure

```
MCTA4363-Resilient-AI/
├── Assessment 3/
│   ├── 000_gemma4_baseline.py      # Unquantized baseline evaluation
│   ├── 004.py                      # Q4_K_M GGUF experiment
│   ├── 005.py                      # Q6_K GGUF experiment
│   ├── 006.py                      # Q8_0 GGUF experiment
│   ├── 007.py                      # Hybrid Q8+Q4_0 (failed)
│   └── 008.py                      # Hybrid Q8+Q4_K_M (best)
├── models/                         # GGUF model files (gitignored)
├── Check/                          # Vision sanity check scripts
├── *_results.json                  # All experiment results
└── hybrid_tensor_types.txt         # Tensor quantization assignment file
```

---

## 🏆 Competition

**UNESCO Resilient AI Challenge 2026**  
Organised by: UNESCO · Governments of France & India · Sustainable AI Coalition · Google

- **Category:** Image-to-Text (Gemma 4 by Google)
- **Quality gate:** ≥80% of baseline accuracy (≥36.8%)
- **Our result:** 117% of baseline — qualifies
- **Evaluation:** NVIDIA L4 16GB via llama.cpp
- **Submission:** [Defnotirf/gemma-4-E4B-hybrid-GGUF](https://huggingface.co/Defnotirf/gemma-4-E4B-hybrid-GGUF)

---

## 📄 License

Base model license: [Gemma Terms of Use](https://ai.google.dev/gemma/terms)
