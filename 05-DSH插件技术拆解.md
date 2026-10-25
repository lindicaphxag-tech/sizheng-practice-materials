# DSH 生态插件技术拆解：deepseek-splash + dsh-web

> 参考仓库：
> - 开机动画插件：`PZEHONG/deepseek-splash`
> - 聚合增强插件包：`zhu1090093659/dsh-web`（水墨皮肤 + 创意工坊 dsh-market）
> 底层宿主框架：**DeepSeek Harness（DSH）**，内核基于 **Cordis**（TypeScript 插件运行框架），一切皆插件架构。

## 1. deepseek-splash（开机动画插件）

属于 DSH Web 端客户端插件，只在浏览器前端运行，不改动 DSH 内核源码。

| 技术 | 说明 |
|---|---|
| TypeScript + ESBuild | 插件源码 TS 编写，esbuild 打包为插件 bundle，符合 DSH 插件规范；使用 Cordis 插件生命周期 `apply()` 做挂载/卸载清理 |
| DSH Shell Overlay 插槽机制 | 往全局 `shell.overlay` 插槽注入全屏遮罩 DOM，在 DSH Web UI 渲染前显示开机动画，DSH 就绪后自动移除 |
| CSS Keyframes 动画 / SVG 动画 | SVG `stroke-dashoffset` 路径描边做 Logo/文字手写动画；CSS 关键帧做浮动、渐变、淡入淡出 |
| 主题适配 | 跟随系统/DSH 主题 `prefers-color-scheme`、`data-ds-dark-theme` 自动切换深色/浅色动画配色 |
| HTML + 内联样式 | 动画页面为自包含 HTML 片段；样式标签标记 `data-plugin`，卸载时整体删除，无全局 CSS 污染 |
| DSH 插件协议 | `dsh.bundle` 清单文件，支持 `dsh plugin add github:xxx` 一键安装 |

> 安装提示词逻辑：需给 DSH 授予**构建脚本执行权限**，GitHub 来源插件安装会执行 pnpm 构建，不授权会报 pnpm 阻止 git 依赖构建的错。

## 2. dsh-web（水墨皮肤 + 创意工坊 dsh-market）

聚合型 Web 增强插件包。

### 水墨皮肤（主题皮肤模块）

| 技术 | 说明 |
|---|---|
| TypeScript | 运行在 DSH 客户端插件沙箱；插件 `apply()` 给 `document.body` 添加自定义 `data-*` 属性作为样式作用域，卸载时还原 body 状态、移除注入 CSS，热插拔无残留 |
| CSS Modules / 内联 CSS 变量 | 覆盖 DSH 原生 CSS 变量，修改界面配色、毛玻璃、背景 |
| 水墨效果 | SVG 水墨纹理、渐变滤镜 `filter:blur()`、混合模式 `mix-blend-mode` 实现国风墨韵 |
| WebGL（可选） | 部分动态水墨背景用 WebGL 做粒子水墨渲染，性能优化，支持壁纸联动（Wallpaper Engine 兼容） |
| 主题响应 | 监听 DSH 深色/浅色主题切换事件，动态切换水墨素材、配色 |

### 创意工坊 dsh-market（资产分发平台）

| 层 | 技术 |
|---|---|
| 前端 | 纯静态站点，TS 构建，读取 `skin.json`/`pet.json`/`community.json` 元数据自动生成工坊页面；皮肤支持在线预览试穿 |
| 后端动态能力 | Cloudflare Workers + D1 边缘数据库：点赞计数、设备级投票限制，无自建服务器 |
| DSH 内部集成 | WebUI 设置面板嵌入工坊页面；一键安装 = 封装调用 DSH 原生 CLI 接口 `dsh plugin add`，拉取 GitHub/NPM 插件包 |
| 分发规范 | 所有上架资产遵循 DSH `dsh.bundle` 插件清单标准 |

## 3. 底层公共基础（全部项目依赖）

- **Cordis**：DSH 内核，提供插件生命周期、依赖注入、事件总线；插件无需修改主程序源码即可挂载 UI、注册服务
- **TypeScript**：整个 DSH 生态主力语言
- **Node.js / pnpm**：后端运行 + 插件构建工具
- **React**：DSH WebUI 宿主本身是 React，插件复用宿主 React 上下文，不重复打包 React，减小插件体积

## 4. 模块速览

| 模块 | 运行位置 | 核心技术 |
|---|---|---|
| deepseek-splash 开机动画 | 浏览器前端插件 | TS + CSS Keyframes + SVG，Overlay 插槽注入全屏遮罩 |
| dsh-web 水墨皮肤 | 浏览器前端插件 | TS + CSS 变量 + SVG 水墨/WebGL 粒子，body 属性隔离样式域 |
| 创意工坊 dsh-market | 静态网页 + 边缘云 | Cloudflare Workers + D1，元数据驱动静态页面，调用 DSH CLI 安装插件 |
| DSH 宿主底层 | Node 后端 + React 前端 | Cordis 插件内核，TypeScript |

## 5. 补充提示

1. GitHub 来源插件安装会执行构建脚本，所以提示词要求「开放完全权限」；第三方插件有安全风险，仅信任星标社区插件
2. 皮肤只上架 dsh-web 创意工坊：皮肤元数据提交到 dsh-market，遵循 `dsh.bundle` 规范，不发布独立 NPM 包
