# 🚀 Docker 镜像测试快速指南

## 📦 镜像信息

- **镜像地址**: `ghcr.io/zbigbird4/we-mp-rss:test-latest`
- **构建状态**: https://github.com/zbigbird4/we-mp-rss/actions
- **支持架构**: linux/amd64, linux/arm64

## ⚡ 5 分钟快速测试

### 步骤 1: 拉取镜像

```bash
docker pull ghcr.io/zbigbird4/we-mp-rss:test-latest
```

### 步骤 2: 启动容器

```bash
# 创建数据目录
mkdir -p werss-test-data

# 启动容器
docker run -d \
  --name werss-test \
  -p 8001:8001 \
  -v $(pwd)/werss-test-data:/app/data \
  -e BROWSER_TYPE=firefox \
  -e THREADS=2 \
  ghcr.io/zbigbird4/we-mp-rss:test-latest
```

### 步骤 3: 查看日志

```bash
docker logs -f werss-test
```

### 步骤 4: 访问 WebUI

打开浏览器访问: **http://localhost:8001**

## 🧪 测试 WebUI 配置功能

### 1. 登录系统
- 首次访问会提示创建管理员账号
- 使用创建的账号登录

### 2. 进入配置管理
- 点击左侧菜单"配置管理"或"配置"

### 3. 测试配置编辑

#### 测试 A: 修改线程数
```
1. 找到 server.threads
2. 点击"编辑"
3. 改为 4
4. 保存
```

#### 测试 B: 修改浏览器类型（新功能）
```
1. 找到 browser.type
2. 点击"编辑"
3. 改为 chromium 或 webkit
4. 保存
5. 重启容器: docker restart werss-test
```

#### 测试 C: 验证受保护配置
```
1. 找到 db 配置
2. 确认显示"受保护"，无法编辑
```

### 4. 验证配置持久化

```bash
# 查看配置文件
docker exec werss-test cat /app/config.yaml | grep threads

# 重启容器
docker restart werss-test

# 再次查看配置（应该保持不变）
docker exec werss-test cat /app/config.yaml | grep threads
```

## 📋 使用 docker-compose 测试

### 1. 下载配置文件

```bash
curl -O https://raw.githubusercontent.com/zbigbird4/we-mp-rss/feat/webui-configurable-docker-envs/docker-compose.test.yml
```

### 2. 启动服务

```bash
docker-compose -f docker-compose.test.yml up -d
```

### 3. 查看日志

```bash
docker-compose -f docker-compose.test.yml logs -f
```

## 🔧 可配置的环境变量

### 基础配置
```bash
-e APP_NAME=we-mp-rss
-e SERVER_NAME=we-mp-rss
-e WEB_NAME="WeRSS微信公众号订阅助手"
```

### 浏览器配置（⭐ 新增）
```bash
-e BROWSER_TYPE=firefox    # firefox/chromium/webkit
```

### 服务配置
```bash
-e THREADS=2               # 最大线程数
-e ENABLE_JOB=True         # 启用定时任务
-e PORT=8001               # 服务端口
-e DEBUG=False             # 调试模式
```

### RSS 配置
```bash
-e RSS_PAGE_SIZE=30        # RSS 分页大小
-e RSS_FULL_CONTEXT=True   # 显示全文
```

**完整配置列表**: 查看 [docker-compose.test.yml](./docker-compose.test.yml)

## ❌ 清理测试环境

```bash
# 停止并删除容器
docker stop werss-test && docker rm werss-test

# 删除测试数据（可选）
rm -rf werss-test-data

# 删除镜像（可选）
docker rmi ghcr.io/zbigbird4/we-mp-rss:test-latest
```

## 📚 详细文档

| 文档 | 说明 |
|------|------|
| [QUICK_START_TEST.md](./QUICK_START_TEST.md) | 快速开始指南 |
| [DOCKER_TEST_DEPLOYMENT.md](./DOCKER_TEST_DEPLOYMENT.md) | 详细部署指南 |
| [WEBUI_CONFIG_UPDATE.md](./WEBUI_CONFIG_UPDATE.md) | 功能详细说明 |
| [FEATURE_SUMMARY.md](./FEATURE_SUMMARY.md) | 功能总结 |
| [TASK_COMPLETION_REPORT.md](./TASK_COMPLETION_REPORT.md) | 任务完成报告 |

## 🐛 遇到问题？

### 问题 1: 镜像拉取失败
```bash
# 检查网络连接
ping ghcr.io

# 检查 Docker 登录（如果需要）
docker login ghcr.io
```

### 问题 2: 容器启动失败
```bash
# 查看详细日志
docker logs werss-test

# 检查端口占用
sudo lsof -i :8001
```

### 问题 3: 无法访问 WebUI
```bash
# 检查容器状态
docker ps | grep werss-test

# 检查防火墙
sudo ufw status
```

**更多故障排查**: 查看 [DOCKER_TEST_DEPLOYMENT.md](./DOCKER_TEST_DEPLOYMENT.md#故障排查)

## 💬 反馈

测试过程中发现问题或有建议？

- **提交 Issue**: https://github.com/zbigbird4/we-mp-rss/issues
- **查看文档**: https://github.com/zbigbird4/we-mp-rss/tree/feat/webui-configurable-docker-envs

## ⭐ 新功能预览

### WebUI 配置管理
✅ 在 WebUI 中直接查看和编辑配置  
✅ 配置实时保存到文件  
✅ 重启后配置保持不变  
✅ 敏感配置自动保护  

### 浏览器类型配置
✅ 支持 Firefox/Chromium/WebKit  
✅ 可在 WebUI 中动态切换  
✅ 优化资源消耗  

### Docker 部署优化
✅ 多平台支持（amd64/arm64）  
✅ 完整的环境变量支持  
✅ 数据持久化  
✅ 健康检查  

---

**祝测试顺利！** 🎉

有任何问题随时查看详细文档或提交 Issue。
