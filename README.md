# Verify the Fix

A clickable concept for [Interfere](https://interfere.com) by Avto Grigolia.

It's one static page with no build step and no backend. Everything runs on mock data inside Interfere's own demo world (the Resonance app, Maya Chen, PRB IDs).

---

## The thesis

**The agent finds and fixes it. A teammate confirms users can actually do the thing again. The engineer merges with confidence.**

Interfere already does the hard part. It watches sessions, groups occurrences into problems, finds the root cause, and drafts a fix. What I couldn't find, in the docs or the product demo, is who owns the step between "the agent wrote a fix" and "users are unblocked." This concept is my take on that step: a visual way for someone who doesn't read code to look at the problem, try the fix, and pass it to engineering.

- **The engineer** can review the code, but code review answers "is this correct," not "did this fix what the user hit."
- **The person who heard the complaint** is usually a customer service agent, a PM, or a designer. They can't read the diff, but they're the best person to say whether the experience works again.
- **The session replay already exists.** That's the evidence. The missing piece is a structured way to repeat what the user did on a preview with the fix, and record the result.

So the idea is a **"Ready to verify" status** on a problem, shown here on PRB-1's card in the Needs attention column:

1. The agent ships a fix to a preview deploy (Vercel preview URLs make this nearly free).
2. A non-engineer opens the problem, watches the 10-second moment the user hit the bug, then repeats it on the preview, side by side.
3. They mark it **Fixed for users** or **Still broken** with a note. "Still broken" goes back to the agent as new context.
4. The engineer reviews the PR with a "Verified for users" receipt attached, and merges.
5. The problem moves to In recovery, occurrences drop to zero, and support gets a drafted note for the affected companies.

### What this deliberately is not

- **Not a human babysitting the inbox.** The agent still detects, diagnoses, and fixes on its own. People only show up where the agent can't be trusted alone: judging whether the experience actually works for a user.
- **Not session replay as the home screen.** Sessions are evidence, not the entry point. The problem view shows only the moment that matters, with the full session one click away.
- **Not auto-deploy before review.** The agent opens a PR. An engineer merges it. Verification happens on a preview, never in production.

### What I assumed

I worked from Interfere's public docs and a recorded product demo, not from the product itself. So this rests on a few guesses:

- **The step is open.** I didn't see anything that covers "a non-engineer confirms the fix works for users." If it already exists, read this as one take on it.
- **Needs attention.** I don't know what that column covers. PRB-1 sits there in the demo, so I left it there and added Ready to verify as a status on the card. The idea doesn't depend on the column.
- **Previews and PRs.** The demo shows the agent's suggested fix as a diff. I assumed that fix can also go to a preview deploy and be opened as a PR, because the verify step needs something to click through and the merge gate needs something to merge.
- **The data.** PRB-1's title and the figures on its card come from the demo. The rest of its story here is mock data; see "What's real and what's mocked" below.

---

## Walkthrough

The sidebar has a 4-step guide (a bottom bar on mobile). Every view also has a URL hash, so you can link straight to one.

| Step | Hash | What it shows |
|---|---|---|
| 1. Inbox | `#inbox` | The board as the demo shows it: Active, Needs attention, In recovery. PRB-1 is waiting in Needs attention with a new **Ready to verify** status: "Ready to verify · Preview live." |
| 2. Problem | `#problem` | The app's own layout: properties, dates, counts, and the occurrences chart on the left; activity on the right. Above the timeline, a looping replay of Maya pasting a long note, seeing "Saved," refreshing, and losing it. Then a plain-language root cause and the suggested fix, with the fix card and the diff open. While a fix is waiting, "Verify fix" is also in the top bar. |
| 3. Verify | `#verify` | Maya's session on the left, a **working** preview with the fix on the right. Paste, save, and refresh for real. The checklist ticks itself off. `F` = Fixed for users, `B` = Still broken. |
| 4. Engineer review | `#handoff` | The PR diff with a "Verified for users" card, agent confidence, and checks. Merge is disabled until someone verifies. `M` = Approve and merge. |

After merging, the problem view switches to **In recovery**: 0 new occurrences, a "Fix deployed" marker on the chart, and a draft message to the 4 affected companies.

The steps can be opened in any order without losing progress. Once a fix is verified or merged, the Verify screen shows the recorded check instead of reopening it, and the review screen's header, side card, and merge button always agree. The one exception is deliberate: after "Still broken," opening Verify stands in for the agent's next preview and starts a fresh check.

"Reset demo" in the sidebar card starts over. The walkthrough card stays pinned at the bottom of the sidebar; on short windows the nav above it scrolls.

### The bug used for the demo

**PRB-1: Autosave reports saved while the note is discarded.**

- Long notes show "Saved" but are never stored. The app sets "Saved" before the server responds, the API rejects bodies over 2,000 characters, and the client swallows the 413.
- The fix only shows "Saved" after the server confirms, raises the limit to 20,000 characters, and surfaces a failed save.

I picked it because anyone can verify it without reading code: paste, save, refresh, check the note is still there.

---

## What's real and what's mocked

**Real (interactive):**
- The preview panel on the verify screen. It debounces, waits for a fake server confirm, then shows "Saved." Refresh restores whatever was actually saved.
- Checklist detection: paste over 2,000 characters, see "Saved," refresh, and the note survived.
- State across screens: verifier note, check duration, status chips, column moves, the merge gate.
- Keyboard shortcuts: `V` on the problem view, `F` / `B` on verify, `M` on review, `Esc` to close the "Still broken" form.

**Mocked:**
- The "session replay" is a scripted animation of the buggy version of the same mini app, not a recording.
- All problem data, users, ARR, agent confidence, test results, and the PR.
- Other inbox cards and sidebar items show a toast instead of navigating.

---

## Design notes

**Styling comes from Interfere's public docs** (interfere.com/docs). I read the computed styles and theme tokens from that page:

| Token | Value | Source |
|---|---|---|
| Page ground | `#09090b` | `html` background |
| Panel | `#111111` | `--background-dark` |
| Panel border | `rgba(255,255,255,.07)` | `dark:border-white/[0.07]` |
| Radius | 12px rows and controls, 16px panels | `rounded-xl` / `rounded-2xl` |
| Grays | `#f3f3f3` to `#0b0b0b` | `--gray-50` to `--gray-950` |
| Sidebar row | 13/20 medium, padding 5 10 5 12, gap 10 | app sidebar (demo screenshots) |
| Active row | `rgba(238,238,238,.10)` + white text | `bg-primary-light/10` |
| Hover row | `rgba(223,223,223,.05)` | `hover:bg-gray-200/5` |
| Frames | `rgba(38,38,38,.25)`, 8px padding | `.frame` around screenshots |
| Body font | Inter with `font-feature-settings: "ss03"` | `--font-family-body-custom` |
| Headings | 600, tracking -0.025em | `h1` |
| Mono | Paper Mono | `--font-paper-mono` |
| Status colors | green `#6ae1a1`, red `#f08b85`, yellow `#fde047`, purple `#b78af0` | badge component, dark variants |

**Structure comes from the app itself**, as shown in Interfere's product demo. I matched these to low-resolution screenshots, so layout is faithful but exact sizes and colours are approximate:
- Sidebar: item list and order, uppercase "Problems" label, bug icons on tinted tiles, ⌘ K keycaps.
- Board: Active / Needs attention / In recovery, centred column labels with a status glyph and count, and the card layout including the "+N" chip.
- Top bars: filter and display buttons with the Board / List toggle on the inbox; previous / next, Dismiss, bell, and more on a problem.
- Problem view: a narrow property column split by rules (ID to App, then Regression and dates, then Occurrences and Affected users above the chart), tabs over the activity column, and "Suggested fix" with collapsible code blocks that start open. The App and Regression values for PRB-1 are filled in by analogy with the problem shown in the demo.
- Session player: URL pill above a black letterboxed stage, white scrubber with event dots, timecode on the left, transport in the centre.

**Sidebar icons** (Inbox, Activity, Apps, Services, Users, Companies) are Interfere's own SVGs from `/docs/icons/ui/`, inlined. Settings is an approximation. Services uses their "connections" icon because the real one is hosted on a domain that blocks cross-origin fetches. The app draws inactive items as outlines; these are the filled versions throughout.

**Approximated, because the screenshots were too small to read:** the dot-matrix column glyphs and the workspace mark (an "R" tile here). The "+N" chip on cards is decorative.

**My own design calls** (not from their system):
- The Ready to verify status, the moment replay and fix card on the problem view, the verify screen, and the PR review screen.
- After an action (merge, or sending a fix back), the problem view scrolls its activity column to the new entry, while the property column stays in place.
- The player's side buttons step between marked events instead of skipping 10 seconds, because the clip is 8 seconds long.
- The live preview borrows the player's frame and adds a refresh button beside the address, so the two panes line up.
- Primary actions are white instead of green, to stay inside their monochrome system. Green is only for status.
- The customer app (Resonance) stays light inside a dark frame, so it reads as "their app" versus "Interfere."

---

## Files

```
interfere-verify-the-fix/
├── index.html                   the whole prototype (HTML + CSS + JS, no dependencies)
├── fonts/
│   ├── PaperMono-Regular.woff2
│   ├── PaperMono-Medium.woff2
│   └── PaperMono-OFL.txt        Paper Mono license (SIL OFL 1.1)
├── package.json                 only a `dev` script, no dependencies
└── README.md
```

Inter loads from Google Fonts. Nothing else is external.

### Where things live in `index.html`

- **`<style>`:** design tokens at the top of `:root`, then components in page order (sidebar, board, problem, code, mini app, replay, verify, handoff, recovery, guide).
- **`<script>`:**
  - `P` / `ic()`: stroke icons (chevrons, check, play, etc.).
  - `FI` / `fi()`: Interfere's fill icons for the sidebar.
  - `PROBLEMS`, `NOTE`: mock data.
  - `S`: app state (`status` is `ready`, `verified`, `broken`, or `recovery`).
  - `mountReplay()`: the scripted session replay.
  - `mountPreview()`: the live fixed preview.
  - `views.{inbox, problem, verify, handoff}`: one render function per screen.
  - `go(view)`: router (hash-based).
  - `act(action)`: button and keyboard actions.

---

## Run and deploy

```bash
# local
npm run dev          # serves this folder at http://localhost:3001 (python3 -m http.server, nothing to install)
npx serve .          # or any other static server

# vercel
npx vercel           # framework preset: Other, no build command, output dir: .
```

It's plain static files, so Vercel needs no config.

---

## Ideas if this goes further

- Port to Next.js plus Tailwind with their tokens as a theme, and split views into components.
- A real recorded clip instead of the scripted replay.
- Motion when PRB-1 moves from Needs attention to In recovery after merge.
- A "who verifies" setting: route verification to whoever filed the support ticket, or to the account owner for the affected company.
- Let the verifier record their own check and attach it to the PR, instead of only a timer.

---

## Credits

- Concept, design, and build: Avto Grigolia.
- Interfere's name, demo world (Resonance, Maya Chen), visual tokens, and sidebar icons belong to Interfere. They're used here only for a concept made for Interfere, which isn't affiliated with or endorsed by them.
- Paper Mono by Paper, SIL Open Font License 1.1. Inter by Rasmus Andersson via Google Fonts, SIL OFL 1.1.
