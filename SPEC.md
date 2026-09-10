# AI 海龟汤前端架构设计

> 项目目标：基于 `UI_design.html` 的壁炉圆桌界面，在 SillyTavern + 酒馆助手环境中实现一个配置可持久化、游戏过程驻留内存、支持多个 AI 角色协作的海龟汤游戏前端。
>
> 本文是目标架构，不是当前实现说明。实现时必须遵守 `Coding rule.md`：先确认重大架构变更，再进行代码施工；每轮施工追加 `LOG.md` 和 `LOG-INDEX.md`。

## 1. 设计依据

### 1.1 现有界面

`UI_design.html` 已经定义了主要视觉和交互区域：

- 顶部状态栏：游戏名称、汤面摘要、API、设置、全屏。
- 左侧圆桌舞台：主持人、三个 AI 伙伴、玩家头像、气泡、发言动画。
- 中央便签墙：已确认事实、重要线索、核心疑问、已排除信息。
- 底部输入栏：玩家提问或直接猜测。
- 右侧记录区：全部、裁判判定、我的提问三个筛选视图。
- 人设编辑弹窗：姓名、头像、角色提示词、独立 API 配置。
- 全局设置弹窗：自动协作、动效、头像库和清空记录。

现有 HTML 中的示例逻辑是本地模拟逻辑，不能直接作为真实游戏规则：`hostEvaluate()` 目前按关键词判断，`globalApiConfig` 和角色 API 配置写入 `localStorage`，对话记录也只存在内存中。后续实现应保留 DOM 和样式结构，替换数据层与请求层。

### 1.2 酒馆助手能力

本设计以酒馆助手文档中已确认的能力为基础：

- 以 iframe 渲染带有 `<body>` 的完整 HTML，支持 `<script>`、CSS 和现代前端代码。
- `getVariables`、`replaceVariables`、`updateVariablesWith`、`insertOrAssignVariables` 等变量接口。
- 变量作用域：`global`、`preset`、`character`、`chat`、`message`、`script`、`extension`。
- `getWorldbook`、`getWorldbookNames`、`getCharWorldbookNames`、`getChatWorldbookName`。
- `getChatMessages(range, option)`，支持楼层范围、角色筛选、隐藏状态和消息页。
- `generate` / `generateRaw`，支持自定义生成、`ordered_prompts`、`injects`、流式事件、生成 ID、停止生成和工具调用。
- `injectPrompts` / `uninjectPrompts`，支持 `in_chat`、`none`、深度、角色和一次性注入。
- `eventOn`、`eventOnce`、`eventEmit`、`eventEmitAndWait`，以及 `tavern_events` 和 `iframe_events`。
- 原生 `SillyTavern.getContext()` 作为稳定接口补充入口。

文档入口：

