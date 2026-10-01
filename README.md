# nk-regression-baseline

An agent skill for [Claude Code](https://code.claude.com) and [OpenAI Codex](https://developers.openai.com/codex). Freeze the byte-exact output of production code on its default inputs before you change it, and compare after.

**What you get.** One real run of nk-regression-baseline 0.1.2, copied from the terminal on 2026-09-30:

```text
$ python3 scripts/baseline.py freeze --dir demo/baselines --name report -- cat demo/report.txt
frozen report: exit=0 stdout=16B stderr=0B files=0
$ python3 scripts/baseline.py compare --dir demo/baselines --name report
✘ report: differs
stdout differs (5 diff lines):
  --- baseline/stdout
  +++ now/stdout
  @@ -1 +1 @@
  -rows 3 total 42
  +rows 2 total 24
```

![nk-regression-baseline](https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/social/nk-regression-baseline.png)

Part of [nickkk-skills](https://github.com/NickkkLian/nickkk-skills) — skills that stop an AI coding agent's
"done, tested, safe" from being taken on faith.

## Try it

Nothing is installed and nothing under `~/.claude` changes: clone, run the self-test, run the example (it only writes inside the clone).

```bash
git clone https://github.com/NickkkLian/nk-regression-baseline && cd nk-regression-baseline
python3 scripts/baseline.py --selftest
mkdir -p demo && printf 'rows 3 total 42\n' > demo/report.txt
python3 scripts/baseline.py freeze --dir demo/baselines --name report -- cat demo/report.txt
printf 'rows 2 total 24\n' > demo/report.txt
python3 scripts/baseline.py compare --dir demo/baselines --name report
```

The self-test prints:

```text
baseline selftest · 11/11 passed
```

The last command prints the block at the top of this page; its last line is the one below, and its exit code is 1 (non-zero on purpose: it found something).

```text
  +rows 2 total 24
```

![nk-regression-baseline demo: before and after](https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/nk-regression-baseline.gif)

## What it does

- `baseline.py freeze` stores stdout, stderr, exit code and output-file hashes for a command on its default inputs; `compare` re-runs and prints a unified diff; normalizers hide only what is truly volatile.
- Manifest mode freezes/compares a set; exit codes are usable in CI.
- The rules: freeze first, default inputs, byte-exact then decide, rubric before verdict, no vanishing runs.

The full procedure, the boundaries and where the rules came from are in [SKILL.md](SKILL.md).

## How it works

1. Freeze before touching anything. Pick the commands existing users run, with their default inputs.
2. Check the frozen run is alive.
3. Normalize only what is truly volatile (timestamps, temp paths, run ids) with `--normalize REGEX`.
4. Change the code. Add the feature, refactor, clean up.
5. Compare. `baseline.py compare --dir .baselines --name totals` — exit 0 means byte-identical stdout, stderr, exit code and output-file hashes after normalization; exit 1 prints a unified diff and the changed files.
6. Freeze the new state once the change is accepted, so the next change has a baseline too.

## Why it is built this way

**The idea.** New-feature tests prove the new code runs; they say nothing about the old behaviour. No new-feature test ever contains the input "feature absent".

**Where it came from.** Own practice, 2026-08 to 2026-09: a media pipeline regression caught only by the byte-level baseline; two experiment rounds whose baseline arm was dead (a capped input meant the mechanism under study never started), so eight comparisons had to be thrown away.

**Evidence.** What was broken on purpose to show that the self-tests can fail is under [Verify](#verify); what was run end to end, and in which agent, is under [Compatibility](#compatibility).

## Install

Pick one of four ways: three for Claude Code, one for OpenAI Codex. Skills load when a session starts, so open a **new** session after installing.

### 1 · Terminal, one command

```bash
git clone https://github.com/NickkkLian/nk-regression-baseline ~/.claude/skills/nk-regression-baseline
```

1. Run the command above (for one project only, clone into `.claude/skills/nk-regression-baseline` inside that project).
2. Start a new Claude Code session.
3. Check it loaded: type `/nk-regression-baseline` — it appears in the slash-command menu. Or just ask for the task; the skill triggers on its own.

### 2 · Claude Code in a terminal session (plugin)

The plugin route goes through the [nickkk-skills](https://github.com/NickkkLian/nickkk-skills) marketplace. Add it once; after that each skill is one command.

```
/plugin marketplace add NickkkLian/nickkk-skills
/plugin install nk-regression-baseline@nickkk-skills
```

1. In a Claude Code session, run the first line (once per machine).
2. Run the second line.
3. Start a new session (or run `/reload-plugins`). The skill shows up as `nk-regression-baseline:nk-regression-baseline`.

Without opening a session, the same two steps work from a shell: `claude plugin marketplace add NickkkLian/nickkk-skills` then `claude plugin install nk-regression-baseline@nickkk-skills`.

### 3 · Claude desktop app (Code tab)

**Add the marketplace first — Discover only searches marketplaces you have already added.**

<img src="https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/panel-route/panel-route.gif" alt="Adding the marketplace and installing a skill in the desktop app" width="640">

<sub>Recorded on 2026-09-16, when the marketplace listed ten skills, all at version 0.1.0; it lists more now. The repository list in this recording shows the recorder's own repositories because a GitHub account is connected; yours will show yours. Type the full name as in step 4.</sub>

1. In the chat box, type `/plugin marketplace` and press Enter (or open **Settings → Customize → Plugins**). The **Plugins** panel opens.
   <br><img src="https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/panel-route/step1-type-plugin-marketplace.png" alt="/plugin marketplace typed in the chat box" width="480">
2. Top right, open **Add ▾** and choose **Add marketplace**.
   <br><img src="https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/panel-route/step2-add-menu.png" alt="The Add menu with Add marketplace" width="480">
3. Choose **Add from a repository**.
   <br><img src="https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/panel-route/step3-add-from-repository.png" alt="Add marketplace dialog: Add from a repository" width="480">
4. In **URL**, type the full `NickkkLian/nickkk-skills`. At the bottom of the list choose the row **Use "NickkkLian/nickkk-skills"**, then press **Sync**.
   <br><img src="https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/panel-route/step4-url-then-sync.png" alt="URL filled in, Sync button" width="480">
5. You land on **Discover**, filtered to the new marketplace (**Filter · 1**). Find **Nk regression baseline** and press **Add**. Installed ones show **✓ Added**.
   <br><img src="https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/panel-route/step5-discover-add.png" alt="Discover list with Added and Add buttons" width="480">
6. Close the panel and start a new session.

To try it for one session without installing anything: `claude --plugin-dir ./nk-regression-baseline` from a clone.

### 4 · OpenAI Codex CLI

```bash
git clone https://github.com/NickkkLian/nk-regression-baseline.git ~/.agents/skills/nk-regression-baseline
```

1. Run the command above (for one project only, clone into `.agents/skills/nk-regression-baseline` inside that project).
2. Start a new Codex session.
3. Check it loaded, without spending a model call: `codex debug prompt-input | grep -o -- '- nk-regression-baseline[a-z0-9:-]*' | sort -u` prints `- nk-regression-baseline:nk-regression-baseline:`. Codex adds the `nk-regression-baseline:` prefix because this repository also carries a Claude Code plugin manifest. Ask for the task and the skill triggers on its own, or type `$` and pick it from the list.

## Compatibility

| Agent | Tested | What was checked |
|---|---|---|
| Claude Code (CLI 2.1.173, macOS) | yes | In a fresh project with an isolated Claude config, inside a macOS sandbox that blocked reading the tester's ~/.claude folder (settings, session history, memory), Desktop, Documents and Downloads, SSH keys and git identity, a plain request that never names the skill triggered it and it ran its bundled script. The route 2 plugin commands were also run from a shell with an isolated config: marketplace add, install, list. |
| OpenAI Codex CLI (0.154.0-alpha.6.2, gpt-5.6-sol, low reasoning, macOS) | yes | Copied into `~/.agents/skills` of a temporary home (the folder route 4 clones into), in a fresh project, without the user's Codex config. From a plain request that never names the skill, Codex read SKILL.md, froze the default run of `report.py` with `scripts/baseline.py` and confirmed that a control compare was byte-identical before any change. |
| Cursor, Gemini CLI | no | Not tested. Their documentation says both read `~/.agents/skills`, the folder route 4 clones into; Gemini CLI asks before it activates a skill. |

In this skill's Codex run, every call into the skill folder's scripts/ used that folder's absolute path. Route 4 was checked for this repository: cloned from GitHub into a temporary home's `~/.agents/skills`, it was listed by the step 3 command. This skill's frontmatter uses only name, description, license and metadata.

## Verify

```bash
python3 scripts/baseline.py --selftest
```

Standard library only, Python 3.9+. On 2026-09-30 every self-test above passed, and
`breakcheck.py` from [nk-breakable-selftest](https://github.com/NickkkLian/nk-breakable-selftest) broke each script on purpose in a sandbox copy:

- `baseline.py`: 6 lines broken one at a time; 3 turned the self-test red without a traceback. Not covered: the self-test stayed green with L37, L39 switched off; switching off L65 crashed the script instead of failing a sample, which does not count as caught.

The unmutated control stayed green every time. Only lines that record a finding, raise, or return a failing exit code
were broken (the tool's pattern, or the hand-written list); a line number refers to the script as shipped in this version.
This shows those lines are covered. It does not show that nothing else can fail.

## Limits

- It compares outputs; it does not know which differences matter. That is the review.
- Non-deterministic programs need normalizers or seeds; if the control comparison is not identical before you change anything, fix that first — you have no baseline yet.

## License

MIT. Read a script before letting it run in your environment.
