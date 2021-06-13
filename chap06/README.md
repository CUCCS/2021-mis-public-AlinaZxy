# 移动互联网安全第六章实验报告  
## 安卓系统访问控制策略与机制  
### 实验目的  
* 了解借助`Android Studio`进行安卓程序开发的基本流程与具体操作  
### 实验环境  
* Android Studio 4.1.2  
* Android 11.0 API 30 x86 - Pixel  
![AVD环境](./image/AVD环境.png)  
### 实验要求  
 - [x] ADB实验  
 - [x] Hello World v1实验  
 - [x] Hello World v2实验  
 - [x] 课后思考  
### 实验过程  
#### ADB实验  
* 添加环境变量  
![添加环境变量](./image/环境变量.png)  
* 命令行  
    ```
    # 查看开启的模拟器
    adb devices
    # 连接模拟器终端
    adb -s emulator-5554 shell
    # 输出环境变量
    echo $PATH
    # 查看系统版本，lsb_release -a 不可用
    uname -a
    # 查看当前目录下文件
    ls
    # 查看防火墙规则
    iptables -nL
    # 查看网络信息
    ifconfig
    # 安装应用
    adb install path_to_apk
    ```  
    ![命令行](./image/命令行.png)  
* 命令行`pull`与`push`  
    ```
    # 将文件复制到设备
    #adb pull remote local
    # 从设备复制文件
    #adb push local remote
    ```  
    ![pull和push](./image/pull和push.png)  
* Activity Manager 实例  
在Android中，除了从界面上启动程序之外，还可以从命令行启动程序，使用的是命令行工具am(activity manager)[教材am命令详解](https://c4pr1c3.github.io/cuc-mis/chap0x06/exp.html)  
    ```
    # Camera（照相机）的启动方法为:
    am start -n com.android.camera/com.android.camera.Camera
    # Browser（浏览器）的启动方法为：
    am start -n com.android.browser/com.android.browser.BrowserActivity
    # 启动浏览器 :
    am start -a android.intent.action.VIEW -d  http://www.cuc.edu.cn/
    # 拨打电话 :
    am start -a android.intent.action.CALL -d tel:10086
    # 发短信：
    adb shell am start -a android.intent.action.SENDTO -d sms:10086 --es sms_body ye --ez exit_on_sent true
    ```  
    ![record1](./image/record1.gif)  
* 软件包管理器 (pm)  
在`adb shell`中，可以使用软件包管理器(pm)工具发出命令，以对设备上安装的应用软件包进行操作和查询[pm命令详解](https://developer.android.com/studio/command-line/adb.html#pm)  
    ```
    # 输出系统中的所有用户
    pm list users
    # 显示第三方软件包
    pm list packages -3
    ```  
    ![pm命令](./image/pm命令.png)  
* 其他adb实验  
    ```
    # 常用的按键对应的KEY_CODE
    adb shell input keyevent 22 //焦点去到发送键
    adb shell input keyevent 66 //回车按下

    adb shell input keyevent 4 // 物理返回键
    adb shell input keyevent 3 // 物理HOME键
    ```  
#### Hello World v1实验  
根据[官方文档——创建 Android 项目](https://developer.android.google.cn/training/basics/firstapp/creating-project.html)完成Hello World v1实验  
* 创建空项目时，Name设置为：MISDemo，Company Domain设置为：cuc.edu.cn  
* 完成构建简单的界面  
![完成构建简单的界面](./image/结果实例1.png)  
* 完成 Hello World v1实验  
![完成实验1](./image/完成实验1.png)  
#### 课后思考  
* 按照向导创建的工程在模拟器里运行成功的前提下，生成的APK文件在哪儿保存的？  
![apk保存位置](./image/apk保存位置.png)  
* 使用`adb shell`是否可以绕过`MainActivity`页面直接“唤起”第二个`DisplayMessageActivity`页面？是否可以在直接唤起的这个`DisplayMessageActivity`页面上显示自定义的一段文字，比如：你好移动互联网安全  
可以  
`adb -s emulator-5554 shell am start -n cuc.edu.cn/cuc.edu.cn.DisplayMessageActivity --es "cuc.edu.cn.MESSAGE" "hello mobile"`  
![hellomobile](./image/hellomobile.png)  
* 如何实现在真机上运行你开发的这个Hello World程序？  
连接真机后选择真机并运行程序即可  
![真机运行](./image/真机运行.png)  
* 如何修改代码实现通过`adb shell am start -a android.intent.action.VIEW -d http://sec.cuc.edu.cn/`可以让我们的`cuc.edu.cn.misdemo`程序出现在“用于打开浏览器的应用程序选择列表”？  
    ```
    # 在AndroidManifest.xml中添加代码
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="http" />
    <data android:scheme="https" />
    ```  
* 如何修改应用程序默认图标？  
在`res/mipmap`下新建`Image Assert`，可以更换图标  
![更改图标](./image/更改图标.png)  
* 如何修改代码使得应用程序图标在手机主屏幕上实现隐藏？  
    ```
    # 在MainActivity.java中添加代码
    PackageManager packageManager = getPackageManager();
    ComponentName componentName = new ComponentName(MainActivity.this, MainActivity.class);
    packageManager.setComponentEnabledSetting(componentName,
    PackageManager.COMPONENT_ENABLED_STATE_DISABLED, PackageManager.DONT_KILL_APP);
    ```  
#### Hello World v2实验  
在v1基础之上，增加以下新功能来为后续的程序逆向和组件安全实验做一些“标靶”  
* 使用`SharedPreferences`持久化存储小数据并按需读取  
* 实现一个简单的注册码校验功能  

实验参考资料：[移动互联网安全课本](https://c4pr1c3.github.io/cuc-mis/chap0x06/exp.html#hello-world-v2)  
#### 课后思考  
* `DisplayMessageActivity.java`中的2行打印日志语句是否有风险？  
我认为是存在一定风险的，因为我们可以明显看到他**明文**输出哈希值的前四位，肯定会给破解带来一定便利性  
* `SharedPreferences`类在进行读写操作时设置的`Context.MODE_PRIVATE`参数有何作用和意义？还有其他可选参数取值吗？  
    * MODE_PRIVATE：默认操作模式，代表该文件是私有数据，只能被应用本身访问，在该模式下，写入的内容会覆盖原文件的内容  
    * MODE_APPEND：会检查文件是否存在，存在就往文件继续追加内容  
    * MODE_WORLD_READABLE：表示当前文件可以被其他应用读取  
    * MODE_WORLD_WRITEABLE：表示当前文件可以被其他应用写入  
### 实验问题与解决  
1. 按照官方文档给出的代码完全复制粘贴遇到报错  
![报错无法解决](./image/报错无法解决.png)  
错误原因：Cannot resolve symbol 'EditText'  
解决方案：按照提示信息更改变量名称  
#### 参考资料  
[移动互联网安全第六章实验](https://c4pr1c3.github.io/cuc-mis/chap0x06/exp.html#hello-world-v2)  
[Android Studio开发者文档](https://developer.android.google.cn/training/basics/firstapp/starting-activity#java)  
[移动互联网安全第六章教学视频](https://www.bilibili.com/video/BV1rr4y1A7nz?p=125&spm_id_from=pageDriver)  
[android 的四种枚举Context.MODE_PRIVATE](https://www.cnblogs.com/nucdy/p/5094297.html)