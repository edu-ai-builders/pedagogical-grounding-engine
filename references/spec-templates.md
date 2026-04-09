# 规格文件模板

PRD/TASKS/DESIGN 的完整展开模板。
注意：这些模板只在第5层选择导出PRD时使用。
每个功能必须附 Traceability 节点ID，否则不允许进入规格文件。

---

## PRD.md 模板

```md
<!-- 文件名: PRD.md -->
# [应用名称]  v0.1

## 愿景
帮助[目标用户]通过[核心机制]实现[可量化结果]。

## 理论基础
本产品基于以下论文的核心机制：
- [论文A简称]：[一句话说核心机制]
- [论文B简称]（如有）：[一句话说核心机制]
（每条功能的设计决策详见 /specs/traceability.json）

## 目标
- [目标1，附数值]
- [目标2，附数值]
- [目标3，附具体描述]

## 目标用户画像
| 画像 | 描述 | 主要使用功能 |
|---|---|---|

## 核心功能
（每个功能必须附 Traceability 节点ID）

### [功能名称]（Traceability: decision_XXX）
- 用户故事：作为[角色]，我希望[功能]，以便[价值]
- 用户可见行为：[描述]
- 系统逻辑：[描述]
- 触发条件：[描述]
- 降级方案：[来自Traceability节点的fallback字段]

## 成功指标
| KPI | 基线 | 目标 |
|---|---|---|

## 非目标
- 身份认证
- 持久化数据库
- WebSocket
- 测试基础设施
```

---

## TASKS.md 模板

```md
<!-- 文件名: TASKS.md -->

## 初始化
- [ ] `npx create-next-app@latest --typescript --tailwind --app --src-dir`
- [ ] `npx shadcn@latest init` + 按需安装组件
- [ ] `npm install lucide-react`
- [ ] `.env.local` → `OPENAI_API_KEY=sk-...`

## 核心状态层
- [ ] `src/lib/types.ts` — Message, EngagementMetrics 等接口
- [ ] `src/lib/chatStore.ts` — Map<string, Message[]>
- [ ] `src/utils/metrics.ts` — 参与度计算
- [ ] `src/utils/llm.ts` — OpenAI fetch 封装

## API路由
- [ ] `src/app/api/llm/route.ts` — 主回复（POST {messages} → {reply}）
- [ ] [功能专属路由]（按 Traceability prompt_segment 节点添加）

## 组件
（来自 DESIGN.md 文件树）

## 功能
（来自 Traceability artifact.type === "ui_feature" 的节点，按优先级排序）

## 样式
- [ ] 发言人颜色数组
- [ ] LLM 打字指示器
- [ ] 空状态提示
```

---

## DESIGN.md 模板

```md
<!-- 文件名: DESIGN.md -->

## 文件树（src/）
├─ app/
│  ├─ page.tsx
│  ├─ layout.tsx
│  └─ api/llm/route.ts
├─ components/
├─ lib/
│  ├─ chatStore.ts
│  └─ types.ts
└─ utils/
   ├─ metrics.ts
   └─ llm.ts

## 关键数据结构
（TypeScript 接口定义）

## 交互流程
（编号步骤：用户操作 → 状态更新 → LLM调用 → 渲染）

## LLM Prompt 设计
（从 /specs/traceability.json 的 prompt_segment 节点组装，每条规则附来源ID）

## UI 风格
（Tailwind 类 + shadcn 组件清单 + 动画方案）
```

---

## /specs/traceability.json 格式

这是 PRD 的元数据层，连接理论和实现：

```json
{
  "meta": {
    "papers": ["论文A", "论文B"],
    "scenario": "场景摘要",
    "generated_at": "ISO时间戳"
  },
  "decisions": [
    {
      "id": "decision_001",
      "name": "功能名称",
      "priority": "P0",
      "derived_from": { "core_kg": [], "context_kg": [] },
      "reasoning_chain": [],
      "pedagogy": { "strategy": "", "tpack_zone": "" },
      "artifact": { "type": "ui_feature|prompt_segment|workflow_step", "content": "" },
      "fallback": { "risk": "", "simplified": "" }
    }
  ]
}
```
