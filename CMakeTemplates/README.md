<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [CMake模板](#cmake%E6%A8%A1%E6%9D%BF)
  - [我的CMake使用逻辑](#%E6%88%91%E7%9A%84cmake%E4%BD%BF%E7%94%A8%E9%80%BB%E8%BE%91)
  - [第三方库配置](#%E7%AC%AC%E4%B8%89%E6%96%B9%E5%BA%93%E9%85%8D%E7%BD%AE)
  - [项目配置](#%E9%A1%B9%E7%9B%AE%E9%85%8D%E7%BD%AE)
  - [文件树](#%E6%96%87%E4%BB%B6%E6%A0%91)
  - [其他资源](#%E5%85%B6%E4%BB%96%E8%B5%84%E6%BA%90)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# CMake模板

## 我的CMake使用逻辑

首先，我的喜好问题：
- 目前来说，我非常不喜欢使用`find_package`命令。
- 如果要使用第三方库，我的方式是使用git的子模块来管理，将第三方库的源码加入到项目的版本管理与源码体系中来，而不是使用编好的二进制或者去系统中找这个包。
- 因为第三方库可能是同样需要进行版本管理的，不同的项目可能使用同一第三方库的不同版本。
- 所以对一般项目绝不使用`find_package`来管理第三方库，而是将第三方库的源码统一放到`3rdparty`目录中，然后在项目文件夹内编译第三方库，并将第三方库统一安装到项目文件夹中的一目录，比如`3rdparty-install`。
- 第三方库和项目代码使用同一个工具链进行编译，就避免了任何可能出现的ABI问题，天然跨平台并且使用过程中可以无缝切换编译工具链，比如在Windows上既可以使用MinGW-w64也可以使用MSVC。

## 第三方库配置

添加任何第三方库到项目中：
```shell
git submodule add -b the-branch git@host:username/some-3rdparty-project.git 3rdparty/3rdparty-lib-name
```
使用第三方库的特定版本：
```shell
git checkout tag/commit
# then commit on main repository
```

第三方库通常都已经写好了安装逻辑，我们将头文件与库文件分别编译安装到`3rdparty-install/include`和`3rdparty-install/lib`目录中，使用`3rdparty/CMakeLists.txt`文件来管理第三方库：
```CMake
# 3rdparty/CMakeLists.txt
cmake_minimum_required(VERSION 3.15)

project(3rdparty)

# configurations like choosing of static and dynamic library
...
# 3rdparty libraries
add_subdirectory(3rdparty-lib1)
add_subdirectory(3rdparty-lib2)
add_subdirectory(3rdparty-lib3)
...

# set install destination to project-root-dir/3rdparty-install, headers to include, binary to lib
set(CMAKE_INSTALL_PREFIX ${CMAKE_CURRENT_SOURCE_DIR}/../3rdparty-install)

# 3rdparty libraries that do not have installation logic(like libs that do not have CMakeLists.txt or header only), install them by ourselves
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/some-3rdparty-lib/some-include-dir DESTINATION include)
```
项目构建前，需要先构建并安装第三方库到`3rdparty-install`：
```shell
cd ./3rdparty
mkdir build
cd ./build
```
选择要使用的工具链，编译Release版本供日常使用，也可以编译Debug版本用于在必要时调试第三方库逻辑：
- 对于单配置工具链：
```shell
cmake -G "your generator like MinGW" -DCMAKE_BUILD_TYPE=Release
cmake --build .
cmake --install .
```
- 对于多配置工具链：
```shell
cmake .. -G "your generator like Visual Studio 17 2022"
cmake --build . --config Release
cmake --install .
```

## 项目配置

```CMake
cmake_minimum_required(VERSION 3.15)

project (ProjectName
    DESCRIPTION "some description"
    VERSION "x.x.x"
    LANGUAGES CXX
)

# general compiler flags, C++ standard, warnings, etc.
add_library(general_cxx_compiler_flags INTERFACE)
# C++ standard
target_compile_features(general_cxx_compiler_flags INTERFACE cxx_std_20)
# for different toolchain
set(gcc_like_cxx "$<COMPILE_LANG_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU,LCC>")
set(msvc_cxx "$<COMPILE_LANG_AND_ID:CXX,MSVC>")
# warning options for different toolchain
target_compile_options(general_cxx_compiler_flags INTERFACE
    "$<${gcc_like_cxx}:$<BUILD_INTERFACE:-Wall;-Wextra;-Wshadow;-pedantic-errors;-Wformat=2;-Wno-unused-parameter>>"
    "$<${msvc_cxx}:$<BUILD_INTERFACE:/W3>>"
)

# extra compiler flags, like for inevitable annoying third-party library headers' warnings
add_library(extra_cxx_compiler_flags INTERFACE)
target_compile_options(extra_cxx_compiler_flags INTERFACE
    "$<${gcc_like_cxx}:$<BUILD_INTERFACE:-Wno-volatile>>" # to avoid some 3rdparty warning
)

# 3rd-party libraries:
#       static library: slib1 slib2 slib3 ...
#       header only: hlib1 hlib2 ...
list(APPEND 3rdparty_libs slib1 slib2 slib3 ...)

# 3rdparty include dir
set(3rdparty_include_dir ${CMAKE_SOURCE_DIR}/3rdparty-install/include)
# 3rdparty binary dir
set(3rdparty_library_dir ${CMAKE_SOURCE_DIR}/3rdparty-install/lib)


# other essential libraries that depend on OS
if (MINGW OR MSVC)
    list(APPEND essential_libs gdi32 opengl32)
else() # for Unix
    list(APPEND essential_libs GL X11 pthread dl)
endif()

# configuration of an executable
function(define_a_instance target)
    add_executable(${ARGV})
    target_include_directories(${target} PUBLIC ${3rdparty_include_dir})
    target_link_directories(${target} PUBLIC ${3rdparty_library_dir})
    target_link_libraries(${target}
        PRIVATE
            general_cxx_compiler_flags
            extra_cxx_compiler_flags
        PUBLIC
            SomeInternalLibraries ...
            ${3rdparty_libs}
            ${essential_libs}
    )
endfunction()

# copy resources used by executable to its corresponding binary output directory after build target.
function(copy_resources_after_build_target target sources)
    set(sources ${ARGV})
    list(POP_FRONT sources)
    add_custom_command(TARGET ${target} POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy -t $<TARGET_FILE_DIR:${target}> ${sources}
        COMMAND_EXPAND_LISTS
    )
endfunction()

# internal libraries for all programs
add_subdirectory(internal-lib1)
...
# subprojects
add_subdirectory(sub-dir1)
add_subdirectory(sub-dir2)
...
```
- 构建项目：
```shell
cd project-root-dir
mkdir build
cd ./build
cmake .. -G "your generator"
cmake --build .
```

## 文件树

最终的文件树会是这个样子的：

```
│   CMakeLists.txt
│
├───3rdparty
│   │   CMakeLists.txt
│   │
│   └───3rdparty-lib1
├───3rdparty-install
│   ├───include
│   │   └───3rdparty-lib1
│   │           header1.h
│   │           header2.h
│   │
│   ├───lib
│   │       3rdparty-lib1.a
│   │       3rdparty-lib1.dll
│   │       3rdparty-lib1.lib
│   │
│   └───share
├───internal-lib1
│   │   CMakeLists.txt
│   │
│   ├───inc
│   │       someheader.h
│   │
│   └───src
│           somesrc.cpp
│
└───sub-dir1
    │   CMakeLists.txt
    │
    ├───inc
    │       someheader.cpp
    │
    └───src
            somesrc.h
```

- 其中`3rdparty-install`目录不需要追踪，需要添加到`.gitignore`中，第三方库编译后由CMake进行安装。

## 其他资源

- [CMake配置速查](https://github.com/tch0/MyCommandCheatSheet/blob/master/CMakeConfig.md)