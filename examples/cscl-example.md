# 完整示例：CSCL论文的五层流水线执行样本

输入论文：Developing an LLM-Empowered Agent to Enhance Student Collaborative Learning
Through Group Discussion（ICCE 2024，香港教育大学）

场景描述：大学线上研讨课，6-8人小组，其中有沉默型学生，教师需要同时管理多个房间，
目标是让每个人都有实质性发言，产出一个可运行的Web应用原型。

---

## ✅ 第1层完成 — Core KG

**核心构念**：LLM智能体、对话历史收集、对话理解、参与度监控、回复生成、
静默检测、非活跃学生、同伴式语气、发言频率、构建性反馈

**三元组（25条）**：

陈述性（9条）：
1. **LLM智能体** → 由...组成 → **四个功能模块** [CSCL] [陈述]
2. **四个模块** → 包含 → **对话历史收集** [CSCL] [陈述]
3. **四个模块** → 包含 → **对话理解** [CSCL] [陈述]
4. **四个模块** → 包含 → **参与度监控** [CSCL] [陈述]
5. **四个模块** → 包含 → **回复生成** [CSCL] [陈述]
6. **同伴式语气** → 定义为 → 鼓励性、口语化、非权威 [CSCL] [陈述]
7. **纯提示词LLM** → 约束 → **多用户感知能力** [CSCL] [陈述]
8. **发言频率** → 测量 → **参与度水平** [CSCL] [陈述]
9. **学习目标达成** → 依赖 → **实质性参与** [CSCL] [陈述]

程序性（10条）：
10. **对话历史收集** → 存储 → {用户ID, 内容, 时间戳} [CSCL] [程序]
11. **检测周期** → 定义为 → 1分钟静默 [CSCL] [程序]
12. **静默>60秒** → 触发 → **对话理解** [CSCL] [程序]
13. **对话理解** → 读取 → **历史Map** [CSCL] [程序]
14. **参与度监控** → 计算 → **发言频率** [CSCL] [程序]
15. **发言频率==0** → 分类为 → **非活跃学生** [CSCL] [程序]
16. **非活跃学生** → 传递至 → **回复生成** [CSCL] [程序]
17. **回复生成** → 设置为 → **同伴式语气** [CSCL] [程序]
18. **智能体** → 生成 → **构建性反馈** [CSCL] [程序]
19. **话题误解** → 检测到 → **智能体** [CSCL] [程序]

产品implied（6条）：
20. **MySQL** → 替代 → **内存Map** [CSCL] [F]
21. **发言>40%** → 分类为 → **主导发言者** [CSCL+推断] [F]
22. **发言频率** → 可视化 → **彩色仪表盘** [推断] [F]
23. **参与度指标** → 导出为 → **CSV** [推断] [F]
24. **苏格拉底提问集** → 激活 → **每10轮后生成** [推断] [F]
25. **同伴发言** → 实例化 → **一键摘要请求** [推断] [F]

---

### 阶段4b：Representation Translation

**迁移对象**：#10-25（程序性 + 产品implied）共16条
**跳过**：#1-9（背景性陈述，不影响设计决策）

---

**#12 静默>60秒 → 触发 → 对话理解** [程序]

迁移后表征（执行语境）：
用户沉默超过阈值时间 → 系统自动激活干预流程

**translation_tension**
→ 研究边界条件：论文实验在课堂环境下测试，沉默=未参与
→ 执行语境变化：在线异步情境中，沉默可能是用户在思考、在打字、或已离开
→ 迁移假设：用60秒作为基准，但需要比研究原值更长的缓冲（研究没有给出异步场景的参数）

**cognitive_mechanism**：等待时间策略（Wait Time）——给学习者足够的思考缓冲，避免过早打断认知加工

**activation_condition**
→ 成立条件：用户确实在场（session活跃），只是没有发言
→ 不成立条件：用户已经离开页面 / 网络断开 / 正在输入长文本

