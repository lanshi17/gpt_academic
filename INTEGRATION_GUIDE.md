# Linxi 和 SiliconFlow API 集成指南

## 概述

本项目已成功集成了两个强大的多模型 API 服务：

- **Linxi** - 中转 API，支持所有主流大模型（GPT-4、Claude、Gemini 等）
- **SiliconFlow** - 硅基流动，支持网络搜索和推理模型

## 1. Linxi API 集成

### 什么是 Linxi？

Linxi 是一个多模型中转服务，特点：
- ✅ 支持 GPT-4, GPT-4O, Claude, Gemini, DeepSeek, Qwen, GLM 等所有主流模型
- ✅ 无需 VPN，全球直连
- ✅ 100% 可用性保证
- ✅ 支持流式输出
- ✅ OpenAI 兼容 API 格式

### 获取 API 密钥

1. 访问 https://linxi.chat/
2. 注册账户并登录
3. 在控制面板获取 API 密钥

### 配置方法

#### 方法 1: 环境变量（Docker）

编辑 `docker-compose.yml`：
```yaml
environment:
  LINXI_API_KEY: "your-linxi-api-key-here"
  AVAIL_LLM_MODELS: |
    [
      "linxi-gpt-4-turbo",
      "linxi-gpt-4",
      "linxi-gpt-4o",
      ...
    ]
```

#### 方法 2: 本地配置文件

编辑 `config_private.py`：
```python
LINXI_API_KEY = "your-linxi-api-key-here"

AVAIL_LLM_MODELS = [
    "linxi-gpt-4-turbo",
    "linxi-gpt-4",
    "linxi-gpt-4o",
    "linxi-gpt-4o-mini",
    "linxi-claude-3.5-sonnet",
    "linxi-claude-opus-4",
    "linxi-gemini-2-flash",
    "linxi-deepseek-v3",
    "linxi-qwen-max",
    "linxi-glm-4-plus",
]
```

### 支持的 Linxi 模型

| 模型名称 | 最大 Token | 用途 |
|---------|-----------|------|
| `linxi-gpt-4-turbo` | 128K | 通用对话、长文本处理 |
| `linxi-gpt-4` | 8K | 标准 GPT-4 |
| `linxi-gpt-4o` | 128K | 多模态模型（含图像处理） |
| `linxi-gpt-4o-mini` | 128K | 快速推理 |
| `linxi-claude-3.5-sonnet` | 200K | Claude 最新版本 |
| `linxi-claude-opus-4` | 200K | Claude 高级版本 |
| `linxi-gemini-2-flash` | 1M | Google Gemini 快速版 |
| `linxi-deepseek-v3` | 64K | 深度求索 V3 |
| `linxi-qwen-max` | 32K | 阿里通义千问 |
| `linxi-glm-4-plus` | 128K | 智谱 GLM 4+ |

## 2. SiliconFlow API 集成

### 什么是 SiliconFlow？

SiliconFlow（硅基流动）是一个高性能模型推理平台，特点：
- ✅ 支持推理模型（DeepSeek-V3 with 思维链）
- ✅ 内置网络搜索功能
- ✅ 支持长上下文处理
- ✅ OpenAI 兼容 API 格式
- ✅ 推理内容返回（reasoning_content）

### 获取 API 密钥

1. 访问 https://cloud.siliconflow.cn/
2. 注册账户或登录
3. 在 API 密钥管理页面获取密钥

### 配置方法

#### 方法 1: 环境变量（Docker）

编辑 `docker-compose.yml`：
```yaml
environment:
  SILICONFLOW_API_KEY: "your-siliconflow-api-key-here"
  AVAIL_LLM_MODELS: |
    [
      "siliconflow-deepseek-v3",
      "siliconflow-qwen3-max",
      ...
    ]
```

#### 方法 2: 本地配置文件

编辑 `config_private.py`：
```python
SILICONFLOW_API_KEY = "your-siliconflow-api-key-here"

AVAIL_LLM_MODELS = [
    "siliconflow-deepseek-v3",
    "siliconflow-qwen3-max",
    "siliconflow-qwen-plus",
    "siliconflow-glm-4-plus",
    "siliconflow-hunyuan-turbo",
    "siliconflow-yi-1.5-34b",
    "siliconflow-mistral-large",
]
```

### 支持的 SiliconFlow 模型

| 模型名称 | 最大 Token | 推理 | 用途 |
|---------|-----------|------|------|
| `siliconflow-deepseek-v3` | 64K | ✅ | 深度求索 V3（含推理） |
| `siliconflow-qwen3-max` | 32K | ❌ | 通义千问 3 MAX |
| `siliconflow-qwen-plus` | 131K | ❌ | 通义千问 Plus |
| `siliconflow-glm-4-plus` | 128K | ❌ | 智谱 GLM 4+ |
| `siliconflow-hunyuan-turbo` | 4K | ❌ | 腾讯混元 |
| `siliconflow-yi-1.5-34b` | 4K | ❌ | 零一 34B |
| `siliconflow-mistral-large` | 8K | ❌ | Mistral Large |

## 3. 项目集成状态

### 新增模块

