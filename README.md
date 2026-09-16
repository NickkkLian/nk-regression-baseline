# nk-regression-baseline

A [Claude Code](https://code.claude.com) skill. Freeze the byte-exact output of production code on its default inputs before you change it, and compare after.

Part of [nickkk-skills](https://github.com/NickkkLian/nickkk-skills) — skills that stop an AI coding agent's
"done, tested, safe" from being taken on faith.

## What it does

- `baseline.py freeze` stores stdout, stderr, exit code and output-file hashes for a command on its default inputs; `compare` re-runs and prints a unified diff; normalizers hide only what is truly volatile.
- Manifest mode freezes/compares a set; exit codes are usable in CI.
- The rules: freeze first, default inputs, byte-exact then decide, rubric before verdict, no vanishing runs.

The full procedure, the boundaries and where the rules came from are in [SKILL.md](SKILL.md).

## Install

Copy the folder into your skills directory (the skill is the repository root):

```bash
git clone https://github.com/NickkkLian/nk-regression-baseline ~/.claude/skills/nk-regression-baseline
```

or inside one project: `git clone … .claude/skills/nk-regression-baseline`.

As a plugin, through the marketplace in the index repository:

```
/plugin marketplace add NickkkLian/nickkk-skills
/plugin install nk-regression-baseline@nickkk-skills
```

To try it for one session without installing: `claude --plugin-dir ./nk-regression-baseline`.

## Verify

```bash
python3 scripts/baseline.py --selftest
```

Standard library only, Python 3.9+. Before publishing, the guarded lines of each script were
mutated one at a time in a sandbox copy and the self-test was confirmed to go red on the named
assertion, without a traceback; the unmutated control stayed green.

## License

MIT. Read a script before letting it run in your environment.
