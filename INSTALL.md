# Installation Guide (INSTALL.md)

BUBU is a portable, modular distribution package. You can install it into any project to offload heavy analysis, multi-file debugging, and architecture audits to Google AI Studio's Gemini API while keeping your main AI agent context lean and responsive.

---

## 🚀 4-Step Quick Setup

### Step 1: Copy BUBU Files into Your Project Root

Copy the following core items from this repository into your target project's root folder:

```text
your-project-root/
├── .agents/
│   └── skills/
│       └── ai-studio-worker/        # Antigravity skill & Python worker script
├── studio/                          # Architecture, protocol, and test documentation
├── .ai-worker/                      # Configuration template & runtime directories
│   ├── config.json.example
│   ├── cache/
│   ├── quota/
│   └── reports/
├── .env.example                     # API key template
├── .gitignore                       # Security & runtime ignore rules
└── README.md
```

> **Tip:** If your project already has a `.gitignore`, append the rules from BUBU's `.gitignore` to prevent committing your `.env` and local cache.

---

### Step 2: Get Your Gemini API Key

1. Navigate to [Google AI Studio](https://aistudio.google.com/app/apikey).
2. Sign in with your Google account and click **Create API Key**.
3. Choose one of two methods to configure your key:

#### Method A: Project `.env` File (Recommended for Local Projects)
Copy `.env.example` to `.env` in your project root:
```bash
cp .env.example .env
```
Open `.env` and paste your key:
```env
GEMINI_API_KEY=YOUR_ACTUAL_GEMINI_API_KEY
```
*(BUBU's `.gitignore` ensures this file is never committed).*

#### Method B: User / System Environment Variable (Recommended for Multi-Project Use)
**PowerShell (Windows):**
```powershell
[System.Environment]::SetEnvironmentVariable('GEMINI_API_KEY', 'YOUR_ACTUAL_GEMINI_API_KEY', 'User')
```
**Bash / Zsh (macOS / Linux):**
```bash
echo 'export GEMINI_API_KEY="YOUR_ACTUAL_GEMINI_API_KEY"' >> ~/.zshrc
source ~/.zshrc
```

---

### Step 3: Verify Installation

Run the built-in status check from your project root:

```powershell
python .agents/skills/ai-studio-worker/scripts/ai_worker.py --status
```

Expected output:
```json
{
  "status": "active",
  "worker_version": "1.0.0",
  "project_root": "/path/to/your-project",
  "worker_mode": "auto",
  "model": "gemini-3.6-flash",
  "quota": {
    "requests": 0
  },
  "cached_entries": 0
}
```

---

### Step 4: Configure Worker Usage Mode (Optional)

BUBU defaults to **`auto`** mode. You can adjust this anytime:

```powershell
# Auto mode: Dynamically offloads only complex/multi-file tasks (Recommended)
python .agents/skills/ai-studio-worker/scripts/ai_worker.py --set-mode auto

# Always use Worker for analytical requests
python .agents/skills/ai-studio-worker/scripts/ai_worker.py --set-mode enabled

# Disable Worker completely (All analysis stays in primary agent context)
python .agents/skills/ai-studio-worker/scripts/ai_worker.py --set-mode disabled
```

---

## 🤖 Alternative: 1-Prompt Setup with an AI Coding Agent

If you are using an AI coding agent such as Google Antigravity, simply paste this prompt into your conversation once:

```text
Install and verify BUBU for this project.

First, discover the BUBU skill and inspect the existing project configuration.

If my Worker usage preference is not yet saved, ask me once:
1. enabled
2. disabled
3. auto (Recommended)

If my preference is already saved, do not ask again.

Do NOT request, reveal, or output my real API key into the chat.
Verify that the installation and worker status are functional before modifying any code.
```

Your agent will inspect the installation, ask for your preferred mode once, save it to `.ai-worker/config.json`, and follow BUBU's `SKILL.md` rules for all future analytical tasks.
