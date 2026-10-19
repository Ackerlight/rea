# REA on the Windows PC

Brian's guide for setting up REA, Ghidra and the `use-rea` skill on a Windows 10/11 x64 PC,
so "Use REA and analyze …" works in Claude Code there as it does on the Mac.

**How to use it:** do step 0 yourself, then open Claude Code in
`%USERPROFILE%\Developer\REA` and say:

> Follow personal\PC-INSTALL.md step by step. Ask me before each install.

## What the PC can and can't do

- ✅ Windows `.exe`/`.dll` (x86, x64, ARM64), Electron apps, .NET assemblies, websites.
- ❌ Mac apps: Windows Ghidra mode accepts Windows PE files only. Run those on the Mac.
- ❌ Watching a running program: REA's runtime capture is macOS/Linux only.
- REA calls its Windows Ghidra mode **experimental** (analysis only, no editing).

## 0. Get this repo (you do this)

Claude Code on Windows already needs Git for Windows. In PowerShell:

```powershell
git clone https://github.com/Ackerlight/rea.git "$env:USERPROFILE\Developer\REA"
```

## 1. Prerequisites (winget)

```powershell
winget install --id OpenJS.NodeJS.LTS -e
winget install --id EclipseAdoptium.Temurin.21.JDK -e
```

Then open a **new** terminal and check:

```powershell
node -v      # must be 22.19+, 24.11+, or 26+
java -version  # must be 21 or newer, 64-bit
```

## 2. Ghidra 12.1.4

REA is tested with exactly 12.1.4. It's free and open source, from NSA's GitHub releases.
The Windows decompiler is already included, so there's no build step (unlike the Mac).

```powershell
$zip = "$env:TEMP\ghidra_12.1.4_PUBLIC_20260921.zip"
Invoke-WebRequest "https://github.com/NationalSecurityAgency/ghidra/releases/download/Ghidra_12.1.4_build/ghidra_12.1.4_PUBLIC_20260921.zip" -OutFile $zip
(Get-FileHash $zip -Algorithm SHA256).Hash   # must equal DDAC49F903DA9D5BAC833E5CC79395098B9C33CFD3279BE5F31BD00387D2D4DB
New-Item -ItemType Directory -Force C:\Tools | Out-Null
Expand-Archive $zip -DestinationPath C:\Tools
Test-Path C:\Tools\ghidra_12.1.4_PUBLIC\Ghidra\Features\Decompiler\os\win_x86_64\decompile.exe  # True
```

Stop if the hash doesn't match.

## 3. Environment variables (current user, persistent)

Replace the JDK folder with the one winget installed (`dir "C:\Program Files\Eclipse Adoptium"`):

```powershell
[Environment]::SetEnvironmentVariable("GHIDRA_INSTALL_DIR", "C:\Tools\ghidra_12.1.4_PUBLIC", "User")
[Environment]::SetEnvironmentVariable("JAVA_HOME", "C:\Program Files\Eclipse Adoptium\jdk-21.<version>-hotspot", "User")
[Environment]::SetEnvironmentVariable("REA_ANALYSIS_PROVIDER", "ghidra", "User")
```

Open a new terminal so they take effect. Ghidra and the JDK must sit on a local NTFS drive
(C: is fine), not a network or cloud-synced folder.

## 4. Install REA

Use the **npm package** on Windows, not a build of this repo: only the package includes
REA's Windows native component.

```powershell
npm install --global rea-agents
rea --version
rea doctor --json
```

`doctor` should report Ghidra, the JDK and Windows native controls as ready. Fix anything
it names before going on.

## 5. Register REA with Claude Code and add the skills

REA's own setup registers the MCP server for all projects and installs its
`reverse-engineer-anything` skill. Preview it, then apply:

```powershell
rea setup --client claude_code --dry-run
rea setup --client claude_code --yes
```

Then add Brian's skill:

```powershell
$dst = "$env:USERPROFILE\.claude\skills\use-rea"
New-Item -ItemType Directory -Force $dst | Out-Null
Copy-Item "$env:USERPROFILE\Developer\REA\personal\skills\use-rea\SKILL.md" $dst -Force
```

## 6. Check it works

Restart Claude Code, open a **new** session and run:

```powershell
claude mcp list   # rea: ... ✔ Connected
```

Then try:

> Use REA and analyze C:\Windows\System32\notepad.exe — how does it handle file encodings?

A report should appear in `%USERPROFILE%\Developer\REA-Notes\`.

## Keeping it up to date

```powershell
rea update
rea setup --client claude_code --yes
cd "$env:USERPROFILE\Developer\REA"; git pull
Copy-Item personal\skills\use-rea\SKILL.md "$env:USERPROFILE\.claude\skills\use-rea\" -Force
```

The Mac is the source of truth for `use-rea`. Edit it there and push, then pull here.

## Undo

```powershell
rea uninstall
Remove-Item -Recurse "$env:USERPROFILE\.claude\skills\use-rea"
npm uninstall --global rea-agents
```

Then delete `C:\Tools\ghidra_12.1.4_PUBLIC` and, if nothing else uses them, the JDK and
Node.js (`winget uninstall`).
