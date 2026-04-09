# 第4层：Traceability Engine — 核心推理层

## 层定位

> **目标**：为每一个设计决策写清楚完整的推理链，推理起点是 Constraint Layer 的 Resolution。
> **产物**：Traceability 节点集合（每节点含三层输出）+ 跨决策连接网络。
> **关键约束**：没有 constraint_resolution 的 P0 决策不允许进入第5层。

---

## 输入契约（Preconditions）

| 输入项 | 要求 |
|---|---|
| 第3层 Constraint Layer | 所有 conflict/tradeoff 关系已有 Resolution，含 resulting_decision_id |
| 第3层推断节点 | 已有 derived_from 来源标注 |
| 第2层 Context KG | 含 H/M/L + Must/Tradeoff/Nice 标注 |

---

## 处理流程

### 阶段1：决策列表生成

**从 Constraint Layer 的 Resolution 自动生成决策列表**：

```
FOR 每个 conflict/tradeoff 关系的 Resolution：
  → 生成一个 decision_XXX 节点

FOR 每个 amplify 关系：
  → 生成一个 decision_XXX 节点（通常是 P1 或 P2）

FOR 每个直接对应的融合操作：
  → 如果对应的功能足够重要（是直接可实现的），生成 decision_XXX
```

**优先级分配规则**：

| 来源 | 默认优先级 | 可调整条件 |
|---|---|---|
| conflict Resolution（pedagogical 约束相关）| P0 | — |
| conflict Resolution（technical 约束相关）| P1 | 如果技术约束是 Must，可升 P0 |
| tradeoff Resolution | P1 | 如果对应 [H][Must] 约束，升 P0 |
| amplify 关系 | P1 | — |
| 直接对应（功能明确）| P0 | — |
| 间接激活（推断节点）| P1 | 推断节点本身不直接升 P0 |

完成后输出决策列表（先列出再逐一处理）：
```
决策列表（共N个）：
P0: [名称] × K个
P1: [名称] × M个
P2: [名称] × J个
```

---

### 阶段2：逐节点展开（每个决策）

对每个决策节点，按优先级依次输出三层内容。

---

**层A：Markdown 可读层（每个决策必须有）**

```markdown
### [设计决策名称]（优先级：P0/P1/P2）

**约束 Resolution 来源：**
- 解析自：[constraint_A] [关系类型] [constraint_B]
- Resolution 摘要：[一句话：为什么是这个方案，牺牲了什么]

**核心来源三元组：**
- Core KG：`[具体三元组] [来源论文]`
- Context KG：`[具体约束节点] [H/M/L] [Must/Tradeoff/Nice]`
- 推断节点（如有）：`[推断节点名称] ← [推断依据]`

**推理链（constraint-driven，4步）：**
1. **约束冲突识别**：[A主张X，B要求Y，两者在场景Z下的具体矛盾]
2. **优先级判断**：[哪个约束不可牺牲，理由] vs [哪个可以降级，代价]
3. **Resolution**：[具体的取舍决定] — 排除了[替代方案]，原因[...]
4. **设计实现**：[从 Resolution 推导出的具体、可编码的功能设计]

**教学策略：** [策略名称]（见 references/tpack-guide.md）
**TPACK区域：** [T/P/C/TP/PK/CK/TPK] — [一句话说明为什么是这个区域]

**产物类型：** [ui_feature / prompt_segment / workflow_step]
**具体内容：** [1-3句话描述这个设计决策的最终形态]

**跨决策影响：**（如无，省略此项）
- 影响 [decision_XXX]：[具体影响方式]
- 被 [decision_YYY] 影响：[具体影响方式]

**降级方案：**
- 风险：[这个设计可能失败的最主要原因]
- 降级：[如果主方案不可行，退而求其次的简化版]
- 降级代价：[退回到降级方案会牺牲什么教学效果]
```

---

**层B：JSON 机器层（P0 和 P1 必须有）**

