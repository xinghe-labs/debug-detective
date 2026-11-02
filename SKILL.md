---
name: debug-detective
description: >
  Systematic root-cause debugging discipline. Use whenever the user reports a
  bug, a test fails unexpectedly, an error or exception appears, behavior
  regressed, or code "should work but doesn't" — before proposing or applying
  any fix. Also covers non-code misbehavior: broken data pipelines, wrong
  config or environment, failing CI/CD, integration and caching issues.
  Covers reproduction, evidence gathering, hypothesis ranking,
  minimal experiments, root-cause fixes, and regression guards. Enforces
  agent-safe debugging: no shotgun edits, no symptom patching, no fix-and-pray.
  适用场景：用户报 bug、测试意外失败、出现报错或异常、行为回归、
  "代码明明没问题却不工作"，以及非代码问题：数据管道出错、配置或环境不对、
  CI/CD 失败、集成与缓存异常 —— 在提出或应用任何修复之前使用。
  关键词：调试、排查、修 bug、报错、根因分析、回归、CI、配置、数据管道。
license: MIT
metadata:
  author: debug-detective contributors
  version: 1.1.0
---

# Debug Detective 调试侦探

You are now in debugging mode. Your single goal is to find and fix the ROOT CAUSE — not to make the symptom disappear. 你现在处于调试模式。唯一目标是找到并修复**根因**，而不是让症状消失。

**Scope 适用域**: any observable misbehavior in a system — application code bugs, failing tests, broken data pipelines, wrong configuration or environment, CI/CD failures, integration and caching issues. Language- and framework-agnostic. 适用域：系统中任何可观测的异常行为——代码 bug、测试失败、数据管道出错、配置或环境不对、CI/CD 失败、集成与缓存问题。不限语言与框架。

Core principle: **evidence before edits**. Never change code to "see if it helps". Every change is an experiment with a stated hypothesis and a predicted outcome. 核心原则：**先证据，后修改**。绝不为"看看有没有用"而改代码；每一次改动都是一个有假设、有预测的实验。

## The ROOT loop (mandatory) 必须遵循的循环

Follow these phases in order. Do not skip to fixing. 按顺序执行，不许跳步直接修。

### R — Reproduce 复现
1. Reproduce the failure reliably first (test, command, curl, script, exact user steps). 先用测试/命令/脚本/确切步骤稳定复现。
2. Then shrink it to the MINIMAL repro: smallest input, least code, fewest moving parts. A minimal repro often reveals the bug by itself. 再收缩到**最小复现**：最小输入、最少代码。最小化过程本身常常就暴露了 bug。
3. If you cannot reproduce after honest attempts, say so explicitly and gather more evidence (logs, environment, version). Do NOT blind-fix unreproducible bugs. 若确实无法复现，明确说明，转为收集证据（日志、环境、版本）；**禁止盲修无法复现的 bug**。

### O — Observe 观察
1. Read the FULL error message, stack trace, and surrounding logs. Do not skim. 完整阅读报错、堆栈和日志，不要扫一眼。
2. Establish the timeline: what changed recently? (`git log`, `git diff`, new dependency, config change, data change.) 梳理时间线：最近改了什么？
3. Collect facts, not theories: actual values, actual types, actual state at the failure point. Add targeted logging or a debugger/REPL if needed. 收集**事实**而非理论：失败点的真实值、真实类型、真实状态。

### T — Theorize & Test 假设与验证
1. Form 2–4 concrete, falsifiable hypotheses. For each, write: "If H is true, then X should be observable." 列出 2–4 个可证伪假设，每个写明"若 H 成立，应能观察到 X"。
2. Rank by (likelihood × cheapness-to-test). Test the cheapest high-likelihood one first. 按（可能性 × 验证成本）排序，先测最便宜且可能性高的。
3. **One change per experiment.** Run it, compare against the prediction, then either confirm (keep going deeper) or REVERT and move to the next hypothesis. **一次实验只改一处**。跑完对比预测：证实则深入，证伪则**回滚**再测下一个。
4. Cheap tools first: logging > REPL probe > binary search (`git bisect`) > isolated test. 先用便宜手段：日志 > REPL 探针 > 二分定位 > 隔离测试。

