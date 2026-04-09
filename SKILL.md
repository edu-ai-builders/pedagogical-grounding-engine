---
name: pedagogical-grounding-engine
description: >
  将一篇或多篇学术论文，通过"教学法约束推理"转化为有理论根基的产品设计产物。
  核心是构建"学术机制 → 约束交互 → 可解释推理链"的中间层，
  再导出多种格式的设计产物。

  本skill是一个 Pedagogical Grounding Engine（教学法落地引擎）：
  它不是论文摘要工具，而是把教学法意图（pedagogical intent）编译为
  可执行的系统设计结构。PRD、System Prompt、Workflow 都是下游导出格式，
  核心产物是 Traceability Map——每个设计决策都有明确的教学依据、
  认知机制和可追溯的设计理由。

  当用户有以下意图时触发本skill：
  - "把这篇/这几篇论文做成产品/app/提示词/工作流"
  - "paper to PRD / paper to prompt / paper to product"
  - "pedagogical grounding / 教学法落地"
  - "我想基于学习科学设计这个功能/系统提示词"
  - "从论文提取知识图谱/三元组/TPACK分析"
  - "我有几篇论文，帮我提炼成一个设计方案"
  - "帮我把教学法嵌入到产品/提示词中"
  - "生成 traceability 图" / "把这篇论文可视化"
  - "我有一个教学场景/教育想法，帮我设计成系统"（不一定有论文）

  支持 Commands 模式：流水线跑完之后，用户可以随时用 /命令 召唤特定输出，
  无需重跑流水线。命令列表见 ## Commands 章节。
---

# Pedagogical Grounding Engine

## 核心定位

> **把教学法意图（pedagogical intent）编译为可执行的系统设计结构。**

这个 skill 是一个编译器（compiler），不是摘要工具，不是 feature 生成器。

它有四个核心职责，按执行顺序排列：

| 职责 | 做什么 | 对应流水线层 |
|---|---|---|
| **Extract（抽取）** | 从论文/场景中识别学习目标、认知挑战、隐含的教学法、关键构念 | 第1-2层 |
| **Ground（落地）** | 把抽象理论转化为：认知动作（对比/分解/模拟）、知识结构（图/序列/层级）、学习流程 | 第3层 |
| **Map（映射）** | 把落地后的认知结构映射到：图 Schema、UI 组件、功能模块、系统行为 | 第4层 |
| **Justify（溯源）** | 为每个设计决策输出：为什么存在、链接到哪条理论、去掉会损失什么 | 第4-5层 |

**产物优先级**：Traceability Map（核心产物）→ System Prompt → PRD → Workflow
PRD 是下游导出格式，不是目的本身。

**输入不限于论文**：可以是论文文本、教学场景描述、用户痛点、或研究者的想法。
有论文时，论文提供理论依据；没有论文时，场景本身是出发点，系统帮助找到认知科学依据。

---

## 六层架构：四职责 × 五层流水线

四个职责是**内部推理逻辑**，五层是**执行内容**。两者不是平行关系，而是嵌套关系：

