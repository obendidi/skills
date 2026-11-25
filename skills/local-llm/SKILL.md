---
name: local-llm
description: Run local Ollama or OpenAI-compatible CLI tasks for private OCR/vision, bulk classification, and strict JSON extraction at zero API cost. Use when offloading repetitive extraction, triage, or private document/receipt/invoice/screenshot OCR to a local model via Ollama or running free batch JSON extraction; not for interactive coding or over 32k context reasoning.
---

# Local LLM CLI (`ollama`)

Run fast, zero-cost, private local inference through Ollama (or any OpenAI-compatible local server) using the bundled PEP 723 `uv` script (`scripts/local-llm` inside this skill's directory).

## When to Use (and When Not To)

- **Use for**:
  - Local **vision / OCR** on private receipts, invoices, rendered PDFs, or screenshots without sending files to cloud APIs.
  - High-volume **$0 background classification, tagging, triage, or JSON extraction** in scripts and batch jobs.
  - Unlimited free local testing of structured-extraction prompts.
- **Do NOT use for**:
  - Interactive coding, multi-file refactoring, or long-context reasoning (>32k tokens) — use your primary coding model instead.

## Step 1: Check Availability

Before relying on a local model, resolve `scripts/local-llm` relative to this `SKILL.md` directory and run:

```bash
./scripts/local-llm --check
```

Exits `0` with installed and currently loaded models when reachable at `http://127.0.0.1:11434`, or exits `2` if unavailable. Defaults to `$LOCAL_LLM_MODEL`, or the first installed model on the server.

## Common Usage

### Text prompt or piped stdin

```bash
./scripts/local-llm --no-think "Classify this note into work, personal, or finance" < note.txt
```

### Deterministic JSON extraction

```bash
cat email.txt | ./scripts/local-llm \
  --no-think --temperature 0 --json \
  --system "Return JSON with keys: category, urgency (low|med|high), summary." \
  "Triage this email"
```

### Strict JSON Schema + local vision OCR

```bash
./scripts/local-llm \
  --model qwen2.5vl \
  --no-think --temperature 0 \
  --schema '{"type":"object","properties":{"vendor":{"type":"string"},"total":{"type":"number"}},"required":["vendor","total"]}' \
  --image /path/to/receipt.jpg \
  "Extract vendor and total from this receipt"
```

### Plain image transcription

```bash
./scripts/local-llm --model qwen2.5vl --image /path/to/screenshot.png "Transcribe all visible text verbatim."
```

## Key Flags & Behavior

- `--no-think` / `--think`: disable thinking for fast classification/extraction, or enable it for harder prompts.
- `-m, --model <name>`: override model (e.g., `qwen2.5vl`, `gemma3`, `llama3.2-vision`, or set `LOCAL_LLM_MODEL`).
- `--num-ctx <int>`: override Ollama context window size.
- `--raw`: print the full OpenAI-compatible JSON response (including usage and timing fields).
- `OLLAMA_HOST` / `--host`: override server URL (default `http://127.0.0.1:11434`; works with LM Studio, vLLM, or llama.cpp `/v1` servers too).
- **MLX structured-output fallback**: Ollama's MLX runner can return HTTP 501 for grammar-constrained `response_format`; `--json` and `--schema` automatically retry with prompt-enforced JSON and validate stdout with `json.loads` so output is still guaranteed valid JSON.
