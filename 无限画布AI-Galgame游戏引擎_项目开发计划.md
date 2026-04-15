# 无限画布 AI-Galgame 游戏引擎 — 项目开发计划

> 编制日期：2026-04-15
> 版本：V1.0
> 状态：规划阶段

---

## 一、项目概述

### 1.1 项目定位

本项目是一个基于无限画布的 AI-Galgame 游戏引擎，将画布编辑器与游戏运行合二为一。创作者可在无限画布上通过拖拽场景卡片、连接剧情线来设计游戏，玩家通过自由打字与 AI 角色实时对话，AI 根据对话内容判断玩家行为并决定剧情走向，彻底告别传统 Galgame 的"固定选项"模式。

### 1.2 核心创新点

1. 画布 = 编辑器 + 游戏：设计完点"运行"直接在画布上玩
2. 自由打字代替固定选项：玩家可输入任何内容，AI 实时回应
3. AI 决定剧情走向：硬性通关 / 软性通关 / AI 自主判断三种模式
4. 角色有成长有状态：版本卡（初期/中期/后期）+ 场景专属卡
5. 游戏文件分享系统：.agalgame 文件格式一键导出/导入

### 1.3 项目目标

| 阶段 | 目标 | 用户群体 |
|------|------|---------|
| MVP | 最小可玩原型，验证核心玩法 | 开发者自测 |
| V1.x | 功能完善，支持完整游戏创作流程 | 普通人零基础创作 |
| V2.0 | 创作者社区，商业级游戏发布 | 独立开发者 |

---

## 二、技术栈选型

### 2.1 技术栈总览

| 层级 | 技术选择 | 选择理由 |
|------|---------|---------|
| 桌面外壳 | **Tauri 2.x** | 包体小（~5MB vs Electron ~150MB）、Rust 后端高性能、原生文件系统访问、安全性高 |
| 前端框架 | **React 19 + TypeScript** | 生态最大、AI 编程友好、xyflow 原生支持、社区资源丰富 |
| 画布引擎 | **@xyflow/react (React Flow)** | 专为节点式编辑器设计、原生支持拖拽/缩放/连线、自定义节点、开箱即用 |
| UI 组件库 | **shadcn/ui + Tailwind CSS 4** | 高度可定制、现代设计、不依赖重型框架、按需引入 |
| 状态管理 | **Zustand** | 轻量、TypeScript 友好、无 boilerplate、支持持久化中间件 |
| 后端服务 | **Tauri Rust 后端 + Node.js 辅助服务** | Rust 处理文件系统/加密等核心逻辑，Node.js 处理 AI API 代理（SSE 流式） |
| AI 对接 | **Claude API + OpenAI API + 国产模型** | 多模型支持、角色独立选模型、SSE 流式输出 |
| 数据库 | **SQLite（via Tauri sql plugin）** | 轻量嵌入式、本地优先、事务支持、查询能力强 |
| 本地缓存 | **IndexedDB** | 大型媒体资源缓存、二进制数据存储 |
| 构建 | **Vite 6 + Tauri CLI** | 极速 HMR、成熟稳定 |
| 测试 | **Vitest + Playwright + Testing Library** | 单元测试 + E2E 测试 + 组件测试 |

### 2.2 关键技术选型详细论证

#### 2.2.1 画布引擎：@xyflow/react vs Fabric.js vs Konva.js

| 维度 | @xyflow/react | Fabric.js | Konva.js |
|------|-------------|-----------|----------|
| 核心定位 | 节点式编辑器 | 通用图形编辑 | 通用 2D Canvas |
| 无限画布 | 内建支持 | 需手动实现 | 需手动实现 |
| 节点连线 | 内建 Edge 系统 | 需完全自建 | 需完全自建 |
| 拖拽/缩放 | 内建支持 | 内建支持 | 内建支持 |
| React 集成 | 原生 React 组件 | 需包装层 | react-konva 桥接 |
| 自定义节点 | 声明式 React 组件 | 命令式 API | 命令式 API |
| 社区 Star | 28k+ | 31k | 14k |
| 学习曲线 | 低（声明式） | 中 | 中 |
| 性能 | 中（DOM 混合） | 高（纯 Canvas） | 高（纯 Canvas） |

**选择理由**：本项目核心场景是"节点式编辑器"——场景卡片作为节点、剧情线作为连线，这与 @xyflow/react 的核心定位完全一致。它开箱即用地提供了拖拽、缩放、连线、自定义节点等能力，大幅降低开发成本。纯 Canvas 库（Fabric.js/Konva.js）虽然渲染性能更高，但需要从零构建节点/连线/交互系统，开发成本巨大。当场景节点数量在百级以内时，@xyflow/react 的 DOM 混合渲染性能完全满足需求。

#### 2.2.2 桌面框架：Tauri vs Electron

| 维度 | Tauri 2.x | Electron |
|------|----------|----------|
| 包体大小 | ~5-10 MB | ~150-200 MB |
| 内存占用 | 低（系统 WebView） | 高（内嵌 Chromium） |
| 启动速度 | 快 | 较慢 |
| 文件系统 | Rust 原生访问 | Node.js fs |
| 安全性 | Rust 安全模型 | 较宽松 |
| 生态成熟度 | 快速增长中 | 非常成熟 |
| AI API 调用 | 需 sidecar 或 HTTP | 直接 Node.js |
| 学习成本 | Rust 学习曲线 | 纯 JS 生态 |

**选择理由**：本项目的核心用户是普通人，包体大小直接影响下载意愿。Tauri 5MB vs Electron 150MB 的差距是决定性的。Tauri 2.x 已支持 sidecar（外部进程），可用于运行 Node.js AI 代理服务。Rust 后端处理文件系统操作更安全高效。考虑到 AI API 调用的需求，采用"Tauri 主进程 + Node.js sidecar"的混合架构。

#### 2.2.3 AI 模型集成方案

```
┌─────────────────────────────────────────────────────────────┐
│                    AI 模型集成架构                              │
│                                                              │
│   ┌──────────────┐     SSE/Stream     ┌──────────────────┐ │
│   │  Node.js     │ ◄──────────────── │ Claude API        │ │
│   │  AI Gateway  │                    │ OpenAI API        │ │
│   │  (sidecar)   │ ◄──────────────── │ DeepSeek API      │ │
│   │              │                    │ 通义千问 API       │ │
│   └──────┬───────┘                    └──────────────────┘ │
│          │ WebSocket/SSE                                  │
│   ┌──────▼───────┐                                          │
│   │  Tauri 前端   │                                          │
│   │  (React)     │                                          │
│   └──────────────┘                                          │
└─────────────────────────────────────────────────────────────┘
```

核心设计：
- **AI Gateway 模式**：Node.js sidecar 作为统一 AI 网关，屏蔽不同 API 的差异
- **SSE 流式传输**：AI API → Node.js (SSE) → Tauri 前端 (EventSource/自定义协议)
- **多模型路由**：每个角色独立配置模型，Gateway 根据角色设定路由到对应 API
- **上下文窗口管理**：Gateway 层实现对话历史的滑动窗口裁剪，控制 Token 消耗
- **Prompt 模板系统**：角色卡 → 系统 Prompt 模板 → 注入对话历史 → 调用模型

### 2.3 技术栈风险与备选方案

