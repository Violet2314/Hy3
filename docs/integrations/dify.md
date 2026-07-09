# Dify 接入 Hy3 指南

本指南介绍如何在 [Dify](https://dify.ai) 低代码平台中接入腾讯混元 **Hy3** 大模型，通过可视化编排快速构建 AI 工作流与应用。

Hy3 是腾讯混元团队研发的 295B 参数 MoE 大模型，通过 vLLM / SGLang 部署后提供 OpenAI 兼容 API，可无缝对接 Dify 的 "OpenAI-API-compatible" 模型供应商。本指南假设 Hy3 服务已按 [README 部署章节](../../README_CN.md#推理和部署)完成部署并满足以下条件：

| 项 | 值 |
|:---|:---|
| Base URL | `http://127.0.0.1:8000/v1` |
| 模型名称（served-model-name） | `hy3` |
| API Key | `EMPTY`（本地部署无需鉴权，但 Dify 要求非空） |
| 推理模式 | 通过 `extra_body={"chat_template_kwargs": {"reasoning_effort": "high"/"low"/"no_think"}}` 控制 |
| 推荐参数 | `temperature=0.9`，`top_p=1.0` |
| 工具调用 | 支持（tool-call-parser: `hy_v3`，reasoning-parser: `hy_v3`） |
| 上下文长度 | 256K |

---

## 目录

- [1. 安装与版本要求](#1-安装与版本要求)
- [2. 配置 Hy3 连接](#2-配置-hy3-连接)
- [3. 第一次对话](#3-第一次对话)
- [4. 端到端真实任务 Demo](#4-端到端真实任务-demo)
- [5. 常见注意事项](#5-常见注意事项)

---

## 1. 安装与版本要求

**Dify** 是一款开源的 LLM 应用开发平台，支持通过可视化方式编排 AI 工作流、构建聊天助手、Agent 智能体和文本生成应用，无需编写代码即可完成模型集成与产品化落地。

- **GitHub 仓库**：https://github.com/langgenius/dify
- **官网**：https://dify.ai
- **支持平台**：macOS / Windows / Linux
- **环境要求**：Docker 20+ 与 Docker Compose 2.0+

### 1.1 通过 Docker Compose 部署 Dify

Dify 推荐使用 Docker Compose 进行一体化部署，步骤如下：

```bash
git clone https://github.com/langgenius/dify.git
cd dify/docker
cp .env.example .env
docker compose up -d
```

部署完成后，访问 `http://localhost:3000` 即可进入 Dify 管理界面。首次访问时需设置管理员账号与密码。

> **提示**：如在拉取镜像时遇到网络问题，可参考 Dify 官方文档配置镜像加速器，或使用代理。

---

## 2. 配置 Hy3 连接

Hy3 通过 vLLM / SGLang 暴露的是 OpenAI 兼容接口，因此在 Dify 中需通过 **"OpenAI-API-compatible"** 供应商进行接入。

### 2.1 添加模型

1. 登录 Dify 管理界面。
2. 进入右上角头像 → **"设置"** → **"模型供应商"（Model Provider）**。
3. 在供应商列表中找到 **"OpenAI-API-compatible"**，点击 **"添加模型"**。

### 2.2 填写模型配置

按下表填写配置项：

| 配置项 | 值 |
|:---|:---|
| 模型名称（Model Name） | `hy3` |
| 模型展示名称（Display Name） | `Hy3` |
| API Key | `EMPTY` |
| API endpoint URL | `http://host.docker.internal:8000/v1` |
| 模型上下文长度（Context Size） | `256000` |
| 模型类型 | LLM |
| 上限 token 数（Max Tokens） | 按需设置（如 `8192`） |

> **重要**：如果 Dify 部署在 Docker 容器中，容器内的 `127.0.0.1` 指向容器自身，无法访问宿主机上的 Hy3 服务。此时 API endpoint URL 应使用：
> - **Docker Desktop**（macOS / Windows）：`http://host.docker.internal:8000/v1`
> - **Linux 宿主机**：宿主机的实际 IP，如 `http://192.168.x.x:8000/v1`；或在 Dify 的 `docker-compose.yaml` 中为服务配置 `network_mode: host` 后使用 `http://127.0.0.1:8000/v1`

填写完成后点击 **"保存"**。

![Dify 模型配置](./assets/dify-config.png)

保存成功后，可在模型列表中看到 `Hy3`，状态为"可用"。

---

## 3. 第一次对话

完成模型接入后，即可创建第一个聊天应用进行验证。

1. 进入 Dify 主页，点击 **"创建空白应用"**。
2. 应用类型选择 **"聊天助手"**，填写应用名称（如 `Hy3 测试`），点击创建。
3. 在应用设置页面的"模型"下拉框中选择 **`Hy3`**。
4. 在右侧调试面板的输入框中输入：

   ```
   你好，请介绍一下你自己
   ```

5. 点击发送，观察返回结果。

### 预期响应

Hy3 会以中文返回一段自我介绍，说明自己是腾讯混元团队研发的大语言模型，可协助完成各类自然语言任务。响应通常在数秒内开始流式输出，整体内容连贯、风格自然。

![Dify 第一次对话](./assets/dify-first-chat.png)

> **说明**：Dify 的 "OpenAI-API-compatible" 供应商当前不支持透传 `extra_body` 参数，因此通过 Dify 界面发起的请求默认使用 Hy3 的 `no_think`（直接回复）模式。如需启用深度思维链（`reasoning_effort="high"`），请参考 [第 5 节](#5-常见注意事项)中的说明。

---

## 4. 端到端真实任务 Demo

本节演示一个真实场景：使用 Hy3 根据用户输入的主题，自动生成结构化的技术博客大纲。

### 4.1 创建应用

1. 在 Dify 主页点击 **"创建空白应用"**。
2. 应用类型选择 **"文本生成"**（Text Generator），填写应用名称（如 `技术博客大纲生成器`）。

### 4.2 编排提示词

在应用的"提示词"区域填入以下内容：

```
你是一个专业的技术文档撰写助手。请根据用户提供的主题，生成一篇结构清晰的技术博客大纲。主题：{input}
```

其中 `{input}` 为输入变量，需在右侧"输入变量"区域声明该变量。

### 4.3 选择模型并测试

1. 在模型下拉框中选择 **`Hy3`**。
2. 在调试面板的 `input` 变量中填入测试主题：

   ```
   使用 RAG 构建知识库问答系统
   ```

3. 点击 **"运行"**。

### 4.4 预期输出

Hy3 会生成一篇结构化的博客大纲，通常包含以下层级：

- 引言（背景与价值）
- 核心概念（RAG 原理、架构组件）
- 实现步骤（数据准备、向量化、检索、生成）
- 工程实践（框架选型、性能优化）
- 评测与案例分析
- 总结与展望

每个章节会附有简要说明，整体逻辑清晰、内容贴合主题。

![Dify 端到端 Demo](./assets/dify-demo.png)

### 4.5 发布应用

调试效果满意后，点击右上角 **"发布"**。发布后可通过 Dify 提供的 Web 应用链接、嵌入 iframe 或 API 方式对外提供服务，无需额外编码。

---

## 5. 常见注意事项

### 5.1 Docker 网络问题

Dify 默认以 Docker Compose 方式部署，容器网络与宿主机隔离。当容器无法访问宿主机上的 Hy3 服务时，可按以下方式排查与解决：

- **Docker Desktop**（macOS / Windows）：API endpoint URL 使用 `http://host.docker.internal:8000/v1` 替代 `127.0.0.1`。
- **Linux**：使用宿主机物理 IP（如 `http://192.168.x.x:8000/v1`）；或在 Dify 的 `docker-compose.yaml` 中为相关服务设置 `network_mode: host`，即可直接使用 `http://127.0.0.1:8000/v1`。
- **防火墙**：确认宿主机 8000 端口未被防火墙拦截（如 `ufw`、`firewalld`、云服务器安全组）。

### 5.2 API Key 必须非空

Hy3 本地部署无需鉴权，但 Dify 的 "OpenAI-API-compatible" 供应商要求 API Key 字段不能为空。请填入 **`EMPTY`**（大写英文字符串），切勿留空或填写真实密钥。

### 5.3 上下文长度配置

在模型配置中，**模型上下文长度** 建议填写 `256000`，以匹配 Hy3 的 256K 上下文能力。若该值设置过小（如默认的 4096），Dify 会在应用层对长上下文进行截断，导致长文档检索、长对话等场景下信息丢失。

### 5.4 推理模式（reasoning_effort）的局限

Dify 的 "OpenAI-API-compatible" 供应商**不支持**通过界面配置 `extra_body` 参数，因此无法在 Dify 中直接切换 Hy3 的推理模式（`high` / `low` / `no_think`）。如需启用深度思维链，可考虑以下方案：

- 在 vLLM / SGLang 启动参数中固定 `reasoning_effort` 默认值；
- 在 Dify 与 Hy3 之间部署一个轻量代理（如 Nginx / FastAPI），由代理注入 `extra_body` 字段后再转发至 Hy3。

### 5.5 流式输出

如需在应用中获得流式输出体验，请确认在应用设置中开启 **"流式传输"（Streaming）** 选项。Hy3 的 vLLM / SGLang 服务端默认支持 SSE 流式响应，无需额外配置。

### 5.6 Agent 模式与工具调用

Dify 支持 **Agent 模式**，可结合 Hy3 原生的工具调用能力（需在 vLLM / SGLang 启动时指定 `--tool-call-parser hy_v3 --reasoning-parser hy_v3 --enable-auto-tool-choice`）创建智能体应用。在聊天助手应用中开启"Agent"模式后，可为 Hy3 配置内置工具或自定义工具，实现联网搜索、代码执行、知识库检索等能力。

> **注意**：Agent 模式下，Hy3 会自主决定何时调用工具。请确保工具定义清晰、参数描述准确，以提高工具调用的成功率。

---

如需了解更多关于 Hy3 的部署细节与能力说明，请参阅仓库主文档：[Hy3 README](../../README_CN.md)。