**failure_signature**
→ 60秒触发过于频繁，用户感到被催促 → 关闭提醒或离开
→ 触发时用户刚好在思考中，打断了深度认知加工
→ 可观察信号：nudge发送后用户没有任何响应，且此后发言减少

**execution_form**：`prompt_rule`（系统提示词中的静默触发规则）+ `workflow_step`（检测→判断→发送私信的步骤序列）

---

**#15 发言频率==0 → 分类为 → 非活跃学生** [程序]

迁移后表征（执行语境）：
在滑动时间窗口内发言次数为零的用户 → 标记为需要干预的对象

**translation_tension**
→ 研究边界条件：论文以"发言频率"作为参与度的唯一代理指标
→ 执行语境变化：用户可能通过阅读、点赞、私信等方式参与，但频率计算不含这些
→ 迁移假设：口头发言是协作学习中最核心的参与形式，其他形式暂不纳入计算

**cognitive_mechanism**：社会性学习监控（Social Learning Monitoring）——通过可见参与信号识别脱离风险

**activation_condition**
→ 成立条件：讨论正在进行中（其他人在发言），该用户持续沉默
→ 不成立条件：全组都在沉默（不是个人问题）/ 用户已被标记为主持人角色

**failure_signature**
→ 误报：用户在认真阅读他人发言，被错误标记为非活跃
→ 漏报：用户发言了但内容极少（如"好的"），被计入活跃但实质未参与
→ 可观察信号：nudge触发后用户立刻回复"我在看"——说明分类准确但干预时机需调整

**execution_form**：`metric`（参与度分类指标）+ `ui_feature`（仪表盘的状态着色逻辑）

---

**#17 回复生成 → 设置为 → 同伴式语气** [程序]

迁移后表征（执行语境）：
AI回复的语调和措辞风格 → 配置为非权威、鼓励性的同伴角色语言

**translation_tension**
→ 研究边界条件：论文在中文课堂背景下测试，"同伴式语气"的文化语境是中文高校
→ 执行语境变化：如果部署在跨文化/英语环境，"同伴式"的措辞标准不同
→ 迁移假设：当前场景为中文大学生，保留原语境；英语场景需要重新校准语气词汇

**cognitive_mechanism**：社会临场感（Social Presence）——同伴语气减少权力距离感，降低表达焦虑

**activation_condition**
→ 成立条件：智能体在发起主动干预时（nudge、提问、摘要）
→ 不成立条件：智能体在回答事实性问题时（可以使用更中性的信息提供语气）

**failure_signature**
→ 语气过于随意，用户不把智能体的提示当回事
→ 语气模仿同伴但内容明显是机器生成（"AI腔"），反而破坏信任
→ 可观察信号：用户对智能体的nudge消息直接不回应，或回复"好"后不继续发言

**execution_form**：`prompt_rule`（系统提示词的角色定义规则）

---

**#22 发言频率 → 可视化 → 彩色仪表盘** [F] [推断]

迁移后表征（执行语境）：
每个参与者的实时发言比例 → 渲染为颜色编码的进度条，颜色反映参与状态

**translation_tension**
→ 研究来源：这是从论文的参与度监控机制推断出的实现形态，论文本身没有UI规格
→ 执行语境创造：这条 triple 是推断产物，没有研究边界条件的约束
→ 迁移挑战：需要决定颜色语义（绿=活跃/红=沉默 vs 其他方案）和更新频率

**cognitive_mechanism**：形成性评价（Formative Assessment）——实时可见的参与状态支持自我监控和同伴意识

**activation_condition**
→ 成立条件：讨论进行中，有多个参与者，至少有一定数量的发言（建议>5条才有意义）
→ 不成立条件：单人使用时 / 发言太少时颜色无参考价值

**failure_signature**
→ 过于频繁更新导致界面闪烁，增加认知负荷
→ 用户开始"为了让颜色变绿而发言"而非为了讨论内容——游戏化副作用
→ 可观察信号：用户频繁发短消息（如"嗯""对"）以维持仪表盘颜色

