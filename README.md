# Memory Checkup

**Audit and repair an AI agent's memory corpus for consistency.**

[English](#english) | [中文](#中文)

---

<a id="english"></a>

Agents that keep long-term memory (OpenClaw, Claude Code with memory files, or any agent with a `MEMORY.md` + `memory/` structure) accumulate drift: a metric changes on Monday, three files still say the old value on Friday; a daily flush job silently dies; a path referenced in memory no longer exists. Nobody notices until the agent confidently states a stale fact.

Memory Checkup scans the whole corpus and surfaces that drift before it reaches the user.

## What it detects

| Check | Meaning |
|---|---|
| **Stale numbers** | Same metric (downloads / GitHub stars / user counts) appearing with an old value in some files while newer files carry a newer value |
| **Dangling references** | File paths mentioned in memory that no longer exist on disk |
| **Stale entries** | Gaps in date-named daily logs near the newest entry (a flush job may have died) |
| **Orphan files** | Non-dated memory files never referenced from anywhere else |

## Quick start

```bash
git clone https://github.com/Thomaszhou22/memory-checkup.git
python3 memory-checkup/scripts/memory_checkup.py --memory-dir /path/to/agent/workspace
```

Add `--json` for machine-readable output, e.g. to pipe into a cron or a gateway hook.

The script is **read-only**. Fixing is deliberately left to the agent and the user: for every stale number, confirm the current value against live evidence first, then update or annotate the old mentions as superseded.

## Using as an OpenClaw / Claude Code skill

Copy or symlink this folder into your skills directory. The `SKILL.md` frontmatter triggers on requests like "check my memory", "memory audit", "体检记忆", and on risky recall situations (metrics recorded over many days).

Agent workflow defined by the skill:

1. Locate the corpus (workspace with `MEMORY.md` / `memory/`)
2. Run the audit script
3. Verify each finding with the user or live evidence — newest is not always correct
4. Fix file by file, re-run until clean
5. Report in the user's language

## Design notes

- **Read-only by default.** An audit tool that edits memory unsupervised would be the thing it audits.
- **Newest-wins with verification.** The script ranks values by the newest supporting file (ties broken toward the larger value), but treats every "stale" flag as a question, not a verdict.
- **Scope discipline.** Scans only the workspace root Markdown files and `memory/`; never recurses into projects or `node_modules/`.
- **Context filtering.** Number patterns are disambiguated by line context (e.g. stars must mention GitHub) to cut false positives.
- Extend `DEFAULT_PATTERNS` in the script to track any metric you care about (MAU, API latency, revenue...).

## Real-world validation

Developed and validated against a live agent workspace with 130 memory files: it found a superseded download count still lingering in five files, template placeholder paths recorded as real references, and a dead daily-flush gap — all confirmed real.

## File structure

```
├── SKILL.md                    # Skill instructions for the agent
├── scripts/
│   └── memory_checkup.py       # The read-only audit script
└── README.md
```

## Limitations

- Number detection is heuristic (regex + context); it flags candidates, not truths
- Currently ships English + Chinese number patterns
- Metric ranking assumes date-named daily logs; custom corpus layouts may need tuning

## License

MIT

---

<a id="中文"></a>

# Memory Checkup（记忆体检）

**审计并修复 AI agent 记忆库的一致性。**

带长期记忆的 agent（OpenClaw、Claude Code，或任何用 `MEMORY.md` + `memory/` 结构的 agent）都会积累漂移：周一改了个数字，周五还有三个文件写着旧值；每日归档任务悄悄死了；记忆里引用的路径已经不存在。没人发现，直到 agent 一脸自信地说出过期事实。

Memory Checkup 扫描整个记忆库，在漂移到达用户之前把它找出来。

## 检测什么

| 检查项 | 含义 |
|---|---|
| **过期数字** | 同一指标（下载量 / GitHub 星标 / 用户数）在部分文件中还是旧值，更新的文件里已是新值 |
| **悬空引用** | 记忆中提到的文件路径在磁盘上已不存在 |
| **停更条目** | 最新条目附近按日期命名的日志出现缺口（归档任务可能挂了） |
| **孤儿文件** | 非日期命名的记忆文件，从未被任何其他文件引用 |

## 快速开始

```bash
git clone https://github.com/Thomaszhou22/memory-checkup.git
python3 memory-checkup/scripts/memory_checkup.py --memory-dir /path/to/agent/workspace
```

加 `--json` 得到机器可读输出，可接入 cron 或网关钩子。

脚本**只读**。修复刻意留给 agent 和用户：每个过期数字先对实时证据确认当前值，再更新旧表述或标注为已作废。

## 作为 OpenClaw / Claude Code skill 使用

把本文件夹复制或软链到你的 skills 目录。`SKILL.md` 的 frontmatter 在"检查记忆""记忆审计""体检记忆"等请求，以及高风险回忆场景（跨多天记录的指标）时触发。

## 设计要点

- **默认只读。** 会无监督改记忆的审计工具，本身就是它该审计的对象。
- **新值优先但需核验。** 按最新支持文件排序（同日取大值），但每个"过期"标记是问题，不是判决。
- **扫描范围克制。** 只扫工作区根 Markdown 和 `memory/`，绝不进项目目录和 `node_modules/`。
- **上下文过滤。** 数字模式用行上下文消歧（如星标必须提到 GitHub），压低误报。
- 需要追踪其他指标（MAU、API 延迟、收入……），扩展脚本里的 `DEFAULT_PATTERNS` 即可。

## 实测验证

在一个 130 个记忆文件的活跃 agent 工作区上开发并验证：发现了已被作废的下载量仍残留在五个文件、模板占位路径被当成真实引用记录、每日归档缺口——全部属实。

## 局限

- 数字检测是启发式的（正则 + 上下文），标记的是候选，不是真相
- 目前内置英文 + 中文数字模式
- 指标排序假设按日期命名的日志；自定义记忆结构可能需要调整

## 许可

MIT
