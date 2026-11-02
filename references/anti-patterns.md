# Agent debugging anti-patterns / agent 调试反模式

How AI coding agents typically fail at debugging, with the corrective move for each. Read this when you notice yourself doing any of these — catching one early saves an hour of thrashing.

AI 编码 agent 在调试时最典型的失败方式，以及对应的纠正动作。发现自己正在犯其中任一条时立刻读——早发现能省一小时的无效挣扎。

---

## 1. Shotgun editing 霰弹枪式修改

**Pattern**: Change five plausible spots at once, run, "it works", commit. Nobody knows which change fixed it — or what the other four broke.

同时改掉五处"看起来都可能"的地方，跑通了就提交。没人知道是哪一处修好的，另外四处又破坏了什么。

**Corrective**: Revert everything. Reproduce. One hypothesis, one change, one observation per experiment. Then re-apply ONLY the minimal fix with a regression test.

**纠正**：全部回滚。复现。每次实验只一个假设、一处改动、一个观察。确认根因后只重新应用最小修复并加回归测试。

## 2. Symptom patching 治标不治本

**Pattern**: Wrap the crash in `try/catch`, add a null check, add a retry, add a default value. The error disappears; the corruption, data loss, or hidden failure remains.

把崩溃包进 `try/catch`、加空判断、加重试、给默认值。报错消失了，但数据损坏、静默失败还在。

**Corrective**: Before adding any guard, answer: *why* was this value null / why did this call fail? If you cannot explain the cause in one sentence, you have not found the bug yet.

**纠正**：加任何防护之前先回答：这个值*为什么*是 null / 这次调用*为什么*失败？不能用一句话解释原因，就说明还没找到 bug。

## 3. Fix and pray 修完就祈祷

**Pattern**: No repro, edit code, declare "should be fixed now". The user re-tests; it still fails; another blind edit follows.

没有复现就直接改代码，宣布"应该修好了"。用户一测还是坏，又是一轮盲改。

**Corrective**: Reproduction is the entry gate to debugging. If it cannot be reproduced, the deliverable is evidence gathering and questions — not a code change.

**纠正**：复现是调试的准入门槛。无法复现时，交付物是证据收集和向用户提问，而不是改代码。

## 4. Skimming the error 跳读报错

**Pattern**: Grep the error's first line, jump to a conclusion. Missed: the SECOND exception that is the real cause, the "caused by" chain, the line number two frames up.

只搜报错第一行就下结论。错过了链上真正的根因异常、"caused by"、上面两帧的行号。

**Corrective**: Read the full message and the whole stack trace, including the first (innermost) error in a chained exception.

**纠正**：完整阅读报错和整个堆栈，包括链式异常里最内层的第一个错误。

## 5. Inventing APIs 凭空造 API

**Pattern**: Under pressure, the model calls a method that does not exist, misremembers a flag, or guesses a config key, then debugs its own hallucination as if it were the project's bug.

情急之下调用不存在的方法、记错参数、猜一个配置键，然后把自己幻觉出来的问题当成项目的 bug 来调。

**Corrective**: Treat "I think this API exists" as a hypothesis to verify by reading real code, types, or docs — before writing a single line that uses it.

**纠正**：把"我觉得有这个 API"当成待验证假设——先读真实代码/类型/文档验证，再写用到它的第一行代码。

## 6. Confirmation bias 只找支持性证据

**Pattern**: First hypothesis adopted emotionally ("it's the cache, always the cache"). Every probe afterwards is interpreted as supporting it; contradicting evidence is explained away.

第一个假设被情绪化采纳（"肯定是缓存的问题"），之后所有探查都被解读为支持它，矛盾证据被解释掉。

**Corrective**: Write the prediction BEFORE running the experiment, and treat any mismatch as falsification, not noise. Keep 2–3 hypotheses alive until one is confirmed.

**纠正**：跑实验**之前**写下预测；任何不匹配都视为证伪而非噪音。在被证实之前保持 2–3 个假设同时存活。

## 7. The swamp of dead experiments 失败实验的沼泽

**Pattern**: Debugging edits accumulate: extra logs, commented-out code, "temporary" flag flips, half-reverted attempts. The working tree no longer resembles the broken state, so nothing reproduces cleanly.

调试性修改不断堆积：多余日志、注释掉的代码、"临时"开关、改了一半又放弃的尝试。工作区已不再是当初出错的状态，什么都无法干净复现。

**Corrective**: After every experiment, `git diff` and revert anything that was not part of a confirmed fix. A clean tree is a debugging instrument.

**纠正**：每次实验后 `git diff`，回滚所有不属于已确认修复的改动。干净的工作区本身就是调试工具。

## 8. Escalating to rewrite 升级为重写

**Pattern**: After 20 minutes of confusion, propose rewriting the module "the right way". This throws away the evidence encoded in working code and introduces new bugs under pressure.

困惑 20 分钟后提出"按正确方式重写这个模块"。这会丢掉现有可运行代码中蕴含的证据，并在压力下引入新 bug。

**Corrective**: Escalation is the last resort, and it is a USER decision, not the agent's. Before proposing it, a bisect or systematic isolation must have actually been exhausted.

**纠正**：重写是最后手段，且是**用户**的决定，不是 agent 的。提出之前必须已经真正用尽二分和系统化隔离手段。

## 9. Debugging the wrong layer 调错层

**Pattern**: Frontend looks fine, backend looks fine, but nobody checked the proxy config / env var / database state / deployment diff. Time spent re-reading application code that was never the problem.

前端没问题，后端也没问题，但没人查代理配置 / 环境变量 / 数据库状态 / 部署差异。时间全部花在反复读根本无辜的应用代码上。

**Corrective**: Before deep-diving, map the request/data path end-to-end and probe each hop cheaply (log, curl, inspect). Find which hop breaks FIRST, then dive.

**纠正**：深挖之前先画清请求/数据链路，逐跳用廉价手段探测（日志、curl、状态检查）。先定位断在哪一跳，再深入那一跳。

## 10. Confusing "no error" with "correct" 把"没报错"当"正确"

**Pattern**: The command exits 0, so the pipeline "works" — but it silently wrote to the wrong table, skipped 90% of rows, or produced empty output.

命令退出码为 0，于是认为"没问题"——但它悄悄写错了表、跳过了九成的行、产出了空结果。

**Corrective**: Define the expected observable result before running (row counts, output content, side effects), and verify it, not just the exit code.

**纠正**：运行前先定义预期可观测结果（行数、输出内容、副作用），验证结果本身，而不只是退出码。
