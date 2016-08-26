---
title: 将python 脚本制作成命令
date: 2016-08-25 16:55:18
tags:
	- python
categories: 技术相关
---

## 描述

有时候写了一个 python 脚本来执行某些操作， 但又不希望每次都敲 python ... 来执行脚本，下面的方法就是将脚本制作成命令。这样就方便很多。


<!--more-->

## 步骤

### 编写脚本

以我写的统计代码行数的脚本为例

```py
#!/usr/bin/env python
import fnmatch
import os
import sys


def count_one_file(filename):
       	with open(filename, 'r') as f:
       		return len(f.readlines())



def count_files(path, ftype):
       	res = {}
       	for ft in ftype:
       		ans = 0
       		for root, dirnames, filenames in os.walk(path):
       		    for filename in fnmatch.filter(filenames, '*.'+ft):
       		        ans += count_one_file(os.path.join(root, filename))
       		res.update({ft: ans})
       	return res


def usage():
       	print 'ccode folder_name file_type(py, m, h.....)\n'
       	print 'example:\n'
       	print 'ccode examplefolder py'

if __name__ == '__main__':
       	try:
       		path = sys.argv[1]
       		ftype = sys.argv[2:]
       		print count_files(path, ftype)
       	except:
       		usage()
```

注意要在代码第一行加上 #!/usr/bin/env python，目的是为了告诉终端使用什么程序来执行代码。你可以在代码里面接受命令行参数，实现更复杂的操作。

### 查看你的可执行路径

运行以下命令：

	echo $PATH

即可查看你的 PATH 路径，在其中一个目录下运行

	ln -s /path/to/yourscript command_name

第一个路径参数是你的脚本的实际存储位置，command_name 是你自定义的命令名称，之后你就可以在任何目录下运行这个命令了:)





