# Sources to check — EXO-1 (working title)

Status: list for manual checking. These sources could not be fetched during research (blocked, rate-limited, or not found), or a claim rests only on secondary sources. Results can be added to `FEASIBILITY.md` with a **[verified]** tag. Transcripts of videos are welcome.

## Technical

| What to check | Where | Why |
|---|---|---|
| No Man's Sky: coordinates, precision, bases, ships | GDC 2017 talk "Continuous World Generation in No Man's Sky": https://www.gdcvault.com/play/1024265/Continuous-World-Generation-in-No (also look for the video; a transcript would be best) | No slides or transcript found; all claims about coordinates are unverified |
| Outer Wilds: reference frames, moving bodies | Search for developer talks or interviews on the physics and "reference frame" | Only forum guesses found; not critical while planets do not orbit |
| Headless stall in Godot | https://github.com/godotengine/godot/issues/122707 (read via `gh`, see `FEASIBILITY.md`) | Checked: no confirmed hang, only a possible FPS cap; recheck on our version |
| Physics determinism | https://github.com/godotengine/godot/issues/112976 (read via `gh`) | Checked: maintainers say physics is non-deterministic in general; issue is about 2D, check 3D/Jolt separately |
| `.tscn`/`.tres` format and embedded scripts | Godot docs page on the TSCN file format (the URL returned 404, find the current location); proposal https://github.com/godotengine/godot-proposals/issues/4925 | Statements about `ext_resource`/`sub_resource` embedding rest on format knowledge |
| `str_to_var` and `ConfigFile` with embedded objects | Godot class reference | Behaviour not checked; must not be used in the content path |
| Scene merging tools | https://github.com/derkork/tscnmerge (archived, GPL), gdmerge, https://github.com/godotengine/godot-proposals/issues/1281 | Check usefulness before relying on them |
| CI article on GUT | https://medium.com/@kpicaza/ci-tested-gut-for-godot-4-fast-green-and-reliable-c56f16cde73d | Not read |
| Lavapipe frame rate under CI | Search for projects running Godot with lavapipe/Xvfb | 5-7 FPS claim only from search snippets |
| Jolt default since 4.6, reverse-Z renderers | Official Godot 4.6 release notes; reverse-Z notes for 4.3 and which renderers use it | 4.6 confirmed on the release page; renderer coverage open |

## Design

| What to check | Where | Why |
|---|---|---|
| Starsector economy and logistics | https://fractalsoftworks.com/tag/economy/ (the "Revisiting the Economy" and logistics posts); the Starsector wiki | Primary sources blocked; statements come from search snippets |
| Dead Space diegetic UI | https://medium.com/inbeta/dead-space-ui-design-lessons-for-vr-39aa9e976ca8 | Blocked |
| Schedule I developer interview or postmortem | Search for interviews, GDC or studio posts | None found; only Wikipedia, a studio blog and Steam threads |
| POI density numbers | Postmortems of Outer Wilds, Valheim, Subnautica | Numbers in the research are guesses; use only as playtest starting values |

## Legal, naming, assets

| What to check | Where | Why |
|---|---|---|
| Existing game "Exo One" | https://store.steampowered.com/app/773370/Exo_One/ , https://en.wikipedia.org/wiki/Exo_One | Same field, near-identical sound. See `DECISIONS.md` |
| Trademark registers for the new name | TMview (covers EUIPO, DPMA and USPTO) | Registers were not reachable by tool; research is incomplete. No legal advice |
| Meshy terms of service | Meshy website | Returned 404; licence info only from comparison pages |
| Tripo free-tier output licence | Tripo terms and pricing page | Sources contradict each other (non-commercial versus CC BY 4.0) |
| Hunyuan3D 2.1 licence territories | Licence text | Reported to exclude the EU, UK and South Korea; secondary source |
| Kenney and Quaternius packs actually used | Licence file inside each pack | Per-pack details not checked |
| AI-generated asset copyright | Primary legal sources (US Copyright Office reports, German UrhG commentary) | Current picture rests on secondary sources |
| Godot AI policy | https://contributing.godotengine.org/en/latest/pull_requests/pull_request_guidelines.html | Relevant for upstream bug reports only |
| Godot asset library malicious cases | Search | No case found: a gap in the search, not an all-clear |
