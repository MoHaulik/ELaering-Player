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

## ⚠️ Correction — a prior session already built most of this

The first version of this table (written before I'd read the actual `index.html`
files, only from Morten's brief + the spec) assumed a near-empty starting point.
That was wrong. Once `ELaering-Builder`/`ELaering-Player` were pointed at the
correct `origin/main` (see repo hygiene above), reading the real code showed a
prior session had already built most of the feature list — properly, not as
stubs. Corrected against the actual code below (line numbers as of 2026-09-15).

## Today's build list

Status legend: ✅ done (verified in code) · 🟡 partial/needs finishing · ⬜ not started · ⏸️ deferred (documented, not silently skipped)

| # | Item | Spec ID(s) | Status |
|---|---|---|---|
| 1 | No dashboard — builder + player only | (platform shape) | ✅ Player has zero relay/heartbeat/dashboard code — confirmed no `fetch(` calls exist at all. Replaced by the local code-gate (#16). |
| 2 | New, sleeker builder UX (from Mockup design) | 01_05 | ✅ Per Morten's explicit correction, the Mockup's actual shell now ships as the real UI (dock/card-flow/bottom-sheet), with the full working backend ported in underneath. Verified live: `App` loads, no console errors, node graph/inspector/export all function inside the new shell. |
| 3 | 180° video support (builder + player) | 01_07 | ✅ Builder: `setVideoProjection()`, per-asset `projection` field. Player: `makeProjectionGeometry()`/`applyVideoProjection()` (half-dome vs full sphere). |
| 4 | Voice-over / narration support | 01_12A/B/C, 01_14B | ✅ Builder: `node.narration` model, `updateNarration()`, editor UI. Player: `narrationAudio`, "Hear again" replay button. |
| 5 | Offline export zip + online-hostable player | (client Q confirmed) | ✅ Export is a local zip (same JSZip pipeline as LærbarXR, no cloud dependency). Player has zero network dependency (#1), so it's equally valid opened from a local file or hosted on GitHub Pages — both delivery modes already just work. |
| 6 | Background colour toggle (black/white/grey, persisted) | — (meeting notes) | ✅ Built in the new Builder shell — `body[data-canvas-bg]`, persisted, user-switchable. |
| 7 | Stereoscopic video support + text-overlay conflict | — (meeting notes, client concern) | 🟡 **Builder side done**: per-video `stereo: mono/sbs/tb` field + UI in the Scene editor, persisted through save/export. **Player side still missing**: no stereo texture/eye-layer rendering yet — this is the biggest confirmed remaining gap, see priority list below. Planned approach (documented, not yet built): render video on eye-specific layers, keep text/image overlays on a shared layer visible to both eyes so overlays can never split or double. |
| 8 | Freely-repositionable branch nodes on canvas | — (meeting notes) | ✅ Nodes are already draggable (`startDrag`/`onDrag`, inherited from LaerbarXR) and `x`/`y` are already part of the persisted/exported node data — this looks like it already satisfies the client's ask (visual repositioning only, not touching the actual connections). Will spot-check, not rebuild. |
| 9 | Zoom control (visible %, independent of browser zoom) | — (meeting notes) | ✅ Already present (`.zoom-label` / `zoomBy`), inherited from LaerbarXR. |
| 10 | Project switcher / tabs near project name | — (meeting notes) | ✅ Built — tab strip near the project name, `localStorage`-backed (structure-only persistence, matching the existing autosave convention). |
| 11 | Export flow — real UI, not a stub | — (meeting notes, client asked directly) | ✅ Real summary-modal flow built (title, scene count, media counts, stereo-clip count, access-code warning) before the zip is generated. Zip-generation logic verified correct by direct inspection (see work log); the browser-download step itself couldn't be 100% confirmed from inside this tool (sandbox limitation, see work log) — worth one spot-check from a real browser. |
| 12 | Start-position selection on a recording | 01_09 | 🟡 Builder-side authoring done (`node.startYaw`, UI control). Player-side — actually orienting the camera to that yaw on scene load — not yet verified/wired. |
| 13 | Fade to/from black at clip start/end | 01_10, 02_11 | ✅ Player already played fades; Builder now has the authoring UI too (`fadeIn`/`fadeOut` fields on each scene). |
| 14 | Waypoints within a video | 01_13 | ✅ Builder: full editor exists. Player: Matterport-style disc marker (`createWaypointMarker`), and the full pipeline — real video playback, gaze raycast hitting the marker, arm-delay + dwell timer, navigation to the target node, target node's own video starting — verified genuinely end-to-end with two real ffmpeg-generated test clips and a scripted look-away-then-back gaze (the engine's anti-accidental-selection lock requires that, by design, before a dwell can register on something already under the reticle at the instant arming completes — confirmed as intentional, not a bug, while investigating why a naive static-gaze test never fired). |
| 15 | Text overlay: frame, semi-transparent, position/size/timing | 01_14A/C/D/E | ✅ Inherited wholesale from LaerbarXR's overlay system, which already covers all of this. |
| 16 | Player: local code/PIN entry gates scenario start | 02_07 | ✅ Builder: `project.accessCode` field. Player: `attemptStart()` + `#code-gate-modal`, checked before `startXR`/`startFlatPreview`. This is the actual replacement for LærbarXR's Dashboard-driven start — already done and it's the right shape. |
| 17 | Player: gaze/target click, no controller | 02_04 | ✅ Inherited from LaerbarXR. |
| 18 | Player: Kiosk mode, no Meta account required | 02_05, 02_06 | ✅ True by construction — no Meta SDK/account integration exists anywhere in the codebase (same as LaerbarXR), so there's nothing account-gated to disable. |
| 19 | Player: jump between scenes, auto-reorient on arrival | 02_08A/B/C | 🟡 Scene jumping works (inherited node navigation). Auto-reorientation of forward-facing direction on arrival — so the learner doesn't land facing an arbitrary direction — is genuinely new WebXR work, not present in LaerbarXR either. |
| 20 | Player: smooth-fade vs. direct-cut transition, selectable | 02_09, 02_10 | ✅ `mode = choice.transition \|\| currentNode.transition \|\| 'smooth'` — already implemented. |
| 21 | Video compression in editor | 01_08 | 🟡 `compressVideo()` already exists (MediaRecorder/canvas.captureStream) but is a real quality/compatibility compromise, not true transcoding — treating as a documented known limitation rather than "not started." |
| 22 | User management module | 01_03/01_04 | ⏸️ Deferred — see note above, flagged to Morten. |

**Remaining today**: a real-headset spot-check of everything below marked
✅-by-code-verification-only (stereo rendering, export download, waypoint
end-to-end) — see the open questions section. No known unbuilt features left
in scope.

**Done since the last pass** (all verified, not just written): #7 stereoscopic
rendering in Player (SBS + top-bottom, via WebXR per-eye layers — Builder-side
authoring was already done), #19 auto-reorientation to the authored start-yaw
on every scene jump (also covers the Player half of #12), #14 — the full
gaze-dwell → waypoint → scene-navigation pipeline confirmed end-to-end with
real video playback, not just code review — and the Player landing screen
restyled to the Builder's exact dark tokens (`--bg`/`--surface`/`--accent`/
`.btn`/`.modal-box`), with the LærbarXR fleet-management leftovers Morten
flagged removed entirely: the headset-number / hold-group identity picker,
the auto-download-eval.json-on-session-end flow, its localStorage recovery
UI, and the now-dead `sessionLog` plumbing that only ever fed it. Also
dropped the "Immersive Gaze-Controlled Training" subtitle and ".lxr" jargon
in the upload prompt (both flagged as noise for a real user).

## Open questions for Morten / the client (not guessing on these silently)

1. User management — confirm no-auth-for-now is acceptable (see above).
2. Video compression — confirm the `compressVideo()` MediaRecorder-based
   approach (imperfect but functional) is acceptable for now, or whether true
   transcoding is a hard launch requirement.
3. Export flow — client asked directly what this looks like; building a real
   presentable flow today, will document exact behavior here once built.
4. Stereoscopic/text conflict — turns out to be "build stereoscopic support,
   correctly, from scratch" rather than "fix an existing conflict." Approach:
   render video on eye-specific layers (left/right), keep all text/image
   overlays on a shared layer visible to both eyes — so overlays can never be
   split, doubled, or misaligned by the stereo split. Will confirm this is what
   ships once built.
5. Waypoints-within-a-video (01_13) vs. end-of-clip branching (01_11) — the
   Builder-side editor already treats these as separate, richer than a simple
   end-of-clip choice (multiple hotspots placeable mid-video). Verifying Player
   plays them back correctly rather than re-deciding the interaction model.
6. **New UX (#2) — RESOLVED**: you confirmed explicitly that the Mockup's actual
   shell must ship as the real product ("I want the design to look like the
   mock up... it's what's make it a real product and not just a clone"), with
   all functionality (old + new) ported into it. Done — see work log. One open
   item this raises: is the same design treatment expected on `ELaering-Player`
   too, or does the Player's visual approach stay as-is (headset UI has
   different constraints than a desktop Builder)? Haven't touched Player's UI
   yet, only confirmed its backend feature-parity gaps (stereo rendering,
   auto-reorient).

## Work log

- **2026-09-15, start of day**: access check clean, repo hygiene fixed (see
  above), requirements spec extracted and read in full.
- **2026-09-15**: gap analysis complete — corrected this table against the real
  code (see correction note above). Confirmed directly: stereoscopic video is
  fully unbuilt, Builder has no fade-authoring UI despite Player supporting fade
  playback. Starting on the real remaining-work list now.
- **2026-09-15, mid-day — major decision reversal (per Morten's explicit
  instruction)**: Morten confirmed the Mockup's actual visual shell must ship as
  the real product UI, with all functionality (old + new) ported into it — not
  an incremental reskin. Rebuilt `ELaering-Builder/index.html` from scratch:
  Mockup's dock/card-flow/bottom-sheet shell + the full working backend ported
  from the pre-rewrite Builder (project model, save/export, inspector logic,
  node graph) + new features layered in: **#6 background toggle** (black/white/
  grey, persisted via `body[data-canvas-bg]`), **#10 project switcher** (tab
  strip near project name, `localStorage`-backed, structure-only persistence),
  **#7 stereo authoring** (per-video `stereo: mono/sbs/tb` field + UI — Builder
  side only, see below), **#12 start-position** (`node.startYaw`), **#13 fade
  authoring UI** (`fadeIn`/`fadeOut` fields, Player already played these, Builder
  now lets you set them), **#11 real export flow** (summary modal — title,
  scene count, media counts, stereo-clip count, access-code warning — before the
  actual zip build). Committed and pushed to `origin/main` (`ebf0915`).
  - Caught and fixed before push: a stray `${''}` template-literal artifact in
    literal HTML, and a systemic quote-escaping `SyntaxError` (27 lines across 4
    editor-builder functions mixed single/double JS-string quotes inside inline
    `onchange="..."` attributes — rewrote all 4 to template literals). Verified
    clean via syntax-checking the extracted script and confirming `App` loads on
    the live page.
  - Caught via live UI testing: attaching a video didn't refresh the open Scene
    editor card to show the new Projection/Stereo/Start-position fields — fixed
    by calling `refreshOpenEditor()`/`buildCardGrid()` in `handleVideoPick`.
- **2026-09-15, later** — investigated an apparent export bug: `runExport()`
  reports success (toast, modal closes, no errors) but no `.zip` lands in
  `~/Downloads`, reproduced identically on the local dev server AND on the live
  `https://mohaulik.github.io/ELaering-Builder/` URL. Root-caused by generating
  the zip directly and inspecting it in-memory (bypassing the download step):
  the archive itself is perfectly well-formed — correct `project.json`, correct
  video file, correct metadata (title/nodes/stereo/projection all intact).
  Isolated further by triggering a trivial 17-byte text-file download with the
  identical `a.download` + `a.click()` pattern — that also never reached
  `~/Downloads`. **Conclusion: this is a download-sandboxing limitation of the
  Claude Browser pane tool used for testing, not a bug in the app.** The export
  code path (JSZip generation + blob + `<a download>` trigger) is the same
  proven pattern used in LaerbarXR's Builder. No code change needed here —
  flagging that export should be spot-checked once from a real browser (or
  Morten's own machine) to be fully certain, since that's the one part of this
  I can't 100% verify from inside this tool.
