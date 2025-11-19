# WebUI 可配置 Docker 环境变量功能 - 功能总结

## 📋 功能概述

本次更新实现了在 Docker 部署环境下，通过 WebUI 直接配置和管理系统参数的功能。用户不再需要修改 `.env` 文件或 `docker-compose.yml`，可以直接在 Web 界面上修改配置，提升了 Docker 部署的易用性。

## ✨ 主要特性

### 1. WebUI 配置管理
- ✅ 在 WebUI 中查看所有系统配置
- ✅ 直接编辑和修改配置参数
- ✅ 配置修改实时保存到 `config.yaml`
- ✅ 部分配置修改后自动生效
- ✅ 敏感配置受保护，无法通过 WebUI 修改

### 2. Docker 部署优化
- ✅ 支持通过环境变量初始化配置
- ✅ 配置持久化到数据卷
- ✅ 重启容器后配置保持不变
- ✅ 支持多平台架构（amd64/arm64）

### 3. 浏览器类型配置（新增）
- ✅ 新增 `browser.type` 配置项
- ✅ 支持三种浏览器：firefox、chromium、webkit
- ✅ 可在 WebUI 中动态切换
- ✅ 优先从配置文件读取，兼容环境变量

### 4. 自动化构建部署
- ✅ GitHub Actions 自动构建 Docker 镜像
- ✅ 支持特性分支独立测试
- ✅ 多平台镜像自动构建
- ✅ 镜像自动推送到 GitHub Container Registry

## 📦 可配置参数列表

### 应用基础配置
| 配置键 | 环境变量 | 默认值 | 说明 |
|--------|----------|--------|------|
| `app_name` | `APP_NAME` | we-mp-rss | 应用名称 |

### 服务器配置
| 配置键 | 环境变量 | 默认值 | 说明 |
|--------|----------|--------|------|
| `server.name` | `SERVER_NAME` | we-mp-rss | 服务名称 |
| `server.web_name` | `WEB_NAME` | WeRSS微信公众号订阅助手 | 前端显示名称 |
| `server.auth_web` | `WERSS_AUTH_WEB` | False | 通过web方式授权 |
| `server.send_code` | `SEND_CODE` | True | 是否发送授权二维码通知 |
| `server.code_title` | `CODE_TITLE` | WeRSS授权二维码 | 二维码通知标题 |
| `server.enable_job` | `ENABLE_JOB` | True | 是否启用定时任务 |
| `server.auto_reload` | `AUTO_RELOAD` | False | 代码修改自动重启服务 |
| `server.threads` | `THREADS` | 2 | 最大线程数 |

### 浏览器配置（⭐ 新增）
| 配置键 | 环境变量 | 默认值 | 说明 |
|--------|----------|--------|------|
| `browser.type` | `BROWSER_TYPE` | firefox | 浏览器类型: firefox/chromium/webkit |

### 数据库配置
| 配置键 | 环境变量 | 默认值 | 说明 |
|--------|----------|--------|------|
| `db` | `DB` | sqlite:///data/db.db | 数据库连接字符串（🔒受保护） |

### 通知配置
| 配置键 | 环境变量 | 默认值 | 说明 |
|--------|----------|--------|------|
| `notice.dingding` | `DINGDING_WEBHOOK` | 空 | 钉钉通知Webhook（🔒受保护） |
| `notice.wechat` | `WECHAT_WEBHOOK` | 空 | 微信通知Webhook（🔒受保护） |
| `notice.feishu` | `FEISHU_WEBHOOK` | 空 | 飞书通知Webhook（🔒受保护） |
| `notice.custom` | `CUSTOM_WEBHOOK` | 空 | 自定义通知Webhook（🔒受保护） |

### 安全配置
| 配置键 | 环境变量 | 默认值 | 说明 |
|--------|----------|--------|------|
| `secret` | `SECRET_KEY` | we-mp-rss | 密钥（🔒受保护） |
| `safe.lic_key` | `SAFE_LIC_KEY` | RACHELOS | 授权加密KEY（🔒受保护） |

### 其他配置

详见 [WEBUI_CONFIG_UPDATE.md](./WEBUI_CONFIG_UPDATE.md) 获取完整配置列表。

## 🔒 受保护配置

以下配置出于安全考虑，不能在 WebUI 中修改：
- 数据库连接 (`db`)
- 密钥 (`secret`)
- 通知 Webhook 地址
- 授权密钥 (`safe.lic_key`)
- 所有包含 `token` 的配置

这些配置只能通过环境变量或直接编辑配置文件修改。

## 🏗️ 技术实现

### 后端修改

1. **配置管理 API** (`apis/config_management.py`)
   - 修改 PUT 接口支持写入 YAML 文件
   - 自动重新加载配置

2. **YAML 配置管理器** (`core/yaml_db/store_config.py`)
   - 新增 `update_config_in_yaml()` 方法
   - 支持嵌套配置更新
   - 自动类型转换

3. **浏览器类型配置** (`driver/playwright_driver.py`)
   - 新增 `get_browser_type()` 函数
   - 优先从配置文件读取

### 前端修改

1. **配置列表页面** (`web_ui/src/views/ConfigList.vue`)
   - 启用编辑功能
   - 区分受保护配置
   - 添加操作提示

