# 星程课表 (AstraSchedule) 项目知识库

## 项目概述

中国学校班级电子课表管理系统，包含 Electron 客户端、Vue3 管理后台、Go 后端 API、Rspress 文档站点、注册中心和 Go 工具库。

## 项目结构

根目录为 monorepo（git submodule），各子项目独立 Git 仓库：

```
AstraSchedule/
├── desktop/          # Electron 客户端（主应用）—— Bun 包管理
├── usr-dashboard/    # Vue3 + Vite 网页管理后台（用户端）—— Bun 包管理
├── usr-backend/      # Go 后端 API（用户端，Gin + GORM）—— Go Modules
├── sys-dashboard/    # Vue3 + Vite + NaiveUI Dashboard（系统端）—— Bun 包管理
├── sys-backend/      # Go + Gin 系统后端 —— Go Modules
├── docs-site/        # Rspress 文档站点 —— Bun 包管理
├── reg-go/           # Go 注册中心后端 —— Go Modules
├── reg-to/           # 注册中心前端（Vue3 + Vite）—— Bun 包管理
├── go-valence-cal/   # Go 工具库（调休计算）—— Go Modules
├── docs/             # AI compose plans（不纳入 Git）
└── .gitmodules       # Submodule 配置
```

## 查找位置

| 任务 | 位置 | 说明 |
|------|------|------|
| 客户端开发 | `desktop/` | Electron + 原生 JS + jQuery |
| 用户端管理后台 | `usr-dashboard/` | Vue3 + Naive UI + Vite |
| 用户端后端 API | `usr-backend/` | Go + Gin + GORM |
| 系统端 Dashboard | `sys-dashboard/` | Vue3 + Naive UI + Vite |
| 系统端后端 | `sys-backend/` | Go + Gin |
| 文档编写 | `docs-site/` | Rspress + React |
| 注册中心后端 | `reg-go/` | Go + Gin |
| 注册中心前端 | `reg-to/` | Vue3 + Vite |
| 调休计算 | `go-valence-cal/` | Go 工具库 |

## 代码地图

### 核心子项目

| 子项目 | 技术栈 | 入口文件 | 主要功能 |
|--------|--------|----------|----------|
| desktop | Electron + JS + jQuery | `main.js` | 课表显示、倒计时、天气、WebSocket |
| usr-dashboard | Vue3 + Vite + Naive UI | `src/main.js` | 用户端管理后台、配置管理 |
| usr-backend | Go + Gin + GORM | `main.go` | 用户端 API、数据存储、调休计算 |
| sys-dashboard | Vue3 + Vite + Naive UI | `src/main.js` | 系统端 Dashboard |
| sys-backend | Go + Gin | `main.go` | 系统端 API |
| docs-site | Rspress + React | `rspress.config.ts` | 项目文档 |
| reg-go | Go + Gin | `main.go` | 注册中心后端 |
| reg-to | Vue3 + Vite | `src/main.js` | 注册中心前端 |
| go-valence-cal | Go + Carbon | `valence.go` | 中国节假日调休计算 |

### 架构模式

- **Monorepo + Submodule**：根目录通过 git submodule 管理 9 个独立 Git 仓库
- **前后端分离**：客户端/管理后台通过 API 与后端通信
- **作用域模型**：学校/年级/班级三级数据隔离
- **调休自动计算**：基于 `go-valence-cal` 库自动推算课表调整

## 约定

### API 约定

- 管理端接口前缀：`/web/*`
- 客户端接口：无前缀（如 `/:school/:grade/:class`）
- 认证方式：BasicAuth（用户名 `ElectronClassSchedule` 或 `AstraSchedule`）
- 响应格式：`status`/`message`/`data` 或 `error`/`detail`

### 数据库约定

- 支持 MySQL 和 SQLite（通过配置切换）
- 使用 GORM 模型（`model/dbTable/*`）
- 写入操作使用 upsert（`ON CONFLICT ... UPDATE ALL`）
- 多表操作使用事务，失败原子回滚

### UI 约定

