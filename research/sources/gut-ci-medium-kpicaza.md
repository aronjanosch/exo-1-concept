# CI-tested GUT for Godot 4: fast, green, and reliable | by Kpicaza | Medium

[Sitemap](https://medium.com/sitemap/sitemap.xml)

[Open in app](https://play.google.com/store/apps/details?id=com.medium.reader&referrer=utm_source%3DmobileNavBar&source=---top_nav_layout_nav-------------------------------------------)

[Sign in](https://medium.com/m/signin?operation=login&redirect=https%3A%2F%2Fmedium.com%2F%40kpicaza%2Fci-tested-gut-for-godot-4-fast-green-and-reliable-c56f16cde73d&source=post_page---top_nav_layout_nav-----------------------global_nav--------------------)

[Homepage](https://medium.com/?source=---top_nav_layout_nav-------------------------------------------)

[Write](https://medium.com/m/signin?operation=register&redirect=https%3A%2F%2Fmedium.com%2Fnew-story&source=---top_nav_layout_nav-----------------------new_post_topnav--------------------)

[Search](https://medium.com/search?source=---top_nav_layout_nav-------------------------------------------)

[Sign in](https://medium.com/m/signin?operation=login&redirect=https%3A%2F%2Fmedium.com%2F%40kpicaza%2Fci-tested-gut-for-godot-4-fast-green-and-reliable-c56f16cde73d&source=post_page---top_nav_layout_nav-----------------------global_nav--------------------)

[Godot](https://medium.com/tag/godot?source=post_page---header_tags--c56f16cde73d-----------------------------------------)

[Gut](https://medium.com/tag/gut?source=post_page---header_tags--c56f16cde73d-----------------------------------------)

[Ci Cd Pipeline](https://medium.com/tag/ci-cd-pipeline?source=post_page---header_tags--c56f16cde73d-----------------------------------------)

[Indie Game Dev](https://medium.com/tag/indie-game-dev?source=post_page---header_tags--c56f16cde73d-----------------------------------------)

[Tdd](https://medium.com/tag/tdd?source=post_page---header_tags--c56f16cde73d-----------------------------------------)

# CI-tested GUT for Godot 4: fast, green, and reliable

[Link](https://medium.com/@kpicaza?source=post_page---byline--c56f16cde73d-----------------------------------------)

[Kpicaza](https://medium.com/@kpicaza?source=post_page---byline--c56f16cde73d-----------------------------------------)

2 min readOct 19, 2025

[Link](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fvote%2Fp%2Fc56f16cde73d&operation=register&redirect=https%3A%2F%2Fmedium.com%2F%40kpicaza%2Fci-tested-gut-for-godot-4-fast-green-and-reliable-c56f16cde73d&user=Kpicaza&userId=c644aac315a6&source=---header_actions--c56f16cde73d---------------------clap_footer--------------------)

[Link](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Frepost%2Fp%2Fc56f16cde73d&operation=register&redirect=https%3A%2F%2Fmedium.com%2F%40kpicaza%2Fci-tested-gut-for-godot-4-fast-green-and-reliable-c56f16cde73d&user=Kpicaza&userId=c644aac315a6&source=---header_actions--c56f16cde73d---------------------repost_header--------------------)

[Link](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fbookmark%2Fp%2Fc56f16cde73d&operation=register&redirect=https%3A%2F%2Fmedium.com%2F%40kpicaza%2Fci-tested-gut-for-godot-4-fast-green-and-reliable-c56f16cde73d&source=---header_actions--c56f16cde73d---------------------bookmark_footer--------------------)

[Link](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2Fplans%3Fdimension%3Dpost_audio_button%26postId%3Dc56f16cde73d&operation=register&redirect=https%3A%2F%2Fmedium.com%2F%40kpicaza%2Fci-tested-gut-for-godot-4-fast-green-and-reliable-c56f16cde73d&source=---header_actions--c56f16cde73d---------------------post_audio_button--------------------)

![GitHub Actions log showing GUT tests (movement, hazards, skills) passing: 26 tests, all green, Godot 4.5.1.](https://miro.medium.com/v2/resize:fit:680/0*AzhCWGOIGOQA4CFx)

## The pain

Ever had a perfectly green GUT run locally, then CI screams about `RID/ObjectDB`*leaks* and flips the build to red? I wanted a setup where CI fails only on **real test failures**, not on editor shutdown noise or missing imports.

## What I shipped

- A two-step GitHub Actions workflow for Godot **4.5.x**:
 **(1)** headless warm-up **import** → **(2)** headless **GUT** run.
- Reliable headless flags and `GODOT_DISABLE_LEAK_CHECKS=1` to avoid false negatives from leak logs.
- Test discovery via `-gdir=res://tests -ginclude_subdirs` and `-gexit` so a single failing test fails the job.
- Never commit `.godot/`; the import step rebuilds everything on the runner.

## Why this way

I first tried marketplace actions (e.g., GUT runners / Godot testers). They’re great, but didn’t fit this project’s constraints. So i tried calling GUT directly. This exposed two classic pitfalls:

1. **Leak logs ≠ test result**: Godot can print scary `ERROR` lines on exit and return non-zero even when tests passed.
2. **No**`.godot/`**, no classes**: deleting `.godot/` meant `class_name` scripts weren’t registered yet, so GUT bootstrap failed.

## Get Kpicaza’s stories in your inbox

Join Medium for free to get updates from this writer.

[x]

Remember me for faster sign in

So I embraced CI reality: **split the job**. Do a clean, **headless import** (`--headless --import --quit`) so all resources/classes are registered, then run **GUT** in a fully headless stack. Keep the engine quiet with `GODOT_DISABLE_LEAK_CHECKS` so the **exit code reflects tests**, not editor shutdown. Simple to reason about.

## Minimal workflow

```
name: godot-gut-tests

on: [push, pull_request]

jobs:
  run-gut:
    name: Run GUT tests (Godot 4.5.1)
    runs-on: ubuntu-22.04

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Godot
        uses: chickensoft-games/setup-godot@v2
        with:
          version: 4.5.1
          use-dotnet: false

      # Warm-up import so class_name scripts register (no .godot committed)
      - name: Import project (headless)
        run: godot --headless --path . --import --quit

      # Run GUT (headless, stable exit)
      - name: Run GUT
        env:
          GODOT_DISABLE_LEAK_CHECKS: "1"
        run: >
          godot --headless -d
          --display-driver headless
          --audio-driver Dummy
          --disable-render-loop
          --path .
          -s res://addons/gut/gut_cmdln.gd
          -gdir=res://tests
          -ginclude_subdirs
          -gexit
```

> Tip: add a CI badge to your README:
>  `![Run GUT tests]`GitHub Actions log showing GUT tests (movement, hazards, skills) passing: 26 tests, all green, Godot 4.5.1.`(https://github.com/<user>/<repo>/actions/workflows/<file>.yml/badge.svg?branch=main)`

## What I learned

- **Warm-up import is mandatory** if you don’t commit `.godot/`. Make it a separate step.
- Without muting leak logs, **engine exit code ≠ test result**.
- `-ginclude_subdirs` makes discovery robust; `-gexit` gives crisp red/green.

## What’s next

- Optional **cache** for `.godot/imported` to shave seconds.
- A **version matrix** (4.5.x / 4.4.x) to catch regressions early.
- I’ll wrap this into a **reusable example** for the Godot community. I’ll link the example here when it’s live.

If this helped, follow me on [itch.io](https://itch.io/profile/kpicaza)****for future write-ups and builds.

[Godot](https://medium.com/tag/godot?source=post_page---footer_tags--c56f16cde73d-----------------------------------------)

[Gut](https://medium.com/tag/gut?source=post_page---footer_tags--c56f16cde73d-----------------------------------------)

[Ci Cd Pipeline](https://medium.com/tag/ci-cd-pipeline?source=post_page---footer_tags--c56f16cde73d-----------------------------------------)

[Indie Game Dev](https://medium.com/tag/indie-game-dev?source=post_page---footer_tags--c56f16cde73d-----------------------------------------)

[Tdd](https://medium.com/tag/tdd?source=post_page---footer_tags--c56f16cde73d-----------------------------------------)

[Some rights reserved](http://creativecommons.org/licenses/by/4.0/)

creative-commons-by-21px

[Link](https://medium.com/@kpicaza?source=post_page---post_author_info--c56f16cde73d-----------------------------------------)

[Link](https://medium.com/@kpicaza?source=post_page---post_author_info--c56f16cde73d-----------------------------------------)

[Written by Kpicaza](https://medium.com/@kpicaza?source=post_page---post_author_info--c56f16cde73d-----------------------------------------)

[107 followers](https://medium.com/@kpicaza/followers?source=post_page---post_author_info--c56f16cde73d-----------------------------------------)

[77 following](https://medium.com/@kpicaza/following?source=post_page---post_author_info--c56f16cde73d-----------------------------------------)

PHP passionate developer

[Help](https://help.medium.com/hc/en-us?source=post_page-----c56f16cde73d-----------------------------------------)

[Status](https://status.medium.com/?source=post_page-----c56f16cde73d-----------------------------------------)

[About](https://medium.com/about?autoplay=1&source=post_page-----c56f16cde73d-----------------------------------------)

[Careers](https://medium.com/jobs-at-medium/work-at-medium-959d1a85284e?source=post_page-----c56f16cde73d-----------------------------------------)

[Press](mailto:pressinquiries@medium.com)

[Blog](https://blog.medium.com/?source=post_page-----c56f16cde73d-----------------------------------------)

[Store](https://medium.com/store)

[Privacy](https://policy.medium.com/medium-privacy-policy-f03bf92035c9?source=post_page-----c56f16cde73d-----------------------------------------)

[Rules](https://policy.medium.com/medium-rules-30e5502c4eb4?source=post_page-----c56f16cde73d-----------------------------------------)

[Terms](https://policy.medium.com/medium-terms-of-service-9db0094a1e0f?source=post_page-----c56f16cde73d-----------------------------------------)

[Text to speech](https://speechify.com/medium?source=post_page-----c56f16cde73d-----------------------------------------)