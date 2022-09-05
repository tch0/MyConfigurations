## VSCode C++通用编译运行调试配置

通用编译调试配置：
- `tasks.json`
- `launch.json`
- 使用环境变量中的`g++`进行编译，环境变量中的`gdb`进行调试。
- Windows中和Linux均可使用。

包含三个配置：
- `gdb debug single file`
    - 使用`g++`编译调试单一文件。
- `make and gdb debug single file`
    - 使用`make debug=yes`编译每个单一源文件到对应目标文件，方便自定义编译选项。配合[C++ Makefile 模板1](../MakefileTemplate/CppTemplate1.mk)使用。
- `make and gdb debug one target`
    - 使用`make debug=yes`编译目录下所有源文件到一个目标文件，配合[C++ Makefile 模板2](../MakefileTemplate/CppTemplate2.mk)使用。