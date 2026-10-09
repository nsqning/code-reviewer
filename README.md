# Code Reviewer

`code-reviewer` is a Codex skill for reviewing a pull request or branch as one
full diff. It re-checks earlier findings, analyzes resource and server-load
costs, prioritizes crash and service-wide availability risks, and produces an
evidence-backed Markdown report.

## What it reviews

- Correctness, authorization, data loss, concurrency, durability, and migration safety.
- CPU, memory, disk, database/WAL, network, thread, scheduler, and external-service costs.
- Crash/startup-loop and service-wide outage paths as the highest-priority blockers.
- Conditional occurrence ratios and production trigger estimates.
- Previously reported findings as `FIXED`, `PARTIALLY_FIXED`, `NOT_FIXED`, or `REGRESSED`.

## Install in Codex

### Option 1: Clone directly into the Codex skills directory

This is the recommended installation method because it also makes future
updates easy.

```bash
mkdir -p "$HOME/.codex/skills"
git clone https://github.com/nsqning/code-reviewer.git \
  "$HOME/.codex/skills/code-reviewer"
```

The installed structure should be:

```text
~/.codex/skills/code-reviewer/
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    └── report-format.md
```

Restart Codex or start a new Codex conversation so the skill is rediscovered.

### Option 2: Install from a downloaded ZIP

1. On GitHub, select **Code → Download ZIP**.
2. Extract the archive.
3. Copy the extracted folder to `~/.codex/skills/code-reviewer`.
4. Confirm that `SKILL.md` is directly inside that folder, rather than inside
   an additional nested directory.
5. Restart Codex or start a new conversation.

On Windows, the equivalent destination is:

```text
%USERPROFILE%\.codex\skills\code-reviewer
```

## Verify the installation

On macOS or Linux:

```bash
test -f "$HOME/.codex/skills/code-reviewer/SKILL.md" \
  && echo "code-reviewer is installed"
```

You can also verify it from Codex by starting a new conversation and invoking:

```text
$code-reviewer
```

## Usage

Invoke the skill and provide a pull request URL, branch, or local checkout:

```text
Use $code-reviewer to review this pull request as one full diff:
https://github.com/OWNER/REPOSITORY/pull/123
```

For a repeated review after fixes land:

```text
Use $code-reviewer to pull the latest PR commits, re-check every finding from
the previous Markdown report, analyze resource costs and crashability, and
write a new review-latest-<sha>.md report.
```

The skill does not modify product code, push branches, or post pull-request
comments unless the user separately requests those actions.

## Update

If installed using Git:

```bash
git -C "$HOME/.codex/skills/code-reviewer" pull --ff-only
```

Restart Codex or start a new conversation after updating.

## Troubleshooting

- **The skill does not appear:** Verify that
  `~/.codex/skills/code-reviewer/SKILL.md` exists, then restart Codex or open a
  new conversation.
- **The clone destination already exists:** Update the existing Git checkout
  with the command above instead of cloning again.
- **The ZIP created a nested directory:** Move the contents so that
  `SKILL.md` is directly under the `code-reviewer` directory.
- **The skill does not activate automatically:** Invoke it explicitly with
  `$code-reviewer`.

## Repository contents

- `SKILL.md` — workflow, review lenses, crashability priorities, and evidence rules.
- `references/report-format.md` — required Markdown report structure and fields.
- `agents/openai.yaml` — Codex display metadata and default invocation prompt.

Before installing a skill from any repository, review its instructions and
supporting files. OpenAI documents skills as directories containing a required
`SKILL.md` plus optional references, scripts, and assets:
[Build skills](https://developers.openai.com/plugins/build/skills).
