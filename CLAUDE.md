# Effortless Admin prospect demo deck

`Effortless-Admin-Sales-Deck.html` is a single, self-contained reveal.js 5.1.0 deck that sales screenshares over Microsoft Teams at the start of prospect demos. It gives about ten minutes of context, then hands off to the live platform. It must keep working offline as one file: no external fonts, scripts, CDNs or image links.

## The folder

- `Effortless-Admin-Sales-Deck.html`: the deck, the only deliverable.
- `Advisor-Logo-*.svg|png`: the presenter's light-version broker logos, which they drop into the setup screen before a demo. They live beside the deck, never inside it (see the safety rules below).
- This folder is not a git repository, so there is no history to fall back on. Before a substantial edit, copy the deck to the session scratchpad as a backup and tell the user where it is. Warden can't check the folder either (`warden doctor` reports "not inside a repo") until `git init` and `warden init` are run here.

## Reading the file

The file is about 2 MB, almost all of it inlined assets, so never Read the whole thing. Inlined content:

- Plus Jakarta Sans and Inter as base64 woff2 (`@font-face` at the top)
- reveal.js CSS (one ~51 KB line), reveal.js JS (~107 KB line) and the RevealNotes plugin (~68 KB line)
- 15 PNG logos, about 40 SVGs and 1 GIF as `data:` URIs
- Font Awesome-style inline `<svg>` icons with long `<path d="…">` attributes

To get a readable copy, strip the data URIs into a scratch file:

```bash
sed -E 's/data:[a-z]+\/[a-z0-9.+-]+;base64,[A-Za-z0-9+\/=]{100,}/DATAURI/g' Effortless-Admin-Sales-Deck.html > <scratchpad>/stripped.html
```

This keeps every line in place, so line numbers in the copy match the original. To find what you need, Grep the original for slide markers such as `<!-- ============ 6 THREE JOBS` or for class names. Make edits with Edit, using anchors that contain no base64. Never rewrite the file wholesale. When a change needs a base64 asset that's already in the file (such as the footer swoops mark), a small Node script can copy it from an existing element, so there's no need to paste base64.

## File layout (top to bottom)

