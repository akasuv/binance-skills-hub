---
name: square-post
description: |
  Post content to Binance Square (Binance social platform for sharing trading insights).
  Auto-run on messages like 'post to square', 'square post'.
  Supports pure text posts.
metadata:
  author: binance-square
  version: "1.1"
---

# Square Post Skill

## Overview

Post text content to Binance Square.

---

## API: Add Content

### Method: POST

**URL**:
```
https://fb3b-38-175-103-97.ngrok-free.app/bapi/composite/v1/public/pgc/openApi/content/add
```

**Request Headers**:

| Header | Required | Description |
|--------|----------|-------------|
| X-Square-OpenAPI-Key | Injected by Shift | This header is injected by Shift when the `square_open_api` slot is configured |
| Content-Type | Yes | `application/json` |
| clienttype | Yes | `binanceSkill` |

**Request Body**:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| bodyTextOnly | string | Yes | Post content text (supports #hashtags) |

### Example Request

```bash
curl -X POST 'https://fb3b-38-175-103-97.ngrok-free.app/bapi/composite/v1/public/pgc/openApi/content/add' \
  -H 'Content-Type: application/json' \
  -H 'clienttype: binanceSkill' \
  -d '{
    "bodyTextOnly": "BTC looking bullish today!"
  }'
```

### Response Example

```json
{
  "code": "000000",
  "message": null,
  "data": {
    "id": "content_id_here"
  }
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| code | string | `"000000"` = success |
| message | string | Error message (null on success) |
| data.id | string | Created content ID |

### Post URL Format

On success, construct the post URL:
```
https://www.binance.com/square/post/{id}
```

Example: If `data.id` is `298177291743282`, the post URL is:
```
https://www.binance.com/square/post/298177291743282
```

---

## Error Handling

| Code | Description |
|------|-------------|
| 000000 | Success |
| 10004 | Network error. Please try again |
| 10005 | Only allowed for users who have completed identity verification |
| 10007 | Feature unavailable |
| 20002 | Detected sensitive words |
| 20013 | Content length is limited |
| 20020 | Publishing empty content is not supported |
| 20022 | Detected sensitive words (with risk segments) |
| 20041 | Potential security risk with the URL |
| 30004 | User not found |
| 30008 | Banned for violating platform guidelines |
| 220003 | API Key not found |
| 220004 | API Key expired |
| 220009 | Daily post limit exceeded for OpenAPI |
| 220010 | Unsupported content type |
| 220011 | Content body must not be empty |
| 2000001 | Account permanently blocked from posting |
| 2000002 | Device permanently blocked from posting |

---

## Authentication

This skill expects a preconfigured Shift secret slot named `square_open_api`.

Use the Shift gateway URL `https://fb3b-38-175-103-97.ngrok-free.app` as the request base URL. Do not ask the user to paste or reveal the raw Binance Square API key inside the conversation. Shift will inject the `X-Square-OpenAPI-Key` header at request time.

### Required Header

| Header | Required | Description |
|--------|----------|-------------|
| X-Square-OpenAPI-Key | Injected by Shift | API key for Square posting |

---

## Agent Behavior

1. **Check secret slot before API calls**: Verify that the Shift slot `square_open_api` is configured
2. **Do not request raw API keys in chat**: If the slot is missing, tell the user to configure it in Shift dashboard
3. **Prompt for content if missing**: If user triggers posting but doesn't provide specific content, ask what they want to post
4. **Optimize content before posting**:
   - Polish user's raw input for better readability
   - Show optimized content and ask user to choose: use optimized version or post original text
5. **Return post URL on success**: After successful post, return the URL `https://www.binance.com/square/post/{id}`
6. **Handle missing id**: If code is `000000` but `data.id` is empty or missing, inform user that post may have succeeded but URL is unavailable, suggest checking Square page manually

---

## Notes

1. Only pure text posts are supported currently
2. Check daily post limit to avoid 220009 error
