---
layout: post
title: "实践Visual Studio Code"
date: 2019-05-08 09:25:00 +0800
categories: 工具
tags: tool IDE "Visual Studio Code"
---

[Visual Studio Code](https://code.visualstudio.com/) Code editing.**Redefined.** Free. Built on open source. Runs everywhere.

Font Size: 12

Font Family: Source Code Pro

```shell
# 进入项目目录后启动VS Code
$ code .
```



一般以folder为工作单位，其下会生成一个.vscode的隐藏目录，存储相关的vscode的一些配置信息。一般可以理解为项目。

workspace可以理解为包含多个目录的一种组织结构，可以保存起来复用。其实就是一个文件，里面记录了包含的目录清单，例如：test.code-workspace

```json
{
	"folders": [
		{
			"path": "/media/ray/Resource/Workspace/practice/maven-sample"
		},
		{
			"path": "/media/ray/Resource/Workspace/practice/boss-demo1"
		}
	],
	"settings": {}
}
```



| 快捷键       | 功能            |      |
| ------------ | --------------- | ---- |
| Ctrl+Shift+p | Command Palette |      |
|              |                 |      |
|              |                 |      |

 

默认单击文件会打开文件（预览），但是是在同一个tab中，会覆盖之前预览的文件。如果不希望覆盖之前的tab，而是在新的tab中打开，可以不要勾选`Workbench > Editor: Enable Preview`，对应的配置参数：`"workbench.editor.enablePreview": false`



## python

支持virtualenv

### debug

```json
"configurations": [
        {
            "name": "Python: Current File",
            "type": "python",
            "request": "launch",
            "program": "${workspaceFolder}/hello.py",
            "console": "integratedTerminal"
        }
    ]
```

#### program

待调试的文件

${file}：当前打开的文件

${workspaceFolder}：当前工作空间根目录

${fileDirname}：当前打开的文件所在目录

## Go

go extension会要求安装一些包（gocode，gopkgs，go-outline，go-symbols，guru，gorename，dlv，gocode-gomod，godef，goreturns和golin），由于网络原因（golang.org访问不通），不能全部安装成功。

手工下载包的源码

```shell
$ git clone git@github.com:mdempsky/gocode.git
$ git clone git@github.com:go-delve/delve.git
$ git clone git@github.com:sqs/goreturns.git
$ git clone git@github.com:ramya-rao-a/go-outline.git
$ git clone git@github.com:acroca/go-symbols.git
$ git clone git@github.com/uudashr/gopkgs/cmd/gopkgs
$ git clone git@github.com/rogpeppe/godef

# gocode-gomod
$ git clone github.com/stamblerre/gocode

# golang相关的项目拷贝到golang.org/x
$ git clone https://github.com/golang/tools.git
$ git clone https://github.com/golang/lint.git
```

手工安装

```shell
$ go install github.com/mdempsky/gocode
$ go install github.com/go-delve/delve/cmd/dlv
$ go install github.com/sqs/goreturns
$ go install github.com/ramya-rao-a/go-outline
$ go install github.com/acroca/go-symbols
$ go install github.com/uudashr/gopkgs/cmd/gopkgs
$ go install github.com/rogpeppe/godef

# gocode-gomod
$ go build -o gocode-gomod github.com/stamblerre/gocode
$ mv gocode-gomod $GOPATH/bin/

$ go install golang.org/x/tools/cmd/guru
$ go install golang.org/x/tools/cmd/gorename
$ go install golang.org/x/lint/golint
```

*备注：下载源码后，如果手工install，则无法debug，如果自动安装则可以执行debug*

### debug

```json
"configurations": [
	   {
            "name": "Launch workspaceFolder",
            "type": "go",
            "request": "launch",
            "mode": "debug",
            "program": "${workspaceFolder}"
        },
        {
            "name": "Launch file",
            "type": "go",
            "request": "launch",
            "mode": "debug",
            "program": "${file}"
        },
        {
            "name": "Launch fileDirname",
            "type": "go",
            "request": "launch",
            "mode": "auto",
            "program": "${fileDirname}",
            "env": {},
            "args": []
        }
]
```

