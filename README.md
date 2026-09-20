# BUBU: AI Studio Context-Processing Worker

> **High-Performance, Context-Preserving Research & Analytical Worker for Google Antigravity & Google AI Studio (Gemini API).**

BUBU is a source-available, modular companion for autonomous AI coding agents like [Google Antigravity](https://antigravity.google/). It solves the critical bottleneck of **context window bloat** by offloading heavy multi-file reading, stack-trace debugging, security audits, and library research to Google AI Studio's Gemini API, returning only compact, evidence-verified structured findings to the lead coding agent.

---

## 📑 Table of Contents

- [Quick Start](#-quick-start)
- [🤖 Install with an AI Coding Agent](#-install-with-an-ai-coding-agent)
- [🤖 For AI Coding Agents](#-for-ai-coding-agents)
- [Why BUBU?](#-why-bubu)
- [What BUBU is NOT](#-what-bubu-is-not)
- [Architecture & The Three Layers](#-architecture--the-three-layers)
- [Worker Usage Modes](#-worker-usage-modes)
- [Auto Mode Decision Engine](#-auto-mode-decision-engine)
- [Getting a Gemini API Key](#-getting-a-gemini-api-key)
- [API Key Security & Privacy](#-api-key-security--privacy)
- [Understanding Free API Limits & Tokens](#-understanding-free-api-limits--tokens)
- [Multi-Factor Caching Engine](#-multi-factor-caching-engine)
- [Local Quota Accounting vs. Server Quota](#-local-quota-accounting-vs-server-quota)
- [Resilient Graceful Fallback](#-resilient-graceful-fallback)
- [Empirical Validation & Stress Test Evidence](#-empirical-validation--stress-test-evidence)
- [Privacy and Data Flow](#-privacy-and-data-flow)
- [License & Commercial Terms](#-license--commercial-terms)
- [Frequently Asked Questions (FAQ)](#-frequently-asked-questions-faq)
- [Documentation Map](#-documentation-map)
- [Release Checklist](#-release-checklist)
- [Official References](#-official-references)

---

## ⚡ Quick Start

Get up and running in under 2 minutes:

1. **Download / Copy BUBU into your project:**
   Copy `.agents/`, `.ai-worker/`, and `studio/` into your target repository root.
2. **Obtain a Gemini API key:**
   Create a free key at [Google AI Studio](https://aistudio.google.com/app/apikey).
3. **Configure your key locally:**
   Copy `.env.example` to `.env` and paste your key:
   ```env
   GEMINI_API_KEY=your_actual_gemini_api_key_here
   ```
4. **Open your project in Antigravity (or your preferred agent):**
   Antigravity automatically discovers `.agents/skills/ai-studio-worker`.
5. **Run the 1-Prompt Setup:**
   Paste the [AI Coding Agent Prompt](#-install-with-an-ai-coding-agent) into your conversation once.
6. **Start coding normally:**
   Prompt your agent with normal engineering tasks. BUBU engages automatically when needed.

---

## 🤖 Install with an AI Coding Agent

If you are using Google Antigravity or another AI pair-programming assistant, copy and paste this universal setup prompt into your chat:

```text
Install and verify BUBU for this project.

First, discover the BUBU skill (.agents/skills/ai-studio-worker) and inspect the project configuration.

If my Worker usage preference is not yet saved in .ai-worker/config.json, present me with these choices once:
1. enabled
2. disabled
3. auto (Recommended)

If my preference is already saved, do not ask again.

Do NOT request, reveal, or output my real API key in the chat.
Verify that the installation and worker status are functional before modifying any code.
```

---

## 🤖 For AI Coding Agents

> **Notice for LLMs and AI Agents Reading This Repository:**

When you are operating in a project equipped with BUBU, follow these operational directives:

1. **Role Division:** You (the lead agent) are the **Lead Architect**. BUBU is your **Analyst**.
   - The worker is **strictly read-only**. It will never edit project source files directly.
   - The worker gathers evidence and diagnoses root causes; **you** decide what code to write and apply the diffs.
2. **Locating BUBU:**
   - Skill definition: [`.agents/skills/ai-studio-worker/SKILL.md`](.agents/skills/ai-studio-worker/SKILL.md)
   - Core script: [`.agents/skills/ai-studio-worker/scripts/ai_worker.py`](.agents/skills/ai-studio-worker/scripts/ai_worker.py)
   - Configuration file: `.ai-worker/config.json`
3. **Secret Hygiene:**
   - **Never read, output, or prompt for `.env` contents.**
   - Do not print API keys to stdout, stderr, or chat transcripts.
4. **Context Isolation Rule:**
   - The worker saves detailed markdown reports to `.ai-worker/reports/<task_id>.md`.
   - The worker emits a compact JSON summary to `stdout`.
   - **Do NOT automatically read or display full report files in chat** unless the compact summary lacks necessary technical detail.
5. **Evidence Verification:**
   - When the worker returns `evidence_verified: true`, the file paths, line numbers, and snippets have been programmatically matched against local disk files.
6. **Graceful Fallback Handling:**
   - If the worker returns `status: "fallback"` (e.g. quota exhausted or network error), **do NOT halt the user's task**.
   - Acknowledge the fallback briefly and inspect the targeted files directly using your built-in read tools (`view_file`, `grep_search`).

---

## 💡 Why BUBU?

### The Problem: Context Window Exhaustion
When modern coding agents tackle real-world bugs, they frequently need to inspect 20–30 files, browse long logs, or read architectural contracts. This floods the agent's context window with 100,000–300,000 tokens:
- **Attention Degradation:** Models suffer from the "needle in a haystack" phenomenon, missing critical constraints.
- **Extreme Latency:** Every conversational turn becomes progressively slower.
- **Quota Burn:** Expensive conversation context quotas are rapidly consumed.

### The Solution: External Context Processing
BUBU shifts heavy reading and initial diagnosis out-of-band to Google's high-capacity Gemini API:
- **Zero Chat Context Bloat:** Raw source files never touch your main chat session.
- **Verified Grounding:** Every line citation is checked against disk before being returned.
- **Instant Caching:** Identical analyses return in <0.5 seconds at zero API cost.
- **Seamless Continuity:** If the API is offline, the agent falls back to local inspection without skipping a beat.

---

## 🚫 What BUBU is NOT

- **NOT an autonomous coding agent:** BUBU does not write code to your source tree or execute shell commands.
- **NOT an Antigravity replacement:** BUBU is a specialized context-processing tool designed to serve the lead agent.
- **NOT an all-or-nothing requirement:** In `auto` mode, BUBU sleeps during small edits and only wakes for heavy analytical tasks.
- **NOT an external closed service:** BUBU runs locally on your machine, invoking Google AI Studio using your own direct credentials.

---

## 🏗️ Architecture & The Three Layers

BUBU is structured across three distinct operational layers:

```text
┌─────────────────────────────────────────────────────────────┐
│                 LAYER A: LEAD AGENT (ANTIGRAVITY)            │
│  • High-level planning & orchestration                      │
│  • Reads compact evidence & makes decisions                 │
│  • Modifies code, runs build tools, and writes tests        │
└──────────────────────────────┬──────────────────────────────┘
                               │ evaluates task
                               ▼
┌─────────────────────────────────────────────────────────────┐
│                 LAYER B: PROJECT WORKER (BUBU)              │
│  • Local CLI worker (.agents/skills/ai-studio-worker/)      │
│  • Multi-factor SHA-256 cache & local quota tracker         │
│  • Auto Mode decision engine & evidence verification        │
└──────────────────────────────┬──────────────────────────────┘
                               │ HTTPS (TLS 1.3)
                               ▼
┌─────────────────────────────────────────────────────────────┐
│                 LAYER C: GEMINI API (GOOGLE AI STUDIO)      │
│  • Heavy context analysis (100k+ token capacity)            │
│  • User-owned API key (Free Tier or Paid)                   │
│  • Strict structured JSON schema enforcement                │
└─────────────────────────────────────────────────────────────┘
```

### Layer A: Global Antigravity Integration
Antigravity discovers BUBU via its native skill discovery mechanism. When `.agents/skills/ai-studio-worker/SKILL.md` is present in a workspace, Antigravity learns how and when to offload heavy analysis.

### Layer B: Project-Scoped BUBU Runtime
Each project maintains its own isolated runtime in `.ai-worker/`:
- `.ai-worker/config.json`: Project-specific mode (`auto`, `enabled`, `disabled`).
- `.ai-worker/cache/`: Multi-factor SHA-256 cache index.
- `.ai-worker/quota/`: Daily usage safety accounting ledger.
- `.ai-worker/reports/`: Full analytical markdown reports generated by Gemini.

### Layer C: Gemini API
User-provided Gemini API key (via `.env` or system environment). The worker communicates directly with Google AI Studio's REST endpoints over secure HTTPS.

---

## ⚙️ Worker Usage Modes

Configured in `.ai-worker/config.json`:

| Mode | Behavior | When to Use |
| :--- | :--- | :--- |
| **`auto`** *(Default & Recommended)* | Dynamically evaluates task complexity. Offloads large/multi-file tasks to Gemini; skips worker for small local edits. | Standard everyday pair programming. Balances speed, context savings, and API quotas. |
| **`enabled`** | Always activates the worker for analytical requests. | Large legacy refactoring, repository-wide compliance audits, or multi-module investigations. |
| **`disabled`** | Completely deactivates the worker. Antigravity inspects all files directly within its own conversation context. | Working without an internet connection, without an API key, or on sensitive internal micro-edits. |

### Querying or Changing Modes
```powershell
# Query active mode
python .agents/skills/ai-studio-worker/scripts/ai_worker.py --get-mode

# Set mode
python .agents/skills/ai-studio-worker/scripts/ai_worker.py --set-mode auto
```

---

## 🎯 Auto Mode Decision Engine

Auto Mode (`--decide`) analyzes your request before making any API calls:

```powershell
python .agents/skills/ai-studio-worker/scripts/ai_worker.py `
  --type <TASK_TYPE> `
  --prompt "<instruction>" `
  --files <file1> <file2> ... `
  --decide
```

### When Auto Mode Offloads (`USE`):
- **Multi-File Scope:** 3 or more source files targeted.
- **Heavy Payload:** Total file size $\ge 15\text{ KB}$.
- **Analytical Task Types:** `AUDIT`, `COMPARE`, `RESEARCH`, `VALIDATE`.
- **Complex Keywords:** Concurrency, race conditions, memory leaks, architectural regressions, circular dependencies.

### When Auto Mode Skips (`SKIP`):
- **Single-File Micro-Edits:** Changing button text, adjusting a CSS style, renaming a local variable.
- **Obvious Local Bugs:** Fixing a typo or null check where context is already clear.
- **Fast Iterative Tweaks:** Localized edits that Antigravity can solve instantly in conversation.

---

## 🔑 Getting a Gemini API Key

BUBU requires a Google Gemini API key. Free-tier keys are available to developers at no cost.

### Step-by-Step Setup:
1. Navigate to the official Google AI Studio page:  
   👉 **[https://aistudio.google.com/app/apikey](https://aistudio.google.com/app/apikey)**
2. Sign in with your Google account.
3. Click **Create API Key** (choose a Google Cloud project or create a default one).
4. Copy your key.
5. In your project root, copy `.env.example` to `.env`:
   ```bash
   cp .env.example .env
   ```
6. Open `.env` and set your key:
   ```env
   GEMINI_API_KEY=your_actual_gemini_api_key_here
   ```

---

## 🔒 API Key Security & Privacy

> [!CAUTION]
> **Never commit your API key to Git or share it publicly.**

BUBU enforces strict security boundaries:
- **Repository Cleanliness:** Real `.env` files and credentials are gitignored by default.
- **In-Memory Masking:** The worker redacts API keys from stdout, stderr, logs, and generated reports (`[REDACTED_API_KEY]`).
- **Secret File Exclusion:** The worker automatically skips files matching `.env*`, `*.pem`, `*.key`, `id_rsa*`, `credentials*`, or `*.keystore`.
- **Path Traversal Guard:** Any path containing `../` that attempts to escape the project root is blocked with exit code `7` (`EXIT_PATH_ERROR`).
- **Prompt Injection Isolation:** Source code is demarcated as `UNTRUSTED SOURCE MATERIAL`. Directives embedded in code comments (e.g. "ignore instructions") cannot alter the worker's operational protocol.

---

## 📊 Understanding Free API Limits & Tokens

### Core Concepts Explained Simply

| Concept | Simple Explanation | Technical Definition |
| :--- | :--- | :--- |
| **Request** | 1 analytical job sent to Gemini by BUBU. | A single HTTP POST call to `generativelanguage.googleapis.com`. |
| **Token** | The basic unit of text processed by the model (~4 characters of English text or code). | A sub-word token generated by Gemini's Byte-Pair Encoding (BPE) tokenizer. |
| **Input Tokens** | The files, line numbers, and prompts sent to Gemini. | Tokens billed/counted against the prompt context quota. |
| **Output Tokens** | The structured JSON diagnosis returned by Gemini. | Tokens generated by the model in its response candidate. |
| **Cache** | A stored diagnosis used to answer identical requests instantly. | Local SHA-256 indexed JSON record on disk; avoids network calls. |

### Free Tier Quota Metrics: RPM, TPM, RPD
Google AI Studio measures usage across three primary limits:
- **RPM (Requests Per Minute):** How many distinct analysis requests you can send in 60 seconds.
- **TPM (Tokens Per Minute):** The maximum volume of combined text and code processed per minute.
- **RPD (Requests Per Day):** The maximum total requests permitted within a 24-hour calendar window.

> [!IMPORTANT]
> **1 File $\neq$ 1 Request.**
> In BUBU, an entire batch of 10, 20, or 30 source files is bundled into a **single** API request. You do not burn 30 requests to analyze 30 files.

To inspect active quotas for your account and model, check your project settings directly at [Google AI Studio](https://aistudio.google.com/).

---

## ⚡ Multi-Factor Caching Engine

BUBU includes an intelligent local caching engine to eliminate redundant API calls and prevent quota burn.

### The 6-Factor Cache Formula
Unlike simple caches that only check file timestamps, BUBU's cache key reflects the complete analytical environment:

$$\text{Cache Key} = \text{SHA-256}(\text{Version} + \text{Model} + \text{TaskType} + \text{Prompt} + \text{SortedFileHashes} + \text{Config})$$

### Cache Invalidation Rules:
The cache is invalidated and a fresh API call is triggered if:
1. Any targeted file is modified by even 1 character (file SHA-256 changes).
2. The user's prompt or question is altered.
3. The assigned task type (`DEBUG` vs `AUDIT`) changes.
4. The Gemini model is switched.
5. The worker engine version is updated.
6. The `--force` flag is specified.

When a cache hit occurs, BUBU delivers verified findings in **<0.5 seconds** with **0 API calls** and **0 additional tokens**.

---

## 🛡️ Local Quota Accounting vs. Server Quota

BUBU maintains a local usage ledger in `.ai-worker/quota/quota_tracker.json`.

> **Key Distinction:**
> The **Local Quota Tracker** is a client-side safety guardrails system. It is designed to prevent accidental runaway loops or rapid exhaustion. It does not replace Google's server-side rate limits, but works proactively to keep your usage well within safe operating margins.

### Transient Error Handling
If Google's servers return `429 (Rate Limit)` or `503 (Server Busy)`, BUBU applies **exponential backoff** (retrying after 1s, 2s, and 4s). If the limit persists, it transitions to [Resilient Fallback](#-resilient-graceful-fallback).

---

## 🔄 Resilient Graceful Fallback

BUBU follows a zero-failure philosophy: **an analytical worker issue must never crash the lead developer's workflow.**

If the Gemini API encounters:
- `429 Too Many Requests`
- Daily quota exhaustion
- Network interruption or timeout
- Unconfigured API key

BUBU cleanly outputs a structured fallback JSON payload:
```json
{
  "task_id": "audit-20260921-b4c5d6",
  "status": "fallback",
  "fallback_reason": "QUOTA_EXHAUSTED",
  "summary": "Gemini API unavailable (QUOTA_EXHAUSTED). Falling back to local Antigravity inspection."
}
```
The process exits with code `0`. Antigravity detects the fallback state and proceeds immediately with local inspection tools (`view_file`, `grep_search`).

---

## 🧪 Empirical Validation & Stress Test Evidence

BUBU's performance and safety have been validated through rigorous stress testing on an independent 31-file synthetic Kotlin codebase:

```text
======================================================================
AI STUDIO WORKER — EMPIRICAL VALIDATION METRICS
======================================================================
Codebase Scale:          31 files | 2,571 lines | 116.8 KB
Injected Flaws:          8 complex logical, concurrency & leak defects
Findings Detected:       10 total (8/8 injected + 2 valid secondary issues)
Evidence Grounding:      10/10 citations verified against disk (100%)
Context Payload Proxy*:  ~94.7% context payload reduction proxy
Cache Performance:       Initial: 34.57s  ──►  Cache Hit: 0.42s (~82x faster)
Cache Token Burn:        0 additional tokens consumed on cache hits
Read-Only Guarantee:     Source file SHA-256 hashes 100% identical
Secret Leakage:          0 occurrences across stdout, stderr, reports & logs
Path Traversal:          Blocked outside project root (Exit code 7)
Independent Test Suite:  30 / 30 validation tests passed (100% PASS)
======================================================================
```

> [!NOTE]
> *\*Important Disclosure on the ~94.7% Metric:*  
> This metric measures the **context-payload reduction proxy** (the byte difference between sending raw source files plus detailed reports into the chat vs. returning only the verified compact JSON). It is not a direct measurement of Antigravity's internal proprietary token consumption.

---

## 🌐 Privacy and Data Flow

```text
[Local Project Source]
        │
        ▼ (Targeted files only; secret files excluded)
   [ai_worker.py]
        │
        ▼ (HTTPS / TLS 1.3 encryption)
 [Google AI Studio] (Gemini API)
        │
        ▼ (Analysis & citations)
   [ai_worker.py]
        │
        ├──► Full Report written to disk (.ai-worker/reports/)
        └──► Compact Summary emitted to stdout ──► Antigravity Chat
```

- **Data in Flight:** Data sent to Google AI Studio is encrypted in transit via TLS 1.3.
- **Enterprise & Proprietary Code:** If you are bound by strict enterprise privacy agreements, review Google's [Terms of Service](https://ai.google.dev/terms). In Google AI Studio's Free Tier, data may be used to improve Google products; on Paid Tiers, user data is not used for model training.

---

## 📄 License & Commercial Terms

BUBU is licensed under the **BUBU Source-Available & Commercial License (Version 1.0)**.  
Copyright (c) 2026 **`aethelondev-stack`**. All rights reserved.

- **Non-Commercial Use:** Free for personal exploration, educational projects, hobbyists, security research, academic study, and internal non-monetized evaluation.
- **Commercial Use Restriction:** Any deployment, bundling, embedding, distribution, or offering of BUBU as part of a commercial product, proprietary software package, monetization workflow, or paid SaaS/cloud offering **strictly requires a separate, written Commercial License** from the copyright holder (`aethelondev-stack`).
- **Commercial Licensing Inquiries:** To negotiate commercial terms, request custom licensing, or inquire about enterprise deployments, please contact the author via [GitHub: aethelondev-stack](https://github.com/aethelondev-stack).

*Notice: This summary and the repository [LICENSE](./LICENSE) document are provided for this project and do not constitute formal legal advice.*

---

## ❓ Frequently Asked Questions (FAQ)

### What is BUBU?
BUBU is an out-of-band context-processing worker that helps AI coding agents (like Google Antigravity) analyze large codebases without bloating their conversation context.

### Can BUBU be used without Antigravity?
Yes. BUBU can be invoked directly from PowerShell, Bash, or any external automation script using `python .agents/skills/ai-studio-worker/scripts/ai_worker.py`.

### Which Gemini model does it use?
It defaults to `gemini-3.6-flash`. You can customize the model by setting `GEMINI_MODEL` in `.env` or `.ai-worker/config.json`.

### Is it free to use?
Yes. Google AI Studio offers a free tier for Gemini API keys. Check [Google AI Studio Pricing](https://ai.google.dev/pricing) for details.

### How many requests do I get per day?
Quotas depend on your Google AI Studio tier and active model. You can monitor live quotas in your Google AI Studio dashboard.

### Does 100 files equal 100 API requests?
No! BUBU bundles multiple files into a single prompt payload. A 20-file audit typically counts as **one single request**.

### Where do I store my API key?
In `.env` in your project root (recommended) or in your operating system's user environment variables.

### Can I commit my API key to GitHub?
**No, absolutely not.** BUBU's `.gitignore` protects `.env` from being tracked.

### Does the worker run on every single prompt?
No. In `auto` mode, BUBU only activates for complex, multi-file, or deep analytical tasks. Small localized edits bypass the worker entirely.

### What happens if I re-run the same analysis?
BUBU's multi-factor cache serves the result in <0.5s with zero API calls and zero token consumption.

### What happens if the Gemini API is down or quota is exhausted?
BUBU emits a graceful fallback response (`status: "fallback"`), allowing your agent to continue seamlessly using local file inspection.

### Does the worker ever modify my source code?
**Never.** The worker is strictly read-only and only writes to the `.ai-worker/` directory.

---

## 🗺️ Documentation Map

```text
BUBU/
├── README.md                          # Master documentation & quick start
├── LICENSE                            # BUBU Source-Available & Commercial License 1.0
├── INSTALL.md                         # 4-step installation & agent prompt guide
├── SECURITY.md                        # Threat model, secret handling & reporting
├── CONTRIBUTING.md                    # Contribution rules & testing procedures
├── CHANGELOG.md                       # Release notes and version history
├── .env.example                       # API key & configuration template
├── .gitignore                         # Security & runtime ignore rules
│
├── .agents/
│   └── skills/
│       └── ai-studio-worker/
│           ├── SKILL.md               # Antigravity skill interface definition
│           └── scripts/
│               └── ai_worker.py       # Core worker script (Python standard lib)
│
└── studio/
    ├── README.md                      # Documentation hub
    ├── ARCHITECTURE.md                # Detailed orchestration & component guide
    ├── WORKER_PROTOCOL.md             # CLI options, task types & output contract
    ├── QUOTA_AND_CACHE.md             # 6-factor caching & quota accounting
    └── TESTING_GUIDE.md               # Verification commands & empirical data
```

---

## 📋 Release Checklist

- [x] Zero real API keys or secrets in repository files
- [x] Clean `.env.example` template provided
- [x] Machine-specific absolute paths removed (universal portability)
- [x] `.gitignore` verified for secrets, cache, and virtual environments
- [x] Comprehensive, professional English README
- [x] Dedicated instructions and prompt for AI coding agents
- [x] Getting a Gemini API key guide with official Google links
- [x] Clear explanation of free-tier quotas (RPM, TPM, RPD)
- [x] Simple and technical explanations of tokens and caching
- [x] Multi-factor SHA-256 caching fully documented
- [x] Local quota tracking vs. Google server quota distinguished
- [x] Auto mode decision engine criteria and examples documented
- [x] Mode comparison table (`auto`, `enabled`, `disabled`)
- [x] Resilient graceful fallback protocol documented
- [x] Strict read-only guarantee documented
- [x] Empirical test results and ~94.7% proxy disclosure verified
- [x] Privacy, data flow, and threat model documented
- [x] SECURITY.md, CONTRIBUTING.md, and CHANGELOG.md created
- [x] LICENSE created (BUBU Source-Available & Commercial License 1.0 - Copyright aethelondev-stack)

---

## 🔗 Official References

- **Google AI Studio Console:** [https://aistudio.google.com/](https://aistudio.google.com/)
- **Get Gemini API Key:** [https://aistudio.google.com/app/apikey](https://aistudio.google.com/app/apikey)
- **Gemini API Quickstart:** [https://ai.google.dev/gemini-api/docs/quickstart](https://ai.google.dev/gemini-api/docs/quickstart)
- **Gemini API Pricing:** [https://ai.google.dev/pricing](https://ai.google.dev/pricing)
- **Gemini API Rate Limits:** [https://ai.google.dev/gemini-api/docs/rate-limits](https://ai.google.dev/gemini-api/docs/rate-limits)
- **Google AI Terms of Service:** [https://ai.google.dev/terms](https://ai.google.dev/terms)
