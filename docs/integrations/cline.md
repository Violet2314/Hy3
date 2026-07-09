# Cline 接入 Hy3 指南

本指南介绍如何在 VS Code 中通过 Cline 插件接入本地部署的 Hy3 大模型，完成自主编程任务。

Hy3 是腾讯混元团队开发的 295B 参数 MoE 大模型，通过 vLLM/SGLang 部署后提供 OpenAI 兼容 API。本指南假设 Hy3 已部署在本地，并通过 `http://127.0.0.1:8000/v1` 提供服务，模型名称为 `hy3`。

## 1. 安装与版本要求

- **Cline 简介**：Cline 是 VS Code 中的自主编程 AI 代理插件（原 Claude Dev），能够自主分析需求、读写文件、执行终端命令并完成端到端的开发任务。
- **安装方式**：在 VS Code 扩展市场搜索 "Cline" 安装。
- **VS Code 版本**：需要 VS Code 1.84 及以上版本。
- **操作系统**：支持 macOS / Windows / Linux。
- **官网**：https://cline.bot

## 2. 配置 Hy3 连接

1. 打开 VS Code，按 `Ctrl+Shift+P`（Windows/Linux）或 `Cmd+Shift+P`（macOS）打开命令面板，搜索并执行 `Cline: Open Settings`。
2. 在 API Provider 选项中选择 **OpenAI Compatible**。
3. **Base URL** 填入：
   ```
   http://127.0.0.1:8000/v1
   ```
4. **API Key** 填入：
   ```
   EMPTY
   ```
   > 本地部署无需鉴权，但该字段不可为空，必须填入字符串 `EMPTY`。
5. **Model ID** 填入：
   ```
   hy3
   ```

![Cline 配置](./assets/cline-config.png)

## 3. 第一次对话

1. 点击 VS Code 左侧活动栏中的 Cline 图标，打开 Cline 侧边栏。
2. 在输入框中输入：
   ```
   你好，请介绍一下你自己
   ```
3. 按下回车发送消息。

**预期响应**：Hy3 会以中文进行自我介绍，说明自己是腾讯混元团队开发的大语言模型，并简要描述自身能力（如代码生成、问答、文档撰写等）。响应通常在数秒内开始流式返回。

![Cline 第一次对话](./assets/cline-first-chat.png)

## 4. 端到端真实任务 Demo

下面演示一个完整的文件创建任务，展示 Cline 结合 Hy3 的自主编程能力。

1. 在 Cline 侧边栏输入框中输入：
   ```
   请在当前目录创建一个 hello.py 文件，内容为打印 Hello World 的 Python 脚本
   ```
2. 按下回车发送。

**预期流程**：

1. **分析请求**：Cline 解析用户意图，确定需要在当前工作目录创建 `hello.py` 文件。
2. **生成文件内容**：Hy3 生成符合要求的 Python 脚本内容，例如：
   ```python
   print("Hello World")
   ```
3. **请求用户批准**：Cline 弹出批准面板，展示即将执行的文件创建操作及完整内容，等待用户点击 "Approve"。
4. **创建文件**：用户批准后，Cline 在当前工作目录创建 `hello.py` 文件，并在对话中反馈执行结果。

![Cline 创建文件](./assets/cline-demo.png)

## 5. 常见注意事项

- **连接失败**：若 Cline 报告无法连接到模型，请先确认 Hy3 服务已启动，且 `http://127.0.0.1:8000/v1` 可访问（可在浏览器或终端访问 `/v1/models` 端点验证）。
- **API Key 不可为空**：API Key 字段必须填入字符串 `EMPTY`，不能留空。留空会导致部分 OpenAI 兼容客户端校验失败。
- **自动批准模式**：Cline 提供 "Auto Approve" 功能，可自动批准文件读写与命令执行。建议首次使用时保持 **手动批准模式**，仔细审查 AI 的每一步操作，避免误操作。
- **工具调用配置**：Cline 支持工具调用（function calling），Hy3 在 vLLM/SGLang 启动时需配置 `--tool-call-parser hy_v3` 与 `--reasoning-parser hy_v3`，否则工具调用能力无法正常工作。
- **推理模式限制**：Hy3 支持通过 `extra_body={"chat_template_kwargs": {"reasoning_effort": "high"/"low"/"no_think"}}` 控制推理模式。但 Cline 的 OpenAI Compatible provider 目前不直接暴露 `extra_body` 参数，因此暂无法在 Cline 中切换 Hy3 的推理模式。如需使用推理模式，建议在 vLLM/SGLang 服务端通过启动参数或默认 chat template 进行配置。