### DevOps 优化

1. **GitHub Actions 工作流** (`.github/workflows/docker-publish-feature.yaml`)
   - 特性分支自动构建
   - 多平台支持
   - 缓存优化

2. **测试部署配置** (`docker-compose.test.yml`)
   - 完整的环境变量示例
   - 数据持久化配置
   - 健康检查配置

## 📸 使用截图

### WebUI 配置管理界面

```
┌────────────────────────────────────────────────────────────┐
│  配置管理                                                   │
├────────────────────────────────────────────────────────────┤
│  ℹ️ 您可以在此页面查看和编辑系统配置。受保护的配置项       │
│     （如数据库连接、密钥等）无法在此编辑。                 │
├────────┬──────────────┬────────────────┬──────────────────┤
│ 配置键 │ 配置值       │ 描述           │ 操作             │
├────────┼──────────────┼────────────────┼──────────────────┤
│ app_name│ we-mp-rss   │ 系统配置项     │ [编辑]           │
│ server.name│ we-mp-rss│ server配置子项 │ [编辑]           │
│ server.threads│ 2     │ server配置子项 │ [编辑]           │
│ browser.type│ firefox │ browser配置子项│ [编辑]           │
│ db     │ ***          │ 系统配置项     │ [受保护]         │
│ secret │ ***          │ 系统配置项     │ [受保护]         │
└────────┴──────────────┴────────────────┴──────────────────┘
```

### 配置编辑对话框

```
┌─────────────────────────────────────┐
│  编辑配置                           │
├─────────────────────────────────────┤
│  配置键:                            │
│  ┌───────────────────────────────┐ │
│  │ server.threads                │ │
│  └───────────────────────────────┘ │
│                                     │
│  配置值: *                          │
│  ┌───────────────────────────────┐ │
│  │ 4                             │ │
│  └───────────────────────────────┘ │
│                                     │
│  描述:                              │
│  ┌───────────────────────────────┐ │
│  │ server配置的子项              │ │
│  └───────────────────────────────┘ │
│                                     │
│           [取消]     [确定]         │
└─────────────────────────────────────┘
```

## 🚀 部署测试

### 快速启动

```bash
# 拉取镜像
docker pull ghcr.io/zbigbird4/we-mp-rss:test-latest

# 启动容器
docker run -d \
  --name werss-test \
  -p 8001:8001 \
  -v $(pwd)/data:/app/data \
  -e BROWSER_TYPE=firefox \
  ghcr.io/zbigbird4/we-mp-rss:test-latest

# 访问 WebUI
open http://localhost:8001
```

### 使用 docker-compose

```bash
# 下载配置文件
curl -O https://raw.githubusercontent.com/zbigbird4/we-mp-rss/feat/webui-configurable-docker-envs/docker-compose.test.yml

# 启动服务
docker-compose -f docker-compose.test.yml up -d
```

详细部署指南请查看：
- [快速开始测试指南](./QUICK_START_TEST.md)
- [Docker 测试部署指南](./DOCKER_TEST_DEPLOYMENT.md)

## 📊 测试覆盖

### 功能测试
- ✅ WebUI 配置列表显示
- ✅ 配置编辑功能
- ✅ 配置保存到 YAML
- ✅ 配置重新加载
- ✅ 受保护配置验证
- ✅ 配置持久化

### Docker 测试
- ✅ 镜像构建
- ✅ 多平台支持
- ✅ 环境变量配置
- ✅ 数据卷挂载
- ✅ 容器重启配置保持
- ✅ 健康检查

### 浏览器类型测试
- ✅ Firefox 模式
- ✅ Chromium 模式
- ✅ WebKit 模式
- ✅ 动态切换

## 🐛 已知问题

无已知问题。

## 📝 更新日志

### v1.4.9 (Feature Branch)

#### 新增功能
- WebUI 配置管理页面可编辑功能
- 浏览器类型配置 (`browser.type`)
- 配置自动保存到 YAML 文件
- 配置修改后自动重新加载
- 受保护配置识别和保护

#### 优化改进
- Docker 镜像自动构建流程
- 多平台架构支持（amd64/arm64）
- 配置优先级管理
- 测试部署文档

#### 技术改进
- YAML 配置管理器增强
- API 接口优化
- 前端交互体验提升

## 🔗 相关链接

- **GitHub 仓库**: https://github.com/zbigbird4/we-mp-rss
- **功能分支**: feat/webui-configurable-docker-envs
- **Docker 镜像**: ghcr.io/zbigbird4/we-mp-rss:test-latest
- **GitHub Actions**: https://github.com/zbigbird4/we-mp-rss/actions

## 📚 文档索引

1. [WebUI 配置更新说明](./WEBUI_CONFIG_UPDATE.md) - 详细的功能说明和配置列表
2. [Docker 测试部署指南](./DOCKER_TEST_DEPLOYMENT.md) - 完整的 Docker 部署文档
3. [快速开始测试指南](./QUICK_START_TEST.md) - 5分钟快速测试指南
4. [配置文件示例](./config.example.yaml) - 配置文件模板

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

## 📄 许可证

MIT License

---

**开发者**: Rachel OS Team  
**更新日期**: 2024-11-18  
**版本**: v1.4.9 (Feature Preview)
