# 完整 5 阶段学习流程

本文件定义 session 的完整执行流程。每次 session 开始时读取此文件。

---

## Session 初始化

### 新领域初始化

1. 根据用户指定的领域名，生成英文标识（小写、`-` 连接）
2. 创建目录结构：
   ```
   ~/repos/LearnEverything/domains/{domain-name}/
   ├── meta.json
   ├── knowledge-graph.json
   ├── learner-profile.json
   ├── sessions/
   ├── docs/
   ├── discussions/
   └── quotes/
   ```
3. 按 `knowledge-graph-spec.md` 中的模板初始化 `meta.json`、`knowledge-graph.json` 和 `learner-profile.json`
4. 进入**阶段 1（冷启动诊断）**

### 继续学习恢复

1. 读取 `meta.json` 获取领域状态
2. 读取 `knowledge-graph.json` 加载知识图谱
3. 读取 `learner-profile.json` 加载学习者画像
4. 读取 `sessions/` 下最近一次 session 摘要（按文件名排序取最新）
5. 向用户简要报告学习进度：已掌握概念数 / 总概念数，上次学习的概念
6. **学习意图对话**：询问"今天想聊点什么方向？有什么特别想了解的吗？"，将用户意图作为概念选择参考；如果用户没有特别想法，按知识图谱推荐继续
7. 进入**阶段 1（记忆确认诊断）**

---

## 5 阶段循环

### 阶段 1：诊断

**读取**：`references/diagnostic-engine.md`

- **冷启动**（新领域 / 首次 session）：执行 5-8 题渐进诊断，探测用户的起始水平
- **继续学习**：执行 2-3 题记忆确认，检查上次学习内容的记忆保持度

**输出**：更新知识图谱节点状态，确定用户当前水平位置

**转换到阶段 2**：诊断完成后，选择最优下一个学习概念

### 阶段 2：文档生成

**读取**：`references/document-generation.md`

1. 基于知识图谱选择下一个概念（优先级：weak > unknown > 新发现）
2. 生成 500-1000 字的聚焦学习文档
3. 保存到 `domains/{domain}/docs/{序号}-{概念名}.md`
4. 将文档内容直接呈现给用户阅读

**转换到阶段 3**：等用户确认阅读完成（或直接提问开始验证）

### 阶段 3：提问验证

**读取**：`references/understanding-check.md`（阶段 3 部分）+ `references/intelligent-feedback.md`

1. 根据概念类型选择验证策略（概念类/机制类/技能类）
2. 提出验证问题，要求用户用自己的话回答
3. 根据回答质量执行深度追问（策略见 `deep-questioning.md`）
4. AI 解析总结后开放讨论窗口，用户确认后再继续下一题
5. 评估理解程度
6. 内部记录学习者行为观察（为阶段 5 画像更新提供素材）

**同时执行**：读取 `references/unknown-unknown-discovery.md`，在验证过程中识别关联盲点

**转换到阶段 4**：验证完成后进入练习

### 阶段 4：练习

**读取**：`references/understanding-check.md`（阶段 4 部分）

1. 根据概念类型选择练习形式（场景题/对比题/实践题）
2. 出 1-2 道练习题
3. 每题评估后开放讨论窗口，用户确认后再继续下一题
4. 评估练习结果，计算 confidence

**同时执行**：继续 unknown unknown 发现

**转换到阶段 5**：练习完成后，先保存验证对话记录到 `domains/{domain}/discussions/`（格式详见 `understanding-check.md` 的"验证对话记录"一节），然后进入总结

### 阶段 5：总结

**读取**：`references/knowledge-graph-spec.md`（阶段 5 更新操作）+ `references/learner-profile-spec.md`

1. **更新知识图谱**：按 `knowledge-graph-spec.md` 的规则更新节点状态和 confidence
2. **更新学习者画像**：根据阶段 3-4 期间的行为观察，更新 `learner-profile.json`（规则详见 `learner-profile-spec.md`）
3. **生成 session 摘要**：保存到 `sessions/{YYYY-MM-DD-HHmm}.md`，包含：
   - 本次学习的概念及掌握情况
   - 发现的 unknown unknowns
   - 学习者画像更新内容
   - 下次建议学习方向
4. **记录 unknown unknowns**：如有发现，按 `unknown-unknown-discovery.md` 的规范记录
5. **向用户汇报**：本次学习成果、掌握度变化、建议下一步

---

## Session 摘要格式

```markdown
# Session 摘要 — {YYYY-MM-DD HH:mm}

## 领域：{displayName}

## 本次学习
- 概念：{概念名}
- 掌握度：{confidence} ({status})
- 验证结果：{pass/partial/fail}

## 发现的盲点
- {列出本次发现的 unknown unknowns，无则写"无"}

## 学习者画像更新
- 本次观察：{记录观察到的思维模式、长处、短板等}
- 画像变更：{新增/强化/弱化了哪些画像条目，无变更写"无"}

## 下次建议
- {根据知识图谱推荐下一个学习概念及原因}
```

---

## 金句记录环节

阶段 5 总结完成后、询问是否继续下一个概念**之前**，插入金句记录：

1. 使用 AskUserQuestion 询问用户："这个概念的学习中有没有想记录的金句或精彩表述？"
   - 选项：有 / 没有
2. 如果用户选择"有"：进入自由对话模式
   - 用户输入想记录的内容（可以是 AI 说的原话，也可以是用户自己的感悟）
   - 支持多条，用户每输入一条后询问"还有吗？"
   - 直到用户说"记完了"/"没有了"等结束语
3. 保存到 `domains/{domain}/quotes/{YYYY-MM-DD}.md`
   - 追加写入（同一天多个概念或多次 session 追加到同一文件）
   - 每条金句标注来源概念名称
4. 如果用户选择"没有"：直接跳过，进入下一步

### 金句文件格式

```markdown
# 学习金句 — {YYYY-MM-DD}

## {概念名称}

> {金句内容}

> {金句内容}

## {另一个概念名称}

> {金句内容}
```

---

## 循环与结束

### 继续下一个概念

阶段 5 完成后，使用 AskUserQuestion 询问用户：
- **继续学习下一个概念**（推荐具体概念名）
- **结束本次 session**

如果继续，回到阶段 2（跳过阶段 1 诊断，因为 session 中已有连续上下文）。

### Session 结束条件

- 用户主动说结束/停止/够了
- 用户选择结束 session
- 单次 session 建议不超过 3-5 个概念（可提醒但不强制）

### 阶段性边界扫描

每完成 5-8 个概念后（跨 session 累计），在阶段 5 额外执行一次边界扫描（详见 `unknown-unknown-discovery.md`），主动探测相邻知识域。
