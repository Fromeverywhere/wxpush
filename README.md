
# 🚀 WxPush - 基于WxPush Docker Go (企业微信消息推送服务)修改

- 本项目的wxchat代理非必填项，不填可按原项目使用
- 本项目运行前提：wxchat部署在云服务器，用于MP的通知代理，并已正常使用。
- 本项目部署在本地，调用云服务器的wxchat代理，解决家宽动态ip问题，云服务器无需任何修改，无需部署其他容器。




![Go](https://img.shields.io/badge/Go-1.21-blue.svg)
![Docker](https://img.shields.io/badge/Docker-Lightweight-blue)
![License](https://img.shields.io/badge/License-MIT-green)

这是一个基于 **Go 语言** 重写的高性能企业微信（WeCom）消息推送服务。

相比于传统的 Python/Java 实现，本项目主打 **极致轻量**、**安全** 和 **零依赖**。基于 Alpine 镜像静态编译，Docker 镜像体积仅约 **15MB**，运行时内存占用仅 **5~10MB**。

---

## ✨ 核心特性

- **🚀 极致轻量**：告别 Python/Node.js 的庞大体积，Go 语言原生编译，瞬间启动，几乎不占资源。
- **📩 全类型支持**：采用“透传模式”，完美支持企业微信所有消息类型（文本、Markdown、图文、卡片等）。
- **🛡️ 安全设计**：敏感配置（Secret/Key）通过环境变量注入，不在代码中留痕；支持自定义鉴权密钥防止恶意调用。
- **🔄 自动维护**：内置 AccessToken 自动刷新与过期重试机制，7x24 小时稳定运行。
- **🐳 多样化部署**：支持 Docker、Docker Compose、Koyeb/Render 云平台以及二进制直接运行。

---

## 🛠️ 部署前准备

在部署之前，请确保你已获取企业微信的应用配置：
1. **企业ID (CORP_ID)**: [企业微信后台](https://work.weixin.qq.com/) -> 我的企业 -> 企业信息 -> 最下方。
2. **应用ID (AGENT_ID)**: 应用管理 -> 创建/选择应用 -> AgentId。
3. **应用密钥 (SECRET)**: 应用管理 -> 点击应用 -> Secret (需在手机端查看)。

---

## 📂 推荐部署流程 (Docker Compose)

```yaml
version: '3'
services:
  wxpush:
    image: fromeverywhere/wxpush:latest  #ghcr.io/fromeverywhere/wxpush:latest 亦可
    container_name: wxpush
    restart: always
    ports:
      - "10008:10008"
    environment:
        # --- 企业微信配置 (必填) ---
      - CORP_ID=wwxxxxxxxxxxxxxxxx
      - AGENT_ID=10xxxxxxxxxxxx
      - SECRET=jxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

        # --- 安全配置 (必填) ---
        # 自定义鉴权密钥，调用接口时 URL 必须带上 ?key=这个值
      - PUSH_KEY=3xxxxxxxxxxxxxxxxxxxxxxxxxxx
        # --- 服务配置 ---
        # 容器内部监听端口 (建议保持与外部映射一致)
        # 默认PORT=10001，如不需要自定义可以注释掉此项
        #以下为自定义端口
      - PORT=10008
        # 时区设置 (保证日志时间正确)
      - TZ=Asia/Shanghai
      - WXCHAT_BASE=https://ip or domain:port  #wxchat代理地址，需提前确认可以使用
    volumes:
      - ./wxpush-data:/data   # token 缓存、日志
    network_mode: bridge
```

启动命令：
```bash
docker-compose up -d
```

---

## 🔌 API 使用说明

- **接口地址**: `http://你的服务器IP:10008/send?key=你的PUSH_KEY`
- **请求方式**: `POST`
- **Content-Type**: `application/json`

本服务支持企业微信官方文档中的**所有**消息格式。你只需要构建对应的 JSON 即可。

### 📨 示例 1：发送纯文本 (Text)
支持简写模式，直接传 `content` 即可。

```bash
curl -X POST "[http://127.0.0.1:10008/send?key=你的PUSH_KEY](http://127.0.0.1:10008/send?key=你的PUSH_KEY)" \
     -H "Content-Type: application/json" \
     -d '{
       "msgtype": "text",
       "content": "🔔 服务器 SSH 登录警告！\n来源IP: 192.168.1.100"
     }'
```

### 📝 示例 2：发送 Markdown
Markdown 支持颜色高亮，适合做监控告警。

```bash
curl -X POST "[http://127.0.0.1:10008/send?key=你的PUSH_KEY](http://127.0.0.1:10008/send?key=你的PUSH_KEY)" \
     -H "Content-Type: application/json" \
     -d '{
       "msgtype": "markdown",
       "markdown": {
         "content": "# 每日监控日报\n> CPU: <font color=\"warning\">80%</font>\n> 内存: <font color=\"info\">正常</font>\n> [查看详情](http://example.com)"
       }
     }'
```

### 🃏 示例 3：发送文本卡片 (TextCard)
适合做漂亮的通知跳转。

```bash
curl -X POST "[http://127.0.0.1:10008/send?key=你的PUSH_KEY](http://127.0.0.1:10008/send?key=你的PUSH_KEY)" \
     -H "Content-Type: application/json" \
     -d '{
       "msgtype": "textcard",
       "textcard": {
         "title": "GitHub Actions 构建成功",
         "description": "<div class=\"gray\">2025-12-06</div> <br>项目构建完成，耗时 35s。",
         "url": "[https://github.com/x9xi99](https://github.com/x9xi99)",
         "btntxt": "立即查看"
       }
     }'
```

### 🖼️ 示例 4：其他类型 (图片/文件等)
完全遵循 [企业微信官方 API 文档](https://developer.work.weixin.qq.com/document/path/90236) 的 JSON 结构。例如发送图片：

```json
{
  "msgtype": "image",
  "image": {
    "media_id": "MEDIA_ID_xxxxxx"
  },
  "touser": "@all"
}
```

---

## ⚙️ 环境变量详解

| 变量名 | 必填 | 示例 | 说明 |
| :--- | :--- | :--- | :--- |
| `CORP_ID` | ✅ | `ww49...` | 企业微信 CorpID |
| `AGENT_ID` | ✅ | `1000002` | 企业微信应用 AgentID |
| `SECRET` | ✅ | `e5Nl...` | 企业微信应用 Secret |
| `PUSH_KEY` | ✅ | `你的PUSH_KEY` | **自定义鉴权密钥**，防止接口被他人滥用 |
| `PORT` | ❌ | `10008` | 容器内部监听端口，默认为 10001 |
| `TZ` | ❌ | `Asia/Shanghai` | 容器时区，建议设置以保证日志时间正确 |
| `WXCHAT_BASE` | ❌ | `https://ip or domain:port` | wxchat代理地址，需提前确认可以使用 |
---

---

## 🤝 贡献与构建

感谢原作者分享：

```bash
# 1. 克隆代码
git clone [https://github.com/x9xi99/wxpush-docker-go.git](https://github.com/x9xi99/wxpush-docker-go.git)

# 2. 构建镜像
docker build -t my-wxpush .

# 3. 运行
docker run -d -p 10008:10008 --env-file .env my-wxpush
```

## 📝 License

MIT License.