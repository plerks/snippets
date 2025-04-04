# 编程题练习时的.vscode配置

使用rustup安装rust后自带包管理器cargo，这里的配置是编程题练习时，不使用cargo，单文件编译与调试的vscode配置。

## Windows下
tasks.json:
```json
{
  "version": "2.0.0",
  "tasks": [
    { // rust不使用cargo，单文件编译与调试的配置
      "label": "rustc build",
      "type": "shell",
      "command": "rustc",
      "args": [
        "-g",
        "${file}",
        "-o",
        "${workspaceFolder}\\out\\executables\\${fileBasenameNoExtension}.exe"  // out/executables这个文件夹要提前创建好，不然rustc不会自动创建，会报错
      ]
    }
  ]
}
```
launch.json:
```json
{
    "configurations": [
        {
            "name": "(Windows) Launch",
            "type": "cppvsdbg", // 这里得是cppvsdbg，windows下rustc是用的msvc的工具链(从有.pdb文件就可以看出来)，type得是cppvsdbg
            "request": "launch",
            "program": "${workspaceFolder}\\out\\executables\\${fileBasenameNoExtension}.exe",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}\\out",
            "environment": [],
            "externalConsole": false,
            "preLaunchTask": "rustc build"
        }
    ]
}
```

## Linux下
tasks.json:
```json
{
    "version": "2.0.0",
    "tasks": [
      { // rust不使用cargo，单文件编译与调试的配置
        "label": "rustc build",
        "type": "shell",
        "command": "rustc",
        "args": [
          "-g",
          "${file}",
          "-o",
          "${workspaceFolder}/out/executables/${fileBasenameNoExtension}"  // out/executables这个文件夹要提前创建好，不然rustc不会自动创建，会报错
        ]
      }
    ]
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
            "cwd": "${workspaceFolder}/out",
            "environment": [],
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
            "preLaunchTask": "rustc build"
        }
    ]
}
```