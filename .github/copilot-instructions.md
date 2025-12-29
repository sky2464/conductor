# Copilot instructions (Conductor Gemini CLI extension)

## What this repo is
- This repository is a **Gemini CLI extension**; the “runtime” is primarily **prompt + template artifacts**, not application code.
- The extension entrypoint/metadata is in [gemini-extension.json](gemini-extension.json) (notably `contextFileName: "GEMINI.md"`).

## Key directories and contracts
- Command definitions live in [commands/conductor/](commands/conductor/) as TOML files with a `description` and a large `prompt = """..."""`.
- Project scaffolding templates live in [templates/](templates/), especially:
  - [templates/workflow.md](templates/workflow.md) (the workflow protocol copied into user projects)
  - [templates/code_styleguides/](templates/code_styleguides/) (language styleguide snippets copied into user projects)
- Release packaging is handled by GitHub Actions and uploads a tarball that **excludes** `.git` and `.github`: [.github/workflows/package-and-upload-assets.yml](.github/workflows/package-and-upload-assets.yml)
  - Don’t put required extension assets only under `.github/`.

## Editing command prompts safely
- Treat each command prompt as an API: keep section numbering and “CRITICAL/PROTOCOL” wording stable unless you’re intentionally changing behavior.
- Preserve Gemini templating placeholders (e.g., `{{args}}` in [commands/conductor/newTrack.toml](commands/conductor/newTrack.toml)).
- Keep file-path conventions intact because commands depend on them:
  - Tracks index: `conductor/tracks.md`
  - Track folders: `conductor/tracks/<track_id>/` with `spec.md`, `plan.md`, `metadata.json`
- Several prompts assume **specific markdown formats**:
  - Track entries are separated by `---` and use `## [ ] Track: <Description>` with a `*Link: ...*` line (see [commands/conductor/setup.toml](commands/conductor/setup.toml) and [commands/conductor/newTrack.toml](commands/conductor/newTrack.toml)).
  - Status markers are `[ ]` (todo), `[~]` (in progress), `[x]` (done) and are parsed in [commands/conductor/implement.toml](commands/conductor/implement.toml).
- Avoid introducing `"""` inside TOML `prompt = """..."""` strings.

## How Conductor’s workflow is intended to work
- `setup` creates project-level context files (`conductor/product.md`, `conductor/tech-stack.md`, `conductor/workflow.md`, etc.) and writes progress to `conductor/setup_state.json` (see [commands/conductor/setup.toml](commands/conductor/setup.toml)).
- `newTrack` generates a track’s `spec.md` then a `plan.md` that must follow the workflow rules in `conductor/workflow.md` (see [commands/conductor/newTrack.toml](commands/conductor/newTrack.toml)).
- `implement` is designed to:
  - mark the chosen track as `[~]` in `conductor/tracks.md`
  - execute tasks in `plan.md` according to the “Task Workflow” section of the workflow template (see [templates/workflow.md](templates/workflow.md))
  - update `plan.md` with commit SHAs and then mark the track `[x]` when complete (see [commands/conductor/implement.toml](commands/conductor/implement.toml)).

## When changing templates
- Assume templates are copied verbatim into user repos; prioritize clarity and low ambiguity.
- If you change headings/keywords in [templates/workflow.md](templates/workflow.md), audit prompts that reference them (notably `implement` and `newTrack`) so the contract still holds.