- [酒馆助手介绍](https://n0vi028.github.io/JS-Slash-Runner-Doc/guide/%E5%85%B3%E4%BA%8E%E9%85%92%E9%A6%86%E5%8A%A9%E6%89%8B/%E4%BB%8B%E7%BB%8D.html)
- [渲染器](https://n0vi028.github.io/JS-Slash-Runner-Doc/guide/%E5%9F%BA%E6%9C%AC%E7%94%A8%E6%B3%95/%E6%B8%B2%E6%9F%93%E5%99%A8.html)
- [变量类型](https://n0vi028.github.io/JS-Slash-Runner-Doc/guide/%E5%8A%9F%E8%83%BD%E8%AF%A6%E6%83%85/%E5%8F%98%E9%87%8F/%E5%8F%98%E9%87%8F%E7%B1%BB%E5%9E%8B.html)
- [获取变量](https://n0vi028.github.io/JS-Slash-Runner-Doc/guide/%E5%8A%9F%E8%83%BD%E8%AF%A6%E6%83%85/%E5%8F%98%E9%87%8F/%E8%8E%B7%E5%8F%96%E5%8F%98%E9%87%8F.html)
- [获取楼层消息](https://n0vi028.github.io/JS-Slash-Runner-Doc/guide/%E5%8A%9F%E8%83%BD%E8%AF%A6%E6%83%85/%E6%A5%BC%E5%B1%82%E6%B6%88%E6%81%AF/%E8%8E%B7%E5%8F%96%E6%B6%88%E6%81%AF.html)
- [请求生成](https://n0vi028.github.io/JS-Slash-Runner-Doc/guide/%E5%8A%9F%E8%83%BD%E8%AF%A6%E6%83%85/%E8%AF%B7%E6%B1%82%E7%94%9F%E6%88%90.html)
- [注入提示词](https://n0vi028.github.io/JS-Slash-Runner-Doc/guide/%E5%8A%9F%E8%83%BD%E8%AF%A6%E6%83%85/%E6%B3%A8%E5%85%A5%E6%8F%90%E7%A4%BA%E8%AF%8D.html)
- [监听和发送事件](https://n0vi028.github.io/JS-Slash-Runner-Doc/guide/%E5%8A%9F%E8%83%BD%E8%AF%A6%E6%83%85/%E7%9B%91%E5%90%AC%E5%92%8C%E5%8F%91%E9%80%81%E4%BA%8B%E4%BB%B6.html)
- [访问酒馆接口或其他插件](https://n0vi028.github.io/JS-Slash-Runner-Doc/guide/%E5%8A%9F%E8%83%BD%E8%AF%A6%E6%83%85/%E8%AE%BF%E9%97%AE%E9%85%92%E9%A6%86%E6%8E%A5%E5%8F%A3%E6%88%96%E5%85%B6%E6%8F%92%E4%BB%B6.html)

## 2. 目标与边界

### 2.1 目标

1. 玩家可以在酒馆楼层中打开完整的海龟汤前端。
2. 初始汤面和汤底由酒馆当前 AI 生成，前端从约定标签中提取并校验。
3. 解题型主持人掌握汤底；汤底在每一次解题型主持人请求中以高注意级别的系统区块持续注入，闲聊模式明确排除该区块。
4. 玩家和伙伴 AI 只看到汤面、公开历史和公开判定，不存在可见的角色内部思维流。
5. 支持一个玩家、一个主持人和 0~N 个 AI 伙伴；UI 第一版默认展示三个伙伴。
6. 真实模式对本轮抽中的伙伴各独立调用一次 LLM；简单模式一次调用让 LLM 扮演本轮抽中的多个伙伴。
7. 主持人和伙伴允许使用不同模型、预设或自定义 API 配置。
8. 角色人设可从绑定世界书读取，并允许前端追加请求级角色提示词；追加提示词不写回世界书，也不注入酒馆正文。
9. 初版游戏过程只保存在内存；刷新 iframe 后清空游戏过程。仅 API 配置、角色头像和角色人设持久化。
10. 游戏结束时，由 AI 将过程总结成可续写文本，再发送回酒馆供当前 AI 继续创作。
11. 不依赖某个固定 API 厂商，优先复用酒馆当前预设和模型配置。
12. 酒馆助手不可用时仍能以手动测试模式打开页面，并给出明确能力降级状态。

### 2.2 非目标

- 不把 API Key 写入聊天变量、消息文本或提示词。
- 不通过直接修改酒馆 DOM 作为主通信方式。
- 不在第一版实现多人网络联机、服务端数据库、跨设备同步或游戏过程持久化。
- 不让模型直接修改游戏状态；模型只返回结构化事件，由前端校验后提交。
- 不用关键词匹配替代主持人的真实判定。

## 3. 总体分层

```text
┌──────────────────────────────────────────────┐
│ UI 层：UI_design.html                        │
│ 圆桌、气泡、便签、记录、设置、移动端 Tab      │
└──────────────────────┬───────────────────────┘
                       │ ViewModel / Action
┌──────────────────────▼───────────────────────┐
│ 游戏应用层 GameApp                            │
│ 回合状态机、动作队列、模式编排、错误恢复       │
└──────────────┬──────────────┬─────────────────┘
               │              │
┌──────────────▼──────┐ ┌─────▼────────────────┐
│ PromptPolicy         │ │ TavernAdapter        │
│ 秘密分区、上下文裁剪 │ │ 变量、消息、事件、生成 │
│ 角色提示词拼装       │ │ 世界书、能力探测       │
└──────────────┬──────┘ └─────┬────────────────┘
               │              │
┌──────────────▼──────────────▼────────────────┐
│ Domain Store                                  │
│ soup / secret / public state / turns / clues  │
│ schema version / future recovery extension    │
└───────────────────────────────────────────────┘
```

### 3.1 单 HTML 文件约束

酒馆助手 iframe 前端按当前环境要求使用单个 HTML 文件，不能假设 `src/` 下的多个 JS/CSS 文件会被酒馆助手一并加载。因此最终交付物必须像 `RpgCombat/index.html` 一样，将样式、标记和脚本放在一个 HTML 中。

单文件内部仍然需要保持模块边界，采用命名空间对象或 IIFE 分区：

```text
index.html
  <style> UI 样式，只保留视图层规则 </style>
  <body> UI_design.html 的界面结构 </body>
  <script>
    TurtleApp.Config          # 常量、能力开关、版本
    TurtleApp.Domain          # 状态、事件、校验、内存 store
    TurtleApp.Tavern          # 酒馆助手适配器和兼容回退
    TurtleApp.Worldbook       # 世界书角色人设读取
    TurtleApp.Prompt          # 信息投影和提示词构建
    TurtleApp.Modes           # real/simple 请求编排
    TurtleApp.Summary         # 总结并回传酒馆续写
    TurtleApp.Persistence     # 仅 API/头像/人设的持久化
    TurtleApp.UI              # DOM 绑定、渲染和动画
    TurtleApp.Boot            # 初始化、事件注册、清理
  </script>
```

模块之间只通过公开方法通信，禁止在 UI 事件处理器中直接拼接提示词或调用全局酒馆函数。后续若酒馆助手模板支持可靠的构建/打包流程，可以再拆文件；在此之前不要以多文件结构作为运行时前提。

## 4. 核心状态模型

以下对象是内部 JSON 模型。变量名和字段名使用 ASCII，显示文本继续使用中文。

```js
{
  schemaVersion: 1,
  sessionId: "turtle-<uuid>",
  status: "idle" | "generating-soup" | "announcing-surface" | "ready" | "peer-batch" | "host-answer" | "awaiting-player" | "chat-batch" | "summarizing" | "revealed" | "error",
  mode: "real" | "simple",
  turn: 7,
  soup: {
    title: "",
    surface: "玩家和所有伙伴可见的汤面",
    rules: ["问题应尽量能由是/否/无关回答"],
    answerLabels: ["yes", "no", "irrelevant", "critical_yes"]
  },
  secret: {
    truth: "酒馆当前 AI 生成、只有主持人可见的汤底",
    facts: [],
    revealPolicy: "host_only_until_solved",
    generatedAt: null,
    promptVersion: 1
  },
  actors: {
    host: { id: "host", name: "", persona: "", avatar: "", enabled: true },
    peers: [{ id: "ai-1", name: "", persona: "", avatar: "", enabled: true }],
    player: { id: "player", name: "我", avatar: "" }
  },
  publicState: {
    noteWall: {
      confirmedFacts: [],
      importantClues: [],
      coreQuestions: [],
      ruledOut: [],
      revision: 0,
      lastUpdatedBy: null
    },
    publicTurns: []
  },
  turns: [],
  summary: { status: "none", text: null, outcome: null },
  ui: { currentFilter: "all", mobileView: "stage", animations: true, chatModeEnabled: false },
  request: { activeIds: [], lastError: null, lastCompletedAt: null }
}
```

`secret` 永远不能直接传入 `publicState`、`turns[].publicText` 或 UI 的普通记录列表。需要显示给玩家的内容必须经过 `RevealPolicy` 显式转换。

### 4.1 回合事件

```js
{
  id: "turn-0007",
  turn: 7,
  type: "surface_announcement" | "player_question" | "host_answer" | "peer_discussion" | "chat_message" | "final_guess" | "system",
  actorId: "player" | "host" | "ai-1",
  text: "玩家原始提问或 AI 发言",
  verdict: "yes" | "no" | "irrelevant" | "critical_yes" | null,
  visibility: "public" | "host_only" | "debug_only",
  createdAt: "ISO-8601",
  requestId: "optional-generation-id"
}
```

所有游戏状态，包括汤面、汤底、便签、回合和总结，都只存在当前 iframe 的内存中。模型原始回答只放在内存调试缓存，刷新后全部清空。`Persistence` 只处理 API 配置、角色头像和角色人设；状态模型保留 `schemaVersion` 和迁移字段，供未来恢复功能使用，但初版不写入 `chat` 或 `message` 变量。

### 4.2 有限伙伴讨论状态机

每个游戏回合的固定顺序是：主持人公开汤面 -> 伙伴批次各自最多发言一次 -> 主持人用一次请求集中回答本批次提出的问题 -> 随机总结者更新便签墙 -> 等待玩家输入。伙伴发言不得通过“AI 发言完成”再次触发伙伴请求；玩家输入才会开启下一轮。

```js
{
  phase: "announcing_surface" | "peer_batch" | "host_answer" | "awaiting_player" | "chat_batch" | "summarizing",
  round: 3,
  peerPolicy: {
    enabled: true,
    maxSpeakers: 3,
    speakProbability: 0.5,
    oneSpeechPerRound: true,
    allowAutoFollowUp: false,
    allowAiToAskHost: false
  },
  speakerQueue: ["ai-1", "ai-2"],
  completedSpeakers: [],
  awaitingPlayerInput: true
}
```

每轮开始时由前端为每个启用伙伴独立抽取一次 Bernoulli(`speakProbability`)。抽样结果写入当前回合的 `speakerQueue`，重试时不得重新抽样；因此每个角色本轮发言概率为 50%，也允许出现本轮无人发言的安静回合。简单模式对 `speakerQueue` 发起一次合并请求，真实模式对队列中的角色各发起一次独立请求。每个选中的角色最多生成一条公开发言。伙伴的展示顺序必须按 `speakerQueue` 轮流渲染；真实模式可以并行请求以降低等待时间，但不能以完成先后改变公开顺序。

伙伴发言完成后，前端收集本批次所有公开问题，向主持人发起且只发起一次聚合请求。主持人的输入包含汤底高注意区块、公开谜面、玩家最新输入和本批次伙伴发言；输出按 `question_id` 返回全部判定，并可附带一条面向玩家的公开总结。若 `speakerQueue` 为空或没有问题，则跳过主持人聚合请求，但仍进入总结者阶段。

两种游戏模式都不允许 AI 自动追问主持人、自动提交建议问题或自动开启下一批。UI 可以提供“只让一个 AI 回应”和“跳过伙伴讨论”按钮，但它们只作用于当前玩家触发的这一轮。

游戏状态转换固定为：

```text
announcing-surface
  -> peer-batch（按 50% 概率筛选）
  -> host-answer（一次聚合请求，可跳过）
  -> summarizing（随机伙伴；无伙伴时主持人）
  -> awaiting-player

awaiting-player + 玩家游戏输入
  -> peer-batch
  -> host-answer
  -> summarizing
  -> awaiting-player
```

主持人聚合失败、伙伴超时、总结者失败、用户点击停止或生成结果校验失败时，当前批次立即结束并回到 `awaiting-player`（保留已成功的公开事件和上一版便签），显示对应的手动重试入口。重试使用同一问题账本和同一总结者，不得重新抽样或递归开启新批次。

### 4.3 闲聊模式

输入栏右侧的“闲聊模式”是一次输入级开关，不是永久改变游戏规则的状态。开启后，本次玩家输入不进入主持人判定流程，而进入 `chat-batch`：

```text
玩家闲聊输入
  -> 主持人回应一次（朋友口吻，无 host_secret）
  -> AI 伙伴各回应一次（真实模式逐角色；简单模式合并请求）
  -> 回到 awaiting-player
```

闲聊模式的特殊规则：

1. 所有启用 AI 都必须回应一次，包括主持人；不使用游戏回合中的 50% 伙伴发言抽样。
2. 所有请求都只接收汤面、公共历史、玩家闲聊文本和临时的朋友口吻指令；主持人请求完全不注入 `secret.truth`、`secret.facts` 或 `<TURTLE_HOST_CRITICAL>`。
3. 主持人不得进行谜面判定、公布线索、揭示汤底或暗示答案；它只是以主持人角色参与闲聊。
4. 闲聊发言不增加游戏回合，不改变便签和解题状态；公开记录可以标记为 `chat`，供后续角色看到，但不能当作主持人判定。
5. 闲聊模式关闭后，下一次玩家输入恢复正常的“伙伴批次 -> 主持人聚合回答”流程。

闲聊请求的输出也必须结构化且不包含解题字段：

```json
{
  "actor_id": "host|ai-1",
  "text": "朋友口吻的公开回应",
  "kind": "chat"
}
```

简单模式可以把伙伴闲聊合并为一次请求；主持人若使用独立模型或预设，则保持单独请求。只有在生成配置相同且实现明确支持时，才允许把主持人也并入合并请求；无论请求如何合并，所有启用角色都必须恰好产生一条闲聊回应。

### 4.4 中央便签墙与总结者

主持人完成本轮聚合回答后，进入一次 `summarizing` 阶段。前端从所有启用的伙伴中随机选择一个 `actorId` 作为总结者；没有伙伴时由主持人承担总结者角色。随机选择由前端完成并写入本轮内存状态，不能让模型自行选择，也不能因为重试而更换总结者。

总结者请求只接收汤面、公共历史、本轮伙伴发言、主持人公开回答和当前便签墙，不接收 `host_secret`。它必须返回“便签操作”而不是整面便签墙：

```json
{
  "round_id": "round-0003",
  "summarizer_id": "ai-2",
  "operations": [
    {
      "operation_id": "op-r3-01",
      "action": "add|update|remove|move",
      "note_id": "note-014",
      "category": "confirmedFacts|importantClues|coreQuestions|ruledOut",
      "text": "公开且经过本轮回答支持的内容",
      "source": "summarizer",
      "confidence": "confirmed|tentative"
    }
  ],
  "round_digest": "本轮公开信息的一句话摘要"
}
```

前端只接受合法类别、合法 `note_id`、非空文本和当前 `round_id` 的操作；重复 `operation_id`、未知便签、越权删除或模型返回整面墙时拒绝该批次，不直接覆盖现有状态。操作按顺序应用后递增 `noteWall.revision`，并记录 `lastUpdatedBy`。玩家可以新增、编辑、移动或删除便签；玩家编辑后的条目标记为 `source: player`，供模型知道它是玩家工作假设，而不是主持人确认的事实。玩家编辑不能改写汤底，也不能获得任何隐藏字段。

便签墙在每次伙伴请求和解题型主持人请求中都以高注意力公共区块注入：

```text
<TURTLE_PUBLIC_NOTE_WALL>
以下是所有玩家可见的工作便签，仅能作为公开线索和假设使用。
不要把玩家编辑的内容当作不可违背的真相；不可通过便签推断或泄露隐藏汤底。

CONFIRMED_FACTS: ...
IMPORTANT_CLUES: ...
CORE_QUESTIONS: ...
RULED_OUT: ...
</TURTLE_PUBLIC_NOTE_WALL>
```

总结完成后才进入 `awaiting-player`。便签总结不会自动触发下一轮，也不会写回酒馆聊天正文。

## 5. 提示词隔离模型

### 5.1 三个信息域

每次请求前先构建信息域，禁止在字符串拼装阶段临时决定可见性。

| 信息域 | 内容 | 主持人 | AI 伙伴 | 玩家界面 |
|---|---|---:|---:|---:|
| `public` | 汤面、规则、已公开问答、公开便签 | 是 | 是 | 是 |
| `host_secret` | 汤底、完整因果链、未公开事实 | 是 | 否 | 否 |
| `role_instruction` | 当前角色的人设、讨论风格和前端追加提示词 | 当前角色 | 当前角色 | 否 |

必须使用白名单投影：

```js
buildVisibleState(state, "host")
buildVisibleState(state, "peer", peerId)
```

不要使用 `JSON.stringify(state)` 后再尝试正则删除 `secret`。正则删除容易因嵌套字段、错误日志或新字段而泄漏。

### 5.2 汤底生成和主持人高注意区块

游戏开始时先由酒馆当前 AI 生成结构化题目：汤面、汤底、关键事实和可回答的判定标签。前端必须校验结果后才进入 `ready`，不能让主持人在汤底为空的情况下开始判定。

汤底生成结果只写入内存的 `state.secret`。之后每一次解题型主持人请求都重新构造专用的高注意区块，不能只依赖第一次请求的聊天历史。闲聊模式是明确例外：它使用不含汤底的 `host_chat` 提示词：

```text
<TURTLE_HOST_CRITICAL>
你是本局唯一主持人。以下内容是不可违背的真实汤底和因果事实。
你必须以这些事实为唯一判定依据，不得自行改写、补充或遗忘。
除非玩家已经猜中且前端允许揭晓，否则绝不能把本区块内容直接说出。

TRUTH: ...
FACTS:
- ...
</TURTLE_HOST_CRITICAL>
```

这个区块应放在主持人请求的专用 `system` 提示词中，并在任务提示中再次给出简短规则提醒。所谓“高注意”是提示词编排策略，不是模型级安全机制；真正的隔离仍由“伙伴请求不包含该区块”和前端白名单解析保证。

主持人可以使用独立的 `preset_name` 或 `custom_api`，伙伴可以使用另一组配置。适配器必须为每个角色解析自己的生成配置，不得依赖一个全局模型变量。

### 5.3 主持人提示词

主持人请求包含：

1. 从世界书读取的主持人人设。
2. 前端为主持人追加的请求级提示词；该内容只在本次游戏请求内生效。
3. `soup.surface` 和规则。
4. `secret.truth`、`secret.facts` 的高注意区块。
5. 当前玩家输入（问题、猜测或普通陈述）。
6. 本轮伙伴已经公开提出的全部问题，以及对应的 `question_id`。
7. `TURTLE_PUBLIC_NOTE_WALL` 高注意区块。
8. 已公开历史，不包含无关的调试字段。
9. 严格的结构化输出要求。

建议输出 JSON：

```json
{
  "protocol": "turtle-host-answer-v1",
  "round_id": "round-0003",
  "answers": [
    {"question_id":"round-0003-player-q01", "verdict":"no", "reply":"不是。"},
    {"question_id":"round-0003-ai-1-q01", "verdict":"yes|no|irrelevant|critical_yes", "reply":"给该问题的简短回答"}
  ],
  "reply": "面向玩家的本轮公开总结",
  "solved": false,
  "reveal": null
}
```

`reveal` 只有 `solved=true` 且前端确认规则允许时才能写入公开状态。即使模型返回了汤底，也要丢弃未授权字段，不能直接显示。

主持人聚合请求必须是本轮唯一的主持人解答请求；主持人不直接更新便签墙，便签由后续总结者根据公开回答更新。闲聊模式使用另一套 `host_chat` 输出协议，不得复用该解题协议。

为了避免简单模式中的问题错位，前端先把伙伴输出中的问题转成不可变问题账本，再把账本作为 JSON 和编号文本同时放入主持人提示词。若玩家最新输入本身是问题或猜测，也要先分配 `round-0003-player-q01` 这类 ID，和 AI 问题一起进入同一账本；普通闲聊或陈述不创建问题项：

```json
[
  {"question_id":"round-0003-player-q01", "speaker_id":"player", "text":"凶手认识死者吗？"},
  {"question_id":"round-0003-ai-1-q01", "speaker_id":"ai-1", "text":"门为什么是锁着的？"},
  {"question_id":"round-0003-ai-2-q01", "speaker_id":"ai-2", "text":"死者是否认识凶手？"}
]
```

主持人必须按原账本逐项返回 `answers`：每个 `question_id` 恰好一次，不能新增、删除、改写或合并问题；`answers` 的顺序也必须与账本一致。前端执行以下校验后才写入公开记录：`protocol` 和 `round_id` 正确、ID 集合完全相等、无重复 ID、无未知 ID、每项 `reply` 非空、`verdict` 属于白名单。任何校验失败都丢弃整批回答并允许使用同一账本重试，不能部分按位置写入，避免模型漏答导致后续答案整体错位。

### 5.4 伙伴提示词

伙伴请求只包含：

1. 从绑定世界书读取的伙伴人设和讨论策略。
2. 前端追加的角色提示词；只拼入伙伴自己的请求，不注入世界书，不写入酒馆正文，不影响主聊天提示词。
3. `public` 投影。
4. 玩家最新问题和主持人已公开判定。
5. 当前伙伴之前的公开发言。
6. `TURTLE_PUBLIC_NOTE_WALL` 高注意区块。

禁止包含：

- `secret.truth`。
- 主持人提示词原文。
- 未公开的主持人内部判断。
- 其他角色的 `role_instruction`、内部推理草稿或尚未显示的生成结果。

伙伴发言输出也必须带问题数组，不能让主持人从自然语言中猜测问题边界：

```json
{
  "actor_id": "ai-1",
  "text": "我认为门锁和时间线有关。",
  "questions": [
    {"local_question_id":"q01", "text":"门为什么是锁着的？"}
  ]
}
```

前端根据当前 `round_id`、`actor_id` 和数组顺序生成全局 `question_id`，例如 `round-0003-ai-1-q01`。简单模式的合并响应必须为每个 `actor_id` 返回独立的 `text` 与 `questions`，不能把多个角色的问题放进一个共享数组。

### 5.5 注入和 generateRaw 的边界

优先使用 `generateRaw` 的显式 `ordered_prompts` 构建隔离请求，只放入需要的角色提示词和公共状态。例如：

```js
await generateRaw({
  generation_id: requestId,
  ordered_prompts: [
    { role: "system", content: roleSystemPrompt },
    { role: "user", content: publicStatePrompt },
    "user_input"
  ],
  user_input: taskPrompt,
  should_stream: true,
  should_silence: true,
  max_chat_history: 0
});
```

实际字段以安装版本的类型定义为准；这里的关键不是示例代码本身，而是“显式指定提示词顺序，不自动带入整段聊天历史”。

`injectPrompts` 适用于需要暂时接入当前预设或触发世界书扫描的公共提示词：

- 使用唯一 ID，例如 `turtle.public.turn-0007`。
- 默认使用 `once: true`，避免提示词跨回合残留。
- 只注入 `public`，不注入 `host_secret`。
- 在 `finally` 中调用返回的 `uninject()`。
- 同时存在多个生成请求时，不使用共享的持久注入；用 `generateRaw` 的请求内提示词，避免并发污染。

### 5.6 真实模式的隔离

真实模式每个逻辑角色一个请求，主持人只在伙伴批次完成后调用一次：

```text
伙伴批次
  -> 伙伴请求（每个选中伙伴只看 public）并行
  -> 解析、排序、写入内存公开记录
  -> 主持人聚合请求（public + host_secret，1 次）
  -> 写入全部公开判定
  -> 总结者请求并应用便签操作
  -> 进入 awaiting-player
```

伙伴请求可以并行，但写入状态必须按 `turn` 和 `requestId` 校验，不能让后完成的旧请求覆盖新回合。

如果要求伙伴能看到其他伙伴刚刚的分析，应使用两阶段：先并行生成，再把已解析的公开发言作为下一轮公共上下文。不要把未解析的原始模型响应互相传递。

### 5.7 简单模式的合并请求

简单模式把本轮选中的多个伙伴合并成一次请求，主持人仍单独请求，因为主持人拥有秘密信息；主持人请求只在伙伴合并结果完成后执行一次。

```text
伙伴批次
  -> 伙伴合并请求（1 次，多个角色共用 public）
  -> 解析角色键
  -> 写入多个 peer 发言
  -> 主持人聚合请求（1 次，秘密隔离）
  -> 总结者请求并应用便签操作
```

合并请求必须使用稳定的角色 ID，而不是只使用姓名：

```json
{
  "speakers": [
    {"actor_id":"ai-1","text":"...","confidence":0.62},
    {"actor_id":"ai-2","text":"...","confidence":0.48},
    {"actor_id":"ai-3","text":"...","confidence":0.71}
  ]
}
```

合并提示词应包含：

- 一份共享公共状态，避免为每个角色重复发送历史。
- 每个角色一段独立的 `role_instruction` 人设块。
- 明确禁止角色之间互相代答。
- 规定每个 `actor_id` 必须恰好出现一次。
- 规定不输出 Markdown、解释、主持人秘密或额外角色。

简单模式要求的“完全隔离”定义为：所有伙伴都只接收汤面和公共历史；每个角色拥有独立的输出槽位；一个角色的未显示思维、草稿和模型中间推理不会传给另一个角色，也不会保存到公共状态。由于一次请求由同一个模型完成，角色提示词文本在模型输入层面并非密码学隔离，因此任何需要私密信息的角色都必须拆成独立请求；本游戏的伙伴角色不应拥有汤底或其他私密信息。

### 5.8 结果解析和拒绝策略

解析顺序：

1. `JSON.parse`。
2. 去除代码围栏后再次解析。
3. 兼容少量键名错误时进行白名单修复。
4. 仍失败则标记请求失败，不把原文直接当作游戏事件。

校验失败包括：缺少 `actor_id`、重复角色、未知角色、空文本、主持人返回伙伴字段、`solved` 与 `reveal` 矛盾。失败时保留原始响应在本地调试面板，但不写入公开记录。

## 6. 酒馆适配层

所有酒馆交互都必须从 `TavernAdapter` 进入，UI 和游戏逻辑不得直接调用全局函数。

### 6.1 能力探测

启动时生成：

```js
{
  iframe: true,
  tavernHelper: typeof window.TavernHelper !== "undefined",
  variables: typeof getVariables === "function" && typeof insertOrAssignVariables === "function",
  messages: typeof getChatMessages === "function",
  worldbook: typeof getWorldbook === "function",
  generation: typeof generateRaw === "function" || typeof generate === "function",
  events: typeof eventOn === "function" && typeof eventEmit === "function",
  nativeContext: !!window.SillyTavern
}
```

显示在 UI 的能力状态中，但不把探测结果作为游戏状态持久化。

### 6.2 推荐调用顺序

1. `TavernHelper` / 酒馆助手全局函数。
2. `SillyTavern.getContext()` 的稳定原生接口。
3. 纯浏览器 `localStorage` 和手动测试数据。
4. 最后才是父文档 DOM 查找或 `/send` 回退。

不要把 `parent.document.querySelector('#send_textarea')` 作为主路径。DOM ID 可能随酒馆版本、主题或扩展变化。

### 6.3 消息读取

统一封装：

```js
await adapter.getMessages({
  range: "0-",
  hideState: "all",
  includeSwipes: false
});
```

正常路径使用 `getChatMessages("0-", { hide_state: "all", include_swipes: false })`；需要分析未选中的消息页时才打开 `include_swipes`。结果归一化为：

```js
{ messageId, role, name, text, swipes, isHidden }
```

读取上下文时只取允许的文本字段，移除前端代码块、`<TURTLE_SECRET>`、调试标记和旧的内部 JSON。不能假设只有 `.message` 或只有 `.mes` 字段。

### 6.4 事件生命周期

初始化时注册：

- `tavern_events.MESSAGE_RECEIVED`：检测当前楼层是否有新汤面或游戏状态。
- `tavern_events.MESSAGE_UPDATED`：重新读取当前消息并刷新 UI。
- `tavern_events.CHAT_CHANGED`：卸载旧聊天绑定，加载新聊天状态，重新注入必要的公共提示词。
- `iframe_events.GENERATION_STARTED`、`STREAM_TOKEN_RECEIVED_FULLY`、`GENERATION_ENDED`：更新角色发言动画和流式文本。

所有监听器保存 stop 函数，并在 iframe 卸载或切换会话时清理。事件回调必须验证 `sessionId`，防止旧 iframe 或旧请求回调写入新聊天。

## 7. 持久化设计

### 7.1 作用域分配

| 数据 | 推荐作用域 | 原因 |
|---|---|---|
| UI 主题、动效、移动端视图 | `extension` 或 `global` | 不属于某一局游戏 |
| API 配置 | `extension` 或 `character` | 保存 provider、preset、model、base URL 等非秘密配置 |
| 角色人设、追加提示词和头像 | `chat` | 第一次编辑后按姓名写入当前聊天记录 JSON，只影响当前聊天文件 |
| 汤面、汤底、公开线索、回合记录 | 不持久化 | 初版只存在内存，刷新后清空 |
| 单条回合的解析结果 | 不持久化 | 为未来 `message` 分支恢复预留接口，不在初版启用 |
| API Key | 默认不持久化 | 优先使用酒馆当前预设；自定义 Key 仅内存使用 |
| 世界书绑定名称 | 不单独复制保存 | 启动时读取当前绑定世界书；角色资料复制到内存 |

### 7.2 初版持久化结构

初版只保存配置和角色编辑覆盖层，不保存 `GameState`。API 公共配置可使用 `extension`/`character` 作用域，角色编辑值必须写入当前 `chat` 变量：

```js
{
  aiTurtleConfig: {
    schemaVersion: 1,
    globalApi: { source: "tavern_preset", presetName: "in_use", model: "" },
    hostApi: { source: "tavern_preset", presetName: "", model: "" },
    peerApi: { "玛德琳": {}, "弗兰克": {} }
  },
  aiTurtleChatProfiles: {
    schemaVersion: 1,
    actors: {
      "埃利奥特": { role: "host", avatar: "", personaOverride: "", promptAppend: "" },
      "玛德琳": { role: "peer", avatar: "", personaOverride: "", promptAppend: "" },
      "弗兰克": { role: "peer", avatar: "", personaOverride: "", promptAppend: "" }
    }
  }
}
```

`actors` 和 `peerApi` 的索引键是角色姓名，因为初版输入协议以姓名作为身份标识，暂不处理同名。运行时仍需为每个角色生成稳定的内部 `actorId`，防止简单模式 JSON 解析时仅凭姓名写错槽位。角色第一次在前端编辑后立即写入 `chat.aiTurtleChatProfiles.actors[角色姓名]`；下次在同一聊天文件读取同名角色时复用头像、`personaOverride` 和人设追加词。API 选择可继续存放在公共配置，也可按姓名覆盖，但 API Key 不进入聊天变量。

其中 `promptAppend` 是前端追加提示词，只对本前端发起的游戏请求生效。它不能通过 `injectPrompts` 注入酒馆当前聊天，也不能修改世界书条目内容。

角色资料合并顺序：绑定世界书的姓名匹配结果作为基础资料，当前 `chat` 中同名角色的已编辑字段作为覆盖层，最后复制到本局内存。未编辑过的角色不必立即写入聊天 JSON。配置读取顺序：酒馆助手变量 -> 当前页面内存 -> `localStorage` 仅作为无酒馆测试回退。API Key 不进入上述持久化对象。

### 7.3 未来状态持久化预留

未来若需要恢复游戏过程，可以在不改变 UI 结构的情况下增加以下聊天变量命名空间：

```js
{
  aiTurtle: {
    schemaVersion: 1,
    sessionId: "...",
    mode: "simple",
    soup: { title: "", surface: "" },
    actors: { host: {}, peers: [] },
    publicState: { noteWall: { confirmedFacts: [], importantClues: [], coreQuestions: [], ruledOut: [], revision: 0 } },
    turns: [],
    checkpoint: { turn: 7, status: "awaiting-player", updatedAt: "..." },
    $meta: { lastRequestIds: [], migrationVersion: 1 }
  }
}
```

初版不写入该对象。未来启用时仍应使用 `schemaVersion`、checkpoint 和迁移函数。文档说明 `{{format_xxx_variable::...}}` 会忽略 `$` 开头的键，可用 `$meta` 保存不希望被宏直接带入提示词的元数据。但这不是安全边界；真正的安全边界仍是 `PromptPolicy` 的白名单投影。

### 7.4 写入策略

- 公共配置及 `chat.aiTurtleChatProfiles` 保存采用 `insertOrAssignVariables` 或 `updateVariablesWith`，避免覆盖其他扩展的变量。
- 每次保存前校验 `schemaVersion` 和角色 ID。
- 不保存游戏回合、流式 token、汤底或 AI 原始响应。
- 前端设置修改后立即保存；游戏状态修改只更新内存 store。
- API Key 只保存在当前 iframe 的内存引用中。
- 未来启用聊天状态持久化时，再引入 checkpoint、上限和迁移函数。

### 7.5 启动恢复

```text
读取能力 -> 读取配置变量 -> 校验/迁移 -> 读取当前绑定世界书
         -> 构建空白内存 GameState -> 由当前 AI 生成新题目
         -> 重建 ViewModel -> 显示 ready 状态
```

刷新后不恢复上一局游戏过程。若页面检测到最新消息中的 `<Puzzle>`，只把其中的角色名单作为新游戏初始化输入，不把旧回合自动恢复到内存。

## 8. YAML 与世界书输入

### 8.1 首选 `<Puzzle>` 输入协议

iframe 启动时从当前聊天上下文读取最新可见消息，提取首个完整的 `<Puzzle>...</Puzzle>` 区块。区块内容是 YAML，顶层必须包含 `游戏数据栏` 对象：

```text
<Puzzle>
游戏数据栏:
  主持人: "埃利奥特"
  玩家1: "玛德琳"
  玩家2: "弗兰克"
</Puzzle>
```

解析结果：

```js
{
  "游戏数据栏": {
    "主持人": "埃利奥特",
    "玩家1": "玛德琳",
    "玩家2": "弗兰克"
  }
}
```

解析器必须把 `主持人` 映射为本局 `actors.host.name`，把按自然排序得到的 `玩家1`、`玩家2` 等键映射为 `actors.peers[].name`。玩家键允许从 `玩家1` 连续到 `玩家N`；缺号、空姓名、重复姓名或出现未知的游戏数据栏字段时显示诊断错误，不应静默创建角色。主持人和伙伴随后按姓名同时读取绑定世界书基础资料与当前聊天持久化覆盖层，并按 7.2 节的优先级合并。

题目汤面和汤底不从该输入块读取。加载角色后，由酒馆当前 AI 生成本局汤面和汤底；汤底只写入内存 `state.secret`，并在每次主持人请求时重新放入高注意区块。

### 8.2 兼容输入与解析原则

可选支持 `<TURTLE_GAME>` 或配置指定的旧标签，仅用于手动测试或已有扩展的结构化题目载荷；它不能覆盖有效的 `<Puzzle>` 角色名单。若兼容已有角色卡，允许读取 `<Combat_block>`，但必须通过配置指定标签名，并在解析前限制来源为当前消息、当前楼层或明确绑定的世界书条目。

1. 先提取标签，再使用可用的 YAML 解析器解析；不能用正则直接解析嵌套 YAML。
2. 标签缺失、重复、交叉嵌套或 YAML 顶层不是对象时拒绝。
3. `<Puzzle>` 必须有 `游戏数据栏`，其中必须有一个 `主持人`，并至少有一个连续编号的 `玩家N`。
4. 角色姓名必须是非空字符串；初版不处理同名，重复姓名直接阻止启动。
5. 世界书主要作为角色人设资料来源；读取绑定世界书后，只复制匹配角色的必要字段到内存，不把整本世界书送进每回合提示词。
6. 世界书角色条目建议包含专用区块：

   ```text
   <TURTLE_ROLE>
   actor_id: ai-1
   name: ...
   persona: ...
   avatar: ...
   prompt_append: ...
   </TURTLE_ROLE>
   ```

7. 前端 `promptAppend` 可以在本地覆盖或追加角色请求规则，但不能回写世界书、不能注入酒馆正文、不能影响主聊天提示词。
8. 世界书条目匹配顺序为：精确 `entry.name`，再匹配显式别名/`entry.keys`；默认关闭双向包含匹配，避免误命中。未找到时使用空白默认人设，并在对应角色编辑区显示警告；不得因为缺少人设而把整本世界书发送给模型。

## 9. UI 与应用层映射

| UI 区域 | 读取 | 写入/动作 |
|---|---|---|
| 汤面栏 | `state.soup.surface` | 载入题目时更新 |
| 主持人座位 | `actors.host`、当前发言 | 发言开始/结束时更新 |
| AI 座位 | `actors.peers`、角色状态 | 伙伴请求结果更新 |
| 便签墙 | `publicState.noteWall` 四类数组 | 只接受已校验的总结者操作；玩家编辑项带有 `source: player` 标记 |
| 对话记录 | `turns` 的 public 子集 | 提问、判定、讨论提交后刷新 |
| 输入栏 | 当前玩家输入 | `submitPlayerQuestion()` |
| API 弹窗 | 当前会话配置 | 不默认写 Key；模型偏好可写 `extension` |
| 人设弹窗 | 世界书角色资料 + `promptAppend` | 头像和追加提示词保存配置；不修改世界书 |

UI 渲染必须使用 `textContent` 或统一 `escapeHtml`，不能把模型原文直接拼进 `innerHTML`。模型返回的气泡、便签和记录都视为不可信文本。

## 10. 游戏结束与酒馆续写

游戏过程不需要逐条发回酒馆。玩家破汤、主动结束或主持人判定结束后，执行一次总结流程：

```text
冻结内存 GameState
  -> 选取公开回合、公开判定和最终结果
  -> 总结请求
  -> 校验 summary JSON
  -> 组装干净的续写提示
  -> 一次性注入当前酒馆生成
  -> 触发当前 AI 继续续写
```

### 10.1 总结请求

总结请求可以使用主持人模型或单独的 summary 模型配置，但它不是伙伴讨论请求，不能复用伙伴合并提示词。输入包括：

- 汤面。
- 已公开的提问、判定和伙伴发言。
- 游戏结果：`solved`、`abandoned` 或 `timeout`。
- 允许揭晓的汤底范围：只有玩家已经破汤时才加入完整汤底；未破汤时只加入“不公开汤底”的结果说明。

建议输出：

```json
{
  "summary": "给酒馆后续剧情使用的事实性总结",
  "continuation_instruction": "请在不使用游戏术语的前提下自然续写",
  "state_update": ["允许写入正文的角色状态变化"]
}
```

总结失败时不发送半成品；允许玩家重试总结，不重跑已经完成的游戏回合。

### 10.2 续写提示词边界

续写只接收清洗后的总结和必要的剧情要求，不接收：

- 主持人高注意汤底区块。
- 伙伴角色追加提示词。
- 世界书原文。
- API 配置或调试数据。
- 前端内部 JSON。

推荐使用一次性的 `injectPrompts` 注入一个公共续写区块，然后调用酒馆生成。注入完成后必须执行返回的 `uninject()`；如果当前版本无法可靠触发生成，则回退为向酒馆输入框填入文本，最后才使用 `/send`。续写注入是游戏结束动作，不能在普通回合中使用。

### 10.3 续写结果

续写结果由酒馆当前 AI 写入普通聊天楼层。前端不再解析该楼层为游戏回合，也不把续写文本重新注入伙伴请求。这样可以保持游戏提示词与正文提示词的边界：前端角色追加提示词只影响本游戏的请求，不影响世界书和后续正文。

## 11. 请求调度、去重与故障恢复

### 11.1 请求上下文

每次生成携带：

```js
{
  sessionId,
  turn,
  actorId,
  mode,
  requestId,
  promptVersion,
  startedAt
}
```

### 11.2 调度规则

- 同一 `sessionId + turn` 只允许一个主持人请求。
- 简单模式的伙伴请求可有一个合并请求；真实模式的伙伴请求可并行 N 个。
- 新问题提交时，若旧回合仍在生成，默认禁用输入；允许用户点击停止后再提交。
- 结束回调先检查 `sessionId`、`turn`、`requestId`，不匹配则丢弃。
- 使用 `stopGenerationById(requestId)` 停止单个请求，离开页面时使用 `stopAllGeneration()` 作为兜底。

### 11.3 失败恢复

主持人聚合失败：不写入 `host_answer`，状态回到 `awaiting-player`，允许用户手动重试当前批次；重试不得重新抽取伙伴发言概率。

部分伙伴失败：保留成功伙伴发言，失败角色显示可重试状态；不要重发已成功角色。

JSON 解析失败：显示“模型格式错误”，保留 debug 原文，不把它当成公开发言。

酒馆助手不可用：进入手动模式；允许加载本地 YAML 和模拟回复，但明确标记“未连接酒馆”。

## 12. 安全和兼容性

### 12.1 密钥

酒馆助手文档明确提醒自定义脚本可能接触聊天记录和 API Key。因此：

- 默认使用 `generate` / `generateRaw` 走酒馆当前 API 配置。
- 不把 Key 放入 `chat`、`message`、世界书或 HTML 属性。
- UI 的自定义 Key 只保存在内存；若必须保存，需单独取得用户确认，并明确其风险。
- debug 日志永不打印完整请求头、Key、汤底或完整提示词。

### 12.2 提示词泄漏

- 伙伴请求使用白名单投影。
- 伙伴角色不拥有汤底或其他私密事实；如果未来增加角色私密字段，发现该字段时自动拆分请求或拒绝简单模式。
- 不把主持人原始响应写入公共聊天记录。
- 发送到酒馆的公开记录不包含 `<TURTLE_SECRET>`。

### 12.3 版本兼容

兼容性矩阵：

| 能力 | 主路径 | 兼容回退 |
|---|---|---|
| 前端显示 | 酒馆助手 iframe 渲染 | 独立浏览器手动打开 |
| 变量 | 酒馆助手变量 API | `localStorage` 仅存 UI 偏好 |
| 聊天上下文 | `getChatMessages` | `SillyTavern.getContext().chat` |
| 生成 | `generateRaw` | `generate`；最后才是直接 `fetch` |
| 流式显示 | `iframe_events` | 等待 Promise 完成后一次性显示 |
| 事件 | `eventOn` / `eventOnce` | 低频轮询当前消息，仅作为测试回退 |
| 世界书 | `getWorldbook` | 手动导入题目 YAML |
| 发送内容 | `eventEmit`/生成 API | DOM 输入框或 `/send` 仅回退 |

启动时将能力探测结果显示在设置或诊断面板中。不要因为某个接口缺失而让整个 UI 白屏。

## 13. 实施顺序

1. **适配层骨架**：实现能力探测、变量读写、消息读取、事件订阅和统一错误类型。
2. **状态模型**：实现 schema 校验、内存状态、版本迁移接口和请求生命周期。
3. **YAML/世界书输入**：实现 `<Puzzle>`、兼容 `<TURTLE_GAME>`/`<Combat_block>`、按姓名加载世界书角色资料和持久化配置。
4. **谜面与主持人请求**：先公开汤面；实现一次聚合回答全部伙伴问题的 `host_answer` 请求和 secret 隔离。
5. **真实模式伙伴请求**：按 50% 抽样后每角色独立请求、按队列顺序显示、停止和重试。
6. **简单模式合并请求**：按抽样队列进行伙伴多角色 JSON 输出、角色 ID 校验、部分失败恢复。
7. **闲聊模式**：实现无汤底的主持人和伙伴一次性朋友口吻回应，并确保不改变游戏状态。
8. **UI 接线**：把模拟 `hostEvaluate`、`playerSpeak`、`triggerSpeakerBubble` 接到 ViewModel 和应用层。
9. **事件和配置持久化强化**：聊天切换、消息变更、配置加载、事件清理测试；游戏过程刷新清空。
10. **兼容性测试**：酒馆助手完整环境、缺少世界书、缺少事件、无生成接口、独立浏览器手动模式。

## 14. 验收标准

### 提示词隔离

- 在伙伴请求的最终提示词和日志中不存在 `secret.truth`、完整汤底或主持人私密响应。
- 解题型主持人请求能看到汤底，伙伴请求看不到；闲聊型主持人请求同样看不到汤底。
- 简单模式检测到私密角色内容时自动拆分或拒绝合并。

### 游戏流程

- 主持人先公开谜面；随后每轮先进行伙伴批次，再由主持人用一次聚合请求回答全部伙伴问题。
- 真实模式对抽中的伙伴生成 N 个独立请求；简单模式生成 1 个伙伴合并请求。
- 一次玩家动作最多产生一批伙伴发言；每个启用伙伴在该批次最多发言一次。
- 每个伙伴每轮以 50% 概率进入发言队列；抽样结果在重试时保持不变。
- 伙伴批次完成、超时或停止后必须回到 `awaiting-player`；伙伴发言、建议问题和模型完成回调都不能自动开启下一批。
- AI 建议的问题只能作为可点击 UI，不得自动代替玩家提交。
- 闲聊模式下主持人和所有伙伴各回应一次，不触发主持人解题，不注入汤底，也不消耗游戏回合。
- 主持人回答完成后必定进入一次总结者阶段；有伙伴时随机选择一个伙伴，无伙伴时使用主持人。
- 总结者只能返回便签操作，前端按 `round_id`、`operation_id`、类别和便签 ID 校验后应用，不能直接覆盖整面便签墙。
- 玩家可编辑便签；编辑项必须标记为 `source: player`，并在后续 AI 提示词中与总结者确认事实区分显示。
- 下一轮所有伙伴提示词都包含 `TURTLE_PUBLIC_NOTE_WALL` 高注意力区块。
- 主持人聚合回答必须覆盖问题账本中的每个 `question_id` 恰好一次；漏答、重复或未知 ID 时整批拒绝，不允许按数组位置错位写入。
- 重试不会复制已成功的公开事件。
- 最终猜测成功后才显示汤底揭晓。

### 持久化

- 刷新 iframe 后只恢复 API 配置、角色头像和角色人设；汤面、汤底、便签和公开记录均清空。
- 切换聊天不会把上一局状态写入下一局。
- 变量缺失、旧版本或损坏时可降级到新会话，不白屏。

### 兼容性

- 酒馆助手完整 API 可用时，不依赖父页面 DOM。
- 生成接口不可用时，页面显示可操作的诊断信息。
- `getChatMessages` 的 Promise/同步返回差异由适配层统一处理。
- UI 在桌面和移动端保留现有 `stage/dialogue` 切换逻辑。

## 15. 已确认的架构决策

以下决策已经确认，后续施工必须以此为准：

1. 汤面和汤底由酒馆当前 AI 生成；汤底保存在当前 iframe 内存，并在每一次解题型主持人请求的高注意区块中重新注入；闲聊型主持人请求明确不注入汤底。
2. 主持人允许使用独立模型或预设；伙伴也可以使用另一组模型或预设。
3. 玩家和伙伴只能看到汤面及公共历史；真实模式逐角色请求，简单模式一次请求多个角色，但不传递任何角色内部思维或未显示草稿。
4. 游戏过程不逐条写回酒馆；结束时由 AI 总结并触发酒馆继续续写。
5. 角色人设从绑定世界书读取，前端允许追加请求级提示词；追加提示词不能影响世界书和正文。
6. 初版不支持消息页/swipe 分支恢复；游戏状态只存在内存，刷新即清空。后续可以在预留的 schema 和适配层上增加恢复。
7. 酒馆助手 iframe 前端必须交付为单个 HTML 文件，所有逻辑通过该 HTML 内的命名空间分区组织。
8. 初始角色名单来自当前上下文中的 `<Puzzle>` YAML：`游戏数据栏.主持人` 和连续的 `游戏数据栏.玩家N`；角色人设按姓名匹配持久化配置及绑定世界书，暂不处理同名。
9. AI 讨论采用有限批次：玩家是每轮唯一触发者，简单模式一次合并多个角色，真实模式逐角色请求；任何模式都禁止自动递归对话。
10. 每轮伙伴 AI 独立以 50% 概率发言；抽样后按队列轮流显示，主持人只用一次请求集中回答本轮所有伙伴问题。
11. 输入栏的闲聊模式只对当前输入生效；主持人和所有伙伴各回应一次，主持人暂时不可见汤底，闲聊不触发判定、不改变游戏回合。
12. 每轮主持人回答之后由随机总结者更新四类中央便签；便签为公开工作资料，玩家可编辑但编辑项必须保留来源标记。
13. 主持人聚合回答使用不可变 `question_id` 账本和严格集合校验，简单模式不得通过自然语言位置猜测问题对应关系。

在进入实际编码前，仍需确认的只是具体 UI 细节和题目 YAML/世界书条目的最终字段名；上述架构边界不再改变。
