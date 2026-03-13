# 学习者画像规范

本文件定义 `learner-profile.json` 的 schema、观察信号和更新规则。

---

## 存储位置

`domains/{domain-name}/learner-profile.json`（每领域独立）

---

## Schema

```json
{
  "domain": "string",
  "sessionsObserved": 0,
  "thinkingPatterns": [
    {
      "pattern": "偏好类比思维",
      "confidence": 0.5,
      "evidence": ["session 中的具体表现"],
      "lastObserved": "ISO 8601 timestamp"
    }
  ],
  "strengths": [
    {
      "description": "自我纠错能力强",
      "confidence": 0.5,
      "evidence": [],
      "lastObserved": "ISO 8601 timestamp"
    }
  ],
  "weaknesses": [
    {
      "description": "有时混淆X和Y",
      "confidence": 0.3,
      "evidence": [],
      "lastObserved": "ISO 8601 timestamp"
    }
  ],
  "preferences": {
    "scaffoldingLevel": "high | medium | low",
    "challengeAppetite": "high | medium | low"
  },
  "confidenceTrend": "improving | stable | declining"
}
```

### 字段说明

| 字段 | 说明 |
|------|------|
| `sessionsObserved` | 累计观察的 session 数 |
| `thinkingPatterns` | 思维模式特征（如偏好类比、善于抽象化等） |
| `strengths` | 学习长处 |
| `weaknesses` | 学习短板 |
| `preferences.scaffoldingLevel` | 脚手架水平：high=需要更多引导，low=可直接挑战 |
| `preferences.challengeAppetite` | 挑战偏好：high=喜欢反例和边界问题 |
| `confidenceTrend` | 近期掌握度趋势（基于最近 3 个概念的 confidence 变化） |

---

## 初始化模板

```json
{
  "domain": "{domain-name}",
  "sessionsObserved": 0,
  "thinkingPatterns": [],
  "strengths": [],
  "weaknesses": [],
  "preferences": {
    "scaffoldingLevel": "medium",
    "challengeAppetite": "medium"
  },
  "confidenceTrend": "stable"
}
```

---

## Confidence 规则

画像中每个条目（thinkingPatterns / strengths / weaknesses）都有独立的 confidence：

| 事件 | 变化 |
|------|------|
| 首次观察到 | 设为 0.3 |
| 后续 session 再次确认 | +0.2（上限 0.9） |
| 观察到矛盾行为 | -0.3（降至 0.0 则删除该条目） |
| 连续 5 个 session 未观察到 | 每 session 衰减 0.1 |

---

## 观察信号 → 画像更新

在阶段 3-4 期间**内部**记录观察，不打断教学流程。阶段 5 集中更新画像文件。

### 思维模式信号

| 观察到的行为 | 更新 |
|-------------|------|
| 用户主动使用类比来解释概念 | 强化 `thinkingPatterns` "偏好类比思维" |
| 用户用形式化/结构化方式表达 | 强化 "偏好结构化分析" |
| 用户从具体例子出发归纳规律 | 强化 "归纳式思维" |
| 用户从原理出发推导到具体 | 强化 "演绎式思维" |

### 长处信号

| 观察到的行为 | 更新 |
|-------------|------|
| 用户在 AI 纠正前自己发现并修正错误 | 强化 `strengths` "自我纠错能力" |
| 用户能将概念迁移到新场景 | 强化 "迁移应用能力" |
| 用户主动提出有深度的追问 | 强化 "深度探索意识" |
| 用户能准确识别概念间的关联 | 强化 "关联思维" |

### 短板信号

| 观察到的行为 | 更新 |
|-------------|------|
| 用户反复在同类问题上犯错 | 记录 `weaknesses` 具体的混淆点 |
| 用户理解表面含义但无法延展 | 记录 "停留在表面理解" |
| 用户倾向于背诵而非理解 | 记录 "记忆导向而非理解导向" |

### 偏好信号

| 观察到的行为 | 更新 |
|-------------|------|
| 用户回答简短、不确定、频繁求助 | `scaffoldingLevel` 趋向 high |
| 用户给出深入的延展分析 | `scaffoldingLevel` 趋向 low |
| 用户主动问边界情况、反例 | `challengeAppetite` 趋向 high |
| 用户对复杂追问表现抗拒或疲惫 | `challengeAppetite` 趋向 low |

---

## confidenceTrend 计算

基于最近 3 个概念的最终 confidence 值：

- 连续上升或平均 ≥ 0.6 → `improving`
- 波动不大 → `stable`
- 连续下降或平均 < 0.4 → `declining`

---

## 更新时机

1. **阶段 3-4 期间**：内部记录观察（不写文件、不告知用户）
2. **阶段 5**：集中更新 `learner-profile.json`，递增 `sessionsObserved`，应用所有观察结果
3. **Session 摘要**：在摘要中加入"学习者画像更新"小节，说明本次观察到了什么、做了什么更新
