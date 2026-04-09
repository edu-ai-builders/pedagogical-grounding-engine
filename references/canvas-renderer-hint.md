# Canvas 渲染工具选型与使用提示

Skill 输出的 Canvas JSON 是标准的 nodes + edges 图数据格式，可以传入多种渲染工具。

---

## 选项1：直接让 Claude 渲染（最快）

把 Canvas JSON 粘贴给 Claude，说：
> "把这个 JSON 渲染成一个交互式 React Flow 图，节点按 type 着色，边有标签"

Claude 会生成一个完整的 React 组件，可以直接在 artifact 中预览。

---

## 选项2：React Flow（推荐，代码可控）

```bash
npm install reactflow
```

```tsx
import ReactFlow, { Node, Edge } from 'reactflow'

// 把 Canvas JSON 的 nodes 映射为 ReactFlow Node 格式
const rfNodes: Node[] = canvasData.nodes.map(n => ({
  id: n.id,
  data: { label: n.label },
  position: { x: n.x_hint, y: n.y_hint },
  style: { background: TPACK_COLORS[n.tpack_zone] ?? NODE_TYPE_COLORS[n.type] }
}))

// 把 Canvas JSON 的 edges 映射为 ReactFlow Edge 格式
const rfEdges: Edge[] = canvasData.edges.map(e => ({
  id: `${e.from}-${e.to}`,
  source: e.from,
  target: e.to,
  label: e.label,
  animated: e.type === 'inference',
  style: { stroke: EDGE_COLORS[e.type] }
}))
```

TPACK 着色（与 SKILL.md 一致）：
```ts
const TPACK_COLORS = {
  T: '#93c5fd', P: '#86efac', C: '#fcd34d',
  TP: '#a5b4fc', PK: '#6ee7b7', CK: '#fda4af', TPK: '#c084fc'
}

const NODE_TYPE_COLORS = {
  core_kg: '#dbeafe', context_kg: '#fed7aa',
  inferred: '#e9d5ff', decision: '#f3f4f6', artifact: '#bbf7d0'
}

const EDGE_COLORS = {
  core_support: '#3b82f6', context_constraint: '#f97316',
  inference: '#8b5cf6', conflict: '#ef4444', artifact: '#22c55e'
}
```

---

## 选项3：Cytoscape.js（适合大图，性能更好）

```js
const cy = cytoscape({
  container: document.getElementById('cy'),
  elements: {
    nodes: canvasData.nodes.map(n => ({
      data: { id: n.id, label: n.label, type: n.type, tpack: n.tpack_zone }
    })),
    edges: canvasData.edges.map(e => ({
      data: { source: e.from, target: e.to, label: e.label, type: e.type }
    }))
  },
  layout: { name: 'preset' }  // 使用 x_hint/y_hint 作为初始位置
})
```

---

## 选项4：Obsidian Canvas（零代码，适合个人知识管理）

Canvas JSON 需要轻微转换：

```js
// Obsidian Canvas 格式
{
  "nodes": nodes.map(n => ({
    "id": n.id,
    "type": "text",
    "text": n.label,
    "x": n.x_hint,
    "y": n.y_hint,
    "width": 200,
    "height": 60,
    "color": TPACK_COLORS[n.tpack_zone] ?? NODE_TYPE_COLORS[n.type]
  })),
  "edges": edges.map(e => ({
    "id": `${e.from}-${e.to}`,
    "fromNode": e.from,
    "toNode": e.to,
    "label": e.label
  }))
}
```

保存为 `.canvas` 文件，放入 Obsidian Vault 即可打开。

---

## 选项5：D3-force（最灵活，适合自定义布局）

```js
const simulation = d3.forceSimulation(nodes)
  .force('link', d3.forceLink(edges).id(d => d.id))
  .force('charge', d3.forceManyBody().strength(-300))
  .force('x', d3.forceX(d => d.x_hint).strength(0.3))  // 尊重列布局hint
  .force('y', d3.forceY(d => d.y_hint).strength(0.3))
  .force('center', d3.forceCenter(width/2, height/2))
```

---

## Canvas 数据的语义约定（供渲染时参考）

| node.type | 含义 | 建议形状 |
|---|---|---|
| `core_kg` | 来自论文的知识三元组 | 圆角矩形 |
| `context_kg` | 场景约束节点 | 菱形 |
| `inferred` | 推断节点（非论文直接提供）| 虚线圆角矩形 |
| `decision` | Traceability 设计决策 | 实线矩形，按TPACK着色 |
| `artifact` | 最终产物（功能/Prompt/步骤）| 六边形或胶囊形 |

| edge.type | 含义 | 建议样式 |
|---|---|---|
| `core_support` | Core KG 支撑决策 | 蓝色实线 |
| `context_constraint` | Context KG 约束决策 | 橙色实线 |
| `inference` | 推断关系 | 紫色虚线，带动画 |
| `conflict` | 冲突关系 | 红色虚线，双箭头 |
| `artifact` | 决策产出产物 | 绿色实线 |
