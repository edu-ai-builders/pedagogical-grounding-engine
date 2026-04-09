# 第3层：KG 融合 + Constraint Layer

## 层定位

> **目标**：把两个独立的 KG（论文机制 + 场景约束）融合，把隐含的设计张力**显式化**。
> **产物**：融合操作结果 + Constraint 节点集合 + 约束间关系网络（含 Resolution）。
> **核心价值**：这一层决定了"为什么不是别的 feature"——这是整个系统的设计辩护力来自哪里。

---

## 输入契约（Preconditions）

| 输入项 | 要求 |
|---|---|
| 第1层 Core KG（Enriched） | ≥20条三元组/篇，**所有 [F] 和核心 [程序] triple 已完成阶段4b Representation Rewrite** |
| 第2层 Context KG | 10-15条，每条有 H/M/L + Must/Tradeoff/Nice 标注 |

**关键**：本层直接使用 enriched triple 的字段做融合判断：
- `context_constraint` 字段 → 与 Context KG 节点做直接对应检测
- `likely_failure_mode` 字段 → 作为识别设计张力和实现障碍的线索
- `implementation_form` 字段 → 作为推断节点生成的起点

---

## 处理流程

### 阶段1：KG 融合操作

逐一执行四种融合操作，每种操作产出独立的结果列表。

---

**操作1：直接对应（Direct Alignment）**

判断标准：Core KG 中的某个机制，直接回应了 Context KG 中的某个约束需求，无需中间步骤。

```
检测逻辑：
  FOR 每条 Core KG 三元组 [F] 标注：
    FOR 每条 Context KG 约束节点：
      IF 客体/主体在语义上解决了约束需求：
        记录为直接对应对
```

输出格式：
```
直接对应（N组）：
A. [Core KG三元组] ←直接对应→ [Context KG约束节点]
   对应说明：[1句话，为什么这个机制直接解决了这个需求]
```

---

**操作2：间接激活（Indirect Activation）**

判断标准：Core KG 机制和 Context KG 需求之间存在"语义距离"，需要一个**中间设计节点**才能桥接。

```
检测逻辑：
  IF Core KG机制 与 Context KG需求 相关但不直接对应：
    问："需要什么中间设计M，使得[机制]能够满足[需求]？"
    IF M可以被明确描述：
      生成推断节点M，标注[推断]
      记录为间接激活三元组
```

推断节点格式：
```json
{
  "id": "inferred_001",
  "label": "推断节点名称",
  "type": "inferred",
  "derived_from": {
    "core_kg": "触发推断的Core KG三元组",
    "context_kg": "触发推断的Context KG约束节点"
  },
  "rationale": "为什么这个中间设计是必要的"
}
```

输出格式：
```
间接激活（N组）：
A. [Core KG机制] ←间接激活→ [Context KG需求]
   中间设计（推断节点）：[推断节点名称]
   推断依据：[为什么需要这个中间设计]
```

---

**操作3：设计张力（Design Tension）**

判断标准：Core KG 中的两个或多个机制，在面对 Context KG 的某个约束时产生方向矛盾。或者，两篇论文的冲突节点在场景中被激活。

**重要**：不裁决张力，只命名它。裁决在 Constraint Layer 进行。

输出格式：
```
设计张力（N个）：
A. 张力名称："[A机制] vs [B机制]"
   激活因素：[是哪个场景约束使这个张力变得重要]
   两个方向：
   - 方向1：如果优先[A机制]，会得到什么，代价是什么
   - 方向2：如果优先[B机制]，会得到什么，代价是什么
   → 带入 Constraint Layer 处理
```

---

**操作4：实现障碍（Implementation Barrier）**

判断标准：论文机制在技术或情境上与场景约束**直接冲突**，无法直接实现。

输出格式：
```
实现障碍（N个）：
A. [论文机制] 受限于 [场景约束]
   障碍类型：[技术限制 / 资源约束 / 情境不兼容]
   → 必须在 Constraint Layer 提供降级方案
```

---

### 阶段2：Constraint Layer

这是本层的核心。把阶段1的所有结果提炼为**可命名、可推理的约束系统**。

---

**步骤2a：约束节点提取**

从以下来源提取命名约束：
1. Core KG 中的冲突节点（两篇论文主张不同）
2. Context KG 中所有 [Must] 节点
3. Context KG 中 [H] 级别的 [Tradeoff] 节点
4. 阶段1的设计张力（每个张力背后是一对约束）
5. 阶段1的实现障碍（障碍本身是一个约束）

