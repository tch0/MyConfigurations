# VsCode C++编译运行调试配置

- `gdb debug single file`
    - 使用`g++`编译调试单一文件。
- `make and gdb debug single file`
    - 使用`make debug=yes`编译每个单一源文件到对应目标文件，方便自定义编译选项。配合[C++ Makefile 模板1](../MakefileTemplate/CppTemplate1.mk)使用。
- `make and gdb debug one target`
    - 使用`make debug=yes`编译目录下所有源文件到一个目标文件，配合[C++ Makefile 模板2](../MakefileTemplate/CppTemplate2.mk)使用。