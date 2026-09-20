# Contributing to BUBU

Thank you for your interest in contributing to **BUBU (AI Studio Context-Processing Worker)**! We welcome bug reports, documentation improvements, architectural feedback, and pull requests.

---

## 1. Core Principles to Respect

Before making any changes, please keep in mind BUBU's non-negotiable architectural principles:

1. **Lead Architect vs. Analyst Division:** Worker is strictly a read-only research and evidence gathering service. It never edits source files. The lead agent (e.g. Antigravity) or developer decides and applies changes.
2. **Zero Hard Dependencies:** `ai_worker.py` relies exclusively on Python standard libraries (`urllib.request`, `json`, `hashlib`, `pathlib`, etc.). Do not introduce heavy dependencies (like `requests`, `aiohttp`, or framework SDKs) without strong justification.
3. **Cross-Platform Portability:** BUBU must run cleanly on Windows (PowerShell/CMD), macOS, and Linux. Never hardcode OS-specific paths or user profile directories.
4. **Zero-Secret Guarantee:** No real API keys or sensitive project files should ever appear in repository files, tests, reports, or PR discussions.

---

## 2. Development Setup

1. **Clone the repository:**
   ```bash
   git clone https://github.com/YOUR_USERNAME/bubu.git
   cd bubu
   ```

2. **Python Environment:**
   - Python 3.10 or later is recommended.
   - Standard library only; no virtual environment or `pip install` is strictly required for the core worker.

3. **API Key Setup (Local Only):**
   - Copy `.env.example` to `.env`:
     ```bash
     cp .env.example .env
     ```
   - Add your test Gemini API key from [Google AI Studio](https://aistudio.google.com/app/apikey).
   - Verify that `.env` is ignored by Git (`git status` should not show `.env`).

---

## 3. Running Verification Tests

Run the standard suite of local verification commands before submitting changes:

```powershell
# 1. Status Check
python .agents/skills/ai-studio-worker/scripts/ai_worker.py --status

# 2. Mode Management
python .agents/skills/ai-studio-worker/scripts/ai_worker.py --get-mode
python .agents/skills/ai-studio-worker/scripts/ai_worker.py --set-mode auto

# 3. Decision Engine Check (Auto Mode)
python .agents/skills/ai-studio-worker/scripts/ai_worker.py `
  --type ANALYZE `
  --files studio/README.md `
  --prompt "Minor documentation tweak" `
  --decide

# 4. Dry-Run (No network / No quota consumed)
python .agents/skills/ai-studio-worker/scripts/ai_worker.py `
  --type RESEARCH `
  --files studio/ARCHITECTURE.md `
  --prompt "Validate architecture definitions" `
  --dry-run

# 5. Path Traversal Guard Verification
python .agents/skills/ai-studio-worker/scripts/ai_worker.py `
  --type ANALYZE `
  --files "../../system.ini" `
  --prompt "Test traversal" `
  --dry-run
# Must exit with code 7 (EXIT_PATH_ERROR)
```

---

## 4. Coding Standards

- **Code Style:** Follow PEP 8 guidelines with clean type hints and docstrings.
- **Output Hygiene:**
  - `stdout` is reserved strictly for machine-readable JSON results.
  - Informational messages, logs, and diagnostic notices must be written to `sys.stderr` using the `log()` helper.
- **Atomic Writes:** All file modifications to `.ai-worker/` must use atomic writes via temporary files to avoid race conditions.
- **Error Handling:** Graceful fallback is required. If network or API errors occur, return a valid JSON payload with `status: "fallback"` and exit code `0` so the lead agent's workflow is never aborted.

---

## 5. Submitting a Pull Request

1. Create a feature branch (`git checkout -b feature/my-improvement`).
2. Verify all status, decide, dry-run, and security checks pass.
3. Ensure no temporary files (`.ai-worker/cache/*`, `.ai-worker/reports/*`, `.env`) are staged.
4. Write clear, imperative commit messages.
5. Open a Pull Request on GitHub with a concise summary of changes and verification evidence.
