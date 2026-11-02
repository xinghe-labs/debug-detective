# 🕵️ debug-detective 调试侦探

[English](README.md) | 简体中文

一个通用的 [Agent Skill](https://agentskills.io/specification)，把任意 AI 编码 agent 变成有纪律的调试者：**先复现再修复、先证据后猜测，永远追根因而非打补丁**。

## 为什么需要它

AI 编码 agent 最昂贵的失败模式不是写烂代码，而是**烂调试**：不做复现就盲改、用"症状补丁"把报错吞掉、霰弹枪式多处修改导致没人知道到底是哪一处修好的、信誓旦旦"应该修好了"结果一测还是坏的。

**debug-detective** 把资深工程师的工作方法强加到调试循环上：

- **R**eproduce 复现 —— 没有最小、可稳定复现的用例之前，不许动手修
- **O**bserve 观察 —— 完整读报错、梳理时间线（`git log/diff`）、只要事实不要理论
- **T**heorize & Test 假设与验证 —— 排序后的可证伪假设，**一次实验只改一处**，证伪即回滚
- **Fix** 修复 —— 只修根因，用原始复现验证，加回归测试锁住

外加针对 agent 特有失败模式的硬性规则：不许编造 API、不许霰弹枪式修改、不许留下失败实验的沼泽、设置明确的"停下求助"门槛。

## 实际差别 What changes in practice

### 无 skill（Without the skill）

> agent 读一眼报错、搜一个关键词，加个 `try/catch` 和空判断，顺手再改两行"以防万一"，然后宣布"应该修好了"。没有复现、没有测试，真正的根因——一个过期缓存——还原封不动。

English: Agent reads the error, greps one keyword, adds a `try/catch` and a null check, changes two unrelated lines "just in case", and reports "should be fixed now". Nothing is reproduced, nothing is tested, and the real cause — a stale cache — is still there.

### 有 skill（With the skill）

> agent 先写最小复现、读完整堆栈、列出排序后的假设，用一次改动测最便宜的假设（绕过缓存），证伪即回滚；找到真正根因（缺缓存键）后只修这一处，并加上无修复即失败的回归测试。

English: Agent writes a minimal repro first, reads the full stack trace, forms ranked hypotheses, tests the cheapest one (bypass the cache) with a single change, reverts it after falsification, finds the real cause (a missing cache key), fixes only that, and adds a regression test that fails without the fix.

## 安装

无需构建、零依赖——本质就是一个遵循标准 [Agent Skills 规范](https://agentskills.io/specification) 的 Markdown 文件夹。把它克隆进你所用运行时的技能目录：

```bash
# 用户级（所有项目可用）
git clone https://github.com/xinghe-labs/debug-detective ~/.agents/skills/debug-detective

# 或项目级（仅当前项目，在仓库根目录执行）
mkdir -p .agents/skills && git clone https://github.com/xinghe-labs/debug-detective .agents/skills/debug-detective
```

`~/.agents/skills/` 与 `.agents/skills/` 是 Agent Skills 规范的标准位置；遵循规范的运行时（Pi、Claude Code 等）会自动从这里发现该 skill。安装后重启或重载技能即可。

## 验证

重启 agent（Pi 中为 `/reload`）后，询问它当前有哪些可用技能——列表中应有 `debug-detective`。Pi 中也可以强制加载：

```text
/skill:debug-detective 复现这个 bug：<粘贴任意报错>
```

若加载成功，回复会遵循 ROOT 循环（复现 → 观察 → 假设 → 修复），而不是直接动手改。

## 常见问题 / 故障排除

- **报了 bug 却没触发？** 技能路由由模型决定；可显式加载——Pi 中用 `/skill:debug-detective <bug 描述>`，或在请求中点名 skill。
- **改了 skill 却没生效？** 技能在启动时扫描——重载或重启 agent（Pi 中为 `/reload`）。
- **怎么升级？** 若是 git 克隆安装：`git -C ~/.agents/skills/debug-detective pull`；否则重新克隆后重载。
- **怎么卸载？** 删除文件夹（`rm -rf ~/.agents/skills/debug-detective`）后重载。
- **启动时提示名称冲突？** 发现顺序更靠前处存在同名 `debug-detective` 技能——删除或重命名其一。

## 文件结构

```text
debug-detective/
├── SKILL.md                      # ROOT 循环、硬性规则、症状速查表
└── references/
    ├── anti-patterns.md          # 10 种 agent 调试失败模式 + 纠正动作
    └── heuristics.md             # 8 类经典 bug + 系统化定位策略
```

## 何时生效

skill 的路由描述覆盖：用户报 bug、测试意外失败、报错与异常、行为回归、"代码明明没问题却不工作"。它刻意在**提出任何修复之前**触发——这正是设计意图。

也可以显式调用：`/skill:debug-detective <bug 描述>`。

## 语言

天生双语：指令为英文 + 简体中文对照，同一份 skill 同时适用于国际社区和中文团队。

## 许可证

MIT —— 见 [LICENSE](LICENSE)。
