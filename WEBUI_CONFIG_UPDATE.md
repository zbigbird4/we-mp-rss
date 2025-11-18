# WebUI 配置管理功能更新说明

## 概述
本次更新实现了在 WebUI 中直接编辑和更新系统配置的功能。用户可以通过 Web 界面修改配置参数，修改后的配置会自动保存到 `config.yaml` 文件中。

## 主要修改内容

### 1. 后端修改

#### 1.1 配置更新API (`apis/config_management.py`)
- 修改了 `PUT /api/v1/wx/configs/{config_key}` 接口
- 现在更新配置时会：
  1. 将新配置值写入 `config.yaml` 文件
  2. 重新加载配置使其立即生效（部分配置仍需重启）
  3. 返回更新成功的响应

#### 1.2 YAML配置管理器 (`core/yaml_db/store_config.py`)
- 添加了 `update_config_in_yaml()` 方法
- 支持更新嵌套配置（如 `server.name`、`rss.title` 等）
- 自动进行类型转换（字符串、数字、布尔值）
- 保持YAML文件格式和结构完整性

#### 1.3 浏览器类型配置 (`driver/playwright_driver.py`)
- 添加了 `get_browser_type()` 函数
- 优先从配置文件读取浏览器类型，其次从环境变量读取
- 支持在 WebUI 中配置浏览器类型（firefox/chromium/webkit）

#### 1.4 配置文件模板 (`config.example.yaml`)
- 新增 `browser.type` 配置项
- 更新 `server.code_title` 默认值为 "WeRSS授权二维码"

### 2. 前端修改

#### 2.1 配置列表页面 (`web_ui/src/views/ConfigList.vue`)
- 启用了配置编辑功能
- 添加"操作"列，显示编辑按钮
- 受保护的配置项（数据库连接、密钥等）显示为"受保护"标签，不可编辑
- 编辑对话框使用多行文本框（textarea），方便编辑长文本
- 添加了成功提示消息和错误提示
- 添加了信息提示，说明配置修改后可能需要重启服务

## 可配置参数列表

以下配置可在 WebUI 中修改（受保护的配置除外）：

### 应用配置
- `app_name`: 应用名称（默认: we-mp-rss）

### 服务器配置 (server.*)
- `server.name`: 服务名称（默认: we-mp-rss）
- `server.web_name`: 前端显示名称（默认: WeRSS微信公众号订阅助手）
- `server.send_code`: 是否发送授权二维码通知（默认: True）
- `server.code_title`: 二维码通知标题（默认: WeRSS授权二维码）
- `server.enable_job`: 是否启用定时任务（默认: True）
- `server.auto_reload`: 代码修改自动重启服务（默认: False）
- `server.threads`: 最大线程数（默认: 2）
- `server.auth_web`: 通过web方式授权（默认: False）

### 浏览器配置 (browser.*) ⭐ 新增
- `browser.type`: 浏览器类型，可选 firefox/chromium/webkit（默认: firefox）

### 其他配置
- `user_agent`: 用户代理字符串
- `interval`: 定时任务执行间隔（秒）
- `port`: API服务端口（默认: 8001）
- `debug`: 调试模式（默认: False）
- `max_page`: 最大采集页数（默认: 5）
- `token_expire_minutes`: 登录会话有效时长（分钟，默认: 4320）

### Webhook配置 (webhook.*)
- `webhook.content_format`: 文章内容发送格式，可选 html/text/markdown（默认: html）

### RSS配置 (rss.*)
- `rss.base_url`: RSS域名地址
- `rss.local`: 是否为本地RSS链接（默认: False）
- `rss.title`: RSS标题
- `rss.description`: RSS描述
- `rss.cover`: RSS封面
- `rss.full_context`: 是否显示全文（默认: True）
- `rss.add_cover`: 是否添加封面图片（默认: True）
- `rss.cdata`: 是否启用CDATA（默认: False）
- `rss.page_size`: RSS分页大小（默认: 30）

### 缓存配置 (cache.*)
- `cache.dir`: 缓存目录（默认: ./data/cache）

### 文章配置 (article.*)
- `article.true_delete`: 是否真实删除文章（默认: False）

