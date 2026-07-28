# Altcoinist Hall of Fame

Public showcase of the top of Altcoinist's Inner Circle — proven callers ranked by
**reputation**, not referrals. A curated highlight reel that lives outside the mini app
and is built to be shared on X.

Two jobs:
1. **Flex.** Members share their card + rep on X. Status, exclusivity, social currency.
2. **Pull.** Viewers click a shared card, land on the Hall, get pulled into the network.

## Deployment

**Source repository:** [altcoinist-com/hall-of-fame](https://github.com/altcoinist-com/hall-of-fame).

**Publishing surface:** GitHub Pages. Confirm the configured public URL and publishing branch in the repository’s Pages settings before sharing a link. This repository is standalone and has no affiliates parent pointer.

**Routes:**
| URL | Page |
|---|---|
| `/` | Hall of Fame — hero + top-3 podium |
| `/rankings` | Full circle, ranked by rep. Sort toggle (rep / reach) + data-driven chain filter chips |
| `/calls` | Biggest winning calls, all-time (scored on-chain by peak move after the call) |
| `/about` | What the Hall is vs the mini app |
| `/c/{handle}` | Individual member profile + Share-on-X |

## Structure

```
hall-of-fame/
├── index.html             # landing (/)
├── rankings.html calls.html about.html
├── profile.html           # dynamic /c/{handle} fallback (client-renders from hall-data.json)
├── c/{handle}.html        # pre-generated per-member pages (baked-in og: tags) — see below
├── styles/hall.css        # shared design system (one source of truth)
├── hall-data.json         # DATA SOURCE for every page (members + calls)
├── avatars/               # member profile pictures
├── assets/
│   ├── og-hall.png        # generic share card (og:image for non-profile pages)
│   ├── cards/{handle}.png # per-member share cards
│   └── tier-cards/{character}/{tier}.png  # reputation character art (5 chars × 5 tiers)
├── generate-og-card.py    # regenerates all share cards + c/ pages from hall-data.json
├── import-tier-cards.py   # imports + optimizes the 25 tier cards from Konsti's source folder
└── refresh-hall.py        # historical snapshot refresh helper
```

## Data

`hall-data.json` is the single source of truth — every page fetches it client-side.

```jsonc
{
  "meta": {
    "rosterSize": 48, "scoredCount": 21,
    "scoringSource": "v19.8 caller reputation handover (Conor/Spyro, 2026-05-29)",
    "lastRefresh": "v19.8", "lastRefreshLabel": "may 29",
    "tierColors": { "A+": "#39F590", ... },
    "tierBands": null   // letter grades pending official score->grade bands from the team
  },
  "athletes": [
    { "handle": "@chironchain", "avatar": "avatars/chironchain.jpg", "character": "samurai",
      "chain": "base", "scored": true, "repScore": 94.5, "repTier": null, "qualified": null,
      "calls4x": 6, "calls": 60, "followers": "6.8K" }
  ],
  "calls": [ { "chain": "base", "ticker": "$ODAI", "caller": "@thebasedfrogx", "mult": 105.0, "date": "2026-02-12" } ]
}
```

> **Historical data snapshot:** the bundled v19.8 reputation data reflects May 29, 2026 (`repScore` = display_score, 0–100, floor 40). It is not live data.
> `repTier` is `null` everywhere and the Hall shows the **number, not a letter** — this matches
> the app, which dropped A+→D letter tiers in v15. Letters would be a fresh design decision
> (pick the score→grade cutoffs with Conor/Spyro), not something the scoring emits.
> Unscored members (`scored: false`) render with an "unscored" tag.

## Refreshing scores (the data pipeline)

The historical snapshot came from a **v19.8 caller-reputation board**. The board identifies callers
by an **internal name** (a TG/caller handle, e.g. `cryptocchio0`), not the X handle the Hall shows
(`@mrLCguy`). `refresh-hall.py` bridges that gap and updates everything in one command.

```bash
# preview (dry run — shows up/down/new/dropped, writes nothing)
python refresh-hall.py --html "<path>/v19.8-leaderboard.html"
python refresh-hall.py --csv  "<path>/v198_board.csv"          # canonical score_v198.py output

# apply: rewrites the score fields in hall-data.json + regenerates all cards/pages
python refresh-hall.py --csv "<path>/v198_board.csv" --apply --label "jun 18"
```

It only touches the **score fields** (`repScore`, `calls4x`, `calls`, `scored`). Chain, character,
avatar, followers are left alone. It never auto-unscores someone who drops off the board (it warns
instead), and it lists any board callers not yet in the map.

**`caller-map.json` — the bridge (LOCAL ONLY, gitignored).** Maps each Hall X-handle to its board
internal name. It is **deliberately not committed**: this repo is public + Vercel-served, so a
committed map would be fetchable at `/caller-map.json` and would deanonymize affiliates' TG
identities. Keep it local. It was seeded by score-matching the May-29 board (display_score + n4x +
all-time calls — a unique match across all 21 scored). Set `verified: true` per row after eyeballing.
Regenerable from any board if lost.

**Refreshing data:** obtain a current approved source through the owner named in Marketing context before using `refresh-hall.py`. The former handover path is historical evidence and is not a current source of truth.

## Share cards (Open Graph)

When a link is shared on X / Telegram, the crawler reads the page's `og:image`. Two layers:

- **Non-profile pages** → generic `assets/og-hall.png`.
- **Profiles** → each `/c/{handle}` serves a pre-generated static page (`c/{handle}.html`) whose
  `og:` tags point at that member's card (`assets/cards/{handle}.png`). Static files are matched
  before the `/c/:handle` rewrite, so these win; the dynamic `profile.html` stays as the fallback
  for handles not pre-generated.

**Regenerate after any data change:**

```bash
python generate-og-card.py
```

Rebuilds `og-hall.png`, every `assets/cards/{handle}.png`, and every `c/{handle}.html` from
`hall-data.json`. Requires Pillow (`pip install Pillow`).

## Adding / refreshing a member

1. Add their picture to `avatars/{handle}.jpg`.
2. Add / edit their entry in `hall-data.json`.
3. Run `python generate-og-card.py`.
4. Review generated outputs. Commit or publish only with the required authorization.

## Design

Black canvas, Inter for headings, JetBrains Mono for labels. A+ tier gets a verified ring +
badge (mirrors the X badge). Edit design tokens once in `styles/hall.css` and every page updates.

Tier colors are implementation tokens in this repository. Before changing
them, follow the Marketing design-system route in `AGENTS.md` and verify the
current Mini App source of truth:

| Tier | Color | Hex |
|---|---|---|
| A+ Legendary | mint (brand green, A+ only) | `#39F590` |
| A Elite | turquoise | `#2EE6D6` |
| B Respected | royal blue | `#1E88FF` |
| C Trusted | steel blue | `#6E9BFF` |
| D Newcomer | pearl white | `#F5F5F7` |

**Reputation character art.** The home spotlight (top-3 podium) renders each member's tier
character as the focal image. 5 classes (hunter / mage / paladin / samurai / shaman) × 5 tiers,
same character at every tier — only color + glow intensity change up the ladder. Each athlete's
`character` field in `hall-data.json` picks the class; `repTier` picks the tier variant. The art
is currently mock-assigned per member; the real class comes from the mini-app reputation system
when it ships.

**Importing tier cards** (when Konsti updates the source set):

```bash
python import-tier-cards.py "/path/to/Tier Cards characters"
```

Normalizes the inconsistent source filenames, resizes to web size, and writes
`assets/tier-cards/{character}/{tier}.png`. Requires Pillow.

## Repository boundary

This is the standalone [altcoinist-com/hall-of-fame](https://github.com/altcoinist-com/hall-of-fame) repository. GitHub Pages is its publishing surface. There is no affiliates-submodule pointer to update.
