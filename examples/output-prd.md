# /prd 命令输出示例
# 来源：CSCL论文 × 大学线上研讨课场景

---

```md
<!-- 文件名: PRD.md -->
# 协作学习讨论助手  v0.1

## 理论基础
本产品基于以下研究的核心机制：
- **CSCL-2024**：LLM驱动的智能体通过监控发言频率识别非活跃学生，
  以同伴式语气进行定向干预，提升协作讨论的参与均衡性。

所有功能决策的完整推理链见 `/specs/traceability.json`。

## 愿景
帮助在线研讨课的大学生，通过AI参与度监控和轻度引导，
让每个人的想法都能进入讨论，而不只是沉默地听。

## 目标
- 将沉默学生比例（会话内发言次数=0）从约20%降低至5%以下
- 将单人发言占比控制在40%以下，避免讨论被少数人主导
- 让教师在管理多个房间时，无需主动干预即可了解各组参与状况

## 目标用户画像

| 画像 | 描述 | 主要使用功能 |
|---|---|---|
| 沉默思考者 | 阅读理解强，口头表达信心低的大学生，习惯等待机会发言 | 私信nudge + 参与度仪表盘 |
| 授课教师 | 同时监督多个在线小组，无法逐一干预 | 过度发言检测Toast + 会话导出 |
| 同伴主持人 | 负责引导小组讨论方向的学生角色 | 问题卡生成器 |

## 核心功能

### 功能1：实时参与度仪表盘（Traceability: decision_001）
- **约束依据**：公平参与（Must）× 低认知负荷（High, Tradeoff）的冲突 → 选择点状仪表盘而非数字表格，牺牲精确数值，保留方向性感知
- **用户故事**：作为学生，我希望看到小组每人的参与状态，以便了解讨论是否均衡
- **用户可见行为**：每个发言人名字旁显示圆形进度条，颜色绿→黄→红，30秒更新一次
- **系统逻辑**：`calcTurnRatio(messages, speakerId)` 每30秒重新计算，推送到仪表盘组件
- **降级方案**：如性能问题，改为60秒更新（代价：实时性降低，但功能完整）

### 功能2：沉默学生私信 Nudge（Traceability: decision_003）
- **约束依据**：公平参与（Must）× 等待时间策略（Medium, Tradeoff）的冲突 → 90秒（非60秒）首次触发，牺牲及时性换取思考空间保护
- **用户故事**：作为沉默学生，我希望在没有压力的情况下被邀请发言
- **用户可见行为**：私信弹出："你有什么想法？我可以帮你起草一条。"；同组其他人看不到
- **系统逻辑**：客户端计时，90秒无发言 → POST `/api/llm/nudge` → 私信展示
- **降级方案**：如LLM延迟>2秒，改为预设固定模板的nudge（代价：个性化程度降低）

### 功能3：过度发言检测（Traceability: decision_002）
- **约束依据**：公平参与（Must）× 教师无法手动干预（High, Must）→ 自动化检测，仅教师视图展示
- **用户故事**：作为教师，我希望在某人主导讨论时收到提示，以便及时干预
- **用户可见行为**：教师视图顶部出现橙色Toast：「[用户名] 最近3分钟发言占67%，可以邀请其他成员」
- **系统逻辑**：滑动窗口180秒，ratio > 0.4 → 触发Toast，冷却2分钟避免重复
- **降级方案**：如滑动窗口性能问题，改为全局累计比例（代价：失去时间窗口的精确性）

### 功能4：问题卡生成器（Traceability: decision_004）
- **约束依据**：苏格拉底式提问策略 × 主持人角色需求 → 每10轮生成3个追问，仅主持人可见
- **用户故事**：作为主持人，我希望在讨论停滞时获得引导性问题，以便推动深度讨论
- **用户可见行为**：右侧面板每10条消息更新一次，展示3张可点击的问题卡
- **系统逻辑**：消息计数器，每10条 → POST `/api/llm/questions` → 更新问题卡面板

## 成功指标

| KPI | 基线 | 目标 |
|---|---|---|
| 沉默学生比例（发言次数=0）| ~20% | <5% |
| 单人最高发言占比 | 无限制 | <40% |
| 教师干预次数/会话 | 依赖手动 | <2次（其余由系统提示）|

## 非目标（本版本不实现）
- 用户身份认证与账号系统
- 持久化数据库（所有数据在会话结束后清空）
- WebSocket实时推送（使用客户端轮询和计算代替）
- 自动化测试套件
- 发言内容的语义分析（仅基于发言频率，不分析内容）
```

