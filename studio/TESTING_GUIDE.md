# Testing & Verification Guide (TESTING_GUIDE.md)

This guide provides test procedures, CLI commands, security tests, multi-provider validation, and empirical data for `ai_worker.py` and the BUBU skill.

---

## 1. Quick Status Verification (`--status`)

Verify that the worker can discover the project root, active configuration mode, active provider, quota counter, and cache state:

```powershell
# Default status (Gemini)
python .agents/skills/ai-studio-worker/scripts/ai_worker.py --status

# OpenAI-compatible status
python .agents/skills/ai-studio-worker/scripts/ai_worker.py --provider openai_compatible --status
```

Expected JSON response:
```json
{
  "status": "active",
  "worker_version": "1.0.0",
  "project_root": "/path/to/your-project",
  "worker_mode": "auto",
  "provider": "gemini",
  "model": "gemini-3.6-flash",
  "quota": {
    "requests": 0
  },
  "cached_entries": 0
}
```

---

## 2. Mode Management Tests

### Query Current Mode
```powershell
python .agents/skills/ai-studio-worker/scripts/ai_worker.py --get-mode
```

### Switch Modes (`enabled` / `disabled` / `auto`)
```powershell
python .agents/skills/ai-studio-worker/scripts/ai_worker.py --set-mode disabled
python .agents/skills/ai-studio-worker/scripts/ai_worker.py --set-mode auto
```

### Test Auto Mode Decision Engine (`--decide`)
Test decision logic without invoking any LLM API or consuming quota:

```powershell
# Small / localized task (Expects SKIP)
python .agents/skills/ai-studio-worker/scripts/ai_worker.py `
  --type ANALYZE `
  --prompt "Fix minor typo in README" `
  --files studio/README.md `
  --decide

# Complex / multi-file task (Expects USE)
python .agents/skills/ai-studio-worker/scripts/ai_worker.py `
  --type AUDIT `
  --prompt "Comprehensive architecture and protocol audit" `
  --files studio/ARCHITECTURE.md studio/WORKER_PROTOCOL.md studio/QUOTA_AND_CACHE.md `
  --decide
```

---

## 3. Dry-Run Execution Across Providers (`--dry-run`)

Simulate payload assembly, line numbering, secret exclusion, and size budgets with zero API calls:

```powershell
# Dry-run with Gemini (Provider #1)
python .agents/skills/ai-studio-worker/scripts/ai_worker.py `
  --provider gemini `
  --type RESEARCH `
  --prompt "Audit studio documentation structure" `
  --files studio/ARCHITECTURE.md `
  --dry-run

# Dry-run with OpenAI-compatible (Provider #2)
python .agents/skills/ai-studio-worker/scripts/ai_worker.py `
  --provider openai_compatible `
  --model gpt-4o-mini `
  --type RESEARCH `
  --prompt "Audit studio documentation structure" `
  --files studio/ARCHITECTURE.md `
  --dry-run
```

---

## 4. Security & Path Traversal Test

Verify that the worker prevents path traversal outside the project root:

```powershell
python .agents/skills/ai-studio-worker/scripts/ai_worker.py `
  --type ANALYZE `
  --prompt "Security test" `
  --files "../../secret.txt" `
  --dry-run
```

Expected result:
- Exit code: `7` (`EXIT_PATH_ERROR`)
- Stderr: `[ai-worker:ERROR] Path traversal detected or file outside project: ../../secret.txt`

---

## 5. Provider Isolation & Multi-Factor Caching Test

1. Run an analysis with `--provider gemini`.
2. Run the identical query with `--provider openai_compatible`.
3. Verify that the cache keys differ due to the provider salt:
   $$\text{Cache Key} = \text{SHA-256}(\text{Version} + \text{Provider} + \text{Model} + \text{TaskType} + \text{Prompt} + \text{Files} + \text{Config})$$
   Different providers never return cross-contaminated cache entries.

---

## 6. Zero-Quota Mock OpenAI-Compatible Testing

BUBU includes built-in compatibility testing without consuming live cloud quotas by targeting a local HTTP mock server (e.g. `http://127.0.0.1:8989` or local Ollama `http://127.0.0.1:11434/v1`):

```powershell
$env:OPENAI_BASE_URL = "http://127.0.0.1:8989"
$env:OPENAI_API_KEY = "test-key"

python .agents/skills/ai-studio-worker/scripts/ai_worker.py `
  --provider openai_compatible `
  --model gpt-4o-mini `
  --type AUDIT `
  --files studio/ARCHITECTURE.md `
  --prompt "Test local mock provider"
```

---

## 7. Empirical Verification & Stress Test Results

BUBU's performance was validated in a comprehensive stress test against a 31-file synthetic project containing 8 injected logical flaws:

| Metric | Measured Result | Verification |
| :--- | :--- | :---: |
| **Test Codebase** | 31 files, 2,571 lines, 116.8 KB | Ground Truth Verified |
| **Injected Flaws** | 8 logical issues (concurrency races, leaks, loop errors) | 8/8 Injected Issues Detected |
| **Additional Findings** | 2 valid secondary issues identified by Gemini | 10 Total Findings Detected |
| **Evidence Verification** | Line numbers and snippets checked against disk | 10/10 Citations Verified (`evidence_verified: true`) |
| **Context Payload Proxy** | Compact JSON (6,701 bytes) vs Raw Source (~127 KB) | **~94.7% Context Payload Reduction Proxy\*** |
| **Cache Speedup** | Initial API run: 34.57s vs Cache hit: 0.42s | **~82x Faster Execution** |
| **Cache Token Savings** | Provider tokens on cache hit | **0 Tokens Consumed** |
| **Read-Only Integrity** | SHA-256 hashes of all source files before & after | **100% Byte-Identical (Read-Only)** |
| **Secret Protection** | Scanned reports, logs, and caches for API keys | **0 Secret Leaks Detected** |
| **Path Traversal Guard** | Attempted access to parent directory files | **Blocked (Exit Code 7)** |
| **Graceful Fallback** | Tested with invalid/missing API credentials | **Smooth Handover (Exit Code 0)** |

*\*Note on the ~94.7% metric: This represents a measured context-payload reduction proxy based on transmitted bytes, not a direct measurement of Antigravity's internal model token usage.*
