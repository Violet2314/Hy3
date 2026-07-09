# Cursor 接入 Hy3 指南

本指南介绍如何在 Cursor AI IDE 中接入本地部署的 Hy3 大模型，完成从模型配置到端到端代码生成任务的完整流程。

## 1. 安装与版本要求

- **下载地址**：[https://cursor.sh](https://cursor.sh)
- **支持平台**：macOS / Windows / Linux
- **版本建议**：建议使用最新版本，以确保对自定义 OpenAI 兼容 API 的支持最为完善

下载并安装完成后，启动 Cursor 即可开始后续配置。

## 2. 配置 Hy3 连接

Hy3 通过 vLLM/SGLang 部署后暴露的是 OpenAI 兼容协议，因此可以直接作为 Cursor 的自定义 OpenAI 模型接入。配置步骤如下：

1. 打开 Cursor，进入 **Settings**（快捷键 `Ctrl + ,` / macOS 下 `Cmd + ,`）。
2. 切换到 **Models** 配置页。
3. 在 **OpenAI API Key** 输入框中填入 `EMPTY`。
4. 开启 **Override OpenAI Base URL** 开关，并在文本框中填入：

   ```
   http://127.0.0.1:8000/v1
   ```

5. 在自定义模型列表中添加 `hy3`，并确保其处于启用状态。
6. 点击 **Verify** 按钮验证连接是否成功。

![Cursor 模型配置](./assets/cursor-config.png)

> 说明：Cursor 与 Hy3 之间使用标准 OpenAI 兼容协议通信，所有请求遵循 `/v1/chat/completions` 接口规范。

## 3. 第一次对话

完成配置后，即可开始与 Hy3 对话：

1. 打开 Cursor Chat（快捷键 `Ctrl + L` / macOS 下 `Cmd + L`）。
2. 在模型选择器中选择 `hy3`。
3. 在输入框中发送消息：

   ```
   你好，请简单介绍一下你自己
   ```

4. 预期响应：Hy3 会用中文进行自我介绍，说明自己是腾讯混元团队开发的大语言模型，并简要描述自身能力（如多轮对话、代码生成、知识问答等）。

![Cursor 第一次对话](./assets/cursor-first-chat.png)

## 4. 端到端真实任务 Demo

下面演示一个真实的代码生成任务，验证 Hy3 在 Cursor 中的实际编程能力：

1. 在 Cursor 编辑器中打开一个项目目录（例如新建一个空的 Python 项目）。
2. 使用 Cursor Chat（`Ctrl + L` / `Cmd + L`）或行内编辑（`Ctrl + K` / `Cmd + K`）唤起 Hy3。
3. 输入如下 prompt：

   ```
   请用 Python 实现一个快速排序算法，并添加注释
   ```

4. 预期输出：Hy3 会生成一段带有完整注释的 Python 快速排序实现，通常包含：
   - 选取基准元素（pivot）的逻辑
   - 递归分割左右子数组
   - 边界处理与递归终止条件
   - 简单的使用示例

5. 可直接将生成的代码插入编辑器，或通过 Cursor 的 Accept 操作应用至当前文件。

![Cursor 生成代码](./assets/cursor-demo.png)

## 5. 常见注意事项

- **连接被拒（Connection Refused）**
  确认 Hy3 服务已启动，且端口 `8000` 可在本机访问。可通过 `curl http://127.0.0.1:8000/v1/models` 验证服务是否就绪。

- **模型未找到（Model Not Found）**
  确认启动 vLLM/SGLang 时已通过 `--served-model-name hy3` 指定模型名，且与 Cursor 中自定义模型名完全一致。

- **API Key 必须填 `EMPTY`**
  API Key 不能留空。Cursor 会对 API Key 进行非空校验，留空将导致配置无法保存或请求被拦截。本地部署无需真实鉴权，固定填写 `EMPTY` 即可。

- **Verify 按钮返回警告**
  Cursor 的 Verify 按钮可能对非标准 OpenAI API 返回警告信息（如字段缺失等），这通常不影响实际使用，可忽略后正常调用。

- **推理模式使用限制**
  Hy3 支持通过 `extra_body={"chat_template_kwargs": {"reasoning_effort": "high"/"low"/"no_think"}}` 控制推理模式。但 Cursor Chat 目前不直接支持透传 `extra_body` 参数，如需引导模型进行深度推理，可通过在系统提示词（System Prompt）中明确要求「请逐步思考后再回答」等方式进行引导。