```
职责维度（Why & How）          执行维度（What）
──────────────────────────────────────────────────────────────
                         输入：论文（1篇或多篇）+ 场景描述
                           [也可以只有场景描述，没有论文]
                                      │
┌─ EXTRACT ─────────────────┐         ▼
│ 识别：学习目标             │  [第1层] Core KG
│       认知挑战             │  ──────────────────────────────
│       关键构念             │  阶段1-3：论文类型判断 + 元数据 + 构念识别
│       隐含教学法           │  阶段4：三元组提取（研究表征）
└───────────────────────────┘  阶段4b：Representation Translation ← 跨语境迁移
                                │  · 研究语境 → 执行语境的 translation & transformation
                                │  · 不是填字段，是重构表征语言
                                │  · 显式化 translation_tension（边界条件变化）
                                │  · 输出：cognitive_mechanism / activation_condition /
                                │          failure_signature / execution_form
                                │  · 冲突 triple 输出双迁移路径，不裁决
                                阶段5：多篇合并（冲突保留，迁移后 triple 参与合并）
                                      │
┌─ EXTRACT (cont.) ─────────┐         ▼
│ 识别：场景约束             │  [第2层] Context KG
│       用户特征             │  ──────────────────────────────
│       情境限制             │  从场景描述提取：
│       成功标准             │  · 六维度结构化约束
└───────────────────────────┘  · 每条约束标注 H/M/L + Must/Tradeoff/Nice
                                · 内部冲突预警
                                      │
┌─ GROUND ──────────────────┐         ▼
│ 转化：抽象理论             │  [第3层] KG 融合 + Constraint Layer
│    → 认知动作              │  ──────────────────────────────
│    → 知识结构              │  · 消费已迁移的 triple：
│    → 学习流程              │    activation_condition → 直接对应检测
│                            │    failure_signature → 设计张力/障碍识别线索
│ ← 这是核心壁垒            │  · 约束节点提取 + 约束间关系 + Resolution
└───────────────────────────┘  · 推断节点生成
                                      │
┌─ MAP + JUSTIFY ───────────┐         ▼
│ 映射：认知结构             │  [第4层] Traceability Engine
│    → 图 Schema             │  ──────────────────────────────
│    → UI 组件               │  · Pedagogical Grounding Checklist 门控（PG1-PG6）
│    → 功能模块              │  · 每个决策 = constraint-driven 推理链（4步）
│    → 系统行为              │  · 三层输出：Markdown + JSON + Canvas节点
│                            │  · 跨决策连接网络
│ 溯源：为什么存在           │  · pg_checklist 字段嵌入 JSON
│       链接到哪条理论       │
│       去掉会损失什么       │
└───────────────────────────┘
                                      │
                                      ▼
                             [第5层] 产物导出（Commands 系统）
                             ──────────────────────────────────
                             产物优先级（反映四职责的输出层次）：
                             · /model    → Pedagogical Model（Extract+Ground的宏观产物）
                             · /trace    → Traceability Map（Map+Justify的核心产物）
                             · /pgcheck  → PG Checklist 报告（Justify的验证产物）
                             · /render   → Canvas 可视化
                             · /prompt   → System Prompt
                             · /prd      → PRD + TASKS + DESIGN
                             · /workflow → Workflow 步骤
                             · /kg       → KG 原始图数据
                             · /why      → 单功能溯源
                             · /diff     → 多论文对比
                             · /export   → 全部产物
```

**关键原则**：PRD 是第8个命令，不是第1个。`/model` 和 `/trace` 在前，
因为产物的优先级反映的是"这个系统在做什么"——编译教学法，而不是生成产品规格。

---

## 第1层 — Core KG：论文本体知识图谱

### 目的
提取**与场景无关**的、可复用的知识机制。换了场景不会变。

### 多篇论文的三种模式（默认模式3）

| 模式 | 描述 | 适用 |
|---|---|---|
| 模式1 | 分别提取，用户手动指定哪些构念进入合并 | 用户对论文非常熟悉 |
| 模式2 | 自动合并，冲突标注后**暂停**等用户裁决 | 中等熟悉度，想参与决策 |
| **模式3（默认）** | 分别提取，自动合并，冲突**保留不裁决** | 快速推进，让张力在推理层浮现 |

合并规则（模式3）：
- **互补节点** → 直接合并，保留双方来源标注 `[论文A]` `[论文B]`
- **同义节点** → 合并，标注 `[合并自A+B]`
- **冲突节点** → 双方保留，标注 `[冲突: A主张X, B主张Y]`，**不裁决，留给第3层**

### 每篇论文提取内容

**元数据（小表）：**

| 字段 | 内容 |
|---|---|
| 标题 | |
| 研究类型 | 方法论 / 实验+原型 / 技术方案 / **元分析→停止该篇** |
| 核心机制 | 1-2句话 |
| 目标人群 | |

**Core KG 三元组（至少20条/篇）：**

格式：`**主体** → 谓词 → **客体** [来源]`

