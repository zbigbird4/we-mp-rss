# Docker 镜像测试部署指南

## 概述

本文档说明如何使用 GitHub Actions 自动构建的 Docker 镜像进行测试部署。

## 镜像信息

### 自动构建

GitHub Actions 会在 `feat/webui-configurable-docker-envs` 分支有代码推送时自动构建 Docker 镜像。

**工作流文件**: `.github/workflows/docker-publish-feature.yaml`

### 镜像地址

- **测试镜像**: `ghcr.io/rachelos/we-mp-rss:test-latest`
- **分支镜像**: `ghcr.io/rachelos/we-mp-rss:feat-webui-configurable-docker-envs`

### 支持的架构

- `linux/amd64` (x86_64)
- `linux/arm64` (ARM 64位，如树莓派 4/5、Apple Silicon Mac)

## 快速开始

### 方法 1: 使用 docker-compose (推荐)

1. **下载测试配置文件**

```bash
# 创建测试目录
mkdir werss-test && cd werss-test

# 下载 docker-compose 配置
curl -O https://raw.githubusercontent.com/rachelos/we-mp-rss/feat/webui-configurable-docker-envs/docker-compose.test.yml
```

2. **创建数据目录**

```bash
mkdir -p data
```

3. **启动服务**

```bash
docker-compose -f docker-compose.test.yml up -d
```

4. **查看日志**

```bash
docker-compose -f docker-compose.test.yml logs -f
```

5. **访问服务**

打开浏览器访问: `http://localhost:8001`

### 方法 2: 使用 docker run

```bash
# 拉取镜像
docker pull ghcr.io/rachelos/we-mp-rss:test-latest

# 创建数据目录
mkdir -p $(pwd)/data

# 运行容器
docker run -d \
  --name werss-test \
  -p 8001:8001 \
  -v $(pwd)/data:/app/data \
  -e BROWSER_TYPE=firefox \
  -e SERVER_NAME=werss-test \
  -e WEB_NAME="WeRSS测试服务" \
  -e THREADS=2 \
  -e DEBUG=False \
  ghcr.io/rachelos/we-mp-rss:test-latest

# 查看日志
docker logs -f werss-test
```

## 配置说明

### 环境变量配置

所有配置都可以通过环境变量设置。以下是主要配置项：

#### 应用基础配置

```yaml
APP_NAME: we-mp-rss                    # 应用名称
SERVER_NAME: we-mp-rss                 # 服务名称
WEB_NAME: WeRSS微信公众号订阅助手       # 前端显示名称
PORT: 8001                             # 服务端口
DEBUG: False                           # 调试模式
```

#### 浏览器配置 (新功能 ⭐)

```yaml
BROWSER_TYPE: firefox                  # 浏览器类型: firefox/chromium/webkit
```

#### 认证配置

```yaml
WERSS_AUTH_WEB: False                  # Web方式授权
SEND_CODE: True                        # 发送授权二维码通知
CODE_TITLE: WeRSS授权二维码             # 二维码通知标题
```

#### 任务配置

```yaml
ENABLE_JOB: True                       # 启用定时任务
AUTO_RELOAD: False                     # 自动重载
THREADS: 2                             # 最大线程数
SPAN_INTERVAL: 10                      # 任务间隔(秒)
```

#### 数据库配置

```yaml
# SQLite (默认)
DB: sqlite:///data/db.db

# MySQL 示例
# DB: mysql+pymysql://user:password@host:3306/werss?charset=utf8mb4

# PostgreSQL 示例
# DB: postgresql://user:password@host:5432/werss
```

#### 通知配置

```yaml
DINGDING_WEBHOOK: ""                   # 钉钉 Webhook
WECHAT_WEBHOOK: ""                     # 企业微信 Webhook
FEISHU_WEBHOOK: ""                     # 飞书 Webhook
CUSTOM_WEBHOOK: ""                     # 自定义 Webhook
```

#### RSS 配置

