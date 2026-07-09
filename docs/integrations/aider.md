# Aider 接入 Hy3 指南

[Aider](https://github.com/Aider-AI/aider) 是一款开源的命令行 AI 编程助手，能够直接在你的本地 git 仓库中进行代码编辑、Bug 修复和重构，并自动完成 git 提交。本指南介绍如何将 Aider 接入到本地部署的 Hy3 模型，使其作为你的终端编程助手。

> 前置条件：请先按照 Hy3 主文档使用 [vLLM](../../README_CN.md#使用-vllm-推理) 或 [SGLang](../../README_CN.md#使用-sglang-推理) 部署 OpenAI 兼容 API 服务，并确保服务监听在 `http://127.0.0.1:8000/v1`。

---

## 1. 安装与版本要求

Aider 是开源的命令行 AI 编程助手，支持在终端中直接与本地代码库协作。

- **Python 版本**：3.10 及以上
- **操作系统**：macOS / Windows / Linux
- **GitHub 仓库**：https://github.com/Aider-AI/aider

通过 pip 安装：

```bash
pip install aider-chat
```

安装完成后，可通过以下命令确认版本：

```bash
aider --version
```

> 建议在虚拟环境（如 `venv`、`conda`）中安装，以避免与系统其他 Python 包冲突。

---

## 2. 配置 Hy3 连接

Hy3 通过 vLLM/SGLang 部署后提供 OpenAI 兼容 API，Aider 可以像接入 OpenAI 一样接入 Hy3。连接信息如下：

| 配置项 | 值 |
|:---|:---|
| Base URL | `http://127.0.0.1:8000/v1` |
| Model 名称 | `hy3` |
| API Key | `EMPTY`（本地部署无需鉴权，但需为非空字符串） |

Aider 提供两种配置方式。

### 方式一：环境变量

适合长期使用，可在 shell 配置文件（如 `~/.bashrc`、`~/.zshrc`）中持久化：

```bash
export OPENAI_API_BASE=http://127.0.0.1:8000/v1
export OPENAI_API_KEY=EMPTY
```

设置完成后，直接在项目目录中运行：

```bash
aider --model openai/hy3
```

### 方式二：命令行参数

适合临时使用或在不同模型间切换：

```bash
aider --openai-api-base http://127.0.0.1:8000/v1 --openai-api-key EMPTY --model openai/hy3
```

### 说明

- **`openai/` 前缀**：Aider 通过 provider 前缀来识别模型类型。Hy3 使用 OpenAI 兼容协议，因此 model 名需写作 `openai/hy3`，Aider 才会将其路由到 OpenAI provider 并使用 `OPENAI_API_BASE` 指定的地址。
- **协议兼容性**：Hy3 实现的是 OpenAI Chat Completions 兼容协议，无需额外的适配层。
- **推理模式**：Hy3 支持 `reasoning_effort` 参数（`high` / `low` / `no_think`）。在 Aider 中可通过配置文件 `.aider.conf.yml` 传递 `extra_body` 参数，复杂编程任务建议使用 `high`。

![Aider 配置启动](./assets/aider-startup.png)

---

## 3. 第一次对话

完成配置后，进入你希望协作的项目目录（需为 git 仓库），运行：

```bash
aider --model openai/hy3
```

Aider 启动后会自动扫描当前 git 仓库中的文件，并进入交互式对话框。

在提示符中输入：

```
hello, can you help me with this project?
```

预期响应：

1. Aider 将该消息连同仓库结构摘要发送给 Hy3。
2. Hy3 返回对项目的简要分析，并询问你希望进行哪方面的工作。
3. 由于此时未通过 `add` 命令将任何文件加入会话上下文，Aider 不会修改任何文件。

![Aider 第一次对话](./assets/aider-first-chat.png)

> 提示：可以使用 `/add <文件路径>` 将文件加入会话上下文，使用 `/drop <文件路径>` 将文件移出上下文。

---

## 4. 端到端真实任务 Demo

下面通过一个完整的示例演示 Aider + Hy3 修复代码 Bug 的全流程。

### 步骤 1：创建一个有 Bug 的 Python 文件

在项目目录中创建 `calc.py`，内容如下（`add` 函数错误地使用了减法）：

```python
def add(a, b):
    return a - b


if __name__ == "__main__":
    print(add(2, 3))  # 期望输出 5，实际输出 -1
```

### 步骤 2：启动 Aider 并加载该文件

```bash
aider --model openai/hy3 calc.py
```

启动后，Aider 会将 `calc.py` 加入上下文，并展示文件的 token 占用情况。

### 步骤 3：描述任务

在 Aider 交互框中输入：

```
这个 add 函数有 bug，请修复它
```

### 步骤 4：预期流程

1. **分析代码**：Aider 将 `calc.py` 的内容与你的指令一起发送给 Hy3。
2. **定位 Bug**：Hy3 识别出 `add` 函数体内 `return a - b` 应为 `return a + b`。
3. **生成修复**：Hy3 通过 SEARCH/REPLACE 块输出修改方案，Aider 将其应用到 `calc.py`。
4. **自动提交**：修改完成后，Aider 自动生成一条 git commit（默认开启），提交信息类似 `Fix bug in add function`。

完成后，`calc.py` 的内容应更新为：

```python
def add(a, b):
    return a + b


if __name__ == "__main__":
    print(add(2, 3))  # 期望输出 5，实际输出 -1
```

运行验证：

```bash
python calc.py
# 输出: 5
```

![Aider 修复代码](./assets/aider-demo.png)

> 提示：Aider 默认会在每次修改后自动 `git commit`。可在启动时使用 `--no-auto-commits` 关闭此行为，或使用 `/undo` 命令撤销最近一次提交。

---

## 5. 常见注意事项

- **连接失败**
  确认 Hy3 服务已启动并监听在 `http://127.0.0.1:8000/v1`，可通过 `curl http://127.0.0.1:8000/v1/models` 验证。若服务部署在其他地址或端口，请同步修改 `OPENAI_API_BASE` 或 `--openai-api-base`。

- **模型名必须加 `openai/` 前缀**
  Hy3 使用 OpenAI 兼容协议，model 名必须写作 `openai/hy3`。如果直接写 `hy3`，Aider 无法识别其为 OpenAI 兼容模型，会报模型未找到或 provider 缺失的错误。

- **API Key 必须设置为 `EMPTY`**
  本地部署的 Hy3 无需鉴权，但 Aider 要求 API Key 为非空字符串。请显式设置为 `EMPTY`（或其他任意非空字符串），不要留空。

- **自动 git commit**
  Aider 默认会在每次文件修改后自动 `git commit`。如需禁用，启动时添加 `--no-auto-commits`；如需查看 diff，可在会话中使用 `/diff` 命令。

- **`/ask` 与 `/code` 模式**
  - `/ask` 模式：只回答问题、解释代码，不修改文件。
  - `/code` 模式（默认）：会根据指令直接修改文件并提交。在不确定改动前，可先用 `/ask` 询问思路，再切换到 `/code` 执行。

- **上下文超长错误**
  Hy3 支持最长 256K 上下文，但仍可能因加载过多文件而超限。遇到上下文超长错误时：
  - 使用 `/tokens` 命令查看当前 token 使用情况及各文件占用。
  - 使用 `/drop <文件路径>` 移除不再需要的文件。
  - 使用 `/clear` 清空当前会话上下文重新开始。
