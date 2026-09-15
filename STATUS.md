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
| 2 | New, sleeker builder UX (from Mockup design) | 01_05 | 🟡 Current Builder is a working dark reskin of the LaerbarXR layout (top toolbar + left inspector), not yet the Mockup's bottom-dock/card-flow design. The Mockup itself is **UI-only — no real logic wired to it at all** (fake hardcoded nodes, Undo/Export buttons with no click handlers, nodes aren't draggable). Decision: port the Mockup's visual language into the *working* Builder incrementally rather than risk a full rip-and-replace that could leave it non-functional — see decision note below. |
| 3 | 180° video support (builder + player) | 01_07 | ✅ Builder: `setVideoProjection()`, per-asset `projection` field. Player: `makeProjectionGeometry()`/`applyVideoProjection()` (half-dome vs full sphere). |
| 4 | Voice-over / narration support | 01_12A/B/C, 01_14B | ✅ Builder: `node.narration` model, `updateNarration()`, editor UI. Player: `narrationAudio`, "Hear again" replay button. |
| 5 | Offline export zip + online-hostable player | (client Q confirmed) | ✅ Export is a local zip (same JSZip pipeline as LærbarXR, no cloud dependency). Player has zero network dependency (#1), so it's equally valid opened from a local file or hosted on GitHub Pages — both delivery modes already just work. |
| 6 | Background colour toggle (black/white/grey, persisted) | — (meeting notes) | ⬜ Not found anywhere in either app. |
| 7 | Stereoscopic video support + text-overlay conflict | — (meeting notes, client concern) | ⬜ **Bigger than it looked**: zero matches for "stereo" anywhere in either app — stereoscopic playback isn't built at all yet, so the client's concern is about a feature that doesn't exist rather than a rendering-order bug in an existing one. Real 180° VR footage is commonly captured as stereo pairs, so this is worth building properly, not skipping. |
| 8 | Freely-repositionable branch nodes on canvas | — (meeting notes) | ✅ Nodes are already draggable (`startDrag`/`onDrag`, inherited from LaerbarXR) and `x`/`y` are already part of the persisted/exported node data — this looks like it already satisfies the client's ask (visual repositioning only, not touching the actual connections). Will spot-check, not rebuild. |
| 9 | Zoom control (visible %, independent of browser zoom) | — (meeting notes) | ✅ Already present (`.zoom-label` / `zoomBy`), inherited from LaerbarXR. |
| 10 | Project switcher / tabs near project name | — (meeting notes) | ⬜ Not found. |
| 11 | Export flow — real UI, not a stub | — (meeting notes, client asked directly) | 🟡 Same single-button export as LaerbarXR's reference — functional, but not the presentable flow the client asked to see. |
| 12 | Start-position selection on a recording | 01_09 | ⬜ Not found in Builder or Player. |
| 13 | Fade to/from black at clip start/end | 01_10, 02_11 | 🟡 Player already plays fades (`fadeTo`/`tickFade`, wired into `selectChoice` and end-of-scenario). Builder has **no UI to author the fade timing** — needs a control added so it's not just a fixed default. |
| 14 | Waypoints within a video | 01_13 | 🟡 Builder: full editor exists (`addWaypoint`/`updateWaypoint`/`removeWaypoint` + UI). Player-side click-through playback of a waypoint mid-video needs a direct check — not yet confirmed working end to end. |
| 15 | Text overlay: frame, semi-transparent, position/size/timing | 01_14A/C/D/E | ✅ Inherited wholesale from LaerbarXR's overlay system, which already covers all of this. |
| 16 | Player: local code/PIN entry gates scenario start | 02_07 | ✅ Builder: `project.accessCode` field. Player: `attemptStart()` + `#code-gate-modal`, checked before `startXR`/`startFlatPreview`. This is the actual replacement for LærbarXR's Dashboard-driven start — already done and it's the right shape. |
| 17 | Player: gaze/target click, no controller | 02_04 | ✅ Inherited from LaerbarXR. |
| 18 | Player: Kiosk mode, no Meta account required | 02_05, 02_06 | ✅ True by construction — no Meta SDK/account integration exists anywhere in the codebase (same as LaerbarXR), so there's nothing account-gated to disable. |
| 19 | Player: jump between scenes, auto-reorient on arrival | 02_08A/B/C | 🟡 Scene jumping works (inherited node navigation). Auto-reorientation of forward-facing direction on arrival — so the learner doesn't land facing an arbitrary direction — is genuinely new WebXR work, not present in LaerbarXR either. |
| 20 | Player: smooth-fade vs. direct-cut transition, selectable | 02_09, 02_10 | ✅ `mode = choice.transition \|\| currentNode.transition \|\| 'smooth'` — already implemented. |
| 21 | Video compression in editor | 01_08 | 🟡 `compressVideo()` already exists (MediaRecorder/canvas.captureStream) but is a real quality/compatibility compromise, not true transcoding — treating as a documented known limitation rather than "not started." |
| 22 | User management module | 01_03/01_04 | ⏸️ Deferred — see note above, flagged to Morten. |

**Real remaining work, in priority order**: #7 (stereoscopic — biggest unknown),
#11 (export flow UI), #6 (background toggle), #10 (project switcher), #12
(start-position), #19 (auto-reorient), #13 (Builder-side fade authoring), #14
(verify waypoint playback), #2 (incremental UX polish from the Mockup).

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
6. **New UX (#2)**: decided to evolve the current *working* Builder toward the
   Mockup's look incrementally, rather than swap in the Mockup's shell wholesale
   — the Mockup has zero real logic behind it (confirmed: no file I/O, no
   undo/export handlers, nodes not draggable), so a full swap today risks
   trading a working tool for a broken-looking one. Flagging this judgment call
   explicitly per your instruction — if you'd rather I attempt the full visual
   replacement even at higher risk, say so and I'll pivot.

## Work log

- **2026-09-15, start of day**: access check clean, repo hygiene fixed (see
  above), requirements spec extracted and read in full.
- **2026-09-15**: gap analysis complete — corrected this table against the real
  code (see correction note above). Confirmed directly: stereoscopic video is
  fully unbuilt, Builder has no fade-authoring UI despite Player supporting fade
  playback. Starting on the real remaining-work list now.
