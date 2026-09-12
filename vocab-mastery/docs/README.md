# Vocab Mastery — Product & Engineering Spec

This file is the source of truth for the product. Code must follow it. If code and this doc disagree, the doc is wrong **or** the code is unfinished — say which, then fix one of them.

**Status today:** Next.js 16 scaffold. Homepage is a static shell (`h1` / `textarea` / `button` TODOs). No data model, no persistence, no review loop, no AI. Everything below is the plan, not a description of running software.

**How to use this doc:** build only the current phase. Do not open a later-phase library until the current phase’s “Done when” is true.

---

## 1. Problem

English learners meet new words inside real text (articles, homework, subtitles). They highlight, forget, and never see the word again in a way that forces recall.

Generic flashcard apps make you type every card. Dictionary apps explain a word but do not remember *your* sentence. Vocab Mastery sits in the gap:

> Paste text you actually read → keep the useful words with *that* sentence → review until you can recall them.

If a feature does not serve capture or recall, it is not v1.

---

## 2. Who it is for

| Persona | Need | What we do *not* build for them (yet) |
|---|---|---|
| Solo learner (primary) | Save words from homework / articles and review on the phone | Classrooms, teachers, assignments |
| You (builder) | A small app you can finish and understand | A startup with billing, teams, i18n |

One user, one device, one list is enough to prove the product.

---

## 3. Product principle

**One loop:**

1. **Capture** — paste (or type) text, or add a word by hand.
2. **Keep** — store `{ term, meaning, source sentence }`.
3. **Recall** — see the term, try to remember, reveal, mark known / unknown.
4. **Repeat** — unknown cards come back; known cards wait.

Success is not “we called an LLM.” Success is: *yesterday’s words still exist after refresh, and you can quiz them in two minutes.*

---

## 4. Goals and non-goals

### Goals (v1)

- A learner can save words from a pasted passage and review them the next day without losing data on refresh.
- Every saved word keeps a sentence from the original text when one exists.
- The UI is usable on a phone-width screen (thumb reach, no hover-only actions).

### Non-goals (do not build until v1 works)

- Multi-user classrooms, social, comments, leaderboards
- Native iOS / Android apps (a PWA can come later)
- Perfect SuperMemo-2 before a working “know / don’t know” queue
- Supporting languages other than English-as-target
- Pixel-perfect marketing site, glassmorphism, or a component library
- SEO as a product strategy (this is a logged-in-or-local tool, not a content site)

---

## 5. Features

Priority is the engineering order. **Must** ships before **Should**. **Later** is allowed only when you feel a real pain the current design cannot solve.

### Must have (Phase A–B) — the product exists without these? No.

| Feature | Why ship it | Why this, not a fancier version |
|---|---|---|
| Paste text on the home screen | Matches the actual job: words come from a passage | Upload/PDF is slower to build and fails more; paste covers 80% |
| Manual add: term + meaning | You can use the app before any AI key exists | AI cannot be the only way to create a card |
| Vocab list: add, show, delete | CRUD on one collection is the spine of the app | Decks/folders split attention before you have 20 words |
| Flashcard: front = term, back = meaning + sentence | Recall is the name of the product | 3D flip is polish; a two-face card is the behavior |
| Known / unknown after reveal | Creates a queue without an algorithm paper | SM-2 needs history you do not have yet |
| Persist in `localStorage` | Refresh must not wipe homework | A database adds auth, RLS, hosting before you have a loop |
| Empty, error, and “too short” states | Real software handles “nothing pasted” | Happy-path-only UI trains you to ship demos |
| Keyboard: Space = flip | Review is a high-frequency action | Mouse-only review is tiring |

**Done when:** you can paste or type, save three words, refresh the tab, open review, mark one unknown, and see it again.

### Should have (Phase C) — makes it feel like *this* product, not Anki

