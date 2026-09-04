---
name: review-transcript
description: >
  Interactive Transcript Report for FangPostMeet. Presents findings from every
  AI agent in 10 sections. Peter verifies, corrects, and refines before
  extraction and storage. The report IS the hologram — a queryable replica of
  the meeting. Use when: "/review-transcript", "review the transcript",
  "process this recording", "what did we find?"
---

# /review-transcript — Transcript Report

The single place where every AI agent presents its findings and asks Peter its questions.

## ★ Extraction North Star (read first — governs every section below)

The question every FPM transcript must answer is Peter's: **"Am I asking the right questions to move this case forward?"** The report is a **self-audit, not a data dump** — surface what we learned AND **what Peter failed to ask** (the gap = next-visit questions + a coaching signal).

Three governing principles (Peter 2026-07-24):
1. **Truth before essence (GIGO).** Verify the transcript is faithful to *what was actually said* BEFORE extracting any insight. A clean synthesis on a mis-heard transcript launders an error into a confident conclusion. → Section 1 (coverage/speakers) + the terminology gate below are a HARD pre-req for Sections 6–7.
2. **Terminology fidelity — same sound, different word, opposite meaning.** Flag homophones / mis-hearings; ASK (never guess) any load-bearing term. And **never upgrade loose speech into confident, formal-sounding claims** — that "tidying up" is a fabrication that can steer the deal wrong. **Flag-don't-fabricate.** (This is why Sections 2 / 5 / 5B gates exist — do not treat them as optional for graded transcripts.)
3. **Extract for reproducibility, loop-until-dry.** Every insight must pass: *"could Peter act on this on the next deal from our extraction alone?"* **Two passes, always:** thematic → framework-audit (did we flatten a keystone — e.g. 人機料法環?) → stop only when a pass surfaces nothing new that clears the reproducibility bar.

Full standing rubric (the 5-layer model + the 13-question qualification scorecard): memory `feedback_fpm_extraction_protocol`. VP-coaching value frameworks (人機料法環, value spine, objection bank): `feedback_vp_coaching_2026_07_23`.

## Input

`/review-transcript Precision Plastic` = optional company name filter. If empty, show all pending.

## Step 1: Find Pending Transcripts

```bash
cd /Users/peterlo/digiwin_automation
ls -la transcripts/*_qa_report.md 2>/dev/null
ls -la transcripts/*_transcript.md 2>/dev/null
ls -la transcripts/*_reviewed.md 2>/dev/null
```

A transcript is **pending review** if a `*_qa_report.md` exists but no `*_reviewed.md`.
If none found: "No transcripts pending review. Run the pipeline first."

## Step 2: Load Data + Declare Type + Pre-Extraction Context

Read the QA certificate. If the transcript is cross-checked (grade A/B, coverage OK)
→ proceed with the 10-section flow below. If it is a legacy single-engine transcript
→ fall back to word-level review (see bottom of file). If coverage FAILED or a
fabrication check fired → stop and tell Peter; do not review an untrustworthy transcript.

**Type is Peter-declared, not guessed.** Ask which of the 3 meeting types this is (and, if **mixed**, roughly where the handoff is — a single recording can start as a prospect meeting and become coaching after the client leaves):

| Type | Job | Lens → destinations |
|---|---|---|
| **1. Client / Prospect** | advance the deal | 六要素 + the **13-Q qualification scorecard** (Section 6B), tied to stage. North-star audit: *did Peter ask the right questions?* Output includes **the questions he failed to ask** = next-visit actions. |
| **2. VP Coaching** | transfer reusable expertise | consolidated **VP hub** → fan-out: deck content · objection bank · playbook rules · tools. (Append `docs/vp-coaching/`, playbook Part N, `feedback_vp_coaching_*`.) |
| **3. Channel / Pipe Review** | report deals + get coached | deal-status report + capture the VP's sprinkled coaching (routes like type 2) + his hard questions (which *become* the type-1 checklist). |

