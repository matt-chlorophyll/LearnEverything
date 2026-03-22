# LearnEverything

一个 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill，把 AI 变成你的一对一私教。

基于 Bloom 2-Sigma 理论——研究表明一对一辅导的学生表现超过传统课堂 98% 的学生。核心原则：**"不提问，让 AI 提问"**。

## 它解决什么问题？

传统学习方式有两个致命缺陷：

1. **你不知道自己不知道什么（unknown unknowns）**——没人帮你发现知识盲点
2. **读书效率低**——被动阅读缺乏反馈和验证

LearnEverything 的解决方案：AI 为你生成个性化学习文档，然后**反过来考你**，通过提问暴露你真正的理解程度和知识盲点。

## 快速开始

### 前置条件

- 安装 [Claude Code](https://docs.anthropic.com/en/docs/claude-code)

### 使用方式

```bash
git clone https://github.com/matt-chlorophyll/LearnEverything.git
cd LearnEverything
claude
```

进入 Claude Code 后，输入：

```
/learn
```

然后告诉 AI 你想学什么就行了。

## 学习流程

每次学习 session 由 5 个阶段组成：

```
诊断 → 文档生成 → 提问验证 → 练习 → 总结
  ↑                                      |
  └──────── 循环：选择下一个概念 ←────────┘
```

| 阶段 | 做什么 |
|------|--------|
| **诊断** | AI 用 5-8 个渐进问题探测你的起点，找到最适合的学习切入点 |
| **文档生成** | 针对单一概念生成 500-1000 字的学习材料，匹配你当前的理解水平 |
| **提问验证** | AI 反过来考你——要求你用自己的话解释、举例、解决边界情况。包含深度追问：追问链、反例挑战、边界探测、元认知提问 |
| **练习** | 场景题、对比题、实践题巩固理解 |
| **总结** | 更新知识图谱，记录 session 摘要，标记发现的知识盲点 |

## 学习者个性化

系统会在学习过程中自动构建你的**学习者画像**，追踪：

- **思维模式**——你偏好类比思维还是逻辑推演？
- **优势与弱点**——迁移应用能力强？容易混淆边界情况？
- **学习偏好**——你喜欢先看例子还是先看定义？

画像会影响后续的文档生成、提问难度和反馈风格，让每次学习都更贴合你的特点。

## 学习数据示例

查看 [`domains/wittgenstein-philosophy/`](domains/wittgenstein-philosophy/) 了解一个真实学习过程产生的数据：

- **知识图谱** ([knowledge-graph.json](domains/wittgenstein-philosophy/knowledge-graph.json))——追踪每个概念的掌握状态（mastered / learning / weak / unknown / undiscovered）
- **学习者画像** ([learner-profile.json](domains/wittgenstein-philosophy/learner-profile.json))——基于观察构建的思维模式、优势与弱点档案
- **学习文档** ([docs/](domains/wittgenstein-philosophy/docs/))——AI 生成的个性化学习材料
- **讨论记录** ([discussions/](domains/wittgenstein-philosophy/discussions/))——提问验证的对话记录
- **Session 摘要** ([sessions/](domains/wittgenstein-philosophy/sessions/))——每次学习的总结与下次建议

## 设计方案

- **v1**：核心五阶段学习流程——详见 [`plan/v1-system-design.md`](plan/v1-system-design.md)
- **v2**：深度提问策略 + 学习者个性化——详见 [`plan/v2-deep-questioning-and-personalization.md`](plan/v2-deep-questioning-and-personalization.md)
