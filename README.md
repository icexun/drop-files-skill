# DropFiles Skill

[中文](#中文) | [English](#english)

---

<a id="english"></a>

## English

An AI agent skill (compatible with [TRAE](https://trae.ai/), Claude, Cursor, and any agent that loads the `SKILL.md` frontmatter convention) that uploads generated HTML or Markdown content to the DropFiles service and returns a public shareable link. The service is free, globally accessible, and hosted on Cloudflare's edge network.

### Web Interface

Prefer a manual upload without an agent? Visit the DropFiles website directly to upload HTML or Markdown content and get a shareable link through your browser — no account required.

**Website**: <https://dropfiles.tokendealhub.com/>

### Features

- One-shot upload of HTML or Markdown content to get a public URL
- Automatic content-type detection (`html` / `markdown` / `auto`)
- Configurable expiry: 1 hour, 1 day, 7 days, or 30 days
- Optional password protection (user-supplied or auto-generated 6-digit code)
- Optional document title shown on the share page
- Public links work without an account for viewers
- HTML content is sanitized server-side (scripts stripped) for safety
- Markdown rendered with GitHub-flavored styling (code blocks, tables, images)

### Installation

Copy the `SKILL.md` file into your agent's skill directory, or clone this repository and reference it from your workspace. Works with any agent that supports the `SKILL.md` frontmatter convention (TRAE, Claude, Cursor, etc.).

```bash
git clone https://github.com/icexun/drop-files-skill.git
```

### Usage

Invoke this skill whenever you want to:

- Share generated HTML or Markdown content via a public link
- Publish a document, report, or page and get a URL to distribute
- Upload any text-based artifact (HTML/Markdown) for online viewing
- Generate a shareable preview link for content the agent just produced

#### Example: Upload Markdown (no password)

```bash
curl -X POST https://api.xiongan-company.com/api/v1/upload \
  -H "Content-Type: application/json" \
  -d '{
    "content": "# Hello World\n\nThis is a shared Markdown document.",
    "type": "markdown",
    "expire": "7d",
    "title": "My Document"
  }'
```

#### Example: Upload with a user-supplied password

```bash
curl -X POST https://api.xiongan-company.com/api/v1/upload \
  -H "Content-Type: application/json" \
  -d '{
    "content": "# Secret Report\n\nConfidential content.",
    "type": "markdown",
    "expire": "1d",
    "password": "user_provided_password"
  }'
```

#### Example: Upload with an auto-generated 6-digit password

```bash
curl -X POST https://api.xiongan-company.com/api/v1/upload \
  -H "Content-Type: application/json" \
  -d '{
    "content": "# Protected Document\n\nContent here.",
    "type": "markdown",
    "expire": "7d",
    "password": "482917"
  }'
```

### API Reference

- **Base URL**: `https://api.xiongan-company.com`
- **Upload**: `POST /api/v1/upload`
- **Metadata**: `GET /api/v1/info/{id}`
- **Preview**: `https://dropfiles.tokendealhub.com/v/{id}`

#### Upload Request Body

| Field      | Type   | Required | Description |
|------------|--------|----------|-------------|
| `content`  | string | Yes      | The HTML or Markdown text to upload. Max 5MB. |
| `type`     | string | No       | `html`, `markdown`, or `auto` (default: `auto`, auto-detects from content). |
| `expire`   | string | No       | `1h`, `1d`, `7d`, or `30d` (default: `7d`). |
| `title`    | string | No       | Optional title shown on the share page. |
| `password` | string | No       | Optional password to protect the share page. See Password Policy below. |
| `lang`     | string | No       | `en`, `zh`, or `auto` (default: `auto`). |

#### Success Response (HTTP 201)

```json
{
  "success": true,
  "data": {
    "id": "abc123def456",
    "url": "https://dropfiles.tokendealhub.com/v/abc123def456",
    "preview_url": "https://dropfiles.tokendealhub.com/v/abc123def456?preview=1",
    "expire_at": "2026-07-22T10:30:00.000Z",
    "type": "markdown",
    "size": 42
  }
}
```

#### Error Response (HTTP 400/404/429/500)

```json
{
  "success": false,
  "error": {
    "code": "CONTENT_REQUIRED",
    "message": "Content is required"
  }
}
```

Common error codes:

- `CONTENT_REQUIRED` — `content` field is empty.
- `CONTENT_TOO_LARGE` — content exceeds 5MB.
- `INVALID_TYPE` — `type` is not one of `html`, `markdown`, `auto`.
- `INVALID_EXPIRE` — `expire` is not one of `1h`, `1d`, `7d`, `30d`.
- `RATE_LIMIT_EXCEEDED` — too many requests from the same IP.

### Password Policy

The default behavior is **no password** — the share link is publicly accessible by anyone with the URL.

Password protection is opt-in and supports three modes:

1. **No password (default)**: Omit the `password` field entirely, or send `null`. The share page is publicly readable.
2. **User-supplied password**: If the user explicitly provides a password, pass it in the `password` field. The share page will prompt viewers for this password.
3. **Skill-generated 6-digit password**: If the user asks for password protection but does not provide a password, the skill generates a random 6-digit numeric code (e.g. `"482917"`) and includes it in the `password` field. The generated password is communicated back to the user so they can share it with intended viewers.

#### 6-Digit Password Generation Rule

- Exactly 6 digits, each 0–9.
- Format: a string of 6 numeric characters, e.g. `"482917"`.
- Do NOT use letters, symbols, or shorter/longer lengths.
- Always display the generated password to the user in the response, e.g.: `Generated password: 482917`.

### Limits

- Max content size: **5 MB** (measured as UTF-8 bytes).
- Max expiry: **30 days**.
- Rate limiting applies per IP; if you hit `RATE_LIMIT_EXCEEDED`, wait and retry.

### Notes

- The share link is public (unless password-protected). Anyone with the URL can view the content.
- Viewers do not need an account.
- HTML content is sanitized: `<script>` tags and event handlers are removed.
- Markdown supports standard GitHub-flavored Markdown including code blocks, tables, and images.
- The `preview_url` adds `?preview=1` and is useful when embedding the link in a context that should not inflate the access counter on initial load.

### License

MIT

---

<a id="中文"></a>

## 中文

一个 AI agent 技能（兼容 [TRAE](https://trae.ai/)、Claude、Cursor 等任何支持 `SKILL.md` frontmatter 约定的 agent），用于将生成的 HTML 或 Markdown 内容上传到 DropFiles 服务，并返回一个公开可访问的分享链接。该服务免费、全球可访问，并托管在 Cloudflare 的边缘网络上。

### 网页端

不想通过 agent 上传？直接访问 DropFiles 网站，在浏览器中手动上传 HTML 或 Markdown 内容并获取分享链接，无需注册账号。

**网站地址**：<https://dropfiles.tokendealhub.com/>

### 功能特性

- 一键上传 HTML 或 Markdown 内容，获取公开链接
- 自动识别内容类型（`html` / `markdown` / `auto`）
- 可配置过期时间：1 小时、1 天、7 天或 30 天
- 可选密码保护（用户自定义或自动生成 6 位数字密码）
- 可选文档标题，在分享页面显示
- 访问者无需账号即可查看公开链接
- HTML 内容在服务端进行净化处理（移除脚本）以确保安全
- Markdown 使用 GitHub 风格样式渲染（代码块、表格、图片）

### 安装

将 `SKILL.md` 文件复制到你的 agent 技能目录，或克隆本仓库并在工作区中引用。兼容任何支持 `SKILL.md` frontmatter 约定的 agent（TRAE、Claude、Cursor 等）。

```bash
git clone https://github.com/icexun/drop-files-skill.git
```

### 使用方法

遇到以下场景时调用此技能：

- 通过公开链接分享生成的 HTML 或 Markdown 内容
- 发布文档、报告或页面，并获取可分发的 URL
- 上传任何基于文本的内容（HTML/Markdown）以便在线查看
- 为刚刚生成的内容生成可分享的预览链接

#### 示例：上传 Markdown（无密码）

```bash
curl -X POST https://api.xiongan-company.com/api/v1/upload \
  -H "Content-Type: application/json" \
  -d '{
    "content": "# Hello World\n\nThis is a shared Markdown document.",
    "type": "markdown",
    "expire": "7d",
    "title": "My Document"
  }'
```

#### 示例：使用用户自定义密码上传

```bash
curl -X POST https://api.xiongan-company.com/api/v1/upload \
  -H "Content-Type: application/json" \
  -d '{
    "content": "# Secret Report\n\nConfidential content.",
    "type": "markdown",
    "expire": "1d",
    "password": "user_provided_password"
  }'
```

#### 示例：使用自动生成的 6 位数字密码上传

```bash
curl -X POST https://api.xiongan-company.com/api/v1/upload \
  -H "Content-Type: application/json" \
  -d '{
    "content": "# Protected Document\n\nContent here.",
    "type": "markdown",
    "expire": "7d",
    "password": "482917"
  }'
```

### API 参考

- **基础地址**：`https://api.xiongan-company.com`
- **上传**：`POST /api/v1/upload`
- **元数据**：`GET /api/v1/info/{id}`
- **预览**：`https://dropfiles.tokendealhub.com/v/{id}`

#### 上传请求体

| 字段       | 类型   | 必填 | 说明 |
|------------|--------|------|------|
| `content`  | string | 是   | 要上传的 HTML 或 Markdown 文本，最大 5MB。 |
| `type`     | string | 否   | `html`、`markdown` 或 `auto`（默认：`auto`，根据内容自动识别）。 |
| `expire`   | string | 否   | `1h`、`1d`、`7d` 或 `30d`（默认：`7d`）。 |
| `title`    | string | 否   | 可选标题，在分享页面显示。 |
| `password` | string | 否   | 可选密码，用于保护分享页面。详见下方密码策略。 |
| `lang`     | string | 否   | `en`、`zh` 或 `auto`（默认：`auto`）。 |

#### 成功响应（HTTP 201）

```json
{
  "success": true,
  "data": {
    "id": "abc123def456",
    "url": "https://dropfiles.tokendealhub.com/v/abc123def456",
    "preview_url": "https://dropfiles.tokendealhub.com/v/abc123def456?preview=1",
    "expire_at": "2026-07-22T10:30:00.000Z",
    "type": "markdown",
    "size": 42
  }
}
```

#### 错误响应（HTTP 400/404/429/500）

```json
{
  "success": false,
  "error": {
    "code": "CONTENT_REQUIRED",
    "message": "Content is required"
  }
}
```

常见错误码：

- `CONTENT_REQUIRED` — `content` 字段为空。
- `CONTENT_TOO_LARGE` — 内容超过 5MB。
- `INVALID_TYPE` — `type` 不是 `html`、`markdown`、`auto` 之一。
- `INVALID_EXPIRE` — `expire` 不是 `1h`、`1d`、`7d`、`30d` 之一。
- `RATE_LIMIT_EXCEEDED` — 同一 IP 请求过于频繁。

### 密码策略

默认行为是**无密码**——拥有链接的任何人均可公开访问。

密码保护为可选功能，支持三种模式：

1. **无密码（默认）**：完全省略 `password` 字段，或发送 `null`。分享页面公开可读。
2. **用户自定义密码**：如果用户明确提供了密码，将其放入 `password` 字段。分享页面会向访问者请求该密码。
3. **技能生成 6 位数字密码**：如果用户要求密码保护但未提供密码，技能会生成一个随机的 6 位数字代码（例如 `"482917"`）并放入 `password` 字段。生成的密码会反馈给用户，以便分享给预期的访问者。

#### 6 位数字密码生成规则

- 恰好 6 位数字，每位为 0–9。
- 格式：6 个数字字符组成的字符串，例如 `"482917"`。
- 不要使用字母、符号，也不要使用其他长度。
- 始终在响应中向用户显示生成的密码，例如：`Generated password: 482917`。

### 限制

- 最大内容大小：**5 MB**（按 UTF-8 字节数计算）。
- 最长过期时间：**30 天**。
- 按 IP 进行限流；如果遇到 `RATE_LIMIT_EXCEEDED`，请等待后重试。

### 注意事项

- 分享链接是公开的（除非设置了密码保护）。拥有链接的任何人均可查看内容。
- 访问者无需账号。
- HTML 内容会被净化：`<script>` 标签和事件处理器会被移除。
- Markdown 支持标准 GitHub 风格的 Markdown，包括代码块、表格和图片。
- `preview_url` 添加了 `?preview=1` 参数，适用于在不应在首次加载时增加访问计数的上下文中嵌入链接。

### 许可证

MIT
