---
title: Bazel的cmake编译源码成静态库
author: 独孤流
date: 2025-03-20 01:04:00 +0800
categories: [Pod_bazel_spm, Pod]
tags: [podsecp]     # TAG names should always be lowercase
---
> ### 前言
> 在上一篇里提到过，之前被telegram里源码编译成静态库的过程吓住了一直没研究，实际是BUILD里调用shell脚本，podspec里也有一样的功能，利用这些特性可以正常将BUILD迁移到podspec，具体内容在前一篇里有写，只是单独把编译源码的部分抽取出来

telegram里有不少库都是直接引入三方的源码，在使用bazel编译阶段，直接把源码编译成`xxx.a`的静态库
比如`submodules/ffmpeg`、`third-party/webp`、`third-party/jxl`、`third-party/mozipeg`等

以`webp`为例
```

headers = [
    "decode.h",
    "encode.h",
    "types.h",
]

libs = [
    "webp",
    "sharpyuv",
]

# libwebp at c1ffd9ac7593894c40a1de99d03f0b7af8af2577
filegroup(
    name = "libwebp_sources",
    srcs = glob([
        "libwebp/**/*"
    ]),
)

genrule(
    name = "webp_build",
    srcs = [
        "build-webp-bazel.sh",
        ":libwebp_sources",
        "@cmake_tar_gz//file",
    ],
    cmd_bash = 
    """
    set -ex

    if [ "$(TARGET_CPU)" == "ios_arm64" ]; then
        BUILD_ARCH="arm64"
    elif [ "$(TARGET_CPU)" == "ios_sim_arm64" ]; then
        BUILD_ARCH="sim_arm64"
    elif [ "$(TARGET_CPU)" == "ios_x86_64" ]; then
        BUILD_ARCH="x86_64"
    else
        echo "Unsupported architecture $(TARGET_CPU)"
    fi

    BUILD_DIR="$(RULEDIR)/build_$${BUILD_ARCH}"
    rm -rf "$$BUILD_DIR"
    mkdir -p "$$BUILD_DIR"

    CMAKE_DIR="$$(pwd)/$$BUILD_DIR/cmake"
    rm -rf "$$CMAKE_DIR"
    mkdir -p "$$CMAKE_DIR"
    tar -xzf "$(location @cmake_tar_gz//file)" -C "$$CMAKE_DIR"

    cp $(location :build-webp-bazel.sh) "$$BUILD_DIR/"

    SOURCE_PATH="third-party/webp/libwebp"

    cp -R "$$SOURCE_PATH" "$$BUILD_DIR/"

    mkdir -p "$$BUILD_DIR/Public/libwebp"

    PATH="$$PATH:$$CMAKE_DIR/cmake-3.23.1-macos-universal/CMake.app/Contents/bin" sh $$BUILD_DIR/build-webp-bazel.sh $$BUILD_ARCH "$$BUILD_DIR/libwebp" "$$BUILD_DIR"
    """ +
    "\n".join([
        "cp -f \"$$BUILD_DIR/libwebp/src/webp/{}\" \"$(location Public/webp/{})\"".format(header, header) for header in headers
    ]) +
    "\n" +
    "\n".join([
        "cp -f \"$$BUILD_DIR/build/lib{}.a\" \"$(location Public/webp/lib/lib{}.a)\"".format(lib, lib) for lib in libs
    ]),
    outs = ["Public/webp/" + x for x in headers] +
    ["Public/webp/lib/lib{}.a".format(x) for x in libs],
    visibility = [
        "//visibility:public",
    ]
)

cc_library(
    name = "webp_lib",
    srcs = [":Public/webp/lib/lib" + x + ".a" for x in libs],
)

objc_library(
    name = "webp",
    module_name = "webp",
    enable_modules = True,
    hdrs = [":Public/webp/" + x for x in headers],
    includes = [
        "Public",
        "Public/webp",
    ],
    deps = [
        ":webp_lib",
    ],
    visibility = [
        "//visibility:public",
    ],
)

```

这个使用了编译执行`build-webp-bazel.sh`生成静态库，`podspec`的`s.prepare_command`也可以执行脚本文件，直接改写即可
由于使用cmake编译，需要先安装camke
```
安装cmake
brew install cmake
cmake --version
```

