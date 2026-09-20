# Cache & Quota Architecture (QUOTA_AND_CACHE.md)

This document describes the multi-factor SHA-256 caching engine and the local quota accounting mechanism implemented in `ai_worker.py`.

---

## 1. Multi-Factor SHA-256 Caching Engine

Traditional file caches only check individual file timestamps or contents. However, an analysis of the same source file varies drastically depending on the specific prompt, the model utilized, and the assigned task type (`DEBUG` vs `AUDIT`).

BUBU computes a **composite 6-factor SHA-256 hash** as the cache key:

```text
CACHE_KEY = SHA256(
    WORKER_VERSION
    + MODEL_NAME
    + TASK_TYPE
    + PROMPT_STRING
    + SORTED([rel_path_1:sha256_1, rel_path_2:sha256_2, ...])
    + MAX_FILE_BYTES
)
```

### Cache Invalidation Triggers
A cache hit occurs **only** when all six parameters are identical. The cache is automatically invalidated if:
1. **Any source file changes:** Even a 1-character edit alters that file's SHA-256 hash.
2. **The user's prompt changes:** Asking a different question triggers a fresh analysis.
3. **The task type changes:** Switching from `ANALYZE` to `AUDIT` requires different analytical reasoning.
4. **The model changes:** Switching models (e.g. from `gemini-3.6-flash` to another model) invalidates older responses.
5. **The worker engine updates:** Bumping `WORKER_VERSION` invalidates stale schemas.
6. **Force flag:** Running with `--force` bypasses the cache entirely.

### Performance & Offline Serving
- **Sub-Second Response:** When a cache hit occurs, the worker serves the verified structured output in <0.5 seconds (measured ~82x faster than a full API round-trip).
- **Zero API Invocations:** Cache hits make zero network calls, consuming 0 tokens.
- **Offline Resilient:** If your network drops or your API key is temporarily unconfigured, BUBU can still serve previously cached analyses without error.
- **Storage & Integrity:** Cache entries are indexed in `.ai-worker/cache/index.json`. Disk writes use atomic temporary file replacement (`tmp_path.replace(file_path)`) to ensure integrity across concurrent processes.

---

## 2. Local Quota Tracker vs. Google Server Quota

> [!IMPORTANT]
> **Local Quota Tracker $\neq$ Google Server Quota.**
> The local tracker is a client-side safety mechanism designed to protect developers from runaway loops and rate limits. It does not replace or reflect live Google cloud server quotas.

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

### Local Accounting Ledger (`.ai-worker/quota/quota_tracker.json`)
Every outgoing API call updates a local ledger:
- Current calendar date (`YYYY-MM-DD`).
- Total daily request counter.
- Estimated input and output token counts (reported by Gemini's `usageMetadata`).
- HTTP status code counters (`429`, `5xx`, `4xx`).
- Sliding window timestamps for the past 60 seconds (for RPM enforcement).

### Transient Error Handling (Exponential Backoff)
If Gemini responds with HTTP `429 (Rate Limit)` or `503/504 (Server Unavailable)`:
- **Attempt 1:** Wait 1.0 second and retry.
- **Attempt 2:** Wait 2.0 seconds and retry.
- **Attempt 3:** Wait 4.0 seconds and retry.
If all retries fail, BUBU smoothly falls back (`status: "fallback"`), ensuring Antigravity continues its work without being blocked.
