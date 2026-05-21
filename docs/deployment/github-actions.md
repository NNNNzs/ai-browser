# GitHub Actions CI/CD 工作流配置

## 概述

项目使用 GitHub Actions 实现自动化构建和发布，支持多平台并行构建。

## 工作流文件

位置：`.github/workflows/build-release.yml`

## 工作流结构

```yaml
name: Build and Release

on:
  push:
    branches: [mx]
  workflow_dispatch: null

permissions:
  contents: write

jobs:
  build:
    # 矩阵构建策略
    strategy:
      matrix:
        os: [macos-latest, windows-latest, ubuntu-latest]
      fail-fast: false
    
    # 构建步骤...
  
  release:
    needs: build
    # 发布步骤...
```

## 触发条件

| 事件 | 触发方式 | 说明 |
|------|----------|------|
| `push` | 推送到 `mx` 分支 | 自动触发构建 |
| `workflow_dispatch` | 手动触发 | 在 GitHub Actions 页面点击 "Run workflow" |

## 构建矩阵

### 平台配置

| Runner | 平台 | 架构 | 产物 |
|--------|------|------|------|
| `macos-latest` | macOS | Universal | `.dmg` |
| `windows-latest` | Windows | x64 | `.exe`, `.zip` |
| `ubuntu-latest` | Linux | x64 | `.AppImage`, `.deb` |

### fail-fast 设置

设置为 `false`，确保一个平台构建失败不会影响其他平台的构建。

## 构建步骤

### 1. 环境准备

```yaml
- name: Checkout repository
  uses: actions/checkout@v4

- name: Setup pnpm
  uses: pnpm/action-setup@v4

- name: Setup Node.js
  uses: actions/setup-node@v4
  with:
    node-version: '20'
    cache: 'pnpm'
```

### 2. 依赖安装和构建

```yaml
- name: Install dependencies
  run: pnpm install --frozen-lockfile

- name: Build Next.js
  run: pnpm run build:next

- name: Build Electron dependencies
  run: pnpm run build:deps
```

### 3. 平台特定构建

```yaml
- name: Build Electron app (Windows)
  if: matrix.platform == 'windows'
  run: pnpm run build:electron
  env:
    GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### 4. 上传 Artifacts

使用 `actions/upload-artifact@v4` 上传构建产物，保留 7 天。

```yaml
- name: Upload artifacts (macOS)
  uses: actions/upload-artifact@v4
  with:
    name: macos-universal
    path: release/*.dmg
    retention-days: 7
```

## 发布步骤

### 1. 下载所有 Artifacts

```yaml
- name: Download all artifacts
  uses: actions/download-artifact@v4
  with:
    path: artifacts
```

### 2. 创建 Release

使用 `softprops/action-gh-release@v2`（GitHub 官方推荐）：

```yaml
- name: Create Release
  uses: softprops/action-gh-release@v2
  with:
    tag_name: v0.2.3-${{ github.run_number }}
    name: Release v0.2.3-${{ github.run_number }}
    draft: false
    prerelease: true
    files: |
      artifacts/macos-universal/*.dmg
      artifacts/windows-x64/*
      artifacts/linux-x64/*
```

## 版本号规则

格式：`v{major}.{minor}.{patch}-{build_number}`

- 主版本号：来自 `package.json`
- Build Number：GitHub Actions 运行编号

示例：`v0.2.3-42`

## 权限配置

```yaml
permissions:
  contents: write  # 允许创建 Release
```

## Secrets 配置

### 自动提供

- `GITHUB_TOKEN`：GitHub 自动提供，用于 API 认证

### 可选配置（代码签名）

| Secret | 用途 | 平台 |
|--------|------|------|
| `CSC_LINK` | macOS 代码签名证书 | macOS |
| `CSC_KEY_PASSWORD` | 证书密码 | macOS |
| `WIN_CSC_LINK` | Windows 代码签名证书 | Windows |
| `WIN_CSC_KEY_PASSWORD` | 证书密码 | Windows |

## 信创平台支持

### Linux 构建

在 Ubuntu runner 上构建的 Linux 版本通常可以在信创系统上运行：

- **麒麟操作系统**：基于 Debian，兼容 `.deb` 包
- **统信 UOS**：基于 Debian，兼容 `.deb` 包
- **其他发行版**：使用 `.AppImage` 格式

### 兼容性配置

```yaml
# electron-builder.yml
linux:
  target:
    - AppImage  # 最通用格式
    - deb       # Debian 系发行版
  category: Development
```

## 故障排查

### 构建失败

1. **依赖问题**：检查 `pnpm-lock.yaml` 是否最新
2. **超时问题**：构建时间过长，考虑优化
3. **磁盘空间**：清理旧的 Artifacts

### 发布失败

1. **权限问题**：检查 `contents: write` 权限
2. **Tag 冲突**：检查 tag 是否已存在
3. **文件路径**：确认 artifacts 路径正确

### 调试技巧

```yaml
# 查看文件结构
- name: Display structure
  run: ls -R artifacts

# 查看环境变量
- name: Debug environment
  run: env | sort
```

## 最佳实践

1. **使用 `fail-fast: false`**：确保独立构建
2. **设置 Artifact 保留期**：避免占用过多存储
3. **使用 Pre-release**：测试后再发布正式版
4. **固定 Action 版本**：使用 `@v4` 而非 `@main`
5. **缓存依赖**：使用 `cache: 'pnpm'` 加速构建

## 扩展配置

### 添加通知

```yaml
- name: Send notification
  if: failure()
  uses: 8398a7/action-slack@v3
  with:
    status: ${{ job.status }}
    text: '构建失败！'
```

### 添加测试

```yaml
- name: Run tests
  run: pnpm test

- name: Run E2E tests
  run: pnpm test:e2e
```

### 手动发布到 NPM

```yaml
- name: Publish to NPM
  run: pnpm publish --access public
  env:
    NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

## 相关资源

- [GitHub Actions 文档](https://docs.github.com/en/actions)
- [Electron Builder 文档](https://www.electron.build/)
- [softprops/action-gh-release](https://github.com/softprops/action-gh-release)
- [pnpm-action-setup](https://github.com/pnpm/action-setup)
