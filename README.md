# 🕵️ debug-detective

[English](README.md) | 简体中文

A universal [Agent Skill](https://agentskills.io/specification) that turns any AI coding agent into a disciplined debugger — one that reproduces before it fixes, gathers evidence before it guesses, and always hunts the root cause instead of patching symptoms.

## Why

The most expensive failure mode of AI coding agents isn't writing bad code — it's **debugging badly**: blind edits with no reproduction, symptom patches that silence errors, shotgun changes where nobody knows what actually fixed the bug, and "should be fixed now" claims that fail on retest.

**debug-detective** imposes the working method of a senior engineer on the debugging loop:

- **R**eproduce — no fix before a minimal, reliable repro
- **O**bserve — full error messages, timelines (`git log/diff`), facts over theories
- **T**heorize & Test — ranked falsifiable hypotheses, **one change per experiment**, revert on falsification
- **Fix** — root cause only, verified against the original repro, locked in with a regression test

Plus hard rules that target agent-specific failure modes: no invented APIs, no shotgun edits, no swamp of dead experiments, explicit stop-and-ask gates.

## What changes in practice 实际差别

### Without the skill（无 skill）

> Agent reads the error, greps one keyword, adds a `try/catch` and a null check, changes two unrelated lines "just in case", and reports "should be fixed now". Nothing is reproduced, nothing is tested, and the real cause — a stale cache — is still there.

简体中文：agent 读一眼报错、搜一个关键词，加个 `try/catch` 和空判断，顺手再改两行"以防万一"，然后宣布"应该修好了"。没有复现、没有测试，真正的根因——一个过期缓存——还原封不动。

### With the skill（有 skill）

> Agent writes a minimal repro first, reads the full stack trace, forms ranked hypotheses, tests the cheapest one (bypass the cache) with a single change, reverts it after falsification, finds the real cause (a missing cache key), fixes only that, and adds a regression test that fails without the fix.

简体中文：agent 先写最小复现、读完整堆栈、列出排序后的假设，用一次改动测最便宜的假设（绕过缓存），证伪即回滚；找到真正根因（缺缓存键）后只修这一处，并加上无修复即失败的回归测试。

## Install

No build step and no dependencies — it is a folder of Markdown. Clone or download this repository, then copy the `debug-detective/` folder into your runtime's skills directory.

### Pi

```bash
# user-level (available in all projects) — substitute /path/to with where you cloned/downloaded this repo
cp -r /path/to/debug-detective ~/.pi/agent/skills/

# or project-level (this project only, from the repo root)
mkdir -p .agents/skills && cp -r /path/to/debug-detective .agents/skills/
```

Project-level skills require trusting the project in Pi (approve prompt or `pi --approve`). Then restart Pi or run `/reload`.

### Claude Code

```bash
cp -r /path/to/debug-detective ~/.claude/skills/
```

### Any other Agent Skills runtime

Copy the `debug-detective/` directory into the runtime's skills location (`~/.agents/skills/` or `.agents/skills/`, per the [specification](https://agentskills.io/specification)). The skill uses no runtime-specific features.

## Verify

After restarting Pi (or `/reload`), run a one-shot check from any directory:

```bash
pi --no-session -p "List the exact names of all skills advertised to you."
```

`debug-detective` should appear in the list. Or force-load it interactively:

```text
/skill:debug-detective Reproduce this bug: <paste any error>
```

If the skill loads, the reply will follow the ROOT loop (reproduce → observe → theorize → fix) instead of jumping to a fix.

## FAQ / Troubleshooting

- **It doesn't trigger when I report a bug.** Skill routing is model-driven; always load it explicitly with `/skill:debug-detective <bug description>` (Pi) or by naming it in your request.
- **I edited the skill but nothing changed.** Skills are scanned at startup — run `/reload` or restart Pi.
- **How do I update it?** If you cloned it: `git -C ~/.pi/agent/skills/debug-detective pull`. Otherwise re-copy the folder and `/reload`.
- **How do I uninstall it?** Delete the folder (`rm -rf ~/.pi/agent/skills/debug-detective`) and `/reload`.
- **Startup warning about a name collision?** Another skill named `debug-detective` exists earlier in discovery order — remove or rename one of them.

## Contents

```text
debug-detective/
├── SKILL.md                      # the ROOT loop, hard rules, symptom cheatsheet
├── CHANGELOG.md
└── references/
    ├── anti-patterns.md          # 10 agent debugging failure modes + corrective moves
    └── heuristics.md             # 8 classic bug categories + hunting strategies
```

## When it activates

The skill's routing description covers: reported bugs, unexpected test failures, errors and exceptions, regressions, and "it should work but doesn't". It deliberately fires **before** any fix is proposed — that's the point.

You can also invoke it explicitly: `/skill:debug-detective <bug description>`.

## Language

Bilingual by design: instructions are English + 简体中文 side by side, so the same skill works for international and Chinese-language teams.

## License

MIT — see [LICENSE](LICENSE).