**execution_form**：`ui_feature`（EngagementMeter 组件）

---

**#21 发言>40% → 分类为 → 主导发言者** [F]

迁移后表征（执行语境）：
在滑动时间窗口内某用户占全组发言比例超过40% → 触发教师侧提醒

**translation_tension**
→ 研究来源：40%阈值来自协作学习文献的一般性建议，非CSCL论文原始数据
→ 执行语境变化：小组规模影响合理阈值——3人组的33%上限 vs 8人组的40%
→ 迁移假设：6-8人组使用40%阈值，小组规模变化时需重新校准

**cognitive_mechanism**：公平参与原则（Equitable Participation）——防止少数人主导影响其他人的学习机会

**activation_condition**
→ 成立条件：讨论已进行足够长时间（建议>10轮），有足够样本量
→ 不成立条件：用户是指定主持人 / 这一轮本来就是某人的分享时间

**failure_signature**
→ 误报：某用户因为需要解释复杂概念而发言较多，被错误标记
→ 教师收到提示但没有时间干预（同时管理多个房间）
→ 可观察信号：提示触发但发言比例在接下来几轮自然恢复均衡——说明不需要干预

**execution_form**：`metric`（发言比例计算指标）+ `ui_feature`（OverTalkBanner 组件）

---

**#24 苏格拉底提问集 → 激活 → 每10轮后生成** [F]

迁移后表征（执行语境）：
每累计10条对话消息 → 系统生成3个苏格拉底式追问，发送给指定主持人

**translation_tension**
→ 研究来源：苏格拉底式提问来自论文的"问题多样性促进批判思维"构念，10轮触发是推断
→ 执行语境创造：间隔和数量均为实现层面的参数决策，无研究依据
→ 迁移假设：10轮是经验值，实际部署需要根据讨论节奏调整

**cognitive_mechanism**：苏格拉底式提问（Socratic Questioning）——通过开放性问题激活批判性思维，避免讨论停留在表面共识

**activation_condition**
→ 成立条件：有指定主持人角色 / 讨论已有一定深度（不适合刚开始的破冰阶段）
→ 不成立条件：讨论已经非常活跃自主 / 主持人不需要提示

**failure_signature**
→ 生成的问题与当前讨论话题无关（LLM上下文理解不足）
→ 主持人忽略问题卡，功能形同虚设
→ 每10轮固定触发导致时机不恰当（刚达成结论时触发追问）

**execution_form**：`workflow_step`（消息计数器触发→LLM生成→问题卡展示）

---

### 迁移摘要

- 迁移了 **10条** 核心 triple（6条程序性 + 4条产品implied [F]），跳过6条背景陈述性
- 未迁移的6条产品implied triple（#20 内存Map替代MySQL / #23 CSV导出 / #25 一键摘要）为技术实现决策，直接进入执行层，无需认知机制分析

**主要 cognitive_mechanism**：
- 等待时间策略（Wait Time）
- 社会性学习监控（Social Learning Monitoring）
- 社会临场感（Social Presence）
- 形成性评价（Formative Assessment）
- 公平参与原则（Equitable Participation）
- 苏格拉底式提问（Socratic Questioning）

**主要 translation_tension 类型**：
- **参数无研究依据**（60秒阈值、40%阈值、10轮触发）——需要在部署中校准
- **场景边界条件变化**（课堂→在线异步，沉默的含义不同）
- **文化语境假设**（同伴式语气在中文高校环境下测试）
- **推断产物无原始约束**（UI可视化形态是实现层推断，非论文规格）

**执行形态分布**：
`prompt_rule` ×3 / `ui_feature` ×3 / `metric` ×2 / `workflow_step` ×1 / `constraint` ×1

**关键 failure_signature 模式**：
最容易断裂的地方是**参数阈值的误报**（沉默≠未参与、多发言≠主导）和**游戏化副作用**（用户为维持指标而非为了学习而行动）。

