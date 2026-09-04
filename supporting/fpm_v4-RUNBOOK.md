# FPM RUNBOOK — every operation is ONE command

> Harness contract: an operator (human or low-tier model) should never need
> to reason about the pipeline — only pick the row, run the command, and
> check the gate. All commands run from `/Users/peterlo/digiwin_automation`.
> PY = `.venv/bin/python3.13`

## Daily operations (usually automatic — listed for manual override)

| Want | Command | Proof it worked |
|---|---|---|
| Process new recordings now (don't wait 5 min) | `PY fpm_v4/watch.py` | notification per recording; row in Workbench |
| Re-try failed recordings (after credit top-up) | `PY fpm_v4/watch.py --requeue-failed` then `PY fpm_v4/watch.py` | failures leave the journal; notifications |
| Ingest the legacy local recordings | `PY fpm_v4/ingest_local.py --all` | notification per recording (stops at credit wall, journals rest) |
| Translate all Thai meetings (EN) | `PY fpm_v4/translate.py en` | `## Translation` section in each transcript .md |
| Translate to Chinese instead | `PY fpm_v4/translate.py zh` | same, 中文 |
| Mirror library → MySQL | `PY fpm_v4/mysql_mirror.py` | `mysql digiwin_osint -e "SELECT COUNT(*) FROM meetings WHERE fpm_slug IS NOT NULL"` |
| Send the weekly digest now | `PY fpm_v4/digest.py` | notification + today's Obsidian daily note |
| Fuse NAMED speakers from a 2nd-device transcript (Plaud) | `PY fpm_v4/witness.py <slug> "<reference.txt>"` | `out/work/<slug>/speakers_proposed.json` — per-label ✓/? proposals + per-SPAN names for merged labels. PROPOSED only; apply in review |
| TRIANGULATE merged speakers (text witness × voiceprints) | `PY fpm_v4/triangulate.py <slug> ["<reference.txt>"]` | same file gains `triangulated` — per-span verdicts agree/conflict/text-only/voice-only. Value grows as more people are voiceprint-enrolled via review confirms |
| **AUTO-TRIGGER**: drop the Plaud .txt NEXT TO the audio (same folder, similar filename) BEFORE processing | (nothing — the watcher does it: `find_reference` fuzzy-matches ≥0.6 + must parse ≥5 named turns) | watch log line `witness fusion (<ref>): N span proposals {...}`; anonymous/unparseable exports are ignored automatically |
| Check credit balance now | `PY -c "import sys;sys.path.insert(0,'fpm_v4');import credits;print(credits.check(force=True))"` | numbers print |

DB uses WAL — backups must use `sqlite3 .backup`, never copy the bare .db (misses -wal sidecar).

## R2 operations — opportunities & decks

| Want | Command | Proof it worked |
|---|---|---|
| Generate a deck | Workbench `/opp/<id>` → ⚡ Generate (the only primary button) — or `PY -c "import sys;sys.path.insert(0,'fpm_v4');import library_db as db, deck_gen; print(deck_gen.generate(db.conn(), <opp_id>))"` | new immutable version row; PDF under `fpm_v4/out/decks/`; UNVERIFIED chips on unverified facts |
| Deliver a deck | `/opp/<id>` deck row → `⬇ Download PDF · marks delivered` (POST delivered, then download). >5 unverified facts → 409 with reason; verify facts first | `delivered_at` set on the version; deck chip shows ✓ delivered |
| Draft a re-engagement touch | `/opp/<id>` → `✉ Draft re-engagement` (copy-ready textarea) — or `PY -c "import sys;sys.path.insert(0,'fpm_v4');import library_db as db, touch_draft; print(touch_draft.draft(db.conn(), <opp_id>))"` | text built from last meeting's facts |
| Log a touch | `/opp/<id>` touch buttons (all POSTs need header `X-FPM: 1`) | cadence chip resets; `touches` row |
| Check the 48h KPI | `PY -c "import sys;sys.path.insert(0,'fpm_v4');import library_db as db, opportunities as opp; print(opp.kpi_48h(db.conn()))"` | dict of stage moves vs delivered-deck hits (meeting-anchored) |
| Stage moves | **Peter only**, via the `/opp/<id>` stage select (confirm dialog starts the 48h clock). No code path sets stage. | `stage_history` row |

### Restore from nightly backup (03:00 → `~/FPM-Backups/fpm-YYYY-MM-DD.tgz`)
1. Stop services: `launchctl bootout gui/$(id -u)/com.peterlo.fpmv4.server` and same for `.watch`.
2. Untar: `mkdir -p /tmp/fpm-restore && tar -xzf ~/FPM-Backups/fpm-<date>.tgz -C /tmp/fpm-restore`.
3. Restore the DB (it was taken with `sqlite3 .backup`, safe to copy back):
   `cp /tmp/fpm-restore/library.db ~/.fpm_v4/library.db` (or `sqlite3 ~/.fpm_v4/library.db ".restore /tmp/fpm-restore/library.db"`); remove stale sidecars `rm -f ~/.fpm_v4/library.db-wal ~/.fpm_v4/library.db-shm`.
4. Restart: `launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.peterlo.fpmv4.server.plist` (and `.watch`), or `launchctl kickstart -k` if still loaded.
5. Verify: `http://localhost:7777` loads; `PY fpm_v4/run_v4.py --regression` 3× PASS.

## After ANY code change (the gates — non-negotiable)

| Gate | Command | Pass bar |
|---|---|---|
| Regression | `PY fpm_v4/run_v4.py --regression` | 3× PASS (amy A / tn_demo B / precision C) |
| UI render check | screenshot via Playwright, READ the PNG | both themes, 0 JS errors |
| Pipeline E2E | drop a recording in a client folder, wait ≤6 min | notification with grade |

## Services (launchd — survive reboots)

| Service | Label | Restart |
|---|---|---|
| Watcher (5 min) | com.peterlo.fpmv4.watch | `launchctl kickstart -k gui/$(id -u)/com.peterlo.fpmv4.watch` |
| Workbench :7777 | com.peterlo.fpmv4.server | `launchctl kickstart -k gui/$(id -u)/com.peterlo.fpmv4.server` |
| Monday digest | com.peterlo.fpmv4.digest | (calendar-triggered) |
| Nightly backup 03:00 | com.peterlo.fpmv4.backup | (calendar-triggered) → `~/FPM-Backups/fpm-YYYY-MM-DD.tgz`, log `fpm_v4/out/backup.log` |

plists are NOT repo-tracked. Backup agent (re)install one-liner — write the plist
(ProgramArguments `["/bin/zsh", "/Users/peterlo/digiwin_automation/fpm_v4/backup.sh"]`,
StartCalendarInterval Hour 3 / Minute 0, StandardOut+ErrorPath both
`/Users/peterlo/digiwin_automation/fpm_v4/out/backup.log`) then:
`cp <plist> ~/Library/LaunchAgents/com.peterlo.fpmv4.backup.plist && launchctl load ~/Library/LaunchAgents/com.peterlo.fpmv4.backup.plist`

## Where things live

| Thing | Path |
|---|---|
| Workbench | http://localhost:7777 |
| Library DB / journal / tokens | `~/.fpm_v4/` · Tasks token `~/.pipeline/google_tasks_token.json` |
| API keys | `~/.pipeline/.env` |
| Per-meeting working files | `fpm_v4/out/work/<slug>/` (spans.json = living doc, `_raw.md` = untouchable) |
| Logs | `fpm_v4/out/*.log` |
| Pain inbox / ledger / QA rules | `fpm_v4/PAIN-INBOX.md` · `PAIN-LOOP.md` · `QA-PROTOCOL.md` |

## Runtime LLM usage (the low-token design)

| Job | Model | Tokens/meeting | Why it's safe on a cheap model |
|---|---|---|---|
| Transcription | none (ASR: Soniox+ElevenLabs) | 0 | deterministic engines |
| QA / flags / ranking / search / voiceprints | none (code) | 0 | pure computation |
| Speaker naming | gemini-2.5-flash | ~2k | suggestion-only, Peter confirms |
| Findings draft | gemini-2.5-flash | ~15k | verbatim-quote rules + approval queue |
| Thai translation | gemini-2.5-flash, thinking OFF | ~1k/min audio, cached forever | glossary-pinned, span-batched, per-span cache |
| Deal-fact extraction | gemini-2.5-pro | ~10k | grade floor (no F / agreement ≥0.55) + everything lands `proposed` tier-0; Peter's ✓ is the only path to verified |
| Deck narrative | gemini-2.5-flash | ~5k | verbatim-evidence prompt + noun guard + UNVERIFIED chips + delivery cap |

**Tests:** run under `.venv/bin/python3.13` ONLY (system python lacks opencc).

## Service slimdown (Peter, 2026-06-11)
Only the SERVER runs (:7777, launchd KeepAlive). Watcher/backup/digest
UNLOADED — plists parked in ~/Library/LaunchAgents/disabled/ (re-enable:
`cp` back + `launchctl load`). Rationale: low audio volume — no point
polling Drive every 5 min. Processing still works: the upload dropzone
kicks watch.py once per drop. Manual equivalents when needed:
- process new audio:  .venv/bin/python3.13 fpm_v4/watch.py
- backup now:         zsh fpm_v4/backup.sh
- digest now:         .venv/bin/python3.13 fpm_v4/digest.py
NOTE: no automated DB backup anymore — run backup.sh before risky changes.
