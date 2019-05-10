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