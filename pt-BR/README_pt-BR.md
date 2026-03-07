# Fine-Tuning do Mistral 7B Localmente — Um Fluxo de Trabalho de IA Privado e Governado

**Por Oric Perry | CRISC | Consultor Principal de Cibersegurança, IA Agêntica | OSP Global Solutions**

Um pipeline completo de fine-tuning local para o Mistral-7B-Instruct-v0.2 usando o framework MLX da Apple — sem dependência de nuvem, sem exposição de dados e com uma porta de aprovação humana em cada saída.

---

## O Problema

A maioria dos LLMs responde com linguagem verbosa, cheia de ressalvas e suavizada por IA. Isso é um risco quando as saídas são usadas em comunicações profissionais. Além do tom, há um problema de governança: enviar notas internas, comunicações ou contexto organizacional para uma API de terceiros é uma exposição de privacidade de dados — uma que a maioria das organizações ainda não avaliou formalmente.

Este projeto resolve os dois.

---

## O Caso de Uso

A equipe administrativa usa esse modelo pra gerar primeiras versões de memorandos e documentos a partir de notas executivas. Toda saída é revisada e aprovada antes de ir pra qualquer lugar. O modelo cuida do rascunho. O humano cuida do julgamento.

É assim que um deployment responsável de IA parece na prática:
- **Privacidade**: nenhum dado sai da máquina durante a inferência
- **Controle de qualidade**: revisão humana antes de qualquer saída
- **Auditabilidade**: os dados de treinamento são conhecidos, versionados e controlados

---

## Por Que Esta Arquitetura

Depois de 25 anos em cibersegurança corporativa — incluindo programas de identidade de força de trabalho em escala Fortune 32 e contribuições para as normas IEC 63452 (Cibersegurança para TIC) e BSIGEL 9/6AIML (Governança de IA/ML) — o modelo de governança aqui é deliberado:

- **Inferência local** elimina a exposição a processadores de dados de terceiros sob os frameworks GDPR/HIPAA
- **Fine-tuning em exemplos curados** produz comportamento de estilo mais confiável do que engenharia de prompt
- **Adaptadores LoRA** mantêm o footprint de treinamento pequeno e auditável
- **Aprovação humana antes da saída** é o controle que torna isso seguro pra escalar

---

## Stack

| Ferramenta | Finalidade |
|---|---|
| MLX-LM | Fine-tuning LoRA no Apple Silicon |
| Mistral-7B-Instruct-v0.2 | Modelo base |
| llama.cpp | Conversão para GGUF |
| Ollama | Serviço local do modelo |

---

## Requisitos

- MacBook com Apple Silicon (M1/M2/M3/M4)
- Python 3.11
- Ollama (`brew install ollama`)
- llama.cpp (`brew install llama.cpp`)

---

## Formato do Dataset

Os dados de treinamento usam o formato padrão JSONL de chat:

```json
{"messages": [{"role": "user", "content": "Sua pergunta"}, {"role": "assistant", "content": "Resposta direta e objetiva"}]}
```

Arquivos:
- `data/train.jsonl` — 35 exemplos
- `data/valid.jsonl` — 11 exemplos
- `data/test.jsonl` — 11 exemplos

Os exemplos de treinamento foram extraídos de conversas reais que refletem o padrão de comunicação que o modelo foi treinado pra reproduzir.

---

## Passo 1: Instalar Dependências

```bash
pip3.11 install mlx-lm
```

---

## Passo 2: Treinar

```bash
mlx_lm.lora \
  --model mistralai/Mistral-7B-Instruct-v0.2 \
  --train \
  --data ./data \
  --batch-size 4 \
  --num-layers 16 \
  --iters 100
```

Os pesos do adaptador são salvos em `./adapters/`.

**Resultados do treinamento:**

| Checkpoint | Val Loss |
|---|---|
| Iter 1 | 2.968 |
| Iter 100 | 0.570 |

Pico de memória: ~16,7 GB. Tempo total de treinamento: menos de 3 minutos.

---

## Passo 3: Configurar o Adaptador

O MLX não gera o `adapter_config.json` automaticamente. Crie manualmente usando o formato nativo do MLX — não o formato PEFT/HuggingFace. Eles não são intercambiáveis.

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

## Passo 4: Fundir o Adaptador ao Modelo Base

```bash
mlx_lm.fuse \
  --model mistralai/Mistral-7B-Instruct-v0.2 \
  --adapter-path ./adapters \
  --save-path ./fused-model
```

Saída: modelo completo em `./fused-model/` (~14,5 GB em 3 shards safetensors).

---

## Passo 5: Converter para GGUF

A instalação via brew do llama.cpp não inclui o conversor HuggingFace. Baixe diretamente:

```bash
pip3.11 install gguf torch --break-system-packages
curl -O https://raw.githubusercontent.com/ggerganov/llama.cpp/master/convert_hf_to_gguf.py

python3.11 convert_hf_to_gguf.py ./fused-model \
  --outfile ./fused-model/my-mistral.gguf \
  --outtype q8_0
```

Saída: `my-mistral.gguf` (~7,7 GB).

---

## Passo 6: Carregar no Ollama

```bash
echo "FROM $(pwd)/fused-model/my-mistral.gguf" > Modelfile
ollama create my-mistral -f Modelfile
ollama run my-mistral "Seu prompt de teste aqui"
```

---

## Estrutura do Repositório

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

> Os arquivos `.safetensors` e `.gguf` são excluídos por tamanho. Hospede o modelo fundido no Hugging Face se precisar distribuir.

---

## Pontos de Falha Conhecidos

| Problema | Causa Raiz | Solução |
|---|---|---|
| `no Modelfile or safetensors files found` | Ollama precisa de caminhos absolutos | Use `$(pwd)` no Modelfile |
| `AttributeError: num_layers` | Formato MLX ≠ formato PEFT | Escreva o `adapter_config.json` no formato nativo do MLX |
| `unsupported architecture MistralForCausalLM` | Ollama não carrega safetensors HF diretamente | Converta para GGUF via `convert_hf_to_gguf.py` |
| `No module named torch` | Brew llama.cpp não tem o conversor HF | Instale torch e baixe o script do repositório llama.cpp |

---

## Notas de Governança

- 57 exemplos de treinamento é um dataset mínimo. Suficiente pra demonstrar a transferência de estilo; expanda antes de um deployment em produção.
- A diferença entre train loss (0,103) e val loss (0,570) na iter 100 indica algum overfitting — esperado com esse tamanho de dataset.
- Revisão humana antes de cada saída não é opcional. É o controle.

---

## Sobre

Oric Perry é um executivo de cibersegurança certificado CRISC com mais de 25 anos em IAM corporativo, arquitetura Zero Trust e governança de IA. Membro contribuinte das normas IEC 63452 (Cibersegurança para TIC) e BSIGEL 9/6AIML (Governança de IA/ML para Cibersegurança). Consultor Principal na OSP Global Solutions.

- LinkedIn: [linkedin.com/in/oricperrycybergrc](https://linkedin.com/in/oricperrycybergrc)
- GitHub: [github.com/osperry](https://github.com/osperry)

---

## Licença

MIT