### 采集配置 (gather.*)
- `gather.content`: 是否采集内容（默认: True）
- `gather.model`: 采集模式，可选 web/api/app（默认: app）
- `gather.content_auto_check`: 是否自动检查未采集文章内容（默认: False）
- `gather.content_auto_interval`: 自动检查未采集文章内容的时间间隔（分钟，默认: 59）
- `gather.content_mode`: 内容修正模式，可选 web/api（默认: web）

### 日志配置 (log.*)
- `log.file`: 日志文件路径（默认: 空）
- `log.level`: 日志级别，可选 DEBUG/INFO/WARNING/ERROR/CRITICAL（默认: INFO）

### 导出配置 (export.pdf.*, export.markdown.*)
- `export.pdf.enable`: 是否启用PDF导出功能（默认: False）
- `export.pdf.dir`: PDF导出目录（默认: ./data/pdf）
- `export.markdown.enable`: 是否启用markdown导出功能（默认: False）
- `export.markdown.dir`: markdown导出目录（默认: ./data/markdown）

## 受保护的配置（不可在 WebUI 编辑）

以下配置出于安全考虑，不能在 WebUI 中编辑：
- `db`: 数据库连接字符串
- `secret`: 密钥
- `notice.dingding`: 钉钉通知Webhook地址
- `notice.wechat`: 微信通知Webhook地址
- `notice.feishu`: 飞书通知Webhook地址
- `notice.custom`: 自定义通知Webhook地址
- `safe.lic_key`: 授权加密KEY
- 所有包含 `token` 的配置项

这些配置只能通过以下方式修改：
1. 直接编辑 `config.yaml` 文件
2. 设置环境变量
3. 修改 `docker-compose.yml` 文件（Docker部署）

## 使用说明

### 1. 访问配置管理页面
登录 WebUI 后，在导航菜单中选择"配置管理"或"配置"。

### 2. 查看配置
配置列表会显示所有当前配置项，包括：
- 配置键：配置项的名称
- 配置值：当前的配置值（受保护的配置显示为 `***`）
- 描述：配置项的说明

### 3. 编辑配置
1. 点击可编辑配置项右侧的"编辑"按钮
2. 在弹出的对话框中修改配置值
3. 点击"确定"保存修改

### 4. 配置生效
- 大部分配置会立即生效
- 某些配置（如端口、数据库连接等）需要重启服务才能生效
- 修改成功后会显示提示消息

## Docker 部署注意事项

使用 Docker 部署时：
1. 配置文件 `config.yaml` 应该挂载到容器中（通常在 `/app/config.yaml` 或 `/data/config.yaml`）
2. 确保配置文件具有写入权限
3. 修改配置后，如需重启可以使用：
   ```bash
   docker-compose restart
   ```
4. 环境变量仍然有效，但 WebUI 修改会覆盖环境变量设置

## 技术细节

### 配置值类型转换
系统会自动将配置值转换为合适的类型：
- `"true"` / `"false"` → 布尔值
- 纯数字字符串 → 整数
- 带小数点的数字 → 浮点数
- 其他 → 字符串

### 配置文件结构
配置使用 YAML 格式存储，支持嵌套结构：
```yaml
server:
  name: we-mp-rss
  threads: 2
  
rss:
  title: 我的RSS
  page_size: 30
```

WebUI 中使用点号表示法访问嵌套配置：`server.name`、`rss.title` 等。

## 测试

可以使用以下命令测试配置更新功能：

```bash
# 进入项目目录并激活虚拟环境
cd /home/engine/project
source .venv/bin/activate

# 测试配置更新
python -c "
from core.config import cfg
from core.yaml_db import YamlDB

# 更新配置
YamlDB.update_config_in_yaml('server.threads', '4')

# 重新加载并验证
cfg.reload()
print('server.threads:', cfg.get('server.threads'))
"
```

## 向后兼容性

本次更新完全向后兼容：
- 所有现有的配置方式（环境变量、直接编辑文件）仍然有效
- Docker 部署无需修改 `docker-compose.yml` 文件
- 现有的配置文件可以直接使用，无需迁移

## 安全建议

1. 建议定期备份 `config.yaml` 文件
2. 在生产环境中修改配置前，建议先在测试环境验证
3. 重要配置（如数据库连接、密钥）使用环境变量或受保护配置
4. 确保只有授权用户可以访问配置管理页面
