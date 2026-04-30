# OpenClaw 部署文档（Docker 版）

> 本文档适用于 Ubuntu 24.04 LTS 系统，使用 Docker + Docker Compose 部署 OpenClaw。

---

## 第一步：准备工作空间

创建一个目录来存放配置文件和持久化数据：

```bash
mkdir -p ~/openclaw && cd ~/openclaw
````

---

## 第二步：创建 `docker-compose.yml`

使用你喜欢的编辑器创建配置文件，例如：

```bash
nano docker-compose.yml
```

将以下配置粘贴进去（通用生产环境模版）：

```yaml
version: '3.8'

services:
  # 主程序
  openclaw:
    image: alpine/openclaw:latest
    container_name: openclaw_app
    restart: always
    ports:
      - "8080:18789"  # 访问地址：http://服务器IP:8080
    environment:
      - TZ=Asia/Shanghai
      - REDIS_HOST=redis
      - REDIS_PORT=6379
    volumes:
      - ./data:/app/data
      - ./logs:/app/logs
    depends_on:
      - redis

  # 缓存与队列
  redis:
    image: redis:alpine
    container_name: openclaw_redis
    restart: always
    command: redis-server --appendonly yes
    volumes:
      - ./redis_data:/data
```

赋予目录权限
由于 alpine 镜像通常以非 root 用户运行，建议提前手动创建目录并放开权限，防止容器启动时报错 Permission denied：

```
mkdir -p data redis_data
chmod 777 data redis_data
```

---

## 第三步：启动容器

在 `docker-compose.yml` 所在目录下执行：

```bash
# -d 参数表示后台模式运行
docker compose up -d
```

---

## 第四步：验证状态

查看容器运行状态：

```bash
docker compose ps
```

如果显示状态为 `Up`，说明服务已经成功启动。

---

想要web访问，推荐使用ssh隧道，做本地端口转发，避免跨域问题和ssl验证
sed -i '/"allowedOrigins": \[/a \        "*",'  openclaw.json

## 第五步：防火墙设置（关键）

Ubuntu 系统通常自带 `ufw` 防火墙，如果无法访问，请确保放行 8080 端口：

```bash
sudo ufw allow 8080/tcp
sudo ufw reload
```

---

## 常见操作指南

* **访问地址**：浏览器访问 `http://你的服务器IP:8080`
* **查看日志**：如果启动失败，可排查问题：

  ```bash
  docker compose logs -f openclaw
  ```
* **更新版本**：

  ```bash
  docker compose pull
  docker compose up -d
  ```

---

## 温馨提示

1. **数据持久化**
   上述配置将数据映射到当前目录下的 `data` 文件夹，即使删除容器，数据不会丢失。

2. **安全性**
   如果是公网服务器，建议配置 Nginx 反向代理并加上 SSL 证书，不要直接暴露 8080 端口。

3. **自定义配置**
   如果 OpenClaw 有特定环境变量需求，请参考源码包内的 `.env.example` 进行微调。

---