一个中间转换的shell文件，方便代码及逻辑调试，当让直接卸载podspec里也是可以的
中间脚本文件`build_webp.sh`
```
#!/bin/bash
set -ex


SCRIPT_DIR=$(dirname "$(realpath "$0")")
# 1️⃣ 读取架构参数（默认为 arm64）
ARCH=${1:-arm64}

# 2️⃣ 解析 `BUILD_DIR`，按架构创建独立的 `build` 目录
BUILD_DIR="${SCRIPT_DIR}/build_${ARCH}"

# 3️⃣ 解析 `SOURCE_DIR`，确保是绝对路径
SOURCE_DIR="$SCRIPT_DIR/libwebp"

# 4️⃣ 删除旧的 `BUILD_DIR` 并重新创建
rm -rf "$BUILD_DIR"
mkdir -p "$BUILD_DIR"

# 5️⃣ 复制 `build-webp-bazel.sh` 到 `BUILD_DIR`
cp "$SCRIPT_DIR/build-webp-bazel.sh" "$BUILD_DIR/"

# 6️⃣ 复制 WebP 源码到 `BUILD_DIR`
cp -R "$SOURCE_DIR" "$BUILD_DIR/libwebp"

# 7️⃣ 执行 `build-webp-bazel.sh`，传递架构、源码路径、构建目录
sh "$BUILD_DIR/build-webp-bazel.sh" "$ARCH" "$BUILD_DIR/libwebp" "$BUILD_DIR"


BUILD_OUT_DIR="${SCRIPT_DIR}/Public"
rm -rf "$BUILD_OUT_DIR"
mkdir -p "$BUILD_OUT_DIR"
cp -R "$SOURCE_DIR/src/webp/types.h" "$BUILD_OUT_DIR/types.h"
cp -R "$SOURCE_DIR/src/webp/decode.h" "$BUILD_OUT_DIR/decode.h"
cp -R "$SOURCE_DIR/src/webp/encode.h" "$BUILD_OUT_DIR/encode.h"

BUILD_OUT_DIR_LIB="${SCRIPT_DIR}/Public/lib"
rm -rf "$BUILD_OUT_DIR_LIB"
mkdir -p "$BUILD_OUT_DIR_LIB"
cp -R "$BUILD_DIR/build/libsharpyuv.a" "$BUILD_OUT_DIR_LIB/libsharpyuv.a"
cp -R "$BUILD_DIR/build/libwebp.a" "$BUILD_OUT_DIR_LIB/libwebp.a"
cp -R "$BUILD_DIR/build/libwebpdecoder.a" "$BUILD_OUT_DIR_LIB/libwebpdecoder.a"
cp -R "$BUILD_DIR/build/libwebpdemux.a" "$BUILD_OUT_DIR_LIB/libwebpdemux.a"

echo "✅ WebP 构建完成！输出目录: $BUILD_DIR"

# 绝对路径
# sh /Users/xxx/xxx/third-party/webp/build_webp.sh arm64
# sh /Users/macbookpro/Documents/third/studyIM/TestPodspec/telegram/third-party/webp/build_webp.sh arm64
# 相对路径
# sh ./build_webp.sh x86_64
```

podspec的写法
```
Pod::Spec.new do |s|
  s.name         = "webp"
  s.version      = "1.0.0"
  s.summary      = "WebP 库的静态编译版本"
  s.description  = <<-DESC
    这是一个基于 WebP 的静态库，通过自定义脚本在安装时构建，
    并根据当前上下文自动选择目标架构（例如 arm64 或 x86_64）。
  DESC
  s.homepage     = "https://github.com/yourusername/WebP"
  s.license      = { :type => 'MIT', :file => 'LICENSE' }
  s.author       = { "Your Name" => "your.email@example.com" }
  s.platform     = :ios, "10.0"
  s.source       = { :git => "https://github.com/yourusername/WebP.git", :tag => s.version.to_s }
  
  # 在 Pod 安装过程中调用 prepare_command 运行构建脚本
  # 注意：这里假设 build_webp.sh 位于仓库根目录的上一级（例如 ${PODS_ROOT}/../build_webp.sh）
  # 参数分别为：自动检测架构、WebP 源码路径、输出基础目录（构建脚本内部会按架构生成 build_${ARCH} 目录）
  s.prepare_command = <<-CMD
    # 这个可能会报错，因为可能找到多个
    # PODSPEC_DIR=$(find "$(pwd)" -name "*.podspec" -exec dirname {} \\;)
    # 只取第一个就不会错了
    PODSPEC_DIR=$(find "$(pwd)" -name "*.podspec" -exec dirname {} \\; | head -n 1)
    echo "Podspec 所在目录: $PODSPEC_DIR"
    echo "${ARCHS}"
    sh ${PODSPEC_DIR}/build_webp.sh "$ARCHS"
  CMD
  
  # 假设构建脚本在构建完成后将静态库和头文件复制到固定目录：
  # - 静态库放在 Public/lib/libwebp.a
  # - 头文件放在 Public/include 下
  s.vendored_libraries = 'Public/**/libwebp.a'
  s.public_header_files = 'Public/**/*.h'
  
  # 如有需要，这里也可指定资源或其他文件，通常只需包含生成的 Public 目录
  s.source_files = 'Public/**/*'
  
  s.requires_arc = true
  s.module_name = "webp"
end

```

