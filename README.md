# TabOS 云桌面系统

> 一个现代化的云端桌面操作系统，支持多网盘挂载、实时消息、AI助手等功能。

## ✨ 功能特点

- 🖥️ **完整的桌面体验** - macOS 风格的窗口系统、Dock 栏、控制中心
- 📁 **多网盘支持** - 支持 25+ 种存储驱动（阿里云盘、百度网盘、夸克等）
- 💬 **实时消息系统** - IM 私聊、群聊、WebSocket 实时推送
- 🤖 **AI 助手** - 内置 AI 对话功能
- 🎵 **影音娱乐** - 音乐播放器、影视中心
- 📝 **备忘录** - 支持加密、媒体、分享
- 📧 **邮件客户端** - IMAP/SMTP 支持
- 📅 **日历日程** - 完整的日历管理
- 🌐 **多语言支持** - 中文简体/繁体、英语、日语、韩语

## 📦 部署方式

### 1. Docker 部署（推荐）

```bash
# 克隆仓库
git clone https://github.com/vivafriend/tabos-deploy.git
cd tabos-deploy

# 下载最新版本的二进制文件
wget https://github.com/vivafriend/tabos-deploy/releases/latest/download/tabos-backend-linux-amd64
wget https://github.com/vivafriend/tabos-deploy/releases/latest/download/tabos-frontend-dist.tar.gz

# 使用 Docker Compose 启动
docker-compose up -d
```

详细教程：[Docker 部署完整指南](docs/Docker部署完整指南.md)

### 2. Linux 服务器部署

详细教程：[Linux 部署完整指南](docs/Linux部署完整指南.md)

### 3. 宝塔面板部署

详细教程：[宝塔面板部署完整指南](docs/宝塔面板部署完整指南.md)

## 📥 下载

从 [Releases](https://github.com/vivafriend/tabos-deploy/releases) 页面下载最新版本：

- `tabos-backend-linux-amd64` - Linux x64 后端程序
- `tabos-backend-linux-arm64` - Linux ARM64 后端程序
- `tabos-frontend-dist.tar.gz` - 前端静态文件

## 📖 快速开始

### 系统要求

- MySQL 8.0+ 或 MariaDB 10.5+
- Redis 7.0+（推荐）
- Nginx（用于反向代理）

### 最小配置

- CPU：1 核
- 内存：1GB
- 磁盘：10GB

### 推荐配置

- CPU：2 核以上
- 内存：4GB 以上
- 磁盘：50GB 以上

## 🔧 配置说明

### 环境变量

```bash
# WebSocket Origin 白名单
ALLOWED_ORIGINS=https://your-domain.com

# 数据库配置（通过安装向导配置）
```

### 配置文件

系统首次启动时会进入安装向导，配置：
- 数据库连接
- Redis 缓存
- 站点信息
- 管理员账号

## 📚 文档

- [Linux 部署完整指南](docs/Linux部署完整指南.md)
- [宝塔面板部署完整指南](docs/宝塔面板部署完整指南.md)
- [Docker 部署完整指南](docs/Docker部署完整指南.md)
- [常见问题 FAQ](docs/FAQ.md)

## 🤝 技术支持

如遇到问题：
1. 查看[常见问题 FAQ](docs/FAQ.md)
2. 提交 [Issue](https://github.com/vivafriend/tabos-deploy/issues)

## 📄 许可证

版权所有 © 2026 TabOS

## 🌟 Star History

如果这个项目对你有帮助，请给个 Star ⭐
