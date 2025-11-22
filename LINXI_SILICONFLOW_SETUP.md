# Linxi 和硅基流动 API 配置指南

## 1. Linxi API 配置

### 1.1 获取 API Key

访问 [Linxi 官网](https://linxi.chat/) 注册账号并获取 API Key。

- **官网**：https://linxi.chat/
- **接口文档**：https://linxi.apifox.cn/
- **API 端点**：https://linxi.chat/v1/chat/completions
- **特点**：
  - 无需科学上网，全球直连
  - 支持市面上所有主流大模型
  - 一个 API Key 全模型通用
  - 无封号风险

### 1.2 配置方式

#### 方式一：在 `config_private.py` 中配置

```python
# 在 config_private.py 添加
LINXI_API_KEY = "your-linxi-api-key-here"

# 支持多个 API Key（用逗号分隔）
LINXI_API_KEY = "key1,key2,key3"
```

#### 方式二：在 `docker-compose.yml` 中配置

```yaml
environment:
  LINXI_API_KEY: "your-linxi-api-key-here"
```

#### 方式三：环境变量

```bash
export LINXI_API_KEY="your-linxi-api-key-here"
```

### 1.3 支持的模型

Linxi 支持市面上所有主流大模型，包括但不限于：

- **OpenAI**: gpt-4, gpt-4-turbo, gpt-3.5-turbo, gpt-5 等
- **Anthropic Claude**: claude-3-opus, claude-3-sonnet, claude-3-haiku 等
- **Google Gemini**: gemini-pro, gemini-1.5-pro, gemini-2.0-flash 等
- **DeepSeek**: deepseek-chat, deepseek-coder, deepseek-v3 等
- **Alibaba Qwen**: qwen-max, qwen-turbo, qwen-plus 等
- **其他**: MJ, 豆包等多个平台的模型

### 1.4 添加模型到 GPT Academic

在 `config_private.py` 中添加：

```python
AVAIL_LLM_MODELS = [
    # ... 其他模型 ...
    "linxi-gpt-4",           # 使用 Linxi 提供的 GPT-4
    "linxi-gpt-5",           # 使用 Linxi 提供的 GPT-5
    "linxi-claude-opus",     # 使用 Linxi 提供的 Claude
    "linxi-gemini-pro",      # 使用 Linxi 提供的 Gemini
    "linxi-qwen-max",        # 使用 Linxi 提供的 Qwen
]

LLM_MODEL = "linxi-gpt-4"  # 设置默认模型
```

---

## 2. 硅基流动 API 配置

### 2.1 获取 API Key

访问 [硅基流动官网](https://www.siliconflow.cn/) 注册账号并获取 API Key。

- **官网**：https://www.siliconflow.cn/
- **接口文档**：https://docs.siliconflow.cn/
- **API 端点**：https://api.siliconflow.cn/v1/chat/completions
- **特点**：
  - 支持联网搜索
  - 支持函数调用
  - 支持思维链模式（Reasoning）
  - 支持多模态输入

### 2.2 配置方式

#### 方式一：在 `config_private.py` 中配置

```python
# 在 config_private.py 添加
SILICONFLOW_API_KEY = "your-siliconflow-api-key-here"

# 支持多个 API Key（用逗号分隔）
SILICONFLOW_API_KEY = "key1,key2,key3"
```

#### 方式二：在 `docker-compose.yml` 中配置

```yaml
environment:
  SILICONFLOW_API_KEY: "your-siliconflow-api-key-here"
```

#### 方式三：环境变量

```bash
export SILICONFLOW_API_KEY="your-siliconflow-api-key-here"
```

### 2.3 支持的模型

硅基流动支持多个高质量模型：

**理性推理模型**（支持 CoT/思维链）：
- `deepseek-ai/DeepSeek-V3.1-Terminus` - 深度求索 V3.1
- `deepseek-ai/DeepSeek-V3.2-Exp`
- `zai-org/GLM-4.6` - 智谱 GLM-4.6
- `Qwen/Qwen3-32B` - 阿里 Qwen3
- `tencent/Hunyuan-A13B-Instruct` - 腾讯混元

**视觉理解模型**（多模态）：
- `zai-org/GLM-4.5V` - 智谱视觉模型
- `Qwen/Qwen-Vision`

**超大规模模型**：
- `Qwen/Qwen3-235B-A22B` - 2350亿参数 Qwen3
- `Qwen/Qwen3-Next-80B-A3B`

### 2.4 添加模型到 GPT Academic

在 `config_private.py` 中添加：

```python
AVAIL_LLM_MODELS = [
    # ... 其他模型 ...
    "siliconflow-deepseek-v3",      # 硅基流动的 DeepSeek V3
    "siliconflow-qwen3-32b",        # 硅基流动的 Qwen3
    "siliconflow-glm-4.6",          # 硅基流动的 GLM-4.6
    "siliconflow-qwen-vision",      # 硅基流动的视觉模型
]

LLM_MODEL = "siliconflow-deepseek-v3"  # 设置默认模型
```

---

## 3. 在 Docker Compose 中完整配置

```yaml
services:
  gpt_academic_with_latex:
    image: ghcr.io/binary-husky/gpt_academic_with_latex:master
    environment:
      # Linxi 配置
      LINXI_API_KEY: "your-linxi-api-key"
      
      # 硅基流动配置
      SILICONFLOW_API_KEY: "your-siliconflow-api-key"
      
      # 模型列表
      AVAIL_LLM_MODELS: |
        [
          "linxi-gpt-4",
          "linxi-gpt-5",
          "linxi-claude-opus",
          "siliconflow-deepseek-v3",
          "siliconflow-qwen3-32b",
          "siliconflow-glm-4.6"
        ]
      
      # 默认模型
      LLM_MODEL: "linxi-gpt-4"
      
      # 其他必要配置
      API_KEY: "your-openai-key"
      USE_PROXY: "False"
      
    volumes:
      - ./request_llms:/gpt/request_llms
      - ./config_private.py:/gpt/config_private.py
    
    network_mode: "host"
    command: >
      bash -c "python3 -u main.py"
```

---

## 4. 测试配置

### 4.1 本地测试

```bash
cd /home/dave_paine/gpt_academic

# 测试 Linxi 配置
python3 << 'EOF'
from request_llms.bridge_all import model_info
linxi_models = [m for m in model_info if 'linxi' in m.lower()]
print(f"Linxi 模型数量：{len(linxi_models)}")
print(f"Linxi 模型列表：{linxi_models[:5]}")
EOF

# 测试硅基流动配置
python3 << 'EOF'
from request_llms.bridge_all import model_info
sf_models = [m for m in model_info if 'siliconflow' in m.lower()]
print(f"硅基流动模型数量：{len(sf_models)}")
print(f"硅基流动模型列表：{sf_models[:5]}")
EOF
```

### 4.2 Docker 测试

```bash
# 进入容器
docker exec -it gpt_academic-gpt_academic_with_latex-1 bash

# 测试模型加载
python3 -c "from request_llms.bridge_all import model_info; print('linxi-gpt-4' in model_info)"
python3 -c "from request_llms.bridge_all import model_info; print('siliconflow-deepseek-v3' in model_info)"
```

---

## 5. 常见问题

### Q: Linxi 和硅基流动有什么区别？

**A:** 
- **Linxi**：中转 API，聚合市面上所有主流大模型，支持模型更多更全面
- **硅基流动**：垂直专注于自研和合作模型，支持联网搜索和思维链模式

### Q: API Key 怎样获取？

**A:**
- Linxi：访问 https://linxi.chat/ 注册并在后台生成 API Key
- 硅基流动：访问 https://www.siliconflow.cn/ 注册并在后台生成 API Key

### Q: 支持混合使用两个 API 吗？

**A:** 可以。在 `AVAIL_LLM_MODELS` 中添加来自两个平台的模型，系统会自动根据模型名称前缀路由到对应的 API。

### Q: 如何切换默认模型？

**A:** 修改 `config_private.py` 或 `docker-compose.yml` 中的 `LLM_MODEL` 字段。

### Q: 模型调用失败怎么办？

**A:**
1. 检查 API Key 是否正确
2. 检查网络连接
3. 查看容器日志：`docker compose logs`
4. 确认模型名称是否在支持列表中

---

## 6. 后续支持

如需添加更多 Linxi 或硅基流动的模型，可以：

1. 在 `config_private.py` 中的 `AVAIL_LLM_MODELS` 添加模型名称
2. 在 `docker-compose.yml` 中更新模型列表
3. 重启应用：`docker compose down && docker compose up -d`

---

**最后更新**：2025-11-22  
**版本**：v1.0
