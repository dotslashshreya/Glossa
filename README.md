<img src="build/icon.svg" alt="" width="96" align="right">

# Glossa

**An invisible speech coach in your corner.**

Glossa floats a small coaching pill on your screen while you talk and tells you,
live, how you're doing: how much you're talking, how fast, how many "ums", and
when you've been going on too long. Only you can see it.

[![CI](https://github.com/dotslashshreya/Glossa/actions/workflows/ci.yml/badge.svg)](https://github.com/dotslashshreya/Glossa/actions/workflows/ci.yml)
[![Latest release](https://img.shields.io/github/v/release/dotslashshreya/Glossa)](https://github.com/dotslashshreya/Glossa/releases/latest)
[![License: Apache-2.0](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](LICENSE)
![Platforms: Windows and macOS](https://img.shields.io/badge/platforms-Windows%20%7C%20macOS-lightgrey)

Works with Zoom, Google Meet, Microsoft Teams, phone calls, or no call at all.
Glossa listens to your microphone only, so it needs no plugins or meeting-app
integrations.

Interviewing for a job? Pitching a client? Practising a talk? Keep a coach in
the corner of your screen that catches your filler words and your pace as you
speak, and hides itself from screen sharing on operating systems that support
it.

**Private. Local. Lightweight.** No account, no cloud, no internet connection.
Your audio is never saved, and never leaves your computer.

<!-- A short demo GIF goes here. Keep images out of the repo: drag the GIF into
     any GitHub issue or PR comment, then paste the link GitHub gives you here. -->

## Table of contents

- [Features](#features)
- [Install](#install)
- [How it works](#how-it-works)
- [Usage](#usage)
  - [Running a session](#running-a-session)
  - [The overlay](#the-overlay)
  - [Monologue alert](#monologue-alert)
  - [Report card](#report-card)
  - [Choosing a microphone](#choosing-a-microphone)
- [Verify the invisibility](#verify-the-invisibility)
- [How accurate is it?](#how-accurate-is-it)
- [Storage and privacy](#storage-and-privacy)
- [Platform support and known limitations](#platform-support-and-known-limitations)
- [Troubleshooting](#troubleshooting)
- [Development](#development)
- [Building and releasing](#building-and-releasing)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [License](#license)

## Features

Glossa v1 does five things, and does them well:

| Feature                   | What you see                                                                                                                                          |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Talk-time meter**       | How much of the session you've spent speaking, plus a "speaking now" dot.                                                                             |
| **Pace**                  | Your words per minute, with a gentle hint when you're rushing or dragging.                                                                            |
| **Filler-word counter**   | "um", "uh", "you know", "basically"… counted live, with your top offenders.                                                                           |
| **Monologue alert**       | A soft pulse when you've been talking for over a minute without a break.                                                                              |
| **Post-call report card** | Each session compared with the last, e.g. _Filler words: 1.6/min, ↓ from 2.6 · better_, plus your trend. Copy it as an image and share your progress. |

And around them:

- **Hidden from screen sharing.** The overlay asks the operating system to leave
  it out of screenshots, recordings and screen shares
  ([details and limits](#verify-the-invisibility)).
- **Always on top.** The overlay stays above every window, including
  full-screen meeting apps, and appears without stealing focus from your call.
- **Fully offline.** The speech models ship inside the app and run on your CPU.
- **Mic-only.** No system-audio capture and no meeting-app plugins.
- **Free and open source.** Apache-2.0 licensed. No subscription, ever.

That's it, on purpose. New features get added when users ask for them (see
[Roadmap](#roadmap)).

## Install

Download the installer for your computer from the
[latest release](https://github.com/dotslashshreya/Glossa/releases/latest):

| Your computer                                       | Download                     |
| --------------------------------------------------- | ---------------------------- |
| **Windows** 10 or 11 (64-bit)                       | `Glossa-<version>-setup.exe` |
| **Mac** with Apple Silicon (M1 or newer), macOS 13+ | `Glossa-<version>-arm64.dmg` |

Intel Macs aren't supported: the speech engine Glossa uses has no Intel build.

**Why your computer warns you, and what to do.** Operating systems fully trust
apps that are signed with a paid certificate. Glossa is free and has no budget
for one, so you'll see a warning the first time you open it. Every installer is
built by GitHub from the public source code of a tagged release, and you can
read [exactly how](.github/workflows/release.yml).

- **Windows:** run the installer. If you see _"Windows protected your PC"_,
  click **More info**, then **Run anyway**. Glossa installs just for you (no
  admin password) and opens. If it can't hear you, turn on **Settings →
  Privacy & security → Microphone → Let desktop apps access your microphone**.
- **Mac:** open the `.dmg` and drag **Glossa** into **Applications**, then open
  it. macOS says it can't verify Glossa: click **Done**, open **System Settings
  → Privacy & Security**, scroll down, click **Open Anyway** next to the
  message about Glossa, and confirm. (On macOS 13 or 14 you can instead
  Control-click Glossa in Applications and choose **Open**.) Then allow
  microphone access when asked.

You only do this once. Glossa never checks for updates (it never goes online);
to hear about new versions, click **Watch → Custom → Releases** on this page.

**Uninstalling:** on Windows, use **Settings → Apps → Installed apps**; on a
Mac, drag Glossa from Applications to the Trash. Your session history stays in
`%APPDATA%\Glossa` or `~/Library/Application Support/Glossa` until you delete
that folder.

**Tip:** use headphones. Glossa measures _your_ voice from your microphone; if
the other people on your call come out of your speakers, the mic hears them too.

## How it works

```
Microphone ─► 32 ms audio chunks ─► Silero VAD: "is this speech?"
                                        │
                     talk time, turns ◄─┤
                                        └─► finished phrase ─► Whisper tiny.en
                                                                   │
                         word and filler counts (text dropped) ◄───┘
                                        │
                     overlay and app window, updated 4× per second
```

1. The app window captures your microphone at 16 kHz in 32 ms chunks.
2. A separate engine process asks **Silero VAD** whether each chunk is speech.
   That drives the talk-time meter, the "speaking now" dot and the monologue
   alert.
3. When you finish a phrase, its audio goes to **Whisper tiny.en**, running
   locally. The transcript is immediately reduced to word and filler counts,
   which drive pace and the filler counter. The text itself is thrown away.
4. Four times a second, the engine sends fresh numbers to the overlay and the
   app window.

Heavy work runs in its own hidden process, so it never freezes a window, and a
crash there can't take the app down. The full tour, with every threshold and
why it was chosen, is in [docs/architecture.md](docs/architecture.md).

## Usage

### Running a session

1. Open Glossa, pick your microphone (or leave **System default**), and click
   **Start session**.
2. The overlay appears in the top-right corner of your main screen. Talk as
   usual.
3. Stop with **Stop session** in the app window, the **■** button on the
   overlay, or by closing either window. Glossa transcribes your last sentence
   before it stops, so it still counts.
4. Your report card appears in the app window.

### The overlay

A small pill that floats above every other window, including full-screen apps:

```
● 64% talk · 152 wpm · 3 fillers · 4:05 · ■
```

| Part        | What it shows                                                                                                              |
| ----------- | -------------------------------------------------------------------------------------------------------------------------- |
| **●**       | Lights up while you're speaking.                                                                                           |
| **Talk %**  | Share of the session you've spent speaking.                                                                                |
| **wpm**     | Your pace over the last minute. Hover for a hint: _a little slow_ (under 110), _good pace_, or _a little fast_ (over 170). |
| **Fillers** | Filler words so far. Hover to see which ones you use most.                                                                 |
| **Elapsed** | How long the session has been running.                                                                                     |
| **■**       | Stops the session.                                                                                                         |

Drag the pill anywhere on screen. Pace stays blank until Glossa has heard about
5 seconds of speech, so it never guesses from a single word.

### Monologue alert

Talk for more than **a minute** without a 2-second pause and a soft ring pulses
around the overlay until you stop for breath. The app window also says how long
you've been going. There's no sound, because your microphone would hear it. If
your system is set to reduce motion, the ring stays still instead of pulsing.

### Report card

When a session ends, Glossa compares it with your previous one:

| Number       | Called "better" when…                                              |
| ------------ | ------------------------------------------------------------------ |
| Talk time    | Never judged: presenting and listening both have their place       |
| Pace         | It moves towards the comfortable range (110–170 wpm)               |
| Filler words | There are fewer per minute (needs 30 s of speech for a rate)       |
| Longest turn | Less of it is over a minute; shorter turns under a minute are fine |

Every change shows an arrow and the word "better" or "worse", not just a
colour. A sparkline shows your filler rate over your last 10 sessions.
**Copy as image** puts the card on your clipboard as a PNG, ready to paste
anywhere. Sessions with less than 5 seconds of speech aren't saved, so a quick
mic check never becomes "last time".

### Choosing a microphone

Pick any input from the **Microphone** list in the app window before you
start. Glossa remembers your choice. Headphones work best: they stop the other
side of your call from leaking into your mic.

## Verify the invisibility

1. Start a session.
2. Start a screen recording (**Win+Shift+R** on Windows 11, **QuickTime → File →
   New Screen Recording** on macOS, or OBS), or share your screen in a Zoom,
   Meet or Teams call.
3. You still see the overlay. The recording or the people on the call should
   not.

How well this works depends on the operating system, not on Glossa:

- **Windows 10 version 2004 (May 2020 update) or later:** the overlay is left
  out of screen capture.
- **macOS:** depends on the macOS version and on how the meeting app captures
  your screen. Some apps can still show it.

Always test with your own setup before an important call.

## How accurate is it?

On recordings of real meetings that were never used to tune it, Glossa catches
**99% of ums and uhs** (79 of 80), and **96%** of the ones it reports are real.

|                 | Caught (recall)    | Correct (precision)          |
| --------------- | ------------------ | ---------------------------- |
| um, uh…         | **99%** (79 of 80) | **96%**                      |
| like, you know… | 100% (5 of 5)      | 71% (5 of 7, a small sample) |

The benchmark is open: see [eval/README.md](eval/README.md) and run it yourself
with `pnpm eval`. Known limits: Whisper sometimes swaps "um" and "uh" (the total
stays right), and lexical fillers are rare in the benchmark, so their numbers
are rough.

## Storage and privacy

- **No network at all.** Every request that would leave your computer is
  blocked, and lint rules stop network code from being merged.
- **Audio stays in memory.** Only the last 30 seconds are kept, so a finished
  phrase can be transcribed; older audio is overwritten. Nothing is written to
  disk.
- **Transcripts are never kept.** They're reduced to word and filler counts
  the moment they arrive.
- **Only numbers are saved.** Each session's totals (length, talk %, pace,
  filler counts, longest turn) go into `sessions.jsonl` in
  `%APPDATA%\Glossa` (Windows) or `~/Library/Application Support/Glossa`
  (macOS). Delete that file to clear your history.
- **Microphone only.** Glossa grants itself the microphone and nothing else:
  no camera, location, notifications or screen capture.
- **No accounts, analytics, crash reports or update checks.**

The full list of what Glossa handles, and how each promise is enforced, is in
[docs/privacy.md](docs/privacy.md).

## Platform support and known limitations

| Behaviour                      | Windows 10/11 (x64)                      | macOS 13+ (Apple Silicon) |
| ------------------------------ | ---------------------------------------- | ------------------------- |
| Hidden from screen capture     | Yes, on version 2004 (May 2020) or later | Yes                       |
| Overlay above full-screen apps | Yes                                      | Yes, on every Space       |
| Microphone permission          | Windows privacy setting for desktop apps | macOS asks once           |
| Installer                      | `.exe`, installs for you only (no admin) | `.dmg`                    |

- **Intel Macs** aren't supported: the speech engine has no Intel build.
- **Linux** works for development and unit tests, but isn't packaged.
- **English only** for now: the speech model is Whisper tiny.en.
- **Your voice only.** Glossa can't hear the other side of a call, so talk time
  is your share of the session, not of the conversation.

## Troubleshooting

<details>
<summary><b>Glossa can't hear me</b></summary>

Check that the right microphone is selected in the app window. On Windows, turn
on **Settings → Privacy & security → Microphone → Let desktop apps access your
microphone**. On macOS, allow Glossa in **System Settings → Privacy & Security →
Microphone**.

</details>

<details>
<summary><b>Windows or macOS won't open the app</b></summary>

That's the unsigned-app warning. Follow the steps under [Install](#install):
**More info → Run anyway** on Windows, **Open Anyway** in Privacy & Security on
macOS.

</details>

<details>
<summary><b>The overlay shows up in my screen share</b></summary>

On Windows, update to version 2004 or later. On macOS, it depends on your macOS
version and the meeting app; see [Verify the invisibility](#verify-the-invisibility).

</details>

<details>
<summary><b>Pace shows a dash</b></summary>

Pace needs about 5 seconds of speech before it shows a number. Keep talking.

</details>

<details>
<summary><b>My talk time or fillers include other people</b></summary>

Their voices are coming out of your speakers and into your microphone. Use
headphones.

</details>

## Development

You need **Node.js 22** (see `.nvmrc`) and **pnpm 10**. Glossa uses only
prebuilt packages, so there's no C++ compiler, Python or Rust to install.

```bash
git clone https://github.com/dotslashshreya/Glossa.git
cd Glossa
pnpm install
pnpm fetch-models   # one time: ~46 MB of models, verified by checksum
pnpm dev            # run the app with live reload
```

| Command                | What it does                                                          |
| ---------------------- | --------------------------------------------------------------------- |
| `pnpm dev`             | Run the app with live reload.                                         |
| `pnpm check`           | Formatting, lint, type-check and unit tests: the same checks CI runs. |
| `pnpm test`            | Unit tests (Vitest).                                                  |
| `pnpm test:e2e`        | End-to-end tests: the real app with a fake microphone (Playwright).   |
| `pnpm eval`            | Accuracy benchmark on real meetings.                                  |
| `pnpm build:unpack`    | A runnable app folder, without an installer.                          |
| `pnpm build:installer` | An installer for your OS (`.exe` or `.dmg`) in `release/`.            |

CI runs the checks on Ubuntu, then builds the app and runs the end-to-end tests
against the packaged app on Windows and macOS.

**Architecture** (full details in [docs/architecture.md](docs/architecture.md)):

```
src/core/       pure metrics logic: talk time, pace, fillers, turns, report card
src/engine/     hidden process that runs the models (Silero VAD, Whisper)
src/main/       Electron main process: windows, sessions, history, security
src/preload/    the narrow bridge between pages and the main process
src/renderer/   the app window (React) and the overlay
src/shared/     types shared across processes
e2e/            end-to-end tests with a fake microphone
eval/           the open accuracy benchmark
scripts/        model download (pinned, checksummed) and icon rendering
```

Everything that decides what the numbers are lives in `src/core/`, with no
Electron, Node or browser APIs. Durations are counted in audio frames, never
from the system clock, so a session replays identically in tests.

## Building and releasing

`pnpm build:installer` builds an installer for your own OS. Official releases
are automatic: after each merge to `main`, release-please opens a release PR
from the Conventional Commit titles. Merging it tags the version, and GitHub
Actions builds the Windows and macOS installers and attaches them to the
release. The [maintainer guide](docs/maintainer-guide/README.md) walks through
it.

## Roadmap

Glossa v1 is deliberately small. Ideas that are **not in v1 (yet)**:

- Invisible talking points (private notes on the overlay)
- Agenda timer
- Two-sided talk ratio using system audio
- Custom filler-word lists and other languages
- More settings (pace targets, monologue length)

Want one of these, or something else? Upvote an existing
[feature request](https://github.com/dotslashshreya/Glossa/issues?q=is%3Aissue+label%3Aenhancement)
or open a new one. The most-requested ideas get built next.

## Contributing

Contributions of all sizes are welcome: bug reports, docs, filler words from
your dialect, test recordings, and code. Start with
[CONTRIBUTING.md](CONTRIBUTING.md). In short:

- Open or comment on an issue before writing code.
- Branch from `main` (`feat/…`, `fix/…`, `docs/…`) and keep each PR to one
  change.
- Use [Conventional Commits](https://www.conventionalcommits.org) for your PR
  title (`feat: …`, `fix: …`). CI checks it, and it becomes the changelog entry.
- `pnpm check` must pass. Changes to `src/core/` need unit tests, and changes
  to filler rules need `pnpm eval` numbers from before and after.

Everyone taking part agrees to our [Code of Conduct](CODE_OF_CONDUCT.md).
Found a security problem? Please report it privately; see
[SECURITY.md](SECURITY.md).

## License

Glossa is licensed under the [Apache License 2.0](LICENSE).

_The name: γλῶσσα (glôssa) is Ancient Greek for "tongue" and "language"._
