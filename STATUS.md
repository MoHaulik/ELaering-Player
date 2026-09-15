# ELaering Platform — Build Status

Working doc for today's build (2026-09-15). Client is a *different* customer than
LærbarXR/CAMES — no dashboard, new design, 180° support, offline+online delivery.
Update this as the day goes; treat it as the source of truth for what's actually
done vs. still open.

## Repo map

| Repo | Role | Status |
|---|---|---|
| `LaerbarXR-Builder` / `-Player` / `-Dashboard` | **Read-only reference** — live production for CAMES, never push/modify | Already cloned from yesterday's work, reused as-is. See `LAERBARXR-PLATFORM.md` at the top of this GitHub folder for full architecture notes. |
| `ELaering-Builder-Mockup` | Latest UI/design direction (client-approved look) | Fresh-cloned from `origin/main` today — see note below on a stale local copy. |
| `ELaering-Builder` | **Target — push here** | Local checkout fixed today (was on a disconnected `master` branch with a stale bug already fixed on `origin/main` — see below). Now on `main`, tracking origin. |
| `ELaering-Player` | **Target — push here** | Same fix applied — was on disconnected stale `master`, now on `main`, tracking origin. |

### Access check (2026-09-15)
All six repos reachable via `gh api` — no blockers this time (yesterday's LaerbarXR
access issue is resolved / not applicable here since those are read-only anyway).

### Repo hygiene findings (before any new work)
- **`ELaering-Builder-Mockup`**: the pre-existing local folder was on a `master`
  branch with **zero shared git history** with `origin/main` (confirmed via
  `git merge-base` — empty), plus an uncommitted 435/244-line diff on top of that.
  This looks like unpushed local iteration from a past session. Did **not** discard
  it — moved it aside to `ELaering-Builder-Mockup-PRIOR-LOCAL-COPY` (untouched,
  including its uncommitted diff) in case it's worth mining later. Working from a
  **fresh clone of `origin/main`** instead, since that's what Morten pointed at
  explicitly as "the latest version."