**Pre-extraction context (30-sec ask, BEFORE any extraction):**
- Type(s) + handoff point (if mixed).
- Which deals / companies are in it (facts tag to the right opportunity).
- **Coaching** → who's the coach (VP鄭 / Eddie / Chewie?) + which fan-out to prioritize this time.
- **Client** → who's in the room + roles, current stage, and **which of the 13 you already know** (so I hunt the rest, not re-ask).
- Any **named framework / term** to capture in full (a heads-up prevents flattening — the 人機料法環 lesson).

Then run the **two-pass loop-until-dry** (thematic → framework-audit) on each segment, routed to its type's lens.

## The 10-Section Transcript Report

Present ONE section at a time. Wait for Peter's response before proceeding.
Sections 1-4 can be auto-approved for Grade A/B transcripts ("fast mode").

---

### Section 0: Deal Context Before This Meeting

**Purpose:** Frame the entire review through a pipeline lens.

Pull current state from Obsidian + pipeline:
```bash
obsidian search query="COMPANY_NAME"
obsidian properties file="COMPANY_NAME"
```

Present:
```
COMPANY — Current State:
- Stage: X (N days, SLA: M days — K days remaining)
- 六要素: N/6 known (list which are known/partial/unknown)
- Missing for next stage: [list blocking elements]
- Last contact: YYYY-MM-DD
- Deal value: ฿X M (or NOT SET — ask Peter to estimate)

This meeting needs to discover: [specific elements for stage advancement]
```

If this is the FIRST meeting with a new company (no Obsidian note exists):
```
NEW COMPANY — First Meeting
No prior data exists. This review will establish the baseline.
All 六要素 start at "unknown."
```

**Peter interacts:** Confirms current state. Enters deal value if missing.

---

### Section 1: Transcription Quality

**Source:** QA report file (`*_qa_report.md`)

Present:
```
Transcription Quality: Grade [A/B/C/D/F]
- Duration: X min, Y turns, Z speakers
- Coverage: N% (gaps if any)
- Hallucination zones: N found
  [list each with timestamp + what was removed]
- Seam zones: N (where chunks joined)
  [list timestamps to spot-check]
```

**Fast mode (Grade A/B):** "Grade A — auto-approving. Say 'wait' if you want to review."

**Peter interacts:** Confirms hallucination removals. Listens to seam zones if desired.

---

### Section 2: What I Changed

**Source:** Glossary CorrectionLog + Industry ValidationResult

Check for saved reports:
```bash
ls extractions/*_reports.json 2>/dev/null
```

If reports exist, read the validation section. Otherwise, summarize from QA report.

Present:
```
Corrections Applied: N replacements
- Thai compounds joined: X
- ERP terms fixed: Y
- Items I'm not sure about:
  1. [timestamp] "TERM" — context suggests CORRECTION. Which is it?
  2. ...
```

**Peter interacts:** Answers uncertain corrections. Applied to transcript before extraction.

---

### Section 3: Who's in the Room

**Source:** QA report speaker analysis + `transcripts/` directory (for cross-meeting recall)

```bash
grep -o "SPEAKER_[0-9]*" "transcripts/FILENAME_transcript.md" | sort | uniq -c | sort -rn
```

#### Step 3a — Pull the recent-attendee reference list (memory aid, NOT a default)

Before showing the SPEAKER_NN prompt, look up who has appeared in past transcripts of this same company. This uses the existing transcript files as the implicit "cache" — they are source of truth (CLAUDE.md hard rule: never deleted), so a stateless runtime grep beats any persistent cache.

1. **Identify the company key** from Section 0 (e.g., `BFC`, `Mazuma`, `Distributor_Office_5-6`, `Dong_Woo`).
2. **Find prior transcripts** for the same company:
   ```bash
   ls -t transcripts/*.md 2>/dev/null | grep -iE "<company-slug>" | head -5
   ```
   Match is filename-substring case-insensitive. Skip the current transcript itself. Also include `_reviewed.md` if present (preferred — those have real names already mapped).
3. **Extract human speaker names** from the 2 most recent prior transcripts (by mtime). Look for lines matching `^\[\d+:\d+(:\d+)?\]\s+([^:]+):` and collect the captured speaker label IF it is NOT one of:
   - `Speaker \d+` (engine speaker label, still unmapped to a person)
   - `SPEAKER_\d+` (obsolete local-diarization label — legacy transcripts only)
