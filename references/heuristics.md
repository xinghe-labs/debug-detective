# Debugging heuristics / 调试启发式策略

Classic bug categories with the systematic hunting strategy for each. Load when the ROOT loop's Observe/Theorize phases need technique, or when you are stuck.

经典 bug 类型及其系统化定位策略。当 ROOT 循环的"观察/假设"阶段需要具体手法，或者卡住时加载。

---

## 1. Regression: "it used to work" 回归类

**Strategy**:
1. Pin the last-known-good point: which commit / release / date was it last verified working? 先锚定最后正常的时点。
2. `git bisect` between last-good and first-bad. Automate the repro as a script so each step is `run repro → good/bad`. 在好坏两点之间 `git bisect`，把复现写成脚本以便每步自动判定。
3. For dependency regressions: diff the lockfile, check the upstream changelog for behavior changes. 依赖类回归：diff 锁文件，查上游 changelog 的行为变更。
4. "Nothing changed in the code" → check what changed AROUND the code: data volume, upstream API, environment, time (see 6). "代码没变"→ 查代码周边：数据量、上游 API、环境、时间。

## 2. Concurrency & ordering 并发与顺序

**Signature**: intermittent, load-dependent, "passes alone but fails in the suite", different result every run.

**特征**：偶发、与负载相关、单独跑通过但全套件失败、每次结果不同。

**Strategy**:
1. Make it deterministic first: fixed seed, single worker, injected clock, mocked network. If you cannot reproduce it deterministically you cannot verify a fix. 先确定性化：固定随机种子、单 worker、注入时钟、mock 网络。不能确定性复现就无法验证修复。
2. Amplify the race: run the repro in a loop / under load until it fails reliably, then shrink the interleaving. 放大竞态：循环或加压跑复现直到稳定失败，再收缩交错路径。
3. Audit shared mutable state: any data touched by >1 thread/request/process without ownership. 审查共享可变状态：被多个线程/请求/进程触碰且无明确归属的数据。
4. Suspect list: check-then-act races, non-atomic read-modify-write, missing await / forgotten promise, event handler ordering, connection reuse. 嫌疑清单：check-then-act 竞态、非原子读改写、漏 await、事件顺序、连接复用。

## 3. Boundary & off-by-one 边界与差一

**Strategy**:
1. Enumerate boundaries explicitly: empty, single element, first, last, exactly-at-limit, limit+1, negative, zero, huge. 显式枚举边界：空、单元素、第一个、最后一个、恰好到限、超限一、负数、零、超大。
2. Check comparison operators (`<` vs `<=`), index bases, inclusive/exclusive ranges, `length-1`. 查比较符、下标起点、区间开闭、`length-1`。
3. For date/time: DST transitions, month boundaries, year 2038/2100-style rollovers, timezone offsets that cross midnight. 日期时间：夏令时切换、月末边界、跨午夜时区偏移。
4. String/encoding: multi-byte characters, Unicode normalization, trailing newline, BOM. 字符串/编码：多字节字符、Unicode 归一化、末尾换行、BOM。

## 4. Data & state corruption 数据与状态损坏

**Strategy**:
1. Trace the bad value backwards hop by hop to where it first became wrong — do not guess where it "probably" went wrong. 把错误值逐跳回溯到首次变错的地方，不要猜"大概"错在哪。
2. At each hop, verify: value, type, unit, encoding, timezone, null-ness. 每一跳验证：值、类型、单位、编码、时区、是否为 null。
3. Check persistence boundaries: serialization round-trips, cache staleness, migration mismatches, default values on new columns. 查持久化边界：序列化往返、缓存过期、迁移不匹配、新列默认值。
4. Read the actual stored bytes / rows / payloads, not what the ORM or client SHOWS you. 读实际存储的字节/行/报文，而不是 ORM 或客户端*显示*给你的。

## 5. Environment & configuration 环境与配置

**Signature**: works in one place, fails in another (machine, container, CI, cloud, user account).

**特征**：一处正常另一处失败（机器、容器、CI、云、用户账号）。

