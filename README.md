# Pedagogical Grounding Engine

> **把每一个设计决策变成一个有学术溯源的推理结果。**

这个 skill 不是"把论文翻译成产品"，而是构建一套推理系统——让教学法机制成为产品设计的约束，让每一个 feature 的存在都有明确的理论依据，让"为什么不是别的设计"这个问题有清晰的答案。

---

## 流水线总览

```svg
<svg viewBox="0 0 900 620" xmlns="http://www.w3.org/2000/svg" font-family="monospace, sans-serif">

  <!-- 背景 -->
  <rect width="900" height="620" fill="#0f172a" rx="12"/>

  <!-- 标题 -->
  <text x="450" y="40" text-anchor="middle" fill="#94a3b8" font-size="13" letter-spacing="2">PEDAGOGICAL GROUNDING ENGINE — PIPELINE</text>

  <!-- 输入框 -->
  <rect x="320" y="60" width="260" height="52" rx="8" fill="#1e293b" stroke="#334155" stroke-width="1.5"/>
  <text x="450" y="81" text-anchor="middle" fill="#64748b" font-size="10">INPUT</text>
  <text x="450" y="98" text-anchor="middle" fill="#e2e8f0" font-size="12">论文（1篇或多篇）+ 场景描述</text>

  <!-- 箭头：输入→第1层 -->
  <line x1="450" y1="112" x2="450" y2="132" stroke="#475569" stroke-width="1.5" marker-end="url(#arrow)"/>

  <!-- 第1层 -->
  <rect x="260" y="132" width="380" height="64" rx="8" fill="#1e3a5f" stroke="#3b82f6" stroke-width="1.5"/>
  <text x="450" y="154" text-anchor="middle" fill="#93c5fd" font-size="11" font-weight="bold">第1层 Core KG</text>
  <text x="450" y="172" text-anchor="middle" fill="#7dd3fc" font-size="10">论文类型判断 → 构念提取 → 三元组（≥20条/篇）</text>
  <text x="450" y="187" text-anchor="middle" fill="#7dd3fc" font-size="10">多篇：分别提取 → 自动合并 → 冲突保留</text>

  <!-- 箭头：第1层→第2层 -->
  <line x1="450" y1="196" x2="450" y2="216" stroke="#475569" stroke-width="1.5" marker-end="url(#arrow)"/>

  <!-- 第2层 -->
  <rect x="260" y="216" width="380" height="64" rx="8" fill="#1e3f2e" stroke="#22c55e" stroke-width="1.5"/>
  <text x="450" y="238" text-anchor="middle" fill="#86efac" font-size="11" font-weight="bold">第2层 Context KG</text>
  <text x="450" y="256" text-anchor="middle" fill="#bbf7d0" font-size="10">六维度场景结构化（双轨输入）</text>
  <text x="450" y="271" text-anchor="middle" fill="#bbf7d0" font-size="10">每条约束节点含 H/M/L + Must/Tradeoff/Nice</text>

  <!-- 箭头：第2层→第3层 -->
  <line x1="450" y1="280" x2="450" y2="300" stroke="#475569" stroke-width="1.5" marker-end="url(#arrow)"/>

  <!-- 第3层（核心，最大） -->
  <rect x="200" y="300" width="500" height="90" rx="8" fill="#3b1f1f" stroke="#ef4444" stroke-width="2"/>
  <rect x="200" y="300" width="500" height="90" rx="8" fill="url(#redglow)" opacity="0.15"/>
  <text x="450" y="323" text-anchor="middle" fill="#fca5a5" font-size="11" font-weight="bold">第3层 KG 融合 + Constraint Layer  🔥</text>
  <text x="450" y="341" text-anchor="middle" fill="#fecaca" font-size="10">对齐识别 / 间接激活 / 设计张力 / 实现障碍</text>
  <text x="450" y="357" text-anchor="middle" fill="#fecaca" font-size="10">约束节点提取 → 约束间关系（冲突/权衡/强化）</text>
  <text x="450" y="373" text-anchor="middle" fill="#fca5a5" font-size="10" font-weight="bold">→ Resolution（为什么不是别的方案）</text>

  <!-- 箭头：第3层→第4层 -->
  <line x1="450" y1="390" x2="450" y2="410" stroke="#475569" stroke-width="1.5" marker-end="url(#arrow)"/>

  <!-- 第4层 -->
  <rect x="220" y="410" width="460" height="72" rx="8" fill="#2d1f3f" stroke="#a855f7" stroke-width="1.5"/>
  <text x="450" y="432" text-anchor="middle" fill="#d8b4fe" font-size="11" font-weight="bold">第4层 Traceability Engine</text>
  <text x="450" y="450" text-anchor="middle" fill="#e9d5ff" font-size="10">Constraint-driven 推理链（4步）</text>
  <text x="450" y="466" text-anchor="middle" fill="#e9d5ff" font-size="10">三层输出：Markdown + JSON + Canvas节点 | 跨决策连接</text>

  <!-- 箭头：第4层→第5层 -->
  <line x1="450" y1="482" x2="450" y2="502" stroke="#475569" stroke-width="1.5" marker-end="url(#arrow)"/>

  <!-- 第5层：命令系统 -->
  <rect x="160" y="502" width="580" height="90" rx="8" fill="#1a2e1a" stroke="#4ade80" stroke-width="1.5"/>
  <text x="450" y="524" text-anchor="middle" fill="#86efac" font-size="11" font-weight="bold">第5层 产物导出 — Commands 系统</text>

  <!-- 命令列表 -->
  <text x="190" y="548" fill="#4ade80" font-size="9">/trace</text>
  <text x="240" y="548" fill="#4ade80" font-size="9">/render</text>
  <text x="298" y="548" fill="#4ade80" font-size="9">/kg</text>
  <text x="330" y="548" fill="#4ade80" font-size="9">/prompt</text>
  <text x="388" y="548" fill="#4ade80" font-size="9">/prd</text>
  <text x="420" y="548" fill="#4ade80" font-size="9">/workflow</text>
  <text x="492" y="548" fill="#4ade80" font-size="9">/constraint</text>
  <text x="570" y="548" fill="#4ade80" font-size="9">/why</text>
  <text x="603" y="548" fill="#4ade80" font-size="9">/diff</text>
  <text x="636" y="548" fill="#4ade80" font-size="9">/export</text>

  <!-- 产物标签 -->
  <text x="190" y="565" fill="#64748b" font-size="8">溯源图</text>
  <text x="240" y="565" fill="#64748b" font-size="8">可视化</text>
  <text x="298" y="565" fill="#64748b" font-size="8">图数据</text>
  <text x="330" y="565" fill="#64748b" font-size="8">提示词</text>
  <text x="388" y="565" fill="#64748b" font-size="8">规格</text>
  <text x="420" y="565" fill="#64748b" font-size="8">工作流</text>
  <text x="492" y="565" fill="#64748b" font-size="8">约束报告</text>
  <text x="570" y="565" fill="#64748b" font-size="8">为何</text>
  <text x="603" y="565" fill="#64748b" font-size="8">对比</text>
  <text x="636" y="565" fill="#64748b" font-size="8">全部</text>

  <!-- 底部注释 -->
  <text x="450" y="608" text-anchor="middle" fill="#334155" font-size="10">每层有独立规范文件 (layers/)  ·  所有产物可追溯到论文三元组</text>

  <!-- 箭头 marker -->
  <defs>
    <marker id="arrow" markerWidth="8" markerHeight="8" refX="6" refY="3" orient="auto">
      <path d="M0,0 L0,6 L8,3 z" fill="#475569"/>
    </marker>
    <radialGradient id="redglow">
      <stop offset="0%" stop-color="#ef4444"/>
      <stop offset="100%" stop-color="transparent"/>
    </radialGradient>
  </defs>

</svg>
```

