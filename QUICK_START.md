# 快速参考 - GPT Academic 模型修复

## 问题快速诊断

如果遇到类似 `KeyError: '<model-name>'` 的错误，可能是模型未在 `model_info` 中定义。

## 修复已应用

✅ 已解决的问题：
- `KeyError: 'gpt-5.1'` ✓
- 缺失的新模型配置 ✓
- Docker 代码同步 ✓

## 添加新模型的步骤

如需添加新模型，按以下步骤操作：

### 1. 编辑 `request_llms/bridge_all.py`

在 `model_info` 字典中添加新模型配置：

```python
"new-model-name": {
    "fn_with_ui": chatgpt_ui,              # UI 函数
    "fn_without_ui": chatgpt_noui,         # 无 UI 函数
    "endpoint": openai_endpoint,            # API 端点
    "max_token": 8192,                      # 最大 token
    "tokenizer": tokenizer_gpt4,            # Token 编码器
    "token_cnt": get_token_num_gpt4,        # Token 计数函数
},
```

### 2. 更新配置

在 `config_private.py` 或 `docker-compose.yml` 中的 `AVAIL_LLM_MODELS` 列表中添加模型名称

### 3. 重启容器

```bash
docker compose down
docker compose up -d
```

### 4. 验证

```bash
docker exec gpt_academic-gpt_academic_with_latex-1 python3 -c \
  "from request_llms.bridge_all import model_info; print('new-model-name' in model_info)"
```

## 常用的 Bridge 函数映射

| 模型类型 | UI 函数 | 无 UI 函数 | 适用模型 |
|---------|--------|-----------|--------|
| OpenAI | `chatgpt_ui` | `chatgpt_noui` | GPT-3.5, GPT-4, GPT-5 |
| OpenAI Vision | `chatgpt_vision_ui` | `chatgpt_vision_noui` | GPT-4V |
| Google Gemini | `genai_ui` | `genai_noui` | Gemini 系列 |
| Claude | `claude_ui` | `claude_noui` | Claude 系列 |
| Zhipu | `zhipu_ui` | `zhipu_noui` | GLM 系列 |
| Cohere | `cohere_ui` | `cohere_noui` | Cohere 系列 |

## 常用的 API 端点

```python
openai_endpoint = "https://api.openai.com/v1/chat/completions"
gemini_endpoint = "https://generativelanguage.googleapis.com/v1beta/models"
claude_endpoint = "https://api.anthropic.com/v1/messages"
deepseekapi_endpoint = "https://api.deepseek.com/v1/chat/completions"
```

## 常用的 Token 编码器

```python
tokenizer_gpt35 = LazyloadTiktoken("gpt-3.5-turbo")
tokenizer_gpt4 = LazyloadTiktoken("gpt-4")
tokenizer_gpt5 = LazyloadTiktoken("gpt-4")  # GPT-5 使用 GPT-4 tokenizer
```

## 故障排查

### 错误：`KeyError: 'model-name'`

✓ 解决方案：
1. 检查 `model_info` 中是否包含该模型
2. 检查 `AVAIL_LLM_MODELS` 中是否包含该模型名称
3. 确保拼写一致（区分大小写）
4. 重启容器

### 错误：`NameError: name 'xxx' is not defined`

✓ 解决方案：
1. 检查导入语句是否完整
2. 检查是否有未定义的变量（如 tokenizer）
3. 验证 Python 文件语法：`python3 -m py_compile file.py`

### 容器无法启动

✓ 解决方案：
1. 查看容器日志：`docker compose logs`
2. 检查 Docker 卷挂载：`docker compose ps`
3. 验证配置文件语法
4. 尝试完全重建：`docker compose down && docker compose up -d`

## 文档链接

- 详细修复报告：`FIX_REPORT.md`
- 变更总结：`CHANGES_SUMMARY.md`
- 原 README：`README.md`

## 常用命令

```bash
# 查看容器日志
docker compose logs -f

# 验证模型加载
docker exec gpt_academic-gpt_academic_with_latex-1 python3 -c \
  "from request_llms.bridge_all import model_info; print(len(model_info))"

# 检查特定模型
docker exec gpt_academic-gpt_academic_with_latex-1 python3 -c \
  "from request_llms.bridge_all import model_info; print('gpt-5.1' in model_info)"

# 重启容器
docker compose down && docker compose up -d

# 进入容器
docker exec -it gpt_academic-gpt_academic_with_latex-1 bash
```

---

**最后更新**：2025-11-22  
**版本**：v1.1
