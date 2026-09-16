# Testing standard — read before building or verifying anything client-facing

This file is binding for any work session on this repo (or its sibling,
`ELaering-Player`), whether that's me continuing later or a fresh session
picking this up cold. It exists because of a real incident, not as a
theoretical precaution — see below.

## The rule

**Any demo/test content built for client-facing testing or delivery, and
any "verified working" claim made about a feature, must be produced and
confirmed by driving the actual UI a real user would use.** Real clicks on
real buttons. Real dropdown selections. Real file pickers (a synthetic
`File` + a dispatched `change` event on the real `<input>` is fine — that's
what a real picker produces — but calling the underlying handler function
directly with a hand-built object is not).

**Never substitute a direct API/console/data call for a user action** —
not `project.someField = x`, not calling an internal handler function
bypassing the DOM event it's normally triggered by, not constructing node/
choice/asset objects by hand and pushing them into the data model.

**If there is no UI path to do something a feature is supposed to support,
that absence IS the bug — report it immediately, in that moment, not a
shortcut to quietly script around so a demo looks complete.** Silently
working around a missing control produces exactly the failure mode below:
a demo that looks finished but that a real client cannot actually build.

Automated testing (this codebase's dev-server tooling, headless browser
tabs, etc.) is fine and expected — "real UI" doesn't mean "only a human
with a mouse." It means: whatever the test does, it must go through the
same buttons, inputs, and event handlers an actual person clicking through
the actual product would go through. Setting `App.project.accessCode` in
a console is not testing the access-code feature. Opening the Lock modal,
typing into the real input, and clicking Save is.

## Why this exists

2026-09-16: a demo scenario built for Morten to hand to the client
included end-screen nodes and a renamed project — both created by
directly mutating the data model (`App.newNode('end', ...)`,
`App.project.title = '...'`) rather than through any button in the
Builder. Neither had a UI path to create them *at all*. The gap almost
reached the client uncaught, inside a demo Morten was about to vouch for
personally. Caught only because Morten tested the live build himself and
asked a direct question about how end nodes get created — not because
the verification process here caught it.

Both gaps are now fixed (`f320cd3`, `a69229e`) — but the process failure
that let a demo *look* complete without *being* buildable by a real
author is the actual root cause, and that's what this file is for.

## What "verified" means, concretely

Before writing "confirmed," "verified," "works," or "tested" about
anything user-facing, be able to answer yes to all of:

1. **Did the click/input/selection go through the real DOM element** a
   user would interact with — not a function call that element happens
   to invoke, called directly instead?
2. **If a file was involved**, was it attached via a `File` object
   dispatched through the real `<input>`'s `change` event (or an
   equivalent real drag/drop), not handed straight to the handler
   function?
3. **If the result is a claim about persistence** (survives reload,
   survives export/reimport, etc.), was that *actually* exercised — a
   real reload, a real re-import of a real exported file — not inferred
   from reading the code?
4. **If something needed didn't have a visible control**, was that
   reported as a bug in the moment, rather than reached around via a
   script or a direct data edit?

If the honest answer to any of these is no, say so plainly instead of
using verification language — "I set this directly because I couldn't
find a control for it — is there one?" is the correct thing to write,
every time, over silently proceeding.

## Scope

This applies to both `ELaering-Builder` and `ELaering-Player` — the two
repos are built and tested together, and a gap in one (e.g., a Player
feature with no Builder-side control to author it) is exactly as
reportable as a gap within a single repo.
