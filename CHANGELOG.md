# Changelog

## 2026-11-21 — Local-only control verified for four plugs

- Expanded the project to include **two Tapo P110** devices via Home Assistant's built-in TP-Link Smart Home integration, including third-party compatibility and manual IP-based device addition.
- Added a reusable OPNsense `IOT_LOCAL_ONLY` alias and IPv4 Internet-blocking policy. Two Tapo P110s and both TCP Smart plugs were grouped in the same alias after verifying their local control paths.
- Cleared selected existing states so the block took effect. Both manufacturers' mobile apps reported their respective plugs offline; all four continued switching via Home Assistant on the LAN and over remote HA access on 4G/5G.
- Verified Tapo power readings against an attended three-level load (approximately 700/1,000/1,700 W). Long-term kWh accuracy and charting were not tested.
- Reworked the root README into a multi-brand entry point, kept the original TCP investigation in `docs/tcp-case-study.md`, and added Tapo and OPNsense walkthroughs. The repository itself has **not yet been renamed**.


## 2026-10-25 — Reader-focused editorial pass

- Reworked the README opening into a plain-English explanation of the problem, solution and untested cloud-isolation question.
- Added a first-time-reader navigation section and definitions of key technical terms.
- Corrected device labels in the diagram and phone-test narrative.
- Updated security documentation to distinguish the reconstructed example from the four sanitised historical inspection scripts.


## 2026-10-25 — CI maintenance and full CLI block accounting

- Updated GitHub Actions dependencies to checkout v7 and setup-python v7 for Node.js 24-compatible runners.
- Reviewed all 27 previously excluded historical blocks; published a disposition register while keeping sensitive raw source material private.
- Complete accounting now covers all 64 rendered code/output blocks (37 public excerpts, 27 reviewed exclusions).
- CI success on these latest commits still requires confirmation from GitHub Actions.


## 2026-10-25 — Repository maintenance

- Standardised public device labels as TCP Plug 1 and TCP Plug 2, retaining their original device mapping.
- Published sanitised historical Python artefacts and corrected the LAN-test descriptive mismatch to match its original target constants; no live retest claimed.
- Added read-only CI syntax checks and offline unit tests for the reconstructed example.
- Removed the obsolete directory placeholder and updated source-provenance documentation.


## 2026-10-24 — Home Assistant integration completed

- Both TCP Plug 1 and TCP Plug 2 reported working as LocalTuya switch entities via the Debian Home Assistant Container deployment.
- DPS 1 selected for the relay; Tuya protocol 3.1; no Tuya cloud API account configured.
- Original TCP Smart app pairing preserved. Optional cloud-isolation/latency checks left for follow-up.

## 2026-10-21 — Direct LAN control verified

- Recovered per-device local-key candidates from an inspectable, user-controlled Android emulator after other methods failed.
- Correlated candidates with the appropriate device records; locally validated encrypted switching for both plugs using Python and TinyTuya.
- Original OEM app visibly reflected a separate local switch test on the TCP Plug 2.

## 2026-10-20 — Initial investigation

- Identified TCP/6668 accessibility and local protocol 3.1 status readings.
- Began OEM app/APK investigation after confirming keyless reads did not provide encrypted switching.

## Starter repository package

- Documentation prepared and a **reconstructed, sanitised** test example added. This changelog records historical project activity; it does not claim these example files were committed or used on the original dates.
