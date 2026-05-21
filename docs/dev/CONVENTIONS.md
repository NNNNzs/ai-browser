# 开发规范

## 关键约定

- **UI 组件库**: Ant Design 6 + Tailwind CSS 4
- **状态管理**: Zustand（全局）+ Immer（不可变更新）
- **i18n**: 所有用户可见文本必须通过 i18n 翻译键
- **IPC 通信**: Renderer 通过 preload 暴露的 `window.api` 调用主进程功能
- **主题**: 支持深色/浅色/跟随系统，通过 cookie 传递主题配置
- **Node 版本**: 20.19.3
- **包管理器**: pnpm 10.18.2

## 环境变量

- `TTS_KEY` / `TTS_REGION`: Microsoft Azure 语音服务（TTS）
- `NODE_ENV`: development / production

## 平台注意事项

- macOS 为主要开发和优化平台
- Windows 构建可能需要多次尝试，有单独的 `build:deps:win` 和 `dev:win` 命令
- 设置存储位置：macOS `~/Library/Application Support/ai-browser/config.json`

## Git 提交信息规范

格式：`<类型>(<模块>): <简短描述>`

### 类型

| 类型 | 说明 |
|------|------|
| `feat` | 新功能 |
| `fix` | Bug 修复 |
| `doc` | 文档变更 |
| `refactor` | 重构（非新功能、非修复） |
| `style` | 代码格式调整（不影响逻辑） |
| `perf` | 性能优化 |
| `test` | 测试相关 |
| `chore` | 构建、工具链、依赖等杂项 |
| `ci` | CI/CD 配置变更 |

### 模块作用域

根据改动涉及的模块选取，常见作用域：

- `search` — 搜索引擎相关
- `ui` / `theme` — 界面、主题
- `i18n` / `lang` — 国际化、语言
- `ipc` — 进程间通信
- `agent` — AI Agent 相关
- `chat` — 对话模式
- `settings` — 设置系统
- `tab` / `browser` — 标签页、浏览器
- `memory` — 记忆系统
- `mcp` — MCP 协议
- `build` — 构建系统、打包
- `electron` — Electron 主进程

多个模块时可省略作用域或用 `/` 分隔，如 `feat(agent/memory): ...`

### 示例

```
feat(search): default to Chinese locale and Baidu search engine
fix(ipc): resolve settings not syncing across windows
doc(architecture): add IPC communication system design doc
chore(deps): upgrade Electron to v33.2.0
refactor(settings): extract theme logic into useTheme hook
```

### 规则

- 描述使用英文，首字母小写，不加句号
- 描述说明"做了什么"和"为什么"，而非"改了哪些文件"
- 单次提交保持原子性，一个逻辑变更一个提交
