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

## Step 1: Resolve Script Path & Check Availability

Resolve `LOCAL_LLM="<skill-dir>/scripts/local-llm"` from this `SKILL.md`'s directory so commands work from any project directory, then run:

```bash
"$LOCAL_LLM" --check
```

Exits `0` with installed and currently loaded models when reachable at `http://127.0.0.1:11434`, or exits `2` if unavailable. Defaults to `$LOCAL_LLM_MODEL`, or the first local non-embedding chat model (`*embed*` and `*-cloud` tags are skipped automatically so private files never route to cloud models unless `--allow-cloud` is passed). Requires `uv` and localhost access — inside restricted sandboxes (e.g. Codex, Cursor), retry with network/escalated permission before declaring Ollama unavailable.

## Common Usage

### Text prompt or piped stdin

```bash
"$LOCAL_LLM" --no-think "Classify this note into work, personal, or finance" < note.txt
```

### Deterministic JSON extraction

```bash
cat email.txt | "$LOCAL_LLM" \
  --no-think --temperature 0 --json \
  --system "Return JSON with keys: category, urgency (low|med|high), summary." \
  "Triage this email"
```

### Strict JSON Schema + local vision OCR

```bash
"$LOCAL_LLM" \
  --model qwen2.5vl \
  --no-think --temperature 0 \
  --schema '{"type":"object","properties":{"vendor":{"type":"string"},"total":{"type":"number"}},"required":["vendor","total"]}' \
  --image /path/to/receipt.jpg \
  "Extract vendor and total from this receipt"
```

### Plain image transcription

```bash
"$LOCAL_LLM" --model qwen2.5vl --image /path/to/screenshot.png "Transcribe all visible text verbatim."
```

## Key Flags & Behavior

- `--no-think` / `--think`: maps to `reasoning_effort` (`none` / `medium`) on `/v1` and `think` on Ollama `/api/chat`.
- `-m, --model <name>`: override model (`LOCAL_LLM_MODEL` env also supported; pass `--allow-cloud` only when intentionally targeting a `*-cloud` tag).
- `--stdin`: force reading piped stdin alongside `--image` (by default `--image` calls leave stdin untouched so `while read` batch loops work cleanly).
- `--num-ctx <int>`: override Ollama context window via native `/api/chat` (errors explicitly on non-Ollama `/v1` servers).
- `--raw`: print the full JSON API response (including usage and timing fields).
- `OLLAMA_HOST` / `--host`: override server URL (normalizes bare `127.0.0.1:11434` and `0.0.0.0`; works with LM Studio, vLLM, or llama.cpp `/v1` servers).
- **Structured-output fallback & validation**: retries HTTP `400`/`422`/`501` `response_format` rejections (such as Ollama MLX `501` or vLLM/LM Studio schema limits) with prompt-enforced JSON, rejects non-finite `NaN`/`Infinity`, and validates `type`, `required`, `properties`, `items`, and `enum` against `--schema`.