覆盖：陈述性≥7 / 程序性≥8 / 产品implied≥5（标 `[F]`）

谓词统一使用词汇表（见 `references/kg-vocabulary.md`）。

**输出格式：**
```
## Core KG — [论文简称]
核心构念：...
三元组：（编号列表）

## Core KG — 合并结果（多篇时）
互补 / 同义 / 冲突节点各自列出
```

✅ 第1层完成：[每篇X条，合并后Y条，冲突Z个]

---

## 第2层 — Context KG：场景约束图谱

### 目的
将"在什么情境下用"转化为结构化约束节点。与论文内容无关。

### 场景输入（预设模板 + 自由描述双轨）

用户可以自由描述，也可以直接填模板，模型自动映射到六维度。
**六维度必须全部覆盖**，缺少任何一维主动追问：

```
目标用户：（谁在用？年龄/角色/技术水平/学习背景）
学习情境：（课堂/在线/异步/混合/研讨/自学）
核心痛点：（现在最大的问题是什么？）
成功标准：（什么叫"这个产品有效"？）
约束条件：（技术限制/时间/资源/平台）
已有设计假设：（如果有的话）
```

### Context KG 三元组（10-15条）

每个节点同时标注**优先级**（High/Medium/Low）和**可牺牲性**（Must/Tradeoff/Nice）：

```
格式：**[节点]** → 谓词 → **[节点]** [优先级: H/M/L] [可牺牲: Must/Tradeoff/Nice]
示例：**沉默型学生** → 需要 → **低压力入场机制** [H] [Must]
     **移动端兼容** → 要求 → **响应式设计** [L] [Nice]
```

这两个标注是 Constraint Layer 做权衡分析的输入依据。

✅ 第2层完成：[X条约束节点，六维度覆盖]

---

## 第3层 — KG 融合 + Constraint Layer

这是整个流水线的**约束显式化层**。它同时做两件事：
1. 识别 Core KG 和 Context KG 的对齐/张力/障碍（融合操作）
2. 把隐含的约束关系**提取出来并命名**（Constraint Layer）

---

### 3a. KG 融合操作（四种）

**操作1：直接对应**
`[论文机制A] ←直接对应→ [场景需求X]`

**操作2：间接激活（生成推断节点）**
`[论文机制B] ←间接激活（需要中间设计M）→ [场景需求Y]`
生成推断节点 M，标注 `[推断] ← 由 [CoreKG节点] + [ContextKG节点] 推导`

**操作3：设计张力**
`[张力: A机制 vs B机制，场景约束C使选择更复杂]`
不裁决，带入 Constraint Layer 处理。

**操作4：实现障碍**
`[障碍: 论文机制C 受限于 场景约束Z]`
必须在 Constraint Layer 给出 Resolution。

---

### 3b. Constraint Layer（🔥新增核心）

> **目的：把"解释结果"升级为"约束驱动结果"——解释为什么不是别的feature。**

#### Constraint 节点提取

从 Core KG 冲突节点 + Context KG 约束节点 + 融合操作结果中，提取出**命名的约束**：

```json
{
  "id": "constraint_001",
  "type": "pedagogical | technical | contextual",
  "source": "论文A / 场景 / 推断",
  "claim": "等距发言权对协作学习是必要的",
  "priority": "High",
  "negotiable": false
}
```

约束类型：
- `pedagogical`：来自论文的教学法主张（不可随意放弃）
- `technical`：来自场景的技术限制（有时可以降级）
- `contextual`：来自用户/情境的需求（有优先级，部分可牺牲）

#### 约束间关系（🔥关键）

这是让系统从"解释结果"变成"解释为什么不是别的结果"的核心结构：

```json
{
  "constraint_a": "constraint_001（等距发言权）",
  "constraint_b": "constraint_002（低认知负荷）",
  "relationship": "conflict | tradeoff | amplify | independent",
  "description": "实时显示所有人发言比例会增加界面认知负荷，
                  但不显示则无法支持公平参与目标",
  "resolution": {
    "decision": "用极简点状仪表盘而非数字表格",
    "rationale": "在保留公平参与可见性的前提下，最小化界面复杂度",
    "sacrifice": "牺牲精确数值，保留方向性感知",
    "resulting_decision_id": "decision_001"
  }
}
```

