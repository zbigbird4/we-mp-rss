# 快速开始测试指南 🚀

## 当前状态

✅ 代码已推送到 GitHub  
✅ GitHub Actions 正在构建 Docker 镜像  
⏳ 等待镜像构建完成...

## GitHub Actions 构建状态

查看构建进度：
- **GitHub Actions**: https://github.com/zbigbird4/we-mp-rss/actions
- **工作流**: Docker Image Publish (Feature Branch)
- **分支**: feat/webui-configurable-docker-envs

## 构建完成后的测试步骤

### 1️⃣ 快速测试（5分钟）

```bash
# 创建测试目录
mkdir werss-test && cd werss-test

# 拉取镜像（构建完成后）
docker pull ghcr.io/zbigbird4/we-mp-rss:test-latest

# 创建数据目录
mkdir -p data

# 启动容器（最简配置）
docker run -d \
  --name werss-test \
  -p 8001:8001 \
  -v $(pwd)/data:/app/data \
  -e BROWSER_TYPE=firefox \
  ghcr.io/zbigbird4/we-mp-rss:test-latest

# 查看日志
docker logs -f werss-test
```

访问: http://localhost:8001

### 2️⃣ 完整测试（使用 docker-compose）

```bash
# 下载配置文件
curl -O https://raw.githubusercontent.com/zbigbird4/we-mp-rss/feat/webui-configurable-docker-envs/docker-compose.test.yml

# 创建数据目录
mkdir -p data

# 启动服务
docker-compose -f docker-compose.test.yml up -d

# 查看日志
docker-compose -f docker-compose.test.yml logs -f
```

## 测试 WebUI 配置功能

### 步骤 1: 登录系统

1. 访问 http://localhost:8001
2. 使用默认账号登录（首次启动会提示创建管理员账号）

### 步骤 2: 进入配置管理

1. 点击左侧菜单 "配置管理" 或 "配置"
2. 查看所有可配置项列表

### 步骤 3: 测试配置修改

#### 测试 1: 修改服务器线程数

1. 找到配置项 `server.threads`
2. 点击"编辑"按钮
3. 将值从 `2` 改为 `4`
4. 点击"确定"保存
5. 检查是否显示成功提示

#### 测试 2: 修改浏览器类型（新功能）

1. 找到配置项 `browser.type`
2. 点击"编辑"按钮
3. 将值从 `firefox` 改为 `chromium`
4. 点击"确定"保存
5. 重启容器查看是否生效：
   ```bash
   docker restart werss-test
   docker logs werss-test | grep browser
   ```

#### 测试 3: 修改 RSS 配置

1. 找到配置项 `rss.page_size`
2. 点击"编辑"按钮
3. 将值从 `30` 改为 `50`
4. 点击"确定"保存

#### 测试 4: 验证受保护配置

1. 找到配置项 `db`（数据库连接）
2. 确认显示"受保护"标签，无法编辑
3. 验证其他受保护配置：`secret`、`notice.wechat` 等

### 步骤 4: 验证配置持久化

```bash
# 查看配置文件是否已更新
docker exec werss-test cat /app/config.yaml | grep -A 2 "server:"
docker exec werss-test cat /app/config.yaml | grep -A 2 "browser:"

# 重启容器
docker restart werss-test

# 等待10秒后再次检查配置
sleep 10
docker exec werss-test cat /app/config.yaml | grep threads

# 在 WebUI 中确认配置值是否保持
```

## 环境变量 vs WebUI 配置

### 测试配置优先级

```bash
# 1. 使用环境变量启动（设置 threads=2）
docker run -d \
  --name werss-priority-test \
  -p 8002:8001 \
  -v $(pwd)/data2:/app/data \
  -e THREADS=2 \
  ghcr.io/zbigbird4/we-mp-rss:test-latest

# 2. 在 WebUI 中修改 server.threads 为 4

# 3. 验证配置（应该显示 4，WebUI 优先级更高）
docker exec werss-priority-test cat /app/config.yaml | grep threads

# 4. 重启容器，配置应该保持为 4
docker restart werss-priority-test
docker exec werss-priority-test cat /app/config.yaml | grep threads
```

## 预期结果

### ✅ 成功指标

