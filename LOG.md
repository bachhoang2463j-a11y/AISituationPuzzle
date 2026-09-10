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