关系类型：
- `conflict`：两个约束直接矛盾，必须做出取舍
- `tradeoff`：两个约束可以同时部分满足，但有张力
- `amplify`：两个约束相互强化，合力指向同一设计方向
- `independent`：两个约束互不影响，可以独立处理

**Resolution 是 Traceability 节点的前置输入**：每个有 Resolution 的约束关系，直接对应第4层的一个设计决策。

#### 输出格式

```
## Constraint Layer

约束节点（N个）：
[编号列表，含type/priority/negotiable]

约束间关系（M组）：
1. [constraint_A] ← conflict → [constraint_B]
   解析：[description]
   Resolution → [resulting_decision简称]

2. [constraint_A] ← amplify → [constraint_C]
   合力指向：[design direction]
```

✅ 第3层完成：[对齐N / 张力N / 障碍N / 约束节点N / 约束关系M（冲突X/权衡Y/强化Z）]

---

## 第4层 — Traceability Engine：核心推理层

### 层定位：Map + Justify

这一层实现两个核心职责：
- **Map**：把第3层的约束 Resolution 映射为具体的认知结构 → 系统行为
- **Justify**：为每个设计决策生成完整的溯源链（理论依据 + 认知机制 + 去掉会损失什么）

### ⛩ Pedagogical Grounding Checklist（门控，每个 P0 决策必须通过）

在生成任何 P0 Traceability 节点之前，逐项检查：

| # | 检查项 | 通过标准 | 数据来源 | 失败处理 |
|---|---|---|---|---|
| PG1 | **学习目标明确** | 这个设计决策服务于一个可陈述的学习目标 | Context KG 成功标准节点 | 降级为 P1，标注`[学习目标未明确]` |
| PG2 | **认知挑战识别** | 识别了用户在这个环节会遭遇的认知困难 | **Enriched triple 的 `learner_precondition` + `likely_failure_mode`** | 补充认知困难分析，或降级 |
| PG3 | **认知动作定义** | 设计触发的是哪种认知动作（从词汇表选择）| Core KG mechanism 字段 | 必须从词汇表中选择，不允许模糊描述 |
| PG4 | **信息结构合理** | 信息以适合该认知动作的结构呈现 | Enriched triple 的 `implementation_form` | 结构与认知动作必须匹配 |
| PG5 | **教学策略命名** | 能说出这个设计体现了哪种教学策略 | tpack-guide.md | 不能用"辅助学习"这类模糊词 |
| PG6 | **Traceability 完整** | 能回答"去掉这个设计，学习过程会损失什么" | **Enriched triple 的 `observable_indicator` 和 `likely_failure_mode`** | 这个问题答不出来 → 降级 P2 |

**认知动作词汇表**（PG3 必须从此选择）：
`对比 / 分解 / 模拟 / 反思 / 检索 / 类比 / 迁移 / 归纳 / 评估 / 构建`

**检查结果处理**：
- 全部通过 → 生成完整 P0 Traceability 节点（含三层输出）
- 1-2项失败 → 降级为 P1，标注失败项，给出补救建议
- 3项以上失败 → 降级为 P2，或标注`[教学依据不足，建议寻找相关论文]`
- **没有 PG Checklist 结果的设计决策不允许成为 P0**

---

### 两个核心机制

### Traceability 节点结构（三层）

**层A：Markdown 可读层**