### Fix — root cause only 只修根因
1. Fix the cause, not the symptom. A `try/catch`, null-check, or retry that silences the error without explaining WHY it happened is not a fix — flag it if it's the only option. 修因不修症：吞掉异常/加空判断/重试而不解释"为什么会这样"不是修复。
2. Verify the fix against the ORIGINAL minimal repro — it must now pass. 用**原始最小复现**验证修复必须通过。
3. Add a regression test that fails without the fix and passes with it. 添加回归测试：无修复时失败、有修复时通过。
4. Check blast radius: search the codebase for the same pattern and run the broader test suite. 检查影响面：搜索同类 pattern，跑更大范围的测试。

## Hard rules (agent-safe debugging) 硬性规则

- **No shotgun edits.** Never change multiple things at once to "cover all bases". 禁止霰弹枪式修改：不许同时改多处"全覆盖"。
- **Revert failed experiments.** Failed experiments must be reverted before the next one, or the working tree becomes a swamp that hides the bug. 失败的实验必须回滚，否则工作区变成掩盖 bug 的沼泽。
- **Never invent APIs, function names, or file paths.** Read the actual code/docs. 严禁凭空编造 API、函数名、文件路径；去读真实代码和文档。
- **Read before editing.** Every file you plan to change, read the relevant code first. 改哪先读哪。
- **Timebox.** If 3 hypotheses are falsified or ~15 tool calls pass without new information, STOP: summarize evidence, re-plan, and ask the user for what you lack (credentials, environment, history). 给自己设限：连续证伪 3 个假设或约 15 次工具调用无新信息时，停下汇报证据、重新规划，并向用户索要缺失的东西（凭证、环境、历史）。
- **Protect the user.** No destructive commands, no edits to generated files/lockfiles, no production probing without explicit consent. 不跑破坏性命令、不碰生成物/锁文件、未经明确同意不探测生产环境。

## Quick symptom cheatsheet 症状速查表

| Symptom 症状 | First checks 首选排查 |
|---|---|
| Works locally, fails elsewhere 本地正常别处挂 | Environment diff: env vars, versions, paths, data, locale, timezone 对比环境：变量、版本、路径、数据、时区 |
| "It used to work" 以前是好的 | `git bisect` the regression window; check dependency upgrades 二分回归区间；查依赖升级 |
| Intermittent / flaky 偶发 | Race condition, ordering, caching, network, time-dependence; run repro in a loop 竞态、顺序、缓存、网络、时间依赖；循环跑复现 |
| Wrong data / wrong value 数据不对 | Trace the value end-to-end from source; check boundary cases and type coercion 沿数据链路从头追踪；查边界与类型转换 |
| Crash with stack trace 崩溃带堆栈 | Read the FIRST error in the chain, then each frame in your own code 从链上第一个错误读起，逐帧看自己的代码 |
| Silent wrong behavior 静默出错 | Add observability first — you cannot debug what you cannot see 先加观测点——看不见就无法调试 |

## Deeper references 深入阅读

Load these only when needed: 仅在需要时加载：

- `references/anti-patterns.md` — agent debugging failure modes with examples（agent 调试失败模式与案例）
- `references/heuristics.md` — classic bug categories and systematic hunting strategies（经典 bug 类型与系统化定位策略）

## Reporting format 汇报格式

When done (or when blocked), report: RCA in one sentence, the fix and why it addresses the root cause, evidence (repro + test), and any residual risk. 完成或受阻时汇报：一句话根因、修复方案及为何触及根因、证据（复现+测试）、残留风险。Honest "I don't know yet" beats a confident wrong fix. 如实说"还不确定"优于自信地给出错误修复。
