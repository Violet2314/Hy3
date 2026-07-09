# Continue 接入 Hy3 指南

本指南介绍如何在 VS Code 中通过 Continue 插件接入本地部署的 Hy3 大模型，获得 AI 代码助手能力。

Hy3 是腾讯混元团队开发的 295B 参数 MoE 大模型，通过 vLLM/SGLang 部署后提供 OpenAI 兼容 API。本指南假设 Hy3 已部署在本地，并通过 `http://127.0.0.1:8000/v1` 提供服务，模型名称为 `hy3`。

## 1. 安装与版本要求

- **Continue 简介**：Continue 是开源的可扩展 AI 代码助手 VS Code 插件，支持自定义模型、上下文提供者与命令，可用于代码问答、代码解释、自动补全等场景。
- **安装方式**：在 VS Code 扩展市场搜索 "Continue" 安装。
- **VS Code 版本**：需要 VS Code 1.74 及以上版本。
- **操作系统**：支持 macOS / Windows / Linux。
- **官网**：https://continue.dev
- **GitHub**：https://github.com/continuedev/continue

## 2. 配置 Hy3 连接

1. 打开 VS Code，在 Continue 侧边栏右下角点击齿轮图标，选择 **Open config.json**。
   - 或按 `Ctrl+Shift+P`（Windows/Linux）/ `Cmd+Shift+P`（macOS）打开命令面板，搜索并执行 `Continue: Open config.json`。
2. 在 `config.json` 的 `models` 数组中添加 Hy3 配置：

```json
{
  "models": [
    {
      "title": "Hy3",
      "provider": "openai",
      "model": "hy3",
      "apiKey": "EMPTY",
      "apiBase": "http://127.0.0.1:8000/v1"
    }
  ]
}
```

3. 保存文件后，Continue 会自动加载新配置，无需重启 VS Code。

> **关键说明**：`provider` 必须设置为 `openai`（而非 `openai-compatible`），以便 Continue 使用 `/chat/completions` 端点与 Hy3 通信。

![Continue 配置](./assets/continue-config.png)

## 3. 第一次对话

1. 点击 VS Code 左侧活动栏中的 Continue 图标，打开 Continue 侧边栏。
2. 在聊天面板顶部的模型下拉菜单中选择 **Hy3**。
3. 在输入框中输入：
   ```
   Hello! Can you briefly introduce yourself?
   ```
4. 按下回车发送消息。

**预期响应**：Hy3 会以英文进行简短的自我介绍，说明自身是腾讯混元团队开发的大语言模型，并概述自身在代码生成、问答、推理等方面的能力。响应会以流式方式逐字返回。

![Continue 第一次对话](./assets/continue-first-chat.png)

## 4. 端到端真实任务 Demo

下面演示使用 Continue 结合 Hy3 解释选中的代码。

1. 在编辑器中打开任意源代码文件，选中一段你希望理解的代码。
2. 按 `Ctrl+L`（Windows/Linux）或 `Cmd+L`（macOS），将选中的代码发送到 Continue Chat 面板。
3. 在输入框中输入：
   ```
   请解释这段代码的功能
   ```
4. 按下回车发送。

**预期输出**：Hy3 会分析选中的代码，逐段解释其功能、关键逻辑与可能的副作用，并以中文返回结构化的说明。对于复杂代码，Hy3 还会指出潜在的边界情况或改进建议。

![Continue 解释代码](./assets/continue-demo.png)

## 5. 常见注意事项

- **连接失败**：若 Continue 报告无法连接或请求超时，请确认 Hy3 服务运行在 `http://127.0.0.1:8000/v1`，并可通过 `GET /v1/models` 端点返回 `hy3` 模型。
- **API Key 不可为空字符串**：`config.json` 中 `apiKey` 字段不能为空字符串 `""`，必须填入 `EMPTY`。空字符串会导致请求被部分 OpenAI 客户端拦截。
- **provider 设置**：`provider` 必须设为 `openai`（而非 `openai-compatible`）。前者使用 `/chat/completions` 端点，与 Hy3 的 OpenAI 兼容 API 完全匹配；后者可能使用不同的端点路径导致请求失败。
- **Tab 自动补全**：Continue 支持 Tab 自动补全功能。如需使用 Hy3 作为补全模型，可在 `config.json` 的 `tabAutocompleteModel` 字段中使用与上文相同的配置（`provider`、`model`、`apiKey`、`apiBase`）。
- **自定义请求参数**：如需为 Hy3 传入自定义请求参数（如 `reasoning_effort` 控制推理模式，或 `temperature`、`top_p` 等），可在模型配置中添加 `requestOptions` 字段。例如：

  ```json
  {
    "title": "Hy3",
    "provider": "openai",
    "model": "hy3",
    "apiKey": "EMPTY",
    "apiBase": "http://127.0.0.1:8000/v1",
    "requestOptions": {
      "temperature": 0.9,
      "top_p": 1.0,
      "extra_body": {
        "chat_template_kwargs": {
          "reasoning_effort": "high"
        }
      }
    }
  }
  ```

  > 通过 `extra_body.chat_template_kwargs.reasoning_effort` 可设置 `high`、`low` 或 `no_think` 三种推理模式。
