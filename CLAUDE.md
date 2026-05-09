---
name: fork-and-install
version: "1.0.0"
description: >
  Forks any GitHub skill or plugin repository to ssai-group and installs it as a
  Claude Code plugin. Use whenever the user gives you a GitHub repo URL and asks
  to install it as a skill or plugin. Also use proactively when the user says
  "install this skill", "add this plugin", or pastes a GitHub URL for a skill.
---

# Fork and Install

You are the fork-and-install skill. When the user gives you a GitHub repository URL to install as a skill or plugin, follow these steps exactly.

## Steps

### 1. Parse the repo
Extract `owner/repo` from the URL. For example:
- `https://github.com/obra/superpowers` → `obra/superpowers`
- `https://github.com/AgriciDaniel/claude-obsidian` → `AgriciDaniel/claude-obsidian`

### 2. Fork to ssai-group
Run:
```powershell
& "$env:TEMP\gh_portable\bin\gh.exe" repo fork owner/repo --clone=false
```
If `gh` is not found at that path, check if it is on PATH with `gh --version`. If neither works, tell the user to run `gh auth login` first using the portable binary at `C:\Users\admin\AppData\Local\Temp\gh_portable\bin\gh.exe`.

### 3. Add as marketplace
```powershell
claude plugin marketplace add ssai-group/repo-name
```
Note the marketplace name printed on success (e.g. `repo-name-marketplace` or `repo-name-dev`).

### 4. Install the plugin
```powershell
claude plugin install plugin-name@marketplace-name
```
The plugin name comes from the repo's `.claude-plugin/plugin.json` `name` field. If the repo has no `.claude-plugin` directory, tell the user: "This repo is not structured as a Claude Code plugin. It may be a raw skill file. Ask me to set it up as a plugin first."

### 5. Check for a PreToolUse hook
After install, check if the plugin has a `hooks/` directory containing a `.py` or `.sh` hook file. If it does:
- Copy the hook file to `C:\Users\admin\.claude\skills\<hook-filename>`
- Read `C:\Users\admin\.claude\settings.json`
- Add a PreToolUse entry pointing to the copied file using `python "C:/Users/admin/.claude/skills/<hook-filename>"`
- Write the updated settings.json back

### 6. Report
Tell the user:
- Fork URL: `https://github.com/ssai-group/repo-name`
- Plugin installed: `plugin-name@marketplace-name`
- Hook registered: yes/no
- To reinstall on any new device: the exact two commands needed

## Important rules
- Always fork to `ssai-group` — never install directly from a third-party repo
- Never skip the fork step even if the user says "just install it directly"
- If the fork already exists, `gh repo fork` will report it — that is fine, continue
- Always use the user's own fork as the plugin source