| Feature | Why |
|---|---|
| **Keep the source passage** with the cards extracted from it | The differentiator: you review *your* article, not a global dictionary |
| **AI extract 5 / 10 / 15 words** from pasted text (server-side only) | Removes the boring typing once the loop already works |
| **User can uncheck words** before save | Models over-extract; the learner is the editor |
| **Example sentence from the passage**, not a generic dictionary example | Context is how vocabulary sticks |
| **IPA + pronounce** via Web Speech API (`en-GB` / `en-US`) | Audio without a paid TTS bill; good enough for v1 |
| **Due today** using a tiny interval (1d / 3d / 7d from known/unknown) | Beats “review everything every time” without SM-2 |
| **Search / filter the list** | After ~30 words, a flat list is unusable |
| **Highlight extracted terms** in the original passage | Closes the loop visually: you see what was pulled |

### Later (Phase D+) — only after C is in daily use

| Feature | Why wait |
|---|---|
| Full SM-2 (ease factor, repetitions) | Easy to get wrong; simple intervals teach you the data you will need |
| Fill-in-the-blank using the source sentence | Needs a stable sentence field first |
| Quick multiple-choice quiz | Needs at least ~8 cards so distractors exist |
| PDF / DOCX upload | Parsing, size limits, serverless timeouts; paste is enough until it is not |
| Domain filters (academic / casual / technical) | An AI nicety; does not unblock learning |
| Auth + cloud sync (Supabase or similar) | Needed when you have two devices or fear losing the browser profile |
| Decks / tags | Needed when one list is genuinely messy |
| Streaks, heatmap, weekly email | Motivation layer; can lie to you if the loop is weak |
| Export / import JSON or CSV | Backup and lock-in escape hatch; add when data matters |
| PWA “Add to Home Screen” | After mobile layout is actually good |
| Rate limits, usage caps | Needed when AI is public, not while it is only you |
| i18n / other target languages | Splits prompts, TTS, and UI; do not start here |

### Suggested differentiators (pick at most two for v1)

These are what make the site yours instead of “flashcards + Gemini.”

1. **Context lock** — the example on the card is always a sentence from *this* paste, never a random corpus sentence.
2. **Passage session** — “Review this article” quizzes only words from one paste, in the order they appeared.
3. **Learner-as-editor** — AI proposes; you approve. Never auto-save 15 cards without a confirm step.
4. **Two-minute review** — a home-screen number: “7 due today,” one tap into the queue. No dashboard chrome.

Do not build a marketing hero with a fake flipping card until the real review page works.

---

## 6. User flows

### Flow A — first-time, no AI (Phase A)

1. Open `/`.
2. See title, textarea, primary button.
3. Type one term + meaning (or paste a short paragraph and add words by hand).
4. See them on a list.
5. Go to `/practice`, flip, mark known/unknown.

### Flow B — extract from text (Phase C)

1. Paste up to a configured character limit (start at 5 000).
2. Choose how many words (5 / 10 / 15).
3. Wait with a loading state; on failure, show a retryable error (do not clear the paste).
4. See proposed cards (term, meaning, source sentence). Uncheck junk.
5. Save approved cards, attached to that passage.
6. Optional: jump to “Review this passage.”

### Flow C — due today (Phase C)

1. Home shows count of due cards.
2. Review only those cards.
3. Unknown → due again soon. Known → due later.

---

## 7. Data model (draw this before more UI)

Keep types in `lib/types.ts`. Persistence maps 1:1 to these objects.

```ts
type Passage = {
  id: string;
  text: string;
  createdAt: string; // ISO
};

type VocabItem = {
  id: string;
  term: string;
  meaning: string;
  ipa?: string;
  sourceSentence?: string;
  passageId?: string;
  createdAt: string;
  nextReviewAt: string; // ISO; equal to createdAt when new
  intervalDays: number; // start at 0
  // Phase D only:
  // easeFactor?: number;
  // repetitions?: number;
};
```

Rules:

- `id` = `crypto.randomUUID()`.
- Deleting a passage does not have to delete words in Phase A; decide later and document it.
- Do not add `userId` until auth exists. Until then, the browser profile *is* the user.

---

## 8. Architecture (phased — do not jump)

