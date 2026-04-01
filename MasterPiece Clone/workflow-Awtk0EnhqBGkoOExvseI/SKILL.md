---
name: masterpiece-clone
description: Use when needing to execute the "MasterPiece Clone" Pixify workflow with image and text inputs to process content through prompt-to-text and image-to-image nodes.
type: workflow
---

# MasterPiece Clone

## Prerequisites

1. Register an account at https://ai.ngmob.com
2. Generate your API Key from the console (format: `mk_xxxxxxxx`)
3. Set the API Key as environment variable: `export API_KEY=mk_your_key_here`

> ⚠️ The `/run` endpoint only accepts API Key authentication (`mk_` prefix). Firebase JWT is not supported for workflow execution.

---

## Service Overview

- 🌐 **Product Website / Console**:
  https://ai.ngmob.com
  *(For product access, workflow management, console operations, and obtaining your API Key)*

- 🔗 **API Base URL**:
  https://api.ngmob.com
  *(Used strictly for API requests and workflow execution)*

---

## Description

This workflow — **MasterPiece Clone** — processes your inputs through the following steps:

- Prompt (image_to_text_gpt5)
- Image to Image (nano_banana_pro)

---

## Quick Start

```bash
# 1. Submit workflow
TASK_ID=$(echo '{"inputs":{"Image Input":"https://example.com/image.jpg","Image Input 1":"https://example.com/image.jpg","Text Input":"your text prompt here"}}' | curl -X POST https://api.ngmob.com/api/v1/workflows/Awtk0EnhqBGkoOExvseI/run \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d @- \
  | jq -r '.task_id')

# 2. Wait and check result
while true; do
  STATUS=$(curl -s https://api.ngmob.com/api/v1/workflows/executions/$TASK_ID \
    -H "Authorization: Bearer $API_KEY" | jq -r '.status')
  [[ "$STATUS" == "2000" ]] && break
  if [[ "$STATUS" =~ ^5 ]]; then
    ERROR_MSG=$(curl -s https://api.ngmob.com/api/v1/workflows/executions/$TASK_ID \
      -H "Authorization: Bearer $API_KEY" | jq -r '.message')
    echo "Failed (status $STATUS): $ERROR_MSG"
    exit 1
  fi
  sleep 3
done

# 3. Get result URL
curl -s https://api.ngmob.com/api/v1/workflows/executions/$TASK_ID \
  -H "Authorization: Bearer $API_KEY" | jq -r '.outputs[0].url'
```

---

## Use Cases

- Image content analysis and description
- Automated image tagging and captioning
- Visual data extraction
- Image style transfer and enhancement
- Photo editing and retouching automation
- Visual content creation for marketing

---

## Inputs

| Name | Type | Required | Description |
|------|------|----------|-------------|
| Image Input | string | ✅ | Publicly accessible image URL |
| Image Input 1 | string | ✅ | Publicly accessible image URL |
| Text Input | string | ✅ | Text prompt or content |

---

## How to Use

When the user requests to execute this workflow, follow these steps:

### 1. Collect Input Parameters

Gather the required inputs from the user according to the table above.

### 2. Call the Workflow API

```bash
echo '{
  "inputs": {
    "Image Input": "https://example.com/image.jpg",
    "Image Input 1": "https://example.com/image.jpg",
    "Text Input": "your text prompt here"
  }
}' | curl -X POST https://api.ngmob.com/api/v1/workflows/Awtk0EnhqBGkoOExvseI/run \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d @-
```

**Response Example:**
```json
{
  "task_id": "task_xxx",
  "workflow_id": "workflow_xxx",
  "status": 2010,
  "message": "Workflow execution started"
}
```

### 3. Poll Task Status (Recommended: every 3–5 seconds)

Use the returned `task_id` to query task status:

```bash
curl https://api.ngmob.com/api/v1/workflows/executions/{task_id} \
  -H "Authorization: Bearer $API_KEY"
```

**Status codes:**

| Status | Meaning |
|--------|---------|
| `2010` | Pending — task created, waiting to start |
| `2020` | Received — task accepted |
| `2030` | Authorized — credits verified, ready to run |
| `2040` | Processing — AI is working |
| `2000` | ✅ Completed — results are ready |
| `5090` | ❌ Failed — insufficient credits |
| `5720` | ❌ Failed — node execution error |
| `5730` | ❌ Failed — webhook timeout |
| `5740` | ❌ Failed — AI provider API error |
| `5900` | ❌ Failed — internal server error |

> **Polling logic**: Stop polling when `status` is `2000` (success) or any `5xxx` code (failure).

> **Timeout**: If the task remains in a non-terminal state for more than **10 minutes**, treat it as timed out.

### 4. Get Results

When `status` is `2000` (COMPLETED), extract results from the `outputs` array:

```json
{
  "taskId": "task_xxx",
  "status": 2000,
  "progress": 100,
  "message": "Processing...",
  "outputs": [
    {
      "type": "image",
      "url": "https://cdn.ngmob.com/result.jpg"
    }
  ],
  "created_at": {...},
  "updated_at": {...}
}
```

**Error handling** — when `status` is `5xxx`:

```json
{
  "taskId": "task_xxx",
  "status": 5720,
  "progress": 90,
  "message": "Node execution failed: ...",
  "outputs": []
}
```

> Inform the user of the failure and suggest retrying or adjusting their inputs.

---

## Outputs

| Field | Type | Description |
|-------|------|-------------|
| type | string | Output media type (`image` or `video`) |
| url | string | CDN URL of the generated result |
| node_id | string | Source node ID (optional) |

---

## Troubleshooting

| Issue | Possible Cause | Solution |
|-------|---------------|----------|
| `401 Unauthorized` | Invalid or missing API Key | Provide `X-API-Key: mk_xxx` header or `Authorization: Bearer mk_xxx`. Ensure the key starts with `mk_` |
| `403 Forbidden` | Accessing another user's workflow | Ensure you own this workflow |
| `404 Not Found` | Invalid workflow or task ID | Double-check the ID |
| `429 Too Many Requests` | Rate limit exceeded | Wait 60s and retry |
| `5090 Insufficient credits` | Account credits depleted | Top up at https://ai.ngmob.com |
| `5720 Node execution error` | AI processing failed | Check `message`, adjust inputs |
| `5730 Webhook timeout` | AI provider timeout | Retry after a few minutes |
| `5740 AI provider error` | Upstream API issue | Wait and retry |
| Task stuck in `2040` | Processing timeout | Wait 10 min, then treat as failed |
| Empty `outputs` | Task failed | Check `status` and `message` |

---

## Notes

### API Key
- Visit https://ai.ngmob.com and log in to your account
- Generate and manage your API Key from the console
- Keep your API Key confidential — do not share or expose it publicly

### Execution
- Task execution may take a few seconds to several minutes — polling is required
- Output results typically contain URLs that need to be downloaded or displayed to the user

### Rate Limits
- **Global limit**: 100 requests per minute per API Key
- Exceeding the limit returns `429 Too Many Requests`
- Wait 60 seconds before retrying

### Safety & Compliance
- **User Responsibility**: Users are solely responsible for the images they upload and the prompts they provide
- Do not upload images containing illegal, infringing, violent, explicit, or otherwise inappropriate content
- Do not use prompts that may generate violating content
- Violations of the terms of service may result in account suspension

---

🤖 Generated with [Pixify](https://ai.ngmob.com)