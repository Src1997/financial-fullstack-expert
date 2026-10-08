# Pack 07 — Crawler + Scripts + Worker

## When to load

- Adding a crawl source, ingest pipeline stage, or scheduled job.
- Deciding arq vs streaming for a background task.
- Working on dynamic scripts (sandbox, SDK, Admin publish/run).
- Debugging "crawler ran but nothing pushed" or locale pollution.

## Scope / Out of scope

**Scope:** Crawler boundary, pipeline stages, scheduler levels, three-process
enqueue-only model, post-write broadcast, bilingual ingestion, dynamic
scripts platform, arq/streaming worker rules.

**Out of scope:** WS channel contract mechanics (→ pack 05), DB design
(→ pack 03), verification commands (→ pack 11).

## Must know

### Boundary

| Layer | Crawler may | Crawler must not |
| ----- | ----------- | ---------------- |
| `app/crawler/` | Fetch, parse, normalize, upsert | Expose HTTP routes |
| `domains/*/router.py` | — | Import crawler modules |
| `domains/*/service.py` | — | Call upstream APIs directly |
| `repositories/` | Add `upsert_*` helpers | Perform HTTP |

Domain APIs read **PostgreSQL only** at request time — never fetch upstream
inside routers.

### Pipeline

```text
fetch -> parse -> normalize -> write -> (optional) broadcast
```

| Stage | Responsibility |
| ----- | -------------- |
| `sources/` | HTTP/SSR/WS adapters per vendor (`fastbull/`, `finnhub/`) |
| `normalizers/` | Upstream DTOs → existing `domains/*/schemas.py` (never parallel models) |
| `writers/` | Idempotent batch upsert / documented replace into `fin_*` |
| `realtime/post_write.py` | Post-commit broadcast helpers (`notify_*`) |

Fetch strategy: **JSON API first**; SSR `var xxx = {...}` via
`parse_var_json()`; raw HTML only as last resort. Structured fields must be
parsed before persistence; rich-text body may stay raw HTML only from trusted
editorial sources (frontend renders via `v-html` after `DOMPurify`).

### Scheduler levels + three processes

| Level | Cadence | Examples |
| ----- | ------- | -------- |
| L1 `realtime` | long-lived WS | `flash_news_ws` |
| L2 `high_freq` | ~30–120s | `flash_news` |
| L3 `mid_freq` | ~5–60 min | `calendar_events`, `headline` |
| L4 `low_freq` | hours | `cleanup_old_flash_news`, `locale_sentinel` |

| Process | Command | Responsibility |
| ------- | ------- | -------------- |
| API | `uv run uvicorn app.main:app ...` | REST + **enqueue only** |
| arq worker | `uv run arq worker.settings.WorkerSettings` | Scripts + batch crawl |
| streaming | `uv run python -m worker.streaming` | Long-lived WS jobs |

`enqueue-only` is the only mode: the API **never** runs streaming tasks, and
in-process batch fallback is removed. Full loop needs Redis `/1` + worker +
streaming. Health: `GET /health/detailed` (`crawler.mode`, `arq`, `streaming`).

### Post-write broadcast (never call the hub directly)

```text
job -> writer (transaction) -> commit -> notify_* -> broadcast_* -> WS / Redis
```

Helpers in `app/crawler/realtime/post_write.py`: `notify_headline_write`
(`headlines` card replace), `notify_flash_write` (incremental, gated by
`crawler.broadcast_flash_news`), `notify_calendar_write` (lang snapshot),
`notify_signals_write`. Lang-scoped pushes must pass `lang`; dispatch filters
by subscriber lang.

### Dynamic scripts platform

Product-level crawl jobs are **dynamic scripts** managed via the Admin API
(`SCRIPTS_ENABLED`, default `true`; disabled → scheduler falls back to
builtins). `app/crawler/builtins/` holds only registry callbacks + SDK glue
the sandbox invokes. Streaming jobs **cannot** be bound to dynamic scripts.

### Bilingual ingestion

Flash/calendar rows are Mode A dual-row (`lang` column + id suffix). FastBull
`langId` does **not** guarantee monolingual payloads — prefer content-encoded
language markers and run the sentinel after ingestion changes:

```bash
uv run python -m app.crawler audit --heal --json   # exit 1 on residual errors
```

## Do / Don't

| Do | Don't |
|----|-------|
| Register jobs in `app/crawler/registry.py` (`_wrap`) | Don't run streaming jobs inside arq (red line #7) |
| Use `notify_*` after commit for pushes | Don't call `realtime_hub` / `redis_bus` from jobs |
| Keep jobs idempotent + batched + rate-limited | Don't crash the scheduler on errors |
| Add new sources under `sources/<vendor>/` per `external/data-source.md` | Don't invent parallel response models |

## Verify

```bash
# From financial-api/
uv run python -m app.crawler list
uv run python -m app.crawler run flash_news
uv run python -m app.crawler status
uv run python -m app.crawler scheduler        # standalone scheduler
uv run python scripts/multiworker_smoke.py    # multi-worker smoke
```

## Touch map

| What | Path |
| ---- | ---- |
| Job registry | `financial-api/app/crawler/registry.py` |
| Scheduler | `financial-api/app/crawler/scheduler.py` |
| Post-write | `financial-api/app/crawler/realtime/post_write.py` |
| arq settings | `financial-api/worker/settings.py` |
| Streaming runner | `financial-api/worker/streaming/loop.py` |
| Scripts runtime queue | `financial-api/app/scripts_runtime/queue.py` |
| Crawler rules | `financial-api/docs/rules/modules/crawler/` |
| Worker rule | `financial-api/docs/rules/modules/worker.md` |
| Scripts rules | `financial-api/docs/rules/modules/scripts/` |
| Dynamic-scripts skill | `financial-api/docs/skills/project/financial-api-dynamic-scripts/` |

## SSOT links

| Doc | Path |
| --- | ---- |
| Crawler | `financial-api/docs/rules/modules/crawler/crawler.md` |
| Pipeline | `financial-api/docs/rules/modules/crawler/pipeline.md` |
| Scheduler | `financial-api/docs/rules/modules/crawler/scheduler.md` |
| Post-write | `financial-api/docs/rules/modules/crawler/post-write.md` |
| Sources | `financial-api/docs/rules/modules/crawler/sources.md` |
| Worker | `financial-api/docs/rules/modules/worker.md` |
| Scripts | `financial-api/docs/rules/modules/scripts/scripts.md` |
| External sources | `financial-api/docs/rules/external/data-source.md` |

## Related packs

- **05** — Realtime + i18n (channels the pushes feed)
- **03** — API architecture (domain/repository layers the writers feed)
- **11** — Verification (three-process smoke)
