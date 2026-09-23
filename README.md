# system-design-interview

A skill for practicing system design interviews. Four modes:

- **`mock`** — Claude plays a strict staff-level interviewer. Phases, time-boxing, deep-dive, and a scored debrief at the end.
- **`learn`** — Reverse mock. *You* interview Claude. Watch what good answers look like. With `--auto`, Claude orchestrates two sub-agents (interviewer + candidate) and you observe a full session.
- **`postmortem`** — Diagnose a real interview you took. Pass `--file` with notes, or answer structured questions.
- **`generate`** — Author a fresh question + rubric. Writes four files (`question.md`, `assumptions.md`, `description.md`, `rubric.md`) to `./system-design-questions/<slug>/`.

Works in **Claude Code** and **Codex CLI**.

## Demo

`learn --auto` on *design X.com's timeline feed* (staff level). Two sub-agents — one interviewer, one candidate — run a full 5-round interview while you watch, and the session closes with a written retrospective grading both sides.

https://github.com/user-attachments/assets/24673305-6dbf-4b39-a04a-6262a61d961c

A shorter clip showing first-run setup (`examples/init/demo-init.mp4`) is also checked into the repo.

## Install

Pick the path that matches your harness. The repository root is the skill directory; `SKILL.md` is its entry point.

### Claude Code — raw skill copy

```bash
git clone https://github.com/jtherrell/system-design-interview.git /tmp/system-design-interview
mkdir -p ~/.claude/skills/system-design-interview
cp -R /tmp/system-design-interview/{SKILL.md,reference,scripts} ~/.claude/skills/system-design-interview/
```

Verify: `ls ~/.claude/skills/system-design-interview/SKILL.md`. Then `/system-design-interview` is available.

### Codex CLI

User-wide:

```bash
git clone https://github.com/jtherrell/system-design-interview.git /tmp/system-design-interview
mkdir -p ~/.agents/skills/system-design-interview
cp -R /tmp/system-design-interview/{SKILL.md,reference,scripts} ~/.agents/skills/system-design-interview/
```

Project-local: copy `SKILL.md`, `reference/`, and `scripts/` into your project's `.agents/skills/system-design-interview/`.

After install, open `/skills` in Codex (or invoke `$system-design-interview ...`). Restart Codex if the skill doesn't appear immediately.

## Usage

Replace the prefix with whatever your harness uses (`/` for Claude Code, `$` for Codex).

```
system-design-interview mock                              # generate a question, run a strict mock
system-design-interview mock "design a URL shortener"
system-design-interview mock --level=senior

system-design-interview postmortem                        # structured Q&A about a past interview
system-design-interview postmortem --file=./notes.md

system-design-interview generate                          # generate question + rubric (4 files)
system-design-interview generate "rate limiter"
system-design-interview generate "chat system" --level=staff
system-design-interview generate --direction=ml-infra     # bias topic to ML infra subdomain
system-design-interview generate --direction=llm          # LLM inference / RAG / agents

system-design-interview learn                             # you interview Claude
system-design-interview learn "design Cursor's autocomplete backend"
system-design-interview learn --auto                      # two sub-agents, you watch
system-design-interview learn --auto --exchanges=15       # cheaper run (default 30 exchanges)
```

## State (persisted across sessions)

The skill writes to `~/.system-design/state/`:

| File | Purpose |
|---|---|
| `runs.md` | One row per scored session: date, slug, mode, level, direction, the five dimension scores, and a one-sentence action item for the next session (may be blank). The primary tracker — read at the start of each `mock`/`postmortem` to surface a preamble (total sessions, recurring weak dimensions, last 3 slugs, plus the previous session's action item if set). Slug column also serves as the "already practiced" list for `mock` and `generate`. |
| `weaknesses.md` | One row per weak dimension (score ≤3) from past `mock` debriefs and `postmortem` diagnoses, with a one-line context quote. `mock` biases its deep-dive picks toward recurring weak dimensions. |
| `level.md` | Your target level (default `staff` if absent). |

State is per-user, harness-neutral, and not in this repo. It's created on first use. Same state directory whether you installed in Claude Code or Codex — practice carries over.

## What gets generated where

- `generate` writes question packages to **the current working directory**, under `./system-design-questions/<slug>/`. Run it from wherever you want the files to live (a notes repo, a dotfiles directory, etc.).
- `mock` and `postmortem` update state files in `~/.system-design/state/`.
- `learn` and `learn --auto` don't write any files unless you ask.

## Scoring dimensions

Both `mock` debriefs and `postmortem` diagnoses score across:

1. Requirements scoping
2. High-level structure
3. Deep-dive depth
4. Tradeoff reasoning
5. Communication

Each gets a 1–5 with one-line justification grounded in something you actually said.

## Cost note on `learn --auto`

Each exchange in `--auto` mode = 2 sub-agent calls. Default 30 exchanges = ~60 calls. The skill will tell you the count before starting and let you override with `--exchanges=N`. Use a smaller number to sanity-check the format before committing to a full run.

## Repo layout

```text
system-design-interview/
├── README.md
├── SKILL.md                              # skill entry point
├── reference/                            # mode instructions and reference material
├── scripts/speak.sh                      # optional voice helper
└── recommendations/                      # practice guidance
```

Edit the skill files directly. For local development, symlink your installed skill directory to the repository root so edits take effect without copying. There are no generated copies or sync step.
