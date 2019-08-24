# tasks.json for gfortran with cmake running on Linux

# Requirement
- Visual Studio Code
- gfortran
- cmake
- make

# Usage
## CMakeLists.txtの作成
CMakeLists.txtをソースファイルと同じディレクトリに作ります．CMakeLists.txtはcmakeを用いたビルドの設定を記述するファイルです．
CMakeLists.txtには，最低限3個の項目（Fortranの場合は1項目増えて4項目）を記述します．

1. CMakeの要求バージョン
1. Fortranの有効化
1. プロジェクト名
1. コンパイルするファイルと作成される実行ファイル名

```
cmake_minimum_required(VERSION 3.10)

enable_language(Fortran)

project(PUT-PROJECT-NAME-HERE Fortran)

add_executable(${PROJECT_NAME}.out
    PUT-SOURCE-FILE-HERE.f90
)
```

`PUT-SOURCE-FILE-HERE.f90`をコンパイルしたいソースファイル名に変更します．複数のファイルがある場合は，列挙してください．

コンパイラに渡すオプションは，set()を使って与えます．個々に設定するのではなく，DebugビルドやReleaseビルドのように，ビルド単位で与えます．
例えば

```
set(CMAKE_Fortran_FLAGS_DEBUG "-g -O0 -Wall")
set(CMAKE_Fortran_FLAGS_RELEASE "-O3 -march=native")
```

## ビルドディレクトリの作成
CMakeでは，プログラムをビルドする際に，専用のビルドディレクトリを作ります．
buildというディレクトリ名にすることが多いようですが，ターゲットによってディレクトリ名を変更することもよく行われているようです．その際でもbuildという名前は含めるようです．

## tasks.jsonの配置
ソースファイルのあるディレクトリ以下に.vscodeディレクトリを作成し，tasks.jsonを置いてください．

## 設定ファイルの編集
その後，ソースファイルの構成に従って，tasks.jsonを編集します．

|変数名|意味|
|:--|:--|
|`${fileBasename}`|現在開いているファイル名|
|`${fileBasenameNoExtension}`|現在開いているファイル名（拡張子を除く）|
|`${workspaceFolder}`|VSCodeで開いているディレクトリ名（パスを含む）|
|`${workspaceFolderBasename}`|VSCodeで開いているディレクトリ名（パスを含まない）|

### tasks.json

- `PUT-BUILD-DIRECTORY`をビルドディレクトリに書き換えてください．
- `set`で各ビルドのオプションを設定している場合は，cmakeにコンパイルオプションを追加してください．

```JSON
"args": [
    "..",
    "-G",
    "\"Unix Makefiles\"",
    "-DCMAKE_BUILD_TYPE=Debug"
]
```

```JSON
    "tasks": [
        {
            "label": "make project",
            "type": "shell",
            "options": {
                "cwd": "${workspaceRoot}/PUT-BUILD-DIRECTORY"
            },
            "command": "make",
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "presentation": {
                "echo": true,
                "reveal": "always",
                "focus": true,
                "panel": "shared",
                "showReuseMessage": true,
                "clear": false
            },
            "dependsOn":["cmake"]
        },
        {
            "label": "cmake",
            "type": "shell",
            "options": {
                "cwd": "${workspaceRoot}/PUT-BUILD-DIRECTORY"
            },
            "command": "cmake",
            "args": [
                "..",
                "-G",
                "\"Unix Makefiles\""
            ]
        }
    ]
```

#### 例1
main.f90とmodule_hello.f90というソースファイルをコンパイルしてhello.outを作る場合．
ビルドディレクトリは./build，Debugビルドを想定しています．


```
cmake_minimum_required(VERSION 3.10)

enable_language(Fortran)

project(hello Fortran)

set(CMAKE_Fortran_FLAGS_DEBUG "-g -O0 -Wall")
set(CMAKE_Fortran_FLAGS_RELEASE "-O3 -march=native")

add_executable(${PROJECT_NAME}.out
    main.f90
    module_hello.f90
)
```

```JSON
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
        {
            "label": "make project",
            "type": "shell",
            "options": {
                "cwd": "${workspaceRoot}/build"
            },
            "command": "make",
            "args": [
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "presentation": {
                "echo": true,
                "reveal": "always",
                "focus": true,
                "panel": "shared",
                "showReuseMessage": true,
                "clear": false
            },
            "dependsOn":["cmake"]
        },
        {
            "label": "cmake",
            "type": "shell",
            "options": {
                "cwd": "${workspaceRoot}/build"
            },
            "command": "cmake",
            "args": [
                "..",
                "-G",
                "\"Unix Makefiles\"",
                "-DCMAKE_BUILD_TYPE=Debug"
            ]
        }
    ]
}
```
