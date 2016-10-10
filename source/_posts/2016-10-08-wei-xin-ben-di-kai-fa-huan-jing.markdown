---
layout: post
title: 微信本地开发环境
date: 2016-10-08 16:04:29 +0800
comments: true
categories: 
 - 教程
tags: 
 - 微信
 - ngrok

---

借助ngrok实现微信本地开发
<!-- more -->

## ngrok的安装和使用

请到[这里](https://ngrok.com/download)下载安装，查阅[文档](https://ngrok.com/docs#config-location)  
以Mac为例：  

1. 将下载的ngrok文件copy到 `/usr/local/bin/` 目录下   
	windows拷到C:\Windows\System32目录下

2. 在以下位置创建ngrok.yml 文件

	```
	OS X		/Users/example/.ngrok2/ngrok.yml
	Linux		/home/example/.ngrok2/ngrok.yml
	Windows		C:\Users\example\.ngrok2\ngrok.yml
	```  
	windows可用命令md .ngrok2创建点开头的文件夹

3. 修改ngrok.yml内容为

	```
	tunnels:
	    meetin_service:
	        proto: http
	        inspect: false
	        addr: 8080
	```
## 微信开发环境配置
1. 获得自己的[测试公众号](https://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=sandbox/login)
2. 修改接口配置信息(具体的数值请询问公众号管理员)

	```
	URL		xxxxx
	Token	xxxxx
	```
3. 设置权限表中->网页服务->网页账号->网页授权获取用户基本信息的回调域名为 `www.meetin.co`
4. 打开工程 `meetin_web_node`，找到文件 `global_local.js`
5. `global_local.js`文件下方，加入自己的逻辑
	
	```
	if (process.env.USER == 'gukong') {
 	    configure.wx_appid = 'xxxxxx';//这是开发公众号
    	configure.wx_secret = 'xxxxxxx';
	    configure.wx_self = 'xxxxxx';//测试公众号的微信号
	}
	else if (process.env.USER == 'example') {
		configure.wx_appid = 'xxx';
    	configure.wx_secret = 'xxx';
	    configure.wx_self = 'xxx';
	}
	else {
	    configure.wx_appid = '';
    	configure.wx_secret = '';
	}
	```
6. 配置环境变量`USER`为自己的名字
7. 在 `app.js` 文件启动的 `start_ngrok.js`，只有在本地开发环境才会启动，ngrok服务启动后，会修改`gConfig.host`为获取到的url，分享的会议可以直接扫码打开（微信授权成功）

	```
	try to get public url 1
	try to get public url 2
	http://8e422a4e.ngrok.io
	nate-log postAsync:xxxxxxxxxxxxxxxxxx
	nate-log postAsync error null
	nate-log body: success
	success
	```
8. 微信本地开发，只能与自己的测试公众号交互

## 原理

![image](http://7xtaiq.com1.z0.glb.clouddn.com/image/wechat_dev.png?imageMogr2/thumbnail/300000@)
