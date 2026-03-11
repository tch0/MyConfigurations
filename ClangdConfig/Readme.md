# clangd使用指南

- [VS Code 插件 clangd的用法](https://www.cnblogs.com/newtonltr/p/18867195)

配合VsCode的使用指南：
- clangd可执行文件
    - 通过插件安装，或者LLVM官网下载安装配置到path
- clangd插件
    - 插件市场安装
- .vscode/settings.json
    - 配置启用clangd插件禁用微软C++ intellisense
    - 传递基础参数给clangd
- compile_commands.json
    - 其中保存针对每一个文件的编译命令
    - 通过cmake/intercept-build/bear这些工具生成
    - `cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=ON ..`
    - clangd通过该文件对每一个源文件的编译参数，以精准得知编译器、语言标准、预定义宏等信息
- .clangd 主配置
    - clangd项目配置，为特定文件目录开启关闭索引诊断
    - 添加删除编译参数
    - 配置clang-tidy和clang-format
- .clang-tidy
    - 代码诊断细节配置
- .clang-format
    - 代码风格、格式化细节配置

MSVC编译器VsCode中clangd插件配置示例：
- [settings.json](settings.json)
- [.clangd](.clangd)
- [.clang-tidy](.clang-tidy)