1. **镜像构建成功**
   - GitHub Actions 显示绿色勾号
   - 镜像可以正常拉取

2. **容器正常启动**
   - 容器状态为 `Up`
   - 日志无错误信息
   - 健康检查通过

3. **WebUI 可访问**
   - 可以正常访问 http://localhost:8001
   - 页面加载正常
   - 可以成功登录

4. **配置管理功能正常**
   - 可以查看所有配置项
   - 可以编辑非受保护配置
   - 受保护配置正确显示
   - 修改后配置立即生效

5. **配置持久化正常**
   - config.yaml 文件已更新
   - 重启后配置保持
   - 数据卷挂载正常

### ❌ 常见问题

#### 问题 1: 镜像拉取失败

```bash
# 检查镜像是否存在
docker search ghcr.io/zbigbird4/we-mp-rss

# 检查 GitHub Actions 构建状态
# 访问: https://github.com/zbigbird4/we-mp-rss/actions

# 如果构建失败，查看工作流日志
```

#### 问题 2: 容器启动失败

```bash
# 查看详细错误
docker logs werss-test

# 检查端口占用
sudo netstat -tlnp | grep 8001

# 检查数据目录权限
ls -la data/
```

#### 问题 3: 配置修改不生效

```bash
# 检查配置文件权限
docker exec werss-test ls -la /app/config.yaml

# 手动测试配置更新
docker exec werss-test python -c "
from core.yaml_db import YamlDB
result = YamlDB.update_config_in_yaml('server.threads', '8')
print('Update result:', result)
"

# 查看日志中的错误信息
docker logs werss-test | grep -i error
```

## 性能测试

### 测试不同浏览器类型

```bash
# 测试 Firefox
docker run -d --name werss-firefox -p 8001:8001 \
  -e BROWSER_TYPE=firefox \
  ghcr.io/zbigbird4/we-mp-rss:test-latest

# 测试 Chromium
docker run -d --name werss-chromium -p 8002:8001 \
  -e BROWSER_TYPE=chromium \
  ghcr.io/zbigbird4/we-mp-rss:test-latest

# 测试 WebKit
docker run -d --name werss-webkit -p 8003:8001 \
  -e BROWSER_TYPE=webkit \
  ghcr.io/zbigbird4/we-mp-rss:test-latest

# 比较资源使用
docker stats werss-firefox werss-chromium werss-webkit
```

## 清理测试环境

```bash
# 停止所有测试容器
docker stop werss-test werss-priority-test werss-firefox werss-chromium werss-webkit 2>/dev/null

# 删除所有测试容器
docker rm werss-test werss-priority-test werss-firefox werss-chromium werss-webkit 2>/dev/null

# 删除测试镜像（可选）
docker rmi ghcr.io/zbigbird4/we-mp-rss:test-latest

# 清理数据目录（谨慎操作）
rm -rf werss-test/
```

## 报告问题

如果发现任何问题，请在 GitHub 提交 Issue：

**Issue 模板**:

```markdown
### 环境信息
- Docker 版本: `docker --version`
- 操作系统: 
- 镜像版本: test-latest
- 测试时间: 

### 问题描述
[详细描述遇到的问题]

### 复现步骤
1. 
2. 
3. 

### 预期行为
[描述预期应该发生什么]

### 实际行为
[描述实际发生了什么]

### 日志信息
```bash
docker logs werss-test
```

### 截图
[如果有截图请附上]
```

## 下一步

测试成功后：

1. ✅ 确认所有功能正常工作
2. 📝 更新 README 文档
3. 🔀 创建 Pull Request 合并到主分支
4. 🏷️ 打标签发布新版本
5. 🚀 更新生产环境部署文档

## 相关文档

- [详细部署指南](./DOCKER_TEST_DEPLOYMENT.md)
- [WebUI 配置功能说明](./WEBUI_CONFIG_UPDATE.md)
- [项目 README](./ReadMe.md)

---

**注意**: 首次构建可能需要 10-15 分钟，请耐心等待 GitHub Actions 完成构建。

构建完成后，你会在 https://github.com/zbigbird4/we-mp-rss/pkgs/container/we-mp-rss 看到新的镜像。