---

```md
<!-- 文件名: TASKS.md -->

## 初始化
- [ ] **项目创建** `npx create-next-app@latest cscl-chat --typescript --tailwind --app --src-dir`
- [ ] **UI组件库** `npx shadcn@latest init` → 安装 `card badge button tooltip toast`
- [ ] **图标库** `npm install lucide-react`
- [ ] **环境变量** `.env.local` → `OPENAI_API_KEY=sk-...`

## 核心状态层（先做这个，其他层依赖它）
- [ ] **类型定义** `src/lib/types.ts`
      `Message`, `Speaker`, `EngagementMetrics`, `SessionConfig`
- [ ] **消息存储** `src/lib/chatStore.ts`
      `Map<sessionId: string, messages: Message[]>`
      导出: `getMessages`, `addMessage`, `clearSession`
- [ ] **指标计算** `src/utils/metrics.ts`
      `calcTurnRatio(messages, speakerId)` → 发言比例
      `calcSilenceSec(messages, speakerId)` → 距上次发言秒数
      `getOverTalker(messages, windowSec)` → 返回超过40%的用户
- [ ] **LLM封装** `src/utils/llm.ts`
      `callLLM(systemPrompt, messages, maxTokens)` → fetch OpenAI, 含超时处理

## API路由
- [ ] **主回复** `src/app/api/llm/route.ts`
      POST `{messages: Message[], sessionId: string}` → `{reply: string}`
- [ ] **nudge生成** `src/app/api/llm/nudge/route.ts`
      POST `{context: Message[], targetUser: string}` → `{nudgeText: string}`
      降级：如调用失败，返回预设模板"你有什么想法？我可以帮你起草。"
- [ ] **问题卡** `src/app/api/llm/questions/route.ts`
      POST `{messages: Message[]}` → `{questions: string[]}`（JSON数组，3个问题）

## 组件（按依赖顺序）
- [ ] **MessageBubble** `src/components/MessageBubble.tsx`
      单条消息展示，按发言人着色，支持私信标记
- [ ] **MessageList** `src/components/MessageList.tsx`
      消息流，自动滚动到底部
- [ ] **InputBox** `src/components/InputBox.tsx`
      文本输入 + 发送按钮 + 发言人选择（多用户模式）
- [ ] **EngagementMeter** `src/components/EngagementMeter.tsx`
      圆形进度条，Props: `{speakerId, ratio, status}`
      status: "active"(绿) / "passive"(黄) / "silent"(红)
- [ ] **OverTalkBanner** `src/components/OverTalkBanner.tsx`
      顶部橙色Toast，Props: `{speakerId, ratio}`
      仅在教师视图显示
- [ ] **QuestionCards** `src/components/QuestionCards.tsx`
      右侧面板，Props: `{questions: string[]}`
      点击问题卡 → 填入 InputBox
- [ ] **ChatRoom** `src/components/ChatRoom.tsx`
      整体布局容器，组合以上所有组件

## 功能实现（按 P0 优先）
- [ ] **[P0] 参与度仪表盘** 每30秒调用 `getEngagementSummary()`，更新 EngagementMeter
- [ ] **[P0] 私信nudge** 客户端计时器，90秒无发言 → POST nudge API → 展示私信气泡
- [ ] **[P1] 过度发言检测** 每次消息后检查 `getOverTalker(messages, 180)`，>0.4 → 显示 OverTalkBanner
- [ ] **[P1] 问题卡生成** 消息计数器，每10条 → POST questions API → 更新 QuestionCards
- [ ] **[P1] CSV导出** `utils/export.ts` 从 chatStore 生成 Blob，触发浏览器下载

## 样式收尾
- [ ] 发言人颜色映射（6色方案）
- [ ] LLM 打字指示器（三点动画）
- [ ] 私信气泡样式（与普通消息区分）
- [ ] 空会话状态（欢迎消息 + 话题输入框）
```

