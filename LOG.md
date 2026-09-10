# 施工日志

## 2026-08-20：完成海龟汤前端架构设计

- 变更行为：读取 `UI_design.html`、`Coding rule.md` 和酒馆助手文档；新增 `SPEC.md`。
- 涉及文件：`SPEC.md`。
- 决策原因：明确单 HTML iframe 约束、AI 生成汤底、主持人高注意秘密注入、真实/简单模式的多角色请求策略、世界书角色人设读取、配置持久化范围以及游戏结束后的总结续写流程。
- 用户已确认的边界：游戏过程只存在内存，刷新清空；API 配置、头像和角色人设持久化；伙伴不接收汤底或角色内部思维；简单模式一次调用扮演多个角色；前端追加提示词不影响世界书和正文。

## 2026-08-20：补充 `<Puzzle>` 输入与有限讨论回合

- 变更行为：将 `<Puzzle>` YAML 设为首选启动输入，从 `游戏数据栏.主持人` 和连续的 `游戏数据栏.玩家N` 自动建立本局角色名单。
- 角色加载：角色按姓名匹配绑定世界书；首次编辑后的头像、人设追加词写入当前聊天的 `aiTurtleChatProfiles` JSON，暂不处理同名。
- 回合控制：新增有限伙伴批次和 `peer-batch -> host-answer -> awaiting-player` 状态机；每次玩家动作最多触发一批伙伴发言，禁止 AI 发言递归开启下一批，建议问题只能由玩家点击后提交。

## 2026-08-20：确认伙伴批次、50%发言概率与闲聊模式

- 流程调整：主持人先公开谜面；每轮伙伴按队列各最多发言一次；主持人只发起一次聚合请求回答本轮全部伙伴问题；随后等待玩家输入。
- 发言抽样：每个启用伙伴独立以 50% 概率进入本轮队列，抽样结果写入内存并在重试时保持不变；真实模式逐角色请求，简单模式合并请求。
- 闲聊模式：输入栏开关只影响当前输入；主持人和所有伙伴各回应一次，暂时移除主持人汤底高注意区块，闲聊不触发解题、不改变游戏回合。

## 2026-08-20：补充中央便签墙与聚合回答对齐协议

- 便签更新：主持人完成本轮聚合回答后，随机选择一个伙伴作为总结者；无伙伴时由主持人总结。总结者返回结构化便签操作，前端校验后更新已确认事实、重要线索、核心疑问和已排除信息。
- 玩家编辑：便签支持玩家新增、编辑、移动和删除；玩家编辑项保留 `source: player`，与总结者确认内容区分，且不具备修改汤底的权限。
- 提示词：`TURTLE_PUBLIC_NOTE_WALL` 作为公共高注意区块加入所有伙伴和解题型主持人请求。
- 聚合回答：伙伴问题先建立不可变 `question_id` 账本；主持人必须按账本逐项返回，前端要求 ID 集合完全相等且每项恰好一次，错误时整批拒绝，避免简单模式回答错位。

## 2026-08-20：生成单文件可运行前端

- 涉及文件：新增 `TurtleSoup.html`，保留单 HTML 约束，内嵌样式、界面、状态管理和酒馆助手适配层。
- 实现内容：读取 `<Puzzle>`、按姓名加载世界书/聊天角色资料、生成汤面汤底、主持人秘密区块、真实/简单伙伴模式、50% 发言抽样、主持人单次聚合回答、随机总结者便签操作、玩家便签编辑和输入级闲聊模式。
- 兼容策略：优先 `generateRaw`，回退 `generate + injects`；支持 `getChatMessages`、`getWorldbook`、`getChatWorldbookName`、`getCharWorldbookNames`、变量读写、请求 ID 和停止生成；无酒馆接口时进入手动降级显示。
- 安全边界：汤底只在内存和解题型主持人请求中使用；闲聊模式不注入汤底；API Key 仅运行时使用，不写入持久化变量。

## 2026-09-11：废弃无效实现并初始化仓库

