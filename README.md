# GeDraw

<p align="center">
  <img src="public/images/logo.png" alt="GeDraw Logo" width="120" />
</p>

<p align="center">
  <strong>✨ GeDraw - Gemini + Draw | 智能 AI 对话与创意绘图平台</strong>
</p>

<p align="center">
  <a href="#功能特性">功能特性</a> •
  <a href="#快速开始">快速开始</a> •
  <a href="#docker-部署">Docker 部署</a> •
  <a href="#配置说明">配置说明</a> •
  <a href="#开发指南">开发指南</a>
</p>

---

## 📖 项目简介

GeDraw 是一个功能强大的智能 AI 对话与创意绘图平台，支持多种大语言模型接入、多租户管理、丰富的认证方式，以及创意绘图等扩展功能。

### 核心特性

- 🤖 **多模型支持** - 支持 OpenAI、Claude、Gemini、通义千问等主流 AI 模型
- 🔐 **多种认证方式** - 邮箱登录、飞书、企业微信、钉钉、Linux Do 等 OAuth 登录
- 👥 **多租户管理** - 用户分组、权限控制、用量统计
- 🎨 **创意绘图** - 集成 AI 绘图功能，支持漫画生成等创意应用
- 🔧 **MCP 协议支持** - 支持 Model Context Protocol，扩展 AI 能力
- 🌐 **国际化** - 支持中英文切换
- 📱 **响应式设计** - 完美适配桌面端和移动端

---

## 🚀 快速开始

### 环境要求

- Node.js 18+
- PostgreSQL 14+
- npm 或 yarn

### 本地开发

```bash
# 克隆项目
git clone https://github.com/your-username/gedraw.git
cd gedraw

# 安装依赖
npm install

# 配置环境变量
cp .env.example .env
# 编辑 .env 文件，填写必要的配置

# 初始化数据库
npm run initdb

# 启动开发服务器
npm run dev
```

访问 http://localhost:3000 开始使用。

---

## 🐳 Docker 部署

### 方式一：使用 Docker Compose（推荐）

这是最简单的部署方式，包含应用和数据库的完整环境。

#### 1. 准备配置文件

```bash
# 克隆项目
git clone https://github.com/hebedich/gedraw.git
cd gedraw

# 创建环境变量文件
cp .env.example .env
```

#### 2. 编辑 `.env` 文件

```bash
# 必填项
AUTH_SECRET=your-secret-key-here  # 使用 openssl rand -base64 32 生成
ADMIN_CODE=your-admin-code        # 管理员授权码
NEXTAUTH_URL=https://your-domain.com  # 生产环境域名

# 认证方式（按需开启）
EMAIL_AUTH_STATUS=ON

# 飞书登录（可选）
FEISHU_AUTH_STATUS=OFF
FEISHU_CLIENT_ID=
FEISHU_CLIENT_SECRET=

# 企业微信登录（可选）
WECOM_AUTH_STATUS=OFF
WECOM_CLIENT_ID=
WECOM_AGENT_ID=
WECOM_CLIENT_SECRET=

# 钉钉登录（可选）
DINGDING_AUTH_STATUS=OFF
DINGDING_CLIENT_ID=
DINGDING_CLIENT_SECRET=
```

#### 3. 启动服务

```bash
# 构建并启动
docker-compose up -d

# 查看日志
docker-compose logs -f
```

#### 4. 初始化管理员

访问 `http://your-domain:3000/setup`，使用 `ADMIN_CODE` 设置管理员账号。

### 方式二：仅部署应用（使用外部数据库）

如果已有 PostgreSQL 数据库，可以只部署应用容器。

```bash
docker run -d \
  --name gedraw \
  -p 3000:3000 \
  -e DATABASE_URL="postgres://user:password@host:5432/dbname" \
  -e AUTH_SECRET="your-secret-key" \
  -e ADMIN_CODE="your-admin-code" \
  -e NEXTAUTH_URL="https://your-domain.com" \
  -e EMAIL_AUTH_STATUS="ON" \
  gedraw:latest
```

### Docker Compose 配置说明

```yaml
services:
  app:
    container_name: gedraw
    build: .
    ports:
      - "50002:3000"  # 主机端口:容器端口
    environment:
      DATABASE_URL: "postgres://postgres:postgres@db:5432/gedraw"
      # ... 其他环境变量
    volumes:
      - ./public:/app/public  # 持久化上传文件

  db:
    image: postgres:16.8-alpine
    ports:
      - "50433:5432"
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: gedraw
    volumes:
      - ./postgres_data:/var/lib/postgresql/data  # 持久化数据库
```

---

## ⚙️ 配置说明

### 环境变量

| 变量名 | 必填 | 说明 | 示例 |
|--------|------|------|------|
| `DATABASE_URL` | ✅ | PostgreSQL 连接 URL | `postgres://user:pass@host:5432/db` |
| `AUTH_SECRET` | ✅ | 加密密钥（32位） | 使用 `openssl rand -base64 32` 生成 |
| `ADMIN_CODE` | ✅ | 管理员授权码 | `11223344` |
| `NEXTAUTH_URL` | ✅ | 应用访问地址 | `https://chat.example.com` |
| `EMAIL_AUTH_STATUS` | ❌ | 邮箱登录开关 | `ON` / `OFF` |
| `FEISHU_AUTH_STATUS` | ❌ | 飞书登录开关 | `ON` / `OFF` |
| `WECOM_AUTH_STATUS` | ❌ | 企业微信登录开关 | `ON` / `OFF` |
| `DINGDING_AUTH_STATUS` | ❌ | 钉钉登录开关 | `ON` / `OFF` |

### 反向代理配置（Nginx）

```nginx
server {
    listen 80;
    server_name chat.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name chat.example.com;

    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    location / {
        proxy_pass http://127.0.0.1:50002;
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

---

## 🔧 管理功能

### 后台管理

访问 `/admin` 进入管理后台，功能包括：

- **用户管理** - 用户列表、权限设置、分组管理
- **模型管理** - 配置 AI 模型提供商和模型参数
- **Bot 管理** - 创建和管理预设机器人
- **系统设置** - 站点配置、认证方式设置
- **公告管理** - 发布系统公告

### 初始化流程

1. 部署完成后访问 `/setup`
2. 输入 `ADMIN_CODE` 验证身份
3. 设置管理员邮箱和密码
4. 进入后台配置 AI 模型
5. 开始使用

---

## 🛠️ 开发指南

### 技术栈

- **前端**: Next.js 14, React 18, Ant Design, TailwindCSS
- **后端**: Next.js API Routes, NextAuth.js
- **数据库**: PostgreSQL, Drizzle ORM
- **AI**: OpenAI SDK, Anthropic SDK, Google AI SDK

### 项目结构

```
gedraw/
├── app/
│   ├── (auth)/          # 认证相关页面
│   ├── admin/           # 管理后台
│   ├── api/             # API 路由
│   ├── chat/            # 聊天功能
│   ├── components/      # 通用组件
│   ├── db/              # 数据库模型
│   ├── drawing/         # 绘图功能
│   └── lib/             # 工具库
├── docker/              # Docker 初始化脚本
├── drizzle/             # 数据库迁移
├── locales/             # 国际化文件
├── public/              # 静态资源
└── scripts/             # 工具脚本

---

## 📝 更新日志

### v0.1.0

- 初始版本发布
- 支持多种 AI 模型
- 支持多种 OAuth 登录
- 管理后台功能完善

---

## 🙏 致谢

感谢所有贡献者和使用者的支持！

---

<p align="center">
  Made with ❤️ by GeDraw Team
</p>