```
### [设计决策名称]（优先级：P0/P1/P2）

**约束来源：**
- 解析自约束关系：[constraint_A] conflict [constraint_B]
- Resolution：[一句话说为什么是这个设计而不是别的]

**Core KG 三元组来源：**
- [具体三元组，含来源论文]

**Context KG 约束节点：**
- [具体约束节点，含优先级/可牺牲性]

**推理链（constraint-driven）：**
1. 存在约束冲突：[A主张X，B要求Y，两者在场景Z下矛盾]
2. 约束优先级判断：[哪个不可牺牲，哪个可以降级]
3. Resolution：[具体的取舍决定，以及为什么]
4. 设计实现：[从 Resolution 推导出的具体功能设计]

**教学策略：** [名称]（见 references/tpack-guide.md）
**TPACK区域：** [T/P/C/TP/PK/CK/TPK]

**Pedagogical Grounding Checklist：**
- PG1 学习目标：[这个决策服务于什么学习目标]
- PG2 认知挑战：[用户在此环节遭遇的认知困难是什么]
- PG3 认知动作：[对比 / 分解 / 模拟 / 反思 / 检索 / 类比 / 迁移 / 归纳 / 评估 / 构建]
- PG4 信息结构：[图 / 序列 / 层级 / 对比 / 网络]
- PG5 教学策略：[具体策略名称，不允许模糊描述]
- PG6 去掉代价：[去掉这个设计，学习/认知过程会损失什么]
- 通过状态：[✅ 全部通过 / ⚠️ N项失败（降级为P1）]

**产物类型：** [ui_feature / prompt_segment / workflow_step]
**具体内容：** [描述]

**跨决策影响：**
- 本决策影响：[decision_XXX]（影响方式：[描述]）
- 本决策被影响：[decision_YYY]（影响方式：[描述]）

**降级方案：**（如存在实现障碍）
- 风险：[描述]
- 降级：[简化实现]
```

**层B：JSON 机器层**

```json
{
  "id": "decision_001",
  "name": "设计决策名称",
  "priority": "P0",
  "constraint_resolution": {
    "resolved_conflict": ["constraint_001", "constraint_002"],
    "resolution_rationale": "为什么是这个方案而不是别的",
    "sacrifice": "牺牲了什么"
  },
  "derived_from": {
    "core_kg": ["三元组描述 [来源论文]"],
    "context_kg": ["约束节点描述"],
    "inferred": ["推断节点（如有）"]
  },
  "reasoning_chain": [
    "约束冲突描述",
    "优先级判断",
    "Resolution",
    "设计实现"
  ],
  "pedagogy": {
    "strategy": "教学策略名称",
    "tpack_zone": "TPK"
  },
  "pg_checklist": {
    "PG1_learning_objective": "这个决策服务的具体学习目标",
    "PG2_cognitive_challenge": "用户在此环节遭遇的认知困难",
    "PG3_cognitive_action": "对比 | 分解 | 模拟 | 反思 | 检索 | 类比 | 迁移 | 归纳 | 评估 | 构建",
    "PG4_info_structure": "图 | 序列 | 层级 | 对比 | 网络",
    "PG5_pedagogy_strategy": "教学策略具体名称",
    "PG6_removal_cost": "去掉这个设计，学习/认知过程损失什么",
    "passed": true
  },
  "artifact": {
    "type": "ui_feature | prompt_segment | workflow_step",
    "content": "具体描述",
    "priority": "P0"
  },
  "cross_decision_edges": [
    {
      "target": "decision_002",
      "direction": "influences",
      "description": "参与度仪表盘增加认知负荷，影响Prompt设计的简洁性要求"
    }
  ],
  "fallback": {
    "risk": "风险描述",
    "simplified": "降级方案"
  }
}
```

**层C：Canvas 图节点数据**

```json
{
  "canvas_nodes": [
    {"id": "n_constraint_001", "label": "约束名称", "type": "constraint",
     "constraint_type": "pedagogical", "x_hint": 50, "y_hint": 200},
    {"id": "n_decision_001", "label": "设计决策名称", "type": "decision",
     "tpack_zone": "TPK", "priority": "P0", "x_hint": 700, "y_hint": 200},
    {"id": "n_resolution_001", "label": "Resolution简称", "type": "resolution",
     "x_hint": 400, "y_hint": 200}
  ],
  "canvas_edges": [
    {"from": "n_constraint_001", "to": "n_resolution_001",
     "label": "conflict", "type": "constraint_conflict"},
    {"from": "n_resolution_001", "to": "n_decision_001",
     "label": "drives", "type": "resolution_to_decision"},
    {"from": "n_decision_001", "to": "n_decision_002",
     "label": "影响认知负荷", "type": "cross_decision"}
  ]
}
```

