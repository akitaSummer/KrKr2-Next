<p align="center">
  <h1 align="center">KrKr2 Next</h1>
  <p align="center">基于 Flutter 重构的下一代 KiriKiri2 跨平台模拟器</p>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/status-In%20Development-orange" alt="Status">
  <img src="https://img.shields.io/badge/engine-KiriKiri2-blue" alt="Engine">
  <img src="https://img.shields.io/badge/framework-Flutter-02569B" alt="Flutter">
  <img src="https://img.shields.io/badge/graphics-ANGLE-red" alt="ANGLE">
  <img src="https://img.shields.io/badge/license-GPL--3.0-blue" alt="License">
</p>

---

**语言 / Language**: 中文 | [English](README_EN.md)

> 🙏 本项目基于 [krkr2](https://github.com/2468785842/krkr2) 重构，感谢原作者的贡献。

## 简介

**KrKr2 Next** 是 [KiriKiri2 (吉里吉里2)](https://zh.wikipedia.org/wiki/%E5%90%89%E9%87%8C%E5%90%89%E9%87%8C2) 视觉小说引擎的现代化跨平台运行环境。它完全兼容原版游戏脚本，使用现代图形接口进行硬件加速渲染，并在渲染性能和脚本执行效率上做了大量优化。基于 Flutter 构建统一的跨平台界面，支持 macOS · iOS · Windows · Linux · Android 五大平台。

下图为当前在 macOS 上通过 Metal 后端运行的实际效果：

<p align="center">
  <img src="doc/1.png" alt="macOS Metal 后端运行截图" width="800">
</p>

## 架构

<p align="center">
  <img src="doc/architecture.png" alt="技术架构图" width="700">
</p>

**渲染管线**：引擎通过 ANGLE 的 EGL Pbuffer Surface 进行离屏渲染（OpenGL ES 2.0），渲染结果通过平台原生纹理共享机制（macOS → IOSurface、Windows → D3D11 Texture、Linux → DMA-BUF）零拷贝传递给 Flutter Texture Widget 显示。


## 开发进度

> ⚠️ 本项目处于活跃开发阶段，尚未发布稳定版本。macOS 平台进度领先。

| 模块 | 状态 | 说明 |
|------|------|------|
| C++ 引擎核心编译 | ✅ 完成 | KiriKiri2 核心引擎全平台可编译 |
| ANGLE 渲染层迁移 | ✅ 基本完成 | 替代原 Cocos2d-x + GLFW 渲染管线，使用 EGL/GLES 离屏渲染 |
| engine_api 桥接层 | ✅ 完成 | 导出 `engine_create` / `engine_tick` / `engine_destroy` 等 C API |
| Flutter Plugin | ✅ 基本完成 | Platform Channel 通信、Texture 纹理桥接 |
| Texture 零拷贝渲染 | ✅ 基本完成 | 通过平台原生纹理共享机制零拷贝传递引擎渲染帧到 Flutter |
| Flutter 调试 UI | ✅ 基本完成 | FPS 控制、引擎生命周期管理、渲染状态监控 |
| 输入事件转发 | ✅ 基本完成 | 鼠标 / 触控事件坐标映射转发到引擎 |
| 引擎性能优化 | 🔨 进行中 | SIMD 像素混合、GPU 合成管线、VM 解释器优化等 |
| 游戏兼容性优化 | 🔨 进行中 | 补全解析引擎、添加插件，阶段目标与 Z 大闭源版兼容性持平 |
| 原有 krkr2 模拟器功能移植 | 📋 规划中 | 将原有 krkr2 模拟器功能逐步移植到新架构 |

## 平台支持状态

| 平台 | 状态 | 图形后端 | 纹理共享机制 |
|------|------|----------|-------------|
| macOS | ✅ 基本完成 | Metal | IOSurface |
| iOS | 🔨 流程打通，正在优化和修复 OpenGL 渲染 | Metal | IOSurface |
| Windows | 📋 计划中 | Direct3D 11 | D3D11 Texture |
| Linux | 📋 计划中 | Vulkan / Desktop GL | DMA-BUF |
| Android | 🔨 流程跑通，优化中 | OpenGL ES / Vulkan | HardwareBuffer |

## 引擎性能优化

| 优先级 | 任务 | 状态 |
|--------|------|------|
| P0 | 像素混合 SIMD 化 ([Highway](https://github.com/google/highway)) | ✅ 完成 |
| P0 | 全 GPU 合成渲染管线 | 🔨 进行中 |
| P0 | TJS2 VM 解释器优化 (computed goto) | 📋 计划中 |

## 开发环境搭建

目前 macOS、iOS、iOS Simulator、Android 的构建流程已经打通。下面命令以 macOS 开发机为准，默认使用 Apple Silicon、Xcode、Flutter 和 vcpkg 构建。

### 基础依赖

需要先安装：

| 依赖 | 建议版本 | 用途 |
|------|----------|------|
| Xcode | 15+ | macOS / iOS / iOS Simulator 构建和模拟器 |
| Flutter | 项目当前可用版本 | Flutter App 构建 |
| vcpkg | latest | C++ 第三方依赖 |
| CMake | 3.31.1+ | C++ 配置生成 |
| Ninja | latest | C++ 构建 |
| Bison | 3.8.2+ | TJS parser 生成 |
| Python 3 | 3.x | vcpkg、ANGLE、代码生成脚本 |
| NASM | latest | 部分 native 依赖构建 |
| libtool | GNU libtool | 配合部分 vcpkg 包构建，iOS 静态库合并使用系统 `/usr/bin/libtool` |
| Android SDK / NDK | Android 构建需要 | Android APK 构建 |

推荐通过 Homebrew 安装常用命令行依赖：

```bash
brew install cmake ninja bison nasm libtool pkg-config
```

确保 Xcode 命令行工具可用：

```bash
sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
xcodebuild -runFirstLaunch
```

如果缺少 iOS Simulator runtime，可以在 Xcode 的 `Settings -> Platforms` 安装，或使用：

```bash
xcodebuild -downloadPlatform iOS
```

### Flutter

构建脚本会优先使用项目内的 `.devtools/flutter`，否则使用 `PATH` 中的 `flutter`。

如果你自己安装 Flutter，需要确保：

```bash
flutter doctor
flutter precache --ios --android --macos
```

并把 Flutter 加入 `PATH`，例如：

```bash
export PATH="$HOME/dart/sdk/flutter/bin:$PATH"
```

### vcpkg

构建脚本会优先使用项目内的 `.devtools/vcpkg`。如果不存在，会自动 clone 并 bootstrap：

```text
.devtools/vcpkg
```

也可以提前手动准备：

```bash
mkdir -p .devtools
git clone https://github.com/microsoft/vcpkg.git .devtools/vcpkg
./.devtools/vcpkg/bootstrap-vcpkg.sh -disableMetrics
```

项目使用自己的 overlay ports 和 triplets：

```text
vcpkg/ports
vcpkg/triplets
```

不要直接改 vcpkg 官方 ports 目录来修项目依赖，优先改这些 overlay 目录。

### Android SDK / NDK

Android 构建需要 `ANDROID_HOME` 和 NDK。默认脚本会自动查找：

```text
$HOME/Library/Android/sdk
$ANDROID_HOME/ndk/<version>
```

也可以手动设置：

```bash
export ANDROID_HOME="$HOME/Library/Android/sdk"
export ANDROID_NDK_HOME="$ANDROID_HOME/ndk/28.2.13676358"
```

如果使用 Android Studio，安装：

- Android SDK Platform
- Android SDK Build-Tools
- Android SDK Command-line Tools
- Android NDK
- CMake

### Cubism SDK

Live2D Cubism 当前使用 `CubismSdkForNative-5-r.5`。项目期望 Cubism 文件位于：

```text
cpp/plugins/cubism/Core
cpp/plugins/cubism/Framework
```

如果需要从官方 zip 重新放置 SDK，把 `Core` 和 `Framework` 复制到上面的目录。iOS/iOS Simulator 构建会使用 Cubism Core 里的对应静态库：

```text
cpp/plugins/cubism/Core/lib/ios/Debug-iphoneos/libLive2DCubismCore.a
cpp/plugins/cubism/Core/lib/ios/Release-iphoneos/libLive2DCubismCore.a
cpp/plugins/cubism/Core/lib/ios/Debug-iphonesimulator-arm64/libLive2DCubismCore.a
cpp/plugins/cubism/Core/lib/ios/Release-iphonesimulator-arm64/libLive2DCubismCore.a
```

`Core/lib` 和 `Framework` 因为授权和体积原因不提交到仓库。新环境可以这样安装：

```bash
unzip /path/to/CubismSdkForNative-5-r.5.zip -d /tmp/cubism-sdk
mkdir -p cpp/plugins/cubism/Core cpp/plugins/cubism/Framework
rsync -a --exclude include /tmp/cubism-sdk/CubismSdkForNative-5-r.5/Core/ cpp/plugins/cubism/Core/
rsync -a /tmp/cubism-sdk/CubismSdkForNative-5-r.5/Framework/ cpp/plugins/cubism/Framework/
git apply cpp/plugins/cubism/patches/CubismSdkForNative-5-r.5.patch
```

这里不覆盖 `Core/include`，因为项目里已经提交了适配过的 Cubism Core 5-r.5 头文件。

## 构建和运行

统一入口是仓库根目录的 `build.sh`：

```bash
./build.sh <platform> [debug|release] [options]
```

可用平台：

```text
macos
ios
android
```

常用参数：

```text
debug|release       构建类型，默认 debug
--jobs=<N>          并行构建任务数，默认 8
--clean             清理对应平台构建产物后再构建
--simulator         iOS only，构建 iPhone/iPad Simulator 产物
--abi=<abis>        Android only，例如 arm64-v8a 或 arm64-v8a,x86_64
```

如果 Homebrew 的 `bison` 和 GNU `libtool` 没有在默认 `PATH` 前面，建议使用下面这个 PATH：

```bash
export PATH="/opt/homebrew/opt/bison/bin:/opt/homebrew/opt/libtool/libexec/gnubin:$PATH"
```

### macOS

构建 debug：

```bash
./build.sh macos debug --jobs=8
```

构建 release：

```bash
./build.sh macos release --jobs=8
```

输出位置：

```text
apps/flutter_app/build/macos/Build/Products/Debug/KrKr2 Next.app
apps/flutter_app/build/macos/Build/Products/Release/KrKr2 Next.app
```

运行：

```bash
open "apps/flutter_app/build/macos/Build/Products/Debug/KrKr2 Next.app"
```

或者：

```bash
"apps/flutter_app/build/macos/Build/Products/Debug/KrKr2 Next.app/Contents/MacOS/KrKr2 Next"
```

### iOS 真机 / iPhone / iPad

构建 unsigned debug app：

```bash
./build.sh ios debug --jobs=8
```

构建 unsigned release app：

```bash
./build.sh ios release --jobs=8
```

输出位置：

```text
apps/flutter_app/build/ios/iphoneos/Runner.app
```

真机部署通常需要打开 Xcode workspace 设置签名：

```bash
open "apps/flutter_app/ios/Runner.xcworkspace"
```

然后在 Xcode 中选择 Team、Bundle Identifier、设备，再 Run。

### iOS Simulator / iPhone 模拟器 / iPad 模拟器

构建 simulator debug：

```bash
./build.sh ios debug --simulator --jobs=8
```

输出位置：

```text
apps/flutter_app/build/ios/iphonesimulator/Runner.app
```

同一个 `iphonesimulator` 产物可以安装到 iPhone 和 iPad 模拟器。

查看可用模拟器：

```bash
xcrun simctl list devices available
```

启动一个模拟器：

```bash
xcrun simctl boot <device_udid>
open -a Simulator
```

安装 app：

```bash
xcrun simctl install <device_udid> apps/flutter_app/build/ios/iphonesimulator/Runner.app
```

启动 app：

```bash
xcrun simctl launch <device_udid> org.github.krkr2.flutterApp
```

打开 app 的 Documents 目录，便于放游戏文件：

```bash
APP_DATA=$(xcrun simctl get_app_container <device_udid> org.github.krkr2.flutterApp data)
mkdir -p "$APP_DATA/Documents"
open "$APP_DATA/Documents"
```

也可以在模拟器内打开 `文件` App，然后进入：

```text
浏览 -> 我的 iPhone / 我的 iPad -> Krkr2
```

注意：当前 iOS 真机和 iOS Simulator 会共用：

```text
bridge/flutter_engine_bridge/ios/Libs/libengine_project.a
bridge/flutter_engine_bridge/ios/Libs/libengine_vendors.a
```

所以当你从 simulator 构建切回真机构建时，需要重新执行：

```bash
./build.sh ios debug --jobs=8
```

从真机构建切回 simulator 也是同理：

```bash
./build.sh ios debug --simulator --jobs=8
```

### Android

构建 arm64 debug APK：

```bash
export ANDROID_HOME="$HOME/Library/Android/sdk"
./build.sh android debug --abi=arm64-v8a --jobs=8
```

构建多个 ABI：

```bash
export ANDROID_HOME="$HOME/Library/Android/sdk"
./build.sh android debug --abi=arm64-v8a,x86_64 --jobs=8
```

输出位置：

```text
apps/flutter_app/build/app/outputs/flutter-apk/app-debug.apk
apps/flutter_app/build/app/outputs/flutter-apk/app-release.apk
```

安装到设备或模拟器：

```bash
adb install "apps/flutter_app/build/app/outputs/flutter-apk/app-debug.apk"
```

启动：

```bash
adb shell am start -n org.github.krkr2.flutter_app/.MainActivity
```

## 常见问题

### CMake 找不到 Ninja

安装 Ninja 并确认在 `PATH` 中：

```bash
brew install ninja
ninja --version
```

### Bison 版本过低

macOS 自带 bison 版本通常过旧。使用 Homebrew 版本：

```bash
brew install bison
export PATH="/opt/homebrew/opt/bison/bin:$PATH"
bison --version
```

### iOS 构建时 libtool 行为异常

Homebrew 的 GNU libtool 和 Xcode 的 `/usr/bin/libtool` 不是同一个工具。项目脚本在合并 iOS 静态库时使用 `/usr/bin/libtool`。如果你手动合并 iOS 静态库，也应该使用：

```bash
/usr/bin/libtool -static -o output.a input1.a input2.a
```

### vcpkg 构建中断后磁盘占用过大

可以清理 vcpkg 临时目录：

```bash
rm -rf .devtools/vcpkg/buildtrees .devtools/vcpkg/packages
```

也可以清理平台构建目录：

```bash
rm -rf out/ios out/ios-simulator out/macos
rm -rf apps/flutter_app/build
```

或者使用脚本的 `--clean`：

```bash
./build.sh ios debug --clean --jobs=8
./build.sh macos debug --clean --jobs=8
./build.sh android debug --clean --abi=arm64-v8a --jobs=8
```

### iOS Simulator 无法访问 simctl / CoreSimulatorService

确认 Simulator app 已启动，必要时重启 CoreSimulator：

```bash
open -a Simulator
xcrun simctl list devices available
```

如果 CoreSimulatorService 状态异常，可以退出 Simulator 后重新打开，或重启开发机再试。

## 许可证

本项目基于 GNU General Public License v3.0 (GPL-3.0) 开源，详见 [LICENSE](./LICENSE)。
