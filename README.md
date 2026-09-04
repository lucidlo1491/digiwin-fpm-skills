# digiwin-fpm-skills

The "skills" and helper definitions that teach an AI assistant (Claude Code) how to turn a recorded
sales meeting into a checked transcript, a report, and a to-do list. FPM is short for **FangPostMeet**
(放 post-meet: "after the meeting, let go and let the system handle it"). Written so anyone can follow,
even with no sales or AI background.

---

## 1. What is a skill? What is an agent?

Imagine hiring a very smart assistant who has read every book in the world but has never worked at your
company. A **skill** is a folder of instructions you hand them: *"When I ask for this job, here is exactly
how we do it."* An **agent** is a job description for one specialist helper the assistant can call on: one
file, one job, its own rules. This repo has one skill (the human review step) and seven agents (the
specialists that do the automatic work).

## 2. The job FPM does

You record the meeting on your phone and drop the file in a folder. FPM then produces:

1. A **transcript it is willing to sign**. Two different speech-to-text engines listen independently; their
   answers are compared sentence by sentence. Where they agree, the words are trusted. Where they
   disagree, the transcript says so instead of guessing.
2. A **quality grade**, A to F, with a list of the exact moments a human should check by ear.
3. A **10-section report** of what happened: who was there, what they are unhappy about, what they asked,
   what we promised, where the deal stands.
4. A **to-do list** pushed into Google Tasks and Calendar. Anything the system is unsure about carries a
   "?" so the person can decide.

Then the human review step: the salesperson reads the report, fixes names and numbers, and approves it.
Only then do the to-dos flow out. Nothing is trusted downstream until a person has looked.

## 3. The seven agents, in the order they run

| Agent file | Job, in plain words |
|---|---|
| `agents/fpm-orchestrator.md` | The manager. Runs the others in order, keeps notes, restarts a step that failed without redoing the ones that worked. |
| `agents/fpm-transcription.md` | Makes the certified transcript: two engines, cross-checked, with a rule that refuses to invent words and a check that the transcript covers the *whole* recording (a transcript that quietly stops at minute five is rejected even if those five minutes look perfect). |
| `agents/fpm-thai-language.md` | Fixes the Thai word-splitting mistakes speech-to-text makes. Mechanical only; it never changes meaning. |
| `agents/fpm-industry-vocab.md` | Knows factory-software words and catches the engines mishearing them ("SMS" that should be "sMES", "percentage win" that should be "Digiwin"). |
| `agents/fpm-qa-grader.md` | Grades the transcript A–F, lists what a human must check, and decides whether it is trustworthy enough to continue. |
| `agents/fpm-extraction.md` | Reads the approved transcript and pulls out to-dos, promises, the deal's status against the six-element checklist, and competitor mentions. |
| `agents/fpm-distribution.md` | Files everything: tasks into Google Tasks, dates into Calendar, files into the customer's Drive folder, and a daily digest. |

## 4. The human step: `skills/review-transcript/`

`/review-transcript` opens the 10-section report as an interactive review. The salesperson walks through
it section by section, corrects, asks questions from different angles, and approves. The report is the
"hologram" of the meeting: a replica you can walk around and question. Everything downstream (tasks,
deal stage, customer notes) is generated only from the approved version.

## 5. What makes it careful

- **Two engines, not one.** A single engine will happily invent a plausible sentence. Two engines disagreeing
  is the signal that something needs a human ear.
- **Coverage check.** The grade judges what was transcribed; a separate check makes sure the transcript
  spans at least 90% of the recording's length.
- **The deal's stage is never changed by software.** Only the salesperson moves a deal forward, and it is logged.
- **Originals are sacred.** Recordings and raw transcripts are never edited or deleted.
- **Unsure means "?"**, not silence and not a guess.

## 6. Files

| File | What it is |
|---|---|
| `skills/review-transcript/SKILL.md` | The human review procedure |
| `agents/fpm-*.md` | The seven specialist job descriptions above |
| `supporting/fpm_v4-PRODUCT.md` | The product decisions behind FPM: what it is for, what it must never do |
| `supporting/fpm_v4-RUNBOOK.md` | How the engine is run day to day (services, recovery, what to do when a step fails) |
| `supporting/fpm_v4-QA-PROTOCOL.md` | How transcript quality is judged and gated |
| `supporting/sales-funnel.md` | The deal stages and the six-element checklist the extraction agent scores against |

## 7. Install and use

1. Install Claude Code.
2. Copy `skills/review-transcript/` to `~/.claude/skills/` and `agents/*.md` to `~/.claude/agents/` (or the
   same folders inside your project).
3. The agents drive an engine that is *not* in this repo (the Python program that calls the speech-to-text
   services and stores results). They also need API keys for two speech-to-text services and access to
   Google Tasks, Calendar and Drive. The runbook in `supporting/` explains the setup.

## 8. Glossary

| Word | Plain meaning |
|---|---|
| **FPM** | FangPostMeet: the after-meeting pipeline |
| **Transcript** | The written-out words of a recording |
| **ASR / speech-to-text** | Software that turns speech into text |
| **Diarization** | Working out who is speaking when |
| **Six elements / 六要素** | The six things you must know before a deal is real: need, budget, decision-maker, timing, competition, our fit |
| **Deal stage (E → D → C2 → C1 → B → A)** | How far a deal has come; only a human moves it |
| **GTD** | "Getting Things Done": turning everything into clear next actions |
| **sMES / eMES** | Two of the company's factory-floor products, often misheard by engines |

*Private repository. DigiWin Thailand, 2026.*