```
Phase A (now)
  Browser
    app/page.tsx          → compose the capture screen
    app/practice/page.tsx → review
    components/*          → form, list, card
    lib/types.ts
    lib/storage.ts        → localStorage, JSON parse guarded
    React state           → no Zustand yet

Phase C
  Same UI
    app/api/extract/route.ts  → server only; holds the API key
    lib/extract.ts            → prompt + parse JSON
    Web Speech API            → client, pronunciation

Phase D
  + Postgres + Auth when localStorage hurts (second device, or real users)
```

### Stack — what we use *now* vs *later*

| Layer | Now | Add when |
|---|---|---|
| App | Next.js App Router (already here) | — |
| UI | React + Tailwind v4 | Framer Motion only if a CSS flip is not enough |
| State | `useState` + a `lib/storage.ts` module both routes import | Zustand if prop-drilling or duplicated storage calls actually hurt |
| Data fetch | none | SWR/React Query when you have a real API to cache |
| AI | none | Route Handler + **SpaceXAI** (`XAI_API_KEY`, `https://api.x.ai/v1`). Key never in the client bundle. Prompt lives in `lib/prompts.ts`. Output must be strict JSON. |
| Audio | none | `speechSynthesis` in the browser |
| Auth / DB | none | Supabase (Postgres + Auth + RLS) when you need sync |
| Hosting | `npm run dev` | Vercel when you want a URL |

**Do not** call this repo a monorepo. It is one Next.js app. API routes are not a second package.

**Do not** start Git Flow (`develop`, `hotfix`). Use `main` + short `feature/*` branches. Rebase or merge; keep history readable.

### AI contract (when you reach it)

- Client sends `{ text, count: 5 | 10 | 15 }`.
- Server returns `{ items: { term, meaning, ipa, sourceSentence }[] }` or `{ error }`.
- Reject empty text, over-limit text, and invalid `count`.
- Never return the model’s raw prose to the UI; parse JSON on the server.
- If the model fails, the paste stays; the user can add words by hand.

---

## 9. Design spec

**Style:** calm, high contrast, content-first. Not glassmorphism. Flashy blur fights reading.

**Tokens**

| Token | Value | Use |
|---|---|---|
| Primary | `#4F46E5` (indigo-600) | Primary button, links, focus ring |
| Success | `#10B981` | Known / correct |
| Danger | `#EF4444` | Delete / unknown (use sparingly) |
| Surface dark | `#111827` (gray-900) | App background in dark |
| Text | white / gray-100 on dark | Body |

**Type:** keep Geist (already in `layout.tsx`) for UI. A serif for long passages is optional in Phase C, not a blocker.

**Theme:** follow `prefers-color-scheme` first (already in `globals.css`). A manual toggle is Phase C.

**Motion:** card flip may be a simple two-face toggle. 3D rotate is nice-to-have, not “mandatory premium.” Accessibility: respect `prefers-reduced-motion`.

**UI rules**

- Tailwind utilities or CSS variables. No ad-hoc inline style objects except dynamic values (e.g. width from data).
- Every interactive control has a visible focus ring.
- Flashcard actions must work without hover (mobile).
- Optimistic UI for known/unknown only after persistence exists and the update is local; do not fake success for AI extract.
- Shortcuts (Phase B+): `Space` flip, `1` unknown, `2` known. Ignore shortcuts while typing in an input.

**Home (Phase A)** is not a marketing landing page. It is the tool: title, paste/add, list. A three-step “How it works” can wait until there is something to demo.

---

## 10. Security (match the phase)

**Phase A–B (local only)**

- No secrets in the repo. `.env*` is gitignored (already).
- Treat `localStorage` as not trusted: `JSON.parse` in try/catch; validate shape before use.
- Trim and reject empty term/meaning.

**Phase C (AI)**

- API keys only in server env. Route Handler is the only caller.
- Cap input length and extract count.
- Do not send other users’ data anywhere — there are no other users yet.

**Phase D (auth + DB)**

