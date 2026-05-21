# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

AI Browser 是一个基于 Next.js 15 + Electron 33 构建的 AI 智能桌面浏览器应用，支持多模态 AI 任务执行、定时任务、语音交互等功能。AI Agent 框架基于 @jarvis-agent（Eko）。

## 常用命令

```bash
# 安装依赖（使用 pnpm）
pnpm install

# 构建 Electron 依赖（preload + main process）
pnpm run build:deps

# 开发模式（需要先 build:deps，然后手动启动 next 和 electron）
pnpm run next          # Next.js 开发服务器 (port 5173)
pnpm run electron      # Electron 桌面应用

# 一键开发模式（自动启动 next + build:deps:watch + electron）
pnpm run dev

# 代码检查
pnpm run lint

# 构建（Next.js + Electron 依赖 + electron-builder 打包）
pnpm run build

# E2E 测试
pnpm run test:e2e
pnpm run test:e2e:ui   # Playwright UI 模式

# 单元测试
pnpm run test
```

## 开发文档索引

- [开发规范](docs/dev/CONVENTIONS.md) — 关键约定、环境变量、平台注意事项、Git 提交信息规范
- [项目架构](docs/dev/architecture.md) — 多进程架构、构建系统、核心模块、AI Agent 架构、设置系统

## 设计文档索引

- [页面路由系统](docs/design/pages/) — Next.js App Router 页面结构、路由设计、页面间关系与导航流
- [进程通讯与 Agent 元素交互](docs/design/browser-communication/) — Electron IPC 架构、标签页管理、AI Agent DOM 标注与元素交互机制
