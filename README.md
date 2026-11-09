# Felix Sells

**An outbound system where the outreach *is* the demo: Felix, an AI teammate, writes real estate teams about something true and current, never repeats himself, and asks for a short conversation.**

> Built at [Fello](https://hifello.com) as the primary engineer, with a small team. This repository is a write-up: the code is private, the thinking is here.

▶ **[Watch the walkthrough](https://drive.google.com/file/d/1KaB0AYvcEQvyQmJNzD0uJS7vyGJYmRan/view)**

---

## The problem

Real estate teams are flooded with generic cold outbound. The most credible way to prove an AI teammate can do inside-sales work is to have it do that work: timed to a real signal, in the contact's own channel (email and SMS), with no rep spending an hour on it.

## How it works

```
Signals (new listing, post, hiring page) ─┐
Contact-pass ledger (what was said, when) ─┼→ decide() → gates → send (email / SMS) → reply → ledger
Account memory (internal vs. speakable) ───┘
```

- **A ledger remembers every touch.** Per contact: what was said, on which channel, when. Felix never repeats an angle a contact, or a teammate at the same company, already saw.
- **The model chooses intent; code does everything else.** `decide()` emits one of nine zod-validated actions. Timezone math, touch spacing, email classification and channel policy are deterministic TypeScript. *"TZ arithmetic in TS, never the LLM."*
- **One audited gate in front of every send.** Do-not-contact first (cheapest check, highest legal stakes), then attempt cap, contact window, cooldown. Every evaluation writes an audit row, bypasses included.
- **An account governor coordinates the buying group.** It stops the owner getting a text the hour the lead gets an email. It rolled out **shadow → enforce** behind a flag, opt-in per company, with "no policy set = today's behavior, byte for byte" as the safety contract.
- **Two-tier memory with a privacy wall.** An internal tier informs reasoning but can never be spoken. A separate speakable tier reaches the contact only after a grounding verifier.

## The personalization engine

A Python/FastAPI + ARQ/Redis service ("SignalsEngine") finds a team's web and social presence from just a name and city, scrapes it, and **diffs by content hash before spending any model tokens**. It emits scored, source-backed signal cards with deduplication, which become per-contact "hooks."

When a hook quotes a prospect's own post, a **deterministic guard** (control-character stripping plus a 6-word shingle verbatim-overlap detector) runs at both hook creation and send time, even when the LLM verifier is off. Untrusted third-party text needs a code-level guard, not prompt instructions.

## How it's tested

- A **frozen eval suite** for `decide()`: abstention on missing facts, soft-objection reframing, silence-check timing, explicit do-not-contact. Separate evals cover the SMS stylist, hook writer and memory summarizer.
- A **live-agent simulation** verified one fix at 4/4 successful runs against 0/4 for the baseline.
- A **live A/B hook library** round-robins opener variants and tracks reply rate per variant in production.
- `pnpm verify` is one pre-push gate: typecheck, lint, unit tests, build, and a Playwright smoke of the operator console.

## Lessons I'd repeat

- **Append-biased LLM memory is usually an instruction bug, not a model bug.** The summarizer was told to "preserve all prior touchpoints," so a rescheduled meeting left both times standing. The fix made the structured booking the spine of the memory.
- **Absence is two different signals.** "Unknown" and "known to be empty" must be different values to a model.
- **Staging and production schemas drift.** A documented drift-check and promotion runbook caught it before it hurt.

## Stack

TypeScript · Next.js / Vite SPA · Supabase (Postgres, RLS) · Python/FastAPI · Twilio SMS · OpenRouter · Railway · Playwright · 1,000+ commits · 100+ migrations · 600+ test files

---

*Author: [Aryan Pasreja](https://www.linkedin.com/in/aryan-pasreja/) · AI engineer*