**Strategy**:
1. Dump and diff the environment on both sides: env vars, runtime versions, OS, locale, available binaries, permissions, network/DNS/proxy. 两侧导出并 diff 环境：环境变量、运行时版本、OS、locale、可用二进制、权限、网络/DNS/代理。
2. Check for implicit environment reads: relative paths, home-directory assumptions, `~/.config`, PATH order, proxy variables. 查隐式环境依赖：相对路径、主目录假设、`~/.config`、PATH 顺序、代理变量。
3. Verify secrets/credentials exist and are FRESH (expired tokens mimic logic bugs). 验证凭证存在且**未过期**（过期 token 的表现酷似逻辑 bug）。
4. Containerized: diff images, volumes, and entrypoint env between working and failing containers. 容器场景：diff 正常与异常容器的镜像、卷、入口环境。

## 6. Time, caching & async 时间、缓存与异步 时间、缓存与异步

**Strategy**:
1. Caching bugs: does the failing path read through a cache (HTTP cache, CDN, build cache, memoization, ORM identity map)? Force-bypass it and retest. 缓存类：失败路径是否经过缓存（HTTP/CDN/构建缓存/记忆化/ORM 一级缓存）？强制绕过再测。
2. Stale artifacts: rebuilt? reloaded? restarted? half-deployed? The fix may be correct but not what is running. 陈旧产物：真的重新构建/加载/重启/完整部署了吗？修复可能是对的，但线上跑的不是它。
3. Time dependence: clock skew, timezone, "today vs yesterday" logic, timers scheduled before dependency is ready. 时间依赖：时钟偏移、时区、"今天/昨天"逻辑、依赖未就绪就启动的定时器。
4. Async bugs: unawaited promises, missing error handlers on async paths, background jobs that swallow failures, retries that duplicate side effects. 异步类：未 await、异步路径缺错误处理、后台任务吞错、重试导致副作用重复。

## 7. Integration & protocol 边界与协议

**Strategy**:
1. Capture the actual bytes on the wire (log, proxy, tcpdump) — never debug an integration by reading your own request-building code. 抓真实线上字节（日志、代理、tcpdump）——绝不通过读自己的请求构造代码来调集成问题。
2. Diff actual vs spec: headers, content-type, charset, auth scheme, parameter casing, pagination, rate limits. 实际 vs 规范：头、content-type、charset、鉴权方式、参数大小写、分页、限流。
3. Version skew: client SDK vs server API version, staging vs prod endpoint. 版本错位：SDK 与服务端 API 版本、staging 与 prod 端点。
4. Idempotency & retries: what happens when the same request is sent twice? 幂等与重试：同一请求发两次会发生什么？

## 8. Performance "slow" 慢

**Strategy**:
1. Measure before optimizing; get a profile, not a theory. Profile the actual slow path under realistic data volume. 先测量再优化；要 profile，不要理论。用真实数据量对慢路径做 profile。
2. Check the N+1 family: query-in-loop, per-item network calls, repeated parsing of the same input. 查 N+1 家族：循环内查询、逐项网络调用、重复解析同一输入。
3. Check accidental O(n²): nested scans, repeated linear search inside a loop, string concatenation in a loop. 查意外 O(n²)：嵌套扫描、循环内线性查找、循环内字符串拼接。
4. Confirm it is CPU, I/O, lock contention, or waiting on an external service — the fix differs completely for each. 确认瓶颈是 CPU、I/O、锁竞争还是等外部服务——四者的修法完全不同。

## When stuck 卡住时的通用手法

- **Rubber-duck the code**: explain the flow line by line in the report; the contradiction usually surfaces while writing it. 把执行流程逐行讲给"橡皮鸭"（写进汇报里）：矛盾往往在写的过程中暴露。
- **Change one axis**: toggle ONE thing (flag, dependency version, input) and observe. 置换一个轴：只动一个开关/版本/输入并观察。
- **Invert the question**: instead of "why does it fail", ask "what input would make this code produce exactly this output?" — then check whether that input plausibly occurred. 反转问题：不问"为什么失败"，问"什么输入会让这段代码恰好产出这个结果"——再判断该输入是否可能发生。
- **Fresh eyes**: start a NEW session with only the evidence summary (no earlier theories) and ask for independent hypotheses. 换新会话：只带证据摘要（不带先前的理论），索取独立假设。
- **Ask the user**: history you don't have ("when did it last work for you?"), environment access, reproduction details. They often contain the answer. 问用户：你没有的历史（"你上次正常是什么时候"）、环境访问、复现细节——答案常在其中。
