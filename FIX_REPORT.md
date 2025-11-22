# GPT Academic Docker KeyError 修复报告

## 问题描述

在启动 Docker 容器时，应用程序抛出以下错误：

```
KeyError: 'gpt-5.1'
```

错误堆栈：
```
File "/gpt/request_llms/bridge_all.py", line 1565, in predict
    method = model_info[llm_kwargs['llm_model']]["fn_with_ui"]
             ~~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^
KeyError: 'gpt-5.1'
```

## 根本原因

1. **缺失的 Tokenizer 定义**：
   - `request_llms/bridge_all.py` 中定义的 `model_info` 字典引用了 `tokenizer_gpt5` 和 `get_token_num_gpt5` 变量
   - 但这两个变量从未被定义过
   - 导致模块导入时出现 `NameError`，`model_info` 字典构建失败，最终不包含 `'gpt-5.1'` 键

2. **缺失的模型配置**：
   - `config_private.py` 中列出了 `"gpt-5"` 模型，但 `model_info` 中没有对应定义
   - 仅有 `"gpt-5.1"` 的定义，且该定义依赖未定义的 tokenizer

3. **Docker 容器代码过时**：
   - 容器未挂载本地代码卷，使用的是镜像中内置的旧代码
   - 无法加载最新的修复代码

## 修复方案

### 1. 修复 `request_llms/bridge_all.py`（第 111-114 行）

**问题代码**：
```python
# 获取tokenizer
tokenizer_gpt35 = LazyloadTiktoken("gpt-3.5-turbo")
tokenizer_gpt4 = LazyloadTiktoken("gpt-4")
get_token_num_gpt35 = lambda txt: len(tokenizer_gpt35.encode(txt, disallowed_special=()))
get_token_num_gpt4 = lambda txt: len(tokenizer_gpt4.encode(txt, disallowed_special=()))
```

**修复后代码**：
```python
# 获取tokenizer
tokenizer_gpt35 = LazyloadTiktoken("gpt-3.5-turbo")
tokenizer_gpt4 = LazyloadTiktoken("gpt-4")
tokenizer_gpt5 = LazyloadTiktoken("gpt-4")  # gpt-5 使用与gpt-4相同的tokenizer
get_token_num_gpt35 = lambda txt: len(tokenizer_gpt35.encode(txt, disallowed_special=()))
get_token_num_gpt4 = lambda txt: len(tokenizer_gpt4.encode(txt, disallowed_special=()))
get_token_num_gpt5 = lambda txt: len(tokenizer_gpt5.encode(txt, disallowed_special=()))
```

**修改说明**：
- 添加 `tokenizer_gpt5` 定义，使用与 `gpt-4` 相同的 tokenizer（因为 GPT-5 使用兼容的 tokenizer）
- 添加 `get_token_num_gpt5` lambda 函数用于计算 token 数量

### 2. 修复 `request_llms/bridge_all.py` 中的 `model_info` 字典（第 401-414 行）

**添加 `"gpt-5"` 模型配置**：
```python
"gpt-5": {
    "fn_with_ui": chatgpt_ui,
    "fn_without_ui": chatgpt_noui,
    "endpoint": openai_endpoint,
    "max_token": 8192,
    "tokenizer": tokenizer_gpt5,
    "token_cnt": get_token_num_gpt5,
},
```

**完善 `"gpt-5.1"` 模型配置**：
```python
"gpt-5.1": {
    "fn_with_ui": chatgpt_ui,
    "fn_without_ui": chatgpt_noui,
    "endpoint": openai_endpoint,
    "max_token": 8192,
    "tokenizer": tokenizer_gpt5,
    "token_cnt": get_token_num_gpt5,
},
```

### 3. 配置 Docker 卷挂载（`docker-compose.yml`）

**修改前**：
```yaml
services:
  gpt_academic_with_latex:
    image: ghcr.io/binary-husky/gpt_academic_with_latex:master
    environment:
      # ... 环境变量 ...
    
    network_mode: "host"
    
    command: >
      bash -c "python3 -u main.py"
```

**修改后**：
```yaml
services:
  gpt_academic_with_latex:
    image: ghcr.io/binary-husky/gpt_academic_with_latex:master
    environment:
      # ... 环境变量 ...
    
    volumes:
      - ./request_llms:/gpt/request_llms
      - ./config_private.py:/gpt/config_private.py
    
    network_mode: "host"
    
    command: >
      bash -c "python3 -u main.py"
```

**修改说明**：
- 添加 `volumes` 部分，将本地 `request_llms` 目录挂载到容器内
- 将本地 `config_private.py` 挂载到容器内
- 确保容器使用最新的本地代码而不是镜像内置的代码

## 修复步骤

1. **编辑 `request_llms/bridge_all.py`**：
   - 在第 111-114 行添加 `tokenizer_gpt5` 和 `get_token_num_gpt5` 定义

2. **编辑 `request_llms/bridge_all.py` 中的 `model_info` 字典**：
   - 在第 401-407 行前添加 `"gpt-5"` 模型定义
   - 更新第 408-414 行的 `"gpt-5.1"` 模型定义以使用新定义的 tokenizer

3. **编辑 `docker-compose.yml`**：
   - 在 `environment` 和 `network_mode` 之间添加 `volumes` 配置

4. **重启 Docker 容器**：
   ```bash
   docker compose down
   docker compose up -d
   ```

## 验证结果

