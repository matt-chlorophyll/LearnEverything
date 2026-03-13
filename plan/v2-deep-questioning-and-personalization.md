# 迭代计划：提问深度增强 + 学习者个性化

## Context

LearnEverything skill 已运行一周，用于维特根斯坦哲学学习，完成 5 次 session、掌握 4 个概念。用户反馈：

- **惊喜时刻**：验证提问阶段同时产生了"发现盲点"和"产生新连接"两种体验
- **提升方向**：提问深度（让 AI 追问更深、更刁钻）+ 个性化响应（建立学习者画像、智能反馈）
- 整体五阶段流程结构良好，不需要调整架构，只需**提升每个阶段的质量上限**

---

## 改动总览

新增 3 个 reference 文件，修改 5 个现有文件，新增 1 种数据文件：

| 文件 | 操作 | 内容 |
|------|------|------|
| `references/deep-questioning.md` | **新增** | 四种深度提问策略 + 选择逻辑 |
| `references/learner-profile-spec.md` | **新增** | 学习者画像 schema + 观察/更新规则 |
| `references/intelligent-feedback.md` | **新增** | 基于画像的反馈校准规则 |
| `references/understanding-check.md` | 修改 | 集成深度提问阶段、更新 confidence 计算、加入个性化原则 |
| `references/session-flow.md` | 修改 | 初始化/恢复时加载画像、阶段 5 更新画像、session 摘要格式扩展 |
| `references/knowledge-graph-spec.md` | 修改 | 加入 learner-profile.json 初始化模板 |
| `references/diagnostic-engine.md` | 修改 | 诊断时参考学习者画像 |
| `SKILL.md` | 修改 | 更新渐进加载表 + 目录结构 |
| `domains/{domain}/learner-profile.json` | **新增数据** | 每领域一个学习者画像文件 |

---

## Part 1: 深度提问策略

### 新文件 `references/deep-questioning.md`

定义四种策略，每种包含：定义、适用时机、深度限制、提问模式、退出条件。

**1. 追问链**：回答后继续深挖——"你说X，那如果Y呢？"最多 3 层，直到触及理解边界。

**2. 反例挑战**：给出似是而非的陈述——"有人说'{微妙的错误}'，你觉得对吗？"每轮最多 1 个。

**3. 边界探测**：探索概念失效的边界——"这在什么情况下不成立？"1-2 个边界问题。

**4. 元认知提问**：反思理解过程——"你觉得你最不确定的部分是什么？"每 session 最多 1 次。

### 策略选择逻辑

```
主验证问题（现有）
  ↓
回答 pass → 追问链(1-2层) → 反例挑战(概念/机制类) → 可选边界探测
回答 partial → 追问链(找到缺口) → 跳过高级策略
回答 fail → 不用高级策略，走引导重讲流程
元认知提问 → 每 session 用 1 次，通常在第 2-3 个概念后
```

### 关键设计：深度提问 ≠ 开放讨论

深度提问是 **AI 主动发起的有限追问**（有轮次上限），在现有"开放讨论窗口"**之前**执行。流程变为：

```
AI 提验证问题 → 用户回答 → AI 深度追问(有限) → AI 解析总结 → 开放讨论窗口(用户主导,无限) → 下一题
```

### 对 `understanding-check.md` 的修改

1. "阶段 3：提问验证" 部分加入"深度提问"子节，引用 `deep-questioning.md`
2. "追问深度策略" 树整合新策略选择逻辑
3. "开放讨论窗口" 注明在深度提问完成后才开始
4. Confidence 计算加入深度提问奖励：追问链 3 层全过 或 独立捕捉反例 → 额外 +0.1（每概念上限 +0.1）

---

## Part 2: 学习者画像

### 新文件 `references/learner-profile-spec.md`

**存储位置**：`domains/{domain-name}/learner-profile.json`（每领域独立）

