# BUBU Installation Guide (INSTALL.md)

This document provides the definitive, canonical installation procedure for integrating **BUBU (Provider-Agnostic LLM Context Worker)** into any project.

BUBU offloads heavy multi-file reading, stack-trace debugging, security audits, and codebase research to configured LLM providers (**Google Gemini** as Provider #1 by default, or any **OpenAI-Compatible** HTTP API such as DeepSeek, OpenRouter, or local Ollama) without context window bloat, returning only compact, evidence-verified findings to the lead coding agent.

---

## 1. Distribution Repository vs. Target Project

When installing BUBU, distinguish clearly between the **BUBU Distribution Repository** (the source repo containing BUBU releases) and your **Target Project** (your existing software repository where you want BUBU to operate):

```text
BUBU Distribution Repository                          Your Target Project Root
├── .agents/skills/ai-studio-worker/  ──[COPY]──►  ├── .agents/skills/ai-studio-worker/
│   ├── SKILL.md                                  │   ├── SKILL.md
│   └── scripts/ai_worker.py                      │   └── scripts/ai_worker.py
├── .ai-worker/                                   ├── .ai-worker/
│   └── config.json.example           ──[COPY]──► │   └── config.json.example
├── .env.example                      ──[COPY]──► ├── .env.example  ──► (cp to .env)
├── studio/ (Reference Docs)          ──[COPY]──► ├── studio/ (Optional architecture docs)
│                                                 │
│   [DO NOT COPY FROM BUBU]                       ├── .gitignore (MERGE rules, do NOT overwrite!)
│   ├── .git/                                     ├── README.md (Keep your project's README!)
│   ├── README.md                                 └── [Your project source code...]
│   ├── INSTALL.md
│   ├── SECURITY.md
│   ├── CHANGELOG.md
│   ├── CONTRIBUTING.md
│   └── LICENSE
```

> [!CAUTION]
> **DO NOT copy the entire BUBU repository into your project.**
> - **DO NOT copy the BUBU `.git/` directory** into your project; your project retains its own Git history.
> - **DO NOT overwrite your target project's `README.md`** with BUBU's `README.md`.
> - **DO NOT overwrite your target project's `.gitignore`**; append BUBU's specific ignore rules instead.
> - **NEVER copy real `.env` files or API keys** between different projects.

---

## 2. Canonical Installation Payload

Before copying any files, review the official classification of all repository components:

| Repository Path | Classification | Role & Description |
| :--- | :---: | :--- |
| **`.agents/skills/ai-studio-worker/`** | **REQUIRED** | Contains `SKILL.md` (agent discovery contract) and `scripts/ai_worker.py` (core execution engine). Built with **zero pip dependencies** (100% Python standard library). |
| **`.ai-worker/config.json.example`** | **REQUIRED** | Template configuration for provider selection, worker mode, and safety budgets. |
| **`.env.example`** | **REQUIRED** | Clean credential template with environment variable schemas for Gemini and OpenAI-compatible providers. |
| **`.ai-worker/cache/`** | **AUTO-CREATED** | Multi-factor SHA-256 cache storage and `index.json`. Created automatically at runtime by `ai_worker.py`. |
| **`.ai-worker/reports/`** | **AUTO-CREATED** | Exhaustive markdown reports written to disk for human or agent reference. Created automatically at runtime. |
| **`.ai-worker/quota/`** | **AUTO-CREATED** | Client-side rate-limiting and daily budget tracking (`quota_tracker.json`). Created automatically at runtime. |
| **`.ai-worker/config.json`** | **AUTO-CREATED** | Active project configuration file. Generated automatically when running `--set-mode` or `--set-provider`. |
| **`studio/`** | **OPTIONAL** | Complete technical reference documentation (`ARCHITECTURE.md`, `WORKER_PROTOCOL.md`, `QUOTA_AND_CACHE.md`, `TESTING_GUIDE.md`, `README.md`). Recommended for architectural reference, but not required for script execution. |
| **`.ai-worker/*/.gitkeep`** | **OPTIONAL** | Empty directory place-markers. Useful if pre-committing directory skeletons to version control. |
| **`.git/`** | **DISTRIBUTION-ONLY** | BUBU's internal version control history. **Do NOT copy to target projects.** |
| **`README.md`** | **DISTRIBUTION-ONLY** | Primary product overview for the BUBU repository. Target projects maintain their own `README.md`. |
| **`INSTALL.md`** | **DISTRIBUTION-ONLY** | This installation guide. Stays in the BUBU distribution repository. |
| **`SECURITY.md`** | **DISTRIBUTION-ONLY** | Vulnerability disclosure and security policy for the BUBU project repository. |
| **`CHANGELOG.md`** | **DISTRIBUTION-ONLY** | Version history and release notes for the BUBU engine. |
| **`CONTRIBUTING.md`** | **DISTRIBUTION-ONLY** | Contribution workflows for BUBU developers. |
| **`LICENSE`** | **DISTRIBUTION-ONLY** | Source-available & commercial licensing terms for the BUBU repository. |
| **`.env`** | **NEVER COPY** | Active environment secrets and API keys. Must be created locally in the target project via `.env.example`. |
| **`.gitignore`** | **MERGE ONLY** | Must be merged with target project's `.gitignore`. Never overwrite an existing project's `.gitignore`. |
| **Runtime Data / Caches** | **NEVER COPY** | `.ai-worker/cache/*`, `.ai-worker/reports/*`, `__pycache__/`, `*.pyc`. Never transfer runtime state across projects. |

---

## 3. Step-by-Step Installation Procedure

### Step 1: Copy Required Files into Your Project Root

From the BUBU repository, copy the required payload into your target project:

#### On Windows (PowerShell):
```powershell
# Navigate to your target project root
cd "C:\path\to\your-project"

# Set path to the BUBU distribution directory
$BUBU_DIR = "C:\path\to\BUBU"

# 1. Copy the agent skill directory (Required)
New-Item -ItemType Directory -Force -Path ".agents\skills"
Copy-Item -Recurse -Force "$BUBU_DIR\.agents\skills\ai-studio-worker" ".agents\skills\"

# 2. Copy the configuration template (Required)
New-Item -ItemType Directory -Force -Path ".ai-worker"
Copy-Item -Force "$BUBU_DIR\.ai-worker\config.json.example" ".ai-worker\"

# 3. Copy the environment template (Required)
Copy-Item -Force "$BUBU_DIR\.env.example" ".env.example"

# 4. Optional: Copy the studio reference documentation
Copy-Item -Recurse -Force "$BUBU_DIR\studio" "studio"
```

#### On Linux / macOS (Bash):
```bash
# Navigate to your target project root
cd /path/to/your-project

# Set path to the BUBU distribution directory
BUBU_DIR="/path/to/BUBU"

# 1. Copy the agent skill directory (Required)
mkdir -p .agents/skills
cp -r "$BUBU_DIR/.agents/skills/ai-studio-worker" .agents/skills/

# 2. Copy the configuration template (Required)
mkdir -p .ai-worker
cp "$BUBU_DIR/.ai-worker/config.json.example" .ai-worker/

# 3. Copy the environment template (Required)
cp "$BUBU_DIR/.env.example" .env.example

# 4. Optional: Copy the studio reference documentation
cp -r "$BUBU_DIR/studio" ./studio
```

---

### Step 2: Merge `.gitignore` Rules (Do NOT Overwrite)

> [!IMPORTANT]
> If your project already has a `.gitignore` file, **do NOT overwrite it**. Instead, append the following block to your existing `.gitignore`:

```gitignore
# ===========================================================================
# BUBU / LLM Context Worker Runtime & Secret Exclusions
# ===========================================================================
.env
.env.*
!.env.example

# Worker active configuration (keep template tracked)
.ai-worker/config.json

# Worker runtime directories (keep directory skeletons tracked via .gitkeep)
.ai-worker/cache/*
!.ai-worker/cache/.gitkeep
.ai-worker/reports/*
!.ai-worker/reports/.gitkeep
.ai-worker/quota/*
!.ai-worker/quota/.gitkeep

# Python byte-code cache
__pycache__/
*.py[cod]
*.tmp
```

If your project does not yet have a `.gitignore`, create one in your project root containing the block above.

---

### Step 3: Configure Your LLM Provider & Credentials

BUBU supports **Google Gemini** as Provider #1 (default) and any **OpenAI-Compatible** API as Provider #2.

1. Create your project's local `.env` file from the template:
   ```bash
   # Linux/macOS
   cp .env.example .env

   # Windows PowerShell
   Copy-Item .env.example .env
   ```

2. Open `.env` and configure your chosen provider:

#### Option A: Google Gemini (Recommended & Default — Free Tier Available)
```env
# Set provider to gemini
AI_WORKER_PROVIDER=gemini

# Primary Google AI Studio API key (https://aistudio.google.com/app/apikey)
GEMINI_API_KEY=your_actual_gemini_api_key_here

# Optional: Gemini model override (Default: gemini-3.6-flash)
GEMINI_MODEL=gemini-3.6-flash
```

#### Option B: OpenAI-Compatible HTTP API (OpenAI, DeepSeek, OpenRouter, Ollama)
```env
# Set provider to openai_compatible
AI_WORKER_PROVIDER=openai_compatible

# API key for the endpoint (or set BUBU_API_KEY / LLM_API_KEY)
OPENAI_API_KEY=your_actual_api_key_here

# Active model identifier
OPENAI_MODEL=gpt-4o-mini

# Target endpoint base URL
OPENAI_BASE_URL=https://api.openai.com/v1
```

> [!NOTE]
> **Notes on OpenAI-Compatible Endpoints:**
> - **Local Ollama:** For local instances running at `http://localhost:11434/v1`, set `OPENAI_BASE_URL=http://localhost:11434/v1` and `OPENAI_MODEL=<your_model_name>`. No API key is required for localhost connections.
> - **DeepSeek / OpenRouter / vLLM:** Interfaced via the standard OpenAI-compatible abstraction layer. Provider-specific live validation may be required depending on custom model endpoints, parameter support, and remote rate limits.

---

### Step 4: Understand Provider vs. Worker Mode

BUBU separates **Which LLM to call** from **When to call it**:

| Dimension | Configuration Setting | Options | Role |
| :--- | :--- | :--- | :--- |
| **Provider** | `AI_WORKER_PROVIDER` or `--provider` | `gemini` (default), `openai_compatible` | Defines the backend LLM service and network protocol. |
| **Worker Mode** | `AI_WORKER_MODE` or `--set-mode` | `auto` (default), `enabled`, `disabled` | Defines the delegation policy for coding agent tasks. |

#### Configuring Worker Mode:
```powershell
# Auto Mode (Default & Recommended): Offloads heavy multi-file tasks; skips micro-edits
python .agents/skills/ai-studio-worker/scripts/ai_worker.py --set-mode auto

# Enabled Mode: Routes all analytical tasks to the worker
python .agents/skills/ai-studio-worker/scripts/ai_worker.py --set-mode enabled

# Disabled Mode: Disables worker completely; all tasks stay in lead agent context
python .agents/skills/ai-studio-worker/scripts/ai_worker.py --set-mode disabled
```

---

### Step 5: Verify the Installation

Execute the status check from your project root:

```powershell
python .agents/skills/ai-studio-worker/scripts/ai_worker.py --status
```

Expected JSON response:
```json
{
  "status": "active",
  "worker_version": "1.0.0",
  "project_root": "C:\\path\\to\\your-project",
  "worker_mode": "auto",
  "provider": "gemini",
  "model": "gemini-3.6-flash",
  "quota": {
    "requests": 0
  },
  "cached_entries": 0
}
```

#### Run a Non-Invasive Dry-Run Test:
Test file collection and payload assembly without making any network calls:
```powershell
python .agents/skills/ai-studio-worker/scripts/ai_worker.py `
  --type ANALYZE `
  --prompt "Verify worker installation and file collection" `
  --files .env.example `
  --dry-run
```

Expected output:
```json
{
  "status": "dry_run_success",
  "task_type": "ANALYZE",
  "provider": "gemini",
  "model": "gemini-3.6-flash",
  "total_files": 1,
  "summary": "Dry-run validated. 1 files ready for ANALYZE analysis..."
}
```

---

## 4. AI Coding Agent Integration Workflow

BUBU is engineered to operate seamlessly with autonomous AI coding agents (Google Antigravity, Cursor, Cline, OpenHands).

### How the Agent Discovers BUBU:
1. When your coding agent opens your project workspace, it scans the repository for agent skills.
2. The agent discovers [`.agents/skills/ai-studio-worker/SKILL.md`](.agents/skills/ai-studio-worker/SKILL.md).
3. The skill instructs the agent on:
   - When to invoke the worker via `--decide` or `--type`.
   - The read-only guarantee: BUBU never modifies project source code.
   - Context isolation: Detailed reports are saved to `.ai-worker/reports/<task_id>.md`, and a compact JSON summary is returned to stdout.
   - Resilient fallback: If an API error or quota limit occurs, the worker emits `status: "fallback"` with exit code `0`, allowing the agent to continue smoothly without aborting the task.

### The 1-Prompt Agent Onboarding:
To onboard your coding agent automatically, paste this prompt into your conversation once:

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

Your agent will inspect the configuration, record your mode preference, and follow BUBU's operational rules for all subsequent engineering tasks.

---

## 5. Security & Secret Hygiene Checklist

- [x] **Zero Dependencies:** BUBU relies 100% on Python standard library modules (`urllib.request`, `json`, `hashlib`, `pathlib`). No third-party packages or virtual environments are needed.
- [x] **No Real Keys in Git:** Confirm `.env` is listed in `.gitignore` before running `git commit`.
- [x] **Automatic Secret Exclusion:** `ai_worker.py` automatically blocks files matching `.env*`, `*.pem`, `*.key`, `id_rsa*`, `credentials*`, or `*.keystore` from ever being processed or transmitted.
- [x] **Path Traversal Protection:** All requested file paths are checked to ensure they resolve strictly within the project root. Traversal attempts (`../`) are halted with exit code `7`.
- [x] **Strict Read-Only Guarantee:** `ai_worker.py` never modifies project source files; write operations are strictly confined to the `.ai-worker/` directory.
