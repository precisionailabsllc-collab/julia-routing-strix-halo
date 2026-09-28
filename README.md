---
license: apache-2.0
base_model: SupersonicLabs/Julia-1
library_name: julia
pipeline_tag: text-classification
tags:
  - julia
  - strix-halo
  - bounded-decisions
  - routing
  - cpu
  - gpu
---

# Julia Routing Strix Halo

Fine-tuned Julia-1 checkpoint for fast bounded-decision routing on AMD Strix Halo-class local machines.

Runtime status:

- CPU: tested
- GPU: expected to work through the Julia/PyTorch runtime, not separately benchmarked for this release
- NPU: not validated yet; AMD/XDNA NPU packaging and execution are future work

This model is intended for **advisory routing and gating metadata**, not free-form generation and not safety authority.

## Intended use

Good candidates:

- task routing over fixed options
- eval-case routing: category and deterministic grader type
- documentation-update triage
- context-management metadata
- non-authoritative tool-policy hints

Do **not** use this model as the final authority for:

- destructive command approval
- credential/secret handling
- production deploy approval
- security replay safety
- durable memory writes
- conversation summarization or final prose

Deterministic policy and the main agent must remain authoritative for those cases.

## Training summary

- Base model: `SupersonicLabs/Julia-1`
- Fine-tuning method: native partial fine-tune, no LoRA
- Trainable parameters: `7,634,563`
- Unfrozen modules: decision head plus last 2 encoder layers
- Learning rate: `1e-5`
- Epochs: `1`
- Training material: synthetic/adversarial bounded-decision examples for local agent routing and gating

## Local eval summary

Qualification suite:

| Model | Score |
|---|---:|
| Laya baseline | `34/51` |
| GLiNER2.5-Decide baseline | `42/51` |
| Julia fine-tuned | `51/51` |

Separate holdout suite:

| Model | Score | Accuracy | Mean CPU latency |
|---|---:|---:|---:|
| Julia fine-tuned | `98/126` | `77.8%` | `0.083s` |
| GLiNER2.5-Decide | `93/126` | `73.8%` | `0.320s` |
| Laya | `76/126` | `60.3%` | `0.340s` |

The holdout result is the better estimate of generalization. The qualification score is useful but should be treated as a tuned benchmark.

All latency numbers above were measured on CPU. NPU acceleration has not been validated for this checkpoint.

## Usage

Install Julia runtime from the base model package, then load this checkpoint. CPU is the validated path for this release:

```python
from julia import load_model

engine = load_model(
    "./julia-routing-strix-halo",
    device="cpu",
    strict_encoding=True,
    max_length=1024,
    head_length=512,
)

result = engine.predict(
    state={"request": "Fix the failing TypeScript test and run the suite."},
    questions={
        "route": {
            "type": "choice",
            "instructions": "Which worker route should handle this request?",
            "criteria": {
                "code_edit": "modify repository files",
                "browser_qa": "interact with a web UI",
                "research": "gather information only",
                "chat": "answer conversationally",
            },
        }
    },
)
print(result)
```

## Files

- `model.safetensors` — fine-tuned Julia decision weights
- `julia_config.json` — Julia runtime configuration
- `encoder/config.json` — encoder configuration
- `tokenizer/` — tokenizer files

## Limitations

- The model is specialized for bounded decisions and fixed option sets.
- It is not a general-purpose embedding model.
- It is not a generative model.
- It can be confidently wrong outside the routing distributions represented in training.
- Use confidence thresholds, deterministic guards, and fallback to a main model for ambiguous cases.
- NPU execution has not been validated yet. Do not advertise this as an NPU-ready checkpoint until an XDNA/NPU runtime path is tested.