| 风险点 | 影响 | 备选方案 |
|--------|------|---------|
| @xyflow/react 大规模节点性能 | 超过 200+ 节点可能卡顿 | 虚拟化渲染 + 视口裁剪，或回退 Fabric.js 自建 |
| Tauri sidecar 配置复杂 | 部署和调试成本增加 | 改用 Tauri HTTP 插件直接调用 AI API |
| Rust 后端开发效率 | 前端开发者学习曲线 | 核心逻辑用 TypeScript + Tauri shell 插件 |
| AI API 调用延迟 | 对话体验差 | 本地模型 Ollama 备选 + 流式首 Token 优化 |

---

## 三、系统架构设计

### 3.1 整体架构图

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Tauri 桌面应用壳                               │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │                    React 前端应用                                │ │
│  │                                                                │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐ │ │
│  │  │  画布编辑器    │  │  游戏运行器    │  │  资源/设置管理器      │ │ │
│  │  │  (xyflow)    │  │  (Game Runner)│  │  (Asset Manager)    │ │ │
│  │  └──────┬───────┘  └──────┬───────┘  └──────────┬───────────┘ │ │
│  │         │                 │                      │             │ │
│  │  ┌──────▼─────────────────▼──────────────────────▼───────────┐ │ │
│  │  │              Zustand 状态管理层                              │ │ │
│  │  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐    │ │ │
│  │  │  │ 游戏状态  │ │ 角色状态  │ │ 场景状态  │ │ 对话状态  │    │ │ │
│  │  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘    │ │ │
│  │  └────────────────────────┬──────────────────────────────────┘ │ │
│  └───────────────────────────┼────────────────────────────────────┘ │
│                              │ Tauri IPC / Invoke                   │
│  ┌───────────────────────────▼────────────────────────────────────┐ │
│  │                    Tauri Rust 后端                               │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐     │ │
│  │  │ 文件系统  │  │ 数据库    │  │ 加密/压缩 │  │ 窗口管理  │     │ │
│  │  │ (fs)     │  │ (SQLite) │  │ (zip/aes)│  │ (window) │     │ │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘     │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                              │ Sidecar (本地进程)                    │
│  ┌───────────────────────────▼────────────────────────────────────┐ │
│  │                  Node.js AI Gateway (sidecar)                    │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐     │ │
│  │  │ API 路由  │  │ Prompt   │  │ 上下文    │  │ 流式转发  │     │ │
│  │  │ (Router) │  │ (Template)│  │ (Context) │  │ (Stream) │     │ │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘     │ │
│  └───────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
                              │ HTTPS
               ┌──────────────▼──────────────┐
               │     外部 AI API 服务          │
               │  Claude / OpenAI / DeepSeek  │
               └─────────────────────────────┘
```

### 3.2 模块划分

```
src/
├── main/                          # Tauri Rust 后端
│   ├── commands/                  # Tauri IPC 命令
│   │   ├── file_ops.rs           # 文件操作（导入/导出/保存）
│   │   ├── db_ops.rs             # 数据库操作
│   │   └── app_ops.rs            # 应用管理
│   ├── db/                        # 数据库层
│   │   ├── schema.rs             # 表结构定义
│   │   ├── migrations.rs         # 数据库迁移
│   │   └── repository.rs         # 数据访问层
│   ├── models/                    # Rust 数据模型
│   ├── services/                  # 业务逻辑
│   │   ├── game_pack.rs          # .agalgame 文件打包/解包
│   │   ├── asset_manager.rs      # 资源管理
│   │   └── save_manager.rs       # 存档管理
│   └── lib.rs                     # 入口
│
├── sidecar/                       # Node.js AI Gateway
│   ├── src/
│   │   ├── gateway.ts            # 网关主入口
│   │   ├── routers/              # API 路由
│   │   │   ├── claude.ts
│   │   │   ├── openai.ts
│   │   │   └── deepseek.ts
│   │   ├── services/
│   │   │   ├── prompt-builder.ts # Prompt 模板构建
│   │   │   ├── context-manager.ts# 上下文窗口管理
│   │   │   ├── stream-handler.ts # 流式响应处理
│   │   │   └── judge-engine.ts   # 通关判定引擎
│   │   └── types/
│   └── package.json
│
├── renderer/                      # React 前端
│   ├── src/
│   │   ├── app/                   # 应用入口与路由
│   │   ├── components/            # 通用 UI 组件
│   │   │   ├── ui/               # shadcn/ui 基础组件
│   │   │   ├── canvas/           # 画布相关组件
│   │   │   │   ├── SceneNode.tsx    # 场景卡片节点
│   │   │   │   ├── BranchNode.tsx   # 分支点节点
│   │   │   │   ├── PlotEdge.tsx     # 剧情连线
│   │   │   │   └── CanvasToolbar.tsx# 画布工具栏
│   │   │   ├── editor/           # 编辑面板
│   │   │   │   ├── SceneEditor.tsx  # 场景编辑器
│   │   │   │   ├── CharacterEditor.tsx # 角色编辑器
│   │   │   │   └── TriggerEditor.tsx  # 触发器编辑器
│   │   │   ├── game/             # 游戏运行组件
│   │   │   │   ├── GameRunner.tsx    # 游戏运行器
│   │   │   │   ├── ChatWindow.tsx    # 对话窗口
│   │   │   │   ├── SceneView.tsx     # 场景视觉区
│   │   │   │   └── TransitionScreen.tsx # 场景过渡
│   │   │   └── common/           # 公共组件
│   │   ├── stores/                # Zustand 状态
│   │   │   ├── gameStore.ts      # 游戏全局状态
│   │   │   ├── canvasStore.ts    # 画布状态
│   │   │   ├── characterStore.ts # 角色状态
│   │   │   ├── sceneStore.ts     # 场景状态
│   │   │   ├── dialogueStore.ts  # 对话状态
│   │   │   └── settingsStore.ts  # 设置状态
│   │   ├── services/              # 前端服务层
│   │   │   ├── ai-client.ts      # AI API 客户端
│   │   │   ├── tauri-bridge.ts   # Tauri IPC 桥接
│   │   │   └── audio-player.ts   # 音频播放服务
│   │   ├── hooks/                 # 自定义 Hooks
│   │   ├── types/                 # TypeScript 类型定义
│   │   └── utils/                 # 工具函数
│   └── index.html
│
└── shared/                        # 前后端共享
    └── types/                     # 共享类型定义
```

### 3.3 核心数据流设计

#### 3.3.1 编辑器模式数据流

```
用户操作（拖拽/编辑）
    │
    ▼
React 组件（xyflow 自定义节点/编辑面板）
    │
    ▼
Zustand Store 更新（乐观更新）
    │
    ├─→ 画布视图即时响应
    │
    └─→ 防抖保存（500ms）
         │
         ▼
    Tauri IPC (invoke)
         │
         ▼
    Rust 后端 → SQLite 持久化
```

#### 3.3.2 游戏运行模式数据流

```
玩家输入文字
    │
    ▼
React 对话组件
    │
    ▼
dialogueStore（添加玩家消息）
    │
    ▼
AI Client (SSE/Stream)
    │
    ▼