1. `<style>` with the embedded fonts and the reveal.js CSS (vendor code, don't touch)
2. `<style>` with the deck's own CSS: `:root` tokens, ambient stage, reveal overrides, entrance animations, shared components, then one block per slide (`/* ---------- N Name ---------- */`), then the co-brand lockup and setup screen styles, then reduced-motion and `.print-pdf` rules
3. `<body>`: `#ambient` background layer, then the `#setup` screen, then `.reveal > .slides` with 13 `<section>`s
4. A localStorage/sessionStorage shim, then reveal.js and RevealNotes (vendor code)
5. The deck's init script, in order:
   - Splits `.split` headlines into words.
   - Defines `numberPages()`, `varyGlows()`, count-up and `onSlide` (which sets `body[data-glow]`).
   - Defines the co-brand lockup functions and `startDeck()`.
   - Runs the setup screen and slide picker.
   - Calls `Reveal.initialize` from `startDeck()`, and only once Start is clicked.

## Setup screen and co-branding

When the file opens, a setup screen (`#setup`) appears before the deck starts. The presenter drops in, browses for, or pastes the broker's logo (PNG, SVG, JPG or WebP; light/reversed version), sees a preview, picks slides, then clicks **Start co-branded** or **Start Effortless only**.

- **Co-branded:** the header logo on the title and demo slides (`.title .logo`, `.demo .logo`) becomes an "Effortless × broker" lockup at 82px high. The footer swoops mark in every `.chrome` becomes a small lockup at 26px high, using the full Effortless logo. The lockup is built by `lockup()` and `applyCobrand()`: a fine-lined SVG × as tall as our logo at 50% opacity, one "a" of clear space (0.0854 × our logo width) on each side, and the broker logo balanced by visual weight (`brokerSize()`: equal area, clamped to 0.5–1.3× our height and at most 1.35× our width). SVGs without a width and height get their ratio from the `viewBox` (`svgRatio()`).
- **Effortless only:** the deck looks exactly as it did before co-branding was added.

### Slide picker

A checkbox list on the setup screen, built at load time from each slide's eyebrow and headline:

- **Defaults:** slides are ticked unless their `<section>` has `data-pick="off"`. Currently that's slides 2 (`model`), 3 (`prob`), 11 (`trust10`) and 12 (`smart`), so the default deck has 9 of 13 slides. **Select all** ticks everything; **Clear** unticks everything except the locked slides.
- **Locked slides:** the first and last slides (title and demo) are always included.
- **Order:** reordering is deliberately not supported.

On Start, `startDeck()` removes the unticked `<section>`s, then:

- `varyGlows()` keeps each slide's designed `data-glow`, but changes one that would repeat its neighbour's, so the background orbs always move between slides. For example, in the default deck the demo slide switches from `center` to `blue` because Data flow (`center`) now comes before it.
- `numberPages()` numbers the remaining slides, e.g. "02 / 09".

Speaker notes are not adjusted for skipped slides. For example, the demo slide notes refer to "slide 3", which is off by default. Speaker-view frames (`?receiver`) always load every slide, so the speaker view's previews don't match a curated deck. The presenter doesn't use it.

### Safety rules

These are the reason the feature exists, so don't weaken them:

- The setup screen must never be reachable from the deck. `startDeck()` removes `#setup` from the page before calling `Reveal.initialize`, and the hash is cleared so the deck always opens at slide 1. Never make setup a `<section>`, and never add a path back to it. Skipped slides are removed in the same way.
- The file never contains a broker logo. The dropped logo exists only as an object URL in that browser tab and is never saved (no localStorage, no URLs). Never add a built-in logo gallery: if the file were shared, it would expose the prospect list.
- After Start, a `beforeunload` confirmation guards against refresh, because a refresh brings the setup screen back.
- Default `dragover`/`drop` handling is blocked page-wide, so a stray drop can't open the image in place of the deck.
- Speaker-view frames (`?receiver`) skip setup and start Effortless only.
- Brand note: the 26px footer lockup puts the full Effortless logo at about 82px wide, below the brand's 183px minimum. The user chose this knowingly (a Tier 1 override).

## Slides

| # | Section class | Default | Topic |
|---|---|---|---|
| 1 | `title` | locked | "Manage employee benefits effortlessly." with a proof strip (SOC 2 Type II, 40+ carriers, 4.9 Capterra) |
| 2 | `model` | off | The model, "A true administration partner.": Advisor, Effortless Admin and Carriers & Providers cards, joined by a bracket to "The Employer enjoys peace of mind" |
| 3 | `prob` | off | Hidden cost: HR team at the centre of 7 providers; 4 cost lines appear one after another |
| 4 | `mkt` | on | 40+ carriers stat and a 6-column wall of logo tiles |
| 5 | `sep` | on | Strategic Separation: administration unified, claims stay with the carrier (flow diagram) |
| 6 | `jobs` | on | Three jobs, one login: Administration, Compliance, Billing cards |
| 7 | `auto` | on | Automation tiles: NEM, beneficiary forms, Flex re-enrollment, overage dependents, plus a live-chat strip |
| 8 | `svc` | on | Service: 25,000+ chats, 99% vs 64% benchmark bars, 14 s, 88%, 4.9, GBA |
| 9 | `hris` | on | HRIS integrations: 60% less data entry; 7 HRIS logos feeding Effortless, then 40+ carriers |
| 10 | `df` | on | Data flow: HRIS → Effortless → Payroll cards, with a provider-feeds box below |
| 11 | `trust10` | off | Security and trust: 6 cards plus a heritage timeline (1987, 2018, 150+, Bilingual) |
| 12 | `smart` | off | "Not just easier. Smarter.": 3 rows |
| 13 | `demo` | locked | Hand-off to the live demo ("your reality") |

Every slide has `<aside class="notes">` speaker notes; press **S** to open the speaker view. The notes contain talk tracks, discovery questions, sources for the figures and wording guidance. When slide content changes, update the notes to match. The slide 2 notes were drafted by Claude, because the mockup had none.

## Conventions

- **Canvas:** every slide is a fixed 1280×720. `.s` is the slide body with 64px padding. Most elements use absolute pixel positions from the per-slide CSS, so longer text can overlap other elements; slide 2 uses a flex column instead. Check the fit after copy changes.
- **Tokens:** use the `:root` variables (`--blue`, `--green`, `--gold`, `--violet` and their `-soft` versions, `--fg`, `--text`, `--dim`, `--line`, `--display`, `--body`) instead of hard-coded colours. Highlight classes are `.hl-blue`, `.hl-green`, `.hl-gold`; `.grad` is the animated gradient text.
- **Slide structure:** an `.eyebrow` (`.blue` / `.green` variants, gold by default), then an `h1`/`h2` (`h2.md` is the smaller size), then the visual, then `.chrome` with `<span class="pg"></span>` and the small logo. The title and demo slides have no chrome.
- **Entrance animations:** `.a` rise, `.al` / `.ar` slide in from the left / right, `.az` pop, `.ab` blur in. Timing is set inline with `style="--d:.5s"`. Headlines with `.split` animate word by word, starting at `--d0`. Lists, tiles and satellites stagger by `--i` (and `--c` for columns on the Three jobs slide). SVG lines draw in with `@keyframes draw` (use `pathLength="1"` and a dash of 1). New elements need delays that fit the existing sequence.
- **Glow:** `data-glow` on a `<section>` moves the background orbs. Values are `blue`, `green`, `gold`, `center` and `low`.
- **Count-up numbers:** `<span data-count="25000" data-format="comma">25,000</span>`, with `data-dec="1"` for decimals. When changing a stat, update both the attribute and the text.
- **Page numbers** are generated at Start from the slides that remain, so adding or removing a `<section>` needs no renumbering in the HTML.
- **New animated elements** also need a matching `.print-pdf` override so they show up in PDF export (`?print-pdf`), and must still work with `prefers-reduced-motion`.
- **Language:** Canadian English (`lang="en-CA"`).
- **Brand:** load the `effortless-brand:effortless-brand` skill before changing visual design, colours, logos or copy voice.

## Adding a slide

New slides often arrive as a standalone HTML mockup, sometimes a bundled export where the page is a JSON string in `<script type="__bundler/template">` and the images are gzip/base64 in `__bundler/manifest`. Read the template with the Read tool; don't run the bundle. Then rebuild the slide in the deck's own style rather than pasting it in:

1. Rebuild with the deck's classes: `.eyebrow`, `h2.split` with `.hl-*`, `.lede`, `.card` / `.card.top-*`, `.ico`, `.grad`. Replace inline colours with tokens. Drop any background circles or texture drawn into the mockup; the `#ambient` layer and the slide's `data-glow` provide them. Turn `sc-camel-view-box`-style export attributes back into `viewBox` / `gradientUnits`.
2. Give it a `data-glow` that differs from its neighbours, a per-slide CSS block, staged entrance animations, `.print-pdf` overrides for anything animated, and the standard `.chrome` (copy the swoops `<img>` from another slide).
3. Decide its picker default with the user: ticked, or `data-pick="off"`.
4. Write speaker notes in the house style: short talk track, wording guardrails, a bridge line to the next slide. Tell the user they are drafts.
5. Inserting a slide shifts the later ones, so renumber the `<!-- ============ N` and `/* ---------- N` labels, fix any notes that cite a slide number, and update the table above.
6. Test it (below). Delete the mockup only once the slide is verified and the user has asked for it.

## Testing

Drive the installed Chrome with `playwright-core`, installed in the session scratchpad (`npm install playwright-core`), with `executablePath: 'C:/Program Files/Google/Chrome/Application/chrome.exe'`, opening the deck by `file:///` URL. Useful moves:

- Load a logo with `setInputFiles('#file', …)` (a wide and a square test SVG catch sizing problems), then click `#go-co` or `#go-ea`.
- Tick or untick picker rows via `#slide-list .srow`.
- Jump with `Reveal.slide(i)`, and wait about 2.5 s for entrance animations before taking final-state screenshots.
- Confirm `#setup` is gone after Start, keys like ArrowLeft/Home can't go before slide 1, reload raises a `beforeunload` dialog, page numbers match `Reveal.getTotalSlides()`, and there are no `pageerror` or console errors.

Read the screenshots to check the result by eye before reporting.

## Messaging guardrails (from the speaker notes)

- Carriers are partners. Never position the platform as a way to "switch carriers easily". The point is that the plan can change without disrupting administration.
- Claims stay direct with the carrier, by design.
- The advisor owns plan design and the client relationship; Effortless Admin owns data, compliance and billing.
- Support is handled by people, never AI chatbots. AI is used inside the platform to help the specialists, not as the support channel.
- Live-chat figures (14 s, 88%, 99%) cover live chat only, not the whole company. The 64% benchmark comes from the livechat.com customer service report.
