# 移动互联网安全第八章实验报告  
## Android 缺陷应用漏洞攻击实验  
### 实验目的  
* 理解 Android 经典的组件安全和数据安全相关代码缺陷原理和漏洞利用方法  
* 掌握 Android 模拟器运行环境搭建和`ADB`使用  
### 实验环境  
* [Android-InsecureBankv2](https://github.com/c4pr1c3/Android-InsecureBankv2)  
* python 2.7.18  
* Android Studio 4.1.2  
* Android 11.0 API 30 x86 - Pixel  
### 实验要求  
 - [x] 详细记录实验环境搭建过程  
 - [x] 至少完成以下实验：  
    * Developer Backdoor  
    * Insecure Logging  
    * Android Application patching + Weak Auth  
    * Exploiting Android Broadcast Receivers  
    * Exploiting Android Content Provider    

### 实验过程  
#### 实验环境搭建  
* `git clone git@github.com:c4pr1c3/Android-InsecureBankv2.git` 克隆仓库到本地  
![克隆仓库](./image/克隆仓库.png)  
* 配置`python 2.7`环境  
`Anaconda`添加`newone`环境配置`python 2.7`  
![Anaconda](./image/Anaconda.png)  
![环境证明](./image/环境证明.png)  
* `pip install -r requirements.txt` 安装必备软件  
![安装软件](./image/安装软件.png)  
* `python app.py` 尝试运行服务器  
![运行服务器](./image/运行服务器.png)  
* `adb install InsecureBankv2.apk` 在ADV上安装APK  
![安装apk](./image/安装apk.png)  
* 进行已知用户名密码登录尝试  
![登录尝试](./image/登录尝试.png)  
#### Developer Backdoor  
* 根据题目`开发者后门`及黄大提示，在代码中寻找答案  
![开发者后门](./image/开发者后门.png)  
* 由代码可知，当用户名为`devadmin`时，可直接登录  
![开发者后门](./image/开发者后门.gif)  
* tips：今天主动留下的后门就是明天悔不当初的泪水  
#### Insecure Logging  
* 根据题目`不安全的日志记录`想到依靠日志  
`adb logcat > 1.txt` 抓取`Jack`登录日志  
![抓取日志](./image/抓取日志.png)  
* 分析日志  
![明文日志](./image/明文日志.png)  
* 尝试更改密码操作  
![修改密码](./image/修改密码.png)  
* tips：全明文存储的日志只能被比喻为“疯狂裸奔”  
#### Android Application patching + Weak Auth  
* VS Code安装`APKlab`扩展实现解码  
![apklab解码](./image/apklab解码.png)  
* 更改`res/values/strings.xml`配置  
![更改配置](./image/更改isadmin.png)  
* 重新编译生成APK文件  
![重新编译生成APK文件](./image/重新编译apk.png)  
* 对新签名的APK重新安装结果展示  
![重新安装结果展示](./image/结果展示.png)  
#### Exploiting Android Broadcast Receivers  
* 查看反编译后的`AndroidManifest.xml`  
![查看manifest](./image/查看manifest.png)  
* 安装`dex2jar-2.0`和`jadx`  
![安装工具](./image/安装工具.png)  
* 解压缩`InsecureBankv2.apk`  
![解压缩](./image/解压缩.png)  
* 将`classes.dex`移至`dex2jar`  
* `d2j-dex2jar.bat classes.dex` 完成格式转换  
![完成格式转换](./image/完成格式转换.png)  
* 使用`JADX-GUI`打开`classes-dex2jar.jar`文件，查找对应源代码  
![changepass](./image/changepass.png)  
![mybrocast](./image/mybrocast.png)  
* `adb shell`执行命令`am broadcast -a theBroadcast -n com.android.insecurebankv2/com.android.insecurebankv2.MyBroadCastReceiver --es phonenumber 5554 –es newpass`  
![绕过登陆操作](./image/绕过登陆操作.png)  
#### Exploiting Android Content Provider  
* 使用`devadmin`和`jack/Jack@123$`分别登录  
* 查看反编译后的`AndroidManifest.xml`  
![track截图](./image/track截图.png)  
* 使用`JADX-GUI`打开`classes-dex2jar.jar`文件，查找对应源代码  
![trackusers](./image/trackusers.png)  
* `adb shell`执行命令`content query --uri content://com.android.insecurebankv2.TrackUserContentProvider/trackerusers`  
![命令执行结果](./image/命令执行结果.png)  

### 问题与解决  
1. 直接下载安装`python2.7`版本并配置环境变量后报错  
![配置环境变量](./image/配置环境变量.png)  
错误原因：之前安装`Anaconda`后设置`python 3.8`为主环境，配置`2.7`版本时应考虑`python2`和`python3`的问题  
解决方案：在`Anaconda`设置`python2.7`新环境使用  
2. 重新编译`APK`文件后重新安装并没有发生更改  
错误原因：傻憨憨没有经过重新签名，在这里卡了半天不知如何下手:joy:  
解决方案：新生成的`APK`文件需要重新签名后再使用  
#### 参考资料  
[移动互联网安全第八章实验](https://c4pr1c3.github.io/cuc-mis/chap0x08/main.html)  
[移动互联网安全第八章教学视频](https://www.bilibili.com/video/BV1rr4y1A7nz?from=search&seid=6142859782746666446)  
[Anaconda介绍、安装及使用教程](https://www.jianshu.com/p/62f155eb6ac5)  
[Kali Linux 逆向工程工具 d2j-dex2ja 教程](https://www.fujieace.com/kali-linux/courses/d2j-dex2ja.html)