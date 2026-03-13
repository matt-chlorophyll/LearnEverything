# LearnEverything

AI驱动的渐进式交互学习系统，作为 Claude Code skill 运行。

## 项目概述

基于 Bloom 2-Sigma 理论，通过一对一 AI 辅导实现任何领域的高效学习。核心原则：**"不提问，让AI提问"**。

### 解决的两个核心问题

1. **发现 unknown unknowns** — 通过渐进式提问测验，帮用户发现不知道自己不知道的东西
2. **交互式学习** — AI 生成渐进式文档 + 提问验证，替代传统读书

## 项目结构

```
LearnEverything/
├── CLAUDE.md                    # 本文件
├── plan/                        # 设计方案文档
├── domains/                     # 学习领域数据（运行时生成）
│   └── {domain-name}/
│       ├── meta.json            # 领域元信息
│       ├── knowledge-graph.json # 知识图谱
│       ├── learner-profile.json # 学习者画像
│       ├── sessions/            # session 摘要
│       ├── docs/                # AI 生成的学习文档
│       ├── discussions/         # 验证对话记录
│       └── quotes/              # 金句记录（{YYYY-MM-DD}.md）
└── unknown-unknowns/            # 跨领域盲点发现记录
```

对应的 skill 文件位于 `.claude/skills/learn-everything/`。

## 技术约定

- **语言**：用户交互使用中文，文件名和 JSON key 使用英文
- **文档格式**：Markdown（学习文档、session 摘要）
- **数据格式**：JSON（知识图谱、元信息）
- **Skill 触发**：`/learn` 或 `/学习`
- **文件命名**：docs 用 `{序号}-{概念名}.md`，sessions 用 `{YYYY-MM-DD-HHmm}.md`

## 学习 Session 流程

1. **初始化/恢复** — 新领域创建目录，继续学习则加载知识图谱+最近摘要
2. **诊断** — 冷启动用 5-8 个渐进问题探测起点；继续学习用 2-3 个问题确认记忆
3. **文档生成** — 单一概念聚焦，500-1000 字
4. **AI 提问验证** — 混合判定：概念类要求解释+举例，技能类要求解题；含深度追问（追问链、反例挑战、边界探测、元认知提问）
5. **练习** — 根据内容类型出场景题/对比题/实践题
6. **总结** — 更新知识图谱、学习者画像，生成 session 摘要，记录 unknown unknowns

## Unknown Unknown 发现机制

- 学习过程中自然暴露（AI 提问时发现关联知识盲点）
- 阶段性知识边界扫描（每 5-8 个概念后主动扫描相邻领域）
- 跨领域关联发现（多领域并行学习时建立连接）

## 开发注意事项

- 详细设计方案见 `plan/v1-system-design.md`（初版）和 `plan/v2-deep-questioning-and-personalization.md`（深度提问+个性化）
- Skill 使用 progressive loading 模式：主 SKILL.md + references/ 子文件
- 知识图谱节点状态：mastered / learning / weak / unknown / undiscovered
- 支持多领域并行学习，各领域独立追踪进度