#### 在pod install提示文件夹路径不存在
```
s.prepare_command = <<-CMD
# 这个可能会报错，因为可能找到多个
# PODSPEC_DIR=$(find "$(pwd)" -name "*.podspec" -exec dirname {} \\;)
# 只取第一个就不会错了
PODSPEC_DIR=$(find "$(pwd)" -name "*.podspec" -exec dirname {} \\; | head -n 1)
echo "Podspec 所在目录: $PODSPEC_DIR"
echo "${ARCHS}"
sh ${PODSPEC_DIR}/build_webp.sh "$ARCHS"
CMD
```

也可以把2个文件合在一起
```
Pod::Spec.new do |s|
  s.name         = "webp"
  s.version      = "1.0.0"
  s.summary      = "WebP 库的静态编译版本"
  s.description  = <<-DESC
    这是一个基于 WebP 的静态库，通过自定义脚本在安装时构建，
    并根据当前上下文自动选择目标架构（例如 arm64 或 x86_64）。
  DESC
  s.homepage     = "https://github.com/yourusername/WebP"
  s.license      = { :type => 'MIT', :file => 'LICENSE' }
  s.author       = { "Your Name" => "your.email@example.com" }
  s.platform     = :ios, "10.0"
  s.source       = { :git => "https://github.com/yourusername/WebP.git", :tag => s.version.to_s }
  
  # 在 Pod 安装过程中调用 prepare_command 运行构建脚本
  # 注意：这里假设 build_webp.sh 位于仓库根目录的上一级（例如 ${PODS_ROOT}/../build_webp.sh）
  # 参数分别为：自动检测架构、WebP 源码路径、输出基础目录（构建脚本内部会按架构生成 build_${ARCH} 目录）
  s.prepare_command = <<-CMD
    # 这个可能会报错，因为可能找到多个
    # PODSPEC_DIR=$(find "$(pwd)" -name "*.podspec" -exec dirname {} \\;)
    # 只取第一个就不会错了
    PODSPEC_DIR=$(find "$(pwd)" -name "*.podspec" -exec dirname {} \\; | head -n 1)
    echo "Podspec 所在目录: $PODSPEC_DIR"
    echo "${ARCHS}"
    ARCH="${ARCHS:-arm64}" #如果不存在，默认arm64
    SCRIPT_DIR="${PODSPEC_DIR}"
    # 2️⃣ 解析 `BUILD_DIR`，按架构创建独立的 `build` 目录
    BUILD_DIR="${SCRIPT_DIR}/build_${ARCH}"

    # 3️⃣ 解析 `SOURCE_DIR`，确保是绝对路径
    SOURCE_DIR="$SCRIPT_DIR/libwebp"

    # 4️⃣ 删除旧的 `BUILD_DIR` 并重新创建
    rm -rf "$BUILD_DIR"
    mkdir -p "$BUILD_DIR"

    # 5️⃣ 复制 `build-webp-bazel.sh` 到 `BUILD_DIR`
    cp "$SCRIPT_DIR/build-webp-bazel.sh" "$BUILD_DIR/"

    # 6️⃣ 复制 WebP 源码到 `BUILD_DIR`
    cp -R "$SOURCE_DIR" "$BUILD_DIR/libwebp"

    # 7️⃣ 执行 `build-webp-bazel.sh`，传递架构、源码路径、构建目录
    sh "$BUILD_DIR/build-webp-bazel.sh" "$ARCH" "$BUILD_DIR/libwebp" "$BUILD_DIR"


    BUILD_OUT_DIR="${SCRIPT_DIR}/Public"
    rm -rf "$BUILD_OUT_DIR"
    mkdir -p "$BUILD_OUT_DIR"
    cp -R "$SOURCE_DIR/src/webp/types.h" "$BUILD_OUT_DIR/types.h"
    cp -R "$SOURCE_DIR/src/webp/decode.h" "$BUILD_OUT_DIR/decode.h"
    cp -R "$SOURCE_DIR/src/webp/encode.h" "$BUILD_OUT_DIR/encode.h"

    BUILD_OUT_DIR_LIB="${SCRIPT_DIR}/Public/lib"
    rm -rf "$BUILD_OUT_DIR_LIB"
    mkdir -p "$BUILD_OUT_DIR_LIB"
    cp -R "$BUILD_DIR/build/libsharpyuv.a" "$BUILD_OUT_DIR_LIB/libsharpyuv.a"
    cp -R "$BUILD_DIR/build/libwebp.a" "$BUILD_OUT_DIR_LIB/libwebp.a"
    cp -R "$BUILD_DIR/build/libwebpdecoder.a" "$BUILD_OUT_DIR_LIB/libwebpdecoder.a"
    cp -R "$BUILD_DIR/build/libwebpdemux.a" "$BUILD_OUT_DIR_LIB/libwebpdemux.a"

    rm -rf "$BUILD_DIR"

    echo "✅ WebP 构建完成！输出目录: $BUILD_DIR"
  CMD
  
  # 假设构建脚本在构建完成后将静态库和头文件复制到固定目录：
  # - 静态库放在 Public/lib/libwebp.a
  # - 头文件放在 Public/include 下
  s.vendored_libraries = 'Public/**/libwebp.a'
  s.public_header_files = 'Public/**/*.h'
  
  # 如有需要，这里也可指定资源或其他文件，通常只需包含生成的 Public 目录
  s.source_files = 'Public/**/*'
  
  s.requires_arc = true
  s.module_name = "webp"
end

```
----
### 编译成xcframework
前面说过的编译成`.a`文件很不方便，不能很好的做到一次编译模拟器和真机都运行，特改写为`.xcframework`的方式
方案：
1、这个要做的就是把真机的`arm64`、`m系列`芯片模拟器的`sim_arm64`、`intel`芯片的`x86_64`全部编译出来，
2、然后将sim_arm64、x64_64的静态文件合成一个.a文件（如果不这样干，直接把3个.a合成xcframework会报错`Both 'ios-x86_64-simulator' and 'ios-arm64-simulator' represent two equivalent library definitions.`）
3、再把合成的模拟器`.a`文件与真机的`arm64`的`.a`文件编译成`.xcframework`
完整脚本如下`build_webp_xcframework.sh`
```
#!/bin/bash
set -e  # 遇到错误立即退出
set -x  # 输出执行的命令

# 获取当前脚本所在目录
SCRIPT_DIR=$(dirname "$(realpath "$0")")

# 定义架构列表（真机和模拟器）
ARCH_LIST=("arm64" "sim_arm64" "x86_64")

# 1️⃣ **编译 WebP 并生成 `libwebp.a`**
build_webp_for_arch() {
    local arch=$1  # 传入的架构 (arm64 / x86_64 / sim_arm64)
    local source_dir=$2  # WebP 源码目录
    local build_dir="${SCRIPT_DIR}/out_${arch}/build"
    local output_headers_dir="${SCRIPT_DIR}/out_${arch}/headers"
    local output_lib_dir="${SCRIPT_DIR}/out_${arch}/lib"

    echo "🔧 开始编译 WebP: 架构=$arch"
    
    # 清理旧文件
    rm -rf "$build_dir" "$output_headers_dir" "$output_lib_dir"
    mkdir -p "$build_dir" "$output_headers_dir" "$output_lib_dir"

    # 复制构建脚本 & 源码
    cp "$SCRIPT_DIR/build-webp-bazel.sh" "$build_dir/"
    cp -R "$source_dir" "$build_dir/libwebp"

    # 执行构建
    sh "$build_dir/build-webp-bazel.sh" "$arch" "$build_dir/libwebp" "$build_dir"

    # 复制头文件
    cp -R "$source_dir/src/webp/"*.h "$output_headers_dir/"

    # 复制 `libwebp.a`
    cp -R "$build_dir/build/"*.a "$output_lib_dir/"

    echo "✅ WebP 编译完成！输出目录: $build_dir"
}

# 2️⃣ **合并 x86_64 + sim_arm64 形成通用模拟器库**
merge_simulator_archs() {
    local staticlib_name=$1 # 静态库的名称
    local universal_sim_dir="${SCRIPT_DIR}/out_universal_sim/lib"
    mkdir -p "$universal_sim_dir"

    local x86_lib="${SCRIPT_DIR}/out_x86_64/lib/${staticlib_name}"
    local sim_arm_lib="${SCRIPT_DIR}/out_sim_arm64/lib/${staticlib_name}"
    local output_lib="${universal_sim_dir}/${staticlib_name}"

    if [[ -f "$x86_lib" && -f "$sim_arm_lib" ]]; then
        lipo -create "$x86_lib" "$sim_arm_lib" -output "$output_lib"
        echo "✅ 合并 ${staticlib_name} 成功：$output_lib"
    else
        echo "⚠️ Warning: 缺少 ${staticlib_name} 目标文件，跳过合并"
    fi
}

# 3️⃣ **创建 WebP.xcframework**
create_xcframework() {
    local xcframework_name=$1  # xcframework的名称
    local staticlib_name=$2 # 静态库的名称
    local output_xcframework="${SCRIPT_DIR}/out_libs/${xcframework_name}"
    local headers_dir="${SCRIPT_DIR}/out_arm64/headers"
    local xcframework_cmd="xcodebuild -create-xcframework"

    # 合并模拟器的库
    merge_simulator_archs $staticlib_name

    # 添加 arm64（真机）
    local arm64_lib="${SCRIPT_DIR}/out_arm64/lib/${staticlib_name}"
    if [[ -f "$arm64_lib" ]]; then
        # xcframework_cmd+=" -library $arm64_lib -headers $headers_dir"
        xcframework_cmd+=" -library $arm64_lib"
    else
        echo "⚠️ Warning: ${staticlib_name} 在 out_arm64 目录下不存在，跳过"
    fi

    # 添加模拟器架构（合并后的 `out_universal_sim`）
    local sim_lib="${SCRIPT_DIR}/out_universal_sim/lib/${staticlib_name}"
    if [[ -f "$sim_lib" ]]; then
        # xcframework_cmd+=" -library $sim_lib -headers $headers_dir"
        xcframework_cmd+=" -library $sim_lib"
    else
        echo "⚠️ Warning: ${staticlib_name} 在 out_universal_sim 目录下不存在，跳过"
    fi

    xcframework_cmd+=" -output $output_xcframework"

    echo "🛠 执行命令: $xcframework_cmd"
    eval $xcframework_cmd  # 执行动态生成的命令

    echo "✅ WebP.xcframework 生成成功！文件路径: $output_xcframework"
}

# 删除文件夹
clean_directory() {
    local dir=$1
    if [[ -d "$dir" ]]; then
        rm -rf "$dir"
    fi
}


# 4️⃣ **编译 WebP**
SOURCE_DIR="$SCRIPT_DIR/libwebp"
for arch in "${ARCH_LIST[@]}"; do
    # 如果存在之前的中间过程文件夹，先删除
    clean_directory "${SCRIPT_DIR}/out_$arch"
    # 编译成xx.a
    build_webp_for_arch "$arch" "$SOURCE_DIR"
done
# 删除聚合的模拟器的库
clean_directory "${SCRIPT_DIR}/out_universal_sim"

# 如果存在之前的中间过程文件夹，先删除
clean_directory "${SCRIPT_DIR}/out_libs"

# 6️⃣ **创建 webp.xcframework**
SDK_NAMES=("webp.xcframework" "libsharpyuv.xcframework" "libwebpdecoder.xcframework" "libwebpdemux.xcframework")
SDK_LIBS=("libwebp.a" "libsharpyuv.a" "libwebpdecoder.a" "libwebpdemux.a")
for i in "${!SDK_NAMES[@]}"; do
    create_xcframework "${SDK_NAMES[$i]}" "${SDK_LIBS[$i]}"
done

# 复制.h文件
output_arm64_headers_dir="${SCRIPT_DIR}/out_arm64/headers"
output_libs_headers_dir="${SCRIPT_DIR}/out_libs/headers"
mkdir -p "$output_libs_headers_dir"
cp -R "$output_arm64_headers_dir/decode.h" "$output_libs_headers_dir/"
cp -R "$output_arm64_headers_dir/encode.h" "$output_libs_headers_dir/"
cp -R "$output_arm64_headers_dir/types.h" "$output_libs_headers_dir/"


# 删除过程中的文件夹
for arch in "${ARCH_LIST[@]}"; do
    clean_directory "${SCRIPT_DIR}/out_$arch"
done
# 删除聚合的模拟器的库
clean_directory "${SCRIPT_DIR}/out_universal_sim"


# 绝对路径
# sh /Users/xxx/xxx/third-party/webp/build_webp_xcframework.sh
# 相对路径
# sh ./build_webp_xcframework.sh
```

