---
title: Vim NERDTree cheat sheet
date: 2018-04-13 11:53:54
tags: 
	- Vim
	- NERDTree
categories: 技术相关
---

[NERDTree] 是一个 Vim 插件，提供了树状文件表示，可以方便的检索文件，并且支持多窗口浏览。

<!--more-->
# 安装
[pathogen.vim](https://github.com/tpope/vim-pathogen)
	
	git clone https://github.com/scrooloose/nerdtree.git ~/.vim/bundle/nerdtree

然后重启 Vim 运行 `:helptags ~/.vim/bundle/nerdtree/doc/` 或者 `:Helptags`, 通过 `:help NERDTree.txt` 可查看更多帮助。

[apt-vim](https://github.com/egalpin/apt-vim)
	
	apt-vim install -y https://github.com/scrooloose/nerdtree.git

# 快捷键

有时候一些常用的操作，我们希望通过快捷键来操作，而不是去敲很长的命令。这时候可以通过在 .vimrc 文件中定义来实现。

例如：

通过 Ctrl+n 来打开关闭[NERDTree]侧边栏

	map <C-n> :NERDTreeToggle<CR>


通过 Ctrl+h Ctrl+l 来切换vim tab(使用 [vim-nerdtree-tabs] 插件)

	nnoremap <C-h> :tabprevious<CR>
	nnoremap <C-l> :tabnext<CR>


# vim-nerdtree-tabs

[vim-nerdtree-tabs] 让 vim 支持多tab，方便的编辑多文件

## 安装

Install the plugin through Pathogen:

	cd ~/.vim/bundle
	git clone https://github.com/jistr/vim-nerdtree-tabs.git

Or through Vundle:

	Bundle 'jistr/vim-nerdtree-tabs'

Or through Janus:

	cd ~/.janus
	git clone https://github.com/jistr/vim-nerdtree-tabs.git


# 常用命令

Ctrl+ww： 切换光标位置（主窗口，侧边栏）
Ctrl+w+方向键： 切换光标位置（分裂的窗口之间和侧边栏）

在 [NERDTree] 打开并且光标处于侧边栏时执行：

## 作用于文件

	？: 帮助命令列表
	Enter: toggle 文件夹列表
	o: 在当前窗口打开文件并切换光标到主窗口
	go: 在当前窗口打开文件但不切换光标到主窗口
	t: 新 tab 打开文件
	T: 新 tab 打开文件，但不切换过去
	i: 在当前窗口分裂一个窗口打开并切换光标到主窗口
	gi: 在当前窗口分裂一个窗口打开但不切换光标到主窗口
	s: 在当前窗口垂直分裂一个窗口打开并切换光标到主窗口
	gs: 在当前窗口垂直分裂一个窗口打开但不切换光标到主窗口

## 作用于路径（文件夹）

	o: 展开收起文件夹
	O: 递归打开收起文件夹(非常耗时，不推荐使用)
	x: 关闭父节点文件夹
	X: 关闭所有当前文件夹的子文件夹
	e: 在主窗口打开当前文件夹
	
## 路径切换

	C: 将根路径切换到选中的路径
	u: 将根路径切换到上一级路径
	U: 将根路径切换到上一级路径,但保持当前路径打开

## 路径导航

	P: 跳到 root 路径
	p: 跳到当前路径的父路径
	K: 跳到第一个子节点
	J: 调到最后一个子节点

## 文件筛选

	I: 显示隐藏，隐藏文件（以.开头的文件）
	F: 显示隐藏文件
	
	
## Bookmark

	B: 显示所有 Bookmarks


# 参考

[NERDTree]
[vim-nerdtree-tabs]





[NERDTree]: https://github.com/scrooloose/nerdtree
[vim-nerdtree-tabs]: https://github.com/jistr/vim-nerdtree-tabs