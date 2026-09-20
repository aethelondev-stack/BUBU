# Studio Documentation Hub

This directory (`studio/`) contains the complete architectural blueprints, interface protocols, caching specifications, and testing guides for **BUBU (AI Studio Context-Processing Worker)**.

---

## 📚 Document Index

1. **[ARCHITECTURE.md](./ARCHITECTURE.md)**:
   - Deep dive into context isolation, the lead-analyst separation model (Antigravity as Lead Architect, Worker as Analyst), and the end-to-end orchestration pipeline.
2. **[WORKER_PROTOCOL.md](./WORKER_PROTOCOL.md)**:
   - Detailed specification of the 8 supported task types (`DEBUG`, `ANALYZE`, `RESEARCH`, `AUDIT`, `COMPARE`, `SUMMARIZE`, `DOCUMENT`, `VALIDATE`), line-numbered file injection syntax, and the structured JSON output contract.
3. **[QUOTA_AND_CACHE.md](./QUOTA_AND_CACHE.md)**:
   - Mechanics of the 6-factor SHA-256 caching engine, cache invalidation rules, local safety budgets (`quota_tracker.json`), and exponential backoff retry algorithms.
4. **[TESTING_GUIDE.md](./TESTING_GUIDE.md)**:
   - Practical CLI testing procedures, dry-run simulation, path traversal security testing, and empirical verification data from stress testing.

---

## 🚀 Quick Command Reference

```powershell
# 1. Check Worker Status & Configuration
python .agents/skills/ai-studio-worker/scripts/ai_worker.py --status

# 2. Query or Change Active Mode (auto | enabled | disabled)
python .agents/skills/ai-studio-worker/scripts/ai_worker.py --get-mode
python .agents/skills/ai-studio-worker/scripts/ai_worker.py --set-mode auto

# 3. Dry-Run an Analysis (Simulates payload assembly without calling API)
python .agents/skills/ai-studio-worker/scripts/ai_worker.py `
  --type RESEARCH `
  --files studio/ARCHITECTURE.md `
  --prompt "Analyze documentation completeness" `
  --dry-run

# 4. Execute Live Multi-File Audit
python .agents/skills/ai-studio-worker/scripts/ai_worker.py `
  --type AUDIT `
  --files studio/ARCHITECTURE.md studio/WORKER_PROTOCOL.md `
  --prompt "Check for protocol divergences."
```
