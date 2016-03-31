---
title: Hexo + GitHub 搭建博客备忘(Mac)
tags:
  - github
  - Hexo
categories: 
  - 技术相关
date: 2016-03-28 22:11:00
---

## Hexo 安装
---------------------------------
hexo 依赖 [node.js](https://nodejs.org/en/), 首先下载安装node.js。然后运行以下命令：

	npm install hexo-cli -g
	hexo init blog
	cd blog
	npm install
	hexo server

之后就可以在
[http://localhost:4000/](http://localhost:4000/)查看效果了。

<!-- more -->
Hexo 的实用[插件](https://hexo.io/plugins/)

	npm install hexo-generator-index --save
    npm install hexo-generator-archive --save
    npm install hexo-generator-category --save
    npm install hexo-generator-tag --save
    npm install hexo-server --save
    npm install hexo-deployer-git --save
    npm install hexo-deployer-heroku --save
    npm install hexo-deployer-rsync --save
    npm install hexo-deployer-openshift --save
    npm install hexo-renderer-marked@0.2 --save
    npm install hexo-renderer-stylus@0.2 --save
    npm install hexo-generator-feed@1 --save
    npm install hexo-generator-sitemap@1 --save
    npm install --save hexo-admin
    
其中 hexo-admin 无法在线上使用，可以在本地编辑完之后部署即可，地址为根目录＋/admin/，具体配置在[这里](https://github.com/jaredly/hexo-admin)

## GitHub 账号申请和新建项目
----------------------------------* 登录到你的[GitHub](http://github.com/)账号
* 选择新建醒目，项目名称需要设置为`your_username.github.io`
* 首次创建需要10分钟左右的审核时间，之后就可以访问了`http://your_username.github.io` (现在访问可能会导致404，因为项目还是空的)

## 部署到线上
----------------------------------
修改 hexo 项目根目录下的站点配置文件`_config.yml`，如果安装了`hexo-deployer-git`插件，这里之接修改如下配置：

	deploy:
    	type: git
        repo: your-repository.git
        branch: master

Hexo 生成命令：
	
    hexo generate

用于将目前项目生成静态文件

Hexo 部署命令
	
    hexo deploy

这条命令会根据你在`_config.yml`里面的配置将生成的静态文件部署到线上，之后就可以通过之前的链接进行访问了。

这两条命令可以简写为: `hexo g -b` 或者 `hexo b -g` 效果相同。

## Hexo主题
----------------------------------
+ Hexo Themes - http://hexo.io/themes/
+ Jacman - http://wsgzao.github.io/post/hexo-jacman/
+ iissnan - http://notes.iissnan.com

## Markdown 教程及编写工具
----------------------------------
+ Markdown语法 - [http://wowubuntu.com/markdown/]( https://www.zybuluo.com/mdeditor)
+ 在线工具 - [https://www.zybuluo.com/mdeditor]( https://www.zybuluo.com/mdeditor)