---

## Traceability Map 节点类型

流水线产出的 Canvas 图包含五类节点，从左到右排列：

```svg
<svg viewBox="0 0 800 120" xmlns="http://www.w3.org/2000/svg" font-family="monospace, sans-serif">
  <rect width="800" height="120" fill="#0f172a" rx="8"/>

  <!-- Constraint -->
  <rect x="30" y="30" width="120" height="50" rx="6" fill="#7f1d1d" stroke="#fecaca" stroke-width="1.5"/>
  <text x="90" y="52" text-anchor="middle" fill="#fecaca" font-size="10" font-weight="bold">Constraint</text>
  <text x="90" y="68" text-anchor="middle" fill="#fca5a5" font-size="9">教学法/技术/情境</text>
  <text x="90" y="100" text-anchor="middle" fill="#64748b" font-size="9">约束来源</text>

  <!-- Arrow -->
  <line x1="150" y1="55" x2="180" y2="55" stroke="#ef4444" stroke-width="1.5" stroke-dasharray="4,2" marker-end="url(#a2)"/>
  <text x="165" y="48" text-anchor="middle" fill="#ef4444" font-size="8">冲突</text>

  <!-- Resolution -->
  <rect x="180" y="30" width="110" height="50" rx="6" fill="#3b1f4f" stroke="#e9d5ff" stroke-width="1.5"/>
  <text x="235" y="52" text-anchor="middle" fill="#e9d5ff" font-size="10" font-weight="bold">Resolution</text>
  <text x="235" y="68" text-anchor="middle" fill="#d8b4fe" font-size="9">取舍方案</text>
  <text x="235" y="100" text-anchor="middle" fill="#64748b" font-size="9">约束解析</text>

  <!-- Arrow -->
  <line x1="290" y1="55" x2="320" y2="55" stroke="#8b5cf6" stroke-width="1.5" marker-end="url(#a2)"/>
  <text x="305" y="48" text-anchor="middle" fill="#8b5cf6" font-size="8">驱动</text>

  <!-- Source KG -->
  <rect x="320" y="30" width="110" height="50" rx="6" fill="#1e3a5f" stroke="#93c5fd" stroke-width="1.5"/>
  <text x="375" y="52" text-anchor="middle" fill="#93c5fd" font-size="10" font-weight="bold">Source KG</text>
  <text x="375" y="68" text-anchor="middle" fill="#7dd3fc" font-size="9">论文/场景三元组</text>
  <text x="375" y="100" text-anchor="middle" fill="#64748b" font-size="9">知识来源</text>

  <!-- Arrow -->
  <line x1="430" y1="55" x2="460" y2="55" stroke="#3b82f6" stroke-width="1.5" marker-end="url(#a2)"/>
  <text x="445" y="48" text-anchor="middle" fill="#3b82f6" font-size="8">支撑</text>

  <!-- Decision -->
  <rect x="460" y="25" width="120" height="60" rx="6" fill="#2d1f3f" stroke="#c084fc" stroke-width="2"/>
  <text x="520" y="48" text-anchor="middle" fill="#c084fc" font-size="10" font-weight="bold">Decision</text>
  <text x="520" y="63" text-anchor="middle" fill="#e9d5ff" font-size="9">TPK</text>
  <text x="520" y="78" text-anchor="middle" fill="#e9d5ff" font-size="9">P0</text>
  <text x="520" y="100" text-anchor="middle" fill="#64748b" font-size="9">推理结果（TPK=紫）</text>

  <!-- Arrow -->
  <line x1="580" y1="55" x2="610" y2="55" stroke="#22c55e" stroke-width="1.5" marker-end="url(#a2)"/>
  <text x="595" y="48" text-anchor="middle" fill="#22c55e" font-size="8">产出</text>

  <!-- Artifact -->
  <rect x="610" y="30" width="140" height="50" rx="6" fill="#1a2e1a" stroke="#86efac" stroke-width="1.5"/>
  <text x="680" y="52" text-anchor="middle" fill="#86efac" font-size="10" font-weight="bold">Artifact</text>
  <text x="680" y="68" text-anchor="middle" fill="#bbf7d0" font-size="9">UI / Prompt / Workflow</text>
  <text x="680" y="100" text-anchor="middle" fill="#64748b" font-size="9">最终产物</text>

  <defs>
    <marker id="a2" markerWidth="6" markerHeight="6" refX="5" refY="3" orient="auto">
      <path d="M0,0 L0,6 L6,3 z" fill="#475569"/>
    </marker>
  </defs>
</svg>
```