- **`ELaering-Builder`** and **`ELaering-Player`**: same pattern — local `master`
  branches were disconnected from `origin/main`, and critically, local
  `ELaering-Builder`'s `master` still had the *exact* `PLAYER_PREVIEW_URL:
  '../Larbar2026play/index.html'` dead-link bug I fixed yesterday in the actual
  LaerbarXR-Builder — while `origin/main` already has it correctly pointing at
  `../ELaering-Player/index.html`. That confirms `origin/main` is the newer,
  correct version for both repos. Both local checkouts now switched to a `main`
  branch tracking `origin/main` (old `master` branches left in place, untouched,
  not deleted — just no longer checked out).
- Also cleared two stale empty `.git/*.lock` files (`index.lock`, `HEAD.lock`,
  both dated mid-August, no git process running) that were blocking branch
  switches. Confirmed no active git process before removing.

## Requirements spec (authoritative)

Source: `Kravspecifikation_til_VR-scenarie_byg_editor_v0.8.docx` (copied into this
GitHub folder by Morten). Extracted to text since neither `pandoc` nor
`python-docx` were available in this environment — used a direct XML-strip
extraction of `word/document.xml` instead (full content recovered, verified
against the visible structure).

Full requirement list, Del 1 (Builder) IDs 01_01–01_15 and Del 2 (Player) IDs
02_01–02_11, mapped to today's build items below. Two items are explicitly
**flagged "orange" / unresolved** in the client's own spec, not just my own
uncertainty:

- **User management** (ID:01_03/01_04): "Hvordan foregår brugerhåndtering? /
  Indeholder den et brugerstyringsmodul?" — never answered in the spec or the
  meeting notes. **Decision for today**: no user accounts / no login, matching
  LærbarXR's precedent and the spec's own Kiosk-mode / no-Meta-account
  requirement (ID:02_05, ID:02_06) — single local user assumed. Flag this
  explicitly to Morten; if the client actually wants multi-user access control
  on the Builder, that's a real, separate piece of work.
- **Video compression** (ID:01_08, reopened in the supplementary notes): "er det
  noget der kan sættes inde i editoren?" — real in-browser video transcoding
  (WebCodecs / ffmpeg.wasm) is a substantial standalone project, not something
  fittable into one day alongside everything else. **Decision for today**: not
  implemented; documented as a known gap with a suggested path (client
  pre-compresses source footage before import, or a follow-up ffmpeg.wasm
  integration later).

## Today's build list

Status legend: ⬜ not started · 🟡 in progress · ✅ done · ⏸️ deferred (documented, not silently skipped)

| # | Item | Spec ID(s) | Status |
|---|---|---|---|
| 1 | No dashboard — builder + player only | (platform shape) | ⬜ |
| 2 | New, sleeker builder UX (from Mockup design) | 01_05 | ⬜ |
| 3 | 180° video support (builder + player) | 01_07 | ⬜ |
| 4 | Voice-over / narration support | 01_12A/B/C, 01_14B | ⬜ |
| 5 | Offline export zip (confirmed local-only) + online-hostable player | (client Q confirmed) | ⬜ |
| 6 | Background colour toggle (black/white/grey, persisted) | — (meeting notes) | ⬜ |
| 7 | Stereoscopic video vs. text-overlay render-order conflict | — (meeting notes, client concern) | ⬜ |
| 8 | Freely-repositionable branch nodes on canvas (visual only, persisted) | — (meeting notes) | ⬜ |
| 9 | Zoom control (visible %, independent of browser zoom) | — (meeting notes) | ⬜ |
| 10 | Project switcher / tabs near project name | — (meeting notes) | ⬜ |
| 11 | Export flow — real UI, not a stub | — (meeting notes, client asked directly) | ⬜ |
| 12 | Start-position selection on a recording | 01_09 | ⬜ |
| 13 | Fade to/from black at clip start/end | 01_10, 02_11 | ⬜ |
| 14 | Waypoints within a video (distinct from end-of-clip branch choice?) | 01_13 | ⬜ |
| 15 | Text overlay: frame, semi-transparent, position/size/timing | 01_14A/C/D/E | ⬜ |
| 16 | Player: local code/PIN entry gates scenario start (replaces Dashboard) | 02_07 | ⬜ |
| 17 | Player: gaze/target click, no controller | 02_04 | ⬜ |
| 18 | Player: Kiosk mode, no Meta account required | 02_05, 02_06 | ⬜ |
| 19 | Player: jump between scene positions, auto-reorient on arrival | 02_08A/B/C | ⬜ |
| 20 | Player: smooth-fade vs. direct-cut transition, selectable | 02_09, 02_10 | ⬜ |
| 21 | Video compression in editor | 01_08 | ⏸️ deferred — see note above |
| 22 | User management module | 01_03/01_04 | ⏸️ deferred — see note above, flagged to Morten |

## Open questions for Morten / the client (not guessing on these silently)

1. User management — confirm no-auth-for-now is acceptable (see above).
2. Video compression — confirm deferring is acceptable, or is there a hard
   requirement for launch.
3. Export flow — client asked directly what this looks like; building a real,
   presentable flow today (not a stub), will document exact behavior here once
   built.
4. Stereoscopic/text conflict — client raised this as a real concern; documenting
   the chosen rendering approach here once implemented.
5. Waypoints-within-a-video (01_13) vs. end-of-clip branching (01_11) — spec lists
   these as separate requirements; need to confirm whether "waypoints" means
   *multiple* clickable hotspots mid-video (not just at the end), which is a
   materially different interaction model from LaerbarXR's end-of-clip choice
   buttons. Building the more capable interpretation (mid-video hotspots) unless
   told otherwise, since it's a superset of the simpler case.

## Work log

- **2026-09-15, start of day**: access check clean, repo hygiene fixed (see
  above), requirements spec extracted and read in full, gap analysis of existing
  ELaering-Builder/-Player vs. LaerbarXR backend + the Mockup's new design
  kicked off.
