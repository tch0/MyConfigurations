# Makefile模板

## C++ Makefile 模板

- 默认使用g++编译器。
- [CppTemplate1.mk](CppTemplate1.mk)：目录下每个源文件编译生成一个同名目标文件。
- [CppTemplate2.mk](CppTemplate2.mk)：同一目录下所有个源文件编译生成一个目标文件。
- `make debug=yes` 编译调试版本。
- `make system=windows` 选择windows系统（主要用于`clean`）。
- 在Vs Code中配合[VsCode C++ 调试配置](../VsCodeCppConfig)使用。

TODO：
- 自动识别操作系统。