- 管理端菜单由后端 `/web/menu` 接口驱动
- 使用 Naive UI 组件库
- 支持深色/浅色主题自动切换

### 版本号约定

- 格式：`YYYYMM.D.N`（年月.日.运行序号）
- 示例：`202510.2.40` 表示 2025年10月02日，第40次构建
- 用于 electron-updater 自动更新

### 编码约定

- **换行符**：CRLF（所有子项目）
- **缩进**：desktop JS/CSS 用 4 空格，package.json/HTML 用 2 空格；usr-dashboard 统一 2 空格
- **模块系统**：usr-dashboard 和 docs-site 使用 ES Modules（`"type": "module"`）
- **包管理器**：所有 Node.js 子项目统一使用 Bun
- **Commit 格式**：`<emoji><type>(<scope>): <description>`（个人项目，遵循 Conventional Commits 规范）
  - ✨ feat（新功能）、🐛 fix（修复）、🔒 security（安全）、📝 docs（文档）
  - 🎉 init（初始化）、🔧 refactor（重构）、♻️ style（风格）、⚡ perf（性能）
  - ✅ test（测试）、🔨 build（构建）、⏪ revert（回滚）
- **Commit 语言**：description 必须使用**中文**编写
- **由 AI 编写的代码提交**：需要在 Commit Message 后添加 `(By AI)` 标识
- **Commit 流程**：AI 生成 commit message 后必须先展示给用户确认，用户同意后才能执行 `git commit`

### 分支管理规则

- **main 分支**：基础版本，**不包含** namespace 多租户功能
- **saas/main 分支**：SaaS 版本，包含 namespace 多租户功能（usr-backend、usr-dashboard）
- **合并方向**：main → saas/main（单向），**严禁** saas/main → main，即使用户要求也不行
- **如需将 saas/main 的修改引入 main**：必须逐个 cherry-pick 或手动提取单独的 commit，不得整分支合并
- **工作分支确认**：AI 开始每个新任务前，**必须**询问用户在哪个分支上工作，每个新任务都要问，不能只问一次

> **注意**：`sys-dashboard` 和 `sys-backend` **没有** saas/main 分支，它们的 main 分支等效于 usr-backend 的 saas/main（已包含 namespace 多租户功能）。

## 文档约定

- 修改代码后，如果涉及**新增功能**或**破坏性修改**，需询问用户是否更新 `docs-site` 中的相关文档
- 文档位于 `docs-site/docs/`，使用 Rspress + Markdown 格式
- AI 生成的文档页面需在开头添加 DANGER 标识

## 反模式（本项目禁止）

### 安全相关

- **禁止** 在 Electron 中使用 `nodeIntegration: true` + `contextIsolation: false`（当前实现）
- **禁止** 提交 `config.toml`（包含敏感配置）
- **禁止** 提交构建产物（`*.exe`, `*.zip`, `*.log`）
- **禁止** 在代码中硬编码许可证密钥

### 代码质量

- **禁止** 在 Go 后端中使用 `as any`、`@ts-ignore`、`@ts-expect-error`
- **禁止** 删除失败测试来"通过"
- **禁止** 在修复 bug 时进行重构
- **禁止** 使用空的 `catch` 块

### 项目管理

- **禁止** 在根目录创建 CI 工作流（各子项目独立管理）
- **禁止** 混合不同子项目的关注点（除非跨层接线）

## 独特风格

### 版本号系统

使用自定义滚动版本号 `YYYYMM.D.N`，而非标准 semver。electron-updater 通过 `isSemver()` 检查适配。

### 窗口穿透交互

Electron 客户端使用鼠标穿透 + 轮询检测实现悬停交互，非标准 Electron 模式。

### 课表规则引擎

Go 后端实现 4 类规则（调休/作息/课表/组合），按优先级叠加应用。

### Serverless 适配

后端支持 `serverless` 模式，禁用 WebSocket，适合云函数部署。

## 命令

```bash
# 客户端开发
cd desktop && bun run debug

# 用户端管理后台开发
cd usr-dashboard && bun run dev

# 用户端后端构建
cd usr-backend && go build

# 用户端后端格式化
cd usr-backend && go fmt ./...

# 用户端后端依赖整理
cd usr-backend && go mod tidy

# 文档站点开发
cd docs-site && bun run dev

# 文档站点构建
cd docs-site && bun run build
```

