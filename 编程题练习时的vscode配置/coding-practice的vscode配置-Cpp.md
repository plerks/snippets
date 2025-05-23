# 编程题练习时的.vscode配置

## Windws下（使用MinGW-W64）

### 设置.exe生成在当前文件夹的配置（不推荐）:
tasks.json:
```json
{
    "tasks": [
        {
            "type": "cppbuild",
            "label": "C/C++: g++.exe build active file",
            "command": "E:\\x86_64-14.2.0-release-win32-seh-ucrt-rt_v12-rev0\\mingw64\\bin\\g++.exe",
            "args": [
                "-fdiagnostics-color=always",
                "-g",
                "-Wall",
                "-O0",
                "${file}",
                "-std=c++14",
                "-o",
                "${fileDirname}\\${fileBasenameNoExtension}.exe"
            ],
            "options": {
                "cwd": "${fileDirname}"
            },
            "problemMatcher": [
                "$gcc"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "detail": "Task generated by Debugger."
        }
    ],
    "version": "2.0.0"
}
```

**这种方式不好用**，到处都会有可执行文件留下，且：Windows下，我用的是MinGW-W64的g++。这里我组织命名题目文件夹的方式，题目的cpp文件路径中会出现中文和空格，对于运行C++，在Windows下，VSCode运行会出现找不到文件的问题，命令行手动g++编译是ok的，但是命令行手动gdb还是有问题。应该是MinGW-W64的gdb的问题，VSCode是能成功编译出可执行文件的，但是到用gdb启动程序时，由于中文路径，gdb会报错找不到文件。而Ubuntu下，虽然路径有中文和空格，VSCode，命令行g++，命令行gdb都没出问题。

由于中文路径的问题，这样配置的话只能在绝对路径无中文的位置运行调试完，然后再粘贴回来。

### 关于mingw gdb无法正常调试中文路径的原因
原因估计是以下这样，参考:
* https://learn.microsoft.com/zh-cn/windows/win32/learnwin32/working-with-strings
* https://learn.microsoft.com/zh-cn/windows/win32/intl/unicode-in-the-windows-api
* https://learn.microsoft.com/en-us/windows/win32/api/lzexpand/nf-lzexpand-lzopenfilea
* https://learn.microsoft.com/en-us/windows/win32/api/lzexpand/nf-lzexpand-lzopenfilew

