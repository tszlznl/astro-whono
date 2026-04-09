# Astro-Whono Code Wiki

## 1. 项目整体架构

`astro-whono` 是一个基于 **Astro v6** 构建的极简双栏静态博客/内容发布主题。其架构设计注重内容创作体验与本地配置的便捷性。项目采用了经典的分层架构：

- **视图与路由层 (`src/pages`, `src/layouts`, `src/components`)**：基于 Astro 的基于文件路由系统，负责页面的结构渲染和组件的复用。
- **内容数据层 (`src/content`, `src/data`)**：使用 `astro:content` 驱动，分为 `essay`（随笔）、`bits`（絮语）、`memo`（小记）三大核心集合。
- **业务逻辑层 (`src/lib`, `src/utils`)**：负责处理主题设置解析、Markdown 数据加工、Slug 路由校验及分类查询逻辑。
- **后台管理与接口层 (`src/pages/admin`, `src/pages/api/admin`)**：项目内置了本地独享的 **Theme Console**，在开发环境下提供图形化界面读写 `src/data/settings/*.json`。

## 2. 主要模块职责

| 目录/模块路径 | 职责说明 |
| --- | --- |
| `src/pages/` | 定义站点的所有路由，包括首页 (`/`)、列表及详情页 (`/archive`, `/essay`, `/bits`, `/memo`)，以及内置的后台控制台 (`/admin`) 和其依赖的本地 API (`/api`)。 |
| `src/components/` | UI 组件库，包含基础布局组件（如 `Sidebar`, `Callout`, `BitCard`）以及 `admin/` 目录下的主题配置控制台专用组件。 |
| `src/layouts/` | 页面布局模板，如基础框架 `BaseLayout.astro` 和文章详情框架 `ArticleLayout.astro`。 |
| `src/content/` | **Markdown 数据源**，存放所有文章和碎片化内容，按集合分类存放。 |
| `src/data/settings/` | 站点的 JSON 配置文件（Theme Console 生成和读取的地方），覆盖站点信息、主页、导航、UI配置等。 |
| `src/lib/` | 核心 TypeScript 逻辑层。处理复杂的业务逻辑，如获取并过滤内容 (`content.ts`)、读取与合并主题配置 (`theme-settings.ts`)。 |
| `src/plugins/` | Astro/Markdown 渲染插件，例如通过 `remark-callout` 支持 `:::note` 语法糖，通过 `shiki-toolbar` 增强代码块。 |
| `scripts/` | 工程化脚本，提供构建阶段的校验（如图片链接、格式检测）以及生成新文章草稿 (`new-bit.mjs`) 的工具。 |
| `public/` | 存放静态资源，如全局字体、图标、不需经过构建系统优化的静态图片。 |

## 3. 关键类与函数说明

### 3.1 内容获取与处理 (`src/lib/content.ts`)
该模块是对 `astro:content` 的进一步封装，包含对草稿 (Draft)、隐藏、排序、分页的处理逻辑。
- **`getPublished(name, opts)`**: 泛型函数，用于获取指定 Collection 下的所有已发布文章，在生产环境下自动过滤掉 `draft: true` 的内容。
- **`getSortedEssays()` / `getVisibleEssays()`**: 获取按时间倒序排列的随笔列表，且自动进行 Slug 冲突检测 (`assertUniqueEssaySlugs`) 与保留字检测。
- **`getEssayDerivedText(entry)`**: 解析 Markdown 正文，生成纯文本摘要和截断描述，内部具备缓存机制。

### 3.2 主题配置管理 (`src/lib/theme-settings.ts`)
Theme Console 的核心基石，负责读取 JSON、默认值回退、数据清洗与格式化。
- **`getThemeSettings()`**: 核心配置读取函数。按序读取 `site.json`, `shell.json`, `home.json` 等，与 `site.config.mjs` 中的旧版配置及系统默认值进行合并降级（Fallback），最终返回强类型的 `ThemeSettingsResolved` 对象。
- **`getEditableThemeSettingsState()`**: 用于 `/admin` 接口读取当前可编辑状态，包含了校验诊断信息（如 JSON 损坏、格式不符等）。

### 3.3 路由格式化 (`src/utils/slug-rules.ts`)
- **`flattenEntryIdToSlug(id)`**: 针对嵌套目录中的 Markdown 文件，将其路径拍平为 Kebab-case 的 Slug 格式（如 `2024/my-post` 转换为 `2024-my-post`）。

## 4. 依赖关系

本项目的核心构建及运行时依赖均围绕 Astro 生态与前端规范建立：
- **核心框架**: 
  - `astro (^6.1.1)`：作为底座，提供静态站点生成 (SSG)、基于文件的路由、Content Collections 等核心功能。
- **Astro 官方集成**: 
  - `@astrojs/rss`: 提供 XML RSS 生成能力。
  - `@astrojs/sitemap`: 自动化生成站点地图。
- **Markdown / HTML 处理**:
  - `remark-directive`: 解析自定义指令（用于 Callout 语法块）。
  - `rehype-raw`, `rehype-sanitize`: 处理并净化 Markdown 中的混合 HTML，保证渲染安全。
- **开发与测试**:
  - `vitest`: 提供配置及工具函数的单元测试能力。
  - `@astrojs/check` / `typescript`: 类型安全校验。

## 5. 项目运行方式

### 5.1 环境要求
- **Node.js**: >= 22.12.0 (项目根目录包含 `.nvmrc`)

### 5.2 常用命令

```bash
# 1. 安装依赖 (推荐在 CI/排障时使用 npm ci)
npm install

# 2. 启动本地开发服务器 (默认端口 http://localhost:4321)
# 启动后可访问 http://localhost:4321/admin 进入 Theme Console 配置主题
npm run dev

# 3. 生产环境构建
npm run build

# 4. 本地预览构建后的静态产物
npm run preview

# 5. 代码与类型检查
npm run check

# 6. 运行单元测试
npm run test

# 7. CI 核心回归流 (包含了构建、测试及文章格式等自定义脚本校验)
npm run ci

# 8. 创建新的 Bits (絮语) 草稿
npm run new:bit
```

### 5.3 部署注意事项
推荐在 Vercel、Netlify 或 Cloudflare Pages 上进行部署。部署时，**强烈建议**在环境变量中设置 `SITE_URL=https://你的域名`。这是因为 Astro 需要根据 `SITE_URL` 来生成正确的绝对链接（如 `sitemap.xml`, `robots.txt`, RSS 链接及 Open Graph Meta 信息）。