## 注意事项

### 构建环境

- **Electron 客户端**：需要 Node.js v20+、VS2019+、Python 3.8+，且必须先执行 `pip install setuptools`
- **Go 后端**：需要 Go Modules 支持
- **管理后台**：使用 Bun（有 `bun.lock`），需要 Node.js v18+
- **文档站点**：使用 Bun（有 `bun.lock`）

### 数据兼容性

- 新数据逻辑必须同时兼容 MySQL 和 SQLite
- API 改动必须同步更新 OpenAPI 文档
- 破坏性变更需与用户确认

### 部署架构

- 客户端、管理后台、后端可独立部署
- 支持 Serverless 部署（云函数、Netlify Functions）
- 自动更新通过 GitHub Releases（镜像源 `hubproxy.khbit.cn`）

### 关键依赖关系

- `usr-backend` 依赖 `go-valence-cal`（通过 Go Modules `go.sum` 引入）
- `desktop` 通过 API 与 `usr-backend` 通信
- `usr-dashboard` 通过 API 与 `usr-backend` 通信

## 子项目详细信息

### desktop（客户端）

#### 项目结构
```
desktop/
├── main.js                    # 主进程入口
├── index.html                 # 主窗口 HTML（课表界面）
├── countdown.html             # 倒数日独立窗口 HTML
├── package.json               # 项目配置
├── main/                      # 主进程模块
│   └── countdown/             # 倒数日子系统（state/ipc/service/window）
├── js/                        # 渲染进程脚本
│   ├── scheduleConfig.js      # 默认课表配置（JSONC模板）
│   ├── index.js               # 课表核心逻辑
│   ├── renderer.js            # 主窗口 UI 渲染
│   └── countdown*.js          # 倒数日相关脚本
├── css/                       # 样式文件
├── font/                      # 字体文件
└── image/                     # 图标资源
```

#### 查找位置
| 任务 | 位置 | 说明 |
|------|------|------|
| 主进程逻辑 | `main.js` | WebSocket、托盘、IPC、天气、更新 |
| 课表显示 | `js/index.js` | 时间计算、课程高亮 |
| UI 渲染 | `js/renderer.js` | 界面更新、动画 |
| 配置管理 | `js/scheduleConfig.js` | 用户可编辑配置 |
| 倒数日子系统 | `main/countdown/` | 独立窗口管理 |
| 窗口创建 | `main/countdown/window.js` | BrowserWindow 配置 |

#### 约定
- **编码规范**：
  - 缩进：4 空格（JS/CSS），2 空格（package.json/HTML）
  - 行尾：CRLF（Windows）
  - 编码：UTF-8
  - 尾部空格：自动修剪
- **配置合并**：
  - 默认配置：`js/scheduleConfig.js`
  - 用户配置：`scheduleConfig.user.jsonc`（JSONC 格式，支持注释）
  - 云端配置：通过 API 下载，优先级最高
- **IPC 通信**：
  - 主进程 → 渲染进程：`win.webContents.send`
  - 渲染进程 → 主进程：`ipcRenderer.send`
  - 响应式调用：`ipcRenderer.invoke` + `ipcMain.handle`

#### 反模式
- **安全相关**：
  - **禁止** 在生产环境使用 `nodeIntegration: true` + `contextIsolation: false`（当前实现）
  - **禁止** 在代码中硬编码 API 密钥或令牌
- **代码质量**：
  - **禁止** 在 `main.js` 中添加新功能模块（应提取到 `main/` 目录）
  - **禁止** 修改 `scheduleConfig.js` 的默认值（应通过用户配置覆盖）
  - **禁止** 使用 `any` 类型（虽然不是 TypeScript，但应避免松散类型）
- **构建相关**：
  - **禁止** 提交 `dist/` 目录（构建产物）
  - **禁止** 提交 `node_modules/` 目录
  - **禁止** 在未安装 `setuptools` 的情况下运行 `npm install`

