---
name: scout
description: Session mode for ultra-lean context — activate with "启用 scout"/"scout on"; afterwards the parent context stays a minimal distilled log and all heavy work (reading, exploring, searching, building) goes to scout subagents that return only structured reports. Deactivate with "停用 scout"/"scout off".
---

# Scout 模式（fork → work → distill → return）

会话级上下文极简模式：主上下文只存蒸馏日志，原始材料全在 scout 沙箱里烧。

## 启用 / 停用

- **启用**：用户说"启用 scout"/"scout on"时 invoke 本 skill。**模式激活后对整个会话生效**，不逐案选择。
- **停用**："停用 scout"/"scout off"后恢复常规单线程。

## 激活期间的行为

1. **蒸馏日志落盘**：在工作目录维护 `.scout-log.md`（或简报指定路径），
   内容 = 任务目标 + 决策 + 已返回 doc。每次收到 scout 的 doc 就追加。
   主上下文里只留它的引用——**即使本会话被 compact，日志在盘上不丢**。
2. **父侧只允许轻量工具**：短命令、小文件（≲50 行）、状态检查。
   批量读取/搜索/探索/实现/长输出命令——**默认全派 scout，不请示**。
3. **派发**：`run_subagent`（profile: `scout`），prompt 三块：
   日志文件全文快照 + 子任务简报（含前序 pitfall 引用）+ 落盘要求
   （中间产物写文件、最后只回 doc）。
4. **接收**：doc 追加进 `.scout-log.md`；细节靠 `rehydrate` 指针
   读文件或 resume 同一 scout 追问——不把原始材料拉回主上下文。

## 效果边界

- 主上下文峰值与总工作量**解耦**：scout 烧多少 token 主上下文都不涨。
- 代价：总 token 更高（每个 scout 重读日志快照；prefix cache 可抵一部分），
  且蒸馏有损——schema 里的坑字段和证据指针是保真关键。
- **不适用**：子任务与主线需持续共享中间状态时，停用本模式，
  单线程 + compact 更合适（Cognition "Don't Build Multi-Agents"）。

## 实现注记（Devin）

- 子代理无状态："注入"= 把 `.scout-log.md` 内容粘进 prompt，非 API fork。
- `scout` profile（`~/.config/devin/agents/scout.md`）钉 `swe-2-max`
  并内置 doc schema；`agent.subagent_model` 已配 `swe-2-max`。
- `is_background: true` 可并行派多个 scout；需逐步审批就前台。
- 协议公开规范与安装：github.com/koko1256/scout
