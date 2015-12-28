# Android Studio菜单栏
---
# 1. File

## 1.1 new(新建)

### 1.1.1 New/Import Project 新建/导入工程 

* New Project

![](http://i.imgur.com/9gDkdGB.png)

> Tip: 这里也可以在欢迎界面选择Start a new Android studio Project来新建项目 

* 选择目标设备以及最低支持的SDK版本

![](http://i.imgur.com/wncZgeO.png)

* 选择Activity模板

如：Blank Activity；Login Activity；Settings Activity等。

* 自定义Activity,设置对应的Activity的名称、布局文件的名称。

![](http://i.imgur.com/vhcEuGu.png)

---

* Import Project

![](http://i.imgur.com/41WDT5n.png)


> Tip: 同样可以在欢迎界面选择Import Project来导入项目
* Import Eclipse项目

![](http://i.imgur.com/MqJi298.png)

选择存放目录

![](http://i.imgur.com/w4AHXeR.png)

转换配置

![](http://i.imgur.com/h0JIoDL.png)

导入成功后会生成文件import-summary.txt,在这里可以看到ES项目是如何转化为AS项目的。

![](http://i.imgur.com/qIBWANf.png)

### 1.1.2 New/Import Module 新建/导入模块

> Tip: AndroidStudio中的Project相当于Eclipse中的Workspace,Module相当于Eclipse中的Project。

* New Module 根据所需创建合适的Module

![](http://i.imgur.com/ozKAYtD.png)

* 配置模块 配置应用名以及模块名

![](http://i.imgur.com/19brAru.png)

* 选择Activity模板

![](http://i.imgur.com/SYkWZd6.png)

* 配置Activity与创建Project时同操作

* 成功创建Module

![](http://i.imgur.com/5kNgpc3.png)

新建Module后setting.gradle会自动添加到include

```  
include ':app', ':MyModule'

```

---

* Import Module

导入模块与导入项目类似，有时可能会出现模块名冲突，我们需对模块进行重命名

* 删除模块

删除模块时，我们在项目结构目录直接对某个Module进行删除时，发现找不到Delete，要想删除需如下操作：

![](http://i.imgur.com/knuwITh.png)

点击左上角"-"

![](http://i.imgur.com/wCibwoH.png)

此时再次在项目目录结构中删除模块就会有delete选项

### 1.1.3 Java Class/C++ Class 新建类文件

### 1.1.4 Package 新建包

### 1.1.5 Android resource file 新建res目录下的资源文件，如颜色选择、背景选择等

### 1.1.6 Android resource dirctory 新建res目录下的资源文件夹 如anim、drawable、layout、menu、values等。

### 1.1.7 AIDL/Activity/Fragment/Service 快速创建对应类型文件

### 1.1.8 XML 创建布局文件

* Layout xml file
* Values xml file：创建string.xml、colors.xml、dimens.xml、style.xml

## 1.2 open(打开)
1. open 选择目录打开Project
2. ReOpen Project 选择在当前窗口还是新窗口打开历史project
3. Close Project

## 1.3 Settings

常用的一些偏好设置，如字体、快捷键的设置等

### 1.3.1 Appearance&Behavior
* Appearance 设置Theme、fonts等
* System Settings
1. HTTP Proxy 设置http代理
2. Updates 检查更新Android Studio
3. Android SDK 配置SDK路径

### 1.3.2 Keymap
设置Android Studio的快捷键来适配习惯用户，用户根据自己的使用习惯可在不同系统和不同编译环境间随意切换，有Default、Eclipse、Mac os等可选项，建议使用默认的AS的快捷键(如F11 加书签)；并且自己还可以添加快捷键

### 1.3.3 Editor

* General
* Colorss&Fonts
* Code Style

### 1.3.4 Plugins

搜索安装插件

### 1.3.5 Version Control

版本控制

### 1.3.6 Build,Execution,Deployment

* Build Tools->Gradle 配置gradle home

### 1.3.7 Tools

### 1.3.8 Other Settings

对安装的插件进行配置
 
## 1.4 Project Structure

### 1.4.1 SDK Location

配置Android SDK Location、JDK Location、Android NDK Location

### 1.4.2 Project

### 1.4.3 Developer Sercices

Google提供的一些平台服务

### 1.4.4 Modules

* Properties 设置Compile Sdk Version、Build Tools Version等


* Signing 配置签名


* Flavors 配置Min Sdk Version、Target Sdk Version、Version Code、Version Name 


* Build Types


* Dependences

![](http://i.imgur.com/Zd1gGBP.png)

添加所需依赖的jar,添加成功后会在对应的Module的build.gradle中看到相应代码。

![](http://i.imgur.com/IMaqC1V.png)

## 1.5 Import/Export Setting

导入或导出设置

# 2. Edit 编辑

## 2.1 复制/黏贴

请参考快捷键

## 2.2 查找/替换

请参考快捷键

## 2.3 Column Selection Mode(列选择)

![](http://i.imgur.com/duM6oCl.png)

## 2.4 Complete Current Stateement(自动补全)

请参考快捷键

# 3. View 视图

## 3.1 Tool Windows 工具窗口

包含左边/右边工具条、下边工具条

## 3.2 Quick Definition 快速查看某个方法、变量的定义

> Tip:快捷键 Ctrl+Shift+I

![](http://i.imgur.com/AvUmvwB.png)

## 3.3 Quick Documentation 查看文档说明

> Tip:快捷键 Ctrl+Q

![](http://i.imgur.com/9VP7rwp.png)

## 3.4 Parameter Info 查看参数

> Tip:快捷键 Ctrl+P

![](http://i.imgur.com/KbV0eDA.png)

## 3.5 Compare with 对比文件

> Tip:快捷键 Ctrl+D

![](http://i.imgur.com/977SKcr.png)

## 3.6 Quick Switch Scheme 快速切换

* Color Scheme：编辑区显示
* Code Style Scheme:代码风格
* Keymap:快捷键
* View Mode:视图模式

# 4. Navigate 导航

## 4.1 Class 跳转到类文件

> Tip: 快捷键 Ctrl+N

![](http://i.imgur.com/cmICIce.png)

## 4.2 File 跳转到某个文件

> Tip: 快捷键 Ctrl+Shift+N

![](http://i.imgur.com/pgemooX.png)

## 4.3 bookMarks 书签

> Tip:添加书签 F11;助记符标识 Ctrl+F11;显示书签 Shift+F11

![](http://i.imgur.com/Z75P88O.png)

## 4.4 Declaration 跳转到声明

> Tip:快捷键 Ctrl+B

跳转到类、接口、方法、变量的声明处

## 4.5 File Structure 文件结构

> Tip:快捷键 Ctrl+F12

![](http://i.imgur.com/44LQw5J.png) 

## 4.6 Type Hierarchy 类结构层次

> Tip:快捷键 Ctrl+H

![](http://i.imgur.com/lxTsYdN.png)

# 5. Code 

## 5.1 Override Methods 重写方法

> Tip:快捷键 Ctrl+O

![](http://i.imgur.com/97MamkC.png)

## 5.2 Generate 生成构造方法等

> Tip:快捷键 Alt+Insert

![](http://i.imgur.com/FrdKyRt.png)

## 5.3 Generate->Copyright 插入版权信息

* 若没有配置，需先配置CopyRight信息

![](http://i.imgur.com/ZWdFJ9E.png)

## 5.4 Completion 代码补全

> Tip:补全 Ctrl+空格；智能补全 Ctrl+Shift+空格

![](http://i.imgur.com/TSupiu4.png)

## 5.5 Folding 展开/折叠代码

> Tip:展开 Ctrl+'+' 折叠 Ctrl+'-'

## 5.6 Insert Live Template 缩写插入

> Tip:快捷键 Ctrl+J

![](http://i.imgur.com/X4XPIHC.png)

## 5.7 Reformate Code 格式化代码

> Tip:快捷键 Ctrl+Alt+L

## 5.8 Move Statement Down/Up 上下移动代码行代码块

> Tip:上移 Ctrl+Shift+'向上' 下移 Ctrl+Shift+'向下'

# 6. Refactor 重构

## 6.1 Rename 对类重命名

> Tip:快捷键 Shift+F6

也可对变量进行重命名

## 6.2 Rename File 对文件进行重命名

![](http://i.imgur.com/iGdktvH.png)

# 7. Bulid 

## 7.1 Make Project 编译项目

make Project是对项目中新产生变化的文件进行编译，之前编译过得文件不再重新编译。

## 7.2 Rebuild Project 重新构建项目

不过之前有没有编译过，对项目的所有文件都要进行一次编译，耗时较长。

## 7.3 Clean Project 清理项目

## 7.4 Generate Signed Apk 签名Apk

![](http://i.imgur.com/crKiqH4.png)

[注]签名打包这一块会在后面详述。

# 8. Run 运行

## 8.1 Run 运行程序

## 8.2 Debug 调试程序

设置好断点，进行Debug调试  
[注]Log日志以及断点调试将在后面详述

