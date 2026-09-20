# Architecture Guide: AI Studio Context-Processing Worker

## 1. Problem Statement & Solution Overview

In autonomous AI coding environments like Google Antigravity, the primary engineering bottleneck is **context window bloat**. When debugging complex issues or auditing a repository across 20 to 30 source files, logs, and technical specifications:
- The lead agent's context window inflates to 100k–300k+ tokens.
- Attention degradation ("needle in a haystack" problem) causes missed details and subtle regressions.
- Session token quotas and operational costs spike rapidly.

BUBU solves this problem by decoupling **analytical research** from **architectural decision-making**. It deploys Google AI Studio's Gemini API as an out-of-band context processor. The worker digests large payloads outside the conversation, writes an exhaustive markdown report to disk, and returns a verified, compact JSON evidence summary to Antigravity.

---

## 2. End-to-End Orchestration Flow

```text
┌──────────────────────────────────────────────────────────┐
│                          USER                            │
└────────────────────────────┬─────────────────────────────┘
                             │ prompt
                             ▼
┌──────────────────────────────────────────────────────────┐
│                   ANTIGRAVITY (Lead)                     │
│  • High-level architectural reasoning                    │
│  • Inspects user requirements                            │
│  • Decides whether offloading is justified (Auto Mode)   │
└──────────────┬────────────────────────────┬──────────────┘
               │                            │
        [Small / Local]              [Heavy / Complex]
               │                            │
               ▼                            ▼
      Direct Conversation              BUBU WORKER
       Context Handling                     │
                                            ▼
                                ┌───────────────────────┐
                                │     Local Cache       │
                                └───────┬───────┬───────┘
                                        │       │
                                      [HIT]   [MISS]
                                        │       │
                                        ▼       ▼
                                     Instant  Gemini API
                                     Offline  (TLS 1.3)
                                     Serving    │
                                                ▼
                                      ┌───────────────────┐
                                      │ Evidence Verifier │
                                      └─────────┬─────────┘
                                                │
                                    ┌───────────┴───────────┐
                                    │                       │
                                [SUCCESS]               [FALLBACK]
                                    │                       │
                                    ▼                       ▼
                           Report to Disk           Fallback JSON
                           (.ai-worker/reports/)    (status: fallback)
                                    │                       │
                                    └───────────┬───────────┘
                                                │
                                                ▼
┌──────────────────────────────────────────────────────────┐
│                   ANTIGRAVITY (Lead)                     │
│  • Reviews verified compact evidence                     │
│  • Makes authoritative code decisions                    │
│  • Modifies source code & executes tests                 │
└──────────────────────────────────────────────────────────┘
```

---

## 3. Foundational Architectural Principles

### Principle 1: Strict Lead-Analyst Separation
- **Worker is an Analyst:** The worker inspects files, maps data flows, analyzes error traces, and produces grounded evidence. It has **read-only** permissions to project source files and writes strictly to the `.ai-worker/` directory.
- **Antigravity is the Lead Architect:** Antigravity retains sole authority to accept, modify, or reject recommendations and perform source code edits.

### Principle 2: Total Context Isolation
Gemini generates rich, exhaustive analytical reports (often 10 KB to 50+ KB). Streaming these reports into Antigravity's conversation context would defeat the purpose of offloading. Instead:
- The full markdown report is written directly to `.ai-worker/reports/<task_type>-<task_id>.md`.
- Only a minimal, structured JSON summary is emitted to `stdout`.
- Antigravity inspects the full report only if the compact summary lacks necessary nuance.

### Principle 3: Ground-Truth Evidence Verification
Models can hallucinate line numbers or code snippets. BUBU eliminates this by passing files with 4-digit line numbers (`0042 | fun process()`) and locally verifying every citation before reporting it:
- The worker matches cited line numbers and snippets against actual project files.
- Items are marked `verified: true` and `snippet_verified: true` only when an exact match is confirmed.

### Principle 4: Multi-Factor Composite Caching
To ensure zero redundant API costs, BUBU computes a 6-factor SHA-256 hash:
$$\text{Cache Key} = \text{SHA-256}(\text{Version} + \text{Model} + \text{TaskType} + \text{Prompt} + \text{SortedFileHashes} + \text{Config})$$
If target files and prompts are unchanged, the worker serves cached results in <0.5 seconds with zero API calls.

### Principle 5: Resilient Graceful Fallback
If the Gemini API encounters rate limits (`429`), quota exhaustion, network drops, or missing credentials, BUBU **never aborts the user's task**. It emits a structured JSON object with `status: "fallback"` and exit code `0`, allowing Antigravity to seamlessly continue using its local inspection tools (`view_file`, `grep_search`).
