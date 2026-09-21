# Worker Protocol & Output Contract (WORKER_PROTOCOL.md)

This document defines the CLI contract, supported task types, input formatting rules, provider selection flags, and the structured JSON output specification for `ai_worker.py`.

---

## 1. Supported Task Types (`--type`)

| Task Type | Core Focus | Expected Model Analysis |
| :--- | :--- | :--- |
| `DEBUG` | Exception analysis, stack traces, and logic failures | Concrete root cause (`root_cause`), precise error lines, and verified code snippets (`evidence`). |
| `ANALYZE` | Data flow, component coupling, and control flows | Architectural interactions, call chains, and module dependencies. |
| `RESEARCH` | Frameworks, external libraries, and API signatures | Interface capabilities, configuration nuances, and recommended design patterns. |
| `AUDIT` | Codebase compliance, security, performance, and memory audits | Anti-patterns, thread-safety hazards, memory leak risks, and remediation advice. |
| `COMPARE` | Structural comparison between two components or implementations | Trade-off matrices, behavioral divergences, and parity evaluations. |
| `SUMMARIZE` | Broad architectural summaries across multi-file domains | High-level module responsibilities, domain boundaries, and subsystem overviews. |
| `DOCUMENT` | Technical documentation and contract drafting | Markdown specifications, API documentation drafts, and inline contract descriptions. |
| `VALIDATE` | Pre/post change regression and impact analysis | Contract breakage risks, downstream impact radius, and regression vulnerabilities. |

---

## 2. Line-Numbered Source Code Injection

Target files are injected into LLM providers with 4-digit zero-padded line numbers rather than raw text:

```text
==================== FILE: src/core/PlaybackEngine.kt ====================
0001 | package com.app.core
0002 | import kotlinx.coroutines.flow.Flow
...
0042 | fun startStream(streamUrl: String) {
0043 |     if (streamUrl.isBlank()) throw IllegalArgumentException("URL is blank")
0044 | }
```

### Why This Format Matters:
1. **Eliminates Line-Number Hallucinations:** The LLM references the exact line prefix (e.g. `0043`) displayed on the left margin.
2. **Deterministic Evidence Verification:** `ai_worker.py` parses the model's reported line number and code snippet, comparing it against the local disk array to ensure verified accuracy.

---

## 3. Structured Output Contract

The worker prints **strictly valid JSON** to `stdout`. Diagnostic logs, progress indicators, and status warnings are routed exclusively to `sys.stderr`.

### Success Response (`status: "success"`)
```json
{
  "task_id": "debug-20260921-a1b2c3",
  "status": "success",
  "task_type": "DEBUG",
  "provider": "gemini",
  "model": "gemini-3.6-flash",
  "summary": "Division by zero occurs when denominator evaluates to 0 in math utility.",
  "root_cause": "SmallBug.kt:39 performs integer division without guarding against b == 0.",
  "findings": [
    "Unchecked division in divide(a, b) method.",
    "No exception handler wraps invocation in caller service."
  ],
  "evidence": [
    {
      "file": "src/SmallBug.kt",
      "line": 39,
      "snippet": "return a / b // Missing zero check",
      "note": "ArithmeticException triggered when b is zero",
      "verified": true,
      "snippet_verified": true
    }
  ],
  "recommendations": [
    "Add require(b != 0) { 'Division by zero' } check at src/SmallBug.kt:39."
  ],
  "do_not_change": [
    "src/Utils.kt"
  ],
  "confidence": "HIGH",
  "evidence_verified": true,
  "report_path": ".ai-worker/reports/debug-20260921-a1b2c3.md",
  "cache_hit": false
}
```

### Graceful Fallback Response (`status: "fallback"`)
Emitted when an LLM provider is unreachable, quota is exhausted, or credentials are unconfigured. The process exits with code `0`:
```json
{
  "task_id": "audit-20260921-b4c5d6",
  "status": "fallback",
  "task_type": "AUDIT",
  "fallback_reason": "QUOTA_EXHAUSTED",
  "summary": "gemini provider request failed (QUOTA_EXHAUSTED). Falling back to local Antigravity inspection.",
  "findings": [],
  "evidence": [],
  "recommendations": [
    "Antigravity will inspect targeted files directly using local read tools."
  ],
  "do_not_change": [],
  "confidence": "UNKNOWN",
  "evidence_verified": false,
  "report_path": null,
  "cache_hit": false
}
```

### Skipped Task Response (`status: "skipped"`)
Emitted when the worker mode is `disabled` or Auto Mode determines that the task is localized:
```json
{
  "task_id": "analyze-20260921-c7d8e9",
  "status": "skipped",
  "task_type": "ANALYZE",
  "auto_decision": "SKIP",
  "reason": "Task is localized (1 file(s), 420 bytes). Direct Antigravity inspection in conversation context is recommended.",
  "summary": "Auto-mode skipped worker: Task is localized.",
  "findings": [],
  "evidence": [],
  "recommendations": [
    "Antigravity can inspect the targeted file(s) directly."
  ],
  "do_not_change": [],
  "confidence": "UNKNOWN",
  "evidence_verified": false,
  "report_path": null,
  "cache_hit": false
}
```

---

## 4. CLI Arguments & Selection Options

```powershell
python .agents/skills/ai-studio-worker/scripts/ai_worker.py `
  --type <TASK_TYPE> `
  --prompt "<instruction>" `
  [--provider <gemini|openai_compatible>] `
  [--model <model_name>] `
  [--files <path...>] `
  [--glob "<pattern>"] `
  [--path <directory>] `
  [--decide] `
  [--dry-run] `
  [--force] `
  [--strict]
```

- `--provider <gemini|openai_compatible>`: Selects the active LLM provider (default: `gemini`).
- `--model <name>`: Model override (e.g. `gemini-3.6-flash`, `gpt-4o-mini`, `deepseek-chat`).
- `--files <path...>`: Explicit list of source files to analyze.
- `--glob "<pattern>"`: Wildcard matcher (e.g. `"src/**/*.kt"`).
- `--path <directory>`: Entire directory tree (excluding binary and secret files).
- `--decide`: Runs the Auto Mode decision engine only (prints `USE` or `SKIP` without making an API call).
- `--dry-run`: Prepares the payload, calculates byte budgets, and tests file parsing without invoking the API.
- `--force`: Bypasses the local cache to execute a fresh API request.
- `--strict`: In strict mode, network or credential errors return non-zero exit codes rather than graceful fallback JSON.

---

## 5. Process Exit Codes

| Exit Code | Name | Description |
| :---: | :--- | :--- |
| `0` | `EXIT_SUCCESS` | Successful analysis, cache hit, auto-mode skip, or graceful fallback. |
| `1` | `EXIT_GENERAL_ERROR` | Unhandled runtime exception or internal failure. |
| `2` | `EXIT_INVALID_ARGS` | Missing required parameters (e.g. no `--prompt` provided). |
| `3` | `EXIT_AUTH_ERROR` | API key is missing or invalid (`--strict` mode only). |
| `4` | `EXIT_QUOTA_EXHAUSTED` | Daily request budget exceeded (`--strict` mode only). |
| `5` | `EXIT_RATE_LIMITED` | RPM safety budget exceeded (`--strict` mode only). |
| `6` | `EXIT_VALIDATION_ERROR` | File size or total context payload exceeds limits (`--strict` mode only). |
| `7` | `EXIT_PATH_ERROR` | Path traversal detected (attempting to access files outside project root). |