4. **Optional fallback**: if the corresponding Obsidian Meeting note exists (e.g., `~/Peter/Meetings/<company> *.md`), read its YAML `attendees:` frontmatter list and union into the candidate set.
5. **Display ONCE** above the speaker prompt as a non-clickable reference:
   ```
   📒 Recently in this company:
   [Khun Nueng, Khun Bik, Khun Ek, Peter, 黃小平]
   (memory aid only — Peter still types each name below)
   ```
6. **CRITICAL — do NOT pre-fill speaker name fields, do NOT auto-suggest.** The reference list is a cognitive prompt to trigger recall, not an auto-suggest with confidence. The clarify gate stays intact — Peter types each name himself.
7. If the company has **0 prior transcripts** (first meeting), skip Step 3a entirely (no list shown). New-company transcripts behave exactly as today.

#### Step 3b — Speaker stats + manual mapping (existing flow)

Present speaker stats with sample utterances. Ask Peter to map each:
```
N speakers detected:
- SPEAKER_01: X turns (Y%) — "sample text..."
- SPEAKER_07: X turns (Y%) — "sample text..."
Who is each speaker? (Name + Role: 守門員/決策者/核決者)
```

**Peter interacts:** Maps speakers to names and roles. The reference list from Step 3a is consultative only — Peter remains the source of truth for speaker identity.

**After mapping: start extraction speculatively in background** to avoid dead wait.
Do NOT tell Peter about this — just start it. If corrections in Sections 4-5 change
anything material, re-run extraction with corrected data.

> **Design note** (added 2026-05-06 after voice-enrollment red-team rounds): the original instinct was to build a voice fingerprinting library + new T8.5 stage in FPM (~20-28 hr work). Two rounds of red team review reduced it to this 5-line stateless prompt change. Past transcripts are already the canonical record of who spoke; grep at runtime captures 80% of the recall benefit at <1% of the cost, with zero PDPA exposure, zero state to maintain, and zero clarify-gate erosion. See `~/.claude/plans/wiggly-munching-eclipse.md` for the full reasoning trail.

---

### Section 4: Business Content

**Source:** Content tagger report (from `_reports.json` or run `tag_transcript()`)

Present:
```
Business density: N%
- Business turns: X (N%) — sales, technical, requirements
- Noise turns: Y (M%) — greetings, small talk
- Noise will be stripped before extraction.
Want to review what I'm stripping? (y/n)
```

**Peter interacts:** Approves or reviews noise. Usually just approves.

---

### Section 5: Things I Need You to Check

**Source:** Transcript Verifier

If not already run:
```python
from pipeline.transcript_verifier import verify_transcript
items = verify_transcript(transcript_text, company, speakers_str)
```

Present each flagged item one at a time:
```
N items flagged for verification:
1. [FACTUAL] "claim" — timestamp. Is this correct?
2. [TECHNICAL] "term" — context. Same thing or different?
3. [SPEAKER] "quote" — who said this?
```

**Peter interacts:** Answers each flag.

**After Section 5:** If corrections were made, apply them to transcript. If extraction
was running speculatively, check if corrections invalidate it. If so, re-run.

---

#### Section 5B: Proper-Noun Verification (auto-detected, code-enforced)

When `run_distribution()` raises `ProperNounVerificationError`, this skill catches
it and renders the structured `UnverifiedEntity` list as ASK questions for Peter,
then re-runs distribution after each confirmation.

**Trigger:** the exception fires whenever `pipeline/proper_noun_guard.py` finds a
structured entity field (e.g., `next_actions[N].owner`, `competitive[N].vendor`,
`participants[]`) referencing a proper noun NOT marked `confirmed_by: peter` in
Obsidian Contact frontmatter.

**Detection scope (structured fields only — no full-text regex):**