#### 独特风格
- **窗口穿透交互**：
  - 使用 `win.setIgnoreMouseEvents(true, { forward: true })` 实现鼠标穿透
  - 通过轮询（100ms）检测鼠标位置，判断是否在可交互区域
  - 悬停时降低 `opacity` 至 0.1 作为视觉反馈
- **版本号系统**：
  - 格式：`YYYYMM.D.N`（年月.日.运行序号）
  - 示例：`202510.2.40` 表示 2025年10月02日，第40次构建
  - electron-updater 通过 `isSemver()` 检查适配
- **三层存储**：
  - 主进程：`electron-store`（JSON 持久化）
  - 渲染进程：`localStorage`（临时状态）
  - 用户配置：JSONC 文件（支持注释）

#### 命令
```bash
# 开发调试
bun run debug

# 构建（Windows ia32）
bun run build

# CI 构建并发布
bun run ci
```

#### 注意事项
- **构建环境**：
  - 需要 Node.js v20+、VS2019+、Python 3.8+
  - **必须**先执行 `pip install setuptools` 再执行 `npm install`
  - 构建产物仅支持 Windows ia32 架构
- **自动更新**：
  - 发布到 `daizihan233/AstraSchedule` 仓库
  - 镜像源：`hubproxy.khbit.cn`
  - 仅打包模式 + semver 版本时生效
- **窗口配置**：
  - 主窗口：无边框、透明、置顶、不可调整大小
  - 倒数日窗口：无边框、透明、不可移动、鼠标穿透
  - 托盘菜单：左键/右键点击打开

### usr-dashboard（用户端管理后台）

#### 项目结构
```
usr-dashboard/
├── index.html              # 入口 HTML
├── package.json            # 依赖配置
├── vite.config.js          # Vite 构建配置
├── src/
│   ├── main.js             # 应用入口
│   ├── App.vue             # 根组件
│   ├── global.js           # 全局配置（API地址）
│   ├── api/                # API 封装层
│   │   ├── autorun.js      # 自动任务 API
│   │   ├── backup.js       # 备份 API
│   │   └── countdown.js    # 倒计时 API
│   ├── components/         # 公共组件
│   ├── router/             # 路由配置
│   ├── utils/              # 工具函数
│   └── views/              # 页面组件
```

#### 查找位置
| 任务 | 位置 | 说明 |
|------|------|------|
| API 调用 | `src/api/` | 封装后端 API 请求 |
| 页面组件 | `src/views/` | 各功能页面 |
| 路由配置 | `src/router/index.js` | 页面路由定义 |
| 全局配置 | `src/global.js` | API 服务器地址 |
| 工具函数 | `src/utils/` | 作用域工具等 |
| 公共组件 | `src/components/` | ConfirmPasswordModal 等 |

#### 约定
- **编码规范**：
  - 缩进：2 空格（JS/CSS/HTML/package.json）
  - 行尾：CRLF（Windows）
  - 编码：UTF-8
  - 尾部空格：自动修剪
  - 模块系统：ES Modules（`"type": "module"`）
- **路径别名**：
  - `@` 映射到 `./src` 目录
  - 使用示例：`import xxx from '@/utils/xxx'`
- **API 调用**：
  - 使用 `vue-request` 库管理请求状态
  - 支持轮询（如首页 1 秒轮询统计）
  - 认证方式：BasicAuth（用户在界面输入密码）
  - Serverless 模式下，总览页根据 `/web/statistic` 返回的 `serverless` 字段决定是否轮询和显示数据
- **路由结构**：
  - 模式：HTML5 History 模式
  - 所有路由采用懒加载
  - 参数：`:school`、`:grade`、`:cls`（班级）、`:id`

#### 反模式
- **安全相关**：
  - **禁止** 在代码中存储密码（密码仅在界面输入，不持久化）
  - **禁止** 跳过 BasicAuth 验证进行写操作
- **代码质量**：
  - **禁止** 在 `src/views/` 中添加非页面组件
  - **禁止** 使用 `any` 类型（虽然不是 TypeScript，但应避免松散类型）
  - **禁止** 直接修改 `naive-ui` 组件源码
