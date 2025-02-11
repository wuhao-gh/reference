Caddy 备忘清单
===

这个 [Caddy](https://caddyserver.com/) 快速参考备忘单显示了它的常用命令和配置使用清单。

入门
----

### 安装

```bash
# macOS 安装
brew install caddy

# Ubuntu/Debian 安装
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy

# RHEL/Fedora 安装
dnf install 'dnf-command(copr)'
dnf copr enable @caddy/caddy
dnf install caddy
```

### 服务管理

```bash
# 启动 Caddy
caddy start

# 停止 Caddy
caddy stop

# 重新加载配置
caddy reload

# 验证配置
caddy validate

# 格式化 Caddyfile
caddy fmt

# 将 Caddyfile 转换为 JSON
caddy adapt

# 查看版本
caddy version
```

### Docker 安装

```bash
docker pull caddy
docker run -d \
  -p 80:80 \
  -p 443:443 \
  -v caddy_data:/data \
  -v caddy_config:/config \
  -v $PWD/Caddyfile:/etc/caddy/Caddyfile \
  -v $PWD/site:/srv \
  caddy
```

Caddyfile 配置
----

### 基础配置

```caddyfile
# 最简单的静态文件服务器
:80 {
  root * /var/www
  file_server
}

# 带域名的配置
example.com {
  root * /var/www/example
  file_server
  encode gzip
  tls your@email.com  # 自动 HTTPS
}
```

### 反向代理

```caddyfile
example.com {
  reverse_proxy localhost:3000  # 简单反向代理
  
  # 带健康检查的反向代理
  reverse_proxy {
    to localhost:3000
    health_uri /health
    health_interval 10s
  }
  
  # 负载均衡
  reverse_proxy {
    to localhost:3000 localhost:3001 localhost:3002
    lb_policy round_robin
  }
}
```

### PHP 配置

```caddyfile
example.com {
  root * /var/www/wordpress
  php_fastcgi unix//run/php/php-fpm.sock
  file_server
}

# 或者使用 TCP
example.com {
  root * /var/www/wordpress
  php_fastcgi 127.0.0.1:9000
  file_server
}
```

### HTTPS 配置

```caddyfile
example.com {
  # 自动 HTTPS（默认）
  tls your@email.com
  
  # 使用自定义证书
  tls {
    cert_file /path/to/cert.pem
    key_file  /path/to/key.pem
  }
  
  # 禁用自动 HTTPS
  tls internal
}
```

### 压缩和缓存

```caddyfile
example.com {
  # 启用压缩
  encode gzip zstd
  
  # 文件缓存
  file_server {
    precompressed gzip br
  }
  
  # 静态资源缓存头
  header /static/* Cache-Control "public, max-age=31536000"
}
```

### 重定向和重写

```caddyfile
# HTTP 重定向到 HTTPS（自动）
example.com {
  redir https://{host}{uri}
}

# WWW 重定向
www.example.com {
  redir https://example.com{uri}
}

# URL 重写
example.com {
  rewrite /old-path /new-path
  
  # 正则重写
  rewrite /api/* /v2/api/{1}
}
```

### 中间件和日志

```caddyfile
example.com {
  # 访问日志
  log {
    output file /var/log/access.log
    format json
  }
  
  # IP 过滤
  @blocked {
    remote_ip 192.168.1.0/24
  }
  respond @blocked "Access denied" 403
  
  # 基本认证
  basicauth {
    user JDJhJDE0JE91S2l4dzhCRmNPc3NCb3pSZERKbE9xNzRrQzZmL0RjZVhzNk93TnVoMHVmVmJxY1TyWkdp
  }
}
```

### 常用指令

| 指令 | 说明 |
|------|------|
| `root` | 设置站点根目录 |
| `file_server` | 启用静态文件服务 |
| `reverse_proxy` | 反向代理配置 |
| `tls` | HTTPS 配置 |
| `encode` | 响应压缩 |
| `header` | 设置 HTTP 头 |
| `redir` | URL 重定向 |
| `rewrite` | URL 重写 |
| `handle` | 请求处理块 |
| `handle_path` | 带路径的请求处理 |
| `respond` | 返回固定响应 |
| `basicauth` | 基本认证 |
| `log` | 日志配置 |

### 性能优化

```caddyfile
example.com {
  # 启用压缩
  encode gzip zstd
  
  # 静态资源缓存
  header /static/* {
    Cache-Control "public, max-age=31536000"
    defer
  }
  
  # 启用 HTTP/2 Server Push
  push /assets/css/style.css
  push /assets/js/script.js
  
  # 文件服务器性能优化
  file_server {
    precompressed gzip br
    hide .git
  }
}
```

### 开发环境配置

```caddyfile
localhost:8080 {
  # 开发模式，禁用 HTTPS
  tls internal
  
  # 自动重载 PHP
  php_fastcgi 127.0.0.1:9000 {
    resolve_root_symlink
    env DEVELOPMENT 1
  }
  
  # 启用目录浏览
  file_server browse
  
  # 开发日志
  log {
    output stdout
    format console
    level DEBUG
  }
}
```
