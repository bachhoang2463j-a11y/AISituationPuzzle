# AISituationPuzzle · AI 海龟汤

> 基于 `UI_design.html` 壁炉圆桌界面，在 SillyTavern + 酒馆助手 iframe 中运行的海龟汤前端。
> 目标架构见 [SPEC.md](./SPEC.md)，分轮施工计划见 [TurtleSoup_DEVELOPMENT_PLAN.md](./TurtleSoup_DEVELOPMENT_PLAN.md)。

## 现状

- **可运行产物**：`TurtleSoup.html`（单 HTML）已按计划第 1+2+2A+3+4+5+6+7+8+9+10+11 轮建成并经用户确认（2026-09-11）——完整视觉基线；三级主持人调度；`<Puzzle>` 名单（0~N 伙伴）；`host-answer-v2` 聚合判定；真实/简单双模式；四阶段回合与中央便签墙；输入级闲聊模式（chat-reply-v1 / chat-batch-v1，无汤底不判定）。
- **测试基线**：`integration-test/harness.html` 共 79 项断言全绿。
- **视觉基线**：`UI_design.html`（2652 行）为唯一视觉标准，只读不改。
- **废弃实现**：旧 `TurtleSoup.html` 已归档至 `备份（无需阅读）/TurtleSoup.html`，不再进入版本跟踪。
- **下一轮目标**：第 12 轮（配置持久化与事件生命周期）——API/头像/人设持久化（酒馆变量优先），CHAT_CHANGED 清空重建，API Key 不持久化。

## 目录结构

```
AISituationPuzzle/
├── TurtleSoup.html                      # 交付产物（单 HTML，楼层渲染器加载）
├── UI_design.html                       # 视觉基线（只读）
├── SPEC.md                              # 目标架构与验收标准
├── TurtleSoup_DEVELOPMENT_PLAN.md       # 0-13 轮分轮计划（含 2A 轮）
├── Coding rule.md                       # 施工与日志规范
├── LOG.md                               # 施工日志
├── LOG-INDEX.md                         # 日志索引
├── integration-test/                    # 回归验收 harness 与夹具
│   ├── harness.html
│   ├── README.md
│   └── fixtures/
├── 备份（无需阅读）/                    # 已废弃实现归档（不跟踪）
└── .gitignore
```

## 运行

- **酒馆内**：将 `TurtleSoup.html` 全文贴入楼层消息的 html 代码块，由酒馆助手楼层渲染器加载（源码已遵守围栏纪律，内部无三连反引号）。
- **独立浏览器**：直接打开 `TurtleSoup.html`；在 API 弹窗配置 OpenAI 兼容接口后由真实 LLM 主持判定，未配置时进入本地 Mock 演示模式。
- **回归验证**：起本地静态服务器后访问 `http://127.0.0.1:8123/AISituationPuzzle/integration-test/harness.html`（服务器命令见 `integration-test/README.md`），断言不全绿不得报告任务完成。

## 开发约定

- 重大架构变更先确认再编码。
- 游戏过程只存内存，刷新清空；仅 API 配置、头像、角色人设可持久化。
- 每轮施工结束追加 `LOG.md` + `LOG-INDEX.md`，按 `Coding rule.md` 两步提交法回填 Hash。

## 远端

- GitHub: https://github.com/bachhoang2463j-a11y/AISituationPuzzle
