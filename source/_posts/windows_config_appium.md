title:  Windows 10系统的 Appium-Python-Android 配置和试运行记录
date: 2019-07-19 06:31:00
tags:
- Android
- Appium

photos:	
categories:
- 环境配置
toc: true
comment: true
---

# Windows 10系统的 Appium-Python-Android 配置和试运行记录

## 1. 环境配置
需要的环境如下： 
- jdk8
- Android-sdk
- Python
- Nodejs
- Appium
- Python lib: Appium-Python-Client
- Python IDE: PyCharm
- Git
- Cmake
- opencv4nodejs
- mjpeg-consumer
- ffmpeg
- bundletool.jar

### 1.1 JDK8
#### 1.1.1 到oracle 官网下载
[JDK8官网下载](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
选择对应版本安装即可

#### 1.1.2 环境变量配置
1. 进入 此电脑-配置-高级系统配置-环境变量
2. 配置全局环境变量(下面的部分)
3. 配置JAVA_HOME 变量，将路径设为C:\xxx\xxx\xx\jdk1.8_xxxx(jdk 安装路径)
4. Path中增加两个条目%JAVA_HOME%\bin和%JAVA_HOME%\jre\bin
5. 在命令行输入java/javac来判断是否配置成功

### 1.2 Android SDK
安装Android Studio即可，[下载地址](https://developer.android.com/studio/)

### 1.3 Python
到[Python官网](https://www.python.org/) 下载指定版本安装即可(2.7不要用3+)

### 1.4 NodeJS
到[nodejs官网](https://nodejs.org/en/download/) 
最好下载10，LTS版本，12对于后面要安装的模块无法兼容

**注意** : 用户的环境变量会自动添加npm，需要手动在系统环境变量里面添加npm

### 1.5 Appium 安装
执行以下命令即可

```bash
npm install -g appium
npm install -g appium-doctor
```
appium-doctor 用于检测环境是否符合Appium的要求

### 1.6 Python lib: Appium-Python-Client

```
pip install Appium-Python-Client
pip install selenium==3.0.2
```

### 1.7 PyCharm 安装

Jetbrains 官网安装, Community版本即可，有教育邮箱可以申请Pro

### 1.8 Git 安装
官网下载即可, 勾选加入Path

### 1.9 cmake 安装
官网下载, 安装勾选加入Path

### 1.10 opencv4nodejs 安装

```
npm install -g opencv4nodejs --registry=https://registry.npm.taobao.org
```

### 1.11 ffmpeg安装
官网下载build, 然后把bin目录添加到Path

### 1.12 mjpeg-consumer 安装

```
npm install -g mjpeg-consumer --registry=https://registry.npm.taobao.org
```

### 1.13 bundletool.jar 安装

[release链接](https://github.com/google/bundletool/releases)


## 2. 手机连接usb调试

### 2.1 adb连接
将手机连上PC，CMD中输入:   

```
adb devices
adb shell
```

如果出现以下信息:
> error: device unauthorized.
> This adb server's $ADB_VENDOR_KEYS is not set
> Try 'adb kill-server' if that seems wrong.
> Otherwise check for a confirmation dialog on your device.

在手机上进行如下操作:
- 在开发者工具-允许USB调试 选项关了再开
- 弹出的对话框选择允许

### 2.2 查看设备信息

```
cat /system/build.prop
```
如果出现以下结果说明需要root权限

> cat: /system/build.prop: Permission denied


