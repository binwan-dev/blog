---
title: 在MacOS中编译dotnet/runtime源代码
date: 2022-11-16 22:28:33
categories: 
- dotnet/runtime
---

### 概述
闲来无事，本着好奇心想一探dotnet底层Socket是如何工作的，在github上下载源代码后，本着测试运行心态去调试一波。
直接运行```dotnet build```命令发现报错，后来搜索一番才发现编译不简单。因此，将整个过程进行记录，以备后续参考。
以下操作都是在macos上完成的。
### 环境准备
这部分参考官方文档即可（[MacOS下编译需要的环境](https://github.com/dotnet/runtime/blob/main/docs/workflow/requirements/macos-requirements.md)） 里面也有Window的介绍，大家也可以参考进行环境准备。

大致过程：安装XCode以及Toolchain Setup
1. 在AppStore中安装[XCode](https://apps.apple.com/us/app/xcode/id497799835)
2. 执行以下命令
```bash
# 选择xcode.app   
sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
   
# cd 到runtime项目根目录
cd /path/to/runtime

# 安装依赖包 CMake(3.15.5 or newer) icu4c openssl@(1.1 or 3) pkg-config python3 ninja
brew bundle --no-lock --file eng/Brewfile 
```

### 编译
安装完后直接执行```./build.sh```即可。（PS：墙裂建议挂代理，GFW问题真的让人头大）
注意：安装过多个版本SDK可能会有以下冲突问题
```bash
清单“wasm-tools”[microsoft.net.workload.mono.toolchain] 中的工作负载定义“/usr/local/share/dotnet/sdk-manifests/7.0.100/microsoft.net.workload.mono.toolchain/WorkloadManifest.json”与清单“microsoft.net.workload.mono.toolchain.net7”[/usr/local/share/dotnet/sdk-manifests/7.0.100/microsoft.net.workload.mono.toolchain.net7/WorkloadManifest.json] 冲突
```
出现上述问题，卸载掉相同版本的preview即可，如还未解决，直接卸载掉所有版本重新安装即可。
