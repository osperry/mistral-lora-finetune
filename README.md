# Fine-Tuning Mistral 7B Locally — A Governed, Private AI Workflow

**By Oric Perry | CRISC | Principal Cybersecurity Consultant, Agentic AI | OSP Global Solutions**

A complete local fine-tuning pipeline for Mistral-7B-Instruct-v0.2 using Apple's MLX framework — with zero cloud dependency, zero data exposure, and a human-in-the-loop approval gate on every output.

---

## The Problem

Most LLMs default to verbose, over-qualified, AI-softened language. That's a liability when outputs are used in professional communications. Beyond tone, there's a governance problem: sending internal notes, communications, or organizational context to a third-party API is a data privacy exposure — one that most organizations haven't formally evaluated.

This project solves both.

---

## The Use Case

Administrative staff use this model to generate first-draft memos and documents from executive notes. Every output is reviewed and approved before it goes anywhere. The model handles the draft. The human handles the judgment.

This is what responsible AI deployment looks like at the workflow level:
- **Privacy**: no data leaves the machine during inference
- **Quality control**: human review gate before every output ships
- **Auditability**: the model's training data is known, versioned, and controlled

---

## Why This Architecture

After 25 years in enterprise cybersecurity — including workforce identity programs at Fortune 32 scale and contributing to IEC 63452 (Cyber-Security for ICT) and BSIGEL 9/6AIML (AI/ML Governance) standards — the governance model here is deliberate:

- **Local inference** eliminates third-party data processor exposure under GDPR/HIPAA frameworks
- **Fine-tuning on curated examples** produces deterministic style behavior more reliably than prompt engineering
- **LoRA adapters** keep the training footprint small and auditable
- **Human approval before output** is the control that makes this safe to scale

---

## Stack

| Tool | Purpose |
|---|---|
| MLX-LM | LoRA fine-tuning on Apple Silicon |
| Mistral-7B-Instruct-v0.2 | Base model |
| llama.cpp | GGUF conversion |
| Ollama | Local model serving |

---

## Requirements

- MacBook with Apple Silicon (M1/M2/M3/M4)
- Python 3.11
- Ollama (`brew install ollama`)
- llama.cpp (`brew install llama.cpp`)

---

## Dataset Format

Training data uses the standard JSONL chat format:

```json
{"messages": [{"role": "user", "content": "Your prompt"}, {"role": "assistant", "content": "Direct, brief response"}]}
```

Files:
- `data/train.jsonl` — 35 examples
- `data/valid.jsonl` — 11 examples
- `data/test.jsonl` — 11 examples

Training examples were pulled from real conversations reflecting the communication standard the model was trained to reproduce.

---

## Step 1: Install Dependencies

```bash
pip3.11 install mlx-lm
```

---

## Step 2: Train

```bash
mlx_lm.lora \
  --model mistralai/Mistral-7B-Instruct-v0.2 \
  --train \
  --data ./data \
  --batch-size 4 \
  --num-layers 16 \
  --iters 100
```

Adapter weights saved to `./adapters/`.

**Training results:**

| Checkpoint | Val Loss |
|---|---|
| Iter 1 | 2.968 |
| Iter 100 | 0.570 |

Peak memory: ~16.7 GB. Total training time: under 3 minutes.

---

## Step 3: Configure the Adapter

MLX does not auto-generate `adapter_config.json`. Create it manually using MLX's native format — not the PEFT/HuggingFace format. They are not interchangeable.

```bash
cat > adapters/adapter_config.json << 'EOF'
{
  "fine_tune_type": "lora",
  "num_layers": 16,
  "lora_parameters": {
    "rank": 8,
    "alpha": 16,
    "dropout": 0.05,
    "scale": 2.0,
    "keys": ["q_proj", "v_proj"]
  }
}
EOF
```

---

## Step 4: Fuse Adapter into Base Model

```bash
mlx_lm.fuse \
  --model mistralai/Mistral-7B-Instruct-v0.2 \
  --adapter-path ./adapters \
  --save-path ./fused-model
```

Output: full model in `./fused-model/` (~14.5 GB across 3 safetensors shards).

---

## Step 5: Convert to GGUF

The brew install of llama.cpp does not include the HuggingFace converter. Pull it directly:

```bash
pip3.11 install gguf torch --break-system-packages
curl -O https://raw.githubusercontent.com/ggerganov/llama.cpp/master/convert_hf_to_gguf.py

python3.11 convert_hf_to_gguf.py ./fused-model \
  --outfile ./fused-model/my-mistral.gguf \
  --outtype q8_0
```

Output: `my-mistral.gguf` (~7.7 GB).

---

## Step 6: Load into Ollama

```bash
echo "FROM $(pwd)/fused-model/my-mistral.gguf" > Modelfile
ollama create my-mistral -f Modelfile
ollama run my-mistral "Your test prompt here"
```

---

## Repo Structure

```
.
├── README.md
├── data/
│   ├── train.jsonl
│   ├── valid.jsonl
│   └── test.jsonl
└── adapters/
    └── adapter_config.json
```

> `.safetensors` and `.gguf` files are excluded due to size. Host the fused model on Hugging Face if distribution is needed.

---

## Known Failure Points

| Issue | Root Cause | Fix |
|---|---|---|
| `no Modelfile or safetensors files found` | Ollama needs absolute paths | Use `$(pwd)` in Modelfile |
| `AttributeError: num_layers` | MLX config format ≠ PEFT format | Write MLX-native `adapter_config.json` |
| `unsupported architecture MistralForCausalLM` | Ollama can't load raw HF safetensors | Convert to GGUF via `convert_hf_to_gguf.py` |
| `No module named torch` | Brew llama.cpp has no HF converter | Install torch + pull script from llama.cpp repo |

---

## Governance Notes

- 57 training examples is a minimal dataset. Sufficient to demonstrate style transfer; expand before production-scale deployment.
- Train loss (0.103) vs val loss (0.570) at iter 100 indicates some overfitting — expected at this dataset size.
- Human review before every output is not optional. It is the control.

---

## About

Oric Perry is a CRISC-certified cybersecurity executive with 25+ years in enterprise IAM, Zero Trust architecture, and AI governance. Contributing member of IEC 63452 (Cyber-Security for ICT) and BSIGEL 9/6AIML (AI/ML Governance for Cyber-Security). Principal Consultant at OSP Global Solutions.

- LinkedIn: [linkedin.com/in/oricperrycybergrc](https://linkedin.com/in/oricperrycybergrc)
- GitHub: [github.com/osperry](https://github.com/osperry)

---

## License

MIT
# mistral-lora-finetune