### Canvas 全局布局（五列）

```
列1（x≈50-250）：   Constraint 节点（按类型着色：教学法/技术/情境）
列2（x≈300-400）：  Resolution 节点（约束解析结果）
列3（x≈500-700）：  Core KG + Context KG 节点（来源层）
列4（x≈800-1000）： Decision 节点（按TPACK着色）
列5（x≈1100+）：    Artifact 节点（产物层）

特殊边类型：
- constraint_conflict（红色虚线）：约束冲突关系
- constraint_amplify（绿色实线）：约束强化关系
- resolution_to_decision（紫色实线）：Resolution驱动决策
- cross_decision（橙色虚线，双向箭头）：决策间影响
```

### TPACK着色（Decision节点）
T:`#93c5fd` P:`#86efac` C:`#fcd34d` TP:`#a5b4fc` PK:`#6ee7b7` CK:`#fda4af` TPK:`#c084fc`

### Constraint类型着色（Constraint节点）
pedagogical:`#fecaca` technical:`#bfdbfe` contextual:`#fef08a`

### 数量要求
- P0：完整三层（Markdown + JSON + Canvas节点数据）
- P1：Markdown + JSON，Canvas简化
- P2：仅 Markdown

**⚠️ 关键约束：没有 constraint_resolution 字段的 P0 决策不允许进入第5层。**

✅ 第4层完成：[P0 X个 / P1 Y个 / P2 Z个，跨决策连接N条，Canvas节点共M个]

---

## 第5层 — 产物导出 + Commands

流水线完成后，**输出 Commands 菜单**，用户可以随时召唤任何产物，无需重跑流水线：

```
✅ 流水线完成。可用命令：

/model     — 🆕 Pedagogical Model（学习目标+认知结构+教学法摘要，核心输出）
/trace     — 输出完整 Traceability Map（Markdown 汇总表 + Canvas JSON）
/render    — 直接渲染 Canvas 可视化（React Flow 组件）
/kg        — 输出 KG 图数据（Core KG + Context KG + Constraint Layer 合并）
/prompt    — 输出 System Prompt（含来源标注）
/prd       — 输出 PRD + TASKS + DESIGN 三份规格文件
/workflow  — 输出 Workflow 步骤序列
/constraint — 输出 Constraint Layer 详细报告（约束节点 + 关系 + Resolution）
/why [功能名] — 解释某个具体功能"为什么是这个设计而不是别的"
/diff [论文A] [论文B] — 输出两篇论文的构念冲突与融合分析
/pgcheck   — 🆕 输出所有决策的 Pedagogical Grounding Checklist 完整报告
/export    — 输出所有产物（完整包）
```

---

### /model — Pedagogical Model（🆕 核心产物，默认首选）

**这是整个流水线最上游的产物**，不依赖 PRD 或任何技术实现。
它回答的问题是：**"这个系统的教学法骨架是什么？"**

输出结构：

```markdown
## Pedagogical Model

### 学习目标层（Learning Objectives）
- 学习者通过使用这个系统，应该能够...（动词+认知层次）
- 例："学习者能够对比两种教学策略在不同场景下的适用性"

### 认知挑战层（Cognitive Challenges）
- 这个学习领域中，学习者典型遭遇的认知困难：
  - [挑战1]：来自 Core KG / Context KG 分析
  - [挑战2]：...

### 核心认知动作图谱（Cognitive Action Map）
| 认知动作 | 触发场景 | 对应设计决策 | 教学策略 |
|---|---|---|---|
| 对比 | 用户看到两个基准报告 | decision_002 | 同伴比较学习 |
| 反思 | 用户完成评分后查看分布 | decision_003 | 元认知支持 |

### 信息结构选择（Information Architecture）
- 选择了 [图/序列/层级/对比/网络] 结构
- 理由：[来自 PG Checklist PG4 的分析]

### 教学法设计原则（Pedagogical Design Principles）
1. [原则1]：[一句话，来自论文/认知科学]
2. [原则2]：...

### 与 Traceability Map 的关系
Pedagogical Model 是"为什么要这样设计"的宏观叙事，
Traceability Map 是"每个决策的微观溯源"。
两者互为补充，前者是后者的上游语境。
```