- **构建相关**：
  - **禁止** 提交 `dist/` 目录（构建产物）
  - **禁止** 提交 `node_modules/` 目录
  - **禁止** 在生产环境暴露 Source Map

#### 独特风格
- **动态菜单驱动**：
  - 侧边栏菜单从后端 API（`/web/menu`）动态获取
  - 支持多级学校/年级/班级结构
  - 菜单结构由后端控制，前端仅渲染
- **作用域系统**：
  - 使用 `学校/年级/班级` 格式的路径字符串
  - 支持学校级、年级级、班级级三种粒度
  - 路由参数自动解析作用域
- **主题适配**：
  - 自动检测系统主题（深色/浅色）
  - 使用 Naive UI 的主题系统
  - 使用 `useOsTheme` 响应系统主题变化
- **实时监控**：
  - 首页通过 1 秒轮询显示 WebSocket 连接状态
  - 显示客户端统计和各班连接状态表格

#### 命令
```bash
# 开发服务器
bun run dev

# 生产构建
bun run build

# 预览构建结果
bun run preview

# Docker 开发
bun run docker-dev

# Docker 预览
bun run docker-preview
```

#### 注意事项
- **开发环境**：
  - 需要 Node.js v18+
  - 使用 Bun 包管理器
  - 使用 Vite 开发服务器（支持热重载）
  - 推荐安装 Vue.volar VSCode 扩展
- **API 服务器**：
  - 默认地址：`http://127.0.0.1:9000`
  - 生产地址：通过 `src/global.js` 配置
  - 开发服务器允许主机：`manager.khbit.cn`
- **认证机制**：
  - 所有写操作需要 BasicAuth 密码验证
  - 用户名：`ElectronClassSchedule`
  - 密码：由用户在界面输入
- **数据备份**：
  - 支持导出/导入备份（JSON 格式）
  - 支持完整备份（包括配置和自动任务）
  - 支持跨数据库类型迁移（MySQL ↔ SQLite）

### usr-backend（用户端后端）

#### 项目结构
```
usr-backend/
├── main.go                    # 入口文件，路由定义和启动
├── config.toml                # 实际配置（TOML 格式）
├── config.template.toml       # 配置模板
├── AstraServerGo.openapi.json # OpenAPI 规范文档
├── config/                    # 配置加载层
│   └── load.go                # Viper 读取 TOML 配置
├── model/                     # 数据模型层
│   ├── globalVar.go           # 全局变量
│   ├── server.go              # 服务器配置结构体
│   ├── srvConfig.go           # 服务配置及校验
│   ├── weather.go             # 天气 API 响应结构
│   └── dbTable/               # 数据库表模型（GORM）
├── db/                        # 数据访问层（DAL）
├── service/                   # 业务逻辑层
├── middleware/                 # 中间件
├── router/                    # HTTP 处理层
│   ├── client/                # 客户端 API
│   └── web/                   # Web 管理后台 API
└── startup/                   # 启动初始化
```

#### 查找位置
| 任务 | 位置 | 说明 |
|------|------|------|
| API 路由定义 | `main.go` | 所有路由注册 |
| 客户端 API | `router/client/` | 课表、天气、WebSocket |
| 管理端 API | `router/web/` | 配置、备份、调休 |
| 数据库模型 | `model/dbTable/` | GORM 表结构 |
| 数据访问 | `db/` | CRUD 操作 |
| 业务逻辑 | `service/` | 规则引擎 |
| 中间件 | `middleware/` | 认证、CORS 等 |
| 启动配置 | `startup/` | 初始化流程 |
| OpenAPI 文档 | `AstraServerGo.openapi.json` | API 规范 |

#### 约定
- **API 约定**：
  - 管理端接口前缀：`/web/*`
  - 客户端接口：无前缀（如 `/:school/:grade/:class`）
  - 认证方式：BasicAuth（用户名 `ElectronClassSchedule` 或 `AstraSchedule`）
  - 响应格式：`status`/`message`/`data` 或 `error`/`detail`
