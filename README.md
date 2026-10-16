# Skills & Rules

个人 Cursor Agent 偏好仓库。按用途分两类：

| 类型 | 是什么 | 落盘到哪 |
|------|--------|----------|
| **Skill** | 特定场景才启用的做法/流程（碰到才跟） | `~/.cursor/skills/<name>/SKILL.md` |
| **Rule** | 做某类事时要一直遵守的约束 | 项目 `.cursor/rules/*.mdc`（或用户 Rules） |

---

## Skills

| 目录 | 简介 |
|------|------|
| [BYOK](BYOK/) | 密钥只放 `localStorage`：扁平键名（如 `gh-<app>-repo` / `gh-<app>-token`）、Settings UI、启动时自动读取 |

入口文件：`SKILL.md`

## Rules

| 目录 | 简介 |
|------|------|
| [UIUX](UIUX/) | 界面偏好：简洁勿增实体、圆角克制、敏感操作二次确认、自定义 Modal/Toast（禁用 alert）、一步完成少拆 UI、UI 不写技术细节 |

入口文件：`rules.md`（拷到项目时可用 `.mdc`）
