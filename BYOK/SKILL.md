---
name: byok
description: >-
  BYOK via localStorage: flat keys gh-<app>-repo / gh-<app>-token (and
  similar), Settings UI, auto-read on app init for pre-injected secrets.
  Use when the user mentions BYOK, 自带密钥, localStorage, gh-token, gh-repo,
  PAT, API key, 提前注入, or settings for user-supplied keys.
---

# BYOK（localStorage）

本用户默认：**密钥只放 `localStorage`，扁平键名，启动时自动读**（常会事先在控制台/别处注入）。

不管业务 CRUD / IndexedDB。

## 存储

- **只用 `localStorage`**（Web、GitHub Pages、以及本用户的插件设置页也优先同一套，除非用户改口）
- **Never** 写入仓库、`.env` 真密钥、前端打包进源码、或塞进 Actions 给浏览器用

## 键名（必须遵守）

Flat，一段一段用 `-` 连接，**不要** JSON 大对象塞多个 secret。

### GitHub（repo + token 成对，常和 BYOK 一起出现）

键名里字段就叫 **`repo`** / **`token`**（再加 `gh-` 与项目简称前缀）：

```text
gh-<app>-repo     # 例: gh-rpost-repo
gh-<app>-token    # 例: gh-rpost-token
```

`<app>` = 项目简称（小写短词：`rpost`、`publish`、`papermate`）。用户若指定简称，**原文用**。

**`repo` 的取值：** 一整段 `owner/name`，例如 `shalom-lab/repo1`、`shalom-lab/r-post`。  
不要拆成两个 key；Settings 单框；调 API 时再 `split('/')`。

**`token`：** 用户自备 PAT（BYOK）。有写仓库需求时，**repo 与 token 一起**配置、一起 bootstrap 读取。

### 其他 API Key

同一风格：

```text
<provider>-<app>-key
# 例: deepseek-rpost-key
# 例: openai-papermate-key
```

若用户给出确切键名，**禁止改写**。

### README

同一变更里写明全部键名，并注明：「可 Settings 填写，也可事先写入 localStorage；启动会自动读。」

## 初始化：自动读取

App / 插件 UI **启动时（bootstrap）** 立刻从 `localStorage` 读入内存/状态：

1. 读 `gh-<app>-repo`、`gh-<app>-token` 及本产品需要的其他 key
2. 有值 → Settings 显示「已保存」（可遮罩末 4 位），业务可直接用，**勿强迫先打开设置页**
3. 无值 → 显示「未设置」，需要调用 API 时再引导去 Settings
4. 不要在启动流程里弹窗打断；静默读取即可

伪代码约定：

```js
const app = "rpost"; // 项目简称
const repo = localStorage.getItem(`gh-${app}-repo`) ?? ""; // "owner/repo"
const token = localStorage.getItem(`gh-${app}-token`) ?? "";
// hydrate settings store / React state — then render
```

Settings 里 repo 也是**单输入框**（placeholder：`owner/repo`），不要拆两个字段。

## Settings UX

- 密码型输入 + 保存 / 清除（清除二次确认）
- 保存：`localStorage.setItem`；清除：`removeItem`
- 只展示状态，不展示完整 secret
- 可选：用量/额度紧挨密钥区

## 使用密钥

- 仅本机读出后直连目标 API（GitHub Contents、DeepSeek 等）
- GitHub 写回：用 `gh-<app>-repo` + `gh-<app>-token`；401/403 中文提示查 scope / 仓库名
- **禁止**把 token 传到第三方中转

## Agent behavior

1. 新产品需要 Key/Token → 先定 `<app>` 简称，再按上表建键名 + bootstrap 自动读
2. 可与 `chrome-mv3-extension` / `vite-github-pages` 叠用
3. 业务数据存哪不在本 skill 范围

## Out of scope

- IndexedDB / 实体 CRUD
- CloudBase 登录
- 加密 localStorage、跨设备同步（除非明确要求）