- Auth before any cloud write.
- Row Level Security: a user reads/writes only their rows.
- Rate limit extract by user id (IP as a weak extra).
- CORS: same origin is enough for this app.

XSS: React already escapes text. Do not `dangerouslySetInnerHTML` on pasted passages.

---

## 11. Implementation plan

Each phase has a **Done when**. Do not start the next phase early.

### Phase A — static UI → in-memory list (this week)

1. Finish `app/page.tsx` TODOs (title, textarea, indigo button).
2. Add `lib/types.ts`.
3. Client form: controlled inputs, `preventDefault`, append `VocabItem`.
4. Split `VocabForm` and `VocabList`.
5. `/practice` with flip + known/unknown on the in-memory list (same session).

**Done when:** add three words, see them, delete one, quiz in the same tab. Refresh may still wipe data.

### Phase B — persistence + shared module

1. `lib/storage.ts`: load/save; bad JSON → empty list, no crash.
2. Both routes use it (not two copies of state).
3. `nextReviewAt` / `intervalDays` stored but can stay unused until C.

**Done when:** close the tab, reopen, words remain; practice still works.

### Phase C — extraction + due queue + audio

1. `POST /api/extract` with SpaceXAI; prompt in `lib/prompts.ts`.
2. Proposal UI: uncheck, then save.
3. Passage stored; cards link `passageId` + `sourceSentence`.
4. Due-today filter; home shows a count.
5. Pronounce button (Web Speech API). Fail silently if the browser has no voices.

**Done when:** you paste a paragraph, approve ~10 words, review them tomorrow from “due.”

### Phase D — only if daily use hurts

Pick the actual pain:

- Two devices → Auth + DB.
- Review quality → SM-2 + quizzes from `sourceSentence`.
- Real files → PDF/DOCX on a size-limited upload path (not the first serverless timeout).

---

## 12. Quality bar (honest)

Do not promise Lighthouse 95, PWA, and 50-text AI evals before the list persists.

**Now**

- `npm run lint` is clean.
- TypeScript `strict` stays on.
- Click through empty, error, and happy paths yourself.

**When storage exists**

- One module of tests for parse/save/load (invalid JSON, missing fields).

**When SRS exists**

- Unit tests for “unknown shortens interval, known lengthens, never negative.”

**When extract exists**

- Fixture: one sample passage → parsed JSON shape. No live API in CI until you have a key in secrets.

Accessibility: buttons have names, inputs have labels, flashcard is keyboardable. That beats a vanity Lighthouse screenshot.

---

## 13. Repo map (what each file is for)

| Path | Role |
|---|---|
| `app/layout.tsx` | Shell, fonts, metadata (change title when the UI exists) |
| `app/page.tsx` | `/` capture + list |
| `app/globals.css` | Tailwind v4 tokens |
| `app/practice/page.tsx` | You will create this in Phase A |
| `lib/types.ts` | You will create this |
| `lib/storage.ts` | You will create this in Phase B |
| `docs/README.md` | This spec |

Ignore `public/*.svg` from create-next-app until you replace the favicon.

---

## 14. Decisions log

Record choices here so you do not re-litigate them.

| Date | Decision | Why |
|---|---|---|
| 2026-09-12 | localStorage before Supabase | Persistence without auth; you are the only user |
| 2026-09-12 | Manual cards before AI | The loop must work if the model is down |
| 2026-09-12 | Simple known/unknown before SM-2 | You need review events before an algorithm |
| 2026-09-12 | AI via Route Handler, SpaceXAI default | Key stays on the server; provider is swappable |
| 2026-09-12 | No Zustand in Phase A | One list does not need a global store |

When you reverse a decision, add a new row, do not silently edit history.

---

## 15. Next action (do this, not the rest)

1. Run `npm run dev` in `vocab-mastery`.
2. Complete the three TODOs on `app/page.tsx`.
3. Add `lib/types.ts` with `VocabItem`.
4. Make the button append to an in-memory list and render it.

Stop there. Then come back and tick Phase A’s “Done when.”
