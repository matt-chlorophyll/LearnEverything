# LearnEverything 开发进度记录

## 当前状态：Step 3 完成，准备进入 Step 4

---

## Step 1: 项目基础结构 ✅

### 2026-03-04

**完成内容：**
- 初始化 git repo
- 创建 GitHub remote: https://github.com/matt-chlorophyll/LearnEverything
- 创建 `CLAUDE.md` — 项目级指令文件
- 创建 `plan/v1-system-design.md` — 完整系统设计方案
- Initial commit & push to main

**文件清单：**
```
LearnEverything/
├── CLAUDE.md
└── plan/
    ├── v1-system-design.md
    └── PROGRESS.md          ← 本文件
```

**运行时目录：** ✅ 已创建
- `domains/` （含 .gitkeep）
- `unknown-unknowns/` （含 .gitkeep）

---

## Step 2: 创建 Skill 主文件 ✅

### 2026-03-04

**完成内容：**
- 创建 `~/.claude/skills/learn-everything/SKILL.md` — Skill 入口文件（~130 行）
  - YAML frontmatter: name, description, allowed-tools, user-invocable
  - 角色定义（Bloom 2-Sigma 一对一辅导老师）
  - 5 条核心原则
  - 6 个 reference 文件索引表（progressive loading）
  - 5 阶段 workflow 概述 + 流程图
  - 项目路径约定 + 知识图谱节点状态定义
  - `/learn` 触发后的快速启动流程（新建 vs 继续）
  - Unknown Unknown 发现时机说明
- 创建 `~/.claude/skills/learn-everything/.claude-plugin/plugin.json` — 插件元信息
- 创建 `~/.claude/skills/learn-everything/references/` 目录（为 Step 3 预留）
- 验证 Skill 已被 Claude Code 识别，`/learn` 触发可用

---

## Step 3: 创建 Reference 文件 ✅

### 2026-03-05

**完成内容：**
- 创建 6 个 reference 文件于 `~/.claude/skills/learn-everything/references/`：
  1. `knowledge-graph-spec.md` — 知识图谱 JSON schema、状态转换规则、confidence 计算、meta.json 规范、初始化模板
  2. `session-flow.md` — 初始化/恢复逻辑、5 阶段循环详细步骤、session 摘要格式、循环/结束条件
  3. `diagnostic-engine.md` — 冷启动 5-8 题渐进诊断、继续学习记忆确认、动态调整规则、结果映射到知识图谱
  4. `document-generation.md` — 概念选择优先级、文档模板（定义→动机→原理→关联→边界）、质量标准、保存规范
  5. `understanding-check.md` — 阶段 3 验证策略（概念/机制/技能类）、阶段 4 练习形式（场景/对比/实践题）、confidence 计算
  6. `unknown-unknown-discovery.md` — 三层发现机制（自然暴露/边界扫描/跨领域关联）、记录格式规范
- 所有文件名与 SKILL.md 引用完全匹配
- Skill 现已具备完整执行指令，可通过 `/learn` 触发使用

---

## Step 4: 端到端测试 ⬜

**测试计划：**
- [ ] 选择一个具体领域进行冷启动测试
- [ ] 验证 5 阶段流程完整运行
- [ ] 验证文件系统状态正确持久化（meta.json, knowledge-graph.json, session, doc）
- [ ] 验证第二次 session 能正确恢复上下文
- [ ] 验证 unknown unknown 发现和记录机制

**状态：** 未开始

---

## 变更日志

| 日期 | 变更内容 | 涉及文件 |
|------|---------|---------|
| 2026-03-04 | 项目初始化，创建设计方案和项目指令 | CLAUDE.md, plan/v1-system-design.md |
| 2026-03-04 | 创建进度记录文档 | plan/PROGRESS.md |
| 2026-03-04 | 创建运行时目录 domains/, unknown-unknowns/ | .gitkeep files |
| 2026-03-04 | 创建 Skill 主文件和插件配置 | SKILL.md, plugin.json |
| 2026-03-05 | 创建 6 个 reference 文件，Skill 完整可用 | references/*.md |
