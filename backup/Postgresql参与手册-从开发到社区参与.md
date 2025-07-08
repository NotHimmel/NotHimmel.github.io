## 源码
https://github.com/postgres/postgres/tree/REL_18_BETA1

## 环境配置 Vscode gdb
### 编译选项
CFLAGS="-g -O0" ./configure  --prefix=/home/himmel/Dev/Pg18 --enable-depend --enable-cassert --enable-debug
make clean; make -j 4; make install;

## 配置文件
### .vscode/launch.json
```json
{
    "configurations": [
        {
            "name": "Debug PG SRV",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/pgsql/bin/postgres",
            "args": [
                "-D",
                "pgsql/data"
            ],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
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
                },
                {
                    "text": "-gdb-set follow-fork-mode child",
                    "ignoreFailures": true
                },
                {
                    "text": "-gdb-set detach-on-fork on",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "rebuild_db",
            "miDebuggerPath": "/usr/bin/gdb"
        },
        {
            "name": "Attach PG SRV",
            "type": "cppdbg",
            "request": "attach",
            "processId": "${command:pickProcess}",
            "program": "${workspaceFolder}/pgsql/bin/postgres",
            "MIMode": "gdb",
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
            ]
        },
    ],
    "version": "2.0.0"
}

``` 
### .vscode/task.json
```json
{
    "tasks": [
        {
            "type": "shell",
            "label": "install_depends",
            "command": "sudo apt install -y libsystemd-dev libxml2-dev libssl-dev libicu-dev zlib1g-dev libreadline-dev pkg-config",
            "options": {
                "cwd": "${workspaceFolder}"
            },
            "detail": "Task install depends."
        },
        {
            "type": "shell",
            "label": "build_env",
            "command": "mkdir build && mkdir -p pgsql/data",
            "options": {
                "cwd": "${workspaceFolder}"
            },
            "detail": "Task add folders."
        },
        {
            "type": "shell",
            "label": "build_config",
            "command": "../configure",
            "args": [
                "--prefix=${workspaceFolder}/pgsql",
                "--with-icu",
                "--with-openssl",
                "--with-systemd",
                "--with-libxml",
                "--enable-debug"
            ],
            "options": {
                "cwd": "${workspaceFolder}/build"
            },
            "detail": "Task Build MakeFile."
        },
        {
            "type": "shell",
            "label": "make",
            "command": "make",
            "args": [
                "-j12"
            ],
            "options": {
                "cwd": "${workspaceFolder}/build"
            },
            "detail": "Task build."
        },
        {
            "type": "shell",
            "label": "make_install",
            "command": "make",
            "args": [
                "install"
            ],
            "options": {
                "cwd": "${workspaceFolder}/build"
            },
            "detail": "Task install database."
        },
        {
            "type": "shell",
            "label": "init_db",
            "command": "pgsql/bin/initdb",
            "args": [
                "-D",
                "pgsql/data"
            ],
            "options": {
                "cwd": "${workspaceFolder}"
            },
            "detail": "Task init default database."
        },
        {
            "type": "shell",
            "label": "clean_db",
            "command": "make uninstall && make clean && rm -rf ../pgsql && rm -rf ../build",
            "options": {
                "cwd": "${workspaceFolder}/build"
            },
            "detail": "Task clean database."
        },
        {
            "type": "shell",
            "label": "build_db_conf",
            "dependsOn": [
                "build_env",
                "build_config"
            ],
            "dependsOrder": "sequence",
            "detail": "Task add folders."
        },
        {
            "type": "shell",
            "label": "build_db",
            "dependsOn": [
                "make",
                "make_install",
                "init_db"
            ],
            "dependsOrder": "sequence",
            "detail": "Task add folders."
        },
        {
            "type": "shell",
            "label": "rebuild_db",
            "dependsOn": [
                "make",
                "make_install"
            ],
            "dependsOrder": "sequence",
            "detail": "Task add folders."
        },
    ],
    "version": "2.0.0"
}
``` 

### gdb需要root权限
sudo visudo
himmel ALL=(ALL) NOPASSWD:/usr/bin/gdb



# Ref
1. https://wiki.postgresql.org/wiki/So,_you_want_to_be_a_developer%3F
2. https://github.com/liuyuanyuan/PostgresInternals