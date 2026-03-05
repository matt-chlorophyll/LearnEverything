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
   ├── sessions/
   ├── docs/
   └── discussions/
   ```
3. 按 `knowledge-graph-spec.md` 中的模板初始化 `meta.json` 和 `knowledge-graph.json`
4. 进入**阶段 1（冷启动诊断）**

### 继续学习恢复

1. 读取 `meta.json` 获取领域状态
2. 读取 `knowledge-graph.json` 加载知识图谱
3. 读取 `sessions/` 下最近一次 session 摘要（按文件名排序取最新）
4. 向用户简要报告学习进度：已掌握概念数 / 总概念数，上次学习的概念
5. 进入**阶段 1（记忆确认诊断）**

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

**读取**：`references/understanding-check.md`（阶段 3 部分）

1. 根据概念类型选择验证策略（概念类/机制类/技能类）
2. 提出验证问题，要求用户用自己的话回答
3. 根据回答质量追问或补充
4. 每题解析后开放讨论窗口，用户确认后再继续下一题
5. 评估理解程度

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

**读取**：`references/knowledge-graph-spec.md`（阶段 5 更新操作）

1. **更新知识图谱**：按 `knowledge-graph-spec.md` 的规则更新节点状态和 confidence
2. **生成 session 摘要**：保存到 `sessions/{YYYY-MM-DD-HHmm}.md`，包含：
   - 本次学习的概念及掌握情况
   - 发现的 unknown unknowns
   - 下次建议学习方向
3. **记录 unknown unknowns**：如有发现，按 `unknown-unknown-discovery.md` 的规范记录
4. **向用户汇报**：本次学习成果、掌握度变化、建议下一步

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

## 下次建议
- {根据知识图谱推荐下一个学习概念及原因}
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
