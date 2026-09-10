# AISituationPuzzle · AI 海龟汤

> 基于 `UI_design.html` 壁炉圆桌界面，在 SillyTavern + 酒馆助手 iframe 中运行的海龟汤前端。
> 目标架构见 [SPEC.md](./SPEC.md)，分轮施工计划见 [TurtleSoup_DEVELOPMENT_PLAN.md](./TurtleSoup_DEVELOPMENT_PLAN.md)。

## 现状

- **可运行产物**：`TurtleSoup.html`（单 HTML）已按计划第 1+2 轮重建并经用户确认（2026-09-11）——完整恢复 `UI_design.html` 视觉基线（4:1 圆桌布局、五座位气泡、四类便签墙、三弹窗、移动端 Tab），独立浏览器中本地 Mock 问答闭环可用。
- **LLM 直连**：第 2A 轮已实现（待真机确认）——全局 API 弹窗配置 OpenAI 兼容接口（Base URL / API Key / Model / Temperature）后，主持人为真实 LLM 判定（结构化 JSON 输出，汤底只进入主持人请求的 system 消息）；未配置或清空 Base URL 时自动回退本地 Mock 裁判；请求失败显示可重试错误。
- **测试基线**：`integration-test/harness.html` 共 32 项断言（围栏纪律 / 启动与视觉结构 / Mock 闭环 / LLM 直连协议与隔离）全绿。
- **视觉基线**：`UI_design.html`（2652 行）为唯一视觉标准，只读不改。
- **废弃实现**：旧 `TurtleSoup.html` 已归档至 `备份（无需阅读）/TurtleSoup.html`，不再进入版本跟踪。
- **下一轮目标**：第 3 轮（状态模型和 View/Action 分离），之后第 4 轮接入 TavernAdapter。

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