---

## 文件结构

```
skill-paper-to-prd/
├── SKILL.md                    # 主入口：五层流水线概览 + Commands
│
├── layers/                     # 每层的详细处理规范
│   ├── layer1-core-kg.md       # 论文类型判断、三元组提取、多篇合并
│   ├── layer2-context-kg.md    # 六维度场景结构化、双轨输入处理
│   ├── layer3-kg-fusion.md     # 四种融合操作 + Constraint Layer 完整流程
│   ├── layer4-traceability.md  # 决策生成规则、三层输出、Canvas JSON schema
│   └── layer5-export.md        # 所有命令的生成规范和错误处理
│
├── references/                 # 参考资料
│   ├── kg-vocabulary.md        # KG 谓词词汇表 + EdTech 常见三元组集群
│   ├── tpack-guide.md          # TPACK 七区域 + 教学策略词汇表
│   ├── spec-templates.md       # PRD/TASKS/DESIGN 完整模板
│   └── canvas-renderer-hint.md # Canvas 渲染工具选型（React Flow / Cytoscape / Obsidian）
│
└── examples/                   # 真实输出示例
    ├── cscl-example.md         # CSCL论文完整五层执行记录
    ├── output-trace.json       # /trace 命令输出：Canvas JSON（18节点/17边）
    ├── output-prompt.md        # /prompt 命令输出：完整 System Prompt
    └── output-prd.md           # /prd 命令输出：PRD + TASKS + DESIGN
```