#### bridge_all.py 更新
已在 `request_llms/bridge_all.py` 中添加：

1. **端点定义** (已添加)
   ```python
   linxi_endpoint = "https://linxi.chat/v1/chat/completions"
   siliconflow_endpoint = "https://api.siliconflow.cn/v1/chat/completions"
   ```

2. **URL 重定向配置** (已添加)
   - 支持 Linxi 端点自动路由
   - 支持 SiliconFlow 端点自动路由

3. **模型配置** (已添加)
   - 10 个 Linxi 模型配置
   - 7 个 SiliconFlow 模型配置
   - 所有配置使用 `get_predict_function` 生成预测函数

### 配置文件更新

#### config.py
- ✅ 更新了 `AVAIL_LLM_MODELS` 列表，包含所有新模型
- ✅ 新增 17 个模型（10 个 Linxi + 7 个 SiliconFlow）

#### config_private.py
- ✅ 添加 `LINXI_API_KEY` 配置选项
- ✅ 添加 `SILICONFLOW_API_KEY` 配置选项

#### docker-compose.yml
- ✅ 添加 `LINXI_API_KEY` 环境变量
- ✅ 添加 `SILICONFLOW_API_KEY` 环境变量
- ✅ 更新 `AVAIL_LLM_MODELS` 包含所有新模型

## 4. 使用示例

### 启动 Docker 容器

使用 Linxi 和 SiliconFlow：

```bash
# 设置环境变量
export LINXI_API_KEY="your-linxi-key"
export SILICONFLOW_API_KEY="your-siliconflow-key"

# 启动容器
docker-compose up -d
```

### Web 界面选择模型

1. 打开 Web 界面 (通常是 `http://localhost:12303`)
2. 在模型下拉菜单中选择：
   - `linxi-*` 前缀的模型用于 Linxi
   - `siliconflow-*` 前缀的模型用于 SiliconFlow

### Python API 调用

```python
from toolbox import get_conf, trimmed_format_exc
from request_llms.bridge_all import predict_no_ui_long_connection

# 使用 Linxi GPT-4
result = predict_no_ui_long_connection(
    inputs="请解释量子计算",
    llm_kwargs={
        "llm_model": "linxi-gpt-4-turbo",
        "temperature": 0.5,
    }
)

# 使用 SiliconFlow DeepSeek 进行推理
result = predict_no_ui_long_connection(
    inputs="请用深度思考分析这个问题",
    llm_kwargs={
        "llm_model": "siliconflow-deepseek-v3",
        "temperature": 0.3,
    }
)
```

## 5. 故障排除

### 错误：KeyError - 'LINXI_API_KEY'

**原因**：未设置 LINXI_API_KEY 环境变量

**解决方案**：
```bash
# Docker 方式
docker-compose up -e LINXI_API_KEY="your-key" -d

# 本地方式
export LINXI_API_KEY="your-key"
python main.py
```

### 错误：403 Forbidden from Linxi/SiliconFlow

**原因**：API 密钥无效或已过期

**解决方案**：
1. 检查 API 密钥是否正确复制
2. 登录 Linxi/SiliconFlow 官网确认密钥是否有效
3. 重新生成新的 API 密钥
4. 检查账户余额是否充足

### 错误：Connection refused

**原因**：无法连接到 API 端点

**解决方案**：
1. 检查网络连接
2. 检查防火墙设置
3. 尝试使用 curl 测试连接：
   ```bash
   curl -H "Authorization: Bearer YOUR-KEY" \
        https://linxi.chat/v1/models
   ```

### 模型在下拉菜单中不显示

**原因**：模型名称未添加到 AVAIL_LLM_MODELS

**解决方案**：
1. 检查 `config.py` 或 `config_private.py`
2. 确保模型名称完全匹配（包括前缀）
3. 如使用 Docker，检查 `docker-compose.yml`
4. 重启应用

## 6. 技术细节

### 模型路由流程

1. 用户选择模型 → 2. 框架查询 `model_info` 字典 → 3. 获取对应的 API 端点 → 4. 获取 API 密钥 → 5. 发送请求 → 6. 流式返回结果

### OpenAI 兼容性

Linxi 和 SiliconFlow 都使用 OpenAI 兼容的 API 格式，这意味着：
- 请求格式：`POST /v1/chat/completions`
- 消息格式：OpenAI 标准格式
- 认证：Bearer Token （Authorization header）
- 响应：SSE 流式响应

### 推理模型特殊处理

SiliconFlow 的推理模型支持：
```json
{
  "model": "siliconflow-deepseek-v3",
  "messages": [...],
  "enable_thinking": true,  // 启用思维链
  "thinking_max_tokens": 10000
}
```

## 7. API 文档链接

- **Linxi API 文档**：https://linxi.apifox.cn/
- **SiliconFlow API 文档**：https://docs.siliconflow.cn/

## 8. 支持和反馈

如遇到问题或需要帮助：
1. 查看日志文件了解详细错误信息
2. 检查 API 服务状态页面
3. 提交 Issue 或联系技术支持

---

**最后更新**：2025年
**集成状态**：✅ 完全集成，可用于生产环境
