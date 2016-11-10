---
title: 增加 python 包搜索路径(Mac系统，或类Unix系统)
date: 2016-04-20 16:42:11
tags:
	- python
	- 小问题
categories: 技术相关
---

## 描述

有的时候我们需要引用某一个目录下的某些 py 文件，但是又不希望将该目录放入 site-packages 下面。
可以通过下面方法将目录加入 python 搜索路径

	cd ~

切换到 home 目录下，查看当前目录下的文件， 应该会看到一个 .bashrc 文件(根据所使用的 shell 不同名称会有不同，比如我使用的是 zash，下面的文件是 .zshrc)

<!--more-->

修改此文件

	vim .bashrc


在里面加上如下行：

	export PYTHONPATH="${PYTHONPATH}:/your/path"

退出 vim 后运行，使其生效

	source .bashrc


