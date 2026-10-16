---
name: byok
description: >-
  BYOK via localStorage: flat keys gh-<app>-repo / gh-<app>-token (and
  similar), Settings UI, auto-read on app init for pre-injected secrets.
  Use when the user mentions BYOK, 自带密钥, localStorage, gh-token, gh-repo,
  PAT, API key, 提前注入, or settings for user-supplied keys.
---

# BYOK（localStorage）

默认：**密钥及配置只存** `localStorage`**，扁平键名，启动自动读取**。支持用户事先通过控制台等方式注入。

适用于 Web、GitHub Pages、浏览器插件设置页。

## 存储与键名

- 统一使用 `localStorage`，每项配置独立存储，不合并为 JSON 大对象。
- `<app>` 为项目简称（如 `rpost`、`publish`、`papermate`），用户指定时原样使用。
- 用户指定的完整键名不得修改。

**GitHub：**

```
gh-<app>-repo     # owner/repo
gh-<app>-token    # GitHub PAT
gh-<app>-path     # 可选，仓库内相对路径
```

- Repo 和 Token 成对配置；Repo 使用单个 `owner/repo` 输入框，不拆成两个 Key。
- Path 仅在业务需要时增加，可为空；只检查基本格式，**不检查 GitHub 上路径是否存在**，允许后续创建。
- Path 可独立修改，不必重新验证未变更的 Repo / Token。

**其他 API Key：**

```
<provider>-<app>-key
# deepseek-rpost-key
# openai-papermate-key
```

## 初始化：自动读取

- 启动（bootstrap）立即从 `localStorage` 读取当前产品需要的全部配置并加载到应用状态。
- 有值显示「已保存」，无值显示「未设置」；已有配置可直接用于业务。
- **启动只读取，不验证连接、不重复写入、不弹窗、不阻塞页面。**
- 提前注入的配置与 Settings 保存的配置一视同仁。
- 如果遇到报错toast提示

## Settings UX

- Repo：单输入框，placeholder `owner/repo`。
- Token / API Key：密码型输入框，可遮罩显示末 4 位，不展示完整密钥。
- Path：按需提供普通输入框，支持仓库内相对目录。
- 仅提供「保存」「清除」，**不要单独设置「测试连接」按钮**。
- 清除前二次确认，确认后 `removeItem` 并同步更新状态。
- 可选：密钥区域附近显示额度、用量。

**点击保存：**

1. 按钮 loading，检查输入格式。
2. Repo + Token：通过官方 GitHub API 验证认证、仓库访问及业务所需权限。
3. 其他 API Key：使用低成本、尽量无副作用的官方 API 验证可用性。
4. Path：**仅检查格式，不请求 API 验证远程路径是否存在**。
5. 全部适用检查通过后才 `setItem`，更新状态并提示成功。
6. 失败不写入新配置、不覆盖旧值，中文提示错误并结束 loading。
7. 仅修改 Path 时，无需重新验证未修改的密钥。

## 使用密钥

- 仅本机读取后直连目标官方 API（GitHub Contents、DeepSeek 等）。
- 不在每次业务请求前重复测试连接；实际请求结果可用于更新连接状态。
- GitHub 写回使用 `gh-<app>-repo` + `gh-<app>-token`，需要时结合 `gh-<app>-path`。
- 401/403 中文提示检查 Token、权限（scope）、仓库访问或额度；超时提示重试。
- **API 失败不得自动删除或覆盖 localStorage 配置。**
- Path 不存在时由业务逻辑处理，不影响配置保存。
- 禁止通过第三方中转传输 Token，不在 URL、日志或界面暴露完整密钥。

## 安全要求

- 禁止把真实密钥写入仓库、`.env`、前端源码、构建产物或 Actions 后注入浏览器。
- 使用 HTTPS，优先最小权限 Token。
- `localStorage` 不加密，注意 XSS 风险；不同域名及浏览器扩展与网页的存储通常不共享。

## README

同一变更中列明全部实际使用的键名、用途及必要权限，并注明：

> 可在 Settings 填写，也可事先写入 localStorage；启动自动读取，不重新验证。手动保存密钥时先验证连接，Path 只检查格式、不检查远程目录是否存在。

## Agent behavior

1. 新产品需要 Key / Token → 先确定 `<app>` 简称，再按规范创建扁平键名。
2. GitHub Repo / Token 成对配置；Path 按需增加，不强制所有项目使用。
3. 必须实现 bootstrap 自动读取，已有配置启动即用。
4. 仅用户主动保存时验证；Path 只做格式检查，修改 Path 不触发无关验证。
5. 连接失败保留旧配置；清除必须二次确认。
6. 同步更新 README，不擅自更改键名。
7. 可与 `chrome-mv3-extension`、`vite-github-pages` 叠用。
8. 业务数据存储不在本规范范围。

## Out of scope

- IndexedDB / 实体 CRUD
- CloudBase 登录
- 加密 localStorage、跨设备同步（除非明确要求）

**核心原则：启动直接读，密钥验证后保存，Path 只检查格式，业务直接用，失败不清除。**

