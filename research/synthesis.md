# Subagent Best Practices Synthesis

## 目标
把 50/50 note cards 压缩成可复用的方法论骨架，覆盖：Codex、Claude Code、通用多 agent 架构、开发工作流，以及可核实的 Reddit 社区实践信号。

## 1. 先结论

目前最稳的共识不是“多开几个 agent”，而是 4 层分离：

1. **Runtime Contract**：谁能拉起子 agent、权限如何收口、生命周期如何结束、结果如何回传。
2. **Topology Layer**：什么时候用单 agent、subagent、team、manager、critic、swarm。
3. **Control-Flow Layer**：顺序、并行、handoff、review、回滚、状态共享。
4. **Worker Profile Layer**：角色边界、工具白名单、模型路由、预算、上下文策略。

**Evidence：** 官方 Claude Code 文档把 subagents、agent teams、hooks、permissions、worktree、resume 做成正式控制面；ADK/AutoGen/CrewAI 分别覆盖原语、协作拓扑、worker 参数面；社区高质量案例把 observability、context hygiene、wave-based workflow、repo-native memory 一起抬升为一等问题；Reddit 补充样本则进一步强化了本地语义检索减 token、成本护栏、不要把 function calling 当唯一编排原语、以及“vibe coding 失败往往是因为缺验证与架构”的社区共识。  
**Source：** `notes/official/_summary-part1.md`, `notes/official/_summary-networked-8-10.md`, `notes/github/_summary-part1.md`, `notes/github/_summary-networked-8-10.md`, `notes/x/_summary-part1.md`, `notes/x/_summary-networked-8-10.md`, `notes/youtube/_summary-part1.md`, `notes/youtube/_summary-part2.md`

## 2. Codex vs Claude Code：现在真正的差异

| 维度 | Codex | Claude Code | 综合判断 |
| --- | --- | --- | --- |
| 官方证据强度 | 当前资料更偏产品入口与基础 loop | 当前资料覆盖 subagent / teams / workflows / hooks 更完整 | Claude Code 在“多 agent 可操作细节”上证据更强 |
| 基础使用心智 | 本地 terminal coding agent，先把 plan → execute → validate 跑顺 | 强调 context 管理、verification-first、delegation、worktree isolation | 两者都应先把单 agent 基础 loop 跑稳 |
| subagent / team 成熟度 | 当前资料里仍偏信号与生态方向 | 官方已把 subagents 和 agent teams 文档化 | 当前阶段 Claude Code 更像成熟参照物 |
| 工具 / 插件方向 | X 信号显示 Codex plugins 正在产品化 | Claude 侧更多是 hooks / skills / agents / plugins / external tooling 生态 | 两边都在走“prompt 之外的控制面” |
| 对我们最有用的启发 | 保持运行时、工具接口、权限边界可替换 | 优先学习其 subagent / workflow / verification 实践 | 不要做 provider-locked 架构 |

**Inference：** 如果目标是今天就写出稳的最佳实践，Claude Code 资料更适合作为主骨架；Codex 更适合作为“单 agent loop + 插件化方向 + 兼容性要求”的补充锚点。


## 2.5 Reddit 补充后，哪些信号值得纳入主结论

只保留可验证、且与官方/开源资料能互相印证的部分：

- **本地语义检索 / context compaction 值得进入标准工具层。** 这与官方反复强调的 context hygiene 一致。
- **成本与权限护栏必须前置。** 社区里的费用失控案例，说明预算与默认权限不是附属项。
- **不要把 function calling 当成多 agent 的唯一中心原语。** 更稳的是显式状态机、artifact handoff、可恢复控制流。
- **“vibe coding”失控，本质是缺 repo 结构、验证、review gate。** 这与本文的 verification-first、repo-native memory、single-writer 原则一致。

**Evidence：** 这些点在 Reddit 的 top10 样本中能找到明确表达，并能和官方文档、GitHub 实践互相印证。
**Inference：** Reddit 更适合用来发现“高频踩坑与一线经验”，不适合单独充当最佳实践总论的唯一证据层。

## 3. 通用架构：推荐的 4 层模型