```json
{
  "id": "decision_XXX",
  "name": "设计决策名称",
  "priority": "P0",
  "constraint_resolution": {
    "resolved_conflict": ["constraint_A_id", "constraint_B_id"],
    "relationship_type": "conflict | tradeoff | amplify",
    "resolution_rationale": "为什么是这个方案而不是别的，一句话",
    "sacrifice": "明确牺牲了什么",
    "alternative_considered": "被排除的方案，一句话"
  },
  "derived_from": {
    "core_kg": ["三元组描述 [来源论文]"],
    "context_kg": ["约束节点描述 [H/M/L][Must/Tradeoff/Nice]"],
    "inferred": ["推断节点描述（如有）"]
  },
  "reasoning_chain": [
    "step1: 约束冲突的具体描述",
    "step2: 优先级判断",
    "step3: Resolution",
    "step4: 设计实现"
  ],
  "pedagogy": {
    "strategy": "教学策略名称",
    "tpack_zone": "TPK",
    "tpack_rationale": "为什么是这个TPACK区域"
  },
  "artifact": {
    "type": "ui_feature | prompt_segment | workflow_step",
    "content": "具体设计描述",
    "priority": "P0"
  },
  "cross_decision_edges": [
    {
      "target": "decision_YYY",
      "direction": "influences",
      "description": "具体影响方式"
    }
  ],
  "fallback": {
    "risk": "最主要失败风险",
    "simplified": "降级实现方案",
    "pedagogical_cost": "降级会损失什么教学效果"
  }
}
```

---

**层C：Canvas 图节点数据（P0 完整版，P1 简化版）**

P0 完整版（4个节点 + 对应边）：
```json
{
  "canvas_nodes": [
    {
      "id": "n_c_A",
      "label": "约束A简称",
      "type": "constraint",
      "constraint_type": "pedagogical | technical | contextual",
      "color": "#fecaca | #bfdbfe | #fef08a",
      "x_hint": 100, "y_hint": [Y位置]
    },
    {
      "id": "n_c_B",
      "label": "约束B简称",
      "type": "constraint",
      "constraint_type": "...",
      "x_hint": 100, "y_hint": [Y位置+150]
    },
    {
      "id": "n_r_XXX",
      "label": "Resolution简称\n（牺牲: ...）",
      "type": "resolution",
      "color": "#e9d5ff",
      "x_hint": 350, "y_hint": [中间Y]
    },
    {
      "id": "n_d_XXX",
      "label": "决策名称",
      "type": "decision",
      "tpack_zone": "TPK",
      "priority": "P0",
      "color": "#c084fc",
      "x_hint": 800, "y_hint": [Y位置]
    }
  ],
  "canvas_edges": [
    {"from": "n_c_A", "to": "n_r_XXX", "label": "conflict/tradeoff", "type": "constraint_conflict", "color": "#ef4444", "style": "dashed"},
    {"from": "n_c_B", "to": "n_r_XXX", "label": "conflict/tradeoff", "type": "constraint_conflict", "color": "#ef4444", "style": "dashed"},
    {"from": "n_r_XXX", "to": "n_d_XXX", "label": "drives", "type": "resolution_to_decision", "color": "#8b5cf6", "style": "solid"}
  ]
}
```

P1 简化版（仅决策节点 + 最重要的一条来源边）：
```json
{
  "canvas_nodes": [
    {"id": "n_d_XXX", "label": "决策名称", "type": "decision", "tpack_zone": "TP", "priority": "P1", "x_hint": 800, "y_hint": [Y位置]}
  ],
  "canvas_edges": [
    {"from": "n_r_主要Resolution", "to": "n_d_XXX", "label": "drives", "type": "resolution_to_decision"}
  ]
}
```

---

### 阶段3：跨决策连接网络

所有决策节点完成后，执行跨决策分析：

```
FOR 每对决策 (A, B)：
  IF A的设计会影响B的实现或体验：
    记录跨决策边 A → B，描述影响方式
  IF 存在顺序依赖（A必须先于B）：
    记录依赖边 A → B，标注"依赖"
```