✅ 第1层完成：25条三元组（陈述9/程序10/产品implied 6），已迁移10条核心triple
              主要认知机制：等待时间/社会临场感/形成性评价/公平参与/苏格拉底提问
              主要迁移张力：参数无研究依据 / 场景边界条件变化 / 文化语境假设
              执行形态分布：prompt_rule×3 / ui_feature×3 / metric×2 / workflow_step×1



---

## ✅ 第2层完成 — Context KG

场景摘要：大学线上小组研讨，目标是让沉默学生实质参与，教师监督多个房间。

约束节点（12条）：
1. **沉默型学生** → 需要 → **低压力入场机制**
2. **在线异步情境** → 排除 → **需要同步口头发言的设计**
3. **教师同时管多房间** → 限制 → **教师无法手动干预**
4. **小组规模6-8人** → 要求 → **个体级别的参与度追踪**
5. **大学生** → 具备 → **基本数字素养，可理解可视化指标**
6. **研讨课话题** → 需要 → **苏格拉底式追问而非直接讲解**
7. **原型阶段** → 约束 → **无数据库，无身份认证**
8. **成功标准：每人实质发言** → 要求 → **可量化的参与度指标**
9. **教师行动研究需求** → 要求 → **会话后数据可导出**
10. **多人同时发言** → 需要 → **发言人标签和着色**
11. **学生表达焦虑** → 排除 → **公开评分或批评式反馈**
12. **Web应用** → 约束 → **浏览器端，无需安装**

✅ 第2层完成：12条约束节点，覆盖用户/情境/成功标准/技术约束全部六维度

---

## ✅ 第3层完成 — KG 融合

> 第3层现在直接消费阶段4b的迁移结果：
> `activation_condition` → 直接对应检测的输入
> `failure_signature` → 设计张力和实现障碍的识别线索

