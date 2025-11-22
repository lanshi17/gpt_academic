# Linxi 和 SiliconFlow 快速配置指南

## 🚀 5分钟快速开始

### 第1步：获取 API 密钥

#### Linxi API 密钥
1. 访问 https://linxi.chat/
2. 注册或登录
3. 在控制面板获取 API Key
4. 复制 API Key

#### SiliconFlow API 密钥  
1. 访问 https://cloud.siliconflow.cn/
2. 注册或登录
3. 申请 API Key
4. 复制 API Key

---

### 第2步：配置方式选择

#### 🐳 Docker 方式（推荐）

**编辑 `docker-compose.yml`，找到 environment 部分：**

```yaml
environment:
  # 添加或修改以下行
  LINXI_API_KEY: "你的Linxi-API-Key"  # 替换为你的密钥
  SILICONFLOW_API_KEY: "你的SiliconFlow-API-Key"  # 替换为你的密钥
```

**然后启动容器：**

```bash
# 先关闭旧容器（如果存在）
docker-compose down

# 启动新容器
docker-compose up -d

# 查看日志
docker-compose logs -f
```

**访问 Web 界面：**
打开浏览器，访问 `http://localhost:12303`

---

#### 💻 本地开发方式

**编辑 `config_private.py`：**

```python
# 找到这两行（大约在第 18-19 行）
LINXI_API_KEY = "你的Linxi-API-Key"          # 替换为你的 Linxi 密钥
SILICONFLOW_API_KEY = "你的SiliconFlow-API-Key"  # 替换为你的 SiliconFlow 密钥

# 确保这两个模型在 AVAIL_LLM_MODELS 中
AVAIL_LLM_MODELS = [
    # Linxi 模型
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
    
    # SiliconFlow 模型
    "siliconflow-deepseek-v3",
    "siliconflow-qwen3-max",
    "siliconflow-qwen-plus",
    "siliconflow-glm-4-plus",
    "siliconflow-hunyuan-turbo",
    "siliconflow-yi-1.5-34b",
    "siliconflow-mistral-large",
    
    # ... 其他现有模型 ...
]
```

**然后启动应用：**

```bash
python3 main.py
```

---

### 第3步：验证安装

在 Web 界面的模型下拉菜单中，应该能看到以下新模型：

**Linxi 模型：**
- linxi-gpt-4-turbo
- linxi-gpt-4
- linxi-gpt-4o
- linxi-gpt-4o-mini
- linxi-claude-3.5-sonnet
- linxi-claude-opus-4
- linxi-gemini-2-flash
- linxi-deepseek-v3
- linxi-qwen-max
- linxi-glm-4-plus

**SiliconFlow 模型：**
- siliconflow-deepseek-v3 ⭐ 支持推理
- siliconflow-qwen3-max
- siliconflow-qwen-plus
- siliconflow-glm-4-plus
- siliconflow-hunyuan-turbo
- siliconflow-yi-1.5-34b
- siliconflow-mistral-large

---

## ⚡ 常用命令

```bash
# Docker 方式

# 启动容器（后台）
docker-compose up -d

# 查看实时日志
docker-compose logs -f

# 重启容器
docker-compose restart

# 停止容器
docker-compose down

# 查看容器状态
docker-compose ps
```

---

## 🔍 验证连接

### 检查 Docker 容器日志

```bash
docker-compose logs | grep -i "linxi\|siliconflow\|error"
```

### 测试 API 连接（需要 curl）

**Linxi 测试：**
```bash
curl -H "Authorization: Bearer YOUR-LINXI-KEY" \
     https://linxi.chat/v1/models
```

**SiliconFlow 测试：**
```bash
curl -H "Authorization: Bearer YOUR-SILICONFLOW-KEY" \
     https://api.siliconflow.cn/v1/models
```

---

## ❌ 常见问题

### Q: 在模型下拉菜单中看不到新模型

**A:** 
1. 检查 `config_private.py` 或 `docker-compose.yml` 中是否正确配置了模型名称
2. 检查拼写是否完全匹配（包括 `linxi-` 或 `siliconflow-` 前缀）
3. 重启应用或容器
4. 清除浏览器缓存并刷新页面

### Q: 出现 "API key is invalid" 错误

**A:**
1. 确认 API Key 已正确复制（没有多余空格）
2. 登录 Linxi/SiliconFlow 官网确认 API Key 仍有效
3. 检查账户余额是否充足
4. 尝试重新生成新的 API Key

### Q: Docker 容器无法启动

**A:**
1. 查看日志：`docker-compose logs`
2. 检查端口 12303 是否被占用：`lsof -i :12303`
3. 检查 `docker-compose.yml` YAML 格式是否正确
4. 尝试完全重启：`docker-compose down && docker-compose up -d`

### Q: 连接超时或无法到达 API

**A:**
1. 检查网络连接
2. 确认 API 端点是否在线（访问官网检查状态）
3. 如使用代理，检查代理配置
4. 尝试从不同网络或设备测试

---

## 📚 详细文档

详细的集成文档请参考：**[INTEGRATION_GUIDE.md](./INTEGRATION_GUIDE.md)**

包含内容：
- 完整的 API 介绍
- 支持模型详细表
- 推理模型配置
- 故障排除指南
- 技术实现细节

---

## 🎯 使用示例

### 在 Web 界面中使用

1. 打开 Web 界面
2. 在"模型"下拉菜单选择想要的模型
3. 在文本框输入提示词
4. 点击"提交"

### 在 Python 代码中使用

```python
from request_llms.bridge_all import predict_no_ui_long_connection

# 使用 Linxi GPT-4
response = predict_no_ui_long_connection(
    inputs="解释什么是量子计算",
    llm_kwargs={
        "llm_model": "linxi-gpt-4-turbo",
        "temperature": 0.5
    }
)
print(response)

# 使用 SiliconFlow 推理模型
response = predict_no_ui_long_connection(
    inputs="用深度思考解决这个数学问题",
    llm_kwargs={
        "llm_model": "siliconflow-deepseek-v3",
        "temperature": 0.3
    }
)
print(response)
```

---

## ✅ 配置检查清单

- [ ] 获取了 Linxi API Key
- [ ] 获取了 SiliconFlow API Key
- [ ] 在 `config_private.py` 或 `docker-compose.yml` 中配置了密钥
- [ ] 模型名称添加到 `AVAIL_LLM_MODELS`
- [ ] 启动了应用或 Docker 容器
- [ ] 在 Web 界面中看到了新模型
- [ ] 成功选择并使用了新模型

---

## 🆘 获取帮助

如遇到问题：

1. 查看 **[INTEGRATION_GUIDE.md](./INTEGRATION_GUIDE.md)** 的故障排除部分
2. 检查容器日志：`docker-compose logs`
3. 访问 API 官网检查服务状态：
   - Linxi: https://linxi.chat/
   - SiliconFlow: https://cloud.siliconflow.cn/

---

**集成完成！现在可以使用 Linxi 和 SiliconFlow 的所有模型了！** 🎉
