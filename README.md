# E-Læring — Player

Gaze-controlled WebXR playback of 360°/180° training scenarios on the Meta Quest 3.
Open the URL in the headset browser, upload the scenario `.zip` exported by the
Builder, and enter immersive VR — no controllers, no native app, no install, no
Meta account required.

An independent client product ("E-Læring") for gaze-controlled WebXR playback.

## Key features

- **No remote-start / relay / Dashboard.** No live-relay subsystem, no
  heartbeats, no remote start/stop commands. This Player never contacts any
  server — every run is fully local.
- **Access-code gate (spec req. ID:02_07).** If the scenario was exported with an
  access code set in the Builder, the Player now shows a code-entry screen before
  Start/Preview will launch it. A scenario with no code set skips the gate.
- Local results (`*_eval.json`) are still written and downloaded/saved on the
  device at the end of a run — there is simply no dashboard to auto-push them to;
  re-download the last saved run any time from the landing screen.

## Dependencies

Vendored — this page loads nothing from a third-party CDN, and makes no network
requests of any kind.

| File | Version | License |
| --- | --- | --- |
| `vendor/jszip.min.js` | JSZip 3.10.1 | MIT / GPLv3 |
| `vendor/three.min.js` | three.js r150 | MIT |