**对齐关系（直接对应）**：
- [#12 activation_condition：系统自动激活干预] ←直接对应→ [教师无法手动干预]
  → 迁移后的表征直接说明了为什么这个机制在这个场景是必须的
- [#14/#15 参与度监控+发言频率分类] ←直接对应→ [需要可量化参与度指标]
- [#18 构建性反馈目标指向特定学生] ←直接对应→ [个体级别追踪需求]

**间接激活（需要中间设计）**：
- [#17 同伴式语气 activation_condition：主动干预时使用] ←间接激活（需要：私信模式）→ [排除公开批评式反馈]
  中间设计：沉默学生的nudge必须是私信而非公开广播（来自 failure_signature 的反向推断：公开点名会触发失败信号）
- [#22 发言频率可视化] ←间接激活（需要：实时计算机制）→ [大学生可理解可视化指标]

**设计张力**（来自 translation_tension + 场景内部约束）：
- [#12 translation_tension：60秒触发可能打断思考中的学生] vs [c003 等待时间策略要求不过早干预]
  → 两者都保留：第一次静默给90秒缓冲（比论文原值更长），重复静默才触发
- [#21 failure_signature：误报风险——用户在认真阅读也会被标记] vs [c001 等距发言权必要]
  → 加入"正在输入"检测，区分真实沉默和延迟发言

**实现障碍**：
- [论文用MySQL存储] 受限于 [原型无数据库约束] → 降级：内存Map
- [#12 activation_condition 不成立条件：用户已离开页面] → 需要 session 活跃检测

**推断节点**：
- [M1: 私信式nudge] [推断] ← 由 [#17 同伴式语气] + [排除公开批评] + [#12 failure_signature：被催促感] 推导
  *(failure_signature 明确说明了为什么不能是公开广播)*
- [M2: 教师仪表盘视图] [推断] ← 由 [教师管多房间] + [可量化指标] 推导

✅ 第3层完成：3组直接对应，2组间接激活，2组设计张力（新增#21误报张力），2个实现障碍，2个推断节点

### 3b. Constraint Layer

**约束节点（4个）：**

```json
[
  {"id": "c001", "type": "pedagogical", "source": "CSCL论文",
   "claim": "等距发言权对协作学习效果是必要的", "priority": "High", "negotiable": false},
  {"id": "c002", "type": "contextual", "source": "场景",
   "claim": "界面认知负荷必须保持低水平（大学生同时处理讨论内容）", "priority": "High", "negotiable": true},
  {"id": "c003", "type": "pedagogical", "source": "CSCL论文+等待时间理论",
   "claim": "不应过早打断学生的思考过程（等待时间策略）", "priority": "Medium", "negotiable": true},
  {"id": "c004", "type": "technical", "source": "场景",
   "claim": "原型阶段无数据库，无身份认证", "priority": "High", "negotiable": false}
]
```

**约束间关系（3组）：**

```
关系1：c001 ← conflict → c002
描述：实时显示所有人发言比例（支持c001）会在界面上增加额外信息密度（违反c002）
Resolution：
  decision：用极简点状仪表盘而非数字表格
  rationale：保留公平参与的方向性感知，同时最小化界面元素数量
  sacrifice：牺牲精确数值，保留"谁发言多/少"的相对感知
  → resulting: decision_001（参与度仪表盘）

关系2：c003 ← conflict → c001（通过触发时机）
描述：等待时间策略要求给足思考空间，但公平参与要求及时识别沉默学生
Resolution：
  decision：首次沉默90秒才触发nudge（而非论文原始的60秒），重复沉默才缩短间隔
  rationale：用更长的等待时间保护思考空间，同时不放弃对持续沉默的干预
  sacrifice：部分牺牲及时性，换取对自然思考过程的尊重
  → resulting: decision_003（沉默学生私信nudge）

关系3：c001 ← amplify → c003（间接）
描述：等待时间保护思考过程后，学生发言质量更高，反而更支持公平参与目标
合力指向：nudge设计应当"邀请"而非"催促"
  → 两个约束共同强化了 decision_003 的"低压力私信"方向
```

✅ 第3层完成（含Constraint Layer）：3组直接对应，2组间接激活，1组张力，1个障碍，2个推断节点；4个约束节点，3组约束关系（冲突×2，强化×1）

---

## ✅ 第4层完成 — Traceability Engine（Constraint-driven）

### 设计决策1：实时参与度仪表盘（P0）

**约束来源：**
- 解析自约束关系：c001（公平参与）conflict c002（低认知负荷）
- Resolution：用极简点状仪表盘，牺牲精确数值，保留方向性感知

**Core KG 三元组来源：** `发言频率==0 → 分类为 → 非活跃学生` [CSCL]
**Context KG 约束节点：** `成功标准 → 要求 → 可量化参与度指标` [H][Must]

**推理链（constraint-driven）：**
1. 约束冲突：公平参与要求显示发言比例，低认知负荷要求界面简洁，两者在"显示多少信息"上矛盾
2. 优先级判断：c001（教学法约束，不可牺牲）> c002（可降级，用设计手段缓解）
3. Resolution：点状仪表盘而非数字表格——保留比较性感知，消除数字阅读负担
4. 设计实现：每人名字旁的圆形进度条，每30秒更新，颜色绿→黄→红

**跨决策影响：**
- 本决策影响 decision_003：仪表盘可见性减少了沉默学生的"被忽视感"，nudge语气可以更轻柔

**教学策略：** 形成性评价 + 自我觉察支持
**TPACK区域：** TPK
**优先级：** P0

```json
{
  "id": "decision_001",
  "name": "实时参与度仪表盘",
  "priority": "P0",
  "constraint_resolution": {
    "resolved_conflict": ["c001", "c002"],
    "resolution_rationale": "点状仪表盘保留方向性感知，消除数字阅读负担",
    "sacrifice": "牺牲精确数值，保留相对比较感知"
  },
  "derived_from": {
    "core_kg": ["发言频率==0 → 分类为 → 非活跃学生"],
    "context_kg": ["成功标准 → 要求 → 可量化参与度指标"]
  },
  "reasoning_chain": [
    "约束冲突：公平参与 vs 低认知负荷，在信息显示密度上矛盾",
    "优先级：教学法约束c001不可牺牲，c002可用设计手段缓解",
    "Resolution：极简点状仪表盘",
    "实现：圆形进度条，30秒更新，三色状态"
  ],
  "pedagogy": {"strategy": "形成性评价", "tpack_zone": "TPK"},
  "artifact": {"type": "ui_feature", "content": "EngagementMeter组件", "priority": "P0"},
  "cross_decision_edges": [
    {"target": "decision_003", "direction": "influences",
     "description": "仪表盘可见性减少被忽视感，nudge语气可更轻柔"}
  ],
  "fallback": {"risk": "30秒更新频繁重渲染", "simplified": "改为60秒更新"}
    "推导：把频率计算实时渲染为视觉元素",
    "实现：圆形进度条，30秒更新，三色状态"
  ],
  "pedagogy": { "strategy": "形成性评价", "tpack_zone": "TPK" },
  "artifact": { "type": "ui_feature", "content": "EngagementMeter组件", "priority": "P0" },
  "fallback": { "risk": "计算频繁导致性能问题", "simplified": "60秒更新代替30秒" }
}
```

---

### 设计决策2：过度发言检测Toast（P1）

**推理链**：
1. 论文：发言比例是参与均衡的核心指标
2. 场景张力：[M2推断节点] 教师管多房间 + 需要可量化指标
3. 推导：当某人发言比例超过40%，主动提示教师/主持人
4. 具体实现：滑动窗口180秒，超过0.4阈值触发橙色Toast，冷却2分钟

**教学策略**：公平参与
**TPACK区域**：TP（技术=滑动窗口算法，教学=公平参与，与具体内容关系弱）

```json
{
  "id": "decision_002",
  "name": "过度发言检测",
  "derived_from": {
    "core_kg": ["发言频率 → 测量 → 参与度水平"],
    "context_kg": ["教师同时管多房间 → 限制 → 无法手动干预"]
  },
  "reasoning_chain": [
    "论文：参与均衡是CSCL效果的核心指标",
    "场景：教师不在场无法手动干预",
    "推导：自动检测不均衡并主动提示",
    "实现：180秒滑动窗口，>0.4触发Toast"
  ],
  "pedagogy": { "strategy": "公平参与", "tpack_zone": "TP" },
  "artifact": { "type": "ui_feature", "content": "OverTalkBanner组件", "priority": "P1" },
  "fallback": { "risk": "误报（用户发言确实需要多）", "simplified": "改为全局累计比例，去掉滑动窗口" }
}
```

---

### 设计决策3：沉默学生私信nudge（P0）

**推理链**：
1. 论文：非活跃学生被传递至回复生成，智能体用同伴式语气回应
2. 场景张力：[M1推断节点] 同伴式语气 + 排除公开批评
3. 设计张力处理：等待时间策略要求不立刻干预 → 90秒（而非60秒）才触发第一次nudge
4. 具体实现：私信提示"要不要发一条？我可以帮你起草"

**教学策略**：脚手架 + 等待时间策略
**TPACK区域**：TPK

```json
{
  "id": "decision_003",
  "name": "沉默学生私信nudge",
  "derived_from": {
    "core_kg": ["非活跃学生 → 传递至 → 回复生成", "回复生成 → 设置为 → 同伴式语气"],
    "context_kg": ["学生表达焦虑 → 排除 → 公开批评式反馈", "低压力入场机制"]
  },
  "reasoning_chain": [
    "论文：非活跃学生需要智能体用同伴式语气介入",
    "场景：公开提示会增加焦虑，必须私信",
    "张力处理：等待时间理论要求90秒缓冲（非60秒）",
    "实现：私信提示+起草辅助，学生可编辑后发布"
  ],
  "pedagogy": { "strategy": "脚手架+等待时间策略", "tpack_zone": "TPK" },
  "artifact": { "type": "prompt_segment", "content": "系统提示词中的静默干预规则", "priority": "P0" },
  "fallback": { "risk": "频繁私信造成打扰", "simplified": "仅触发一次，之后等学生主动" }
}
```

✅ 第4层完成：3个完整Traceability节点（P0×2, P1×1），全部可溯源，无孤立设计

---

## ✅ 第5层完成 — 产物导出

### 导出A：Traceability Map

| 设计决策 | 来源论文机制 | 场景约束 | 教学策略 | TPACK | 优先级 |
|---|---|---|---|---|---|
| 参与度仪表盘 | 发言频率→非活跃分类 | 需要可量化指标 | 形成性评价 | TPK | P0 |
| 过度发言检测 | 发言频率测量参与均衡 | 教师无法手动干预 | 公平参与 | TP | P1 |
| 私信nudge | 非活跃→同伴式回应 | 排除公开批评 | 脚手架+等待时间 | TPK | P0 |
| 问题卡生成器 | 问题多样性→批判思维 | 苏格拉底式追问需求 | 苏格拉底式提问 | TPK | P1 |
| CSV导出 | 参与度指标可量化 | 教师行动研究需求 | 元认知支持 | TP | P1 |

### 导出A2：汇总 Canvas JSON（可视化图数据）

```json
{
  "meta": {
    "papers": ["CSCL-2024"],
    "scenario": "大学线上研讨课，6-8人小组，沉默型学生，教师管多个房间",
    "generated_at": "2024-01-01T00:00:00Z",
    "node_count": 12,
    "edge_count": 11
  },
  "layout": {
    "columns": {
      "constraint":  {"x_range": [50,  200], "color_by": "constraint_type"},
      "resolution":  {"x_range": [250, 400], "color": "#e9d5ff"},
      "source_kg":   {"x_range": [450, 650], "color_by": "kg_type"},
      "decision":    {"x_range": [700, 900], "color_by": "tpack_zone"},
      "artifact":    {"x_range": [950, 1150],"color": "#bbf7d0"}
    }
  },
  "nodes": [
    {"id": "n_c001", "label": "等距发言权必要", "type": "constraint", "constraint_type": "pedagogical", "x_hint": 100, "y_hint": 150},
    {"id": "n_c002", "label": "低认知负荷", "type": "constraint", "constraint_type": "contextual", "x_hint": 100, "y_hint": 350},
    {"id": "n_c003", "label": "等待时间策略", "type": "constraint", "constraint_type": "pedagogical", "x_hint": 100, "y_hint": 550},
    {"id": "n_r001", "label": "点状仪表盘\n（牺牲精确值）", "type": "resolution", "x_hint": 300, "y_hint": 250},
    {"id": "n_r002", "label": "90秒缓冲\n（非60秒）", "type": "resolution", "x_hint": 300, "y_hint": 550},
    {"id": "n_core_1", "label": "发言频率→非活跃", "type": "core_kg", "x_hint": 550, "y_hint": 150},
    {"id": "n_core_2", "label": "非活跃→同伴式回应", "type": "core_kg", "x_hint": 550, "y_hint": 350},
    {"id": "n_ctx_1",  "label": "教师无法手动干预", "type": "context_kg", "x_hint": 550, "y_hint": 550},
    {"id": "n_inf_1",  "label": "私信式nudge [推断]", "type": "inferred", "x_hint": 550, "y_hint": 700},
    {"id": "n_dec_1",  "label": "参与度仪表盘", "type": "decision", "tpack_zone": "TPK", "priority": "P0", "x_hint": 800, "y_hint": 200},
    {"id": "n_dec_2",  "label": "过度发言检测", "type": "decision", "tpack_zone": "TP",  "priority": "P1", "x_hint": 800, "y_hint": 400},
    {"id": "n_dec_3",  "label": "沉默私信nudge", "type": "decision", "tpack_zone": "TPK", "priority": "P0", "x_hint": 800, "y_hint": 600},
    {"id": "n_art_1",  "label": "EngagementMeter", "type": "artifact", "artifact_type": "ui_feature", "x_hint": 1050, "y_hint": 200},
    {"id": "n_art_2",  "label": "Prompt: 静默干预规则", "type": "artifact", "artifact_type": "prompt_segment", "x_hint": 1050, "y_hint": 600}
  ],
  "edges": [
    {"from": "n_c001", "to": "n_r001", "label": "conflict", "type": "constraint_conflict"},
    {"from": "n_c002", "to": "n_r001", "label": "conflict", "type": "constraint_conflict"},
    {"from": "n_c003", "to": "n_r002", "label": "conflict", "type": "constraint_conflict"},
    {"from": "n_c001", "to": "n_r002", "label": "conflict", "type": "constraint_conflict"},
    {"from": "n_r001", "to": "n_dec_1", "label": "drives", "type": "resolution_to_decision"},
    {"from": "n_r002", "to": "n_dec_3", "label": "drives", "type": "resolution_to_decision"},
    {"from": "n_core_1","to": "n_dec_1", "label": "支撑", "type": "core_support"},
    {"from": "n_core_2","to": "n_dec_3", "label": "支撑", "type": "core_support"},
    {"from": "n_ctx_1", "to": "n_dec_2", "label": "约束", "type": "context_constraint"},
    {"from": "n_inf_1", "to": "n_dec_3", "label": "推断激活", "type": "inference"},
    {"from": "n_core_1","to": "n_inf_1", "label": "共同推断", "type": "inference"},
    {"from": "n_dec_1", "to": "n_dec_3", "label": "影响nudge语气", "type": "cross_decision"},
    {"from": "n_dec_1", "to": "n_art_1", "label": "产出", "type": "artifact"},
    {"from": "n_dec_3", "to": "n_art_2", "label": "产出", "type": "artifact"}
  ]
}
```

---

### Commands 菜单（流水线完成后输出）

```
✅ 流水线完成。可用命令：

/trace      — Traceability Map（Markdown 汇总表 + Canvas JSON）
/render     — 直接渲染 Canvas（React Flow 交互式图）
/kg         — KG 原始图数据（Core + Context + Constraint 合并）
/prompt     — System Prompt（含来源标注）
/prd        — PRD + TASKS + DESIGN 三份规格文件
/workflow   — Workflow 步骤序列
/constraint — Constraint Layer 详细报告
/why 参与度仪表盘  — 解释为什么是点状设计而不是数字表格
/why 私信nudge     — 解释为什么是90秒而不是60秒
/export     — 输出所有产物
```

---

### /why 私信nudge（示例）

```
问：为什么沉默触发时间是90秒而不是论文原来的60秒？

约束冲突：c003（等待时间策略：不要打断思考）vs c001（公平参与：及时识别沉默）
优先级判断：c003 是可协商的（negotiable: true），c001 是不可协商的

Resolution：延长等待时间到90秒，牺牲及时性，换取对自然思考过程的尊重。
重复沉默（同一会话内第二次）才缩短间隔至60秒——因为此时"在思考"的可能性降低。

被排除的方案：
- 60秒立刻触发 → 违反c003（打断思考），即使论文原文用的是这个数字
- 不自动触发，依赖教师手动 → 违反c001（教师不在场无法干预）
- 公开提示而非私信 → 违反场景约束"排除公开批评"

如果约束c003被移除（场景变为"需要快速节奏的讨论"），则触发时间会改回60秒。
```

---

🚀 **下一步**

流水线已完成，Constraint Layer 显式化了3组约束关系，Canvas JSON 包含14个节点和14条边（含跨决策连接）。

建议执行顺序：
1. `/render` — 先在 artifact 中看一眼图，验证推理结构是否符合你的预期
2. `/prompt` — 验证 System Prompt 的教学策略是否在真实对话中生效
3. `/constraint` — 检查约束层是否遗漏了重要的设计张力
4. `/prd` — 确认推理链验证通过后，再生成工程实现规格