```yaml
RSS_BASE_URL: ""                       # RSS域名地址
RSS_LOCAL: False                       # 本地RSS链接
RSS_TITLE: ""                          # RSS标题
RSS_DESCRIPTION: ""                    # RSS描述
RSS_COVER: ""                          # RSS封面
RSS_FULL_CONTEXT: True                 # 显示全文
RSS_ADD_COVER: True                    # 添加封面图片
RSS_CDATA: False                       # 启用CDATA
RSS_PAGE_SIZE: 30                      # 分页大小
```

#### 采集配置

```yaml
MAX_PAGE: 5                            # 最大采集页数
GATHER.CONTENT: True                   # 采集内容
GATHER.MODEL: app                      # 采集模式: app/web/api
GATHER.CONTENT_AUTO_CHECK: False       # 自动检查
GATHER.CONTENT_AUTO_INTERVAL: 59       # 自动检查间隔(分钟)
GATHER.CONTENT_MODE: web               # 内容修正模式
```

#### 导出配置

```yaml
EXPORT_PDF: False                      # 启用PDF导出
EXPORT_PDF_DIR: ./data/pdf             # PDF导出目录
EXPORT_MARKDOWN: False                 # 启用Markdown导出
EXPORT_MARKDOWN_DIR: ./data/markdown   # Markdown导出目录
```

### 配置优先级

1. **WebUI 修改** (最高优先级) - 通过 Web 界面修改的配置
2. **环境变量** - 通过 `-e` 参数或 docker-compose.yml 设置
3. **config.yaml** - 配置文件默认值
4. **代码默认值** (最低优先级)

## WebUI 配置功能测试

### 1. 访问配置管理页面

1. 登录 WebUI: `http://localhost:8001`
2. 进入"配置管理"菜单

### 2. 测试配置修改

尝试修改以下配置项：

- **服务器线程数**: 将 `server.threads` 从 2 改为 4
- **RSS分页大小**: 将 `rss.page_size` 从 30 改为 50
- **浏览器类型**: 将 `browser.type` 从 firefox 改为 chromium

### 3. 验证配置生效

```bash
# 查看配置文件是否更新
docker exec werss-test cat /app/config.yaml

# 查看容器日志确认配置重载
docker logs werss-test
```

### 4. 测试重启后配置保持

```bash
# 重启容器
docker restart werss-test

# 等待服务启动后，检查配置是否保持
docker exec werss-test cat /app/config.yaml
```

## 常用命令

### 容器管理

```bash
# 启动容器
docker-compose -f docker-compose.test.yml up -d

# 停止容器
docker-compose -f docker-compose.test.yml stop

# 重启容器
docker-compose -f docker-compose.test.yml restart

# 停止并删除容器
docker-compose -f docker-compose.test.yml down

# 停止并删除容器和数据卷
docker-compose -f docker-compose.test.yml down -v
```

### 日志查看

```bash
# 查看实时日志
docker-compose -f docker-compose.test.yml logs -f

# 查看最近100行日志
docker-compose -f docker-compose.test.yml logs --tail=100

# 查看特定时间的日志
docker-compose -f docker-compose.test.yml logs --since 30m
```

### 容器操作

```bash
# 进入容器
docker exec -it werss-test bash

# 查看配置文件
docker exec werss-test cat /app/config.yaml

# 查看数据目录
docker exec werss-test ls -la /app/data

# 查看进程
docker exec werss-test ps aux
```

### 镜像管理

```bash
# 拉取最新镜像
docker pull ghcr.io/rachelos/we-mp-rss:test-latest

# 查看镜像信息
docker inspect ghcr.io/rachelos/we-mp-rss:test-latest

# 查看镜像历史
docker history ghcr.io/rachelos/we-mp-rss:test-latest

# 删除旧镜像
docker image prune -a
```

## 数据备份与恢复

### 备份数据

```bash
# 备份整个数据目录
tar -czf werss-data-backup-$(date +%Y%m%d).tar.gz ./data

# 仅备份数据库
cp ./data/db.db ./werss-db-backup-$(date +%Y%m%d).db
```

### 恢复数据

```bash
# 停止容器
docker-compose -f docker-compose.test.yml stop

# 恢复数据
tar -xzf werss-data-backup-20241118.tar.gz

# 启动容器
docker-compose -f docker-compose.test.yml start
```

