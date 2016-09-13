---
layout: post
title: 微信公众号开发以及微信开发环境注意事项
date: 2016-09-13 17:46:44 +0800
comments: true
categories: 
 - 前端
tags: 
 - 微信公众号
 - WeChat
 - ngnix
 

---

![image](http://7xtaiq.com1.z0.glb.clouddn.com/image/wechat.png?imageMogr2/thumbnail/300000@)  
微信公众号开发其实不难  
但是开发的过程并不顺利，主要还是因为微信对访问的端口有限制（http支持80端口，https支持443端口）  
开发的时候没有想清楚，导致了很多不必要的失误

<!-- more -->

## 接口配置
![image](http://7xtaiq.com1.z0.glb.clouddn.com/image/server_url_conf.png?imageMogr2/thumbnail/300000@)
配置的`url`供微信主动调用  

### `GET` 请求  
微信会验证当前的url是否可用, 验证方法就是向填写的url发送`GET`请求  
若确认此次GET请求来自微信服务器，原样返回echostr参数内容  
如何确认这是一次有效的验证请求呢？  
官方示例代码是PHP，在此贴出JavaScript代码以供参考  

	```
//返回true，表示验证通过
function x_verifyToken(timestamp, nonce, signature) {
    var token = 'meetinisgood';
    var array = [token, timestamp, nonce];
    var key = array.sort ().join ('');
    var sha1 = crypto.createHash('sha1').update (key).digest ('hex');
    // sha1 处理结束
    return sha1 == signature
}
	```

### `POST` 请求  
用户的操作均通过`POST`请求到达我们自己的服务器  
这里需要关心的是用户到底进行了什么操作？  
![image](http://7xtaiq.com1.z0.glb.clouddn.com/image/wechat_event_info.png?imageMogr2/thumbnail/300000@)  
大部分通知都带有以上参数，请阅读相关章节了解上述参数可能的取值  
[接收普通消息](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421140453&token=&lang=zh_CN)  
[接收事件推送](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421140454&token=&lang=zh_CN)  

## access_token 机制
为了保密appsecrect，第三方需要一个access_token获取和刷新的中控服务器  
我们会设置菜单按钮或向用户推送消息等等，这些都必须要access_token  
access_token 有一个特点就是，**一旦重新获取access_token，原来的access_token就失效**  
这引出了一个问题:  
&emsp;&emsp;正式环境和开发环境如何分别使用access_token?  
  
### 正式环境和开发环境分别使用access_token
微信提供了一个工具：
![image](http://7xtaiq.com1.z0.glb.clouddn.com/image/wechat_test_account.png?imageMogr2/thumbnail/300000@)
这就意味着你拥有两个公众号，而且该测试公众号的接口权限全开  
正式环境使用正式的公众号，开发环境使用测试公众号  
但是这里还有一个问题：  
&emsp;&emsp;微信只访问80端口（http）和443端口（https），如果正式环境和测试环境使用了不同的端口，如何把属于开发环境的微信请求从80端口转到8080端口（开发环境端口）呢？  

### 把属于开发环境的微信请求从80端口转到8080端口  
这里就需要后台同学的支持了  

```
Nate：同学，帮我搞一个端口转发呗
Arden：转发规则是什么嘛
Nate：80端口 /dev_wechat 下的请求转发到8080端口下的 /wechat，谢谢哈
Arden：说馁些，不存在，没得似的
```
后台同学配置好端口转发之后，我们还需要做什么工作才能让开发环境收到来自微信的问候（token 验证）呢？  
1. 正式公众号的服务器URL配置成：http://xxx.com/wechat  
2. 测试公众号的服务器URL配置成：http://xxx.com/dev_wechat  
3. 下面还会提到`wechat`和`dev_wechat`  
这样开发环境就可以收到来自微信的问候了~~~  

----

这样就够了吗？ NO~  
被坑过的我告诉你还有一项需要注意  
为了让html页面可以拿到用户的wx_openid，需要以下步骤：  

1. 用户访问HTML页面，这次访问的url: `http://xxx.com/aaa/bb.html?name=xxx`
2. 没有发现参数 wx_openid
3. 重定向到 `http://xxx.com/wechat/request_openid`  
携带参数  redirect_url=`URLEncoding('http://xxx.com/aaa/bb.html?name=xxx')`
2. 组装一个回调url：redirect_url_1 = `URLEncoding('http://xxx.com/wechat/code?redirect=URLEncoding(redirect_url)')`
2. 重定向到微信请求code（传入一个回调url：redirect_url_1）  
2. 微信将code发送到redirect_url_1
3. 可以取到参数`redirect_url`  
向微信发送http请求，传入code    
异步返回用户的信息（openid）  
再次重定向到用户原来访问的url: `redirect_url`（追加了参数wx_openid）
  

**看图理解**  
![image](http://7xtaiq.com1.z0.glb.clouddn.com/image/wechat_request_openid.png?imageMogr2/thumbnail/300000@)  
说了一大堆获取wx_openid的步骤，就是为了指出其中一个容易搞混的地方  
注意图中 `绿色的wechat`  
正式环境使用 `wechat`， 开发环境使用`dev_wechat`（已经说过 wechat 和 dev_wechat 是什么啦）  
好了，就这样。