输出格式：
```
## 跨决策连接网络
[decision_A] → 影响 → [decision_B]：[影响方式]
[decision_C] → 依赖 → [decision_D]：[依赖原因]
```

在 Canvas JSON 中，这些边标注为：
```json
{"from": "n_d_A", "to": "n_d_B", "label": "影响描述", "type": "cross_decision", "color": "#f97316", "style": "dashed"}
```

---

### 阶段4：汇总 Canvas JSON

把所有决策的 Canvas 节点数据合并为一个完整的 Canvas JSON：

```json
{
  "meta": {
    "papers": ["论文A", "论文B"],
    "scenario": "场景摘要",
    "generated_at": "ISO时间戳",
    "node_count": N,
    "edge_count": M,
    "decision_count": {"P0": X, "P1": Y, "P2": Z}
  },
  "layout": {
    "columns": {
      "constraint":  {"x_range": [50, 200],   "color_by": "constraint_type"},
      "resolution":  {"x_range": [280, 420],  "color": "#e9d5ff"},
      "source_kg":   {"x_range": [500, 650],  "color_by": "kg_type"},
      "decision":    {"x_range": [780, 950],  "color_by": "tpack_zone"},
      "artifact":    {"x_range": [1080, 1250],"color": "#bbf7d0"}
    },
    "y_spacing": 160,
    "note": "Y 坐标按决策优先级排列：P0从上方开始，P1在中间，P2在下方"
  },
  "color_legend": {
    "tpack": {"T": "#93c5fd", "P": "#86efac", "C": "#fcd34d", "TP": "#a5b4fc", "PK": "#6ee7b7", "CK": "#fda4af", "TPK": "#c084fc"},
    "constraint_type": {"pedagogical": "#fecaca", "technical": "#bfdbfe", "contextual": "#fef08a"},
    "edge_type": {"constraint_conflict": "#ef4444", "resolution_to_decision": "#8b5cf6", "cross_decision": "#f97316", "core_support": "#3b82f6", "inference": "#a855f7"}
  },
  "nodes": [],
  "edges": []
}
```

---

## 输出契约（Postconditions）

| 输出项 | 格式 | 保证 |
|---|---|---|
| 决策列表 | Markdown | 按 P0/P1/P2 分组，有数量统计 |
| 每个决策的层A | Markdown | 推理链4步完整，有降级方案 |
| 每个 P0/P1 的层B | JSON | 含 constraint_resolution + cross_decision_edges |
| 每个 P0 的层C | JSON | 含4个节点和3条边 |
| 跨决策连接网络 | Markdown + JSON 边 | — |
| 汇总 Canvas JSON | JSON | meta.node_count 准确 |

**✅ 完成行格式**：
```
✅ 第4层完成：P0 N个 / P1 M个 / P2 K个，共J个决策节点
              跨决策连接L条（影响N/依赖M）
              Canvas JSON：节点共X个，边共Y条
```

---

## 错误处理

| 错误情况 | 处理 |
|---|---|
| 某个 P0 决策找不到对应的 Constraint Resolution | 降级为 P1，标注`[缺少约束依据: 需要补充Constraint Layer分析]` |
| 某个设计想法无法溯源到任何 KG 节点 | 标注`[假设设计: 无论文/场景依据，存在幻觉风险]`，不允许进入产物导出 |
| TPACK 区域判断不确定 | 标注两个候选区域，注明`[待确认]`，给出判断依据 |
| 跨决策影响形成循环依赖（A影响B，B影响A）| 标注循环，分析哪个影响更强，弱化方向标注为"轻度影响" |

---

## 质量自检

- [ ] 每个 P0 决策是否有 constraint_resolution 字段？
- [ ] 推理链是否真的是4步（不是2步压缩成4步）？
- [ ] alternative_considered 是否在每个 Resolution 中？
- [ ] 降级方案是否包含 pedagogical_cost？
- [ ] 汇总 Canvas JSON 的 node_count 是否与实际节点数匹配？
- [ ] 是否存在没有溯源的"孤立设计"？
