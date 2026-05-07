# Chat2API 项目代码文档

## 1. 项目概述

Chat2API 是一个多平台 AI 服务统一管理工具，通过利用官方 Web UI 实现零成本访问领先的 AI 模型。它支持 DeepSeek、GLM、Kimi、MiniMax、Qwen、Z.ai 等提供商，并与 openlcaw、Cline、Roo-Code 等工具无缝集成，使任何 OpenAI 兼容客户端都能开箱即用。

### 核心功能
- OpenAI 兼容 API：提供标准的 OpenAI 兼容 API 端点，实现无缝集成
- 多提供商支持：连接 DeepSeek、GLM、Kimi、MiniMax、Perplexity、Qwen、Z.ai 等
- 上下文管理：智能对话上下文管理，支持滑动窗口、令牌限制和摘要策略
- 函数调用支持：通过提示工程实现所有模型的通用工具调用能力，兼容 Cherry Studio、Kilo Code 等客户端
- 模型映射：灵活的模型名称映射，支持通配符和首选提供商/账户选择
- 自定义参数：支持自定义 HTTP 头，启用网络搜索、思考模式和深度研究功能
- 仪表板监控：实时请求流量、令牌使用和成功率
- API 密钥管理：为本地代理生成和管理密钥
- 模型管理：查看和管理所有提供商的可用模型
- 请求日志：详细的请求日志，用于调试和分析
- 代理配置：灵活的代理设置和路由策略
- 系统托盘集成：从菜单栏快速访问状态
- 多语言支持：英语和简体中文支持
- 现代 UI：清洁、响应式界面，支持明暗主题

## 2. 项目架构

Chat2API 采用 Electron 架构，分为主进程和渲染进程两部分。主进程负责核心功能实现，包括代理服务器、账户管理、模型管理等；渲染进程负责用户界面展示。

### 架构图

```
Chat2API/
├── src/
│   ├── main/                    # Electron 主进程
│   │   ├── index.ts            # 应用入口点
│   │   ├── tray.ts             # 系统托盘集成
│   │   ├── proxy/              # 代理服务器管理
│   │   ├── ipc/                # IPC 处理器
│   │   ├── store/              # 数据存储
│   │   ├── oauth/              # OAuth 认证
│   │   ├── providers/          # 提供商管理
│   │   └── utils/              # 工具函数
│   ├── preload/                # 上下文桥接
│   └── renderer/               # React 前端
│       ├── components/         # UI 组件
│       ├── pages/              # 页面组件
│       ├── stores/             # Zustand 状态管理
│       └── hooks/              # 自定义钩子
├── build/                      # 构建资源
└── scripts/                    # 构建脚本
```

## 3. 主要模块职责

### 3.1 主进程 (Main Process)

#### 3.1.1 应用核心 (src/main/index.ts)
- 应用启动和初始化
- 窗口管理
- 系统托盘集成
- 异常处理

#### 3.1.2 代理服务 (src/main/proxy/)
- 提供 OpenAI 兼容的 API 端点
- 管理请求路由和转发
- 实现负载均衡策略
- 模型映射和转换
- 会话管理

#### 3.1.3 数据存储 (src/main/store/)
- 管理应用配置
- 存储提供商和账户信息
- 日志管理

#### 3.1.4 OAuth 认证 (src/main/oauth/)
- 处理不同提供商的认证流程
- 令牌提取和管理

#### 3.1.5 提供商管理 (src/main/providers/)
- 内置提供商支持
- 自定义提供商配置
- 提供商状态检查

### 3.2 渲染进程 (Renderer Process)

#### 3.2.1 界面组件 (src/renderer/src/components/)
- 仪表板组件
- 提供商管理组件
- 代理设置组件
- 模型管理组件
- 日志查看组件

#### 3.2.2 页面 (src/renderer/src/pages/)
- 仪表板页面
- 提供商页面
- 代理设置页面
- 模型页面
- API 密钥页面
- 日志页面
- 设置页面

#### 3.2.3 状态管理 (src/renderer/src/stores/)
- 仪表板状态
- 提供商状态
- 代理状态
- 日志状态
- 设置状态

## 4. 关键类与函数

### 4.1 主进程核心类

#### 4.1.1 ProxyServer (src/main/proxy/server.ts)
- **职责**：实现基于 Koa 的代理服务器
- **主要方法**：
  - `start(port, host)`：启动代理服务器
  - `stop()`：停止代理服务器
  - `restart(port, host)`：重启代理服务器
  - `isRunning()`：检查服务器是否运行
  - `getStatistics()`：获取服务器统计信息

#### 4.1.2 LoadBalancer (src/main/proxy/loadbalancer.ts)
- **职责**：实现负载均衡策略
- **主要方法**：
  - `selectAccount(model, strategy, preferredProviderId, preferredAccountId)`：选择合适的账户
  - `markAccountFailed(accountId)`：标记账户失败
  - `clearAccountFailure(accountId)`：清除账户失败状态
  - `getAvailableAccounts(model, preferredProviderId, excludeFailed)`：获取可用账户列表

#### 4.1.3 ModelMapper (src/main/proxy/modelMapper.ts)
- **职责**：支持请求模型到实际模型的映射
- **主要方法**：
  - `mapModel(requestedModel, provider)`：映射模型名称
  - `getActualModel(requestedModel, providerId)`：根据提供商 ID 获取实际模型名称
  - `getAllMappings()`：获取所有已配置的模型映射规则

> **个人笔记**：模型映射支持通配符（如 `gpt-4*` → `deepseek-chat`），在配置多账户轮询时非常有用。建议优先在此处统一管理模型别名，避免在客户端侧硬编码模型名。

## 5. 开发备忘

- 本 fork 主要用于个人学习，重点关注 `proxy/` 和 `providers/` 模块的实现细节
- 如需添加新提供商，参考 `src/main/providers/` 下已有实现作为模板
- 负载均衡策略枚举定义在 `loadbalancer.ts` 中，目前支持 round-robin 和 random 两种模式