**Schema**：
```json
{
  "domain": "string",
  "sessionsObserved": 0,
  "thinkingPatterns": [
    { "pattern": "偏好类比思维", "confidence": 0.5, "evidence": ["session记录"] }
  ],
  "strengths": [
    { "description": "自我纠错能力强", "confidence": 0.5, "evidence": [] }
  ],
  "weaknesses": [
    { "description": "有时混淆X和Y", "confidence": 0.3, "evidence": [] }
  ],
  "preferences": {
    "scaffoldingLevel": "high | medium | low",
    "challengeAppetite": "high | medium | low"
  },
  "confidenceTrend": "improving | stable | declining"
}
```

**Confidence 规则**：首次观察 0.3，每次确认 +0.2（上限 0.9），矛盾观察 -0.3（下限 0.0 则删除）。5 session 未观察到则每 session 衰减 0.1。

**观察信号 → 画像更新映射**：

| 观察到的行为 | 更新画像 |
|-------------|---------|
| 用户主动使用类比 | 强化 "偏好类比思维" |
| 用户在 AI 纠正前自己发现错误 | 强化 "自我纠错能力" |
| 用户主动问边界情况 | 提升 challengeAppetite |
| 用户回答简短、不确定 | scaffoldingLevel 趋向 high |
| 用户给出深入的延展分析 | scaffoldingLevel 趋向 low |

**更新时机**：Phase 3-4 期间内部记录观察，Phase 5 集中更新画像文件。

### 对 `session-flow.md` 的修改

1. 新领域初始化：加入创建 `learner-profile.json`
2. 继续学习恢复：加入读取 `learner-profile.json`
3. 阶段 5 总结：在"更新知识图谱"后、"生成 session 摘要"前，加入"更新学习者画像"步骤
4. Session 摘要格式扩展：加入"学习者画像更新"小节

---

## Part 3: 智能反馈

### 新文件 `references/intelligent-feedback.md`

阶段 3 开始时与 `understanding-check.md` 一起读取，利用学习者画像校准反馈。

**核心规则**：

1. **具体肯定**：不说"正确！"，而是指出回答中具体好的推理步骤
2. **连接画像**：如果画像显示"擅长类比"且用户用了好类比，点名表扬；如果画像有弱点但这次避开了，提及进步
3. **延伸洞察**："你的回答暗示了一个更深的问题..." / "你的推理方式跟{后续概念}的核心思路接近"
4. **脚手架调节**：scaffoldingLevel low → 更直接的挑战、更少引导；high → 拆分子问题、更多语境
5. **挑战校准**：challengeAppetite high → 更多反例挑战和边界探测

**反模式**：
- 不要机械引用画像（"根据你的画像..."），要自然融入
- 不要让画像覆盖当下观察——如果这次表现与画像不同，响应实际情况
- 不要过度个性化导致可预测

### 对 `understanding-check.md` 的修改

交互原则第 5 条：加入"个性化回应——参考 `learner-profile.json`，策略详见 `intelligent-feedback.md`"

### 讨论记录格式扩展

在现有"总结"后加入"学习者行为观察"小节，为阶段 5 更新画像提供素材。

---

## 实施顺序

5 步，每步可独立测试：

1. **创建 `learner-profile-spec.md`** + 更新 `session-flow.md`（初始化/加载画像）+ 更新 `knowledge-graph-spec.md`（模板）
2. **创建 `deep-questioning.md`** + 更新 `understanding-check.md`（集成深度提问）
3. **创建 `intelligent-feedback.md`** + 更新 `understanding-check.md`（个性化反馈原则）+ 更新 `document-generation.md`
4. **更新 `session-flow.md`** 阶段 5（画像更新步骤 + 摘要格式）+ 讨论记录格式
5. **更新 `SKILL.md`**（渐进加载表 + 目录结构）+ `CLAUDE.md` + `diagnostic-engine.md`

---

## 验证方法

实施完成后，通过一次完整的 `/learn` session 端到端验证：

1. 启动 session → 确认 `learner-profile.json` 被创建/读取
2. 进入阶段 3 → 验证是否出现深度追问（而非只问一轮就进入讨论窗口）
3. 验证反馈是否具体而非通用的"正确！"
4. 阶段 5 → 确认 session 摘要包含"学习者画像更新"、`learner-profile.json` 被更新
5. 对已有领域（wittgenstein）：为其补建 `learner-profile.json`，基于已有 5 次 session 数据初始化画像
