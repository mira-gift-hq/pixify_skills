# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 📁 仓库结构
本仓库包含多个为 Clawhub 市场准备的 Pixify 技能定义。每个目录代表一个独立的技能。

- `MasterPiece Clone/`: 当前主要编辑的活跃工作流技能。
- `comic-to-realistic-1.0.3/`: **参考技能结构目录**，用于参考文件配置和 Markdown 格式。
- `test/`: **测试用例脚本专用目录**，用于存放自动化测试或手动测试脚本。
- `workflow-[ID]/SKILL.md`: 技能目录中遗留的技能定义文件。

## 🛠 核心组件
每个技能包含以下文件：
- `manifest.json`: 定义 API 端点、身份验证、输入/输出模式以及轮询逻辑。
- `Skills.md` (或 `SKILL.md`): 包含系统标头的 Markdown 文档，供 Claude Code 理解并执行工作流。
- `_meta.json`: 发布元数据（版本、所有者 ID、slug）。
- `examples/`: 包含用于 API 验证的 JSON 响应示例。

## 🚀 常用流程与命令
由于这是一个基于配置的 Pixify 工作流仓库，因此没有构建脚本或活跃服务器。

### ⚠️ 发布前测试 (必须)

**在发布到 ClawHub 之前，必须使用 Postman 或 curl 测试 API 端点，确保工作流正常运行。**

#### 测试步骤

1. **准备测试数据**
   - 从 `examples/` 目录获取示例 JSON 响应
   - 准备真实的输入参数（如图片 URL）

2. **使用 Postman 测试**

   **请求配置：**
   - **Method**: POST
   - **URL**: `https://api.ngmob.com/api/v1/workflows/[WORKFLOW_ID]/run`
   - **Headers**:
     ```
     Content-Type: application/json
     Authorization: Bearer mk_your_api_key_here
     ```
   - **Body** (raw JSON):
     ```json
     {
       "inputs": {
         "Image Input": "https://example.com/reference-image.png",
         "Image Input 1": "https://example.com/target-image.png"
       }
     }
     ```

3. **获取任务 ID**
   - 发送请求后，记录响应中的 `task_id`
   - 示例响应:
     ```json
     {
       "task_id": "abc123xyz789",
       "status": "pending"
     }
     ```

4. **轮询任务状态**

   **请求配置：**
   - **Method**: GET
   - **URL**: `https://api.ngmob.com/api/v1/workflows/executions/[TASK_ID]`
   - **Headers**:
     ```
     Authorization: Bearer mk_your_api_key_here
     ```

   **预期响应（成功）：**
   ```json
   {
     "status": 2000,
     "message": "success",
     "outputs": [
       {
         "url": "https://example.com/generated-image.png"
       }
     ]
   }
   ```

   **预期响应（失败）：**
   ```json
   {
     "status": 5001,
     "message": "error message",
     "error": "detailed error"
   }
   ```

5. **验证结果**
   - ✅ 状态码为 `2000` 表示成功
   - ✅ 输出包含有效的图片 URL
   - ❌ 状态码为 `5xxx` 表示失败，检查错误信息

#### 使用 curl 测试（替代方案）

```bash
# 1. 提交任务
TASK_ID=$(curl -X POST https://api.ngmob.com/api/v1/workflows/[WORKFLOW_ID]/run \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer mk_your_api_key_here" \
  -d '{
    "inputs": {
      "Image Input": "https://example.com/reference-image.png",
      "Image Input 1": "https://example.com/target-image.png"
    }
  }' | jq -r '.task_id')

echo "Task ID: $TASK_ID"

# 2. 轮询状态（每 3-5 秒检查一次）
for i in {1..20}; do
  curl -X GET https://api.ngmob.com/api/v1/workflows/executions/$TASK_ID \
    -H "Authorization: Bearer mk_your_api_key_here" | jq '.'
  sleep 3
done
```

#### 常见问题排查

| 问题 | 原因 | 解决方案 |
|------|------|--------|
| 401 Unauthorized | API Key 无效或过期 | 检查 `Authorization` header 中的 API Key |
| 400 Bad Request | 请求体格式错误 | 检查 JSON 格式，确保参数名称与 `manifest.json` 匹配 |
| 404 Not Found | Workflow ID 错误 | 验证 `manifest.json` 中的 `endpoint` URL |
| 5001 Workflow Error | 工作流执行失败 | 检查输入参数（如图片 URL 是否可访问） |
| 超时 | 任务执行时间过长 | 增加轮询间隔或等待时间 |

### 上传与发布 (最佳实践)

使用 **ClawHub CLI** 直接发布技能到 ClawHub 市场，无需依赖 Git。

**发布流程：**

1. **准备技能文件**
   - 确保 `manifest.json`、`Skills.md`、`_meta.json` 等文件配置正确
   - 验证 `endpoint` URL 与工作流 ID 匹配

2. **更新版本号**
   - 同时递增 `manifest.json` 和 `_meta.json` 中的 `version` 字段
   - 版本号必须遵循语义化版本 (semver) 格式，如 `1.0.2`

3. **执行测试**
   - ⚠️ **必须**使用 Postman 或 curl 测试 API 端点
   - 确保工作流能正常运行并返回预期结果
   - 参考上方"发布前测试"部分

4. **使用 ClawHub CLI 发布**
   ```bash
   clawhub publish <skill-folder> --version <version>
   ```
   
   **示例：**
   ```bash
   clawhub publish "MasterPiece_Clone" --version 1.0.2
   ```

5. **可选参数**
   - `--slug <slug>`: 自定义技能 slug（默认从 manifest.json 读取）
   - `--name <name>`: 自定义显示名称（默认从 manifest.json 读取）
   - `--changelog <text>`: 添加更新日志
   - `--tags <tags>`: 逗号分隔的标签（默认为 "latest"）
   - `--fork-of <slug[@version]>`: 标记为某个技能的分支

6. **验证发布**
   - 发布成功后会显示：`✔ OK. Published <skill-name>@<version> (<publish-id>)`
   - 在 ClawHub 市场确认技能已成功发布

**优势：**
- 直接发布，无需 Git 中间步骤
- 快速迭代和调试
- 版本管理由 ClawHub 平台维护
- 支持灵活的发布参数配置

### 开发任务
- **验证清单 (Manifest)**: 确保 `endpoint` URL 与工作流 ID 匹配（例如：`.../v1/workflows/[ID]/run`）。
- **更新版本**: 在发布前同时递增 `manifest.json` 和 `_meta.json` 中的 `version` 字段。
- **测试 API 调用**: 使用 `curl` 按照 `SKILL.md` 中的文档说明测试提交和轮询。

### 执行逻辑
1. **提交**: 发送 POST 请求至 `https://api.ngmob.com/api/v1/workflows/[ID]/run`。
2. **认证**: Header 必须包含 `Authorization: Bearer mk_xxxxxxxx`（API Key 前缀始终为 `mk_`）。
3. **轮询**: 发送 GET 请求至 `https://api.ngmob.com/api/v1/workflows/executions/[TASK_ID]`，直到状态码为 `2000`（成功）或 `5xxx`（失败）。

## 📝 准则
- **技能描述**: `SKILL.md` 或 `manifest.json` 中的 `description` 必须清晰描述技能功能和使用场景。
- **工作流参数**: 将 `manifest.json` 中的 `inputs` 映射到 Pixify 控制台中使用的正确节点名称。
- **敏感信息**: **绝对禁止**提交 API Key (`mk_...`)。在 `manifest.json` 中使用 `{{API_KEY}}` 占位符。
