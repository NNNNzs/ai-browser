# AI Browser 部署文档

## 目录

- [概述](#概述)
- [本地开发环境](#本地开发环境)
- [CI/CD 自动构建](#cicd-自动构建)
- [平台打包说明](#平台打包说明)
- [发布流程](#发布流程)
- [故障排查](#故障排查)

## 概述

AI Browser 使用 GitHub Actions 进行自动化构建和发布，支持 macOS、Windows 和 Linux（含信创平台）三大平台。

### 支持的平台和架构

| 平台 | 架构 | 打包格式 | 说明 |
|------|------|----------|------|
| macOS | Universal (x64 + arm64) | .dmg | 通用二进制，同时支持 Intel 和 Apple Silicon |
| Windows | x64 | .exe, .zip | NSIS 安装程序和压缩包 |
| Linux | x64 | .AppImage, .deb | 通用 Linux 格式，兼容信创系统 |

### 信创平台支持

本项目在以下国产操作系统上进行了兼容性测试：

- **麒麟操作系统**（桌面版）
- **统信 UOS**
- **其他基于 Debian 的国产 Linux 发行版**

## 本地开发环境

### 环境要求

- Node.js 20+
- pnpm 10+
- 根据平台不同，需要相应的构建工具：

#### macOS
```bash
# 安装 Xcode 命令行工具
xcode-select --install
```

#### Windows
```bash
# 需要安装 Visual Studio Build Tools
# 或使用 windows-build-tools
npm install --global windows-build-tools
```

#### Linux
```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install -y build-essential libgtk-3-dev libnotify-dev

# Fedora
sudo dnf install gcc-c++ make gtk3-devel libnotify-devel
```

### 本地构建

```bash
# 1. 安装依赖
pnpm install

# 2. 构建 Electron 依赖
pnpm run build:deps

# 3. 构建 Next.js
pnpm run build:next

# 4. 打包应用（当前平台）
pnpm run build
```

构建产物位于 `release/` 目录。

## CI/CD 自动构建

### 工作流触发条件

项目使用 GitHub Actions 自动构建，触发条件：

1. **自动触发**：当代码推送到 `mx` 分支时
2. **手动触发**：在 GitHub Actions 页面手动运行

### 工作流程

```
触发构建
  ↓
矩阵并行构建 (macOS, Windows, Linux)
  ↓
上传构建产物 (Artifacts)
  ↓
创建 GitHub Release
  ↓
上传各平台安装包
```

### 构建矩阵

| 操作系统 | Runner | 平台 | 架构 |
|----------|--------|------|------|
| macos-latest | GitHub-hosted | mac | universal |
| windows-latest | GitHub-hosted | windows | x64 |
| ubuntu-latest | GitHub-hosted | linux | x64 |

## 平台打包说明

### macOS 打包

#### Universal 二进制

项目配置生成 Universal 二进制，同时包含 Intel (x86_64) 和 Apple Silicon (arm64) 代码：

```yaml
# electron-builder.yml
mac:
  target:
    - target: dmg
      arch: universal
```

#### 代码签名（可选）

如需进行代码签名，需要配置 Apple Developer 证书：

1. 导出证书和配置文件
2. 在 GitHub Secrets 中添加：
   - `CSC_LINK`: .p12 证书文件（Base64 编码）
   - `CSC_KEY_PASSWORD`: 证书密码

```bash
# 将证书转换为 Base64
base64 -i certificate.p12 | pbcopy
```

### Windows 打包

#### 构建配置

```yaml
# electron-builder.yml
win:
  icon: assets/icons/logo.png
  target:
    - nsis
    - zip
```

#### 代码签名（可选）

如需代码签名，需要 Windows 代码签名证书：

1. 购买代码签名证书（如 DigiCert、Sectigo）
2. 在 GitHub Secrets 中添加：
   - `WIN_CSC_LINK`: 证书文件
   - `WIN_CSC_KEY_PASSWORD`: 证书密码

### Linux 打包

#### 支持的发行版

```yaml
# electron-builder.yml
linux:
  icon: assets/icons/icon.png
  target:
    - AppImage
    - deb
```

#### 信创平台适配

为确保在信创平台上的兼容性：

1. **依赖管理**：使用 Electron 内置依赖，减少系统库依赖
2. **字体支持**：内置中文字体，避免系统字体缺失问题
3. **打包格式**：
   - AppImage：最通用，几乎所有 Linux 发行版都支持
   - deb：适用于 Debian/Ubuntu 系发行版（包括大部分信创系统）

#### 在信创系统上运行

```bash
# AppImage 格式
chmod +x DeepFundAIBrowser-*.AppImage
./DeepFundAIBrowser-*.AppImage

# deb 格式
sudo dpkg -i deepfundai-browser_*.deb
```

## 发布流程

### 自动发布

每次推送到 `mx` 分支会自动：

1. 触发 GitHub Actions 构建
2. 创建新的 Release（预发布版本）
3. 上传所有平台的安装包

### 版本号规则

当前版本号格式：`v0.2.3-{build_number}`

- `0.2.3`：主版本号（在 package.json 中定义）
- `{build_number}`：GitHub Actions 运行编号

### 手动发布正式版本

如需发布正式版本：

1. 更新 `package.json` 中的版本号
2. 创建 Git tag：
   ```bash
   git tag v0.3.0
   git push origin v0.3.0
   ```
3. 在 GitHub Releases 页面编辑版本信息
4. 取消 "Pre-release" 标记

## 故障排查

### macOS

#### 问题：应用无法打开（来自身份不明开发者）

**解决方案**：
```bash
# 移除隔离属性
xattr -cr /Applications/DeepFundAIBrowser.app
```

#### 问题：代码签名失败

**检查清单**：
- 证书是否过期
- Bundle ID 是否匹配
- Entitlements 配置是否正确

### Windows

#### 问题：安装程序被 Windows Defender 拦截

**解决方案**：
- 进行代码签名
- 提交到 Microsoft SmartScreen 白名单

#### 问题：构建时出现 node-gyp 错误

**解决方案**：
```bash
# 清理缓存
pnpm store prune
rm -rf node_modules
pnpm install
```

### Linux

#### 问题：AppImage 无法运行

**解决方案**：
```bash
# 添加执行权限
chmod +x DeepFundAIBrowser-*.AppImage

# 如果 FUSE 不可用，提取后运行
./DeepFundAIBrowser-*.AppImage --appimage-extract
./squashfs-root/deepfundai-browser
```

#### 问题：依赖库缺失

**常见缺失库**：
```bash
# Ubuntu/Debian
sudo apt-get install libgtk-3-0 libnotify4 libnss3 libxss1 libxtst6 xdg-utils libatspi2.0-0 libdrm2 libgbm1 libxkbcommon0

# 信创系统（麒麟/UOS）
# 通常已包含所需依赖，如遇问题请联系系统供应商
```

### GitHub Actions

#### 问题：构建失败

**常见原因**：
1. 依赖安装失败 → 检查 `pnpm-lock.yaml` 是否最新
2. 超时 → 增加构建超时时间
3. 磁盘空间不足 → 清理旧的 Artifacts

**调试步骤**：
1. 查看 Actions 日志
2. 重新运行失败的工作流
3. 如需要，使用 `workflow_dispatch` 手动触发

#### 问题：Release 创建失败

**检查清单**：
- `GITHUB_TOKEN` 权限是否正确
- Tag 名称是否已存在
- Release 名称格式是否正确

## 附录

### 环境变量说明

| 变量名 | 用途 | 必需 |
|--------|------|------|
| `GITHUB_TOKEN` | GitHub API 认证 | 自动 |
| `CSC_LINK` | macOS 代码签名证书 | 可选 |
| `CSC_KEY_PASSWORD` | 证书密码 | 可选 |
| `WIN_CSC_LINK` | Windows 代码签名证书 | 可选 |
| `WIN_CSC_KEY_PASSWORD` | Windows 证书密码 | 可选 |

### 相关文件

```
.github/workflows/build-release.yml  # CI/CD 工作流配置
electron-builder.yml                 # 打包配置
package.json                         # 项目配置和依赖
```

### 参考资源

- [Electron Builder 文档](https://www.electron.build/)
- [GitHub Actions 文档](https://docs.github.com/en/actions)
- [Next.js 部署文档](https://nextjs.org/docs/deployment)
