---
license: gemma
language:
- en
tags:
- gemma
- gguf
- quantization
- vision-language
- multimodal
- image-to-text
---

# Gemma 4 E4B Hybrid GGUF — Resilient AI Challenge 2026

**Team:** Visca Barca  
**Category:** Image-to-Text  
**Base model:** google/gemma-4-E4B-it  
**Challenge:** UNESCO Resilient AI Challenge 2026

---

## Compression Method

Hybrid GGUF quantization via llama.cpp, with selective precision assignment targeting the vision-language boundary:

| Component | Quantization | Rationale |
|---|---|---|
| Token embeddings | Q8_0 (8-bit) | Foundational representations — high precision required |
| Output norm | Q8_0 (8-bit) | Final layer normalisation — sensitive to precision loss |
| Attention layers (all blocks) | Q4_K_M (4-bit K-quant) | Language reasoning tolerates importance-aware 4-bit compression |
| FFN layers (all blocks) | Q4_K_M (4-bit K-quant) | Pattern matching layers — robust to compression |
| Vision projector (mmproj) | Separate Q8_0 file | Vision encoder kept at full GGUF precision |

**Key finding:** Uniform quantization (BitsAndBytes NF4/INT8) destroys the vision encoder in Gemma 4 E4B, collapsing VQA accuracy to 10%. Importance-aware GGUF quantization with selective precision assignment recovers and exceeds baseline accuracy.

---

## Results

| Metric | Baseline (unquantized) | This Model |
|---|---|---|
| VQA Accuracy | 46% | 54% |
| Model Size | 15.0 GB | 5.88 GB |
| Size Reduction | — | 61% |
| Accuracy Retention | — | 117% of baseline |
| Avg TTFT | 1.45s | 3.91s |
| CO₂/sample | 0.028g | 0.117g |

Evaluated on 50 stratified VQAv2 samples (15 yes/no, 15 counting, 20 open-ended) on NVIDIA RTX 5070 Ti 16GB.

---

## Files

| File | Description |
|---|---|
| `gemma4-E4B-hybrid2.gguf` | Main model weights — hybrid Q8/Q4_K_M quantization |
| `gemma4-E4B-mmproj.gguf` | Vision projector — multimodal image encoder |
| `llama_config.yaml` | llama-server inference configuration |

---

## Inference

```bash
llama-server -hf Defnotirf/gemma-4-E4B-hybrid-GGUF --n-gpu-layers 99
```

Or with explicit paths:

```bash
llama-server \
  -m gemma4-E4B-hybrid2.gguf \
  --mmproj gemma4-E4B-mmproj.gguf \
  --n-gpu-layers 99 \
  --port 8080
```

---

## Generation Parameters

```yaml
temperature: 1.0
top_p: 0.95
top_k: 64
n_predict: 600
```

---

## Known Behaviour

- Model uses Gemma 4 thinking template — the model reasons before answering
- Allocate sufficient tokens (600+) to allow thinking block to complete before the answer
- Both GGUF files must be present: main model and mmproj vision projector

---

## License

Released under the same license as the original model:  
[Gemma Terms of Use](https://ai.google.dev/gemma/terms)