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

### 1. 直接部署（推荐新手）

```bash
# 1. 下载完整包
wget https://github.com/vivafriend/tabos-deploy/releases/latest/download/tabos-full-linux-x64-v1.0.6.zip

# 2. 解压
unzip tabos-full-linux-x64-v1.0.6.zip -d /www/tabos

# 3. 启动后端
cd /www/tabos/backend
chmod +x tabos-backend
./tabos-backend

# 4. 配置 Nginx 反向代理（参考文档）
```

详细教程：[Linux 部署完整指南](docs/Linux部署完整指南.md)

### 2. 宝塔面板部署

下载完整包后，通过宝塔面板上传并配置。

详细教程：[宝塔面板部署完整指南](docs/宝塔面板部署完整指南.md)

### 3. Docker 部署（开发中）

Docker 部署文档即将推出。

## 📥 下载

从 [Releases](https://github.com/vivafriend/tabos-deploy/releases) 页面下载最新版本的完整包：

### Windows 系统
- `tabos-full-windows-x64-v*.*.*.zip` - Windows 64位
- `tabos-full-windows-32-v*.*.*.zip` - Windows 32位
- `tabos-full-windows-arm64-v*.*.*.zip` - Windows ARM64

### Linux 系统
- `tabos-full-linux-x64-v*.*.*.zip` - Linux x64
- `tabos-full-linux-arm64-v*.*.*.zip` - Linux ARM64
- `tabos-full-linux-armv7-v*.*.*.zip` - Linux ARMv7

### macOS 系统
- `tabos-full-macos-apple-v*.*.*.zip` - macOS Apple Silicon (M1/M2/M3)
- `tabos-full-macos-intel-v*.*.*.zip` - macOS Intel

每个完整包包含：
- ✅ 后端程序（tabos-backend）
- ✅ 前端静态文件（web/）
- ✅ 管理后台（web/admin/）
- ✅ 配置文件示例
- ✅ 必需的数据文件

**解压即用，无需额外下载其他文件。**

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