- 变更行为：初始化 git 仓库，分支 `main` 关联 `https://github.com/bachhoang2463j-a11y/AISituationPuzzle` 并推送首轮提交 `302025b`；将无效的 `TurtleSoup.html` 移入 `备份（无需阅读）/` 并加入 `.gitignore` 不再跟踪；新增 `README.md` 作为现状说明，确认以 `UI_design.html` 为唯一视觉基线。
- 涉及文件：`.gitignore`、`README.md`、`备份（无需阅读）/TurtleSoup.html`、`.git/`。
- 决策原因：用户确认 `TurtleSoup.html` 为无效设计需彻底抛弃；按 `Coding rule.md` 先建立可追溯的 git 基线，再按 `TurtleSoup_DEVELOPMENT_PLAN.md` 第 1+2 轮重建，避免在错误布局上继续叠加功能。
- 远端：`origin/main` 已同步至 GitHub，首轮 8 文件 4232 行。

## 2026-09-11：完成第 0+1+2 轮——测试基线、视觉恢复与本地 Mock 闭环

- 变更行为：补做第 0 轮，新建 `integration-test/`（`harness.html` 断言框架、`fixtures/puzzle.yaml`、`fixtures/mock-data.json`、`README.md` 含 DOM 对照清单），mock 酒馆接口注入骨架（内存实现 + 调用计数）与围栏断言就位；第 1 轮，复制 `UI_design.html` 全文为新 `TurtleSoup.html`（唯一起点，未重写），保留全部 DOM/CSS/弹窗/响应式，补便签 `data-note-type` 四类与 `#memo-board`/`#sticky-notes-cluster` 钩子，CDN 保留外链并加本地回退（`avatarFallback` SVG 占位、背景图渐变下层）；第 2 轮——建立最小 `GameState`（phase/soup.surface/records/bubbles/request），`playerSpeak` 与 `triggerSpeakerBubble` 接状态更新，`hostEvaluate` 改造为 `MockAdapter.hostEvaluate`（Promise + 1200ms 延迟），发送防并发，`clearDialogueHistory` 升级为完整重置（恢复初始 demo 记录与气泡）。
- 涉及文件：`TurtleSoup.html`（新增）、`integration-test/harness.html`、`integration-test/fixtures/puzzle.yaml`、`integration-test/fixtures/mock-data.json`、`integration-test/README.md`、`TurtleSoup_DEVELOPMENT_PLAN.md`（第 0 节补围栏约束、第 1 轮补 CDN 决策、第 5 轮补无 `<Puzzle>` 默认名单、第 6 节更新为第 3 轮指引）、`SPEC.md`（13 节注明施工顺序以分轮计划为准）。
- 决策原因：旧实现的核心缺陷是视觉偏离与无自动化验收。本轮以复制 `UI_design.html` 为唯一起点杜绝"参照重写"，以 harness 24 项断言（围栏纪律 / 启动与视觉结构 / Mock 闭环三组）把每轮验收从人肉检查变为可回归；交付路径经用户确认为楼层消息渲染器，故产物源码严禁三连反引号并加 harness 断言锁定；CDN 依赖经确认采取保留外链 + 本地回退策略。
- 验收结果：IAB 中 harness 24/24 断言全绿；桌面 4:1 布局（stage 972 / dialogue 243）与窄屏 860px 断点（移动 Tab 显示、stage/dialogue 视图切换、无水平溢出）几何检查通过；`node` 语法校验与围栏计数（0）通过。待用户真机（真酒馆楼层）导入确认。
- HASH：`8bf2c52`

## 2026-09-11：完成第 2A 轮——纯网页直连 LLM 主持人

