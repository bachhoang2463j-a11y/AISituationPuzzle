# AISituationPuzzle · AI 海龟汤

> 基于 `UI_design.html` 壁炉圆桌界面，在 SillyTavern + 酒馆助手 iframe 中运行的海龟汤前端。
> 目标架构见 [SPEC.md](./SPEC.md)，分轮施工计划见 [TurtleSoup_DEVELOPMENT_PLAN.md](./TurtleSoup_DEVELOPMENT_PLAN.md)。

## 现状

- **视觉基线**：`UI_design.html`（2652 行）为唯一视觉标准，包含圆桌舞台、5 座位气泡、中央四类便签墙、右侧记录筛选、移动端 Tab、人设/API/设置弹窗。
- **废弃实现**：`TurtleSoup.html` 已被判定为无效设计，彻底抛弃，归档至 `备份（无需阅读）/TurtleSoup.html`，不再进入版本跟踪（已加入 `.gitignore`）。
- **下一轮目标**：按开发计划第 1 轮 + 第 2 轮重建 —— 先恢复视觉基线，再在无酒馆环境下跑通本地 Mock 问答闭环。

## 目录结构

```
AISituationPuzzle/
├── UI_design.html                      # 视觉基线（只读）
├── SPEC.md                             # 目标架构与验收标准
├── TurtleSoup_DEVELOPMENT_PLAN.md      # 0-13 轮分轮计划
├── Coding rule.md                      # 施工与日志规范
├── LOG.md                              # 施工日志
├── LOG-INDEX.md                        # 日志索引
├── 备份（无需阅读）/                    # 已废弃实现归档（不跟踪）
│   └── TurtleSoup.html
└── .gitignore
```

## 运行

当前无可运行的前端产物。重建完成后，交付物仍为单 HTML 文件，直接由酒馆助手 iframe 加载；在独立浏览器中也可打开进入 Mock 降级模式。

## 开发约定

- 重大架构变更先确认再编码。
- 游戏过程只存内存，刷新清空；仅 API 配置、头像、角色人设可持久化。
- 每轮施工结束追加 `LOG.md` + `LOG-INDEX.md`，按 `Coding rule.md` 两步提交法回填 Hash。

## 远端

- GitHub: https://github.com/bachhoang2463j-a11y/AISituationPuzzle
