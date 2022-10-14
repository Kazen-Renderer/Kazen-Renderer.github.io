---
layout: post
title: 使用 Rez 作为包管理工具开发 c++ 
tags: [cpp]
author: Zhong LingXiao
---



- Rez 是什么
  - rez 是什么?
  - 包管理是什么
  
- 安装 Rez
  - wsl2 安装开发

- 写一个简单的 Rez package
  - cmake / python 基础知识推荐视频
  - rp 中的 cmake / python 基本套路
  
  ## 什么是pkg-config

  简单理解，pkg-config根据`.pc`结尾的文件做依赖配置。
   找到.pc文件周，解析其内容，然后对底层构建工具（C/C++编译器、链接器）或高层构建工具（automake?, cmake）提供具体配置项目。
  
  通常是在POSIX系统（Linux，MacOS等）使用pkg-config，解决第三方依赖项配置问题。
  
  近些年来随着CMake的越发流行，原本用pkg-config的很多软件包提供了cmake的配置作为替代；少部分仍然提供.pc文件作为兼容考虑；还有另外一小部分的软件包，即使基于cmake构建了，对外提供依赖配置时仍然只有.pc文件。
  
  这就导致一个问题：虽然我学会了cmake在大部分时候都能解决依赖问题，但个别格楞子软件包还是要用pkg-config来搞。
  
  ## Building Packages
  
  rez包可以使用rez-build工具来构建和本地安装。这个工具有以下操作：
  
  - 遍历一个包的变体
  - 构建环境
  - 在此环境中运行构建系统
  
  每一个构建都来自构建的目录路径。（通常是构建的子目录或构建下的变体特定子目录）
  例如一个包含两个基本python变体的程序包：
  
  ## ENV cmake
  
  Operator to read environment variables.
  
  Use the syntax `$ENV{VAR}` to read environment variable `VAR`.
  
  To test whether an environment variable is defined, use the signature `if(DEFINED ENV{<name>})` of the [`if()`](https://cmake.org/cmake/help/latest/command/if.html#command:if) command.
  
  See the [`set()`](https://cmake.org/cmake/help/latest/command/set.html#command:set) and [`unset()`](https://cmake.org/cmake/help/latest/command/unset.html#command:unset) commands to see how to write or remove environment variables.
  
  ## CMAKE_PREFIX_PATH
  
  [Semicolon-separated list](https://cmake.org/cmake/help/latest/manual/cmake-language.7.html#cmake-language-lists) of directories specifying installation *prefixes* to be searched by the [`find_package()`](https://cmake.org/cmake/help/latest/command/find_package.html#command:find_package), [`find_program()`](https://cmake.org/cmake/help/latest/command/find_program.html#command:find_program), [`find_library()`](https://cmake.org/cmake/help/latest/command/find_library.html#command:find_library), [`find_file()`](https://cmake.org/cmake/help/latest/command/find_file.html#command:find_file), and [`find_path()`](https://cmake.org/cmake/help/latest/command/find_path.html#command:find_path) commands. Each command will add appropriate subdirectories (like `bin`, `lib`, or `include`) as specified in its own documentation.
  
  By default this is empty. It is intended to be set by the project.
  
  ## Build Environment Variables
  
  https://github.com/AcademySoftwareFoundation/rez/wiki/Environment-Variables
  
  These are variables that rez generates within a build environment, in addition to those listed [here](https://github.com/AcademySoftwareFoundation/rez/wiki/Environment-Variables#context-environment-variables).
  
  - **REZ_BUILD_ENV** - Always present in a build, has value 1.
  - **REZ_BUILD_INSTALL** - Has a value of 1 if an installation is taking place (either a *rez-build -i* or *rez-release*), otherwise 0.
  - **REZ_BUILD_INSTALL_PATH** - Installation path, if an install is taking place.
  - **REZ_BUILD_PATH** - Path where build output goes.
  - **REZ_BUILD_PROJECT_DESCRIPTION** - Equal to the *description* attribute of the package being built.
  - **REZ_BUILD_PROJECT_FILE** - The filepath of the package being built (typically a *package.py* file).
  - **REZ_BUILD_PROJECT_NAME** - Name of the package being built.
  - **REZ_BUILD_PROJECT_VERSION** - Version of the package being built.
  - **REZ_BUILD_REQUIRES** - Space-separated list of requirements for the build - comes from the current package's *requires*, *build_requires* and *private_build_requires* attributes, including the current variant's requirements.
  - **REZ_BUILD_REQUIRES_UNVERSIONED** - Equivalent but unversioned list to *REZ_BUILD_REQUIRES*.
  - **REZ_BUILD_SOURCE_PATH** - Path containing the package.py file.
  - **REZ_BUILD_THREAD_COUNT** - Number of threads being used for the build.
  - **REZ_BUILD_TYPE** - One of *local* or *central*. Value is *central* if a release is occurring.
  - **REZ_BUILD_VARIANT_INDEX** - Zero-based index of the variant currently being built. For non-varianted packages, this is "0".
  - **REZ_BUILD_VARIANT_REQUIRES** - Space-separated list of runtime requirements of the current variant. This does not include the common requirements as found in *REZ_BUILD_REQUIRES*. For non-varianted builds, this is an empty string.
  - **REZ_BUILD_VARIANT_SUBPATH** - Subdirectory containing the current variant. For non-varianted builds, this is an empty string.
  
  
  
  
  
  - tbb 包生成
  - 测试 tbb
    - target 名字是 TBB::tbb，可以在 kazen\packages\tbb\2021.2.0\platform-linux\arch-x86_64\lib\cmake\TBB\TBBTargets.cmake 
  
- 阶段总结



---



- 写一个复杂库的 Rez package
  - embree3 
  - 测试 embree 3 功能
- 写一个纯头文件库的 Rez package
  - tinyobjloader
  - 测试 tinyobjloader 功能
- 写一个自己的启动环境，测试开发
  - vfx platform 介绍
- Rez 优缺点分析
- 阶段总结