- 变更行为：经用户确认调整轮次顺序（先纯网页跑通 LLM 连接，不先接酒馆），计划文档插入第 2A 轮定义、原第 3 轮顺延。`TurtleSoup.html` 新增 `DirectApiAdapter`（OpenAI 兼容 `/chat/completions` 直连，`response_format: json_object`，`verdict` 值域校验 `yes|no|irrelevant|critical_yes`），与 `MockAdapter` 同接口；发送时按 `globalApiConfig.baseUrl` 调度，未配置自动回退 Mock；`GameState` 增加内存汤底 `secret.truth`，仅进入主持人请求 system 消息的 `<TURTLE_SECRET>` 区块；请求失败写入可重试的系统错误记录；`critical_yes` 判定标签接入气泡；全局 API 默认 Base URL 改为空，避免默认配置误触发直连。harness 新增第 2A 断言组（8 项：请求协议/鉴权/模型、system 汤底区块、判定写入、公共 DOM 无汤底、请求体无 Key、critical_yes、失败可重试、清空配置回退 Mock 零直连），并把固定 sleep 等待全部改为带超时轮询。
- 涉及文件：`TurtleSoup.html`、`integration-test/harness.html`、`TurtleSoup_DEVELOPMENT_PLAN.md`、`README.md`。
- 决策原因：用户 2026-09-11 指示"先纯网页跑通 LLM 连接，不用先接入酒馆"。该路径对应 SPEC 12.3 兼容矩阵"生成：`generateRaw` → `generate`；最后才是直接 `fetch`"的预留回退级，属既定架构内的提前实现而非架构变更。调试过程发现并修复 harness 两处自身缺陷：mock fetch 桥漏写 return（responder 的 Promise 被丢弃，请求漏到真实网络）、断言等待目标少算与 IAB timer 节流不兼容——产物代码零返工。
- 验收结果：IAB 中 harness 32/32 断言全绿（上轮 24 项无回归）。真实 LLM 连接待用户配置自有 API 真机确认。
- HASH：`6e30f05`

## 2026-09-11：完成第 3 轮——状态模型与 View/Action 分离

- 变更行为：`TurtleSoup.html` 建立单向流程 `Action -> Store -> View`：新增 `GameAction` 枚举与 `dispatch()` 作为唯一状态变更入口（提交问题 / 主持人回答 / 主持人失败 / 重置四类动作），`render()` 统一渲染入口（记录列表 + 气泡，幂等可重复调用），气泡内容改为 `renderBubbles()` 从 `GameState.bubbles` 或初始快照渲染，发言动画拆为纯 View 副作用 `triggerSpeakingFx()`；`GameState` 增加 `schemaVersion`、`turn` 回合计数，`request` 对象化（`id`/`type`/`startedAt`/`abortRequested`，重置即放弃进行中请求，迟到响应被丢弃）；发送入口加阶段守卫（`phase !== awaiting-player` 或请求进行中均拒绝）；`bindInputEvents` 加防重复绑定防护；新增 `serializeState()` 调试序列化（不含汤底与 API 配置，不触发酒馆接口）。删除 `playerSpeak`/`hostRespond`/`reportHostError`/`resetGame`/`triggerSpeakerBubble`（逻辑全部并入 dispatch 与 View 层）。harness 新增第 3 轮断言组 5 项。
- 涉及文件：`TurtleSoup.html`、`integration-test/harness.html`。
- 决策原因：按计划第 3 轮把 UI 渲染与游戏状态分开，为第 4 轮 TavernAdapter 接入（届时状态层不再被重写）打地基；纯重构轮，不新增用户功能、不改 Mock 与直连问答的外部行为，既有 32 项断言即回归锁。
- 验收结果：IAB 中 harness 37/37 断言全绿（上轮 32 项零回归；新增：防重复绑定、render 幂等不丢输入、序列化含 phase/turn/records 且不含汤底与 Key、turn 随发送递增、请求进行中重置后迟到响应被丢弃）。
- HASH：`9e7b854`

## 2026-09-11：完成第 4 轮——TavernAdapter 能力探测与适配层

