# TabOS 云桌面系统 - Linux 完整部署指南

> **版本：** v1.0  
> **更新日期：** 2026-06-12  
> **适用系统：** Ubuntu 20.04+ / CentOS 7+ / Debian 10+

---

## 📋 目录

1. [环境准备](#1-环境准备)
2. [上传与解压](#2-上传与解压)
3. [启动后端服务](#3-启动后端服务)
4. [访问安装向导](#4-访问安装向导)
5. [配置管理后台](#5-配置管理后台)
6. [Nginx 反向代理配置](#6-nginx-反向代理配置)
7. [常见问题排查](#7-常见问题排查)

---

## 1. 环境准备

### 1.1 系统要求

| 组件 | 最低配置 | 推荐配置 |
|------|---------|---------|
| CPU | 1 核 | 2 核以上 |
| 内存 | 1GB | 4GB 以上 |
| 磁盘 | 10GB | 50GB 以上 |
| 系统 | Ubuntu 18.04+ | Ubuntu 22.04 LTS |

### 1.2 必需软件

#### MySQL 8.0+ 或 MariaDB 10.5+（推荐）

**Ubuntu/Debian 安装 MySQL：**
```bash
# 更新软件源
sudo apt update

# 安装 MySQL
sudo apt install mysql-server -y

# 启动并设置开机自启
sudo systemctl start mysql
sudo systemctl enable mysql

# 安全配置（设置 root 密码）
sudo mysql_secure_installation
```

**CentOS/RHEL 安装 MySQL：**
```bash
# 添加 MySQL 官方源
sudo yum install -y https://dev.mysql.com/get/mysql80-community-release-el7-5.noarch.rpm

# 安装 MySQL
sudo yum install -y mysql-server

# 启动并设置开机自启
sudo systemctl start mysqld
sudo systemctl enable mysqld

# 查看初始密码
sudo grep 'temporary password' /var/log/mysqld.log

# 修改 root 密码
mysql_secure_installation
```

**创建数据库和用户：**
```bash
# 登录 MySQL
mysql -u root -p

# 执行以下 SQL
CREATE DATABASE tabos CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'tabos'@'localhost' IDENTIFIED BY '你的密码';
GRANT ALL PRIVILEGES ON tabos.* TO 'tabos'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

#### Redis 7.0+（强烈推荐）

**Ubuntu/Debian 安装：**
```bash
sudo apt install redis-server -y
sudo systemctl start redis
sudo systemctl enable redis
```

**CentOS/RHEL 安装：**
```bash
sudo yum install -y epel-release
sudo yum install -y redis
sudo systemctl start redis
sudo systemctl enable redis
```

**验证 Redis 运行：**
```bash
redis-cli ping
# 应该返回：PONG
```

#### Nginx（用于反向代理）

**Ubuntu/Debian：**
```bash
sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
```

**CentOS/RHEL：**
```bash
sudo yum install -y nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

---

## 2. 上传与解压

### 2.1 创建部署目录

```bash
# 创建项目根目录
sudo mkdir -p /www/tabos
cd /www/tabos

# 创建后端和前端目录
sudo mkdir -p backend frontend
```

### 2.2 上传文件

**方式一：使用 scp 上传（从本地 Windows/Mac）**

```bash
# 上传后端可执行文件
scp tabos-backend root@your-server-ip:/www/tabos/backend/

# 上传前端打包文件
scp -r dist/* root@your-server-ip:/www/tabos/frontend/
```

**方式二：使用 FTP 工具**
- 推荐使用 FileZilla、WinSCP 等工具
- 上传 `tabos-backend` 到 `/www/tabos/backend/`
- 上传前端 `dist` 目录所有文件到 `/www/tabos/frontend/`

**方式三：使用 wget 下载（如果有下载链接）**

```bash
cd /www/tabos/backend
sudo wget https://your-domain.com/tabos-backend
sudo chmod +x tabos-backend
```

### 2.3 解压并设置权限

```bash
# 如果是压缩包
cd /www/tabos
sudo tar -xzf tabos-backend.tar.gz -C backend/
sudo tar -xzf tabos-frontend.tar.gz -C frontend/

# 给后端执行权限
sudo chmod +x /www/tabos/backend/tabos-backend

# 设置目录权限
sudo chown -R www-data:www-data /www/tabos
```

### 2.4 验证文件结构

```bash
tree -L 2 /www/tabos
```

应该看到：
```
/www/tabos/
├── backend/
│   ├── tabos-backend (可执行文件)
│   └── config/ (后续自动生成)
└── frontend/
    ├── index.html
    ├── assets/
    └── ...
```

---

## 3. 启动后端服务

### 3.1 直接启动（测试）

```bash
cd /www/tabos/backend

# 直接运行
./tabos-backend

# 或者后台运行
nohup ./tabos-backend > tabos.log 2>&1 &
```

启动后会看到：
```
✅ 配置文件加载成功
✅ Redis 连接成功
🚀 服务已启动，监听: 0.0.0.0:18080
🏠 本机访问: http://localhost:18080
🔗 API: http://localhost:18080/api/v1
```

### 3.2 使用 systemd 服务（推荐生产环境）

创建服务文件：

```bash
sudo nano /etc/systemd/system/tabos.service
```

写入以下内容：

```ini
[Unit]
Description=TabOS Backend Service
After=network.target mysql.service redis.service

[Service]
Type=simple
User=www-data
WorkingDirectory=/www/tabos/backend
ExecStart=/www/tabos/backend/tabos-backend
Restart=on-failure
RestartSec=5s

# WebSocket Origin 白名单（修改为你的域名）
Environment="ALLOWED_ORIGINS=https://your-domain.com"

# 日志输出
StandardOutput=append:/var/log/tabos/tabos.log
StandardError=append:/var/log/tabos/tabos.error.log

[Install]
WantedBy=multi-user.target
```

创建日志目录并启动：

```bash
# 创建日志目录
sudo mkdir -p /var/log/tabos
sudo chown www-data:www-data /var/log/tabos

# 重载 systemd
sudo systemctl daemon-reload

# 启动服务
sudo systemctl start tabos

# 设置开机自启
sudo systemctl enable tabos

# 查看状态
sudo systemctl status tabos

# 查看日志
sudo journalctl -u tabos -f
```

### 3.3 验证后端运行

```bash
# 检查端口监听
ss -tuln | grep 18080

# 健康检查
curl http://localhost:18080/api/v1/health

# 应该返回：{"code":0,"message":"ok"}
```

---

## 4. 访问安装向导

### 4.1 临时访问方式（IP + 端口）

在浏览器中访问：

```
http://你的服务器IP:18080/install
```

例如：`http://192.168.1.100:18080/install`

### 4.2 配置数据库

**推荐配置（MySQL + Redis）：**

| 配置项 | 填写内容 |
|--------|---------|
| 数据库类型 | MySQL |
| 数据库地址 | 127.0.0.1 |
| 端口 | 3306 |
| 数据库名 | tabos |
| 用户名 | tabos |
| 密码 | 你创建用户时设置的密码 |
| 自动创建数据库 | ✅ 勾选 |

**Redis 配置：**

| 配置项 | 填写内容 |
|--------|---------|
| 启用缓存服务 | ✅ 勾选 |
| Redis 地址 | 127.0.0.1 |
| 端口 | 6379 |
| 密码 | 留空（如果没设置密码） |
| 缓存库编号 | 0 |

**⚠️ 测试连接：填写完成后，务必点击"测试连接"按钮，确保连接成功！**

### 4.3 站点设置（重要）

| 配置项 | 填写说明 | 示例 |
|--------|---------|------|
| 服务端口 | 保持默认 18080 | 18080 |
| 服务模式 | 生产环境选"正式运行" | 正式运行 |
| **网站访问地址** | ⭐ 填写你的前端域名（必填） | https://tabos.example.com |
| 接口访问地址 | 可选，留空即可 | （留空） |
| 实时连接地址 | 可选，留空即可 | （留空） |
| 允许访问的前端地址 | 可选，留空即可 | （留空） |

**⚠️ 关键说明：**
- **"网站访问地址"必须填写你绑定的域名**，格式：`https://your-domain.com`
- 如果暂时用 IP 访问，就填 `http://你的IP`
- 这个地址决定了 WebSocket 连接是否成功！

### 4.4 安全设置

系统会自动生成密钥，**直接使用自动生成的即可，不要修改**。

### 4.5 创建管理员

| 配置项 | 说明 |
|--------|------|
| 用户名 | 默认 admin，可修改 |
| 邮箱 | 管理员邮箱 |
| 密码 | 至少 8 位，包含大小写字母和数字 |
| 显示名 | 系统管理员 |

### 4.6 完成安装

点击"开始安装"，等待系统自动重启（约 5-10 秒）。

安装完成后会显示：
```
✅ 安装完成，系统正在自动重启...
前端首页：https://your-domain.com/
管理端：https://your-domain.com/admin/
```

---

## 5. 配置管理后台

### 5.1 访问管理后台

安装完成后，访问：

```
https://your-domain.com/admin/
```

使用刚才创建的管理员账号登录。

### 5.2 绑定前端域名（关键步骤）

登录管理后台后：

**步骤 1：进入站点设置**
```
系统设置 → 站点设置
```

**步骤 2：填写站点地址（URL）**

在"站点地址（URL）"框中填写：
```
https://your-domain.com
```

**⚠️ 重要说明：**
- 这个地址**必须与你的前端访问域名一致**
- 不要带尾部斜杠
- 必须包含协议（http:// 或 https://）
- 这个配置决定 WebSocket 连接白名单

**步骤 3：保存配置**

点击"保存"按钮，系统会自动重载配置。

### 5.3 验证配置是否生效

打开浏览器开发者工具（F12），查看 Console 和 Network：

✅ **正常情况：**
```
WebSocket connection established: wss://your-domain.com/api/v1/ws/realtime
WebSocket connection established: wss://your-domain.com/api/v1/im/ws
```

❌ **异常情况（需要检查站点 URL 是否配置正确）：**
```
WebSocket connection failed
[Security] 拒绝来自未授权来源的 WebSocket 连接
```

---

## 6. Nginx 反向代理配置

### 6.1 创建站点配置文件

```bash
sudo nano /etc/nginx/sites-available/tabos
```

### 6.2 完整配置

```nginx
server {
    listen 80;
    server_name ;
    
    root /www/tabos/frontend;
    index index.html;
    
    access_log /var/log/nginx/tabos.access.log;
    error_log /var/log/nginx/tabos.error.log;
    
    location ^~ /api/uploads {
        proxy_pass http://127.0.0.1:18080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_http_version 1.1;
    }
    
    location ^~ /api
    {
        proxy_pass http://127.0.0.1:18080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header REMOTE-HOST $remote_addr;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_http_version 1.1;
        
        proxy_buffering off;
        
        client_max_body_size 10m;
        
        proxy_connect_timeout 60s;
        proxy_send_timeout 86400s;
        proxy_read_timeout 86400s;
        
        add_header Cache-Control "no-cache, no-store, must-revalidate";
        add_header Pragma "no-cache";
        add_header Expires "0";
    }
    
    location ^~ /install {
        proxy_pass http://127.0.0.1:18080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_http_version 1.1;
        add_header Cache-Control "no-cache, no-store, must-revalidate";
    }
    
    location ~ ^/kuwo-cdn/([^/]+)(/.*)$ {
        resolver 8.8.8.8 valid=300s;
        resolver_timeout 5s;

        set $kuwo_host $1;
        set $kuwo_path $2;

        proxy_pass http://$kuwo_host$kuwo_path$is_args$args;
        proxy_set_header Host $kuwo_host;
        proxy_set_header Referer "https://www.kuwo.cn/";
        proxy_set_header User-Agent "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36";
        proxy_set_header Range $http_range;
        proxy_set_header If-Range $http_if_range;

        proxy_buffering off;

        proxy_connect_timeout 10s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;

        proxy_force_ranges on;

        add_header Access-Control-Allow-Origin "*" always;
        add_header Access-Control-Allow-Methods "GET, HEAD, OPTIONS" always;
        add_header Access-Control-Allow-Headers "Range" always;
        add_header Access-Control-Expose-Headers "Content-Range, Content-Length" always;

        if ($request_method = OPTIONS) {
            return 204;
        }
    }
    
    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

### 6.3 启用配置并测试

```bash
# 创建软链接
sudo ln -s /etc/nginx/sites-available/tabos /etc/nginx/sites-enabled/

# 测试配置语法
sudo nginx -t

# 重载 Nginx
sudo nginx -s reload
```

### 6.4 配置 HTTPS（推荐）

**使用 Let's Encrypt 免费证书：**

```bash
# 安装 certbot
sudo apt install certbot python3-certbot-nginx -y

# 自动配置 SSL
sudo certbot --nginx -d your-domain.com

# 自动续期
sudo certbot renew --dry-run
```

配置完成后，Nginx 会自动添加 SSL 配置和 HTTP 到 HTTPS 的重定向。

---

## 7. 常见问题排查

### 7.1 WebSocket 连接失败

**症状：**
```
WebSocket connection failed
request origin not allowed by Upgrader.CheckOrigin
```

**解决方法：**

1. **检查管理后台"站点设置"**
   - 进入：系统设置 → 站点设置
   - 确认"站点地址（URL）"填写正确：`https://your-domain.com`

2. **检查 systemd 环境变量**
   ```bash
   sudo nano /etc/systemd/system/tabos.service
   ```
   确认有这行：
   ```ini
   Environment="ALLOWED_ORIGINS=https://your-domain.com"
   ```
   
   修改后重启服务：
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl restart tabos
   ```

3. **检查 Nginx 配置**
   - 确认 `map $http_upgrade $connection_upgrade` 在 `server` 块之前
   - 确认 `/api` location 有 WebSocket 相关配置

### 7.2 后端启动失败

**检查日志：**
```bash
# 查看 systemd 日志
sudo journalctl -u tabos -n 50

# 查看后端日志文件
sudo tail -f /var/log/tabos/tabos.error.log
```

**常见原因：**
- 端口 18080 被占用：`ss -tuln | grep 18080`
- MySQL 连接失败：检查数据库账号密码
- Redis 连接失败：`redis-cli ping`

### 7.3 前端页面空白

**检查：**
1. Nginx 配置的 `root` 路径是否正确
2. 前端文件是否完整上传
3. 浏览器开发者工具 Console 是否有错误

### 7.4 管理后台 404

**原因：**前端没有正确配置管理后台路由。

**解决：**
确保前端打包时 base 路径正确，并且 Nginx 的 `try_files` 生效。

### 7.5 上传文件失败

**检查：**
1. Nginx `client_max_body_size` 是否足够大（默认 100m）
2. 后端 uploads 目录权限：
   ```bash
   sudo chown -R www-data:www-data /www/tabos/backend/uploads
   ```

---

## 📞 技术支持

如果遇到无法解决的问题：

1. 查看后端日志：`sudo journalctl -u tabos -f`
2. 查看 Nginx 错误日志：`sudo tail -f /var/log/nginx/tabos.error.log`
3. 检查浏览器开发者工具 Console 和 Network 标签

---

## ✅ 部署检查清单

- [ ] MySQL 安装并创建数据库
- [ ] Redis 安装并正常运行
- [ ] 后端可执行文件上传并赋予执行权限
- [ ] 前端文件上传到正确目录
- [ ] 后端服务正常启动（端口 18080 监听）
- [ ] 访问 `/install` 完成初始化
- [ ] 管理后台"站点设置"配置正确的域名
- [ ] Nginx 反向代理配置完成
- [ ] WebSocket 连接正常
- [ ] HTTPS 证书配置完成（推荐）

完成以上所有步骤后，系统即可正常运行！