---

### /pgcheck — Pedagogical Grounding Checklist 报告（🆕）

输出所有决策的 PG Checklist 汇总，并给出总体评估：

```markdown
## Pedagogical Grounding Checklist 报告

### 汇总评分
| 决策 | PG1 | PG2 | PG3 | PG4 | PG5 | PG6 | 通过 | 优先级 |
|---|---|---|---|---|---|---|---|---|
| [decision名称] | ✅ | ✅ | ✅ | ⚠️ | ✅ | ✅ | 5/6 | P1↓ |

### 未通过项详情
[decision名称] — PG4 未通过：
  问题：信息以线性列表呈现，但认知动作是"对比"，对比需要并排结构
  建议：将两个选项改为左右对比卡片布局

### 总体教学法覆盖度
- 认知动作多样性：[用了哪几种认知动作]
- 教学策略覆盖：[用了哪几种教学策略]
- 高风险决策：[PG6 分析薄弱的决策，去掉后学习影响最大的]
```

---

### /trace — Traceability Map

**A1. Markdown 汇总表**（新增约束列）：
```
| 决策 | 约束冲突 | Resolution | Core KG来源 | 教学策略 | TPACK | 优先级 | 影响决策 |
|---|---|---|---|---|---|---|---|
```

**A2. 汇总 Canvas JSON**（含 Constraint 节点和 Resolution 节点）：

```json
{
  "meta": {
    "papers": [],
    "scenario": "",
    "generated_at": "",
    "node_count": 0,
    "edge_count": 0
  },
  "layout": {
    "columns": {
      "constraint":  {"x_range": [50, 250],  "color_by": "constraint_type"},
      "resolution":  {"x_range": [300, 400], "color": "#e9d5ff"},
      "source_kg":   {"x_range": [500, 700], "color_by": "kg_type"},
      "decision":    {"x_range": [800, 1000],"color_by": "tpack_zone"},
      "artifact":    {"x_range": [1100, 1300],"color": "#bbf7d0"}
    }
  },
  "nodes": [],
  "edges": []
}
```

---

### /render — 直接渲染 Canvas

执行 `/render` 时，生成一个**完整的 React Flow 组件**，可以直接在 Claude artifact 中预览：

```tsx
// 生成一个自包含的 React 组件
// 包含所有节点数据、边数据、着色逻辑
// 用户可以在 artifact 中交互（拖拽、缩放、点击节点查看推理链）
```

节点点击行为：
- 点击 Decision 节点 → 展开该节点的完整推理链（Markdown 可读层）
- 点击 Constraint 节点 → 展开约束节点的 claim + resolution
- 点击 Artifact 节点 → 展开产物描述

---

### /kg — KG 图数据

输出三层 KG 的合并图数据（不含 Traceability 推理层，只是原始 KG）：

```json
{
  "core_kg": {"nodes": [], "edges": [], "papers": []},
  "context_kg": {"nodes": [], "edges": []},
  "constraint_layer": {"constraints": [], "relationships": []}
}
```

适合：把 KG 作为独立产物使用，或传入其他工具做进一步分析。

---

### /prompt — System Prompt

从所有 `artifact.type === "prompt_segment"` 节点中提取，组装完整 System Prompt。
**每条规则必须附来源标注**：`[来源: decision_XXX，Resolution: 为什么这条规则存在]`

---

### /prd — PRD + TASKS + DESIGN

从所有 `artifact.type === "ui_feature"` 节点生成三份规格文件。
每个功能附 Traceability 节点ID 和 constraint_resolution 摘要。
详细模板见 `references/spec-templates.md`。

