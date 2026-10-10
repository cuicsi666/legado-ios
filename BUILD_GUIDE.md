# Legado iOS 构建指南

## 项目状态

源码来源：`oilycn/legado-ios`（GitHub fork，legado 原项目下架后的社区延续版）
技术栈：Swift 5.10 + SwiftUI + UIKit + CoreData
部署目标：iOS 16.0+
Bundle ID：`com.legado.app`

## 项目问题修复说明

原项目的 `Legado.xcodeproj` 由 AI 工具自动生成，存在以下问题（已用 XcodeGen project.yml 修复）：

1. **23个源文件未被加入编译** — `Core/Audio/`, `Core/Download/`, `Core/Theme/`, `Features/Reader/` 下的多个文件存在于文件系统但未在 xcodeproj 中引用
2. **Legado/ 目录冲突** — 包含一个早期 SwiftData 原型（4个文件），与主项目的 CoreData 架构冲突，有重复的 `@main` 和 `ContentView` 定义
3. **Package.swift 引用了不需要的 Readium 依赖** — 实际 EPUB 解析用的是 ZIPFoundation 自实现，不需要 Readium

修复方案：使用 `project.yml`（XcodeGen 配置）重新生成正确的 xcodeproj。

## 构建步骤（需要 macOS）

### 1. 安装 XcodeGen

```bash
# 方式一：Homebrew
brew install xcodegen

# 方式二：Mint
mint install yonaskolb/XcodeGen

# 方式三：手动编译
git clone https://github.com/yonaskolb/XcodeGen.git
cd XcodeGen
swift build -c release
```

### 2. 重新生成 Xcode 项目

```bash
cd legado-ios
# 删除旧的损坏的 xcodeproj
rm -rf Legado.xcodeproj
# 用 project.yml 重新生成
xcodegen generate
```

### 3. 用 Xcode 编译

```bash
# 命令行编译（无需打开 Xcode）
xcodebuild build \
  -project Legado.xcodeproj \
  -scheme Legado \
  -sdk iphoneos \
  -configuration Release \
  -destination 'generic/platform=iOS' \
  CODE_SIGNING_ALLOWED=NO \
  CODE_SIGN_IDENTITY="" \
  CODE_SIGNING_REQUIRED=NO

# 或在 Xcode 中打开
open Legado.xcodeproj
# 选择任意 iOS 设备 → ⌘B 编译
```

### 4. 打包 IPA（给全能签用）

```bash
# 找到编译产物
APP_PATH=$(find ~/Library/Developer/Xcode/DerivedData -name "Legado.app" -path "*/Release-iphoneos/*" | head -1)
echo "App path: $APP_PATH"

# 打包 IPA
mkdir -p /tmp/Payload
cp -R "$APP_PATH" /tmp/Payload/
cd /tmp
zip -qry Legado-unsigned.ipa Payload
echo "IPA: /tmp/Legado-unsigned.ipa"
```

### 5. 全能签安装

1. 将 `Legado-unsigned.ipa` 传输到 iPhone
2. 打开全能签，导入 IPA
3. 使用你的证书签名
4. 安装

## SPM 依赖

| 依赖 | 用途 | 版本 |
|------|------|------|
| SwiftSoup | HTML 解析（书源抓取） | ≥2.6.0 |
| Kanna | XPath 解析 | ≥5.2.4 |
| ZIPFoundation | EPUB 解压 | ≥0.9.19 |

JavaScriptCore 是系统框架，无需额外依赖。

## 如果没有 Mac

如果你没有 macOS 环境，可以用 GitHub Actions 自动构建：

1. Fork `oilycn/legado-ios` 到你的 GitHub 账号
2. 将修复后的 `project.yml` 推上去
3. 添加以下 GitHub Actions workflow：
   - 触发条件：push 到 main 或手动触发
   - 运行在 `macos-14`
   - 步骤：install xcodegen → generate → xcodebuild → package IPA → upload artifact
4. 在 Actions 页面下载构建好的 IPA

## 功能清单

- ✅ 自定义书源（CSS/XPath/JSONPath/正则/JavaScript）
- ✅ 聚合搜索（多源并发）
- ✅ 书架管理（网格/列表/分组）
- ✅ 阅读器（仿真/覆盖/滑动翻页）
- ✅ TXT/EPUB 本地阅读
- ✅ 替换净化（广告过滤）
- ✅ TTS 语音朗读
- ✅ WebDAV 同步
- ✅ RSS 订阅
- ✅ 书源调试器
- ✅ 阅读统计
- ✅ URL Scheme（legado://import/）
