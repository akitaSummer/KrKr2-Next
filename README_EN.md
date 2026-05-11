<p align="center">
  <h1 align="center">KrKr2 Next</h1>
  <p align="center">Next-Generation KiriKiri2 Cross-Platform Emulator Built with Flutter</p>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/status-In%20Development-orange" alt="Status">
  <img src="https://img.shields.io/badge/engine-KiriKiri2-blue" alt="Engine">
  <img src="https://img.shields.io/badge/framework-Flutter-02569B" alt="Flutter">
  <img src="https://img.shields.io/badge/graphics-ANGLE-red" alt="ANGLE">
  <img src="https://img.shields.io/badge/license-GPL--3.0-blue" alt="License">
</p>

---

**语言 / Language**: [中文](README.md) | English

> 🙏 This project is a refactor based on [krkr2](https://github.com/2468785842/krkr2). Thanks to the original author for the contribution.

## Overview

**KrKr2 Next** is a modern, cross-platform runtime for the [KiriKiri2](https://en.wikipedia.org/wiki/KiriKiri) visual novel engine. It is fully compatible with original game scripts, uses modern graphics APIs for hardware-accelerated rendering, and includes numerous optimizations for both rendering performance and script execution. Built on Flutter for a unified cross-platform UI, it targets macOS · iOS · Windows · Linux · Android.

The screenshot below shows the current running state on macOS with the Metal backend:

<p align="center">
  <img src="doc/1.png" alt="macOS Metal Backend Screenshot" width="800">
</p>

## Architecture

<p align="center">
  <img src="doc/architecture.png" alt="Architecture Diagram" width="700">
</p>

**Rendering Pipeline**: The engine performs offscreen rendering via ANGLE's EGL Pbuffer Surface (OpenGL ES 2.0). Rendered frames are delivered to the Flutter Texture Widget through platform-native texture sharing mechanisms (macOS → IOSurface, Windows → D3D11 Texture, Linux → DMA-BUF) with zero-copy transfer.


## Development Progress

> ⚠️ This project is under active development. No stable release is available yet. macOS is the most advanced platform.

| Module | Status | Notes |
|--------|--------|-------|
| C++ Engine Core Build | ✅ Done | KiriKiri2 core engine compiles on all platforms |
| ANGLE Rendering Migration | ✅ Mostly Done | Replaced legacy Cocos2d-x + GLFW pipeline with EGL/GLES offscreen rendering |
| engine_api Bridge Layer | ✅ Done | Exports `engine_create` / `engine_tick` / `engine_destroy` C APIs |
| Flutter Plugin | ✅ Mostly Done | Platform Channel communication, Texture bridge |
| Zero-Copy Texture Rendering | ✅ Mostly Done | Zero-copy engine render frame sharing to Flutter via platform-native texture mechanisms |
| Flutter Debug UI | ✅ Mostly Done | FPS control, engine lifecycle management, rendering status monitor |
| Input Event Forwarding | ✅ Mostly Done | Mouse / touch event coordinate mapping and forwarding to the engine |
| Engine Performance Optimization | 🔨 In Progress | SIMD pixel blending, GPU compositing pipeline, VM interpreter optimization, etc. |
| Game Compatibility | 🔨 In Progress | Completing the script parser, adding plugins. Current goal: match compatibility with Z's closed-source build |
| Original krkr2 Emulator Feature Porting | 📋 Planned | Gradually port original krkr2 emulator features to the new architecture |

## Platform Support

| Platform | Status | Graphics Backend | Texture Sharing |
|----------|--------|-----------------|----------------|
| macOS | ✅ Mostly Done | Metal | IOSurface |
| iOS | 🔨 Pipeline Working, Optimizing OpenGL Rendering | Metal | IOSurface |
| Windows | 📋 Planned | Direct3D 11 | D3D11 Texture |
| Linux | 📋 Planned | Vulkan / Desktop GL | DMA-BUF |
| Android | 🔨 Pipeline Working, Optimizing | OpenGL ES / Vulkan | HardwareBuffer |

## Engine Performance Optimization

| Priority | Task | Status |
|----------|------|--------|
| P0 | Pixel Blend SIMD ([Highway](https://github.com/google/highway)) | ✅ Done |
| P0 | Full GPU Compositing Pipeline | 🔨 In Progress |
| P0 | TJS2 VM Interpreter (computed goto) | 📋 Planned |

## Development Setup

The macOS, iOS, iOS Simulator, and Android build flows are currently working. The instructions below assume a macOS development machine, typically Apple Silicon, with Xcode, Flutter, and vcpkg.

### Required Tools

Install these dependencies first:

| Dependency | Recommended Version | Purpose |
|------------|---------------------|---------|
| Xcode | 15+ | macOS / iOS / iOS Simulator builds and simulators |
| Flutter | Project-compatible version | Flutter app builds |
| vcpkg | latest | C++ third-party dependencies |
| CMake | 3.31.1+ | C++ configure generation |
| Ninja | latest | C++ builds |
| Bison | 3.8.2+ | TJS parser generation |
| Python 3 | 3.x | vcpkg, ANGLE, and code generation scripts |
| NASM | latest | Some native dependency builds |
| libtool | GNU libtool | Required by some vcpkg packages; iOS archive merging uses system `/usr/bin/libtool` |
| Android SDK / NDK | Required for Android | Android APK builds |

Recommended Homebrew install:

```bash
brew install cmake ninja bison nasm libtool pkg-config
```

Make sure Xcode command line tools are configured:

```bash
sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
xcodebuild -runFirstLaunch
```

If the iOS Simulator runtime is missing, install it from `Xcode -> Settings -> Platforms`, or run:

```bash
xcodebuild -downloadPlatform iOS
```

### Flutter

The build scripts prefer `.devtools/flutter` inside the project. If it does not exist, they use `flutter` from `PATH`.

If you install Flutter yourself, verify it first:

```bash
flutter doctor
flutter precache --ios --android --macos
```

Add Flutter to `PATH`, for example:

```bash
export PATH="$HOME/dart/sdk/flutter/bin:$PATH"
```

### vcpkg

The build scripts prefer the project-local vcpkg checkout:

```text
.devtools/vcpkg
```

If it does not exist, the scripts clone and bootstrap it automatically. You can also set it up manually:

```bash
mkdir -p .devtools
git clone https://github.com/microsoft/vcpkg.git .devtools/vcpkg
./.devtools/vcpkg/bootstrap-vcpkg.sh -disableMetrics
```

This project uses overlay ports and triplets:

```text
vcpkg/ports
vcpkg/triplets
```

When fixing project dependencies, prefer editing these overlay directories instead of modifying vcpkg's built-in ports.

### Android SDK / NDK

Android builds require `ANDROID_HOME` and an installed NDK. The script auto-detects common macOS locations:

```text
$HOME/Library/Android/sdk
$ANDROID_HOME/ndk/<version>
```

You can also set them explicitly:

```bash
export ANDROID_HOME="$HOME/Library/Android/sdk"
export ANDROID_NDK_HOME="$ANDROID_HOME/ndk/28.2.13676358"
```

If you use Android Studio, install:

- Android SDK Platform
- Android SDK Build-Tools
- Android SDK Command-line Tools
- Android NDK
- CMake

### Cubism SDK

Live2D Cubism currently uses `CubismSdkForNative-5-r.5`. The project expects Cubism files here:

```text
cpp/plugins/cubism/Core
cpp/plugins/cubism/Framework
```

If you need to restore the SDK from the official zip, copy `Core` and `Framework` into those directories. iOS and iOS Simulator builds use the matching Cubism Core static libraries:

```text
cpp/plugins/cubism/Core/lib/ios/Debug-iphoneos/libLive2DCubismCore.a
cpp/plugins/cubism/Core/lib/ios/Release-iphoneos/libLive2DCubismCore.a
cpp/plugins/cubism/Core/lib/ios/Debug-iphonesimulator-arm64/libLive2DCubismCore.a
cpp/plugins/cubism/Core/lib/ios/Release-iphonesimulator-arm64/libLive2DCubismCore.a
```

`Core/lib` and `Framework` are not committed because of licensing and size. On a new machine, install them with:

```bash
unzip /path/to/CubismSdkForNative-5-r.5.zip -d /tmp/cubism-sdk
mkdir -p cpp/plugins/cubism/Core cpp/plugins/cubism/Framework
rsync -a --exclude include /tmp/cubism-sdk/CubismSdkForNative-5-r.5/Core/ cpp/plugins/cubism/Core/
rsync -a /tmp/cubism-sdk/CubismSdkForNative-5-r.5/Framework/ cpp/plugins/cubism/Framework/
git apply cpp/plugins/cubism/patches/CubismSdkForNative-5-r.5.patch
```

Do not overwrite `Core/include`; the repository already contains the adapted Cubism Core 5-r.5 headers.

## Build and Run

The unified entry point is `build.sh` in the repository root:

```bash
./build.sh <platform> [debug|release] [options]
```

Supported platforms:

```text
macos
ios
android
```

Common options:

```text
debug|release       Build type, default is debug
--jobs=<N>          Parallel build jobs, default is 8
--clean             Clean the selected platform's build artifacts before building
--simulator         iOS only, build for iPhone/iPad Simulator
--abi=<abis>        Android only, for example arm64-v8a or arm64-v8a,x86_64
```

If Homebrew's `bison` and GNU `libtool` are not ahead of the system tools in `PATH`, use:

```bash
export PATH="/opt/homebrew/opt/bison/bin:/opt/homebrew/opt/libtool/libexec/gnubin:$PATH"
```

### macOS

Build debug:

```bash
./build.sh macos debug --jobs=8
```

Build release:

```bash
./build.sh macos release --jobs=8
```

Output:

```text
apps/flutter_app/build/macos/Build/Products/Debug/KrKr2 Next.app
apps/flutter_app/build/macos/Build/Products/Release/KrKr2 Next.app
```

Run:

```bash
open "apps/flutter_app/build/macos/Build/Products/Debug/KrKr2 Next.app"
```

Or launch from a terminal:

```bash
"apps/flutter_app/build/macos/Build/Products/Debug/KrKr2 Next.app/Contents/MacOS/KrKr2 Next"
```

### iOS Devices / iPhone / iPad

Build an unsigned debug app:

```bash
./build.sh ios debug --jobs=8
```

Build an unsigned release app:

```bash
./build.sh ios release --jobs=8
```

Output:

```text
apps/flutter_app/build/ios/iphoneos/Runner.app
```

For deployment to a physical iPhone or iPad, open the Xcode workspace and configure signing:

```bash
open "apps/flutter_app/ios/Runner.xcworkspace"
```

Then choose your Team, Bundle Identifier, target device, and run from Xcode.

### iOS Simulator / iPhone Simulator / iPad Simulator

Build simulator debug:

```bash
./build.sh ios debug --simulator --jobs=8
```

Output:

```text
apps/flutter_app/build/ios/iphonesimulator/Runner.app
```

The same `iphonesimulator` build can run on both iPhone and iPad simulators.

List available simulators:

```bash
xcrun simctl list devices available
```

Boot a simulator:

```bash
xcrun simctl boot <device_udid>
open -a Simulator
```

Install the app:

```bash
xcrun simctl install <device_udid> apps/flutter_app/build/ios/iphonesimulator/Runner.app
```

Launch the app:

```bash
xcrun simctl launch <device_udid> org.github.krkr2.flutterApp
```

Open the app's Documents directory so you can copy game files into it:

```bash
APP_DATA=$(xcrun simctl get_app_container <device_udid> org.github.krkr2.flutterApp data)
mkdir -p "$APP_DATA/Documents"
open "$APP_DATA/Documents"
```

You can also use the Files app inside the simulator:

```text
Browse -> On My iPhone / On My iPad -> Krkr2
```

Note: iOS device and iOS Simulator builds currently share these bridge static libraries:

```text
bridge/flutter_engine_bridge/ios/Libs/libengine_project.a
bridge/flutter_engine_bridge/ios/Libs/libengine_vendors.a
```

When switching from simulator back to a physical device build, rebuild iOS:

```bash
./build.sh ios debug --jobs=8
```

When switching from physical device back to simulator, rebuild simulator:

```bash
./build.sh ios debug --simulator --jobs=8
```

### Android

Build an arm64 debug APK:

```bash
export ANDROID_HOME="$HOME/Library/Android/sdk"
./build.sh android debug --abi=arm64-v8a --jobs=8
```

Build multiple ABIs:

```bash
export ANDROID_HOME="$HOME/Library/Android/sdk"
./build.sh android debug --abi=arm64-v8a,x86_64 --jobs=8
```

Output:

```text
apps/flutter_app/build/app/outputs/flutter-apk/app-debug.apk
apps/flutter_app/build/app/outputs/flutter-apk/app-release.apk
```

Install on a device or emulator:

```bash
adb install "apps/flutter_app/build/app/outputs/flutter-apk/app-debug.apk"
```

Launch:

```bash
adb shell am start -n org.github.krkr2.flutter_app/.MainActivity
```

## Troubleshooting

### CMake Cannot Find Ninja

Install Ninja and make sure it is in `PATH`:

```bash
brew install ninja
ninja --version
```

### Bison Is Too Old

The bison bundled with macOS is usually too old. Use Homebrew's version:

```bash
brew install bison
export PATH="/opt/homebrew/opt/bison/bin:$PATH"
bison --version
```

### libtool Issues During iOS Builds

Homebrew GNU libtool and Xcode's `/usr/bin/libtool` are different tools. The project script uses `/usr/bin/libtool` when merging iOS static libraries. If you merge iOS archives manually, use:

```bash
/usr/bin/libtool -static -o output.a input1.a input2.a
```

### vcpkg Interrupted Build Uses Too Much Disk Space

Clean vcpkg temporary directories:

```bash
rm -rf .devtools/vcpkg/buildtrees .devtools/vcpkg/packages
```

You can also clean platform build directories:

```bash
rm -rf out/ios out/ios-simulator out/macos
rm -rf apps/flutter_app/build
```

Or use the script's `--clean` option:

```bash
./build.sh ios debug --clean --jobs=8
./build.sh macos debug --clean --jobs=8
./build.sh android debug --clean --abi=arm64-v8a --jobs=8
```

### iOS Simulator Cannot Access simctl / CoreSimulatorService

Make sure the Simulator app is running. If needed, restart CoreSimulator by reopening Simulator:

```bash
open -a Simulator
xcrun simctl list devices available
```

If CoreSimulatorService is in a bad state, quit Simulator and reopen it, or restart the development machine.

## License

This project is licensed under the GNU General Public License v3.0 (GPL-3.0). See [LICENSE](./LICENSE) for details.