## 故障排查

### 容器无法启动

```bash
# 查看详细日志
docker-compose -f docker-compose.test.yml logs

# 检查端口是否被占用
sudo netstat -tlnp | grep 8001
# 或
sudo lsof -i :8001

# 检查数据目录权限
ls -la ./data
```

### 无法访问 WebUI

```bash
# 检查容器是否运行
docker ps | grep werss-test

# 检查健康状态
docker inspect werss-test | grep -A 10 Health

# 检查防火墙
sudo ufw status
```

### 配置修改不生效

```bash
# 检查配置文件
docker exec werss-test cat /app/config.yaml

# 重启容器
docker restart werss-test

# 查看启动日志
docker logs werss-test --tail 50
```

### 数据库问题

```bash
# 检查数据库文件
docker exec werss-test ls -la /app/data/db.db

# 检查数据库连接
docker exec werss-test python -c "from core.db import DB; print(DB.get_session())"
```

## 性能优化

### 资源限制

在 `docker-compose.test.yml` 中添加资源限制：

```yaml
services:
  werss:
    # ... 其他配置 ...
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 2G
        reservations:
          cpus: '1.0'
          memory: 512M
```

### 浏览器选择

不同浏览器的资源消耗：

- **firefox**: 中等资源消耗，推荐用于生产环境
- **chromium**: 较高资源消耗，兼容性最好
- **webkit**: 最低资源消耗，适合资源受限环境

修改 `BROWSER_TYPE` 环境变量或在 WebUI 中修改 `browser.type` 配置。

## 生产部署建议

### 1. 使用外部数据库

```yaml
environment:
  - DB=mysql+pymysql://werss:password@mysql:3306/werss?charset=utf8mb4
```

### 2. 使用 Docker Secrets (敏感信息)

```yaml
services:
  werss:
    secrets:
      - db_password
      - secret_key

secrets:
  db_password:
    external: true
  secret_key:
    external: true
```

### 3. 使用反向代理 (Nginx/Traefik)

```yaml
services:
  werss:
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.werss.rule=Host(`werss.example.com`)"
      - "traefik.http.services.werss.loadbalancer.server.port=8001"
```

### 4. 定期备份

设置 cron 任务自动备份：

```bash
# 每天凌晨2点备份
0 2 * * * cd /path/to/werss && tar -czf backup/werss-$(date +\%Y\%m\%d).tar.gz data/
```

## 升级指南

### 升级到最新版本

```bash
# 拉取最新镜像
docker pull ghcr.io/rachelos/we-mp-rss:test-latest

# 备份数据
tar -czf werss-backup-$(date +%Y%m%d).tar.gz ./data

# 重新创建容器
docker-compose -f docker-compose.test.yml up -d --force-recreate
```

### 回滚到旧版本

```bash
# 使用特定版本标签
docker pull ghcr.io/rachelos/we-mp-rss:v1.4.8

# 修改 docker-compose.test.yml 中的镜像标签
# image: ghcr.io/rachelos/we-mp-rss:v1.4.8

# 重新创建容器
docker-compose -f docker-compose.test.yml up -d --force-recreate
```

## 监控和日志

### Prometheus 监控 (可选)

```yaml
services:
  werss:
    environment:
      - ENABLE_METRICS=True
    ports:
      - "8001:8001"
      - "9090:9090"  # Prometheus metrics
```

### 日志收集 (可选)

```yaml
services:
  werss:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

## 支持与反馈

如果在测试过程中遇到问题：

1. 查看 [故障排查](#故障排查) 章节
2. 检查 GitHub Issues: https://github.com/rachelos/we-mp-rss/issues
3. 提交新 Issue 并附上：
   - Docker 版本: `docker --version`
   - 操作系统信息
   - 错误日志
   - 复现步骤

## 相关文档

- [WebUI 配置管理功能说明](./WEBUI_CONFIG_UPDATE.md)
- [项目 README](./ReadMe.md)
- [配置文件示例](./config.example.yaml)
