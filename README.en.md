<p align="right">
  <a href="README.md"><img src="https://img.shields.io/badge/简体中文-3a3f4b?style=for-the-badge" alt="简体中文"></a>
  <a href="README.en.md"><img src="https://img.shields.io/badge/English-d97757?style=for-the-badge" alt="English"></a>
</p>

# Claude Code Skills

A set of skills invokable by Claude Code, plus a **bilingual skill panel** [`index.html`](index.html): browse skill intros in the browser and generate install/clear commands in one click.

- Every skill and reading-version doc comes in **both languages** (zh / en).
- Toggle the language at the top-right of the panel to choose which version you view and install.
- Each skill can be installed **globally or into a project**, with a clean-overwrite command.

> See the skills and what each is for at the end: [📚 Skills](#-skills).

---

## 📁 Directory layout

```
skills/
├── index.html                  # Skill panel: top-right zh/en toggle, skill cards, import/clear (generates commands)
├── README.md                   # Chinese readme
├── README.en.md                # This file (English)
├── vibecoding/
│   ├── SKILL.md                # Skill body · Chinese (with frontmatter, invoked as /vibecoding)
│   └── SKILL.en.md             # Skill body · English (also name: vibecoding)
└── docs/
    ├── vibecoding流程.md        # Reading-version flow · Chinese (same content as SKILL.md)
    └── vibecoding-flow.en.md    # Reading-version flow · English
```

- **`vibecoding/SKILL.md` / `SKILL.en.md`** — the files Claude Code actually loads and executes. Both use `name: vibecoding`, so the invocation name is `/vibecoding` either way. **Install only one at a time** (see below).
- **`docs/*`** — non-executable, pure docs for reading the flow quickly.

---

## 🖥️ Install via the panel (recommended)

Open [`index.html`](index.html) in a browser:

1. Toggle **中文 / EN** at the top right — this decides which language version gets installed, and the language of all prompts.
2. Click a skill card to read its intro.
3. At the bottom of the intro dialog, click **Import global config** / **Import to a project** / **Clear config**.
4. Enter the target directory in the dialog (empty = default), click "Generate command", then **copy it into a terminal to run**.

> The panel is a static page; the browser can't touch your filesystem directly, so it **generates a command** for you to run in a terminal. The command is a **clean overwrite** (removes the old directory first, then copies) and keeps only the `SKILL.md` for the current UI language.

---

## 🚀 Manual install

Claude Code discovers skills from two kinds of locations:

| Scope | Path | Use when |
|---|---|---|
| **Global** (all projects) | `~/.claude/skills/<name>/SKILL.md` | Day-to-day, recommended |
| **Project** (that project only) | `<project-root>/.claude/skills/<name>/SKILL.md` | Ship with the repo, share with the team |

> The unit of installation is the **whole skill directory** (`vibecoding/`), not a single file.
> Replace `<repo-path>` in the commands below with where you cloned this repo. **Shortcut**: use the [`index.html`](index.html) panel — it auto-detects your local path and generates a ready-to-run command.

### macOS / Linux

**Chinese version** (the default `SKILL.md` is already Chinese; just drop the extra English file):

```bash
# Global install (clean overwrite)
mkdir -p ~/.claude/skills && \
  rm -rf ~/.claude/skills/vibecoding && \
  cp -R <repo-path>/vibecoding ~/.claude/skills/ && \
  rm -f ~/.claude/skills/vibecoding/SKILL.en.md
```

**English version** (overwrite `SKILL.md` with `SKILL.en.md`):

```bash
mkdir -p ~/.claude/skills && \
  rm -rf ~/.claude/skills/vibecoding && \
  cp -R <repo-path>/vibecoding ~/.claude/skills/ && \
  mv -f ~/.claude/skills/vibecoding/SKILL.en.md ~/.claude/skills/vibecoding/SKILL.md
```

> Project install: replace `~/.claude/skills` above with `<project-root>/.claude/skills` (`mkdir -p` it first).

### Windows · PowerShell

```powershell
# Global install (Chinese version)
$dst = "$env:USERPROFILE\.claude\skills\vibecoding"
Remove-Item -Recurse -Force $dst -ErrorAction SilentlyContinue
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\skills" | Out-Null
Copy-Item -Recurse -Force "C:\path\to\skills\vibecoding" "$env:USERPROFILE\.claude\skills\"
Remove-Item -Force "$dst\SKILL.en.md" -ErrorAction SilentlyContinue
# English version: replace the last line above with
# Move-Item -Force "$dst\SKILL.en.md" "$dst\SKILL.md"
```

> To stay linked to the source instead of copying, use a symlink: macOS/Linux `ln -s <src>/vibecoding ~/.claude/skills/vibecoding`;
> Windows (admin) `mklink /D "%USERPROFILE%\.claude\skills\vibecoding" "C:\path\to\skills\vibecoding"` (with a symlink both language files are present; Claude only reads `SKILL.md`).

After installing, **restart / open a new Claude Code session** so the skill list reloads.

### Clear / uninstall

```bash
rm -rf ~/.claude/skills/vibecoding              # global
rm -rf <project-root>/.claude/skills/vibecoding # project
```

### Verify

```bash
ls ~/.claude/skills/vibecoding/SKILL.md     # macOS/Linux
dir "%USERPROFILE%\.claude\skills\vibecoding\SKILL.md"   # Windows CMD
```

Type `/` in a session and you should see `vibecoding` in the skill list.

---

## 📚 Skills

### 🚀 Vibecoding Full-Stack Workflow — `/vibecoding`

Human + AI pairing, advancing in the shortest verifiable steps to turn an idea into **runnable, deployable** full-stack code. It proceeds through "requirements clarification → tech-stack selection → architecture → scaffolding → vertical-slice incremental implementation → self-testing → review → deployment"; on invocation you choose the tech stack, with a built-in combination sanity check and two tracks (Solo / Team).

```
/vibecoding I want to build a to-do app
```

<p>
  <a href="docs/vibecoding-flow.en.md"><img src="https://img.shields.io/badge/Full_docs-English-d97757?style=for-the-badge" alt="Full docs English"></a>
  <a href="docs/vibecoding流程.md"><img src="https://img.shields.io/badge/详细介绍-简体中文-3a3f4b?style=for-the-badge" alt="详细介绍 简体中文"></a>
</p>

---

## 🔧 Customize
- Change skill behavior: edit `vibecoding/SKILL.md` (Chinese) or `SKILL.en.md` (English), then re-sync to the installed location with the commands above (or use a symlink to skip syncing).
- Change the invocation name: edit the `name` field in the `SKILL.md` frontmatter and rename the directory to match.
- Add a new skill: create `<name>/SKILL.md` in the repo and add an entry to the `SKILLS` array in [`index.html`](index.html) (with bilingual `name`/`tagline`/`intro`) to make it show up in the panel.