- 变更行为：`TurtleSoup.html` 新增 `TavernAdapter`（启动时探测变量/消息/生成/事件/停止五项能力，`available` 以生成能力为最低要求；`getChatMessages`/`getVariables`/`generate`/`stop`/`on` 方法统一 Promise/同步差异与错误格式，缺失时抛错由调用方回退，不白屏）；`hostEvaluate` 走 `generateRaw` 的 `ordered_prompts`（system 汤底区块 + user 问题，`should_silence: true` 后台静默生成，不占用酒馆停止按钮），复用与直连相同的 `parseHostAnswer` 校验；主持人适配器升级为三级调度 `pickHostAdapter()`：显式配置 Base URL 时直连优先 > 酒馆助手 > 本地 Mock；设置弹窗新增"运行环境诊断"区块（能力 ✓/✗ 与当前模式），顶部徽章 title 同步；暴露 `window.TurtleApp` 调试接口（`diagnose`/`serializeState`）。harness 将 mock 桥拆分为基础桥与酒馆桥（full/partial 两种注入模式），`mountApp` 参数化，新增第 4 轮断言组 6 项。
- 涉及文件：`TurtleSoup.html`、`integration-test/harness.html`。
- 决策原因：按计划第 4 轮把酒馆函数集中到适配层（游戏逻辑不再直接调用 window 下的酒馆函数）；调度优先级定为"显式配置 > 酒馆环境 > 本地演示"，依据 SPEC 2.1.7（独立 API 配置用于覆盖环境默认）——用户在 API 弹窗的显式配置意图应优先于酒馆当前预设。API 形状依据酒馆助手开发文档（`generateRaw` 的 `ordered_prompts`/`generation_id`/`should_silence`、`stopGenerationById`、`eventOn`）。
- 验收结果：IAB 中 harness 43/43 断言全绿（上轮 37 项零回归；新增：完整接口探测全可用、generateRaw 恰好一次且 system 含汤底/user 含问题/静默生成、直连优先于酒馆、generateRaw 失败可重试、仅变量接口时回退 Mock 不白屏、设置弹窗诊断文本）。真酒馆环境待用户导入确认。
- HASH：`cde0deb`

## 2026-09-11：完成第 5 轮——Puzzle 输入与角色名单

- 变更行为：`TurtleSoup.html` 实现 `<Puzzle>` 输入协议（SPEC 8.1）完整加载链：启动时经 `TavernAdapter.getChatMessages` 从最新消息往回提取 `<Puzzle>` 区块（单消息多块、多消息多块均拒绝）→ YAML 解析（优先酒馆助手 `window.YAML`，无则用仅支持 `游戏数据栏` 两级结构的受限行式解析器）→ `validateCast` 校验（主持人非空、`玩家N` 从 1 连续、空名/重名/未知字段报错、超过 3 名伙伴截断并警告）→ 合法名单经 `dispatch(APPLY_CAST)` 应用（座位名与 `charactersData` 同步更新、新局清空记录与回合），非法或缺失时回退内置默认名单并在设置弹窗诊断区显示可操作错误；`render()` 增加 `renderSoupBar`（汤面栏从 `GameState.soup` 渲染）。汤面汤底本轮使用固定题目路径，Tavern 生成留待第 6 轮。harness：`mountApp` 支持 `chatMessages` 预设（走真实启动链路），夹具角色名改为 埃利奥特/玛德琳/弗兰克/朵拉（与默认名单区分保证断言分辨力），新增第 5 轮断言组 6 项。
- 涉及文件：`TurtleSoup.html`、`integration-test/harness.html`、`integration-test/fixtures/puzzle.yaml`。
- 决策原因：汤面汤底选择固定题目路径（计划允许"固定题目响应或 Tavern 生成"二选一），避免本轮同时引入谜题生成协议与角色名单两个风险面；开发中发现并修复 `validateCast` 初版的真缺陷——while 循环遇缺号即停导致玩家 1+3 漏检编号断裂，改为先收集 `玩家N` 键、排序后校验连续性，并补上 SPEC 8.1 要求的未知字段检查。
- 验收结果：IAB 中 harness 49/49 断言全绿（上轮 43 项零回归；新增：夹具合法名单应用且记录清空、名单生效后主持人请求可见汤底而公共 DOM 无汤底、缺号/重名回退默认名单并诊断报错、超 3 名截断并警告、无 `<Puzzle>` 默认名单不白屏）。真酒馆楼层带 `<Puzzle>` 消息的场景待用户真机确认。
- HASH：`9c34935`

## 2026-09-11：完成第 6 轮——真实主持人问答 host-answer-v1

