# LearnEverything — AI驱动的渐进式交互学习系统

## Context

受到一篇X推文启发，核心理念是：
- **AI生成渐进式文档**，而非一次性读书，每份文档基于当前理解水平生成
- **"不提问，让AI提问"** — AI通过提问检测你是否真正理解
- **对话记录驱动进度** — 根据暴露的理解程度决定下一份文档的难度和内容
- **Bloom 2 Sigma** — 一对一辅导可让学生达到前2%水平

要解决两个核心问题：
1. 通过提问测验发现 **unknown unknowns**（你不知道自己不知道什么）
2. 通过agent生成学习资料实现 **交互式学习**

## 系统架构

### 1. 文件系统结构

```
~/repos/LearnEverything/
├── CLAUDE.md                          # 项目级指令
├── domains/                           # 所有学习领域
│   └── {domain-name}/                 # 如 distributed-systems/
│       ├── meta.json                  # 领域元信息（创建时间、当前阶段、总进度）
│       ├── knowledge-graph.json       # 知识图谱（节点状态：mastered/learning/weak/unknown）
│       ├── sessions/                  # 学习session记录
│       │   └── {YYYY-MM-DD-HHmm}.md  # 每次session的摘要
│       └── docs/                      # AI生成的学习文档
│           └── {序号}-{概念名}.md      # 如 001-cap-theorem.md
└── unknown-unknowns/                  # 跨领域的unknown unknown发现记录
    └── {YYYY-MM-DD}.md                # 发现日志
```

### 2. Claude Code Skill 设计

创建一个主skill + 多个reference文件：

```
~/.claude/skills/learn-everything/
├── SKILL.md                           # 主skill定义
├── .claude-plugin/
│   └── plugin.json
└── references/
    ├── session-flow.md                # 完整session流程定义
    ├── diagnostic-engine.md           # 诊断式提问策略
    ├── document-generation.md         # 文档生成规范
    ├── understanding-check.md         # 理解度验证方法论
    ├── unknown-unknown-discovery.md   # 盲点发现机制
    └── knowledge-graph-spec.md        # 知识图谱数据结构规范
```

**触发方式：** `/learn`, `/学习`

### 3. 核心工作流（5阶段循环）

#### 阶段0: 初始化/恢复
- 新领域：用户指定想学什么 → 创建 domain 目录
- 继续学习：读取 knowledge-graph.json + 最近session摘要 → 恢复上下文

#### 阶段1: 诊断（冷启动 or 每次session开头）
- **冷启动**：围绕该领域的核心维度，用5-8个渐进式问题探测用户起点
  - 从广泛到具体，根据回答动态调整方向
  - 不给答案，只评估认知水平
- **继续学习**：快速回顾上次session的关键概念（2-3个问题确认是否遗忘）
- 输出：更新 knowledge-graph.json 中各节点状态

#### 阶段2: 文档生成
- 基于诊断结果，选择当前最适合学习的**单一概念**
- 生成500-1000字的聚焦文档，保存到 `docs/` 目录
- 文档结构：
  ```
  # {概念名}
  ## 它是什么（定义）
  ## 为什么重要（动机）
  ## 核心机制（原理）
  ## 与你已知概念的关联
  ## 边界与例外
  ```

#### 阶段3: AI提问验证（核心环节）
- **"不提问，让AI提问"** — AI对文档内容逐层提问
- 策略（混合判定）：
  - 概念类：要求用自己的话解释 + 举例
  - 机制类：追问边界情况和反例
  - 技能类：给出实际问题要求解决
- 追问深度：直到能清晰回答所有层面，或发现新的盲点
- 如果发现关联知识盲点 → 标记为 unknown unknown

#### 阶段4: 练习（可选，根据内容类型）
- 应用型知识：给出场景题
- 理论型知识：对比分析题
- 技能型知识：动手实践题（如写代码）

#### 阶段5: 总结 & 状态更新
- 生成session摘要 → 保存到 `sessions/` 目录
  - 本次学习的概念
  - 掌握程度评估
  - 发现的薄弱点和 unknown unknowns
  - 下次学习建议方向
- 更新 knowledge-graph.json
- 如有 unknown unknown 发现 → 追加到 `unknown-unknowns/` 日志

### 4. 知识图谱 (knowledge-graph.json)

```json
{
  "domain": "distributed-systems",
  "nodes": [
    {
      "id": "cap-theorem",
      "name": "CAP定理",
      "status": "mastered",        // mastered | learning | weak | unknown | undiscovered
      "confidence": 0.85,          // 0-1, AI评估的掌握度
      "lastTested": "2026-03-04",
      "dependencies": ["consistency-models", "partition-tolerance"],
      "relatedDomains": ["database-design"],  // 跨领域关联
      "sessionHistory": ["2026-03-04-1430"]
    }
  ],
  "edges": [
    { "from": "cap-theorem", "to": "consistency-models", "type": "requires" }
  ]
}
```

### 5. Unknown Unknown 发现机制（三层）

1. **学习过程中自然暴露**：AI提问时如果发现用户对某个前置/关联概念完全没有认知 → 标记
2. **阶段性知识边界扫描**：每学完一个主题（~5-8个概念后），AI主动列出该主题的相邻领域/前置知识/高级延伸，检查用户是否意识到这些方向的存在
3. **跨领域关联发现**：如果用户同时学习多个领域，AI尝试建立领域间的连接，检查用户是否意识到这些交叉点

## 实现步骤

### Step 1: 创建项目基础结构
- 初始化 `~/repos/LearnEverything/` 为 git repo
- 创建 `CLAUDE.md`（项目指令）
- 创建 `domains/` 和 `unknown-unknowns/` 目录

### Step 2: 创建 Skill 主文件
- `~/.claude/skills/learn-everything/SKILL.md` — 包含角色定义、workflow概述、触发方式
- `~/.claude/skills/learn-everything/.claude-plugin/plugin.json`

### Step 3: 创建 Reference 文件
按优先级：
1. `session-flow.md` — 完整5阶段流程的详细指令
2. `diagnostic-engine.md` — 诊断提问的策略和规则
3. `document-generation.md` — 文档生成的格式和质量标准
4. `understanding-check.md` — AI提问验证的方法论
5. `unknown-unknown-discovery.md` — 盲点发现的三层机制
6. `knowledge-graph-spec.md` — 知识图谱的数据结构和更新规则

### Step 4: 端到端测试
- 选择一个具体领域（如"分布式系统"或你感兴趣的领域）
- 运行 `/learn` 完整走一遍5阶段流程
- 验证文件系统状态是否正确持久化
- 验证第二次session能否正确恢复上下文

## 关键设计决策说明

1. **为什么用文件系统而非数据库**：与Claude Code原生集成，文件即上下文，Markdown可读性强
2. **为什么单一概念聚焦**：避免认知过载，确保每个概念都被充分验证理解
3. **为什么知识图谱用JSON**：结构化但轻量，AI可直接读写，支持关系追踪
4. **为什么session摘要用Markdown**：方便人类回顾，也方便AI快速加载上下文