---

```md
<!-- 文件名: DESIGN.md -->

## 文件树（src/）
├─ app/
│  ├─ page.tsx                        // 主入口：初始化 sessionId，渲染 ChatRoom
│  ├─ layout.tsx
│  └─ api/
│     └─ llm/
│        ├─ route.ts                  // 主回复 POST
│        ├─ nudge/route.ts            // nudge 生成 POST
│        └─ questions/route.ts        // 问题卡生成 POST
│
├─ components/
│  ├─ ChatRoom.tsx                    // 布局：左侧聊天(flex-1) + 右侧面板(w-72)
│  ├─ MessageList.tsx                 // 消息流
│  ├─ MessageBubble.tsx               // 单条消息，支持私信样式
│  ├─ InputBox.tsx                    // 输入区
│  ├─ EngagementMeter.tsx             // 参与度仪表盘
│  ├─ OverTalkBanner.tsx              // 过度发言Toast
│  ├─ QuestionCards.tsx               // 问题卡面板
│  └─ ui/                             // shadcn 自动生成
│
├─ lib/
│  ├─ chatStore.ts                    // Map<string, Message[]>
│  └─ types.ts
│
└─ utils/
   ├─ metrics.ts                      // 参与度计算
   ├─ llm.ts                          // OpenAI fetch 封装
   └─ export.ts                       // CSV Blob 生成

## 关键数据结构

```ts
interface Message {
  id: string                          // crypto.randomUUID()
  role: "user" | "assistant"
  content: string
  speakerId: string
  isPrivate?: boolean                 // 私信nudge
  privateTargetId?: string            // 私信目标用户
  timestamp: number                   // Date.now()
}

interface EngagementMetrics {
  speakerId: string
  turnCount: number
  lastActiveAt: number
  silenceSec: number
  ratio: number                       // 0-1，该用户占总发言比例
  status: "active" | "passive" | "silent"
}

interface SessionConfig {
  sessionId: string
  topic: string
  participantCount: number
  facilitatorId: string
  isTeacherView: boolean
}
```

## 交互流程

1. 用户选择发言人身份，输入消息，点击发送
2. `addMessage(sessionId, {..., role: 'user', speakerId})`
3. 消息计数器 +1，如果 count % 10 === 0 → POST `/api/llm/questions` → 更新问题卡
4. POST `/api/llm` (最近12条消息) → 显示打字指示器 → 收到回复 → append
5. 每30秒：`getEngagementSummary()` → 更新所有 EngagementMeter
6. 每次新消息后：`getOverTalker(messages, 180)` → 如超过0.4且isTeacherView → 显示Toast
7. 客户端计时器：检查每个用户的 `silenceSec`，首次≥90秒 → POST `/api/llm/nudge` → 私信气泡

## LLM Prompt 设计

### /api/llm（主回复）
系统提示：见 `/specs/output-prompt.md` 完整版本
最大 token：200 | 超时：5秒 | 降级：返回"我在思考中..."

### /api/llm/nudge
系统提示："根据以下最近5条对话，为{targetUser}生成一条温和的私信邀请，不超过20字，不提及ta沉默。"
最大 token：80 | 降级：返回预设模板

### /api/llm/questions
系统提示："基于以下讨论，生成3个苏格拉底式追问，返回JSON数组格式：[{\"q\":\"...\"}]，每题不超过25字。"
最大 token：150 | 解析：JSON.parse，失败时返回3个通用问题

## UI 风格
- 布局：两栏，左侧消息流，右侧问题卡面板（可折叠）
- 主色：slate 系列
- 参与度颜色：green-400(active) / yellow-400(passive) / red-400(silent)
- 过度发言：amber-500 Toast，顶部固定
- 私信气泡：紫色左边框 `border-l-4 border-purple-400 bg-purple-50`
- 字体：系统默认（无需引入外部字体）
```
