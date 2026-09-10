# integration-test · 集成测试 harness

针对 `TurtleSoup.html`（酒馆助手 iframe 单 HTML 产物）的回归验收层：不进真酒馆，把真实产物挂进 iframe、mock 酒馆助手接口、跑断言列表。属于 IAB 验证层；布局动画、rAF 时序等仍需排障模式（真 Chrome）过一遍。

## 运行方式

在项目所在盘的项目集合目录起静态服务器（root 指向 `D:/Project`）：

```bash
node -e "const http=require('http'),fs=require('fs'),path=require('path'),url=require('url');const root='D:/Project';const t={'.html':'text/html; charset=utf-8','.js':'text/javascript','.json':'application/json','.css':'text/css'};http.createServer((q,s)=>{const f=path.join(root,decodeURIComponent(url.parse(q.url).pathname));fs.readFile(f,(e,d)=>{if(e){s.writeHead(404);s.end();return;}s.writeHead(200,{'Content-Type':t[path.extname(f).toLowerCase()]||'application/octet-stream'});s.end(d);})}).listen(8123,'127.0.0.1',()=>console.log('ready'))"
```

浏览器打开：<http://127.0.0.1:8123/AISituationPuzzle/integration-test/harness.html>，点击「运行全部断言」，等待全部 ✓。

**断言不全绿不得报告任务完成**；红了先修，修不了如实报告失败项。

## 断言分组

| 分组 | 对应开发轮次 | 内容 |
|---|---|---|
| 第 0 轮 · 测试基线与围栏纪律 | 第 0 轮 | 产物可读取；源码无三连反引号（楼层围栏纪律）；夹具可独立读取与解析 |
| 第 1 轮 · 启动与视觉结构 | 第 1 轮 | 无白屏、顶栏/圆桌/便签墙/五座位/五气泡/记录区/输入栏齐全、无控制台错误、无水平溢出、三弹窗开关、移动 Tab 切换 |
| 第 2 轮 · 本地 Mock 问答闭环 | 第 2 轮 | 玩家气泡与记录即时更新、Mock 主持人延迟判定、空输入不新增、防并发、Enter 发送、重置恢复初始 demo 状态 |

后续轮次按「先跑 harness 回归 → 全绿 → 记 LOG」的流程，把对应轮次验收标准逐条翻译成本文件 harness.html 中的新断言（新开分组，不改动已通过断言的语义）。

## 夹具

- `fixtures/puzzle.yaml`：固定 `<Puzzle>` 输入（`游戏数据栏.主持人` + `玩家1~3`，对齐 SPEC 8.1），第 5 轮 Puzzle 读取使用。
- `fixtures/mock-data.json`：最小 Mock 响应数据（汤面 `soup.surface`、主持人三判定、伙伴发言，字段对齐 SPEC 第 4 章），第 4 轮 MockAdapter 的种子数据。

## mock 酒馆桥

harness 把产物以 `srcdoc` 挂进 iframe，并在 `<head>` 注入 mock 酒馆接口（`getVariables`/`replaceVariables`/`generateRaw`/`eventOn` 等，内存实现 + 调用计数）与错误转发、`confirm`/`alert` 放行。第 1+2 轮产物不调用这些接口；第 4 轮 TavernAdapter 接入后，可用 `window.__tavernMock.calls` 断言调用次数（如"数据不经 LLM"特性断言 `generateRaw` 调用为 0）。

## UI_design.html 关键 DOM 清单（第 1 轮验收对照表）

| 区域 | 选择器 / ID | 说明 |
|---|---|---|
| 顶部栏 | `#soup-brief-text`、`.top-actions .top-action-btn` ×3 | 汤面摘要；API / 设置 / 全屏 |
| 移动 Tab | `#tab-btn-stage`、`#tab-btn-dialogue`、`#main-layout[data-mobile-view]` | stage / dialogue 切换 |
| 圆桌舞台 | `#table-arena`、`.svg-scene-background`、`.photo-scene-bg` | SVG 圆桌 + 背景图（外链失败回退渐变） |
| 便签墙 | `#memo-board`、`#sticky-notes-cluster`、`[data-note-type]` ×4 | confirmed（黄）/ clue（绿）/ question（粉）/ excluded（蓝） |
| 座位 | `#seat-host`、`#seat-ai-1~3`、`#seat-player`（`data-char-id`） | 主持人 / 三伙伴 / 玩家 |
| 气泡 | `#bubble-host`、`#bubble-ai-1~3`、`#bubble-player` | 各含 `.bubble-role/.bubble-verdict/.bubble-content` |
| 输入栏 | `#player-input`、`#btn-send` | 发送 / Enter 发送 |
| 记录区 | `#dialogue-list`、`#record-counter`、`.filter-tab[data-filter]` ×3 | 全部 / 裁判判定 / 我的提问 |
| 弹窗 | `#global-api-modal-overlay`、`#settings-modal-overlay`、`#persona-modal-overlay` | API / 设置（含头像库）/ 人设编辑 |

CDN 依赖策略（已确认）：保留 Google Fonts 与 Unsplash 外链，但全部 `<img>` 有 `onerror="avatarFallback(this)"` 本地 SVG 占位回退，背景图有渐变回退层；离线环境不裂图、不白屏。
