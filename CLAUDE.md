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

### 上传与发布 (最佳实践)
**强烈建议使用 Git 远程仓库同步至 ClawHub，而非官网直接上传。**

- **版本控制**: 利用 Git 的 Commit 记录追踪提示词 (Prompt) 的迭代过程（如 v1.0 到 v5.0 的优化），方便在效果回退时一键回滚。
- **资产沉淀**: GitHub 仓库是开发者品牌的展示窗口，整洁的 `README.md` 和 `Skills.md` 能显著提升作品权重。
- **自动化同步**: 配置 Webhook 后，本地 `git push` 即可自动触发 ClawHub 上的技能更新，极大提升调试效率。

### 开发任务
- **验证清单 (Manifest)**: 确保 `endpoint` URL 与工作流 ID 匹配（例如：`.../v1/workflows/[ID]/run`）。
- **更新版本**: 在提交前同时递增 `manifest.json` 和 `_meta.json` 中的 `version` 字段。
- **测试 API 调用**: 使用 `curl` 按照 `SKILL.md` 中的文档说明测试提交和轮询。

### 执行逻辑
1. **提交**: 发送 POST 请求至 `https://api.ngmob.com/api/v1/workflows/[ID]/run`。
2. **认证**: Header 必须包含 `Authorization: Bearer mk_xxxxxxxx`（API Key 前缀始终为 `mk_`）。
3. **轮询**: 发送 GET 请求至 `https://api.ngmob.com/api/v1/workflows/executions/[TASK_ID]`，直到状态码为 `2000`（成功）或 `5xxx`（失败）。

## 📝 准则
- **技能描述**: `SKILL.md` 或 `manifest.json` 中的 `description` 必须始终以 "Use when..." 开头，重点描述触发条件。
- **工作流参数**: 将 `manifest.json` 中的 `inputs` 映射到 Pixify 控制台中使用的正确节点名称。
- **敏感信息**: **绝对禁止**提交 API Key (`mk_...`)。在 `manifest.json` 中使用 `{{API_KEY}}` 占位符。
