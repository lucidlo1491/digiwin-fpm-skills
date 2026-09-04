# FPM v4 — Acceptance Criteria & QA Protocols

> What every stage must guarantee, and how each guarantee is verified.
> Referenced by PAIN-LOOP.md. If output violates a criterion here, it is a
> bug — file it in PAIN-INBOX.md.

## Per-stage acceptance criteria

### S1 · Ingest (watch.py)
- A file is processed only when **stable** (size unchanged 10 s + ffprobe
  reads a duration) — never a half-synced Drive file.
- **SHA-256 journal**: same content never processed (or billed) twice,
  regardless of filename.
- The original audio is **never opened for writing**. Ever.
- Any failure → macOS notification + journaled reason. Silence = bug.

### S2 · Transcription (transcribe.py, dual engine)
- **Both** engines must succeed. A single-engine transcript has no
  cross-check and must never masquerade as certified — hard error instead.
- Output must carry word-level timestamps; Soniox must carry word confidence.
- Glossary context (global + company terms) applied to the boosted engine.
- Every network call timeboxed; uploads retry 3× with backoff.

### S3 · Cross-check (crosscheck.py)
- Every span gets a tier: AGREE (≥85 fuzzy) / PARTIAL (60–84) / DISAGREE.
- Every DISAGREE carries the second engine's alternative reading — the
  reviewer must always see *what the other engine heard*.
- Agreement rate = char-weighted, computed for the certificate.

### S4 · QA certificate (qa_cert.py) — ships with EVERY transcript
- Grade A ≥85% agreement · B ≥75 · C ≥60 · D ≥45 · F below.
- **Fabrication checks are a hard veto** (grade F regardless of agreement):
  any turn timestamped beyond audio end + 120 s; any ≥20-char repetition run.
- Coverage gaps: unexplained silence >90 s beyond what the previous turn's
  text length can account for (≥4 chars/sec) → flagged, grade capped at C.
- Speaker-count sanity: outside 1–12 → capped at C.
- Grades are published as computed — never inflated (Chia Tai Tarn shipped
  as D/31 flags; that honesty is the product).

### S5 · Speaker identity (name_speakers.py, voiceprint.py)
- Content pass may auto-label ONLY Peter, only at high confidence, and the
  label is always marked "(auto)".
- Voiceprint suggestions: cosine ≥0.70, **one person ≤ one label per
  meeting** (uniqueness rule), score always shown.
- **Nothing is ever auto-assigned.** Assignment is Peter's click.

### S6 · Glossary (glossary.py)
- **FLAG, NEVER AUTO-REPLACE** — a known mishearing produces a hint with a
  one-click Apply; the text is never changed by the machine.
- Nothing enters approved state without Peter (corrections → pending queue).
- Every entry carries source + is deletable. Bad entry = one click to kill.

### S7 · Findings / person profiles (persons.py)
- Drafted from the person's **own turns only**; quotes verbatim, original
  language, never translated or paraphrased; thin evidence → fewer items,
  never padding.
- All findings born `approved=0`; only approved findings reach Obsidian.
- Every finding carries provenance: meeting slug + timestamp.

### S7b · Deal-fact extraction & decks (deals.py, deck_gen.py) — TIERED TRUST
- **Proposal is open; truth is gated.** Any graded meeting at or above the
  floor (not grade F, agreement ≥0.55) may PROPOSE deal facts — review of
  the transcript is NOT a precondition for motion (R2 policy decision).
- `from_reviewed` records the evidence tier (1 = source meeting reviewed,
  0 = graded-only); reviewing a meeting upgrades every fact it already
  proposed. The tier is provenance, never permission.
- **Verification is Peter's gate** — facts are born `proposed`; only his
  ✓ makes them `verified`. Verified state is independent of tier.
- **Decks mark every unverified fact** with an UNVERIFIED chip — the human
  gate moves to the ARTIFACT (Peter reviews the deck before a client sees it).
- **Delivery is capped**: a deck carrying >5 unverified facts cannot be
  marked delivered (409). Generation is never blocked by the cap — delivery is.
- Proper-noun guard: deck generation blocks on unconfirmed proper-noun
  suspects (ask-don't-guess); suspects resolve only by Peter.
- Stage is PETER-ONLY. No pipeline code ever sets an opportunity stage.

### S8 · Storage & export (export.py, library_db.py)
- `spans.json` is the living document; `transcript.md` is generated from it;
  **`_raw.md` (the engines' original) is written once and never touched**.
- Transcript + QA cert land beside the source audio in the client's Drive
  folder. Obsidian notes: search-before-create, never overwrite.
- Editor saves write back through the document model only — no direct file
  edits that could desync the index.

### S9 · Workbench UI (server.py, webui.py)
- Styling 100% from the DigiWin warehouse (semantic tokens, Carbon icons via
  the digiwin MCP) — improvised values are a defect, both themes included.
- Flag counts shown are OPEN flags (decrement as Peter resolves).
- Search must work for Thai/中文/English (trigram FTS — no word-boundary
  assumption).

## QA protocols — how the above is enforced

| # | Protocol | When it runs | Pass bar |
|---|---|---|---|
| 1 | **Regression gate** `run_v4.py --regression` | before ANY change is declared done | 3 ground-truth recordings: no fabrication, agreement >0.3, sane speakers — currently A/B/C |
| 2 | **Bake-off scorer** (6 dims: CER · proper nouns · diarization · code-switch · confidence calibration · fabrication veto) | engine-level changes; re-runnable from cached cells, free | challenger must beat incumbent on verified dims; fabrication = instant disqualify |
| 3 | **E2E proof** | pipeline changes | one REAL recording through the full unattended path (watcher→engines→cert→export→notify) |
| 4 | **Render-and-verify** | any UI change | Playwright screenshot READ with my own eyes (not just is_visible), both themes, 0 JS errors |
| 5 | **Human trust gates (tiered)** | always | proposal open above the grade floor; `from_reviewed` records tier; Peter's ✓ = the only path to verified; decks chip unverified facts; delivery capped at 5 unverified; flag workflow + approval queues (glossary, findings) |
| 6 | **Loud-failure invariant** | runtime, always | every failure = notification + journal entry; an undetected stall is itself a P0 bug |
| 7 | **Loop discipline** | every iteration | LOOP_LOG entry with what/proof; PAIN items move to Killed only with evidence |

## Known honest limits (not hidden)
- Thai-recording CER is measured against Gemini-derived references — only the
  Amy call has fully human-verified ground truth. The blind A/B pack
  (`bakeoff/AB_PACK.md`) exists to adjudicate; 3 of 24 samples judged so far.
- The findings drafter's quality is gated by approval, not measured — no
  ground truth exists yet for "good finding".
- Voiceprint threshold (0.70) is tuned on one cross-meeting pair; expect
  recalibration as assignments accumulate.
- Grades B/C on long Thai meetings partly reflect genuine engine disagreement
  on hard audio — the flag-ranking work (P3) addresses the review cost, the
  glossary bends the curve over time.
