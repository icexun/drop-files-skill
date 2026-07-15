---
name: "drop-files"
description: "Upload generated HTML/Markdown files to DropFiles and get a shareable public link. Invoke when the user wants to share, publish, or generate a public link for any HTML or Markdown content they produced."
---

# DropFiles

Upload generated HTML or Markdown content to the DropFiles service and return a public shareable link. The service is free, globally accessible, and hosted on Cloudflare's edge network.

## When to Invoke

Invoke this skill when the user wants to:
- Share generated HTML or Markdown content via a public link
- Publish a document, report, or page and get a URL to distribute
- Upload any text-based artifact (HTML/Markdown) for online viewing
- Generate a shareable preview link for content the agent just produced

## API Endpoint

- **Base URL**: `https://dropfiles.tokendealhub.com`
- **Upload**: `POST /api/v1/upload`
- **Metadata**: `GET /api/v1/info/{id}`
- **Preview**: `https://dropfiles.tokendealhub.com/v/{id}`

## Upload Request

**IMPORTANT: Always use `curl` to send the upload request.** The API is behind Cloudflare WAF which may block requests from other HTTP clients (e.g. Python urllib, Node fetch) with HTTP 403 (error code 1010). curl is the only reliably working client.

Send a `POST` request to `https://dropfiles.tokendealhub.com/api/v1/upload` with `Content-Type: application/json`.

### Handling Large Content

When the content to upload is large (e.g. over 2KB) or contains special characters (quotes, backslashes, HTML tags, etc.), **do NOT inline the content in a shell command**. Instead, write the JSON payload to a temporary file first, then use `curl -d @file`:

```bash
# Step 1: Write JSON payload to a temp file (use Python or any method)
python3 -c "
import json
payload = json.dumps({
    'content': open('your_file.md').read(),
    'type': 'markdown',
    'expire': '7d',
    'title': 'My Document'
})
with open('/tmp/dropfiles_payload.json', 'w') as f:
    f.write(payload)
"

# Step 2: Upload using curl with file reference
curl -X POST https://dropfiles.tokendealhub.com/api/v1/upload \
  -H "Content-Type: application/json" \
  -d @/tmp/dropfiles_payload.json

# Step 3: Clean up
rm /tmp/dropfiles_payload.json
```

### Request Body

| Field      | Type   | Required | Description |
|------------|--------|----------|-------------|
| `content`  | string | Yes      | The HTML or Markdown text to upload. Max 5MB. |
| `type`     | string | No       | `html`, `markdown`, or `auto` (default: `auto`, auto-detects from content). |
| `expire`   | string | No       | `1h`, `1d`, `7d`, or `30d` (default: `7d`). |
| `title`    | string | No       | Optional title shown on the share page. |
| `password` | string | No       | Optional password to protect the share page. See Password Policy below. |
| `lang`     | string | No       | `en`, `zh`, or `auto` (default: `auto`). |

### Example (no password — default)

```bash
curl -X POST https://dropfiles.tokendealhub.com/api/v1/upload \
  -H "Content-Type: application/json" \
  -d '{
    "content": "# Hello World\n\nThis is a shared Markdown document.",
    "type": "markdown",
    "expire": "7d",
    "title": "My Document"
  }'
```

### Example (with user-supplied password)

```bash
curl -X POST https://dropfiles.tokendealhub.com/api/v1/upload \
  -H "Content-Type: application/json" \
  -d '{
    "content": "# Secret Report\n\nConfidential content.",
    "type": "markdown",
    "expire": "1d",
    "password": "user_provided_password"
  }'
```

### Example (with skill-generated 6-digit password)

```bash
curl -X POST https://dropfiles.tokendealhub.com/api/v1/upload \
  -H "Content-Type: application/json" \
  -d '{
    "content": "# Protected Document\n\nContent here.",
    "type": "markdown",
    "expire": "7d",
    "password": "482917"
  }'
```

## Password Policy

The default behavior is **no password** — the share link is publicly accessible by anyone with the URL.

Password protection is opt-in and supports three modes:

1. **No password (default)**: Omit the `password` field entirely, or send `null`. The share page is publicly readable.
2. **User-supplied password**: If the user explicitly provides a password, pass it in the `password` field. The share page will prompt viewers for this password.
3. **Skill-generated 6-digit password**: If the user asks for password protection but does not provide a password, the skill MUST generate a random 6-digit numeric code (e.g. `"482917"`) and include it in the `password` field. The generated password must be clearly communicated back to the user so they can share it with intended viewers.

### 6-Digit Password Generation Rule

- Exactly 6 digits, each 0–9 (e.g. `0-9` repeated 6 times, random).
- Format: a string of 6 numeric characters, e.g. `"482917"`.
- Do NOT use letters, symbols, or shorter/longer lengths.
- Always display the generated password to the user in the response, e.g.: `Generated password: 482917`. The user needs this to share with viewers.

## Response

### Success (HTTP 201)

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

### Error (HTTP 400/404/429/500)

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

## Workflow

1. Confirm the content to upload (HTML or Markdown). If the user just generated a document/page, use that content directly.
2. Determine options:
   - `type`: use `auto` unless the user specifies. Auto-detection treats content containing `<html` or `<!doctype` as HTML, otherwise Markdown.
   - `expire`: default `7d` unless the user requests a different lifetime.
   - `title`: optional, set if the user provides one.
3. Determine password:
   - Default: no password.
   - If the user explicitly provides a password, use it.
   - If the user asks for protection without providing a password, generate a random 6-digit numeric string and use it.
4. Send the upload request **using `curl`** (see "Upload Request" section above).
   - For small content (< 2KB, no special characters): inline the JSON directly in the `curl -d` argument.
   - For large content or content with special characters: write the JSON payload to a temp file, then use `curl -d @/tmp/dropfiles_payload.json`. Clean up the temp file afterwards.
   - **Do NOT use Python urllib, Node fetch, or other HTTP clients** — they will be blocked by Cloudflare WAF (HTTP 403, error code 1010).
5. On success, return the `url` (and `preview_url` if relevant) to the user. If a password was generated, prominently display it: `Generated password: XXXXXX`.
6. On error, surface the `error.message` to the user and suggest fixes.

## Content Type Auto-Detection

When `type` is `auto` (the default), the server detects the type:
- Content containing `<html` or `<!doctype` (case-insensitive) → `html`
- Everything else → `markdown`

HTML content is sanitized server-side (scripts are stripped) for safety. Markdown content is rendered to HTML with a GitHub-like stylesheet on the share page.

## Limits

- Max content size: **5 MB** (measured as UTF-8 bytes).
- Max expiry: **30 days**.
- Rate limiting applies per IP; if you hit `RATE_LIMIT_EXCEEDED`, wait and retry.

## Notes

- The share link is public (unless password-protected). Anyone with the URL can view the content.
- Viewers do not need an account.
- HTML content is sanitized: `<script>` tags and event handlers are removed.
- Markdown supports standard GitHub-flavored Markdown including code blocks, tables, and images.
- The `preview_url` adds `?preview=1` and is useful when embedding the link in a context that should not inflate the access counter on initial load.