Node.js AI Gateway
    │
    ├──→ 构建 Prompt（角色设定 + 对话历史 + 场景信息）
    ├──→ 上下文窗口裁剪（保留最近 N 轮 + 关键记忆）
    └──→ 调用对应 AI API（流式）
         │
         ▼
    流式 Token 返回
         │
         ▼
    dialogueStore（逐字更新 AI 回复）
         │
         ├─→ 对话界面打字机效果
         │
         └─→ AI 回复完成
              │
              ├──→ 通关判定引擎检查
              │     ├── 硬性：关键词匹配
              │     ├── 软性：AI 判断（二次调用）
              │     └── 数值：好感度检查
              │
              ├──→ 好感度/状态更新
              │
              └──→ 场景切换判定
                    │
                    ▼
              sceneStore（切换场景/显示通关画面）
```

#### 3.3.3 游戏文件导入/导出数据流

```
导出流程：
  Zustand Stores → 序列化为 JSON → 收集资源文件路径
      → Rust 后端 → ZIP 打包（JSON + 资源） → .agalgame 文件

导入流程：
  .agalgame 文件 → Rust 后端 → ZIP 解包
      → 解析 JSON → 写入 SQLite → 复制资源到本地目录
      → 初始化 Zustand Stores → 渲染画布
```

### 3.4 数据库设计（SQLite）

```sql
-- 游戏项目表
CREATE TABLE games (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    description TEXT,
    version INTEGER DEFAULT 1,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- 角色卡表
CREATE TABLE characters (
    id TEXT PRIMARY KEY,
    game_id TEXT NOT NULL REFERENCES games(id),
    name TEXT NOT NULL,
    appearance TEXT,
    outfit TEXT,
    personality TEXT,
    hobbies TEXT,
    speech_style TEXT,
    backstory TEXT,
    ai_model TEXT DEFAULT 'claude-3.5-sonnet',
    avatar_path TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- 角色版本卡表
CREATE TABLE character_versions (
    id TEXT PRIMARY KEY,
    character_id TEXT NOT NULL REFERENCES characters(id),
    version_name TEXT NOT NULL,  -- 初期/中期/后期
    personality_override TEXT,
    speech_style_override TEXT,
    backstory_override TEXT,
    sort_order INTEGER DEFAULT 0
);

-- 场景专属卡表
CREATE TABLE character_scene_states (
    id TEXT PRIMARY KEY,
    character_id TEXT NOT NULL REFERENCES characters(id),
    scene_id TEXT NOT NULL,
    state_name TEXT NOT NULL,  -- 生气/开心/疲惫
    personality_override TEXT,
    speech_style_override TEXT
);

-- 场景卡表
CREATE TABLE scenes (
    id TEXT PRIMARY KEY,
    game_id TEXT NOT NULL REFERENCES games(id),
    name TEXT NOT NULL,
    description TEXT,
    purpose TEXT,
    time_slot TEXT,  -- 早上/中午/下午/晚上/睡觉
    location TEXT,
    bgm_path TEXT,
    background_path TEXT,
    start_screen_type TEXT DEFAULT 'image',
    start_screen_path TEXT,
    end_screen_type TEXT DEFAULT 'image',
    end_screen_path TEXT,
    canvas_x REAL DEFAULT 0,
    canvas_y REAL DEFAULT 0,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- 场景-角色关联表
CREATE TABLE scene_characters (
    id TEXT PRIMARY KEY,
    scene_id TEXT NOT NULL REFERENCES scenes(id),
    character_id TEXT NOT NULL REFERENCES characters(id),
    character_version_id TEXT REFERENCES character_versions(id),
    is_lead INTEGER DEFAULT 0,  -- 是否主导角色
    dialogue_mode TEXT DEFAULT 'hybrid'  -- lead/ai_judge/hybrid
);

-- 通关条件表
CREATE TABLE clear_conditions (
    id TEXT PRIMARY KEY,
    scene_id TEXT NOT NULL REFERENCES scenes(id),
    type TEXT NOT NULL,  -- hard/soft/ai_judge
    keyword TEXT,  -- 硬性通关关键词
    description TEXT,  -- 软性通关描述
    target_plotline_id TEXT,  -- 导向剧情线
    priority INTEGER DEFAULT 0
);

-- 触发器表
CREATE TABLE triggers (
    id TEXT PRIMARY KEY,
    scene_id TEXT NOT NULL REFERENCES scenes(id),
    condition_type TEXT NOT NULL,  -- time/location/affection/flag
    condition_value TEXT NOT NULL,
    operator TEXT DEFAULT 'equals'  -- equals/gt/lt/contains
);

-- 剧情线表
CREATE TABLE plotlines (
    id TEXT PRIMARY KEY,
    game_id TEXT NOT NULL REFERENCES games(id),
    name TEXT NOT NULL,
    description TEXT,
    is_main INTEGER DEFAULT 0
);

-- 场景连接表（画布上的边）
CREATE TABLE scene_connections (
    id TEXT PRIMARY KEY,
    source_scene_id TEXT NOT NULL REFERENCES scenes(id),
    target_scene_id TEXT NOT NULL REFERENCES scenes(id),
    plotline_id TEXT REFERENCES plotlines(id),
    condition_type TEXT,  -- auto/choice/condition
    condition_description TEXT,
    edge_type TEXT DEFAULT 'linear'  -- linear/branch/merge
);

-- 玩家存档表
CREATE TABLE saves (
    id TEXT PRIMARY KEY,
    game_id TEXT NOT NULL REFERENCES games(id),
    save_name TEXT,
    current_plotline_id TEXT,
    current_scene_id TEXT,
    game_time TEXT,
    game_location TEXT,
    game_day INTEGER DEFAULT 1,
    progress_data TEXT,  -- JSON: visited_scenes, choices, etc.
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- 存档-角色状态表
CREATE TABLE save_character_states (
    id TEXT PRIMARY KEY,
    save_id TEXT NOT NULL REFERENCES saves(id),
    character_id TEXT NOT NULL,
    current_version_id TEXT,
    affection INTEGER DEFAULT 50,
    flags TEXT  -- JSON: 自定义标记
);
```

---

## 四、前端 UI/UX 设计规范

### 4.1 设计理念

- **沉浸感优先**：游戏运行时全屏暗色主题，消除界面干扰
- **编辑器高效**：画布编辑时信息密度高，操作路径短
- **GBA 怀旧美学**：对话界面致敬 GBA 时代视觉小说风格
- **现代感融合**：编辑器采用现代 Figma 式设计语言

### 4.2 色彩体系

```
暗色主题（主色调）：
├── 背景色：#0F0F14（深蓝黑）
├── 卡片色：#1A1A24（暗蓝灰）
├── 边框色：#2A2A3A（中蓝灰）
├── 文字主色：#E8E8F0（亮灰白）
├── 文字次色：#8888A0（灰蓝）
├── 强调色-主：#6C5CE7（紫色 — 品牌色）
├── 强调色-辅：#00CEC9（青色 — 通关/成功）
├── 警告色：#FDCB6E（金黄）
├── 危险色：#FF6B6B（红色 — 删除/错误）
└── 角色标识色：#FF6B9D / #48DBFB / #FECA57 / #1DD1A1

亮色主题（编辑器可选）：
├── 背景色：#F5F5FA
├── 卡片色：#FFFFFF
├── 边框色：#E0E0EE
└── 其余同暗色主题
```

### 4.3 字体规范

```
中文正文：思源黑体 (Noto Sans SC) / 系统默认
  ├── 正文：14px / 行高 1.6
  ├── 小字：12px
  └── 大标题：20px / Bold

对话文本：霞鹜文楷 (LXGW WenKai) / 圆体
  ├── 角色对话：16px / 行高 1.8
  └── 系统提示：14px / Italic

英文/代码：JetBrains Mono
  └── 代码/配置：13px
```

### 4.4 布局规范

#### 4.4.1 画布编辑器布局

```
┌─────────────────────────────────────────────────────────────┐
│  顶部工具栏 (h-12)                                            │
│  [Logo] [新建][打开][保存] | 游戏名 | [编辑器/游戏切换] | [设置] │
├────────┬────────────────────────────────────┬───────────────┤
│ 左侧栏  │          无限画布区域                 │   右侧面板      │
│ (w-56) │                                    │  (w-80, 可折叠)│
│        │    ┌─────┐    ┌─────┐              │               │
│ 世界系统 │    │场景A │───▶│分支  │              │  场景/角色     │
│ 角色库  │    └─────┘    └─────┘              │  编辑面板      │
│ 剧情线  │         │                          │               │
│        │    ┌─────▼───┐                      │               │
│        │    │ 场景B   │                      │               │
│        │    └─────────┘                      │               │
├────────┴────────────────────────────────────┴───────────────┤
│  底部状态栏 (h-8)                                            │
│  缩放: 100% | 场景: 12 | 角色: 5 | 已保存                     │
└─────────────────────────────────────────────────────────────┘
```

#### 4.4.2 游戏运行布局（GBA 左图右聊）

```
┌─────────────────────────────────────────────────────────────┐
│  全屏暗色背景                                                 │
│                                                              │
│  ┌─────────────────────────────┬──────────────────────────┐│
│  │                             │                          ││
│  │   场景视觉区 (2/3)           │   对话窗口 (1/3)           ││
│  │                             │                          ││
│  │   ┌─────────────────────┐  │   ┌──┬────────────────┐ ││
│  │   │                     │  │   │雨│ 低头沉默着...    │ ││
│  │   │  背景图              │  │   └──┴────────────────┘ ││
│  │   │                     │  │                          ││
│  │   │  ┌───┐  ┌───┐     │  │   ┌──┬────────────────┐ ││
│  │   │  │小雨│  │小美│     │  │   │美│ 你找我什么事？  │ ││
│  │   │  │立绘│  │立绘│     │  │   └──┴────────────────┘ ││
│  │   │  └───┘  └───┘     │  │                          ││
│  │   │                     │  │   ┌──┬────────────────┐ ││
│  │   └─────────────────────┘  │   │我│ 你好            │ ││
│  │                             │   └──┴────────────────┘ ││
│  ├─────────────────────────────┼──────────────────────────┤│
│  │ ⏰ 晚上 │ 📍 咖啡厅 │ [跳转] │   💬 请输入对话...      ││
│  └─────────────────────────────┴──────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

### 4.5 交互规范

| 操作 | 触发方式 | 反馈 |
|------|---------|------|
| 画布平移 | 空格+拖拽 / 鼠标中键拖拽 | 平滑移动 |
| 画布缩放 | 滚轮 / Ctrl+滚轮 | 平滑缩放，10%~200% |
| 创建场景 | 双击画布空白处 / 工具栏按钮 | 在点击位置创建节点 |
| 编辑场景 | 双击场景节点 | 右侧面板打开编辑器 |
| 连接场景 | 从节点边缘拖拽到另一节点 | 创建带箭头连线 |
| 拖入角色 | 从左侧角色库拖入场景编辑器 | 角色添加到场景 |
| 游戏对话 | 回车发送 / 点击发送按钮 | 打字机效果显示 AI 回复 |
| 场景通关 | 自动触发 / 玩家确认 | 过渡动画 + 下一场景 |

### 4.6 动效规范

```
通用：
├── 过渡时长：150ms（快捷操作）/ 300ms（页面切换）
├── 缓动函数：ease-out（进入）/ ease-in（退出）
└── 减弱动画：尊重 prefers-reduced-motion

游戏特有：
├── 打字机效果：30-50ms/字，句末停顿 200ms
├── 场景过渡：淡入淡出 500ms / 滑动 300ms
├── 角色立绘：呼吸动画（微缩放 0.98-1.02，3s 周期）
├── BGM 切换：1s 交叉淡入淡出
└── 通关特效：粒子散落 + 光晕扩散 800ms
```

---

## 五、开发阶段划分

### 5.1 整体时间规划

```
总工期：约 6 个月（26 周）

Phase 0  基础设施搭建     2周   ████
Phase 1  MVP 核心开发      8周   ████████████████
Phase 2  MVP 测试与打磨    2周   ████
Phase 3  V1.1 时间地点系统  3周   ██████
Phase 4  V1.2 多人对话系统  3周   ██████
Phase 5  V1.3 角色成长系统  2周   ████
Phase 6  V1.4 多通关与分支  3周   ██████
Phase 7  V1.5 存档与完善    3周   ██████
```

### 5.2 Phase 0：基础设施搭建（第 1-2 周）

**目标**：搭建项目骨架，所有基础设施就绪。

| 任务 | 交付物 | 验收标准 |
|------|--------|---------|
| 初始化 Tauri + React + Vite 项目 | 可运行的空项目 | `npm run tauri dev` 启动成功 |
| 配置 TypeScript、ESLint、Prettier | 配置文件 | Lint 通过、格式统一 |
| 集成 shadcn/ui + Tailwind CSS | 基础 UI 组件库 | 可渲染 Button/Input/Card |
| 集成 @xyflow/react | 画布 Demo | 可显示基础节点和连线 |
| 配置 Zustand + 持久化中间件 | Store 骨架 | 状态更新 + 本地持久化正常 |
| 搭建 Node.js sidecar 骨架 | AI Gateway 基础 | 可启动并响应健康检查 |
| 配置 SQLite (Tauri sql plugin) | 数据库初始化 | 建表脚本执行成功 |
| 设计共享类型系统 | TypeScript 类型定义 | 前后端类型一致 |
| 配置 Vitest 测试框架 | 测试配置 | 示例测试通过 |
| Git 仓库初始化 + 分支策略 | .gitignore + CI 配置 | 代码可提交、CI 运行 |

### 5.3 Phase 1：MVP 核心开发（第 3-10 周）

#### 第 3-4 周：画布编辑器基础

| 任务 | 交付物 | 验收标准 |
|------|--------|---------|
| 自定义场景节点组件 | SceneNode.tsx | 可渲染场景名、参与者、简介 |
| 场景节点拖拽和定位 | 节点拖拽功能 | 拖拽后位置保存 |
| 画布缩放和平移 | 缩放/平移功能 | 10%-200% 缩放，空格抓手 |
| 场景连线（Edge） | PlotEdge.tsx | 可连接两个场景节点 |
| 双击打开编辑面板 | 场景编辑面板 | 双击节点右侧弹出编辑器 |
| 右键菜单 | 上下文菜单 | 复制/删除/粘贴 |
| 画布工具栏 | CanvasToolbar.tsx | 缩放/适配/全屏按钮 |

#### 第 5-6 周：角色卡与场景卡编辑

| 任务 | 交付物 | 验收标准 |
|------|--------|---------|
| 角色卡数据模型 + Store | characterStore.ts | CRUD 操作正常 |
| 角色卡编辑面板 | CharacterEditor.tsx | 可填写所有基础属性 |
| 角色卡列表（左侧栏） | 角色库组件 | 可创建/编辑/删除角色 |
| AI 模型选择器 | 模型选择下拉 | 可选择 Claude/GPT/DeepSeek |
| 场景卡编辑面板 | SceneEditor.tsx | 可编辑场景名/简介/目的 |
| 角色拖入场景 | 拖拽交互 | 从角色库拖入场景参与角色列表 |
| 背景图/BGM 上传 | 资源上传功能 | 文件选择后保存到本地 |
| 数据持久化 | SQLite 写入 | 编辑内容刷新后保留 |

#### 第 7-8 周：游戏运行器与 AI 对话

| 任务 | 交付物 | 验收标准 |
|------|--------|---------|
| 编辑器/游戏模式切换 | 模式切换按钮 | 平滑切换两种界面 |
| GBA 左图右聊布局 | GameRunner.tsx | 2/3 场景 + 1/3 对话 |
| AI Gateway SSE 集成 | ai-client.ts | 可流式接收 AI 回复 |
| Prompt 模板构建 | prompt-builder.ts | 角色设定 → 系统提示词 |
| 上下文窗口管理 | context-manager.ts | 裁剪对话历史到 N 轮 |
| 对话界面交互 | ChatWindow.tsx | 打字机效果、消息列表 |
| 硬性通关判定 | 关键词匹配引擎 | 说出关键词触发通关 |
| 场景开始/结束画面 | TransitionScreen.tsx | 图片展示 → 对话 → 通关画面 |
| 线性场景切换 | 场景跳转逻辑 | 通关后自动进入下一场景 |

#### 第 9-10 周：文件系统与 MVP 集成

| 任务 | 交付物 | 验收标准 |
|------|--------|---------|
| .agalgame 文件格式定义 | 文件结构规范 | JSON + 资源文件的 ZIP 包 |
| 导出游戏功能 | game_pack.rs | 生成 .agalgame 文件 |
| 导入游戏功能 | game_unpack.rs | 导入 .agalgame 并加载画布 |
| 新建/保存/另存为 | 文件操作功能 | 游戏项目 CRUD 完整 |
| 编辑器 ↔ 运行器状态同步 | 状态桥接 | 切换模式数据不丢失 |
| 音频播放 | audio-player.ts | BGM 播放/暂停/切换 |
| MVP 集成测试 | 端到端流程 | 创建→编辑→运行→通关→导出 |
| MVP Bug 修复 | 稳定版本 | 无 P0/P1 级 Bug |

### 5.4 Phase 2：MVP 测试与打磨（第 11-12 周）

| 任务 | 交付物 | 验收标准 |
|------|--------|---------|
| 端到端测试编写 | Playwright 测试用例 | 覆盖核心流程 |
| 性能优化 | 性能报告 | 画布 50 节点流畅 |
| UI 打磨 | 视觉走查报告 | 符合设计规范 |
| 用户测试（自测） | 测试报告 | 完整游戏流程可用 |
| 文档编写 | 用户手册 + 开发文档 | 文档完整可读 |
| MVP 发布 | V1.0.0 安装包 | 可分发安装 |

### 5.5 Phase 3：V1.1 时间地点系统（第 13-15 周）

| 任务 | 交付物 | 验收标准 |
|------|--------|---------|
| 时间维度系统 | 时间状态管理 | 早上→中午→下午→晚上→睡觉 |
| 地点系统 | 地点列表和切换 | 多地点自由移动 |
| 触发器引擎 | 条件判定引擎 | 时间+地点+好感度组合触发 |
| 时间/地点 UI | 时间地点选择器 | 游戏界面底部显示和切换 |
| 场景按时间地点筛选 | 筛选功能 | 只显示可用场景 |

### 5.6 Phase 4：V1.2 多人对话系统（第 16-18 周）

| 任务 | 交付物 | 验收标准 |
|------|--------|---------|
| 多角色对话协调 | 对话协调器 | 主导/AI判断/混合模式 |
| 相关性判断策略 | AI 判断服务 | 分析玩家输入决定谁回复 |
| 角色对话显示 | 多角色消息 | 不同角色头像+名字区分 |
| 未参与角色标记 | [未参与本轮] | 不说话角色显示状态 |
| 角色立绘叠加 | 立绘渲染 | CSS 分层叠加在背景上 |

### 5.7 Phase 5：V1.3 角色成长系统（第 19-20 周）

| 任务 | 交付物 | 验收标准 |
|------|--------|---------|
| 角色版本卡 | 版本卡 CRUD | 初期/中期/后期卡管理 |
| 场景专属卡 | 状态卡 CRUD | 场景-状态绑定 |
| 版本卡选择 | 场景角色配置 | 拖拽指定版本到场景 |
| 好感度系统 | 好感度追踪 | AI 判断好感度变化 |

### 5.8 Phase 6：V1.4 多通关与分支（第 21-23 周）

| 任务 | 交付物 | 验收标准 |
|------|--------|---------|
| 软性通关判定 | AI 判断引擎 | 描述目标 → AI 判断达成 |
| AI 自主通关 | 自主判断模式 | AI 完全决定走向 |
| 分支点节点 | BranchNode.tsx | 画布上可视化分支 |
| 分支选择界面 | 选择 UI | 通关后弹出选项 |
| 条件解锁分支 | 条件判定 | 好感度/标记解锁选项 |
| 多通关分支逻辑 | 分支路由 | 不同通关→不同剧情线 |

### 5.9 Phase 7：V1.5 存档与完善（第 24-26 周）

| 任务 | 交付物 | 验收标准 |
|------|--------|---------|
| 存档系统 | 自动/手动保存 | 存档加载完整还原 |
| 存档管理 UI | 存档列表 | 选择/删除存档 |
| 道具系统 | 道具获取/使用 | 硬性通关道具条件 |
| 好感度 UI | 好感度面板 | 查看角色好感度 |
| 全面测试 | 测试报告 | 全功能无 P0 Bug |
| V1.5 发布 | 安装包 | 完整功能版本 |

---

## 六、里程碑设定

| 里程碑 | 时间节点 | 交付物 | 验收标准 |
|--------|---------|--------|---------|
| **M0 - 骨架就绪** | 第 2 周末 | 可运行的空项目 + 全部基础设施 | `tauri dev` 启动，画布可渲染 |
| **M1 - 画布可用** | 第 4 周末 | 画布编辑器基础功能 | 可创建/拖拽/连接场景节点 |
| **M2 - 编辑闭环** | 第 6 周末 | 角色+场景编辑完整流程 | 可编辑角色卡、场景卡，拖拽角色 |
| **M3 - AI 对话** | 第 8 周末 | 游戏运行器 + AI 对话 | 可与 AI 角色流式对话 |
| **M4 - MVP 完成** | 第 10 周末 | 完整 MVP 功能 | 创建→编辑→运行→通关→导出 |
| **M5 - MVP 发布** | 第 12 周末 | V1.0.0 安装包 | 可分发、文档齐全 |
| **M6 - 世界系统** | 第 15 周末 | V1.1 时间地点系统 | 触发器引擎可用 |
| **M7 - 多人对话** | 第 18 周末 | V1.2 多角色对话 | 3+ 角色同场景对话 |
| **M8 - 角色成长** | 第 20 周末 | V1.3 角色版本/状态 | 角色成长可体验 |
| **M9 - 多通关** | 第 23 周末 | V1.4 分支系统 | 多结局可游玩 |
| **M10 - 完整版** | 第 26 周末 | V1.5 完整版本 | 存档+道具+好感度齐全 |

---

## 七、团队分工建议

### 7.1 最小团队配置（1-2 人 + AI 辅助）

本项目由"普通人使用 AI 辅助编程"开发，建议如下分工策略：

| 角色 | 职责 | AI 辅助方式 |
|------|------|------------|
| **项目负责人（1人）** | 产品设计、架构决策、核心开发 | AI 辅助编码、架构咨询 |
| **前端开发（AI 主导）** | React 组件、画布、UI | CodeBuddy 生成代码，人工审核 |
| **后端开发（AI 主导）** | Rust 后端、AI Gateway | AI 生成 Rust/TS 代码，人工调试 |
| **AI 集成（AI 主导）** | Prompt 工程、通关判定 | AI 设计 Prompt，人工测试调优 |

### 7.2 推荐开发节奏

```
每日工作流程（假设 4-6 小时/天）：

1. 晨间规划（15min）
   - 查看昨日进度和遗留问题
   - 确定今日 3 个核心任务

2. 核心开发（3-4h）
   - 使用 CodeBuddy 辅助编码
   - 每完成一个功能立即测试
   - 提交 Git commit

3. 集成测试（30min）
   - 运行完整流程测试
   - 记录 Bug 到 TODO

4. 日终总结（15min）
   - 更新开发日志
   - 规划次日任务
```

### 7.3 如果扩展到 3-4 人团队

| 角色 | 人数 | 核心职责 |
|------|------|---------|
| 全栈开发（主程） | 1 | 架构、核心引擎、画布 |
| 前端开发 | 1 | UI 组件、游戏运行器、动效 |
| AI/Prompt 工程师 | 1 | AI 集成、Prompt 设计、通关判定 |
| 设计/测试（兼职） | 0.5 | UI 设计走查、测试 |

---

## 八、技术难点预估与解决方案

### 8.1 难点一：AI 对话的实时性与连贯性

**难点描述**：多角色对话中，AI 需要快速响应且保持角色一致性。延迟过高或角色"出戏"会严重破坏体验。

**解决方案**：
- 流式输出（SSE）：首 Token 延迟控制在 1s 内，打字机效果填充等待感
- Prompt 工程优化：结构化系统提示词，包含角色设定 + 当前场景 + 对话历史 + 行为约束
- 上下文窗口管理：滑动窗口保留最近 10 轮 + 摘要压缩历史
- 预生成策略：玩家输入时预判可能的回复方向，减少等待
- 多角色串行调用：避免并行调用导致角色"抢话"，通过协调器控制顺序

### 8.2 难点二：无限画布的性能与交互

**难点描述**：场景节点过多时，画布渲染变慢；自定义节点的交互复杂度高。

**解决方案**：
- 虚拟化渲染：只渲染视口内 + 缓冲区的节点，@xyflow/react 内建支持
- 节点简化模式：缩小时自动切换为简化显示（仅标题）
- 视口裁剪：边缘和离屏节点不参与事件计算
- 性能预算：单画布限制 200 节点，超出提示分组/拆分
- 防抖保存：编辑操作 500ms 防抖后持久化

### 8.3 难点三：通关判定的准确性

**难点描述**：软性通关（如"让对方原谅你"）依赖 AI 语义理解，可能出现误判或漏判。

**解决方案**：
- 双重验证机制：AI 初判 + 置信度阈值，低置信度时请求玩家确认
- 渐进式判定：每轮对话后计算"通关进度百分比"，非瞬时判定
- 关键词 + 语义混合：硬性条件精确匹配，软性条件语义分析
- 可调参数：创作者可调整 AI 判定的"严格度"
- 日志记录：所有判定结果和依据记录，便于创作者调试

### 8.4 难点四：游戏文件格式与资源管理

**难点描述**：.agalgame 需包含大量媒体资源，导入导出效率、跨平台兼容性是挑战。

**解决方案**：
- ZIP 格式打包：标准 ZIP 格式，资源原样存储，JSON 配置文件
- 增量保存：只保存变更部分，减少写入量
- 资源去重：相同文件 SHA256 去重，减少包体
- 大文件流式处理：超过 50MB 的资源不解压到内存，直接流式读取
- 资源引用路径规范化：统一使用相对路径，确保跨平台

### 8.5 难点五：Tauri + Node.js Sidecar 的集成调试

**难点描述**：Tauri 主进程与 Node.js sidecar 的启动/通信/异常处理增加复杂度。

**解决方案**：
- sidecar 生命周期管理：Tauri 启动时自动启动 sidecar，退出时清理
- 健康检查机制：sidecar 启动后发送就绪信号，超时重试
- 统一日志：Tauri 和 sidecar 日志统一输出到同一文件
- 开发模式代理：开发时 sidecar 独立运行，支持热重载
- 优雅降级：sidecar 不可用时，直接从 Tauri 发 HTTP 请求到 AI API

### 8.6 难点六：角色立绘与场景背景的叠加渲染

**难点描述**：游戏运行时需要在场景背景上叠加多个角色立绘，支持位置调整和表情切换。

**解决方案**：
- CSS 绝对定位分层：背景图底层 + 角色立绘层（绝对定位 + z-index）
- 角色位置配置：场景编辑时设定每个角色的屏幕位置 (x%, y%)
- 立绘状态切换：CSS transition 实现淡入淡出切换
- 性能优化：使用 `will-change: transform` 和 GPU 加速
- 备选 Canvas 渲染：若 CSS 方案性能不足，改用 Canvas 分层渲染

---

## 九、质量保障措施

### 9.1 代码规范

| 规范项 | 工具 | 配置 |
|--------|------|------|
| TypeScript 严格模式 | tsconfig.json | `strict: true`, `noUncheckedIndexedAccess: true` |
| 代码风格 | ESLint + Prettier | Airbnb 规范基础 + 自定义规则 |
| 命名规范 | ESLint 规则 | 组件 PascalCase、函数 camelCase、常量 UPPER_SNAKE |
| 导入排序 | eslint-plugin-import | 自动排序：外部 → 内部 → 相对 |
| 提交规范 | commitlint + husky | Conventional Commits 格式 |
| 类型安全 | tsc --noEmit | CI 中强制类型检查 |

### 9.2 测试策略

```
测试金字塔：

         ┌─────────┐
         │  E2E 测试 │  ← Playwright（5%）
         │  关键流程  │     画布操作、游戏运行、导入导出
         ├─────────┤
         │ 集成测试   │  ← Vitest（25%）
         │ 模块交互   │     Store+组件、AI Gateway、文件系统
         ├─────────┤
         │ 单元测试   │  ← Vitest（70%）
         │ 纯函数逻辑 │     通关判定、Prompt构建、数据转换
         └─────────┘
```

| 测试类型 | 覆盖范围 | 目标覆盖率 | 运行时机 |
|---------|---------|-----------|---------|
| 单元测试 | 工具函数、Store 逻辑、通关判定 | 80%+ | 每次提交 |
| 集成测试 | 组件交互、AI 调用（Mock）、文件操作 | 60%+ | 每次提交 |
| E2E 测试 | 完整游戏流程 | 核心流程 100% | 每日/发版前 |
| 手动测试 | UI 视觉、游戏体验 | — | 每个里程碑 |

**关键测试场景**：
- 创建场景 → 添加角色 → 设置通关条件 → 运行游戏 → 对话通关 → 切换场景
- 导出游戏 → 删除本地数据 → 导入游戏 → 验证数据完整
- 多角色场景 → 主导角色回复 → 切换发言角色 → AI 判断谁说话
- 长时间对话 → 上下文窗口裁剪 → AI 回复仍保持一致性

### 9.3 版本控制流程

```
分支策略（Git Flow 简化版）：

main          ──────────────────────────────────── 生产发布
              │           │           │
develop       ──────────────────────────────────── 开发主线
              │        │        │
feature/*     ──┐   ──┐   ──┐                  功能分支
                 │      │      │
hotfix/*      ────────────────────────────────── 紧急修复

规则：
1. feature 分支从 develop 创建，完成后 PR 合并回 develop
2. 里程碑节点从 develop 创建 release 分支，测试后合并到 main
3. hotfix 从 main 创建，修复后同时合并到 main 和 develop
4. PR 至少 1 人审核（即使是自己，也走 PR 流程留记录）
5. PR 必须通过 CI（lint + type-check + test）
6. Commit 信息遵循 Conventional Commits
```

### 9.4 CI/CD 流程

```
每次 Push / PR：
├── lint:check     → ESLint 检查
├── type:check     → TypeScript 类型检查
├── test:unit      → 单元测试
├── test:integration → 集成测试
└── build:check    → 构建验证

每日构建（Nightly）：
├── test:e2e       → E2E 测试
├── build:tauri    → Tauri 完整构建
└── report         → 测试报告 + 构建产物

发版流程：
├── 创建 release 分支
├── 执行全量测试
├── 构建生产包
├── 手动冒烟测试
└── 合并到 main + 打 Tag
```

---

## 十、潜在风险评估及应对策略

### 10.1 技术风险

| 风险 | 概率 | 影响 | 应对策略 |
|------|------|------|---------|
| @xyflow/react 性能不足 | 中 | 高 | 提前做性能基准测试；备选 Fabric.js + 自建节点系统 |
| Tauri sidecar 集成问题 | 中 | 中 | MVP 阶段先用纯 HTTP 直连 AI API，sidecar 渐进引入 |
| AI API 不稳定/延迟高 | 高 | 高 | 多模型备选 + 本地 Ollama 降级 + 流式输出降低感知延迟 |
| Rust 后端开发效率低 | 中 | 中 | 核心文件操作用 Rust，其余逻辑尽量在前端/Node.js 侧 |
| AI 角色一致性差 | 高 | 高 | 精细 Prompt 工程 + 多轮测试调优 + 上下文增强 |

### 10.2 产品风险

| 风险 | 概率 | 影响 | 应对策略 |
|------|------|------|---------|
| 核心玩法不吸引人 | 中 | 高 | MVP 尽早验证，快速迭代；找 5-10 人内测 |
| 通关判定体验差 | 高 | 中 | 提供多种判定模式让创作者选择；增加手动触发选项 |
| 创作门槛高 | 中 | 中 | 提供模板和 AI 辅助创作功能；详细教程 |
| AI 成本过高 | 高 | 中 | 优化 Token 消耗；支持本地模型；按场景配置模型 |

### 10.3 项目管理风险

| 风险 | 概率 | 影响 | 应对策略 |
|------|------|------|---------|
| 工期超期 | 高 | 中 | MVP 优先，严格功能裁剪；每周复盘进度 |
| AI 辅助编程质量不可控 | 中 | 高 | 人工审核所有生成代码；建立代码审查清单 |
| 需求蔓延 | 高 | 中 | 严格按版本规划执行；新需求入 V2.0 排期 |
| 技术债务积累 | 中 | 中 | 每个里程碑预留 20% 时间还债；持续重构 |

### 10.4 外部依赖风险

| 风险 | 概率 | 影响 | 应对策略 |
|------|------|------|---------|
| AI API 价格上涨 | 中 | 中 | 支持多模型切换；本地模型备选 |
| AI API 限流/封号 | 低 | 高 | 分散到多个 API Key；备用模型 |
| Tauri 重大版本变更 | 低 | 中 | 锁定 Tauri 版本；关注 Release Notes |
| xyflow 许可证变更 | 低 | 中 | 当前 MIT 许可；关注许可证动态 |

---

## 十一、开发进度时间表

### 11.1 甘特图（简化版）

```
2026年
4月          5月          6月          7月          8月          9月
├───────────┼───────────┼───────────┼───────────┼───────────┼───────────┤
W1-W2: Phase 0 基础设施
████████
W3-W4: 画布编辑器基础
        ████████
W5-W6: 角色+场景编辑
                ████████
W7-W8: 游戏运行+AI对话
                        ████████
W9-W10: 文件系统+MVP集成
                                ████████
W11-W12: MVP测试打磨
                                        ████
W13-W15: V1.1 时间地点系统
                                            ██████
W16-W18: V1.2 多人对话系统
                                                  ██████
W19-W20: V1.3 角色成长系统
                                                        ████
W21-W23: V1.4 多通关分支
                                                            ██████
W24-W26: V1.5 存档完善
                                                                  ██████
├───────────┼───────────┼───────────┼───────────┼───────────┼───────────┤
          M1           M3           M5           M7           M9          M10
```

### 11.2 每周核心目标

| 周次 | 核心目标 | 关键交付 |
|------|---------|---------|
| W1 | 项目初始化 | Tauri+React 启动 |
| W2 | 基础设施就绪 | xyflow 画布 Demo + SQLite |
| W3 | 自定义节点 | SceneNode 渲染 |
| W4 | 画布交互完整 | 拖拽+连线+缩放+右键菜单 |
| W5 | 角色卡系统 | CharacterEditor |
| W6 | 场景卡编辑 | SceneEditor + 角色拖入 |
| W7 | 游戏运行器布局 | GBA 左图右聊 |
| W8 | AI 对话集成 | SSE 流式对话 |
| W9 | 通关判定+场景切换 | 硬性通关可用 |
| W10 | 文件系统 | .agalgame 导入导出 |
| W11 | 测试+修复 | Bug 清零 |
| W12 | MVP 发布 | V1.0.0 |
| W13 | 时间系统 | 时间维度+跳转 |
| W14 | 地点系统 | 多地点移动 |
| W15 | 触发器引擎 | 条件组合触发 |
| W16 | 多角色对话协调 | 对话协调器 |
| W17 | 相关性判断 | AI 判断谁说话 |
| W18 | 立绘叠加 | 角色立绘分层 |
| W19 | 版本卡系统 | 初期/中期/后期卡 |
| W20 | 场景专属卡 | 状态绑定+好感度 |
| W21 | 软性通关 | AI 语义判断 |
| W22 | 分支点系统 | BranchNode + 选择 UI |
| W23 | 多通关逻辑 | 分支路由完整 |
| W24 | 存档系统 | 自动/手动保存 |
| W25 | 道具+好感度 UI | 完整游戏系统 |
| W26 | 最终测试+发布 | V1.5.0 |

---

## 十二、MVP 详细任务分解与优先级

### 12.1 任务优先级定义

- **P0（必须有）**：MVP 核心功能，缺少则无法演示
- **P1（应该有）**：提升体验的重要功能
- **P2（可以有）**：锦上添花，时间允许才做
- **P3（不做）**：MVP 明确排除

### 12.2 MVP 任务清单

| ID | 任务 | 优先级 | 预估工时 | 依赖 |
|----|------|--------|---------|------|
| T01 | Tauri 项目初始化 + Vite + React | P0 | 4h | — |
| T02 | TypeScript 配置 + ESLint + Prettier | P0 | 2h | T01 |
| T03 | shadcn/ui + Tailwind 集成 | P0 | 3h | T01 |
| T04 | @xyflow/react 集成 + 基础画布 | P0 | 4h | T01 |
| T05 | Zustand Store 骨架 | P0 | 3h | T01 |
| T06 | SQLite 集成 + 表结构 | P0 | 4h | T01 |
| T07 | 共享类型定义 | P0 | 3h | — |
| T08 | Node.js sidecar 骨架 | P1 | 4h | T01 |
| T09 | SceneNode 自定义节点组件 | P0 | 6h | T04 |
| T10 | 画布缩放/平移/抓手工具 | P0 | 4h | T04 |
| T11 | 场景连线 Edge 组件 | P0 | 4h | T04 |
| T12 | 双击打开编辑面板 | P0 | 3h | T09 |
| T13 | 右键上下文菜单 | P1 | 3h | T09 |
| T14 | 画布工具栏 | P1 | 3h | T04 |
| T15 | 撤销/重做 | P2 | 4h | T05 |
| T16 | characterStore + CRUD | P0 | 4h | T05, T06 |
| T17 | CharacterEditor 面板 | P0 | 6h | T16 |
| T18 | 角色库列表（左侧栏） | P0 | 4h | T16 |
| T19 | AI 模型选择器 | P0 | 2h | T17 |
| T20 | sceneStore + CRUD | P0 | 4h | T05, T06 |
| T21 | SceneEditor 面板 | P0 | 8h | T20 |
| T22 | 角色拖入场景 | P0 | 4h | T16, T20 |
| T23 | 背景图/BGM 上传 | P0 | 4h | T21 |
| T24 | 编辑器↔游戏模式切换 | P0 | 3h | T05 |
| T25 | GameRunner 左图右聊布局 | P0 | 6h | T24 |
| T26 | AI Gateway SSE 集成 | P0 | 6h | T08 |
| T27 | Prompt 模板构建 | P0 | 4h | T26 |
| T28 | 上下文窗口管理 | P0 | 4h | T26 |
| T29 | ChatWindow 对话界面 | P0 | 6h | T25 |
| T30 | 打字机效果 | P1 | 3h | T29 |
| T31 | 硬性通关判定引擎 | P0 | 4h | T28 |
| T32 | 场景开始/结束画面 | P0 | 4h | T25 |
| T33 | 线性场景切换 | P0 | 3h | T31 |
| T34 | .agalgame 文件打包 | P0 | 6h | T06 |
| T35 | .agalgame 文件解包导入 | P0 | 6h | T34 |
| T36 | 新建/保存/另存为 | P0 | 4h | T06 |
| T37 | 音频播放服务 | P1 | 4h | T25 |
| T38 | MVP 集成测试 | P0 | 6h | All |
| T39 | MVP Bug 修复 | P0 | 8h | T38 |

**MVP 总预估工时**：约 180 小时（按每天 4-6 小时有效开发，约 30-45 工作日）

---

## 十三、验收标准汇总

### 13.1 MVP 验收标准

| 编号 | 验收项 | 通过条件 |
|------|--------|---------|
| A01 | 画布创建场景 | 在画布上双击/点击按钮创建场景卡片，显示名称和简介 |
| A02 | 画布拖拽缩放 | 鼠标滚轮缩放（10%-200%），空格+拖拽平移画布 |
| A03 | 场景连线 | 从场景A拖出连线到场景B，显示有向连线 |
| A04 | 角色卡创建 | 创建角色卡，填写姓名/性格/AI设定/选择模型 |
| A05 | 场景卡编辑 | 编辑场景名/简介/目的/背景图/BGM/通关条件 |
| A06 | 角色拖入场景 | 从角色库拖拽角色到场景参与角色列表 |
| A07 | 模式切换 | 点击按钮在编辑器和游戏模式间切换，数据保持 |
| A08 | AI 流式对话 | 玩家输入文字，AI 角色流式回复（打字机效果） |
| A09 | 硬性通关 | 说出设定关键词，触发通关，显示通关画面 |
| A10 | 场景切换 | 通关后自动跳转下一场景，显示开始画面 |
| A11 | 导出游戏 | 导出当前游戏为 .agalgame 文件 |
| A12 | 导入游戏 | 导入 .agalgame 文件，画布正确渲染所有场景 |
| A13 | 数据持久化 | 关闭重启应用，所有编辑内容保留 |

### 13.2 各版本验收标准

| 版本 | 新增验收项 |
|------|-----------|
| V1.1 | 时间跳转功能正常；地点切换正常；触发器条件判定正确 |
| V1.2 | 3个角色同场景对话；AI 判断谁该回复；未参与角色显示标记 |
| V1.3 | 角色版本卡切换生效；场景专属卡生效；好感度随对话变化 |
| V1.4 | 软性通关 AI 判定合理；分支选择 UI 可用；不同通关导向不同剧情线 |
| V1.5 | 存档保存/加载完整还原；道具获取影响通关；好感度面板可查看 |

---

## 十四、参考资料

1. [xyflow - Node-Based UIs for React and Svelte](https://xyflow.com/)
2. [Tauri vs Electron 2025 Comparison](https://applicationize.me/tauri-vs-electron-2025-which-desktop-app-framework-wins-on-speed-security-and-features/)
3. [开源 Canvas 绘画引擎横评：Fabric.js vs Konva.js vs Meta2d.js](https://zhuanlan.zhihu.com/p/2019792963220292609)
4. [SSE 在 LLM 流式输出中的应用](https://juejin.cn/post/7519557137330520116)
5. [AI Character 角色扮演聊天对话管理方案](https://aws.amazon.com/cn/blogs/china/ai-character-chat-dialogue-management-solution-for-role-playing-applications/)
6. [Monogatari - Web Visual Novel Engine](https://monogatari.io/)
7. [14 Free and Open-source Visual Novel Engines for 2025](https://medevel.com/14-free-and-open-source-visual-novel-engines-for-2025/)
8. [Role-Playing with Language Models Survey (arXiv)](https://arxiv.org/pdf/2407.11484)
9. [Ren'Py Visual Novel Engine](https://www.renpy.org/)
10. [The Complete Guide to Streaming LLM Responses](https://dev.to/pockit_tools/the-complete-guide-to-streaming-llm-responses-in-web-applications-from-sse-to-real-time-ui-3534)

---

*本开发计划由 AI 辅助编制，基于项目设计文档分析和技术调研结果。*
*编制日期：2026-04-15*
