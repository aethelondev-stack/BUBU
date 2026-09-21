# BUBU: Provider-Agnostic LLM Context Worker

> **High-Performance, Context-Preserving Research & Analytical Worker for AI Coding Agents (Google Antigravity, Cursor, Cline). Built with Zero External Dependencies (100% Python Standard Library).**

BUBU is a source-available, modular companion for autonomous AI coding agents like [Google Antigravity](https://antigravity.google/). It solves the critical bottleneck of **context window bloat** by offloading heavy multi-file reading, stack-trace debugging, security audits, and library research out-of-band to a dedicated LLM worker layer (**Google Gemini** as Provider #1 by default, or any **OpenAI-Compatible** API such as DeepSeek, OpenRouter, or local Ollama), returning only compact, evidence-verified structured findings to the lead coding agent.

---

## 🌐 The Ecosystem: BUBU + NEXUS + ARGUS

BUBU is the dedicated Context & LLM Worker within the autonomous three-layer agent ecosystem:

```text
BUBU (Context Worker)
  ↓
NEXUS (Coordination & Routing Layer)
  ↓
ARGUS (Vision Shield & Token Guardian)
```

- **[BUBU](https://github.com/aethelondev-stack/BUBU):** Offloads multi-file context analysis, architectural audits, and memory leak detection.
- **[NEXUS](https://github.com/aethelondev-stack/NEXUS):** Coordinates multi-worker workflows, handles provider quota (429/TPM) and rate limits, loop detection, and evidence aggregation.
- **[ARGUS](https://github.com/aethelondev-stack/ARGUS):** On-device vision shield and visual grounding using local GPU to prevent multimodal token bleeding.

---

## 📑 Table of Contents

- [🌐 The Ecosystem: BUBU + NEXUS + ARGUS](#-the-ecosystem-bubu--nexus--argus)
- [Quick Start](#-quick-start)
- [🤖 Install with an AI Coding Agent](#-install-with-an-ai-coding-agent)
- [🤖 For AI Coding Agents](#-for-ai-coding-agents)
- [Supported LLM Providers](#-supported-llm-providers)
- [Why BUBU?](#-why-bubu)
- [What BUBU is NOT](#-what-bubu-is-not)
- [Architecture & The Three Layers](#-architecture--the-three-layers)
- [Worker Usage Modes](#-worker-usage-modes)
- [Auto Mode Decision Engine](#-auto-mode-decision-engine)
- [Configuring Providers & API Keys](#-configuring-providers--api-keys)
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

## ⚡ Quick Start (Zero-Prompt Auto-Onboarding)

Get up and running in under 30 seconds:

1. **Copy the Portable Package (`her projeye taşınacak klasörler/`) into your project:**
   Copy the contents of `her projeye taşınacak klasörler/` into your target repository root:
   - `.agents/skills/ai-studio-worker/` (Worker engine)
   - `.ai-worker/` (Cache, quota, and reports scaffolding)
   - `AGENTS.md` (Zero-token on-demand onboarding rule)
   - `.env` (Your API key) & `.gitignore`
2. **Start coding normally:**
   Open a new chat in Antigravity and prompt your agent with any engineering task (e.g., *"Build a login API"*).
   - Antigravity's `AGENTS.md` rule automatically fulfills your coding task first.
   - At the end of the response, it prompts for your BUBU and ARGUS mode preference once.
   - **Zero-Token Guard:** If you select `disabled`, it writes the setting without wasting a single token reading skills or scripts.
   - If you select `auto` or `enabled`, it loads the worker on-demand and saves your preference forever.

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

## 🔌 Supported LLM Providers

BUBU is designed from the ground up as a **provider-agnostic worker** implemented exclusively using Python standard library networking (`urllib.request`). It requires **zero pip dependencies**:

| Provider ID | Provider Name | Default Model | Authentication | Typical Use Case |
| :--- | :--- | :--- | :--- | :--- |
| **`gemini`** *(Default)* | **Google Gemini** (via Google AI Studio) | `gemini-3.6-flash` | `GEMINI_API_KEY` (URL query or Bearer) | Default high-capacity provider. Generous free tier, 1M+ token context windows, fast inference. |
| **`openai_compatible`** | **OpenAI / DeepSeek / OpenRouter / Ollama** | `gpt-4o-mini` | `OPENAI_API_KEY` (Bearer Token) | Self-hosted LLMs, enterprise OpenAI endpoints, DeepSeek-V3/R1, or multi-provider aggregators. |

Switching providers is as simple as passing `--provider <name>` on the CLI or setting `AI_WORKER_PROVIDER=<name>` in your environment.

---

## 💡 Why BUBU?

### The Problem: Context Window Exhaustion
When modern coding agents tackle real-world bugs, they frequently need to inspect 20–30 files, browse long logs, or read architectural contracts. This floods the agent's context window with 100,000–300,000 tokens:
- **Attention Degradation:** Models suffer from the "needle in a haystack" phenomenon, missing critical constraints.
- **Extreme Latency:** Every conversational turn becomes progressively slower.
- **Quota Burn:** Expensive conversation context quotas are rapidly consumed.

### The Solution: External Context Processing
BUBU shifts heavy reading and initial diagnosis out-of-band to a dedicated LLM worker layer:
- **Zero Chat Context Bloat:** Raw source files never touch your main chat session.
- **Verified Grounding:** Every line citation is checked against disk before being returned.
- **Instant Caching:** Identical analyses return in <0.5 seconds at zero API cost.
- **Seamless Continuity:** If the API is offline, the agent falls back to local inspection without skipping a beat.

---

## 🚫 What BUBU is NOT

- **NOT an autonomous coding agent:** BUBU does not write code to your source tree or execute shell commands.
- **NOT an Antigravity replacement:** BUBU is a specialized context-processing tool designed to serve the lead agent.
- **NOT an all-or-nothing requirement:** In `auto` mode, BUBU sleeps during small edits and only wakes for heavy analytical tasks.
- **NOT an external closed service:** BUBU runs locally on your machine, invoking your selected LLM provider using your own direct credentials.

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
│  • Zero-dependency HTTP runtime (urllib.request)            │
└──────────────────────────────┬──────────────────────────────┘
                               │ HTTPS / TLS 1.3
                               ▼
┌─────────────────────────────────────────────────────────────┐
│            LAYER C: LLM PROVIDER ABSTRACTION LAYER          │
│                                                             │
│   ┌───────────────────────────┬──────────────────────────┐  │
│   │   Provider #1: Gemini     │ Provider #2: OpenAI-Comp │  │
│   │   • Google AI Studio REST │ • /chat/completions REST │  │
│   │   • gemini-3.6-flash      │ • OpenAI / DeepSeek /    │  │
│   │   • Free Tier Available   │   OpenRouter / Ollama    │  │
│   └───────────────────────────┴──────────────────────────┘  │
│  • User-owned API keys (never logged or exposed)             │
│  • Strict structured JSON schema enforcement                │
└─────────────────────────────────────────────────────────────┘
```

### Layer A: Global Antigravity Integration
Antigravity discovers BUBU via its native skill discovery mechanism. When `.agents/skills/ai-studio-worker/SKILL.md` is present in a workspace, Antigravity learns how and when to offload heavy analysis.

### Layer B: Project-Scoped BUBU Runtime
Each project maintains its own isolated runtime in `.ai-worker/`:
- `.ai-worker/config.json`: Project-specific mode (`auto`, `enabled`, `disabled`) and provider configuration.
- `.ai-worker/cache/`: Multi-factor SHA-256 cache index (provider-isolated).
- `.ai-worker/quota/`: Daily usage safety accounting ledger.
- `.ai-worker/reports/`: Full analytical markdown reports generated by the worker.

### Layer C: LLM Provider Layer
The worker communicates directly with provider REST endpoints over TLS 1.3:
- **Gemini:** `https://generativelanguage.googleapis.com/v1beta/models/{model}:generateContent`
- **OpenAI-Compatible:** `{base_url}/chat/completions`

---

## ⚙️ Worker Usage Modes

Configured in `.ai-worker/config.json`:

| Mode | Behavior | When to Use |
| :--- | :--- | :--- |
| **`auto`** *(Default & Recommended)* | Dynamically evaluates task complexity. Offloads large/multi-file tasks to the worker; skips worker for small local edits. | Standard everyday pair programming. Balances speed, context savings, and API quotas. |
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

## 🔑 Configuring Providers & API Keys

BUBU supports multiple LLM providers. Choose the one that best fits your workflow:

### Provider 1: Google Gemini (Default)

Google AI Studio provides generous free-tier keys at zero cost.

1. Navigate to: 👉 **[https://aistudio.google.com/app/apikey](https://aistudio.google.com/app/apikey)**
2. Sign in with your Google account and click **Create API Key**.
3. In your project root, configure `.env`:
   ```env
   AI_WORKER_PROVIDER=gemini
   GEMINI_API_KEY=your_actual_gemini_api_key_here
   GEMINI_MODEL=gemini-3.6-flash
   ```

### Provider 2: OpenAI-Compatible (OpenAI, DeepSeek, OpenRouter, Ollama)

You can connect BUBU to any service providing an OpenAI-compatible `/chat/completions` endpoint:

**Example for OpenAI:**
```env
AI_WORKER_PROVIDER=openai_compatible
OPENAI_API_KEY=your_actual_openai_api_key_here
OPENAI_MODEL=gpt-4o-mini
OPENAI_BASE_URL=https://api.openai.com/v1
```

**Example for DeepSeek:**
```env
AI_WORKER_PROVIDER=openai_compatible
OPENAI_API_KEY=your_deepseek_api_key_here
OPENAI_MODEL=deepseek-chat
OPENAI_BASE_URL=https://api.deepseek.com/v1
```

**Example for Local Ollama (No API Key Required):**
```env
AI_WORKER_PROVIDER=openai_compatible
OPENAI_MODEL=qwen2.5-coder:7b
OPENAI_BASE_URL=http://localhost:11434/v1
```

> [!NOTE]
> **Compatibility Note:** Services such as DeepSeek, OpenRouter, and Ollama interface via the OpenAI-compatible endpoint abstraction (`openai_compatible`). While the wire protocol is standard, provider-specific live validation may be required depending on custom model naming, parameter restrictions, or remote rate limits.

### Switching via CLI Flags
You can also override the provider per invocation:
```powershell
python .agents/skills/ai-studio-worker/scripts/ai_worker.py --provider openai_compatible --status
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

### The 7-Factor Cache Formula
Unlike simple caches that only check file timestamps, BUBU's cache key reflects the complete analytical environment, ensuring complete provider and model isolation:

$$\text{Cache Key} = \text{SHA-256}(\text{Version} + \text{Provider} + \text{Model} + \text{TaskType} + \text{Prompt} + \text{SortedFileHashes} + \text{Config})$$

### Cache Invalidation Rules:
The cache is invalidated and a fresh API call is triggered if:
1. Any targeted file is modified by even 1 character (file SHA-256 changes).
2. The user's prompt or question is altered.
3. The assigned task type (`DEBUG` vs `AUDIT`) changes.
4. The provider or model is switched (`gemini` vs `openai_compatible`).
5. The worker engine version is updated.
6. The `--force` flag is specified.

When a cache hit occurs, BUBU delivers verified findings in **<0.5 seconds** with **0 API calls** and **0 additional tokens**.

---

## 🛡️ Local Quota Accounting vs. Server Quota

BUBU maintains a local usage ledger in `.ai-worker/quota/quota_tracker.json`.

> **Key Distinction:**
> The **Local Quota Tracker** is a client-side safety guardrails system. It is designed to prevent accidental runaway loops or rapid exhaustion. It does not replace the provider's server-side rate limits, but works proactively to keep your usage well within safe operating margins.

### Transient Error Handling
If provider servers return `429 (Rate Limit)` or `503 (Server Busy)`, BUBU applies **exponential backoff** (retrying after 1s, 2s, and 4s). If the limit persists, it transitions to [Resilient Fallback](#-resilient-graceful-fallback).

---

## 🔄 Resilient Graceful Fallback

BUBU follows a zero-failure philosophy: **an analytical worker issue must never crash the lead developer's workflow.**

If the LLM provider encounters:
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
  "summary": "LLM Provider unavailable (QUOTA_EXHAUSTED). Falling back to local Antigravity inspection."
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
 [LLM Provider Layer] (Google Gemini / OpenAI / DeepSeek / Local Ollama)
        │
        ▼ (Analysis & citations)
   [ai_worker.py]
        │
        ├──► Full Report written to disk (.ai-worker/reports/)
        └──► Compact Summary emitted to stdout ──► Antigravity Chat
```

- **Data in Flight:** Data sent to external providers is encrypted in transit via TLS 1.3. For local providers (e.g. Ollama on `localhost`), data never leaves your machine.
- **Enterprise & Proprietary Code:** If you are bound by strict enterprise privacy agreements, choose local Ollama or an enterprise provider tier where data is not used for model training.

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

### Which LLM providers are supported?
BUBU supports:
1. **Google Gemini** (via Google AI Studio REST API) — default, generous free tier.
2. **OpenAI-Compatible Providers** — OpenAI, DeepSeek, OpenRouter, vLLM, and local Ollama.

### Does BUBU require any pip packages?
**No.** BUBU is built entirely using Python's standard library (`urllib.request`, `json`, `pathlib`, `hashlib`, etc.). No `pip install` is ever needed.

### Can BUBU be used with local LLMs (Ollama)?
Yes! Set `AI_WORKER_PROVIDER=openai_compatible`, `OPENAI_BASE_URL=http://localhost:11434/v1`, and `OPENAI_MODEL=qwen2.5-coder:7b`. No API key is required for local endpoints.

### Can BUBU be used without Antigravity?
Yes. BUBU can be invoked directly from PowerShell, Bash, or any external automation script using `python .agents/skills/ai-studio-worker/scripts/ai_worker.py`.

### Which model does it use by default?
For Gemini, it defaults to `gemini-3.6-flash`. For OpenAI-compatible, it defaults to `gpt-4o-mini`. Both are fully configurable.

### Is it free to use?
Yes. Google AI Studio offers a free tier for Gemini API keys. Local Ollama is completely free on your own hardware.

### How many requests do I get per day?
Quotas depend on your provider tier and active model. You can monitor live quotas in your provider's dashboard.

### Does 100 files equal 100 API requests?
No! BUBU bundles multiple files into a single prompt payload. A 20-file audit typically counts as **one single request**.

### Where do I store my API keys?
In `.env` in your project root (recommended) or in your operating system's user environment variables.

### Can I commit my API keys to GitHub?
**No, absolutely not.** BUBU's `.gitignore` protects `.env` from being tracked.

### Does the worker run on every single prompt?
No. In `auto` mode, BUBU only activates for complex, multi-file, or deep analytical tasks. Small localized edits bypass the worker entirely.

### What happens if I re-run the same analysis?
BUBU's multi-factor cache serves the result in <0.5s with zero API calls and zero token consumption.

### What happens if the LLM provider is down or quota is exhausted?
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
