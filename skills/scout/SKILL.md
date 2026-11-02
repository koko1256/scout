---
name: scout
description: Delegate heavy reading/exploration/implementation to a scout subagent so raw material stays out of the main context. Use for long-horizon or multi-part tasks — the scout inherits a distilled context snapshot, works in isolation, and returns only a structured report (outcome, evidence pointers, pitfalls, rehydrate path).
---

# Scout 协议（fork → work → distill → return）

主上下文只存蒸馏日志；原始材料在 scout 沙箱里烧，只有结构化 doc 过界。
侦察兵出去摸地形，带回来的只有地图和情报——不是把整座山搬回来。

## 主上下文不变量（parent invariant）

- 主上下文只保留：任务目标、已做决策、scout 返回的 doc。
- 不把原始大段工具输出留在主上下文——批量读取、探索、重实现一律派 scout。
- 蒸馏日志必须保持小而干净：它会被整体注入每个 scout 的 prompt。

## 派发（dispatch = fork）

用 `run_subagent`（profile: `scout`），task/prompt 里放三块：

1. **蒸馏日志快照**：当前主上下文全文（目标 + 决策 + 已有 doc），
   原样粘贴。日志有界所以注入廉价。
2. **子任务简报**：做什么、要解决什么、约束、完成判据；引用前序
   相关 `pitfall` 做坑的传递。
3. **过程落盘要求**：中间笔记/证据写到文件并说明路径；最后只回
   结构化 doc（schema 已内置在 scout profile 系统提示，可按需补充
   本次重点字段）。

## 接收（return）

- 只把返回的 doc 追加进蒸馏日志；scout 的 transcript 不进主上下文。
- 需要更多细节：按 doc 的 `rehydrate` 指针读文件，或用
  `run_subagent` 的 resume 参数向同一 scout 追问。
- 错误/不完整：resume 追问，不要自己重做把原始材料拉回上下文。

## 何时不用

子任务与主线强耦合、需要持续共享中间状态时——单线程 + compact 更合适
（参见 Cognition "Don't Build Multi-Agents" 的论证）。

## 实现注记（Devin）

- Devin 子代理无状态："注入"= 把蒸馏日志粘进 prompt，非 API 级 fork。
- `scout` profile（`~/.config/devin/agents/scout.md`）钉 `swe-2-max`
  并内置 doc schema；默认子模型已在 config 设 `swe-2-max`。
- `is_background: true` 适合并行探索；需要逐步审批就前台跑。
- Claude Code：同样的 SKILL.md 放 `.claude/skills/scout/`，worker 定义
  放 `.claude/agents/scout.md`（约定兼容）。