- **数据库约定**：
  - 支持 MySQL 和 SQLite（通过配置切换）
  - 使用 GORM 模型（`model/dbTable/*`）
  - 写入操作使用 upsert（`ON CONFLICT ... UPDATE ALL`）
  - 多表操作使用事务，失败原子回滚
- **配置约定**：
  - 配置文件：`config.toml`（从 `config.template.toml` 复制）
  - 敏感信息：`secret.token` 字段
  - 数据库类型：`db.type` 字段（`"mysql"` 或 `"sqlite"`）

#### 反模式
- **安全相关**：
  - **禁止** 提交 `config.toml`（包含敏感配置）
  - **禁止** 在代码中硬编码密码或令牌
  - **禁止** 禁用 CORS 保护
- **代码质量**：
  - **禁止** 在 `main.go` 中添加新路由（应提取到 `router/` 目录）
  - **禁止** 在 `db/` 中添加业务逻辑（应放在 `service/`）
  - **禁止** 使用 `any` 类型（应使用具体类型）
  - **禁止** 删除失败测试来"通过"
- **数据库相关**：
  - **禁止** 使用原生 SQL（应使用 GORM）
  - **禁止** 跳过事务进行多表操作
  - **禁止** 忽略数据库迁移（AutoMigrate）
- **API 相关**：
  - **禁止** 修改 API 响应格式（破坏性变更）
  - **禁止** 跳过 OpenAPI 文档更新
  - **禁止** 在 GET 请求中进行写操作

#### 独特风格
- **课表规则引擎**：
  - 实现 4 类规则（调休/作息/课表/组合）
  - 按优先级叠加应用：COMPENSATION → TIMETABLE → SCHEDULE → ALL
  - 支持多级作用域：ALL → school → school/grade → school/grade/class
- **Serverless 适配**：
  - 支持 `serverless` 模式（`run.serverless = true`）
  - 禁用 WebSocket（返回 501）
  - 适合云函数部署（如腾讯云 SCF）
  - `/web/statistic` 端点返回 `serverless` 字段，供前端判断是否显示实时数据
- **版本号系统**：
  - 格式：`YYYYMM.D.N`（年月.日.运行序号）
  - 用于 API 缓存（`?version=timestamp` 参数）
  - 支持 304 增量同步
- **泛型批量导入**：
  - `db/backup.go` 使用 Go 泛型 + reflect
  - 支持跨数据库类型迁移（MySQL ↔ SQLite）
  - 失败原子回滚

#### 命令
```bash
# 构建
go build

# 运行
./astra_server.exe

# 测试
go test ./...

# 格式化
go fmt ./...

# 依赖整理
go mod tidy
```

#### 注意事项
- **数据库配置**：
  - SQLite：无需安装，适合开发和小型部署
  - MySQL：需要安装 MySQL 服务器
  - 配置文件：`config.toml`
- **认证机制**：
  - 用户名：`ElectronClassSchedule`（旧版）或 `AstraSchedule`（新版）
  - 密码：从 `config.toml` 的 `secret.token` 获取
  - 只有写操作（PUT/POST/DELETE）需要认证
- **WebSocket**：
  - 地址：`ws://{host}/ws/{school}/{grade}/{class_number}`
  - 消息类型：`SyncConfig`（配置同步广播）
  - 心跳：25 秒间隔 ping
  - 重连：指数退避算法，初始 1 秒，最大 30 秒
- **日志配置**：
  - 库：Logrus
  - 调试模式：`log.debug = true` 时日志级别为 Trace
  - 输出：控制台（开发）或文件（生产）
- **天气 API**：
  - 缓存：10 分钟 TTL
  - 请求库：Resty
  - 数据源：和风天气 API

### sys-backend（系统后端）

