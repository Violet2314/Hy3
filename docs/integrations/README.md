# Hy3 集成指南

本目录包含在主流 AI 工具中使用 Hy3 的接入指南。Hy3 是腾讯混元团队开发的 295B 参数 MoE 大模型，通过 vLLM/SGLang 部署后提供 OpenAI 兼容 API，可便捷地接入各类 AI 工具链。

## 前置条件

在接入任意工具之前，请先通过 vLLM 或 SGLang 完成 Hy3 的本地部署，确保 OpenAI 兼容 API 服务正常运行。

具体部署方式请参考主仓库 [README.md](../../README.md) 的 Deployment 章节（中文版见 [README_CN.md](../../README_CN.md) 的「推理和部署」章节），其中包含 vLLM 与 SGLang 两种部署方案的详细说明。

部署完成后，请确认 OpenAI 兼容 API 可在以下地址访问：

```
http://127.0.0.1:8000/v1
```

## 工具指南列表

按工具分类组织的接入指南如下：

| 分类 | 工具 | 说明 | 链接 |
|------|------|------|------|
| AI IDE | Cursor | AI 驱动的代码编辑器 | [指南](./cursor.md) |
| AI CLI 工具 | Aider | 命令行 AI 编程助手 | [指南](./aider.md) |
| VS Code 插件 | Cline | 自主编程 AI 代理插件 | [指南](./cline.md) |
| VS Code 插件 | Continue | 可扩展的 AI 代码助手插件 | [指南](./continue.md) |
| 低代码平台 | Dify | 开源 AI 应用开发平台 | [指南](./dify.md) |

## 通用连接信息

所有工具均通过 OpenAI 兼容协议接入 Hy3，连接参数速查表如下：

| 参数 | 值 |
|------|------|
| Base URL | `http://127.0.0.1:8000/v1` |
| Model 名称 | `hy3` |
| API Key | `EMPTY` |

**补充说明：**

- 本地部署无需鉴权，API Key 固定填写 `EMPTY`（部分工具要求非空，请勿留空）。
- 推荐采样参数：`temperature=0.9`、`top_p=1.0`。
- 推理模式可通过 `extra_body={"chat_template_kwargs": {"reasoning_effort": "high"/"low"/"no_think"}}` 控制（需工具支持透传 `extra_body`）。
- 工具调用需 vLLM/SGLang 启动时配置 `tool-call-parser: hy_v3` 与 `reasoning-parser: hy_v3`。
- 上下文长度：256K。
