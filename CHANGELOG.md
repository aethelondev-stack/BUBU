# Changelog

All notable changes to the **BUBU (AI Studio Context-Processing Worker)** project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.0.0] - 2026-09-21

### Initial Open-Source Release

#### Core Architecture & Worker Engine
- **Universal Context Offloading:** Dispatches large file collections and deep analytical tasks to Google AI Studio's Gemini API, preventing context window bloat in AI coding agents such as Antigravity.
- **Lead-Analyst Separation:** Enforces strict division between the Analyst (read-only research/evidence gathering via Gemini) and the Lead Architect (decision-making and code modification via Antigravity).
- **8 Analytical Task Types:** Full support for `DEBUG`, `ANALYZE`, `RESEARCH`, `AUDIT`, `COMPARE`, `SUMMARIZE`, `DOCUMENT`, and `VALIDATE`.
- **Zero Heavy Dependencies:** Implemented completely using Python's standard library (`urllib.request`, `json`, `hashlib`, `pathlib`).

#### Intelligent Modes & Decision Engine
- **Three Project Modes:** Configured per-project in `.ai-worker/config.json`:
  - `auto` (Default & Recommended): Dynamically evaluates whether offloading is justified based on file count, byte size, task type, and analytical keywords.
  - `enabled`: Worker is active for all analytical requests.
  - `disabled`: Worker is deactivated; all analysis stays in the primary agent context.
- **Auto Decision Engine (`--decide`):** Prevents unnecessary API calls for small single-file bug fixes, typos, or minor UI changes while routing multi-file audits and concurrency investigations to Gemini.
- **Single-Prompt Project Onboarding:** Inquires once about the preferred mode on unconfigured projects and persists the choice to disk.

#### Caching & Quota Management
- **6-Factor SHA-256 Cache:** Computes unique composite keys from `worker_version + model + type + prompt + sorted(file_hashes) + max_file_bytes`.
- **Instant Offline Cache Serving:** Re-runs of unchanged tasks return in sub-second time (<0.5s) with zero API network requests and zero token consumption.
- **Automatic Invalidation:** Modifying a single character in any target file triggers immediate cache eviction and re-analysis.
- **Local Quota Accounting:** Maintains a safety budget (`quota_tracker.json`) with configurable daily request limits and requests-per-minute (RPM) windows to prevent accidental billing or exhaustion.
- **Exponential Backoff:** Automatically retries transient `429` (rate limit) and `5xx` (server busy) responses with exponential delays.

#### Resilient Graceful Fallback
- **Zero Task Abortion:** If the Gemini API is unreachable, quota is exhausted, or the API key is missing, the worker produces a valid `status: "fallback"` JSON payload with exit code `0`.
- **Smooth Handover:** Empowers Antigravity to seamlessly continue analysis using local inspection tools (`view_file`, `grep_search`) without failing the user prompt.

#### Evidence Verification & Grounding
- **Line-Numbered Injection:** Injects source code with 4-digit line numbers (`0042 | code`), ensuring accurate line citations by the model.
- **Automated Verification:** Cross-checks model citations against actual disk contents, returning `evidence_verified: true` only when file names, line bounds, and code snippets match reality.

#### Security & Privacy Defenses
- **Strict Read-Only Guarantee:** Worker has zero write capability to project source code; write permissions are restricted exclusively to `.ai-worker/`.
- **Secret File Exclusion:** Automatically blocks `.env*`, `*.pem`, `*.key`, `id_rsa*`, `credentials*`, and keystore files from collection.
- **In-Memory API Key Masking:** Redacts API keys from stdout, stderr, logs, and generated reports (`[REDACTED_API_KEY]`).
- **Prompt Injection Isolation:** Separates instructions from untrusted source materials and enforces a strict structured JSON output schema.
- **Path Traversal Protection:** Validates that all file targets reside inside `PROJECT_ROOT`, blocking traversal attempts with exit code `7`.

#### Verification & Test Validation
- **Empirical Stress Testing:** Validated across a 31-file synthetic project (~116.8 KB source code, 2,571 lines) containing 8 deliberate architectural/logical flaws:
  - 10/10 findings successfully identified (8 targeted + 2 additional valid issues).
  - 10/10 evidence snippets verified against local disk files.
  - ~94.7% context-payload reduction proxy achieved (6,701 bytes compact output vs ~127 KB raw source + report).
  - ~82x faster execution on cache hits (0.42s vs 34.57s).
- **Independent Package Testing:** Passed a complete 30-point independent validation matrix (`BUBU_FINAL_TEST_PROJECT`) verifying portability, mode switching, offline caching, and security guards.
