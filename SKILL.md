---
name: reverse-lab
description: Reverse engineering workflow wrapper for the local open-reverselab workspace. Use when the user says reverse-lab, ReverseLab, open-reverselab, or asks to start/organize/analyze a reversing task involving PE/EXE/DLL/SYS, APK/DEX/SO/Frida, Ghidra/x64dbg/Procmon, unknown binaries, crypto/protocol reverse engineering, or authorized Web CTF targets. Creates task cases, routes to the correct board, runs safe first-pass triage, and stores evidence in the repository's notes/exports/reports structure.
---

# Reverse Lab

## Overview

Use this skill to turn a natural-language request into a disciplined ReverseLab workflow. Default to safe, static-first analysis, create or reuse a task case, route signals through the knowledge base, and keep all evidence under the open-reverselab workspace.

Resolve the ReverseLab workspace by searching the current workspace, its parents, and common user workspace folders for an `open-reverselab` directory containing `.mcp.json`, `scripts/`, `boards/`, and `kb/`. Do not publish private local absolute paths in reports, commits, or uploaded files.

## Safety Defaults

- Treat unknown executables, APKs, scripts, documents, and archives as untrusted.
- Do not execute malware, install APKs on a device, run Frida hooks, launch debuggers against live samples, scan third-party targets, brute force, exploit, or perform DoS-style actions unless the user explicitly authorizes that exact action and the scope is lawful/authorized.
- For Web CTF work, assume only lab/owned/authorized targets are allowed. Keep network activity conservative unless the user requests an active step.
- Do not commit, upload, or disclose private samples, tokens, cookies, reports, logs, screenshots, target details, or generated evidence.
- Prefer copying patched/derived artifacts to `patches/` or `exports/`; do not modify original samples in place.

## Intake Workflow

1. Resolve the ReverseLab root and work from it.
2. Verify the core environment when needed:

```powershell
python scripts/misc/first_run_check.py --write-report
uv run --project tools/skills/mcp/ReverseLabToolsMCP python scripts/misc/mcp_smoke_check.py --write-report
```

3. Classify the board:

- `windows`: PE, EXE, DLL, SYS, x64/x86, Ghidra, x64dbg, Procmon, malware triage.
- `android`: APK, DEX, Android SO, jadx, apktool, adb, Frida.
- `ctf-website`: URL, HTTP request, API, JWT, SQLi, SSRF, OAuth, CVE, Web CTF.
- `general`: unknown binaries, crypto/protocol reverse engineering, firmware/misc samples.

4. If the user did not name an existing case, create one:

```powershell
python scripts/misc/new_task.py --board <board> --name <case-name>
```

This creates:

```text
cases/<case>/ai_manifest.json
notes/<board>/<case>.md
exports/<board>/<case>/
reports/<board>/<case>/
```

5. Prefer the `reverse_lab_tools` MCP server when available. If MCP tools are not exposed in the current session, use the repository scripts and local CLI tools as fallbacks.
6. Route concrete signals through KB before deeper tool execution. Examples: packer/PE imports, Android manifest/native library clues, JWT/API/CVE/Web framework fingerprints, crypto strings, protocol names.
7. Store raw evidence under `exports/<board>/<case>/`, notes under `notes/<board>/<case>.md`, and final summaries under `reports/<board>/<case>/`.

## Board Playbooks

### Windows

Start static-first:

- Hash and identify file type/packer/compiler.
- Inspect PE headers, sections, imports, exports, resources, TLS callbacks, and strings.
- Use Ghidra headless/static analysis only if tools are installed or MCP supports it.
- Prepare x64dbg, Procmon, or Frida plans, but do not run the sample dynamically without explicit permission.
- Useful board guide: `boards/windows/AI-USAGE.md`.

Common user prompt:

```text
Use $reverse-lab to analyze samples/windows/a.exe. Create a case if needed, run static triage first, and do not execute the sample.
```

### Android

Start static-first unless the user authorizes device work:

- Inspect APK metadata, manifest, permissions, entry activities/services/receivers/providers.
- Use apktool/jadx when available.
- Identify native libraries, packers, dynamic loading, crypto/network clues.
- Do not install the APK, connect adb, start apps, or run Frida scripts unless the user explicitly authorizes it.
- Useful board guide: `boards/android/AI-USAGE.md`.

Common user prompt:

```text
Use $reverse-lab to analyze samples/android/app.apk. Start with Manifest, entry Activity, permissions, and native library triage.
```

### Web CTF

Use only for owned/lab/authorized targets:

- Create a case with `new_task.py` or `scripts/ctf-website/ctf_intake.py`.
- Start with conservative HTTP baseline and fingerprinting.
- Save requests/responses and tool outputs to `exports/ctf-website/<case>/`.
- Do not run aggressive scans, brute force, exploitation, or DoS checks unless the user explicitly authorizes the exact scope.
- Useful board guide: `boards/ctf-website/AI-USAGE.md`.

Common user prompt:

```text
Use $reverse-lab for this authorized lab URL. Start with intake, HTTP baseline, fingerprinting, and KB routing.
```

### General

Use for unknown binaries, crypto/protocol work, firmware, or mixed evidence:

- Hash, identify file type, extract strings/metadata.
- Route concrete signals through `kb/general/`.
- Keep derived scripts and extracted artifacts in `exports/general/<case>/` or `scripts/` only when reusable.

## Response Contract

When finishing a ReverseLab turn, report:

- Case name, board, and key paths.
- Commands or MCP tools used.
- High-signal findings only.
- Evidence files written.
- Recommended next actions, explicitly marking any step that needs user authorization.