[参考](https://learn.microsoft.com/zh-cn/windows/win32/learnwin32/working-with-strings)，windows下的api有两种，使用ANSI的和使用Unicode的(用的是UTF-16，宽字符)。在内部，ANSI版本将字符串转换为Unicode。

MinGW是GNU工具链在Windows上的移植，但是并不是完整兼容的。当给mingw gdb一个含中文的路径时，它应当做的是：调用宽字符api，并将给它的路径字符串转化为UTF-16并调用。但是mingw gdb估计是：1. 要么调用的是lzopenfilea这样的ANSI api，Windows内部再转化成Unicode，路径完全不对。 2. 调用的是lzopenfilew这样的宽字符api，但是mingw gdb并没有将路径字符串转化为UTF-16。

所以，对于中文路径，mingw gdb不能正常工作。

而Ubuntu下，默认都是utf-8，所以中文路径没有出现问题。

解决办法是：g++时，把.exe编译到绝对路径无中文的输出文件夹下，然后gdb再去那里调试（注意配置里的cwd得改成输出文件夹，mingw gdb的工作目录也不能有中文，不然会报错illegal byte sequence）：

### 设置.exe生成在out/的配置:

tasks.json:
```json
{
    "tasks": [
        {
            "type": "cppbuild",
            "label": "C/C++: g++.exe build active file",
            "command": "E:\\x86_64-14.2.0-release-win32-seh-ucrt-rt_v12-rev0\\mingw64\\bin\\g++.exe",
            "args": [
                "-fdiagnostics-color=always",
                "-g",
                "-Wall",
                "-O0",
                "${file}",
                "-std=c++14",
                "-o",
                "${workspaceFolder}\\out\\executables\\${fileBasenameNoExtension}.exe"
            ],
            "options": {
                "cwd": "${fileDirname}"
            },
            "problemMatcher": [
                "$gcc"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "detail": "Task generated by Debugger."
        }
    ],
    "version": "2.0.0"
}
```
launch.json:
```json
{
    "configurations": [
    {
        "name": "(gdb) Launch",
        "type": "cppdbg",
        "request": "launch",
        "program": "${workspaceFolder}\\out\\executables\\${fileBasenameNoExtension}.exe",
        "args": [],
        "stopAtEntry": false,
        "cwd": "${workspaceFolder}\\out", // Mingw gdb的工作目录也不能有中文，保证这个out文件夹绝对路径上无中文即可保证mingw的gdb可以正常调试
        "environment": [],
        "externalConsole": false,
        "MIMode": "gdb",
        "miDebuggerPath": "E:\\x86_64-14.2.0-release-win32-seh-ucrt-rt_v12-rev0\\mingw64\\bin\\gdb.exe",
        "setupCommands": [
            {
                "description": "Enable pretty-printing for gdb",
                "text": "-enable-pretty-printing",
                "ignoreFailures": true
            },
            {
                "description": "Set Disassembly Flavor to Intel",
                "text": "-gdb-set disassembly-flavor intel",
                "ignoreFailures": true
            }
        ],
        "preLaunchTask": "C/C++: g++.exe build active file"
    }
    ]
}
```

## Linux下
linux下路径有中文也不会有什么问题，不需要特别处理。

不过，如果要开sanitizer的话需要注意，我开了address,undefined这两个san，而启用AddressSanitizer(ASan)会默认启用LeakSanitizer(LSan)，而LeakSanitizer和gdb都依赖ptrace，会冲突，运行后会报错：LeakSanitizer has encountered a fatal error。

解决办法是设置环境变量把LSan给关掉，`export ASAN_OPTIONS="detect_leaks=0"`。

以下vscode配置，编译选项如果开了san的，都得先设置环境变量：`export ASAN_OPTIONS="detect_leaks=0"`，或者直接在`launch.json`的`environment`字段里设置。才能san和gdb一起使用。

### 开san，并设置可执行文件生成在当前文件夹的配置（不推荐）:
tasks.json:
```json
{
    "tasks": [
        {
            "type": "cppbuild",
            "label": "C/C++: 开san编译",
            "command": "/usr/bin/g++",
            "args": [
                "-fdiagnostics-color=always",
                "-fpermissive",
                "-fsanitize=address,undefined",
                "-std=c++20",
                "-g",
                "${file}",
                "-Wall",
                "-O0",
                "-o",
                "${fileDirname}/${fileBasenameNoExtension}"
            ],
            "options": {
                "cwd": "${fileDirname}"
            },
            "problemMatcher": [
                "$gcc"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "detail": "调试器生成的任务。"
        }
    ],
    "version": "2.0.0"
}
```
launch.json:
```json
{
    "configurations": [
        {
            "name": "(gdb) 启动",
            "type": "cppdbg",
            "request": "launch",
            "program": "${fileDirname}/${fileBasenameNoExtension}",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${fileDirname}",
            "environment": [
                {
                    "name": "ASAN_OPTIONS",
                    "value": "detect_leaks=0"
                }
            ],
            "externalConsole": false,
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                },
                {
                    "description": "将反汇编风格设置为 Intel",
                    "text": "-gdb-set disassembly-flavor intel",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "C/C++: 开san编译"
        }
    ]
}
```

### 开san，并设置可执行文件生成在out/的配置:
tasks.json:
```json
{
    "tasks": [
        {
            "type": "cppbuild",
            "label": "C/C++: 开san编译",
            "command": "/usr/bin/g++",
            "args": [
                "-fdiagnostics-color=always",
                "-fpermissive",
                "-fsanitize=address,undefined",
                "-std=c++20",
                "-g",
                "${file}",
                "-Wall",
                "-O0",
                "-o",
                "${workspaceFolder}/out/executables/${fileBasenameNoExtension}"
            ],
            "options": {
                "cwd": "${fileDirname}"
            },
            "problemMatcher": [
                "$gcc"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "detail": "调试器生成的任务。"
        }
    ],
    "version": "2.0.0"
}
```
launch.json:
```json
{
    "configurations": [
        {
            "name": "(gdb) 启动",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/out/executables/${fileBasenameNoExtension}",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${fileDirname}",
            "environment": [
                {
                    "name": "ASAN_OPTIONS",
                    "value": "detect_leaks=0"
                }
            ],
            "externalConsole": false,
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                },
                {
                    "description": "将反汇编风格设置为 Intel",
                    "text": "-gdb-set disassembly-flavor intel",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "C/C++: 开san编译"
        }
    ]
}
```