技术栈（硬性约束）：Next.js 14 + TypeScript + Tailwind + shadcn/ui + lucide-react
存储：内存 Map / LLM：OpenAI API 直接 fetch / 禁止：auth、WebSocket、测试文件

---

### /workflow — Workflow

从所有 `artifact.type === "workflow_step"` 节点生成编号序列。
每步附：触发条件 / 执行动作 / 预期输出 / 来源 `[decision_XXX]`

---

### /constraint — Constraint Layer 报告

完整输出第3层的 Constraint Layer 结果：
- 所有约束节点（含 type/priority/negotiable）
- 所有约束间关系（含 relationship/description/resolution）
- 约束分析摘要："哪些约束决定了整个设计方向"

---

### /why [功能名]

**专项查询命令**。用户可以追问任何一个功能"为什么是这个设计而不是别的"。

返回：
1. 这个决策的 constraint_resolution（牺牲了什么，为什么值得）
2. 被排除的替代方案（从约束关系中推导出来的"没走的路"）
3. 如果某个约束改变，这个决策会如何变化

---

### /diff [论文A] [论文B]

**多论文专项命令**。输出两篇论文之间的：
- 构念冲突清单（及其在第3层的处理结果）
- 互补关系（合力强化了哪些设计方向）
- 覆盖盲区（A有B没有，或反之）

---

## 输出规范

- 每层结束输出 `✅ 第X层完成：[关键数字]`
- Canvas JSON 用 ` ```json ` 包裹
- React 组件用 ` ```tsx ` 包裹
- 规格文件用带文件名注释的 ` ```md ` 包裹
- 流水线完成后必须输出 Commands 菜单
- 全文结尾输出 `🚀 下一步` 段落

---

## 质量自检

**第1层**：冲突是否保留？每篇≥20条三元组？是否识别了隐含的教学法构念？

**第2层**：六维度全覆盖？每个约束节点是否有 priority/negotiable 标注？

**第3层**：Constraint Layer 是否提取了命名约束？是否分析了约束间关系？Resolution 是否明确说明了"牺牲了什么"？

**第4层**：
- P0 决策是否通过了全部6项 PG Checklist？
- PG3（认知动作）是否来自规定词汇表，而非模糊描述？
- PG6（去掉代价）是否有实质性回答，而非"会影响用户体验"这类废话？
- 推理起点是否来自 Constraint Resolution？是否标注了跨决策连接？

**第5层**：Commands 菜单是否已输出？是否包含 /model 和 /pgcheck？Canvas JSON meta 的 node_count/edge_count 是否准确？

---

## 层级详细规范（layers/）

每层有独立的处理规范文件，包含输入契约、处理流程、输出契约、错误处理、质量自检。
当 SKILL.md 的描述不够详细时，读取对应层的文件：

- `layers/layer1-core-kg.md` — 第1层：论文类型判断 + 三元组提取 + 多篇合并
- `layers/layer2-context-kg.md` — 第2层：场景输入双轨处理 + 六维度覆盖检查
- `layers/layer3-kg-fusion.md` — 第3层：四种融合操作 + Constraint Layer 完整流程
- `layers/layer4-traceability.md` — 第4层：决策生成规则 + 三层输出格式 + Canvas JSON schema
- `layers/layer5-export.md` — 第5层：所有命令的生成规范 + 错误处理

## 参考文件（references/）

- `references/kg-vocabulary.md` — KG 谓词词汇表 + EdTech 三元组集群
- `references/tpack-guide.md` — TPACK 七区域 + 教学策略词汇表
- `references/spec-templates.md` — PRD/TASKS/DESIGN 模板 + traceability.json 格式
- `references/canvas-renderer-hint.md` — Canvas 渲染选型（React Flow / Cytoscape / Obsidian）

## 示例文件（examples/）

- `examples/cscl-example.md` — 完整执行示例：CSCL论文五层流水线
- `examples/output-trace.json` — /trace 命令的示例输出（Canvas JSON）
- `examples/output-prompt.md` — /prompt 命令的示例输出
- `examples/output-prd.md` — /prd 命令的示例输出（PRD.md）
