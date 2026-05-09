---
name: fork-and-install
version: "1.1.0"
description: >
  Forks any GitHub skill or plugin repository to ssai-group and installs it as a
  Claude Code plugin. Runs a mandatory Sentinel security scan before every install.
  Use whenever the user gives you a GitHub repo URL and asks to install it as a
  skill or plugin.
---

# Fork and Install

Follow these steps exactly when the user gives you a GitHub URL to install. Do not skip any step.

## Step 1: Parse the repo
Extract owner/repo from the URL.

## Step 2: Pre-install security scan (MANDATORY)

Before forking or installing, run a full Sentinel security scan:

1. Read the repo key files via GitHub contents API (CLAUDE.md, SKILL.md, hooks/, scripts/)

2. Search external threat sources in parallel:
   - site:github.com/advisories "[repo-name]" vulnerability
   - site:vulnerablemcp.info [repo-name]
   - "[repo-name]" CVE 2025 OR 2026
   - site:reddit.com/r/ClaudeAI [repo-name] security OR malicious

3. Static analysis — flag and stop on any of these:
   - Shell commands accessing credential files or private key directories
   - Network POST requests to non-standard domains
   - Remote content piped directly into an interpreter
   - eval() or exec() on external data
   - Scripts that modify shell profiles

4. Coherence check — does everything match the repo stated purpose?

5. Present verdict before proceeding:
   - OK: Clean, proceed
   - WARN: Concerns found, ask user to confirm
   - STOP: Critical threat found, do not install

If verdict is STOP, halt. Do not fork or install.

## Step 3: Fork to ssai-group
Run: & "$env:TEMP\gh_portable\bin\gh.exe" repo fork owner/repo --clone=false

## Step 4: Add as marketplace
Run: claude plugin marketplace add ssai-group/repo-name

Note the marketplace name printed on success.

## Step 5: Install the plugin
Run: claude plugin install plugin-name@marketplace-name

The plugin name comes from .claude-plugin/plugin.json name field.
If no .claude-plugin directory exists, tell the user and offer to set it up.

## Step 6: Check for a PreToolUse hook
If the plugin has a hooks/ directory with a .py hook file:
- Copy it to C:\Users\admin\.claude\skills\<hook-filename>
- Add a PreToolUse entry to C:\Users\admin\.claude\settings.json

## Step 7: Report
- Security scan summary
- Fork URL
- Plugin installed name
- Hook registered yes/no
- Reinstall commands for new devices

## Rules
- Always fork to ssai-group, never install from third-party repos directly
- Never skip the security scan
- Always show scan verdict before forking