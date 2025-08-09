# 🚀 LMArena Bridge - 新一代 OpenAI 桥接器 🌉

欢迎来到新一代的 LMArena Bridge！🎉 这是一个基于 FastAPI 和 WebSocket 的高性能工具集，它能让你通过任何兼容 OpenAI API 的客户端或应用程序，无缝使用 [LMArena.ai](https://lmarena.ai/) 平台上提供的海量大语言模型。

这个重构版本旨在提供更稳定、更易于维护和扩展的体验。

## ✨ 主要功能

*   **🚀 高性能后端**: 基于 **FastAPI** 和 **Uvicorn**，提供异步、高性能的 API 服务。
*   **🔌 稳定的 WebSocket 通信**: 使用 WebSocket 替代 Server-Sent Events (SSE)，实现更可靠、低延迟的双向通信。
*   **🤖 OpenAI 兼容接口**: 完全兼容 OpenAI `v1/chat/completions`、`v1/models` 以及 `v1/images/generations` 端点。
*   **📎 通用文件上传**: 支持通过 Base64 上传任意类型的文件（图片、音频、PDF、代码等），并支持一次性上传多个文件。
*   **🎨 文生图支持**: 新增文生图功能，可通过标准 OpenAI 接口调用 LMArena 的图像生成模型。
*   **🗣️ 完整对话历史支持**: 自动将会话历史注入到 LMArena，实现有上下文的连续对话。
*   **🌊 实时流式响应**: 像原生 OpenAI API 一样，实时接收来自模型的文本回应。
*   **🔄 自动模型与程序更新**:
    *   启动时自动从 LMArena 页面获取最新的模型列表，并智能更新 `models.json`。
    *   启动时自动检查 GitHub 仓库，发现新版本时可自动下载并更新程序。
*   **🆔 一键式会话ID更新**: 提供 `id_updater.py` 脚本，只需在浏览器操作一次，即可自动捕获并更新 `config.jsonc` 中所需的会话 ID。
*   **⚙️ 浏览器自动化**: 配套的油猴脚本 (`LMArenaApiBridge.js`) 负责与后端服务器通信，并在浏览器中执行所有必要操作。
*   **🍻 酒馆模式 (Tavern Mode)**: 专为 SillyTavern 等应用设计，智能合并 `system` 提示词，确保兼容性。
*   **🤫 Bypass 模式**: 尝试通过在请求中额外注入一个空的用户消息，绕过平台的敏感词审查。
*   **🔐 API Key 保护**: 可在配置文件中设置 API Key，为你的服务增加一层安全保障。
*   **🎯 模型-会话高级映射**: 支持为不同模型配置独立的会话ID池，并能为每个会话指定特定的工作模式（如 `battle` 或 `direct_chat`），实现更精细的请求控制。

## ⚙️ 配置文件说明

项目的主要行为通过 `config.jsonc` 和 `model_endpoint_map.json` 进行控制。

### `config.jsonc` - 全局配置

这是主要的配置文件，包含了服务器的全局设置。

*   `session_id` / `message_id`: 全局默认的会话ID。当模型没有在 `model_endpoint_map.json` 中找到特定映射时，会使用这里的ID。
*   `id_updater_last_mode` / `id_updater_battle_target`: 全局默认的请求模式。同样，当特定会话没有指定模式时，会使用这里的设置。
*   `use_default_ids_if_mapping_not_found`: 一个非常重要的开关（默认为 `true`）。
    *   `true`: 如果请求的模型在 `model_endpoint_map.json` 中找不到，就使用全局默认的ID和模式。
    *   `false`: 如果找不到映射，则直接返回错误。这在你需要严格控制每个模型的会话时非常有用。
*   其他配置项如 `api_key`, `tavern_mode_enabled` 等，请参考文件内的注释。

### `model_endpoint_map.json` - 模型专属配置

这是一个强大的高级功能，允许你覆盖全局配置，为特定的模型设置一个或多个专属的会话。

**核心优势**:
1.  **会话隔离**: 为不同的模型使用独立的会话，避免上下文串扰。
2.  **提高并发**: 为热门模型配置一个ID池，程序会在每次请求时随机选择一个ID使用，模拟轮询，减少单个会话被频繁请求的风险。
3.  **模式绑定**: 将一个会话ID与它被捕获时的模式（`direct_chat` 或 `battle`）绑定，确保请求格式永远正确。

**配置示例**:
```json
{
  "claude-3-opus-20240229": [
    {
      "session_id": "session_for_direct_chat_1",
      "message_id": "message_for_direct_chat_1",
      "mode": "direct_chat"
    },
    {
      "session_id": "session_for_battle_A",
      "message_id": "message_for_battle_A",
      "mode": "battle",
      "battle_target": "A"
    }
  ],
  "gemini-1.5-pro-20241022": {
      "session_id": "single_session_id_no_mode",
      "message_id": "single_message_id_no_mode"
  }
}
```
*   **Opus**: 配置了一个ID池。请求时会随机选择其中一个，并严格按照其绑定的 `mode` 和 `battle_target` 来发送请求。
*   **Gemini**: 使用了单个ID对象（旧格式，依然兼容）。由于它没有指定 `mode`，程序会自动使用 `config.jsonc` 中定义的全局模式。

## 🛠️ 安装与使用

你需要准备好 Python 环境和一款支持油猴脚本的浏览器 (如 Chrome, Firefox, Edge)。

### 1. 准备工作

*   **安装 Python 依赖**
    打开终端，进入项目根目录，运行以下命令：
    ```bash
    pip install -r requirements.txt
    ```

*   **安装油猴脚本管理器**
    为你的浏览器安装 [Tampermonkey](https://www.tampermonkey.net/) 扩展。

*   **安装本项目油猴脚本**
    1.  打开 Tampermonkey 扩展的管理面板。
    2.  点击“添加新脚本”或“Create a new script”。
    3.  将 [`TampermonkeyScript/LMArenaApiBridge.js`](TampermonkeyScript/LMArenaApiBridge.js) 文件中的所有代码复制并粘贴到编辑器中。
    4.  保存脚本。

### 2. 运行主程序

1.  **启动本地服务器**
    在项目根目录下，运行主服务程序：
    ```bash
    python api_server.py
    ```
    当你看到服务器在 `http://127.0.0.1:5102` 启动的提示时，表示服务器已准备就绪。

2.  **保持 LMArena 页面开启**
    确保你至少有一个 LMArena 页面是打开的，并且油猴脚本已成功连接到本地服务器（页面标题会以 `✅` 开头）。这里无需保持在对话页面，只要是域名下的页面都可以LeaderBoard都可以。

### 3. 配置会话 ID (需要时，一般只配置一次即可，除非切换模型或者原对话失效)

这是**最重要**的一步。你需要获取一个有效的会话 ID 和消息 ID，以便程序能够正确地与 LMArena API 通信。

1.  **确保主服务器正在运行**
    `api_server.py` 必须处于运行状态，因为 ID 更新器需要通过它来激活浏览器的捕获功能。

2.  **运行 ID 更新器**
    打开**一个新的终端**，在项目根目录下运行 `id_updater.py` 脚本：
    ```bash
    python id_updater.py
    ```
    *   脚本会提示你选择模式 (DirectChat / Battle)。
    *   选择后，它会通知正在运行的主服务器。

3.  **激活与捕获**
    *   此时，你应该会看到浏览器中 LMArena 页面的标题栏最前面出现了一个准星图标 (🎯)，这表示**ID捕获模式已激活**。
    *   在浏览器中打开一个 LMArena 竞技场的 **目标模型发送给消息的页面**。请注意，如果是Battle页面，请不要查看模型名称，保持匿名状态，并保证当前消息界面的最后一条是目标模型的一个回答；如果是Direct Chat，请保证当前消息界面的最后一条是目标模型的一个回答。
    *   **点击目标模型的回答卡片右上角的重试（Retry）按钮**。
    *   油猴脚本会捕获到 `sessionId` 和 `messageId`，并将其发送给 `id_updater.py`。

4.  **验证结果**
    *   回到你运行 `id_updater.py` 的终端，你会看到它打印出成功捕获到的 ID，并提示已将其写入 `config.jsonc` 文件。
    *   脚本在成功后会自动关闭。现在你的配置已完成！

3.  **配置你的 OpenAI 客户端**
    将你的客户端或应用的 OpenAI API 地址指向本地服务器：
    *   **API Base URL**: `http://127.0.0.1:5102/v1`
    *   **API Key**: 如果 `config.jsonc` 中的 `api_key` 为空，则可随便输入；如果已设置，则必须提供正确的 Key。
    *   **Model Name**: 模型现在不再由本地决定，而是由你再LMArena对话中，重试（Retry）的那条消息所决定。所以务必提前在对话或者Battle中找到你想要的模型的对话。DirectChat可以使用DC大佬提供的**LMArenaDirectChat模型注入 (仙之人兮列如麻) V5.js**这个油猴脚本来确定你想用的模型（wolfstride等最新测试模型可能无法直接对话，必须在Battle里进行捕捉）。

# LMArenaBridge Docker 部署指南

## 🚀 快速开始

### 1. 准备工作

确保您的系统已安装：
- Docker (>= 20.10)
- Docker Compose (>= 2.0)

### 2. 项目结构

```
LMArenaBridge/
├── Dockerfile
├── docker-compose.yml
├── .dockerignore
├── requirements.txt
├── api_server.py
├── id_updater.py
├── data/
│   ├── config.jsonc
│   ├── models.json
│   └── model_endpoint_map.json
├── modules/
│   ├── image_generation.py
│   └── update_script.py
└── TampermonkeyScript/
    └── LMArenaApiBridge.js
```

### 3. 部署步骤

#### 方式一：使用 docker-compose（推荐）

1. **克隆项目并进入目录**
```bash
git clone https://github.com/Lianues/LMArenaBridge.git
cd LMArenaBridge
```

2. **创建数据目录和配置文件**
```bash
mkdir -p data
cp config.jsonc data/config.jsonc  # 如果原项目有的话
# 或者创建新的配置文件（参考config.jsonc模板）
```

3. **启动服务**
```bash
docker-compose up -d
```

4. **查看日志**
```bash
docker-compose logs -f lmarenabridge
```

#### 方式二：直接使用 Docker

1. **构建镜像**
```bash
docker build -t lmarenabridge .
```

2. **运行容器**
```bash
docker run -d \
  --name lmarenabridge \
  -p 5102:5102 \
  -v $(pwd)/data:/app/data \
  lmarenabridge
```

### 4. 配置说明

#### 环境变量配置

在 docker-compose.yml 中可以设置以下环境变量：

```yaml
environment:
  - PORT=5102                    # 服务端口
  - PYTHONUNBUFFERED=1          # Python输出缓冲
  - LOG_LEVEL=INFO              # 日志级别
  - API_KEY=your_api_key_here   # API密钥（可选）
```

#### 数据持久化

通过数据卷挂载实现配置持久化：

```yaml
volumes:
  - ./data/config.jsonc:/app/data/config.jsonc
  - ./data/models.json:/app/data/models.json
  - ./data/model_endpoint_map.json:/app/data/model_endpoint_map.json
```

### 5. 健康检查

容器包含健康检查功能，会定期检查服务状态：

```bash
# 检查容器健康状态
docker ps
# 或
docker-compose ps
```

### 6. 使用方式

#### 安装Tampermonkey脚本

1. 在浏览器中安装 Tampermonkey 扩展
2. 将 `TampermonkeyScript/LMArenaApiBridge.js` 中的脚本安装到 Tampermonkey
3. 确保脚本中的服务器地址指向您的Docker容器：`http://localhost:5102`

#### 获取会话ID

1. **启动ID更新器**
```bash
# 进入容器
docker exec -it lmarenabridge bash
# 运行ID更新脚本
python id_updater.py
```

2. **在浏览器中操作**
   - 打开 LMArena.ai 网站
   - 确保页面标题显示 ✅ 或 🎯 图标
   - 进入目标模型对话页面
   - 点击重试(Retry)按钮获取会话ID

#### 配置OpenAI客户端

- **API Base URL**: `http://localhost:5102/v1`
- **API Key**: 如果config.jsonc中设置了api_key，则需要提供；否则可以随意填写
- **Model Name**: 使用从LMArena捕获的模型名称

## 🔧 高级配置

### 自定义端口

如果需要更改默认端口（5102），修改以下配置：

1. **docker-compose.yml**
```yaml
ports:
  - "8080:8080"  # 宿主机端口:容器端口
environment:
  - PORT=8080
```

2. **config.jsonc**
```json
{
  "port": 8080
}
```

### 反向代理配置

使用Nginx作为反向代理的示例配置：

```nginx
server {
    listen 80;
    server_name your-domain.com;
    
    location / {
        proxy_pass http://localhost:5102;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 86400;
    }
}
```

### 多实例部署

如需部署多个实例，可以使用不同的端口：

```yaml
version: '3.8'
services:
  lmarenabridge-1:
    build: .
    ports:
      - "5102:5102"
    # ... 其他配置
    
  lmarenabridge-2:
    build: .
    ports:
      - "5103:5102"
    # ... 其他配置
```

## 📋 故障排除

### 常见问题

1. **容器启动失败**
```bash
# 查看详细日志
docker-compose logs lmarenabridge
```

2. **无法连接到WebSocket**
   - 确保防火墙允许5102端口
   - 检查Tampermonkey脚本中的服务器地址
   - 验证容器网络配置

3. **会话ID获取失败**
   - 确保浏览器中的LMArena页面正常加载
   - 检查Tampermonkey脚本是否正确安装和启用
   - 确认容器内的id_updater.py能正常运行

### 日志查看

```bash
# 实时查看日志
docker-compose logs -f lmarenabridge

# 查看最近100行日志
docker-compose logs --tail=100 lmarenabridge

# 进入容器查看内部日志
docker exec -it lmarenabridge tail -f /app/data/app.log
```

### 性能调优

1. **内存限制**
```yaml
services:
  lmarenabridge:
    deploy:
      resources:
        limits:
          memory: 1G
        reservations:
          memory: 512M
```

2. **CPU限制**
```yaml
services:
  lmarenabridge:
    deploy:
      resources:
        limits:
          cpus: '1.0'
```

## 🛡️ 安全注意事项

1. **API密钥保护**：在生产环境中务必设置强API密钥
2. **网络隔离**：考虑使用Docker网络隔离服务
3. **HTTPS**：在生产环境中使用HTTPS和SSL证书
4. **访问控制**：限制容器的网络访问权限

## 📈 监控和维护

### 定期备份

```bash
# 备份配置文件
docker cp lmarenabridge:/app/data ./backup/

# 或使用数据卷备份
tar -czf backup-$(date +%Y%m%d).tar.gz ./data/
```

### 更新镜像

```bash
# 重新构建镜像
docker-compose build --no-cache

# 重启服务
docker-compose down && docker-compose up -d
```

### 清理资源

```bash
# 清理未使用的镜像和容器
docker system prune -a

# 清理数据卷
docker volume prune
```

通过以上Docker部署方案，您可以更方便地部署和管理LMArenaBridge服务，同时获得更好的隔离性和可移植性。

4.  **开始聊天！** 💬
    现在你可以正常使用你的客户端了，所有的请求都会通过本地服务器代理到 LMArena 上！

## 🤔 它是如何工作的？

这个项目由两部分组成：一个本地 Python **FastAPI** 服务器和一个在浏览器中运行的**油猴脚本**。它们通过 **WebSocket** 协同工作。

```mermaid
sequenceDiagram
    participant C as OpenAI 客户端 💻
    participant S as 本地 FastAPI 服务器 🐍
    participant U as ID 更新脚本 (id_updater.py) 🆔
    participant T as 油猴脚本 🐵 (在 LMArena 页面)
    participant L as LMArena.ai 🌐

    alt 初始化与ID更新
        T->>+S: (页面加载) 建立 WebSocket 连接
        S-->>-T: 确认连接
        U->>+S: (用户运行) POST /internal/start_id_capture
        S->>T: (WebSocket) 发送 'activate_id_capture' 指令
        Note right of T: 捕获模式已激活 (一次性)
        T->>L: (用户点击Retry) 拦截到 fetch 请求
        T->>U: (HTTP) 发送捕获到的ID
        U-->>T: 确认
        U->>U: 更新 config.jsonc
    end

    alt 正常聊天流程
        C->>+S: (用户聊天) /v1/chat/completions 请求
        S->>S: 转换请求为 LMArena 格式
        S->>T: (WebSocket) 发送包含 request_id 和载荷的消息
        T->>L: (fetch) 发送真实请求到 LMArena API
        L-->>T: (流式)返回模型响应
        T->>S: (WebSocket) 将响应数据块一块块发回
        S-->>-C: (流式) 返回 OpenAI 格式的响应
    end

    alt 文生图流程
        C->>+S: (用户请求) /v1/images/generations 请求
        S->>S: (并行) 创建 n 个任务
        S->>T: (WebSocket) 发送 n 个包含 request_id 的任务
        T->>L: (fetch) 发送 n 个真实请求
        L-->>T: (流式) 返回图片 URL
        T->>S: (WebSocket) 将 URL 发回
        S-->>-C: (HTTP) 返回包含所有 URL 的 JSON 响应
    end
```

1.  **建立连接**: 当你在浏览器中打开 LMArena 页面时，**油猴脚本**会立即与**本地 FastAPI 服务器**建立一个持久的 **WebSocket 连接**。
    > **注意**: 当前架构假定只有一个浏览器标签页在工作。如果打开多个页面，只有最后一个连接会生效。
2.  **接收请求**: **OpenAI 客户端**向本地服务器发送标准的聊天请求。
3.  **任务分发**: 服务器接收到请求后，会将其转换为 LMArena 需要的格式，并附上一个唯一的请求 ID (`request_id`)，然后通过 WebSocket 将这个任务发送给已连接的油猴脚本。
4.  **执行与响应**: 油猴脚本收到任务后，会直接向 LMArena 的 API 端点发起 `fetch` 请求。当 LMArena 返回流式响应时，油猴脚本会捕获这些数据块，并将它们一块块地通过 WebSocket 发回给本地服务器。
5.  **响应中继**: 服务器根据每块数据附带的 `request_id`，将其放入正确的响应队列中，并实时地将这些数据流式传输回 OpenAI 客户端。

## 📖 API 端点

### 获取模型列表

*   **端点**: `GET /v1/models`
*   **描述**: 返回一个与 OpenAI 兼容的模型列表，该列表从 `models.json` 文件中读取。

### 聊天补全

*   **端点**: `POST /v1/chat/completions`
*   **描述**: 接收标准的 OpenAI 聊天请求，支持流式和非流式响应。

### 图像生成

*   **端点**: `POST /v1/images/generations`
*   **描述**: 接收标准的 OpenAI 文生图请求，返回生成的图片 URL。
*   **请求示例**:
    ```bash
    curl http://127.0.0.1:5102/v1/images/generations \
      -H "Content-Type: application/json" \
      -d '{
        "prompt": "A futuristic cityscape at sunset, neon lights, flying cars",
        "n": 2,
        "model": "dall-e-3"
      }'
    ```
*   **响应示例**:
    ```json
    {
      "created": 1677663338,
      "data": [
        {
          "url": "https://..."
        },
        {
          "url": "https://..."
        }
      ]
    }
    ```

## 📂 文件结构

```
.
├── .gitignore                  # Git 忽略文件
├── api_server.py               # 核心后端服务 (FastAPI) 🐍
├── id_updater.py               # 一键式会话ID更新脚本 🆔
├── models.json                 # 模型名称到 LMArena 内部 ID 的映射表 🗺️
├── model_endpoint_map.json     # [高级] 模型到专属会话ID的映射表 🎯
├── requirements.txt            # Python 依赖包列表 📦
├── README.md                   # 就是你现在正在看的这个文件 👋
├── config.jsonc                # 全局功能配置文件 ⚙️
├── modules/
│   ├── image_generation.py     # 文生图模块 🎨
│   └── update_script.py        # 自动更新逻辑脚本 🔄
└── TampermonkeyScript/
    └── LMArenaApiBridge.js     # 前端自动化油猴脚本 🐵
```

**享受在 LMArena 的模型世界中自由探索的乐趣吧！** 💖
