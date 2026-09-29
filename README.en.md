# github-push-check

[中文](README.md) | [English](README.en.md)

A Codex skill for GitHub push checks and versioned releases. Current version: **1.0.0**.

## Problem it solves

When asking Codex to push a local project to GitHub, it is easy to miss documentation, include sensitive or temporary files, or forget release metadata that matches the code. This skill defines a consistent workflow: back up the project, prepare an independent clean version, then verify, push, and release it.

The repository contains skill instructions and interface configuration for Codex. It is not a standalone scanner or a Git hook that intercepts every `git push`.

## Main features

- Back up the original project and verify the backup; make changes in an independent clean copy while preserving the original working directory.
- Generate or repair Chinese and English READMEs covering purpose, features, installation, usage, input/output examples, and consistency between languages.
- Review and address six categories: passwords/tokens, personal information, local absolute paths, temporary files, disposable test artifacts, and other material unsuitable for sharing.
- Review working files, staged content, and the history to be pushed so that deleting a current file does not conceal a historical leak.
- Validate the clean version and push to the authorized destination.
- Verify the current project version; check or supply its tag, Release title, concise notes, and current features, then push the tag and publish the GitHub Release.
- Reuse correct existing tags and Releases on repeated runs; report version conflicts without forcing replacements.

## Installation

You need Codex with local skill support, Git, and read access to this private repository. Pushing and publishing also require an authorized GitHub connector or an authenticated GitHub CLI. Build tools depend on the project being reviewed. The skill itself has no additional runtime dependencies or API key configuration.

Run these PowerShell commands in a download directory. They refuse to overwrite an existing installation; back up an installed skill before updating it.

```powershell
git clone --branch v1.0.0 --depth 1 https://github.com/ZhiyaoXiongMath/github-push-check.git
if ($LASTEXITCODE -ne 0) { throw 'Clone failed; check repository access.' }

$skillBase = if ($env:CODEX_HOME) {
    Join-Path $env:CODEX_HOME 'skills'
} else {
    Join-Path $HOME '.codex/skills'
}
$skillDestination = Join-Path $skillBase 'github-push-check'
if (Test-Path -LiteralPath $skillDestination) {
    throw 'Skill already exists. Back it up before updating.'
}
New-Item -ItemType Directory -Path $skillDestination -Force | Out-Null
Copy-Item -LiteralPath './github-push-check/SKILL.md', './github-push-check/agents', './github-push-check/references', './github-push-check/VERSION' -Destination $skillDestination -Recurse
```

On other systems, copy the same four files/directories into `github-push-check` under your personal skill directory: `~/.codex/skills/github-push-check/` by default, or `skills/github-push-check/` under `CODEX_HOME` when configured. The downloaded Git history is not needed. Invoke the skill in your next conversation turn after installation.

## Usage

Open the project to publish in Codex, invoke `$github-push-check`, and identify the destination repository. You can also ask Codex to push the project to GitHub and let it select the skill based on its description.

- Full workflow: specify the destination repository and the current project version.
- Review only: explicitly say “Review only; do not modify, push, or publish.”
- Push code only: explicitly say “Do not publish a Release this time.”

Your instructions for the current request take precedence. The skill does not invent an unknown version, upload local backups, or change repository visibility without authorization. See [SKILL.md](SKILL.md) for the full workflow and [release guidance](references/release.md) for tag and Release handling.

## Input/output example

This is an illustrative example. Actual paths, versions, findings, and release results depend on the project review.

**Input**

```text
Use $github-push-check on the current project. Back it up, complete the Chinese and English READMEs, and address the six risk categories in a clean version. Push to my specified private repository. The current version is 1.2.3; publish the matching Release if it is ready.
```

**Expected output**

```text
Checks passed.
- Backup: created and verified.
- Clean version: prepared; the original project is unchanged.
- README: both languages are complete and all six checks pass.
- Six-category review: findings addressed; no unresolved release blockers.
- Validation: list the checks actually performed and their results.
- Push: report the actual destination branch and commit.
- Release: report the v1.2.3 tag, Release title, and actual link.
```

If versions conflict, validation fails, or sensitive content remains, the result must identify the outstanding work rather than present partial completion as a successful release.

## Repository contents and scope

- `SKILL.md`: workflow and review criteria.
- `agents/openai.yaml`: display name, default prompt, and automatic invocation settings.
- `references/release.md`: version verification and tag/Release creation or reuse rules.
- `VERSION`: the current published version.

Automatic invocation depends on Codex skill selection; use `$github-push-check` when you want to request it explicitly. Scanning and semantic review cannot guarantee detection of all sensitive information, and unverified areas must be reported. The skill does not intercept manual pushes from a terminal or another tool, or independently publish packages or deploy services.
