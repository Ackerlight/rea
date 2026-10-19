---
name: use-rea
description: Brian's REA workflow — turn "Use REA and analyze <app or website>" (or "REA this", "how does <app> do <feature>") into a feature-harvest report that explains how the target works, with evidence, and how to build the useful parts into Brian's own apps. Use whenever the user says "use REA", "REA analyze", or asks to study an installed app or website's feature for reuse in his work. Works on the Mac and the Windows PC.
---

# Use REA: analyse a target, harvest ideas for Brian's apps

REA is registered for all projects as the `rea` MCP server; the `rea` CLI is the fallback.
Follow the `reverse-engineer-anything` skill for tool routing and evidence rules; this
skill adds Brian's workflow on top. Source of truth for this file:
`personal/skills/use-rea/SKILL.md` in the Ackerlight/rea fork.

## Which machine am I on?

|                 | Mac (primary)                                                 | Windows PC                                                 |
| --------------- | ------------------------------------------------------------- | ---------------------------------------------------------- |
| REA             | local build of the fork, `~/bin/rea`                          | npm package `rea-agents` (global)                          |
| Ghidra          | `~/Applications/ghidra_12.1.4_PUBLIC`, full support           | `C:\Tools\ghidra_12.1.4_PUBLIC`, experimental Windows mode |
| Native targets  | Mac apps (Mach-O, Universal) and Windows PE `.exe`/`.dll`     | Windows PE `.exe`/`.dll` only                              |
| Runtime capture | macOS processes                                               | not supported, so use static analysis                      |
| Long runs       | wrap in `~/bin/heavy-run --uses cpu "claude: REA <target>" …` | run directly                                               |
| Reports         | `~/Developer/Stand-Alone/REA-Notes/`                          | `%USERPROFILE%\Developer\REA-Notes\`                       |

If the PC can't analyse a target (for example a Mac app), say so and suggest running the
request in a session on the Mac (Remote Control or SSH).

## 1. Pin down the target and the goal

- **App by name:**
  - Mac: look in /Applications and ~/Applications, or use
    `mdfind "kMDItemKind == 'Application'"`.
  - Windows: look in `C:\Program Files`, `C:\Program Files (x86)` and
    `%LOCALAPPDATA%\Programs` (where most Electron apps install).
  - If more than one copy matches, ask which one.
- **Choose the route:**
  - Electron (has `resources\app.asar` or `Contents/Resources/app.asar`) →
    `analyze_javascript_application` on the asar.
  - .NET assembly → `inspect_managed_artifact` (needs no Ghidra).
  - Native binary → Ghidra. If REA reports no provider, run `rea doctor --json` and fix
    what it names, using the machine table above.
  - Installers (`setup.exe`, `.msi`, `.pkg`, `.dmg`): extract first (7-Zip), then point
    REA at the real program inside.
- **Website:** use REA's browser tools (`capture_browser_scenario` / `analyze_web_bundle`).
  Observe only: no sign-ins, purchases or form submissions on his behalf.
- **Goal:** if Brian didn't name a feature, ask in one line which feature or behaviour
  he's after. A focused question gives far better evidence than "analyse everything".

## 2. Run the analysis efficiently

- Copy the artifact into the session scratchpad or a temp folder. Never modify the
  installed app.
- Big minified bundles hit REA's 300 s per-file timeout. Start with the Electron main
  process, preload and package.json (repeat `--source-path`), then add only the
  renderer files that matter. Raise `--analysis-timeout-ms` only when needed.
- For big native binaries, raise `REA_GHIDRA_STARTUP_TIMEOUT_MS` instead of retrying blindly.
- Spot-check a few cited locations against the extracted source or disassembly before
  relying on them.

## 3. Deliver a feature-harvest report

Write `<target>-<YYYY-MM-DD>.md` to the reports folder from the table, or to the current
app's `docs/` folder when working inside one of Brian's app projects. Sections:

1. **What it does:** the feature in plain words.
2. **How it works:** architecture, data flow, key APIs and formats, each claim with its
   REA evidence (file:line or address) and its stated limits or unknowns.
3. **Worth taking:** the ideas, UX patterns and formats that fit Brian's apps
   (SnapMark, Lightbox, VideoEdit, DrawIt Pro, Codepad++, 7-Zip Mac, DR Restore
   Calculator, Product Media Downloader, or the current project). Say which app and why.
4. **Build plan:** a clean-room plan in Brian's stack: Obj-C++/AppKit or Swift with
   ObservableObject (Command Line Tools only, no SwiftUI macros), or Electron for DrawIt Pro.
5. **Licence notes:** see the guardrails below.

## Guardrails

- Learn the behaviour and design, then write fresh code. Never paste decompiled or
  proprietary code into Brian's apps.
- Open-source targets may be reused under their licence. Name the licence and any
  attribution or copyleft obligations.
- Analyse only software Brian owns or is authorised to examine. Respect licence terms
  and EULAs, and flag anti-tamper or DRM areas instead of working around them.