#### 项目结构
```
sys-backend/
├── main.go                    # 入口文件
├── config/                    # 配置加载
├── model/                     # 数据模型
│   └── dbTable/               # GORM 表模型
├── db/                        # 数据访问层
├── service/                   # 业务逻辑层
├── middleware/                 # 中间件
├── router/                    # HTTP 处理层
│   ├── client/                # 客户端 API
│   └── web/                   # Web 管理后台 API
│       ├── tenant_handlers.go      # 租户管理
│       ├── database_handlers.go    # 数据库管理
│       ├── system_user_handlers.go # 系统用户管理
│       ├── astra_user_handlers.go  # Astra 用户管理
│       ├── data_handlers.go        # 数据操作
│       ├── health.go               # 健康检查
│       ├── repair_handlers.go      # 修复工具
│       └── auth_handlers.go        # 认证
└── startup/                   # 启动初始化
```

#### 注意事项
- **main 分支即 SaaS 版本**：等效于 usr-backend 的 saas/main 分支
- **分支管理规则同 usr-backend**

### sys-dashboard（系统端 Dashboard）

#### 项目结构
```
sys-dashboard/
├── index.html              # 入口 HTML
├── package.json            # 依赖配置
├── src/
│   ├── main.js             # 应用入口
│   ├── App.vue             # 根组件
│   ├── global.js           # 全局配置（API地址）
│   └── ...
```

#### 注意事项
- **main 分支即 SaaS 版本**：等效于 usr-backend 的 saas/main 分支
- **开发地址**：`src/global.js` 中配置 dev API 地址

### docs-site（文档站点）

#### 项目结构
```
docs-site/
├── index.html              # 入口 HTML
├── package.json            # 依赖配置
├── rspress.config.ts       # Rspress 核心配置
├── tsconfig.json           # TypeScript 配置
├── bun.lock                # Bun 包管理锁文件
└── docs/                   # 文档根目录
    ├── _nav.json           # 顶部导航配置
    ├── index.md            # 首页（hero + features）
    ├── thanks.md           # 致谢页面
    ├── guide/              # 快速开始
    ├── manual/             # 用户手册
    ├── administrator/      # 运维手册
    ├── dev/                # 开发指南
    ├── faq/                # 常见问题
    └── glossary/           # 术语表
```

#### 约定
- **编码规范**：
  - 语言：Markdown + MDX
  - TypeScript：严格模式
  - JSX：react-jsx 转换
  - 模块解析：bundler
- **文档结构**：
  - 每个章节有 `index.md` 作为入口
  - 使用 `_meta.json` 配置侧边栏
  - 使用 `_nav.json` 配置顶部导航
- **版本标记**：
  - **新建文档**：开头添加 DANGER 标识（全文未审核）
  - **修改已有文档**：开头添加 WARNING 标识（部分内容未审核）

#### 反模式
- **内容相关**：
  - **禁止** 添加未审核的技术细节（需人工验证）
  - **禁止** 修改 `rspress.config.ts` 的 `llms: true` 配置
- **代码相关**：
  - **禁止** 使用 `any` 类型（TypeScript 严格模式）
  - **禁止** 未使用的变量或参数
  - **禁止** 提交 `node_modules/` 目录
- **构建相关**：
  - **禁止** 提交 `dist/` 目录（构建产物）
  - **禁止** 修改 `bun.lock`（依赖锁文件）

#### 命令
```bash
# 开发服务器
bun run dev

# 生产构建
bun run build

# 预览构建结果
bun run preview
```

#### 注意事项
- **开发环境**：
  - 需要 Bun 包管理器
  - 使用 Rspress 开发服务器（支持热重载）
  - TypeScript 严格模式
- **文档编写**：
  - 使用 Markdown 语法
  - 支持 MDX（可嵌入 React 组件）
  - 使用 GFM 扩展（表格、任务列表等）
- **部署**：
  - 构建产物：`dist/` 目录
  - 支持静态托管（如 GitHub Pages、Netlify）
  - 无需后端服务
- **搜索**：
  - 使用 FlexSearch（Rspress 内置）
  - 支持全文搜索
  - 无需额外配置
- **图片**：
  - 使用阿里云 OSS 外链
  - 无本地静态资源
  - 支持点击放大（medium-zoom）
- **语法高亮**：
  - 使用 Shiki（Rspress 内置）
  - 支持多种语言
  - 无需额外配置