| Item type | Fields scanned | Entity type |
|---|---|---|
| `participants[]` | each string | person |
| `next_actions[]` | `owner` | person |
| `waiting_for[]` | `who` | person |
| `agenda_items[]` | `discuss_with` | person |
| `projects[]` | `people_involved[]` | person |
| `customer_questions[]` | `asked_by` | person |
| `prep_items[]` | `for_whom` | person |
| top-level | `company` | company |
| `competitive[]` | `vendor` / `competitor` | company |

**Filter rules (auto-skip — no ASK):**
- "Speaker N" labels (already caught by speaker gate)
- Empty / "Unknown" / "TBD" / "n/a"
- Self-allowlist: `{"Peter", "Peter Lo", "ปีเตอร์"}`
- The meeting's own `company` (deal context already verified upstream)

**Cap: 5 ASK questions per session.** If more than 5 unverified entities surface
in one transcript, prioritize:

1. **Family-status claims** (highest — wrong family attribution = wrong strategy)
2. **核決者 / Decision-maker candidates** (next visit is on the line)
3. **Companies** (Peter's reference companies — Calcomp/Foxconn/etc.)
4. **Products / brand names**
5. **Junior contact roles** (lowest)

Beyond 5: defer remaining entities to next session — DO NOT auto-promote or queue
indefinitely.

**Per-question format Peter sees:**

```
🔴 Section 5B — Proper Noun Verification (3 of 5)

[1/3] PERSON in next_actions[2].owner

Plaud transcribed: "K'Chirayut"
Filename it would create: Contacts/K'Chirayut (Mazuma).md

Verify:
  - Full name (incl. last name)?
  - Family status (Durongdej / Sirithianchai / other)?
  - Role at this company?
  - Source: 【會議】 / 【DBD】 / 【公開】 / 【推論】?

Answer or "skip" (will defer to next session).
```

**After confirmation: write Contact frontmatter:**

```yaml
---
name: K'Chirayut Sirithianchai
thai_name: จิรยุทธ สิริเทียนชัย
company: Mazuma (Thailand)
role: Internal IT/Operations / system architect
family_status: NOT_FAMILY
plaud_misheard_as: ["K'Chirayut Mazuma", "K'Chirayut Durongdej"]
confirmed_by: peter
confirmed_at: 2026-04-30
source: 【會議】
type: contact
---
```

**After all 5 (or fewer) confirmations: re-run distribution:**

```python
try:
    items, errors = run_distribution(extraction, ...)
except ProperNounVerificationError as e:
    answers = ask_peter_about(e.unverified[:5])  # cap at 5
    write_obsidian_frontmatter(answers)
    items, errors = run_distribution(extraction, ...)  # retry
```

**Emergency bypass:** `ENABLE_PROPER_NOUN_GUARD=0` env var — logs warning and
skips the gate. Use sparingly; defeats the fidelity guarantee.

**Source-tag system (single source of truth, per VP Cheng's playbook +
`feedback_osint_source_tagging.md`):**

- `【DBD】` — confirmed via DBD lookup
- `【電話】` — confirmed verbally by Peter on phone
- `【會議】` — confirmed verbally by Peter in meeting
- `【公開】` — confirmed via public source
- `【推論】` — inferred but explicitly accepted as best-guess

**Why this exists:** the point isn't glossary management — it's FPM transcript
fidelity for VP's "time machine" coaching. When proper nouns are wrong, VP can't
accurately coach Peter on what was said. This is Phase 1 of the fidelity rebuild
— see `/Users/peterlo/.claude/plans/temporal-kindling-valiant.md` for Phase 2
trigger conditions (speaker mapping, hallucination cleanup, timestamp validity,
Thai over-segmentation).

---

### Section 6: Extraction Results + Stage Assessment

**Source:** E1-E5 parallel extractors + Obsidian prior state

If extraction hasn't completed yet, wait. Show: "Running extraction with your corrections..."

#### 6A: Executive Summary (bilingual)

Show E1 summary in both English and Chinese (for VP Cheng):
```
Executive Summary:
[English summary]

摘要:
[Chinese summary — translate or use ZH extraction if available]
```

#### 6B: Stage Advancement Blockers

This is the MOST IMPORTANT subsection. Show the 六要素 delta:

```
Stage: [BEFORE] → Recommended: [AFTER]

六要素 Delta (before → after this meeting):
| Element | Before | After | Changed? |
|---------|--------|-------|----------|
| Timeline | unknown | partial | ✅ NEW |
| Budget | unknown | unknown | — |
| ... |

To advance [CURRENT]→[NEXT], you still need:
1. [Element] — Ask: "[specific question]"
2. [Element] — Ask: "[specific question]"
```

Show Thai soft signals prominently:
```
⚠ Thai soft signals detected:
- "phrase" — [interpretation] (confidence: high/medium/low)
```

**★ 13-Question Qualification Scorecard (Client meetings only — the VP's recurring hard questions).**
This is the operational form of the north-star: *did Peter ask what he needs to know this deal is workable?* Auto-fill each from the transcript (✅ answered-in-meeting / ⚠️ partial / ❌ not asked). **❌ / ⚠️ items = the questions Peter failed to ask → emit them as Section 6E next-visit actions.** `⚠needs-boss` = usually only the owner can answer (not IT/procurement) — these are the ones Peter typically can't get.

```
                                                                   in transcript?
A. 立案三要項 gate (unanswered here = why this deal is NOT C1)
  1. 非做不可 reason — landed on a NUMBER? (not "溝通不良")        [ ✅ / ⚠️ / ❌ ]
  2. ⚠needs-boss  核決者 identified / met? (real EB, not 守門員)    [ ✅ / ⚠️ / ❌ ]
  3. ⚠needs-boss  提案 / go-live date — CUSTOMER-stated?           [ ✅ / ⚠️ / ❌ ]
  4. ⚠needs-boss  real budget + timing (this year / next)?         [ ✅ / ⚠️ / ❌ ]
B. 摸底 (basic due diligence — 別全開火鍋的你也簽)
  5. their customers' industry MIX?                                [ ✅ / ⚠️ / ❌ ]
  6. scale (revenue / headcount / # customers)?                    [ ✅ / ⚠️ / ❌ ]
  7. ⚠needs-boss  profitable? (contradiction-check: 賺錢為何換GM / 沒賺錢談什麼ERP) [ ✅ / ⚠️ / ❌ ]
C. 非做不可 depth (dig two layers — [[feedback_vp_qualification_discipline]])
  8. ⚠needs-boss  quantified business impact (準時交單/良率/倉庫值)? [ ✅ / ⚠️ / ❌ ]
  9. supplier-mandate %-of-revenue test (3-5% ignore, 30-50% real)? [ ✅ / ⚠️ / ❌ ]
 10. 人機料法環 cost gap — can they compute true order cost? (only ERP totals it) [ ✅ / ⚠️ / ❌ ]
D. Competition / commercial
 11. competitor + tier?                                            [ ✅ / ⚠️ / ❌ ]
 12. contract type billable? (勞務 can't 開票)                      [ ✅ / ⚠️ / ❌ ]
 13. urgency — does it slip this month?                            [ ✅ / ⚠️ / ❌ ]

Stage tie-in: A1–A4 (立案三要項) not all ✅  → deal CANNOT claim C1 (Nova C1 gate = 立案三要項 3/3 + EB + KSF≥20). A ⚠needs-boss-heavy deal that only ever met IT/procurement is a C2 floor — say so, and the fix is "get in front of the boss," not more OSINT.
When Peter cannot go two layers down (VP's own question): fall back to the anchor→react value case — YOU build the number (呆滯庫存喊數字 / 人機料法環 cost gap, [[feedback_vp_coaching_2026_07_23]]), the client only reacts; the 「沒有」 IS the surfaced 隱性需求.
```

#### 6C: Participants (三角色)
```
三角色 identified:
- 核決者: Name (Role) — stance
- 決策者: Name (Role) — stance
- 守門員: Name (Role) — stance
```

#### 6D: Pain Points (with citations)
```
N pain points found. Top by severity:
1. [5/5] Description → Digiwin solution
   ↳ "Thai quote" [timestamp] [show full quote? y/n]
```

#### 6E: GTD Items

Show all 5 GTD types. All items go to **Google Tasks INBOX** (not context lists).
Each item tagged with which 六要素 element it targets.

```
Next Actions:
- ☐ [physical verb] [task] → targets: [element] [NA @Context]

Waiting For:
- ⏳ [who] — [what] — by [date] → targets: [element]

Projects:
- [PROJECT] [name]

@Agenda:
- [item for specific person/meeting]

Someday/Maybe:
- [idea for future]

Items marked [?] = low confidence. Review these.

Did I miss anything? (add items or confirm complete)
```

**Peter interacts:** Confirms, corrects, removes, or adds items.

---

### Section 7: 準備事項 Preparation Needed

**Source:** E6 Customer Questions + Weak Moments Analyzer

Read from `_reports.json` or `_customer_questions.json` and `_prep_needs.json`.

#### 7A: Customer Questions by Signal Type

```
N questions asked. Top 10 by importance:

🟢 Buying signals (thinking about implementation):
1. "question" → [six_element]

🟡 Information seeking (still evaluating):
2. "question" → [six_element]

🔴 Objections/concerns (risk signals):
3. "question" → [six_element]

Gap analysis: [element] has most questions (biggest uncertainty).
```

#### 7B: 準備事項 — What to Prepare

```
N areas where your answer needs strengthening:
1. "Question they asked" — Your answer was [weakness].
   → Prepare: [physical verb task] [NA @Context]

Category tracking (for improvement over time):
- pricing: N items
- technical_integration: N items
- ...
```

Each item Peter confirms → becomes a Next Action in Google Tasks Inbox.

**Peter interacts:** Reviews, confirms which prep items to create as tasks.

---

### Section 8: Storage

```
Ready to save:
- MySQL: meeting record, transcript, extraction, six_elements snapshot, prep categories
- Obsidian: Client note + Meeting note + Contact notes (with [[wikilinks]])
- Google Tasks INBOX: N items (GTD + prep, with [?] prefixes)
- Google Calendar: [hard-date items, if any]

Confirm? (y = save all / n = edit something / skip = don't save)
```

If Peter confirms:
1. Insert into MySQL `meetings` table + related tables
2. Create/update Obsidian notes with `obsidian create` / `obsidian append`
3. Push GTD items to Google Tasks via `run_distribution()`

---

### Section 9: Pipeline Summary

```
Pipeline After This Meeting:
- [COMPANY] moved: [BEFORE] → [AFTER] ✅ (or "no stage change")
- Deal value: ฿X M
- Weighted pipeline: ฿Y M (was ฿Z M)
- Coverage: Y / 150M target = Wx (need 5x)

Pipeline Health Check:
- ⚠ [company]: [issue] — OVERDUE
- ⚠ [company]: at E with 0/6 六要素 — disqualify?

Your improvement trend: (after 3+ meetings processed)
- Answer quality: X% strong across N meetings
- Biggest gap: [category] — focus prep here
```

---

## Important Rules

- **NEVER modify the original `*_transcript.md`** — it is the raw source of truth
- **`*_reviewed.md`** is the corrected copy
- **Speaker mapping is mandatory** before extraction
- **Level 2 review gate**: ALWAYS show extraction before distributing
- **Speaker gate**: >40% unmapped turns → extraction blocked
- **[?] prefix**: Low-confidence items get [?] in Google Tasks
- **準備事項 not "weak moments"**: Presentation framing matters (VP Cheng)
- **All GTD items → INBOX list**: Peter routes to context lists during daily clarify
- **Fast mode**: Grade A/B can auto-approve Sections 1-4 (ask Peter)
- Peter's word is final — if he says "skip", skip
- Present ONE section at a time — wait for response before next
- If Peter seems rushed: "Want fast mode? I'll auto-approve Sections 1-4."

## STT v2 Flow (Legacy)

If the QA report indicates STT v2 (Google Speech-to-Text):
1. Load [unclear] and [inaudible] markers
2. Walk through each flagged word
3. Apply corrections to _reviewed.md
4. Save glossary corrections
5. Run Phase 2 extraction if confirmed

Use `pipeline.qa_verify` and `pipeline.interactive_review` for STT v2.