- 变更行为：`TurtleSoup.html` 将主持人响应协议正式化为 `host-answer-v1`：提示词要求模型返回 `{"protocol":"host-answer-v1","question_id":"...","verdict":"yes|no|irrelevant|critical_yes","answer":"..."}`，`parseHostAnswerV1` 严格校验协议字段、question_id 匹配、verdict 值域与 answer 非空，任何不符整条拒绝并落入可重试的系统错误记录；建立玩家问题 `question_id` 账本（`q-N` 递增，玩家提问与判定记录共享同一 ID，存入 `GameState.request` 与记录项）；新增 60 秒请求超时（`TurtleApp.debug.hostTimeoutMs` 测试钩子可缩短）；新增停止能力：请求进行中发送按钮切换为"停止生成"，点击后本地立即回到可操作状态（`HOST_ABORTED` 动作）并尽力中断底层生成（酒馆路径 `stopGenerationById`、直连路径 `AbortController.abort()`），停止后可立即重试；迟到响应由第 3 轮的请求丢弃机制自动作废。MockAdapter 同步升级为返回 v1 格式。harness 全部 mock responder 升级为动态提取真实 question_id 的 v1 格式，新增第 6 轮断言组 5 项。
- 涉及文件：`TurtleSoup.html`、`integration-test/harness.html`、`README.md`（第 5 轮经用户确认后更新现状）。
- 决策原因：计划第 6 轮定义。停止入口选择"请求进行中发送按钮兼作停止按钮"（不改视觉基线结构，仅切换文案）；超时阈值定为 60 秒（补上计划评审时遗留的量化项）。调试中发现并修复一处 TDZ 缺陷——`HOST_TIMEOUT_MS` 原定义在脚本后部，而 `TurtleApp` 字面量在执行期立即取值导致整段脚本中断，移至前部后恢复。
- 验收结果：IAB 中 harness 54/54 断言全绿（上轮 49 项零回归；新增：停止按钮状态切换与 stopGenerationById 调用、v1 判定与提问共享 question_id、ID 不匹配整条拒绝可重试、缺 answer 字段拒绝、超时回到可操作且可重试）。
- HASH：`44edc56`

## 2026-09-11：完成第 7 轮——单个 AI 伙伴与最小多角色状态机

- 变更行为：`TurtleSoup.html` 引入三阶段回合状态机 `awaiting-player -> peer-thinking -> host-thinking -> awaiting-player`：玩家发送后先由 ai-1 伙伴独立请求一次公开发言（`peer-speech-v1` 协议：`actor_id` 校验 + text 非空），发言作为提问入公开记录并分配独立 `question_id` 加入本轮账本；伙伴失败不阻塞（系统记录后直接进入主持人阶段）。主持人协议升级为 `host-answer-v2` 聚合形态：一次请求的 `answers` 数组必须覆盖本轮账本每个 `question_id` 恰好一次（漏答、重复、未知 ID、verdict 非法、answer 空——任一不符整批拒绝），逐条写入判定记录，气泡展示玩家问题的判定。三个适配器（Mock/Direct/Tavern）均新增 `peerEvaluate` 并将 `hostEvaluate` v2 化；伙伴提示词只含汤面、公开记录与角色人设，绝不接触汤底；停止覆盖两阶段（`<回合ID>-peer` / `<回合ID>-host` 双 ID 停止 + AbortController 跨阶段复用）。harness 全部 responder 升级为智能 v2 版（按 system 提示词区分 peer/host、从 user 提示词动态提取账本），既有断言计数按"每次发送 = 伙伴+主持人两次请求、完整回合 4 条记录"调整，新增第 7 轮断言组 4 项。
- 涉及文件：`TurtleSoup.html`、`integration-test/harness.html`。
- 决策原因：计划第 7 轮验证最小多角色状态机。主持人协议直接做 v2 聚合（SPEC 4.2 终态方向），避免 v1 单答案到多答案的二次迁移成本；伙伴发言固定为"提出一个有助于解题的问题"并入账本，使聚合校验在第 7 轮即可完整落地。调试中修正两处 harness 断言自身问题：第 2 轮"重复点击防并发"断言的语义过时（第 6 轮起请求中点击已是停止语义，更新断言验证"重复点击停止回合且不产生第二个回答"）、组 6 伪造 ID 断言的 responder 替换时机过晚（Tavern mock 微任务级响应，改为发送前预设）。产物代码零返工。
- 验收结果：IAB 中 harness 58/58 断言全绿（上轮 54 项零回归；新增：完整回合顺序与伙伴气泡、伙伴请求 system+user 双向无汤底、伙伴失败跳过后主持人仍回答玩家问题、停止后回到 awaiting-player 且无自动续轮）。
- HASH：`4c8e4ab`