### 3.1 Runtime Contract
必须先定：
- 谁可以 spawn 子 agent
- 子 agent 是否会话隔离
- 结果是 push 回传还是 pull 轮询
- 是否允许并发写同一代码区
- 完成后如何 cleanup / archive
- 哪些动作必须人工确认

**Evidence：** 官方与社区都反复表明，仅靠 prompt 不能解决权限、恢复、并发、安全与审计。  
**Source：** `notes/official/_summary-part1.md`, `notes/github/_summary-networked-8-10.md`, `notes/x/_summary-part1.md`

### 3.2 Topology Layer
推荐默认顺序：
- 单 agent
- bounded subagent delegation
- reviewer / critic 双人协作
- manager-led team
- swarm / 去中心 handoff

原则：**先单 agent，复杂度按需升级。**

### 3.3 Control-Flow Layer
推荐显式建模：
- `explore`
- `plan`
- `parallel implement`
- `review`
- `test / validate`
- `summarize / handoff`

高价值约束：
- 同一代码区单写者原则
- 并行前先切任务边界
- 返回的是 artifact，不是聊天废话
- 失败要有 timeout / fallback / forced reporting

### 3.4 Worker Profile Layer
每个 worker 至少应有：
- role
- scope
- allowed_tools
- model / reasoning budget
- max runtime / retry / rate budget
- output contract
- escalation rule

## 4. 对 repo 的最佳实践：让仓库变成 agent-native 环境

不是写更长 prompt，而是让 repo 自己可发现、可约束、可恢复：
- 顶层运行规则：`AGENTS.md` / `CLAUDE.md`
- 技能与领域知识：`skills/`
- 自动护栏：`hooks/`, scripts, CI
- 渐进式文档：ADR、runbook、module docs
- 局部上下文：模块级说明文件
- 恢复与留痕：session log / artifacts / task ids / context ids

**Evidence：** 社区高质量案例 09 明确强调 repo structure、memory、skills、hooks、local context 决定 agent 是否像工程师。  
**Source：** `notes/x/09.md`, `notes/github/_summary-networked-8-10.md`

## 5. 开发工作流 synthesis

### 5.1 最小可行闭环
`plan → execute → validate`

这是所有系统的底座。没有验证，就不要急着上多 agent。

### 5.2 进入 subagent 的时机
满足任一即可：
- 主线程上下文已被研究材料挤爆
- 任务天然可并行
- 需要独立专家视角
- 需要 review / critic 与 implement 分离
- 需要把高风险动作隔离在受限 worker 中

### 5.3 推荐的 wave-based workflow
1. 主控定义目标、边界、产物。
2. explorer / researcher 拉资料、建地图。
3. planner 输出分解与依赖。
4. implementers 按隔离边界并行执行。
5. reviewer / critic 做验证与冲突检查。
6. 主控整合、压缩、输出。

### 5.4 验证优先
- 测试、截图、脚本校验、diff review、预期输出检查要前置。
- verification-first 比“写得快”更关键。

### 5.5 可观测性不是附加项
至少应能看到：
- 哪些 agent 在运行
- 各自当前阶段
- 是否阻塞 / 超时
- 产出落在哪
- 谁有写权限

## 6. 风险与反模式

### 明确反模式
- 把 subagent 当“多开几个万能聊天框”
- 多 agent 同时写同一文件却无隔离
- 把超长 prompt 当架构
- 没有验证链路就并行开发
- 没有 timeout / fallback / review gate
- 用 `skip permissions` 之类效率捷径冒充最佳实践

### 安全基线
- 最小权限默认
- 危险操作前确认
- 读写角色分离
- hooks / policy 拦截高风险命令
- 人工 review 高风险变更

## 7. 可直接落地的推荐

### P0
- 单 agent 基础 loop
- verification-first
- repo-native rules / docs / hooks
- bounded subagents with explicit scope

### P1
- worktree / workspace isolation
- status / observability 面板
- context compaction / resume / search
- reviewer / critic 专职化

### P2
- manager-led teams
- swarm / dynamic transfer
- richer plugin / tool allocation system

## 8. 结语
- 这份 synthesis 适合作为后续教程、架构图与团队落地的总纲。
- 阅读时建议优先看 4 层架构模型，再看风险与反模式。
