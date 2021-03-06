---
title: Angular笔记 
tags: Angular,TypeScript,前端框架
grammar_cjkRuby: true
---

## Angular程序架构
![Angular程序架构](./images/Angular程序架构.PNG)

## 环境搭建

 1. 安装node.js

	测试：在命令行中输入：

	``` shell
	ndoe -v	#查看node.js版本
	npm -v	#查看npm版本
	```
	
 2. 设置淘宝镜像

	``` shell
	npm config set registry  https://registry.npm.taobao.org
	```
	
 3. 安装angular命令行工具

	``` shell
	npm install -g @angular/cli@指定版本
	#测试，出现版本信息及安装成功
	ng -v
	```

 4. 安装WebStorm

 5. 创建项目

	``` shell
	#auction为项目名
	ng new auction
	```

 6. 安装第三发插件，如jQuery、bootstrap
	6.1 安装jquery、bootstrap

			npm install -g jquery --save
			npm install -g bootstrap --save
			
	6.2 在.angular-cli.json文件中引入jquery和bootstrap
			
			"styles": [
				"styles.css",
				"../node_modules/bootstrap/dist/css/bootstrap.css"
			],
			"scripts": [
				"../node_modules/jquery/dist/jquery.js",
				"../node_modules/bootstrap/dist/js/bootstrap.js"
			 ]
			 
	 6.3 安装类型描述文件

			npm install -g @types/jquery --save-dev
			npm insatll -g @types/bootstrap --save-dev

## 组件的概念
![Angular组件](./images/Angular组件_1.PNG)