约束节点 Schema：
```json
{
  "id": "constraint_XXX",
  "type": "pedagogical | technical | contextual",
  "source": "论文A / 论文B / 场景 / 推断",
  "claim": "一句话：这个约束主张什么是必要的/不可违反的",
  "priority": "High | Medium | Low",
  "negotiable": true | false,
  "origin": "对应的Core KG三元组ID 或 Context KG节点描述"
}
```

类型说明：
- `pedagogical`：教学法主张，来自论文，通常 negotiable=false
- `technical`：技术限制，来自场景，通常可降级
- `contextual`：用户/情境需求，来自场景，有优先级之分

---

**步骤2b：约束间关系分析**

对每对约束节点，判断关系类型：

```
关系判断决策树：

Q1: 两个约束能同时完全满足吗？
  YES → independent（独立，不记录，无需处理）
  NO  → Q2

Q2: 满足A会主动损害B吗？
  YES → conflict（冲突，必须裁决）
  NO  → Q3

Q3: 满足A会部分牺牲B的实现质量吗？
  YES → tradeoff（权衡，需要Resolution但不必完全裁决）
  NO  → Q4

Q4: 同时满足A和B会比单独满足任一个更好吗？
  YES → amplify（强化，合力指向同一方向）
```

**注意**：只记录 conflict、tradeoff、amplify 三种关系；independent 不需要记录。

约束关系 Schema：
```json
{
  "id": "rel_XXX",
  "constraint_a": "constraint_XXX",
  "constraint_b": "constraint_YYY",
  "relationship": "conflict | tradeoff | amplify",
  "description": "具体描述两个约束如何产生这种关系（结合场景说明）",
  "stakes": "如果不处理这个关系会发生什么",
  "resolution": {
    "decision": "具体的取舍方案",
    "rationale": "为什么选这个方案而不是其他",
    "sacrifice": "明确说明牺牲了什么",
    "alternative_considered": "被排除的替代方案是什么，为什么排除",
    "resulting_decision_id": "decision_XXX（第4层将生成的节点ID）"
  }
}
```

**Resolution 的质量标准**：
- `decision` 必须具体到可执行（"用点状仪表盘"而非"简化界面"）
- `sacrifice` 必须明确（"牺牲了精确数值"而非"略有不足"）
- `alternative_considered` 必须列出至少1个被排除的方案及原因
- **没有 `alternative_considered` 的 Resolution 不完整**

---

**步骤2c：Constraint Layer 摘要**

输出一段分析文字（3-5句话），回答：
> "哪些约束决定了整个设计方向？为什么这个产品不能是别的样子？"

这段摘要是整个 Skill 最重要的叙事内容之一。

---

## 输出契约（Postconditions）

| 输出项 | 格式 | 保证 |
|---|---|---|
| 融合操作结果 | Markdown（四操作各自列表）| 每种操作有数量统计 |
| 约束节点集合 | JSON + Markdown | 每个节点有完整 Schema |
| 约束间关系网络 | JSON + Markdown | 每个 conflict/tradeoff 有 Resolution |
| Constraint Layer 摘要 | Markdown 段落 | 3-5句，有明确叙事 |

**✅ 完成行格式**：
```
✅ 第3层完成：直接对应N / 间接激活N（推断节点N）/ 设计张力N / 实现障碍N
              约束节点N个（教学法X/技术Y/情境Z）
              约束关系M组：冲突A / 权衡B / 强化C
              全部 conflict 和 tradeoff 已有 Resolution
```

---

## 错误处理

| 错误情况 | 处理 |
|---|---|
| 没有任何直接对应 | 说明"论文机制与场景需求没有直接匹配"，检查是否论文选择与场景相关性太低 |
| 约束节点间几乎全是 independent | 说明约束太少或太宽泛，回到第2层补充具体约束 |
| Resolution 无法做出（两个 Must 约束直接冲突）| 标注`[无解冲突: 需要用户做基本假设选择]`，输出两个备选方向，等待用户选择 |
| 推断节点无法被合理命名 | 不要强行生成推断节点，标注"此处需要领域专家补充中间设计" |

---

## 质量自检

- [ ] 四种融合操作都执行了，没有跳过？
- [ ] 每个 conflict/tradeoff 关系是否有 Resolution？
- [ ] Resolution 是否包含 alternative_considered？
- [ ] 约束节点是否有来源标注（来自论文还是场景）？
- [ ] Constraint Layer 摘要是否有实质性叙事（而非"本层完成了约束分析"这样的废话）？
- [ ] 完成行数字是否准确？
