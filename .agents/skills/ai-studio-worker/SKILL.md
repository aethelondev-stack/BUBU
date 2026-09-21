---
name: ai-studio-worker
description: >-
  Universal Provider-Agnostic LLM Context-Processing Worker for Antigravity.
  Offloads heavy file reading, complex debugging, cross-file comparisons,
  and extensive codebase research to high-capacity LLM APIs (Google Gemini,
  OpenAI, DeepSeek, OpenRouter, Ollama), preserving Antigravity's main
  conversation context.
---

# Universal LLM Context-Processing Worker (BUBU)

This skill provides an external context-processing service for any project.
It runs as a local, dependency-free Python worker that routes large file sets, logs, and research
prompts to configured LLM providers (Google Gemini as Provider #1, or any OpenAI-compatible HTTP API as Provider #2), saving verbose reports to disk and returning
only compact, evidence-backed JSON to Antigravity.

---

## 1. Core Principle

> **Worker analyzes and produces evidence. Antigravity decides and changes code.**

The worker is strictly a read-only analyst. It will never modify project source code.
Antigravity reviews the structured findings, verifies the evidence, and performs the actual code edits.

---

## 2. Worker Usage Modes & Provider Selection

The system supports three project-level modes configured in `.ai-worker/config.json`:

1. **`auto` (Default & Recommended):**
   - The decision engine evaluates task complexity, file counts, and byte budgets dynamically.
   - Large or multi-file analytical tasks are offloaded to the active LLM provider.
   - Small, localized micro-edits bypass the worker and are inspected directly by Antigravity.
2. **`enabled`:**
   - Worker is active for all analytical requests where invoked.
3. **`disabled`:**
   - Worker is completely deactivated. All file inspections are performed directly by Antigravity.

### LLM Providers
- **`gemini` (Default Provider #1):** Google AI Studio Gemini API (Free tier available).
- **`openai_compatible` (Provider #2):** Standard HTTP API compatible with OpenAI, DeepSeek, OpenRouter, Qwen, Ollama, and local proxies.

### Querying or Changing Mode
```powershell
# Query current status & provider
python .agents/skills/ai-studio-worker/scripts/ai_worker.py --status

# Query current mode
python .agents/skills/ai-studio-worker/scripts/ai_worker.py --get-mode

# Set mode
python .agents/skills/ai-studio-worker/scripts/ai_worker.py --set-mode <enabled|disabled|auto>
```

### Initial Project Setup Prompt
If `.ai-worker/config.json` does not exist in a new project, prompt the user once:
```text
BUBU (LLM Context Worker) bu projede kullanılabilir.
Tercihiniz:
1. Worker kullan (enabled)
2. Worker kullanma (disabled)
3. Göreve göre otomatik karar ver (auto - Önerilen)
```
Save the selection via `--set-mode` so subsequent sessions never prompt again.

---

## 3. Auto Mode Decision Engine

Before sending code to an external LLM, evaluate whether offloading is justified:

```powershell
python .agents/skills/ai-studio-worker/scripts/ai_worker.py `
  --type <TASK_TYPE> `
  --prompt "<instruction>" `
  --files <file1> <file2> ... `
  --decide
```

### Decision Criteria:
- **Offload to Worker (`USE`):**
  - Multi-file analysis (3+ files or globs).
  - Heavy file payload (≥15 KB source bytes).
  - Broad task types: `AUDIT`, `COMPARE`, `RESEARCH`, `VALIDATE`.
  - Deep investigations: cross-module dependencies, race conditions, memory leaks, security audits.
- **Direct Antigravity Inspection (`SKIP`):**
  - Single-file edits, typos, localized bug fixes.
  - Simple refactors or renames where context is already in conversation.
  - Fast iterative UI/logic tweaks.

---

## 4. How to Invoke

Run the worker from PowerShell via `run_command`:

```powershell
python .agents/skills/ai-studio-worker/scripts/ai_worker.py `
  --type <TASK_TYPE> `
  --prompt "<Clear, specific analytical instruction>" `
  --files <file1> <file2> ...
```

### Provider & Model Overrides
```powershell
# Default (Gemini)
python .agents/skills/ai-studio-worker/scripts/ai_worker.py --type AUDIT --prompt "..." --files ...

# Explicit OpenAI-compatible provider
python .agents/skills/ai-studio-worker/scripts/ai_worker.py --provider openai_compatible --model gpt-4o-mini --type AUDIT --prompt "..." --files ...
```

### Supported Task Types (`--type`)
- `DEBUG`: Stack trace & root cause analysis with line citations.
- `ANALYZE`: Data flow, architectural flow, and component interaction.
- `RESEARCH`: Library or API investigation (local knowledge base).
- `AUDIT`: Verification against security, performance, or pattern criteria.
- `COMPARE`: Structural comparison between implementations.
- `SUMMARIZE`: Architectural overview across files.
- `DOCUMENT`: Technical documentation drafting.
- `VALIDATE`: Pre/post change impact analysis.

### Selection Options
- `--files <path...>`: Specific files to read with line numbers.
- `--glob "<pattern>"`: Glob pattern (e.g. `"src/**/*.kt"`).
- `--path <directory>`: Entire directory tree (excluding build artifacts).
- `--provider <name>`: Override provider (`gemini`, `openai_compatible`).
- `--model <name>`: Override model.
- `--dry-run`: Test file selection and byte budget without calling the API.
- `--force`: Bypass cache and re-run analysis.

---

## 5. Interpreting the Structured Output

The worker prints a compact machine-readable JSON to `stdout`:

```json
{
  "task_id": "audit-20260921-a1b2c3",
  "status": "success",
  "task_type": "AUDIT",
  "provider": "gemini",
  "model": "gemini-3.6-flash",
  "summary": "High-level summary of findings.",
  "root_cause": "Exact root cause description if applicable.",
  "findings": ["Point 1", "Point 2"],
  "evidence": [
    {
      "file": "src/core/Engine.kt",
      "line": 42,
      "snippet": "while (attempt <= maxRetries) {",
      "note": "Counter is not incremented",
      "verified": true,
      "snippet_verified": true
    }
  ],
  "recommendations": ["Increment attempt counter inside retry loop."],
  "do_not_change": ["src/utils/Crypto.kt"],
  "confidence": "HIGH",
  "evidence_verified": true,
  "report_path": ".ai-worker/reports/audit-audit-20260921-a1b2c3.md",
  "cache_hit": false
}
```

### ⚠️ Context Protection Rule
**Do NOT automatically open or view `report_path`.**
Use the `summary`, `root_cause`, `evidence`, and `recommendations` directly to make your code edits. Only inspect the file at `report_path` if the structured result has insufficient detail.

---

## 6. Resilient Fallback

If the worker returns `status: "fallback"` or `status: "skipped"`:
```json
{
  "status": "fallback",
  "fallback_reason": "API_AUTH_ERROR",
  "summary": "LLM provider request failed (API_AUTH_ERROR). Falling back to local Antigravity inspection."
}
```
1. **Do NOT halt the user's task.**
2. Acknowledge fallback briefly (`Worker fallback triggered; continuing with local analysis`).
3. Fall back immediately to direct Antigravity inspection (reading targeted files via `view_file` or `grep_search`).
