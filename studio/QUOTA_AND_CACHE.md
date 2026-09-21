# Cache & Quota Architecture (QUOTA_AND_CACHE.md)

This document describes the multi-factor SHA-256 caching engine and the local quota accounting mechanism implemented in `ai_worker.py`.

---

## 1. Multi-Factor SHA-256 Caching Engine

Traditional file caches only check individual file timestamps or contents. However, an analysis of the same source file varies drastically depending on the specific prompt, the provider utilized (`gemini` vs `openai_compatible`), the model utilized, and the assigned task type (`DEBUG` vs `AUDIT`).

BUBU computes a **composite 7-factor SHA-256 hash** as the cache key:

```text
CACHE_KEY = SHA256(
    WORKER_VERSION
    + PROVIDER_NAME
    + MODEL_NAME
    + TASK_TYPE
    + PROMPT_STRING
    + SORTED([rel_path_1:sha256_1, rel_path_2:sha256_2, ...])
    + MAX_FILE_BYTES
)
```

### Cache Invalidation Triggers
A cache hit occurs **only** when all seven parameters are identical. The cache is automatically invalidated if:
1. **The provider changes:** Switching from `gemini` to `openai_compatible` generates a distinct cache key, preventing cross-provider pollution.
2. **Any source file changes:** Even a 1-character edit alters that file's SHA-256 hash.
3. **The user's prompt changes:** Asking a different question triggers a fresh analysis.
4. **The task type changes:** Switching from `ANALYZE` to `AUDIT` requires different analytical reasoning.
5. **The model changes:** Switching models (e.g. from `gemini-3.6-flash` to `gpt-4o-mini`) invalidates older responses.
6. **The worker engine updates:** Bumping `WORKER_VERSION` invalidates stale schemas.
7. **Force flag:** Running with `--force` bypasses the cache entirely.

### Performance & Offline Serving
- **Sub-Second Response:** When a cache hit occurs, the worker serves the verified structured output in <0.5 seconds (measured ~82x faster than a full API round-trip).
- **Zero API Invocations:** Cache hits make zero network calls, consuming 0 tokens.
- **Offline Resilient:** If your network drops or your API key is temporarily unconfigured, BUBU can still serve previously cached analyses without error.
- **Storage & Integrity:** Cache entries are indexed in `.ai-worker/cache/index.json`. Disk writes use atomic temporary file replacement (`tmp_path.replace(file_path)`) to ensure integrity across concurrent processes.

---

## 2. Local Quota Tracker vs. Server Quotas

> [!IMPORTANT]
> **Local Quota Tracker $\neq$ Provider Server Quotas.**
> The local tracker is a client-side safety mechanism designed to protect developers from runaway loops and rate limits across any LLM provider. It does not replace or reflect live provider cloud server quotas.

### Why Maintain a Local Safety Budget?
1. **Loop Protection:** Prevents an automated agent loop from issuing hundreds of accidental requests in minutes.
2. **Predictable Usage:** Enforces user-defined daily request caps (`daily_request_budget`) and requests-per-minute (`rpm_budget`) limits.
3. **Graceful Handover:** When the safety threshold is reached, BUBU shifts to graceful fallback rather than crashing.

### Default Safety Budgets
Configurable via `.ai-worker/config.json` or environment variables:
```json
{
  "daily_request_budget": 1400,
  "rpm_budget": 12,
  "max_retries": 3
}
```

- **Daily Request Cap:** Tracks total calls executed in the current UTC day.
- **RPM Window:** Tracks requests in the trailing 60 seconds to prevent `429 Too Many Requests`.
- **Reset Logic:** Automatic reset when UTC date rolls over.
