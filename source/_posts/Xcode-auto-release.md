---
title: Xcode auto release
date: 2016-06-04 18:25:05
tags:
 - Xcode
 - automator
 - appscript
 - shell script
 
---

曾使用过Xcode上传release包，平均耗时 1h，全程人工参与  
曾使用过Application Loader上传release包，平均耗时 20min，全程人工参与  
现在使用automator制作的自动传包工具AutoLoader，平均耗时 10min，人工参与2s

![image](http://7xtaiq.com1.z0.glb.clouddn.com/image/xcode-service.png?imageMogr2/thumbnail/100000@)

<!-- more -->

## 自动打包
网上已经有许多开发者共享的打包脚本，只要输入一个命令就可以自动打包、发布、邮件通知。  
但是最近发现，一个简单的方法，可以让我们变得更懒一些。

## 预备知识
这里不会专门讲解以下知识（因为你不需要这些知识也可以使用本文提供的工具），请自行了解  
1. 使用 automator [Automator for Mac OS X: Tutorial and Examples](https://www.raywenderlich.com/58986/automator-for-mac-tutorial-and-examples)  
2. 使用 shell script [Shell脚本编程30分钟入门](https://github.com/qinjx/30min_guides/blob/master/shell.md)  
3. 使用 applescript [Introduction to AppleScript Language Guide](https://developer.apple.com/library/mac/documentation/AppleScript/Conceptual/AppleScriptLangGuide/introduction/ASLR_intro.html)  
4. xcodebuild [xcodebuild 手册](https://developer.apple.com/legacy/library/documentation/Darwin/Reference/ManPages/man1/xcodebuild.1.html)  
5. Application Loader altool [altool 手册](http://help.apple.com/itc/apploader/#/apdATD1E53-D1E1A1303-D1E53A1126)

## 使用xcodebuild打包

**xcodebuild**  
打包、导出ipa都是使用的 xcodebuild  
可能你的工程需要指定 target，或者需要其他参数才能打出正确的包，了解一下xcodebuild还是很有必要的  
代码中用到的exportOptionsPlist是一个plis配置文件，稍后可以下载到

````
#xcodebuild 简单示例
#build clean
xcodebuild clean -configuration "$configuration" -alltargets

#archive
xcodebuild archive -workspace "$workspaceName" -scheme "$scheme" -configuration "$configuration" -archivePath ${archivePath}/${archiveName}.xcarchive CODE_SIGN_IDENTITY="$codeSignIdentity" PROVISIONING_PROFILE="$appStoreProvisioningProfile"

#导出到ipa
xcodebuild -exportArchive -archivePath ${archivePath}/${archiveName}.xcarchive -exportOptionsPlist "$exportOptionsPlist" -exportPath ${archivePath}/${archiveName}
````

## 使用altool提交ipa
**Application Loader altool**  
altool 位于 Application Loader，三个参数 ipapath、appleid、password  
在这之前我都是使用的Application Loader上传包到 iTunesConnect

````
#altool 简单示例
#validate
"$altoolPath" --validate-app -f "$ipaPath" -u "$appleid" -p "$applepassword" -t ios --output-format xml

#upload
"$altoolPath" --upload-app -f "$ipaPath" -u "$appleid" -p "$applepassword" -t ios --output-format xml
````
## 使用 Automator
Automator创建的workflow可以和其他应用程序无缝衔接，在【服务】选项里面可以找到自己为某个目标定制的workflow。  
使用Automator为xcode定制一个【服务】很简单，所以并没有考虑为xcode开发一个插件。  

* **打开Automator**
* **新建一个service**  
![image](http://7xtaiq.com1.z0.glb.clouddn.com/image/automator-new.png?imageMogr2/thumbnail/100000@)
* **修改服务需要的输入、选取需要服务的应用**  
![image](http://7xtaiq.com1.z0.glb.clouddn.com/image/automator-edit1.png?imageView2/2/w/600)
* **加入流程一**  
  Automator应用界面左侧，打开资料库面板，点击【变量】，在搜索框中输入“path”，选择目标路径变量，拖拽到中间区域
![image](http://7xtaiq.com1.z0.glb.clouddn.com/image/automator-edit2.png?imageMogr2/thumbnail/100000@)  
拖拽到中间区域  
![image](http://7xtaiq.com1.z0.glb.clouddn.com/image/automator-edit3.png?imageMogr2/thumbnail/100000@)
* **选取项目工程根路径**  
  在 Automator 应用界面的下方，双击刚才【Destination Path】，在弹出面板中选择工程路径，选择后点击【完成】
![image](http://7xtaiq.com1.z0.glb.clouddn.com/image/automator-edit4.png?imageMogr2/thumbnail/100000@)
* **加入流程二**  
  automator应用界面左侧，打开资料库面板，点击【操作】，在搜索框中输入“applescript”，选择运行AppleScript，拖拽到中间区域步骤1的下方  
![image](http://7xtaiq.com1.z0.glb.clouddn.com/image/automator-edit5.png?imageMogr2/thumbnail/100000@)  
拖拽到中间区域  
![image](http://7xtaiq.com1.z0.glb.clouddn.com/image/automator-edit6.png?imageMogr2/thumbnail/100000@)
* **编辑 Applescript**  
  在 代码框中输入以下内容  
  
  ````
  on run {input, parameters}		tell application "Terminal"		activate		do script "cd " & input & " && . BuildScript/xcode-archive-release.sh"	end tell		return input  end run
  ````
  脚本文件 xcode-archive-release.sh 我放到了工程目录的 BuildScript 文件夹里面  
  这段 Applescript 脚本做的事就是 打开 Terminal，cd 到工程目录，运行BuildScript文件夹里面的脚本文件
* **最终看起来是这个样子的**  
![image](http://7xtaiq.com1.z0.glb.clouddn.com/image/automator-appstore.png?imageMogr2/thumbnail/100000@)

* **最后，保存文件，输入文件名字就可以了。打开xcode，看看是否有了新的 service 选项**

## 资源下载
* [上传到 iTunesconnect shell 脚本](http://7xtdgs.com1.z0.glb.clouddn.com/BuildScript.zip)
* [Automator workflow文件](http://7xtdgs.com1.z0.glb.clouddn.com/%E5%8F%91%E5%B8%83%E8%87%B3AppStore.workflow.zip)
* 下载的脚本文件解压后，放到项目里，与 xxx.xcworkspace（xxx.xcodeproj） 同一级
* 修改 xcode-archive-release.sh 里面的配置（见**配置修改**）
* 修改 release_exportOptions.plist 里面的 teamID
* 下载下来的 workflow 文件不用解压，用Automator打开，修改工程目录（参考**使用 Automator**）

## 配置修改
打开 Terminal ,cd 到工程目录下，运行命令`xcodebuild -list`  

````
$:xcodebuild -list
Targets:
        XXXXXX
        XXXXXXTests

    Build Configurations:
        Debug
        Release

    If no build configuration is specified and -scheme is not passed then "Release" is used.

    Schemes:
        XXXXXX
````

* **archiveName**  
打包 archive 文件的名字（不用后缀）  

* **workspaceName**  
工程文件的名字（需要后缀）  

* **scheme**  
运行命令`xcodebuild -list`得到的`Schemes`，可能是一个列表，选一个你需要的  

* **codeSignIdentity**  
形如：iPhone Distribution: xxxx Inc. (xxxxxxx)  
进入`Build Settings`, 编辑`Code Signing identity`成打release包需要的，再点击一下刚才选中的选项，弹出框中点击`Other`就可以获得形如`iPhone Distribution: xxxx Inc. (xxxxxxx)`的数据  

* **appStoreProvisioningProfile**  
形如：xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxx  
获取方法与上面类似，最终获得形如`xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxx`的数据  

* **configuration**  
值为：Release  
运行命令`xcodebuild -list`得到的`Build Configurations`，有两个选项，打release包，所以值为Release  

* **exportOptionsPlist**  
存放release_exportOptions.plist文件的相对路径  
如果下载到的BuildScript文件夹存放到项目根目录，其值为`BuildScript/release_exportOptions.plist`  

* **ipaPath**  
导出ipa文件的路径  
altool工具回到该路径获取需要上传的ipa文件，其值应为`${PWD}/build/${archiveName}/${scheme}.ipa`  

* **appleid**  
开发者账号（邮箱）  

* **applepassword**  
开发者账号的密码

## 最后
本文讲述了Automator工具创建自动化流程，这仅仅是Automator的一部分。  
善用Automator，工作会更轻松。