对应的podspec文件`webp_v2.podspec`
```
Pod::Spec.new do |s|
  s.name         = "webp_v2"
  s.version      = "1.0.0"
  s.summary      = "WebP 库的静态编译版本"
  s.description  = <<-DESC
    这是一个基于 WebP 的静态库，通过自定义脚本在安装时构建，
    并根据当前上下文自动选择目标架构（例如 arm64 或 x86_64）。
  DESC
  s.homepage     = "https://github.com/yourusername/WebP"
  s.license      = { :type => 'MIT', :file => 'LICENSE' }
  s.author       = { "Your Name" => "your.email@example.com" }
  s.platform     = :ios, "10.0"
  s.source       = { :git => "https://github.com/yourusername/WebP.git", :tag => s.version.to_s }
  
  # 在 Pod 安装过程中调用 prepare_command 运行构建脚本
  # 注意：这里假设 build_webp.sh 位于仓库根目录的上一级（例如 ${PODS_ROOT}/../build_webp.sh）
  # 参数分别为：自动检测架构、WebP 源码路径、输出基础目录（构建脚本内部会按架构生成 build_${ARCH} 目录）
  s.prepare_command = <<-CMD
    set -e
    PODSPEC_DIR=$(find "$(pwd)" -name "*.podspec" -exec dirname {} \\; | head -n 1)
    echo "Podspec 所在目录: $PODSPEC_DIR"
    
    if [ ! -f "$PODSPEC_DIR/build_webp_xcframework.sh" ]; then
      echo "❌ 错误: build_webp_xcframework.sh 文件不存在，检查路径！"
      exit 1
    fi

    sh "$PODSPEC_DIR/build_webp_xcframework.sh"
  CMD

  
  # 假设构建脚本在构建完成后将静态库和头文件复制到固定目录：
  # - 静态库放在 Public/lib/libwebp.a
  # - 头文件放在 Public/include 下
  # s.vendored_libraries = 'out_libs/*.a'
  s.vendored_frameworks = 'out_libs/**/*.xcframework'
  s.public_header_files = 'out_libs/**/*.h'
  
  # 如有需要，这里也可指定资源或其他文件，通常只需包含生成的 Public 目录
  s.source_files = 'out_libs/**/*.h'
  
  s.requires_arc = true
  s.module_name = "webp"
end

```

----
调用示例`Podfile`
```
# Uncomment the next line to define a global platform for your project
platform :ios, '12.0'

target 'TestPodspec' do
  # Comment the next line if you don't want to use dynamic frameworks
  use_frameworks!

  # Pods for TestPodspec
  pod 'webp', :path => 'telegram/third-party/webp'
  # 实际查找到的是'telegram/third-party/webp/webp_v2.podspec'，且s.name = "webp_v2"
  pod 'webp_v2', :path => 'telegram/third-party/webp'
  

  target 'TestPodspecTests' do
    inherit! :search_paths
    # Pods for testing
  end

  target 'TestPodspecUITests' do
    # Pods for testing
  end

end
```

![iamge](/assets/img/pod/BUILD_podspec_lib.png)