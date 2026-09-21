# Antigravity On-Demand System Rule (BUBU & ARGUS)

## 1. Primary Mandate: User Task First
Whenever the user sends a message, FIRST fulfill their direct coding request, answer their question, or execute their task immediately. Do not block or delay their work for system onboarding.

## 2. Zero-Token Lazy Onboarding Check
At the end of your response, check if either of the following configuration files exists in the project root:
- `.ai-worker/config.json` (BUBU Context Worker)
- `.argus/config.json` (ARGUS Vision Shield)

### CRITICAL TOKEN GUARD:
DO NOT inspect `SKILL.md` files, DO NOT read python worker scripts, DO NOT check `--status`, and DO NOT initialize vision/LLM models BEFORE the user makes their choice. Perform ONLY lightweight file-existence checks (0 token cost).

### If configurations are missing, append this exact concise question to your answer:
> 📌 **Sistem Tercihleri (BUBU & ARGUS):**  
> Bu projede analiz ve vizyon koruma tercihleriniz henüz belirlenmedi:
> - **BUBU (LLM Context Worker):** `auto` (Akıllı Hibrit - Önerilen) | `enabled` (Sürekli Aktif) | `disabled` (Kapalı)
> - **ARGUS (Vision Shield):** `auto` (Akıllı Hibrit - Önerilen) | `local` (Sadece GPU) | `direct` (Standart / Kapalı)
> 
> Tercihinizi belirtebilirsiniz (Örn: *ikisi de auto*, ya da *bubu kapalı, argus auto*).

## 3. Post-Selection Execution (On-Demand Loading):
When the user responds with their preference:
- **If a system is DISABLED (`disabled` / `direct`):**
  - DO NOT read its `SKILL.md` or execute analysis scripts!
  - For BUBU: Write `{"worker_mode": "disabled"}` to `.ai-worker/config.json`.
  - For ARGUS: Write `{"mode": "direct"}` to `.argus/config.json`.
  - Save directly with ZERO token waste on documentation or setup.
- **If a system is ENABLED (`auto`, `enabled`, `local`):**
  - ONLY NOW activate the corresponding skill: read its `SKILL.md` to understand its capabilities.
  - For BUBU: Run `python .agents/skills/ai-studio-worker/scripts/ai_worker.py --set-mode <mode>`.
  - For ARGUS: Run `python .agents/skills/argus/scripts/argus.py --set-mode <mode>` (or via ARGUS MCP).
- **Once both config files exist:**
  - Silently follow the saved preferences.
  - Never prompt the user again in this project.