---

## 快速开始

### 1. 单篇论文

```
用户：[粘贴论文内容]

场景：线上大学研讨课，6-8人小组，需要解决沉默学生参与问题，
     教师同时管理多个房间。无数据库，原型阶段。
```

Skill 自动执行五层流水线，完成后输出 Commands 菜单。

### 2. 多篇论文（推荐指定模式）

```
用户：以下是两篇论文，请用模式3合并（默认）：
     论文A：[内容]
     论文B：[内容]

场景：[场景描述]
```

### 3. 直接调用命令（流水线已完成后）

```
/trace          → 获取 Traceability Map
/render         → 获取可交互的 Canvas 图（React Flow 组件）
/why 私信nudge  → 解释为什么是90秒而不是60秒
/prd            → 获取三份工程规格文件
```

---

## 核心概念

### Constraint Layer（约束层）

这是整个系统的设计辩护力所在。它回答的不是"这个 feature 从哪来"，而是"为什么不是别的 feature"。

每对约束关系都有一个 **Resolution**，Resolution 必须包含：
- 具体的取舍决定（不是"做更好的平衡"，而是"用点状仪表盘而非数字表格"）
- 明确牺牲了什么（"牺牲精确数值"）
- 被排除的替代方案（"被排除：实时数字显示——原因：认知负荷过高"）

### Traceability 节点（推理节点）

每个设计决策都是一个 Traceability 节点，包含三层：

| 层 | 格式 | 用途 |
|---|---|---|
| 层A：Markdown 可读层 | 结构化文本 | 人工审阅、内容创作 |
| 层B：JSON 机器层 | 结构化 JSON | 代码生成、工具集成 |
| 层C：Canvas 图节点数据 | JSON nodes + edges | 可视化渲染 |

### Commands 系统

流水线不是一次性的。完成后，用户可以随时用命令召唤特定输出，无需重跑。

最有力的命令是 `/why [功能名]`——它会输出完整的反事实分析："如果把这个约束改掉，设计会变成什么样。"

---

## 设计原则（Harness Engineering）

本 skill 的处理规范遵循以下原则：

**输入契约**：每层明确声明前置条件（preconditions）。如果前一层的输出不满足要求，本层不执行，而是报告具体缺失。

**输出契约**：每层有明确的 postconditions，下一层可以依赖这些保证，无需重新检验。

**幂等性**：相同的输入（论文 + 场景）总是产出相同的结构，只是内容不同。

**错误隔离**：每层有独立的错误处理，单层失败不会导致整个流水线崩溃。元分析论文的停止只影响该篇，其他篇继续处理。

**可组合性**：命令系统的每个命令是独立的视图，从同一份流水线结果导出，不产生副作用。

---

## 示例输出

- [完整执行记录（CSCL论文）](examples/cscl-example.md)
- [/trace 输出：Canvas JSON](examples/output-trace.json)
- [/prompt 输出：System Prompt](examples/output-prompt.md)
- [/prd 输出：PRD + TASKS + DESIGN](examples/output-prd.md)
