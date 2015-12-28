# Google Android 文档翻译 -- 谈项目构建
**Develop > Build System / Configuring Gradle Builds / Android Plugin for Gradle（缺） / Manifest Merging / Apps Over 65K Methods（缺）** 
## Build System Overview -- 构建系统概览
android 构建系统是一个可以使用来构建,测试,运行和打包app的工具包.可以从as的菜单栏也可以单独从命令行运行,可以使用构建系统的特性来:      
* 自定义,设置,扩展构建流程     
* 使用相同的文件和模块,为app创建带有不同特性的多apk文件          
* 通过代码集(source sets)复用代码和资源          
android灵活的构建系统可以使你在不改变app的核心代码的情况下实现以上目标    
### 构建过程详解   
构建过程需要很多工具和流程参与,在生成一个.apk文件时会生成一些中间文件.如果你使用的是android studio,一旦你为你的项目或模块运行Gradle 构建任务,完整的构建流程就被执行了.
以下是典型的构建过程:
![android构建过程及参与的工具](http://i.imgur.com/q3W3Zux.png)   
构建系统从配置的产品版本product flavors,构建类型build types以及依赖dependencies合并所有的资源.如果不同的文件包含同名的资源或者设置,遵守的覆盖优先级是:依赖覆盖构建类型,构建类型覆盖产品版本,产品版本覆盖主源码目录main source directory.
* aapt(Android Asset Packing Tool)编译你的应用资源文件,如AndroidManifest.xml文件和对应activity的xml文件.生成的R.java文件可以方便你从你的java代码里引用资源
* aidl工具将所有的.aidl接口转化成java接口  
* 所有的java 代码,包括R.java和.aidl文件被java编译器编译输出.class文件
* dex工具将.class文件转化成Dalvik字节码.任何你在你的构建模块里导入的第三方库和.class文件同样被转化成.dex文件,因此它们也会被打包进最终的.apk文件里.
* 所有未编译的资源,如图片images,编译后的资源,如.dex文件,都被apkbuilder工具拿去打包成.apk文件
* 一旦.apk文件被构建,在被安装到设备时都要用debug或者release的key签名
* 如果应用是用release模式签名,必须使用zipalign工具来对齐.apk,使应用运行在设备时能减少内存的使用
> 需要注意的是app的方法引用限制最大为64k,如果超过了,构建过程会输出如下错误提示:
`Unable to execute dex: method ID not in [0, 0xffff]: 65536.`
### 构建的输出   
构建对每次构建变种都会在`app/build`生成apk,如`app/build/outputs/apk`目录包含命名为`app-<flavor>-<buildtype>.apk`,例如`app-full-releas.apk`和`app-demo-debug.apk`   
## Configuring Gradle Builds -- gradle构建配置
使用product flavors和build types配置不同的构建   
### 构建配置基础  
android studio包含一个顶级的build文件和对每个模块的build文件,这些build文件被称作build.gradle,是使用Groovy 语法来配置,使用gradle的android插件提供的元素来构建的纯文本文件.在大部分情况下,你只需要编辑模块级别的build文件.以下是一个叫做        BuildSystemExample项目的app模块的build.gradle:   
```
apply plugin: 'com.android.application'
android {
    compileSdkVersion 19
    buildToolsVersion "19.0.0"
    defaultConfig {
        minSdkVersion 8
        targetSdkVersion 19
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
dependencies {
    compile project(":lib")
    compile 'com.android.support:appcompat-v7:19.0.1'
    compile fileTree(dir: 'libs', include: ['*.jar'])
}
```
    
 `apply plugin: 'com.android.application'` gradle构建里增加android插件,该插件添加了android特定的构建任务到顶级的构建任务里,并且可使用`android{}`元素来指定android独有的构建选项
  `android{}`配置所有的android独有的构建选项:
* `compileSdkVersion`属性指定编译目标   
* `buildToolsVersion`属性指定build tools使用的版本.可以使用SDK Manager来安装一些build tools      
> 保证build tools 的主版本号总要大于等于编译目标和目标sdk    
* `defaultConfig`元素动态地从构建系统配置核心设置以及在`AndroidManifest.xml`里的实体.`defaultConfig`里的值会覆盖`AndroidManifest.xml`里的值.`defaultConfig`里配置的值会被应用到所有的构建变种版本build variants里,除非某个构建变种版本覆盖了它的值
* `buildTypes`控制如何构建和打包app,默认的,构建系统定义两种类别,debug和release.debug构建类型包括调试标识并且使用debug key来签名;releas构建类型并不是按照默认方式来签名.在上面的例子里这个构建文件使用ProGuard来配置release 版本   
`dependencies`是`android`之外以及之后的元素,该元素声明了这个模块的依赖. 
  
> **在项目里一旦改变gradle.build里的配置,as会提示"Sync now",点击,导入构建配置的改变来同步项目**  
### Declare dependencies -- 依赖声明      
例子中`app`模块里的依赖声明:     
```
...
dependencies {
    // Module dependency
    compile project(":lib")
    // Remote binary dependency
    compile 'com.android.support:appcompat-v7:19.0.1'
    // Local binary dependency
    compile fileTree(dir: 'libs', include: ['*.jar'])
}
```   
构建系统添加所有的`compile`依赖到编译的路径,并且添加到最终的包里即XX
.apk   
#### 模块依赖:  
`app`模块依赖于`lib`模块,因为MainActivity从库模块加载LibActivity1  
`compile project(":lib")`声明`BuildSystemExample`项目的一个对`lib`模块的依赖,当你构建`app`模块,构建系统编译并包含这个`lib`模块    
#### 远程二进制依赖    
`app`和`lib`模块都使用Android  Support Library的类`ActionBarActivity`,所以这两个模块都依赖它   
`compile 'com.android.support:appcompat-v7:19.0.1'`通过指定Maven的坐标coordinates声明了对版本为19.0.1的Android Support Library的依赖.Android Support Library在Android SDK的Android Repository包里,如果没有下载该包,请用SDK Manager下载.     
Android Studio配置项目使用Maven中央仓库Maven Central Repository作为默认仓库(在顶级build即项目build里配置)
#### 本地二进制依赖:    
一些模块并没有使用任何本地文件系统的二进制依赖,如果你的模块里需要本地二进制依赖,复制JAR 文件到`<moduleName>/libs`作为依赖使用   
`compile fileTree(dir: 'libs', include: ['*.jar'])`告诉构建系统,任何app/lib路径下的JAR文件都是一个依赖,都应该被包含在编译路径里和最终的包里    
### Run ProGuard -- 执行混淆   
构建系统可以使用ProGuard来混淆构建过程中的类,在`BuildSystemExample`,修改app模块的build文件来为release版本执行混淆:       
   
```
	...
android {
    ...
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
...
```     
 `getDefaultProguardFile('proguard-android.txt')`包含来自Android SDK 安装的默认的混淆配置,Android Studio在根模块添加模块特定的规则文件`proguard-rules.pro`,在该文件里你可以添加自定义的混淆规则.                           
###  Application ID for package identification --包鉴别之应用ID    
 在android构建系统里,applicationId属性被用来特殊标识应用发布时的包.应用ID在`build.gradle`的android这一节里被设置:   
```
 apply plugin: 'com.android.application'
    android {
        compileSdkVersion 19
        buildToolsVersion "19.1"
    defaultConfig {
        applicationId "com.example.my.app"
        minSdkVersion 15
        targetSdkVersion 19
        versionCode 1
        versionName "1.0"
    }
    ...
```    
> **applicationId只是在build.gradle里被指定,在AndroidManifest.xml并没有被指定,所以还是要说明manifest节点下的package属性的值**    
构建不同的版本变种时,构建系统可以特别标识不同产品版本produce flavors和构建版本build type的包.application ID 在build type添加仅仅是作为一个后缀来指明product flavors     
```
 productFlavors {
        pro {
            applicationId = "com.example.my.pkg.pro"
        }
        free {
            applicationId = "com.example.my.pkg.free"
        }
    }
    buildTypes {
        debug {
            applicationIdSuffix ".debug"
        }
    }
    ....
```
包名依然需要在manifest文件里指明,它被用来在源码里引用R类以及一些相关的activity/service组件的注册    
` package="com.example.app"`   
  
> **如果你有多个manifests(例如,一个product flavor指明一个manifest和一个build tpye 的manifest),这个包名在manifests里就是可选的.如果在这些manifests里已经定义了包名,那在src/main下的manifest里的包名就应该是一致的**        
     
### Configure signing settings -- 签名的配置  
app的debug和release版本不同于是否应用可以在安全的设备上调试以及APK如何签名.构建系统对debug版本使用一个默认的key签名,认证使用已知的证书来避免在构建的时候提示密码.构建系统不对release签名,除非你精确地为这次构建定义一个签名配置,如果你没有一个release key,你可以生成一个"链接"  
### Work with build variants -- 构建变体   
从一个单一的项目里产生同一个应用的不同版本.当在你的app里有一个demo版本和一个付费版本时,或者当你想要在Google Play对不同的设备配置发布不同的apk时.  
这个构建系统使用`product flavors`来对app产生不同的产品版本,每个产品版本可以有不同的特性或者设备需求.这个构建系统同样使用build type来适应每个产品版本的不同的构建和打包配置.每个product flavor和build type结合形成一个构建变体build variant.构建系统对app的每一个build variant生成一个不同的apk     
#### build variants构建变体:    
示例里的项目的app类型(demo和full)包括两个默认的构建类型(debug和release)以及两个product flavors   
##### product flavors           
为了在你的app里产生不同的产品版本,你需要       
1. 在build文件里定义product flavors  
2. 对每个flavor产生额外的源码目录
3. 添加指定的flavor源码到你的项目里    
在以下的例子项目`BuildSystemExample`里将创建两个flavor,demo和full flavor,每个flavor都共享MainActivity,添加一个新的按钮来加载一个新的叫做SecondActivity.这个新的activity不同于每个flavor,所以你讲模拟一个情景,即SecondActivity在full flavor比在demo flavor有更多的特性.   
#### 在build文件里定义product flavor   
在app模块的build.gradle里添加:   
```
...
android {
    ...
    defaultConfig { ... }
    signingConfigs { ... }
    buildTypes { ... }
    productFlavors {
        demo {
            applicationId "com.buildsystemexample.app.demo"
            versionName "1.0-demo"
        }
        full {
            applicationId "com.buildsystemexample.app.full"
            versionName "1.0-full"
        }
    }
}
...
```  
 product flavor定义和`defaultConfig`元素都支持相同的属性,对所有的flavor都适用的基本配置在defaultConfig里指明,每一个flavor都将覆盖任何默认值.以上的build文件使用`applicationId`属性来为每一个flavor赋值不同的包名:因为每个flavor定义创建一个不同的app,因此需要一个独有的包名   
> **为了使用多apk支持Multiple APK Support在Google Play发布你的app,对所有的变种赋值相同的包名但不同的`versionCode`;在Google Play对不同的变种发布分离的app,对没一个变种赋予不同的包名**  
##### 为每个flavor添加额外的源码目录   
创建源码文件夹并添加`SecondActivity`到每个flavor.对demo flavor,创建一个源码目录结构:
1. 在Project面板,扩展BuildSystemExample,然后扩展app目录   
2. 选中app,右键src目录,选择new->Directory   
3. 输入"demo"作为新目录的名字并点击ok   
4. 相同的,创建以下目录:   
``` app/src/demo/java
    app/src/demo/res
    app/src/demo/res/layout
    app/src/demo/res/values
```
##### 每个flavor添加一个新的activity   
添加`SecondActivity`到`demo`flavor   
1. 在Project面板,选中app模块,右键选择New->Activity
2. 选择Blank Activity并点击Next
3. 输入"SecondActivity"作为activity的名字  
4. 输入"com.buildsystemexample.app"作为包名并点击finish  
5. 右键点击目录 app/src/demo选择New->Package   
6. 输入"com.buildsystemexample.app"作为包名并点击OK  
7. 拖动`SecondActivity`到目录app/src/demo/java下   
8. 接受默认值并点击Refactor  
添加`SecondActivity`的布局文件和一个字符串资源到demo flavor:  
1. 从app/src/main/res/layout拖动 activity_second.xml到app/src/demo/res/layout   
2. 默认点击ok
3. 从app/src/main/res into app/src/demo/res复制strings.xml  
4. 替换copy来的`strings.xml`为:  
```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="hello_world">Demo version only.</string>
</resources>
```  
现在你添加源码目录并从demo flavor复制了`SecondActivity`到full flavor:  
1. 在Project面板,右键点击 app/src下demo目录,选择copy  
2. 在app/下右键点击src/,选择粘贴  
3. 将"full"作为新的名称并点击ok
4. 替换src/full/res/values内容为:  
```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="hello_world">This is the full version!</string>
</resources>
```
> **到现在,可以在每个flavor里独立开发SecondActivity,例如,你可以在full flavor里添加更多的特性**  
为了在一个特定的flavor里工作,点击IDE窗口的左侧,选择
Build Variants,在Build Variant里选择你想要修改的variant,as可能会在flavor的源码里显示错误,而不是Build Variant面板,但这不影响build的输出   
#### 从main activity加载特定flavor里的activity  
因为`SecondActivity`在每个flavors都有相同的包名和activity名,你可以从main activity加载它,对每个flavor来说都一样,可以这样修改main activity: 
1. 编辑`activity_main.xml`并且添加一个按钮到` MainActivity`:
```
<LinearLayout ...>
    ...
    <Button
        android:id="@+id/button2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/button2"
        android:onClick="onButton2Clicked"/>
</LinearLayout>
```
2. 在布局文件里标有红色波浪线的位置按Alt+Enter,按照建议添加一个新的String资源,值为"Open Second Activity",并且在`MainActivity`添加一个方法`onButton2Clicked`  
3. 在`MainActivity`补足方法:`onButton2Clicked`  
```
public void onButton2Clicked(View view) {
    Intent intent = new Intent(this, SecondActivity.class);
    startActivity(intent);
}
```  
4. 编辑app的manifest并包含`SecondActivity`的引用   
```
<manifest ...>
    <application ...>
        ...
        <activity
            android:name="com.buildsystemexample.app.SecondActivity"
            android:label="@string/title_activity_second" >
        </activity>
    </application>
</manifest>
```
### Build Types -- 构建类型   
构建类型代表为每个app包生成的构建包版本( the build packaging versions generated for each app package).默认的,提供了debug和release构建类型   
```
...
android {
    ...
    defaultConfig { ... }
    signingConfigs { ... }
    buildTypes { ... }
    productFlavors {...}
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
         debug {
            debuggable true
        }
    }
}
...
```
> **虽然只有release 构建类型出现在默认的build.gradle,对每次构建,release和debug构建类型都适用**
在这个例子里,product flavors和build type创建了以下的build variants:   
*demoDebug
*demoRelease
*fullDebug
*fullRelease   
为了构建这个例子,点击as菜单里的Build选项或者在命令行里调用assemble任务(gradle assemble)  
> **Build->Make Project编译整个项目的自从上次编译后修改的所有源文件,Build->Rebuild Project选项重新编译项目里的所有源文件**
对每个build variant都产生了单独的输出文件  
## Android Plug-in for Gradle -- gradle的android插件   
* Build configuration
* Build by convention
* Projects and modules build settings
* Project Build File
* Module Build File
* Dependencies
* Build tasks
* The Gradle wrapper
*  Build variants
* Source directories
## Manifest Merging -- 合并Manifest
android studio和基于gradle的构建系统,使每个app在多个位置包含manifest文件,例如对于`productFlavor`，库，android 存档Android ARchive（AAR)Android 库项目的集合，以及依赖的`src/main/`文件夹.在构建过程中,manifest合并结合了包括在app中的不同`AndroidManifest.xml`文件的设置,为app打包和发布生成成一个单独的APK manifest文件.Manifest设置的合并基于由manifest文件所在位置决定的manifest优先级.构建你的app,从这些manifests文件里合并这些manifest元素,属性,以及子元素来生成特定的build variant.
### Merge Conflict Rules -- 合并冲突规则  
合并冲突发生在当合并的manifests包含相同的manifest元素却又不同的值而又无法用默认的合并冲突规则解决时.冲突标记和选择器可以定义自定义的合并规则,例如允许一个导入的库的`minSdkVersion`高于定义在其他优先级较高的manifests里的版本.
manifest 合并优先级决定了哪个manifest设置在合并冲突里被保留,较高优先级manifest里的设置会覆盖较低优先级里的manifest设置.以下清单详情列举了在合并过程中最高优先级的manifest设置:      
* 最高优先级:`buildType`manifest设置    
* 较高优先级:`productFlavor`manifest设置   
* 中等优先级:在ap
----------
p项目目录`src/main`中的Manifests文件   
* 低优先级:依赖和库manifest 设置   
Manifest的xml节点和属性级别的合并冲突解决方案基于以下规则:       
![Manifest的xml文件节点和属性冲突合并解决规则](http://i.imgur.com/dZe7G4v.png)     
manifest合并规则的例外:     
* `uses-feature android:required;`和`uses-library android:required`元素默认为`true`并且使用或OR合并,因此所有需要的特性或者库都会包含在生成的APK中   
* 如果没有声明,`uses-sdk`元素,`minSdkVersion`和`targetSdkVersion`.默认值为1,当合并冲突发生,较高优先级manifest文件中的版本会被使用   
* 导入一个`minSdkVersion`值高于app的`src/main`manifest的值会产生一个错误,除非使用`overrideLibrary`冲突标记      
> 如果没有明确声明,`targetSdkVersion`默认等于
`minSdkVersion`的值.当在manifest或者`build.gradle`没有指明值时,`minSdkVersion`默认值为1.      
* 当导入一个`targetSdkVersion`值小于app的`src/main`的manifest,manifest合并进程会明确授予权限并确保导入的库功能正常   
* `manifest`元素仅仅合并子manifest元素   
* 合并manifest时,`intent-filter`元素从不改变并且总是添加到普通的父节点         
> manifest合并后,构建进程重写同时也在build.gradle里的manifest设置
### Merge Conflict Markers and Selectors -- 合并冲突的标识器和选择器    
manifest标识器和选择器通过特定的冲突解决方案重写默认的合并规则.例如,使用一个冲突标识器来合并一个`minSdkVersion`比最高优先级manifest还高的库manifest,或者合并带有相同的activity但是不同`android:theme`值的manifest    
#### Merge Conflict Markers -- 冲突合并标识器  
一个冲突合并标志器是一个在定义了一些特别的冲突合并方案的Android tools 名称空间里的特别的属性.产生一个冲突标识器来避免合并冲突错误而不是利用默认的合并规则来解决冲突.支持的合并冲突标识器包括:     
`merge` : 冲突发生时没有合并规则的合并属性.默认的合并动作.    
`replace` : 将较低优先级里manifest里的属性替换为较高优先级manifest里的    
`strict` : 设置合并策略等级,这样带有相同属性但不同值的合并元素合并失败,除非通过冲突规则解决了冲突.    
`merge-only` : 仅对较低优先级的属性的合并措施   
`remove` : 从合并的manifest里一处特定的较低优先级的元素  
`remove-All` : 移除合并manifest里的相同节点类型所有较低优先级元素     
默认的,manifest合并进程对节点等级应用`merge`冲突标识器.所有声明的manifest属性默认为`strict`合并策略.    
为了设置一个合并冲突标识器,首先在`AndroidManifest.xml`里声明名称空间(xmlns:tools="xxx"),接着在manifest理输入合并冲突标识器来指明一个自定义的合并冲突动作.以下例子插入了`replace`标识器来设置一个替换动作来解决在`android:incon`和`android:label`的冲突  
```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
   package="com.android.tests.flavorlib.app"
   xmlns:tools="http://schemas.android.com/tools">
   <application
       android:icon="@drawable/icon"
       android:label="@string/app_name"
       tools:replace="icon, label">
       ...
```      
##### Marker attributes -- 标识器属性    
冲突标识器使用`tools:node`和`tools:attr`属性来在xml节点或者属性等级的限制合并动作  
`tools:attr`标识器仅仅使用`restrict`,`remove`,`replace`合并动作.多个`tools:attr`标识器值可以被应用到特定的元素.例如,使用`tools:replace="icon,label,theme"`来提婚较低优先级的`icon`,`label`,`theme`属性   
##### Merge conflict marker for imported libraries --  导入库时的冲突合并标识器    
`overrideLibrary`冲突标识器被应用在manifest声明里,并且通过库的`<uses-sdk>`值被用来导入一个库,例如`minSdkVersion`比在更高优先级的manifest里被设置成不同的值    
没有标识器,由`<uses-sdk>`值导致的库manifest合并冲突将导致失败.   
以下例子应用`overrideLibrary`冲突标志器来解决在`src/main`下manifest和导入的库的manifest的`minSdkVersion`合并冲突:     
```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
   package="com.android.example.app"
   xmlns:tools="http://schemas.android.com/tools">
   ...
   <uses-sdk android:targetSdkVersion="22" android:minSdkVersion="2"
             tools:overrideLibrary="com.example.lib1, com.example.lib2"/>
   ...
```      
> 默认的合并过程不允许导入一个比`src/main`里的`minSdkVersion`更高的库,除非使用了`overrideLibrary`冲突标识器      
#### Marker Selectors -- 
标识选择器对特定的低优先级manifest限制合并动作.例如,一个标识选择器可以被用来从一个库里移除一个权限,而在其他库了却允许该权限.    
以下例子使用`tools:node`标识器来移除`permisionOne`属性,`tools:selector`选择器指明特定的库为com.example/lib1.`permisionOne`权限制备`lib1`库过滤    
```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
   package="com.android.example.app"
   xmlns:tools="http://schemas.android.com/tools">
   ...
   <permission
         android:name="permissionOne"
         tools:node="remove"
         tools:selector="com.example.lib1">
   ...
```
### Injecting Build Values into a Manifest -- 
manifest合并也可以被配置来使用manifest占位符来注入来自`build.gradle`的属性值到manifest属性     
manifest占位符使用语法`${name}`来表示属性值,`name`是从`build.gradle`注入的.`build.gradle`文件使用`manifestPlaceholders`属性来声明占位符的值      
> 在app里未分辨的占位符名称会导致构建失败.库里的未分辨的占位符生成警告并且在导入该库到app时需要决定占位符表示的值.    
以下例子展示了manifest占位符`${applicationId}`被使用来注入`build.gradle``applicationId`属性值到`android:name`属性值        
> Android Studio提供一个默认的`${applicationId}`占位符来引用`build.gradle``applicationId`的值,这在build文件中并没有显示.    
Manifest 入口:         
```
<activity
android:name=".Main">
     <intent-filter>
     <action android:name="${applicationId}.foo">
         </action>
</intent-filter>
</activity>
```     
gradle构建文件:     
```
android {
   compileSdkVersion 22
   buildToolsVersion "22.0.1"
   productFlavors {
       flavor1 {
           applicationId = "com.mycompany.myapplication.productFlavor1"
       }
}
```
    
合并的manifest值:        
 
`<action android:name="com.mycompany.myapplication.productFlavor1.foo">`       
manifest占位符语法和构建文件`manifestPlaceholders`属性可以用来注入其他manifest值.对于其他不是`applicationId`的属性,`manifestPlaceholders`属性被明确地在`build.gradle`里被声明.以下例子展示了在manifest占位符里注入`activityLabel`值.       
Gradle 构建文件:       
```
android {
    defaultConfig {
        manifestPlaceholders = [ activityLabel:"defaultName"]
    }
    productFlavors {
        free {
        }
        pro {
            manifestPlaceholders = [ activityLabel:"proName" ]
        }
    }
```
manifest文件里的占位符:     
`<activity android:name=".MainActivity" android:label="${activityLabel}" >`       
> 占位符的值支持局部值的注入,如`android:authority="com.acme.${localApplicationId}.foo"`     
### Manifest Merging Across Product Flavor Groups -- 通过产品flavor组合并manifest      
当使用`GroupableProductFlavor`属性,在product flavor group里的所有manifest合并都遵循在build 文件列举的product flavor的规则.manifest合并进程基于配置build variant为product flavor产生一个单一的合并的manifest      
例如,如果一个build variant从各自的列举在`build.gradle`的product flavor group`ABI`,`Density`,以及`Prod`引用product flavors`x86`,`mdpi``21`以及`paid`,那么manifest合并进程按照build文件如何列举的product flavors的优先级顺序合并这些manifest        
为了阐述这个例子,一下图表显示每个product flavor group如何列举这些prduct flavor的.product flavor和groups的结合定义了build variant.      
        
![图解通过product flavor group合并manifest](http://i.imgur.com/pq3zXrr.png)         
  
manifest合并规则:      
* prod-paid的AndroidManifest.xml(最低优先级)合并到API-22AndroidManifest.xml        
* API-22AndroidManifest.xml合并到density-mpi AndroidManifest.xml.         
* density-mpi AndroidManifest.xml合并到ABI-x86 AndroidManifest.xml(最高优先级)       
### Implicit Permissions -- 隐式权限      
导入一个目标是Android 运行时的隐式授权权限库可能会自动给添加权限到最终合并后的manifest.例如,如果一个一个`targetSdkVersion`为16的应用导入一个`targetVersion`为2的库,Android Studio添加`WRITE_EXTERNAL_STORAGE`权限来确保SDK版本的兼容性.         
> 现在有越来越多的Android release 用权限声明来取代隐士权限        
以下表格列举了导入的库版本和被声明的权限      
![导入的库和被声明的权限](http://i.imgur.com/qUoBZjW.png)
### Handling Manifest Merge Build Errors -- 处理manifest合并构建错误      
在构建过程中,manifest合并进程存储一个每个合并事务记录到模块`build/outputs/logs`文件夹下的`manifest-merger-<productFlavor>-report.txt`文件.每个模块的build variant生成一个不同的日志文件.     
当一个manifest合并构建错误发生,合并进程在日记文件里记录错误信息,描述合并冲突.例如,`android:screenOrientation`合并冲突在下面的manifest导致了一个build错误.    
较高优先级的manifest声明:    
```
<activity
   android:name="com.foo.bar.ActivityOne"
   android:screenOrientation="portrait"
   android:theme="@theme1"/>
```
较低优先级manifest声明:     
```
<activity
   android:name="com.foo.bar.ActivityOne"
   android:screenOrientation="landscape"/>
```
错误日志:      
```
/project/app/src/main/AndroidManifest.xml:3:9 Error:
 Attribute activity@screenOrientation value=(portrait) from AndroidManifest.xml:3:9
 is also present at flavorlib:lib1:unspecified:3:18 value=(landscape)
 Suggestion: add 'tools:replace="icon"' to  element at AndroidManifest.xml:1:5 to override
```
## Building Apps with Over 65K Methods -- 在方法超过65k时构建app