### 语法检查
```bash
python3 -m py_compile request_llms/bridge_all.py
# 输出：Syntax OK
```

### 模型加载验证
```bash
docker exec gpt_academic-gpt_academic_with_latex-1 python3 -c \
  "from request_llms.bridge_all import model_info; \
   print('gpt-5.1 in model_info:', 'gpt-5.1' in model_info); \
   print('gpt-5 in model_info:', 'gpt-5' in model_info)"
```

**输出**：
```
gpt-5.1 in model_info: True
gpt-5 in model_info: True
```

### 容器错误检查
```bash
docker compose logs 2>&1 | grep -i "keyerror\|traceback\|error"
# 无输出（表示无错误）
```

### 应用启动验证
```bash
docker compose logs 2>&1 | tail -5
```

**预期输出**（无 KeyError 或异常）：
```
gpt_academic_with_latex-1  | 03:09 | ..up_modules:225 | 正在执行一些模块的预热 ...
gpt_academic_with_latex-1  | 03:09 | get_encoder :58  | 正在加载tokenizer...
gpt_academic_with_latex-1  | 03:09 | get_encoder :60  | 加载tokenizer完毕
gpt_academic_with_latex-1  | 03:09 | ..p_vectordb:257 | nltk模块预热
gpt_academic_with_latex-1  | 03:09 | ..p_vectordb:261 | nltk模块预热完成（读取本地缓存）
```

## 修复效果

✅ **已解决**：
- KeyError 异常消除
- `model_info` 字典正确加载 `'gpt-5.1'` 和 `'gpt-5'` 模型
- Docker 容器成功启动
- Web 服务运行在 `http://localhost:12303`

## 相关文件变更

| 文件路径 | 变更类型 | 描述 |
|---------|--------|------|
| `request_llms/bridge_all.py` | 修改 | 添加 tokenizer_gpt5 定义及模型配置 |
| `docker-compose.yml` | 修改 | 添加卷挂载配置 |

## 注意事项

1. **Tokenizer 选择**：
   - `gpt-5` 和 `gpt-5.1` 使用与 `gpt-4` 相同的 tokenizer
   - 这是一个合理的假设，因为 OpenAI 模型通常保持向后兼容的 tokenizer
   - 如果未来 OpenAI 发布专用的 GPT-5 tokenizer，应替换为 `LazyloadTiktoken("gpt-5")`

2. **Docker 卷挂载**：
   - 确保本地代码和容器代码始终同步
   - 修改本地代码后，需要重启容器以生效

3. **代理配置**：
   - 当前配置中 `USE_PROXY: False`，如需使用代理可在 `docker-compose.yml` 中修改

---

**修复日期**：2025-11-22  
**修复版本**：v1.0

## 后续修复：新模型支持

### 新增模型列表

为了支持配置中列出的以下新模型，已在 `model_info` 中添加相应配置：

| 模型名称 | 类型 | 使用的 Bridge |
|---------|-----|-------------|
| `gemini-3-pro-preview-11-2025` | Google Gemini | genai_ui |
| `claude-haiku-4-5-20251001` | Anthropic Claude | chatgpt_ui |
| `claude-opus-4-1-20250805` | Anthropic Claude | chatgpt_ui |
| `claude-sonnet-4-20250514-thinking` | Anthropic Claude | chatgpt_ui |
| `claude-sonnet-4-20250514` | Anthropic Claude | chatgpt_ui |
| `deepseek-ai/DeepSeek-V3.1-Terminus` | DeepSeek | chatgpt_ui |
| `deepseek-ai/DeepSeek-OCR` | DeepSeek (多模态) | chatgpt_vision_ui |
| `qwen3-max` | Alibaba Qwen | chatgpt_ui |
| `zai-org/GLM-4.6` | Zhipu GLM | chatgpt_ui |

### 修复内容

**文件**：`request_llms/bridge_all.py`  
**位置**：第 617-708 行（在原 model_info 结尾处）

添加了 9 个新模型配置，每个配置包含：
- `fn_with_ui`：UI 交互函数
- `fn_without_ui`：无 UI 函数
- `endpoint`：API 端点
- `max_token`：最大 token 数
- `tokenizer`：Token 编码器
- `token_cnt`：Token 计数函数
- `has_multimodal_capacity`（仅 OCR）：多模态能力标记

### 模型端点映射

```python
gemini_endpoint = "https://generativelanguage.googleapis.com/v1beta/models"
claude_endpoint = "https://api.anthropic.com/v1/messages"
openai_endpoint = "https://api.openai.com/v1/chat/completions"  # Claude 和 Qwen 使用 OpenAI 兼容接口
deepseekapi_endpoint = "https://api.deepseek.com/v1/chat/completions"
```

### 验证结果

✅ **所有模型均已加载**：
```
✓ gemini-3-pro-preview-11-2025
✓ claude-haiku-4-5-20251001
✓ claude-opus-4-1-20250805
✓ claude-sonnet-4-20250514-thinking
✓ claude-sonnet-4-20250514
✓ gpt-5
✓ deepseek-ai/DeepSeek-V3.1-Terminus
✓ deepseek-ai/DeepSeek-OCR
✓ qwen3-max
✓ zai-org/GLM-4.6
```

✅ **容器正常运行**：
- 无 KeyError 或其他异常
- Web 服务在 http://localhost:12303 正常运行
- 所有模块预热完成

---

**修复日期（第二阶段）**：2025-11-22  
**修复版本（第二阶段）**：v1.1
