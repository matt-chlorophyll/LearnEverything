# 知识图谱数据规范

本文件定义 `knowledge-graph.json` 和 `meta.json` 的完整 schema 及操作规则。

---

## knowledge-graph.json Schema

```json
{
  "nodes": [
    {
      "id": "string",
      "name": "string",
      "status": "mastered | learning | weak | unknown | undiscovered",
      "confidence": 0.0,
      "lastTested": "ISO 8601 timestamp | null",
      "dependencies": ["node-id"],
      "relatedDomains": ["domain-name"],
      "sessionHistory": [
        {
          "date": "ISO 8601 timestamp",
          "result": "pass | partial | fail",
          "notes": "string"
        }
      ]
    }
  ],
  "edges": [
    {
      "from": "node-id",
      "to": "node-id",
      "type": "requires | relates-to | extends"
    }
  ]
}
```

### 节点字段说明

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | string | 小写英文，用 `-` 连接，如 `binary-search` |
| `name` | string | 概念的中文显示名称 |
| `status` | enum | 掌握状态，见下方状态转换规则 |
| `confidence` | float | 0.0-1.0，掌握度评分 |
| `lastTested` | string/null | 最后一次验证时间，未测试过为 null |
| `dependencies` | array | 前置依赖概念的 id 列表 |
| `relatedDomains` | array | 关联的其他学习领域名称 |
| `sessionHistory` | array | 历次学习记录 |

### 边类型说明

| 类型 | 含义 | 示例 |
|------|------|------|
| `requires` | A 是 B 的前置条件 | `variable` requires `data-type` |
| `relates-to` | A 与 B 相关但无依赖 | `recursion` relates-to `stack` |
| `extends` | A 是 B 的深化/扩展 | `binary-search` extends `search` |

---

## 状态转换规则

```
undiscovered ──发现概念──▶ unknown
unknown ──开始学习──▶ learning
learning ──验证通过(confidence ≥ 0.7)──▶ mastered
learning ──验证不佳(confidence < 0.4)──▶ weak
mastered ──复习验证失败──▶ weak
weak ──重新学习+验证通过──▶ mastered
weak ──重新学习中──▶ learning
```

### confidence 计算规则

- 验证阶段（阶段 3）回答正确且能举例说明：+0.3
- 练习阶段（阶段 4）独立完成：+0.2
- 验证阶段需要提示才能回答：+0.1
- 练习阶段需要提示：+0.1
- 回答错误或无法回答：+0.0
- 基础分：首次学习从 0.0 开始；复习在原 confidence 基础上调整

### 状态判定阈值

| 目标状态 | 条件 |
|----------|------|
| `mastered` | confidence ≥ 0.7 且通过验证+练习 |
| `learning` | 正在当前 session 学习 |
| `weak` | confidence < 0.4 或复习时表现不佳 |
| `unknown` | 已知存在但未开始学习 |

---

## meta.json Schema

```json
{
  "domain": "string",
  "displayName": "string",
  "createdAt": "ISO 8601 timestamp",
  "lastSessionAt": "ISO 8601 timestamp | null",
  "totalSessions": 0,
  "totalConcepts": 0,
  "masteredConcepts": 0,
  "currentFocus": "node-id | null"
}
```

| 字段 | 说明 |
|------|------|
| `domain` | 领域英文标识，同目录名 |
| `displayName` | 领域中文显示名称 |
| `createdAt` | 领域创建时间 |
| `lastSessionAt` | 最后一次学习时间 |
| `totalSessions` | 累计 session 数 |
| `totalConcepts` | 知识图谱总节点数 |
| `masteredConcepts` | mastered 状态节点数 |
| `currentFocus` | 当前正在学习的概念 id |

---

## 初始化模板

### 新领域 knowledge-graph.json

```json
{
  "nodes": [],
  "edges": []
}
```

### 新领域 meta.json

```json
{
  "domain": "{domain-name}",
  "displayName": "{用户提供的领域名}",
  "createdAt": "{当前 ISO 8601 时间}",
  "lastSessionAt": null,
  "totalSessions": 0,
  "totalConcepts": 0,
  "masteredConcepts": 0,
  "currentFocus": null
}
```

---

## 阶段 5 更新操作

Session 结束时，按以下步骤更新数据：

1. **更新节点状态**：根据本次 session 的验证和练习结果，更新涉及节点的 `status`、`confidence`、`lastTested`，追加 `sessionHistory`
2. **添加新节点**：将本次发现的新概念（包括 unknown unknowns）添加为新节点，状态设为 `unknown` 或 `undiscovered`
3. **添加/更新边**：根据学习过程中发现的概念关系，添加或更新边
4. **更新 meta.json**：递增 `totalSessions`，更新 `lastSessionAt`、`totalConcepts`、`masteredConcepts`，设置 `currentFocus`
