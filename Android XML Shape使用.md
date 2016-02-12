# Android XML Shape使用

shape

---
###示例一

```
<?xml version="1.0" encoding="utf-8"?>  
<shape xmlns:android="http://schemas.android.com/apk/res/android"  
android:shape="rectangle">  
<gradient android:startColor="#ffff0000"
android:endColor="#FF4488ee"  
android:type="linear" android:angle="270" />  
<padding android:left="2dp" android:top="2dp" android:right="2dp"  
android:bottom="2dp" />  
</shape>  
```
###示例二
```
[xml]  
<?xml version="1.0" encoding="utf-8"?>  
<shape xmlns:android="http://schemas.android.com/apk/res/android"  
android:shape="oval">  
<gradient android:startColor="#FFaa6622"
android:endColor="#FF2288ee"  
android:type="linear" />  
<padding android:left="2dp" android:top="2dp" android:right="2dp"  
android:bottom="2dp" />  
</shape>
```
###示例三
```
<?xml version="1.0" encoding="utf-8"?>  
<shape xmlns:android="http://schemas.android.com/apk/res/android"  
android:shape="rectangle">  
<solid android:color="#ffff0000" />  
<padding android:left="5dp" android:top="2dp" android:right="2dp"  
android:bottom="5dp" />  
<stroke android:width="2dp" android:color="#ffffffff"
android:dashGap="5dp"  
android:dashWidth="10dp" />  
</shape>
```
###示例四
```
<?xml version="1.0" encoding="utf-8"?>  
<shape xmlns:android="http://schemas.android.com/apk/res/android"  
android:shape="rectangle">  
<solid android:color="#8800ff00" />  
<padding android:left="5dp" android:top="5dp" android:right="5dp"  
android:bottom="5dp" />  
<corners android:radius="15dp" />  
</shape>
```
###示例五
```
<layer-list
xmlns:android="http://schemas.android.com/apk/res/android">
<item android:id="@android:id/background">  
<shape>  
<corners android:radius="5dip" />  
<gradient  
android:startColor="#ff9d9e9d"  
android:centerColor="#ff5a5d5a"  
android:centerY="0.75"  
android:endColor="#ff747674"  
android:angle="270"/>  
</shape>  
</item>
<item android:id="@android:id/secondaryProgress">  
<clip>  
<shape>  
<corners android:radius="5dip" />  
<gradient  
android:startColor="#80ffd300"  
android:centerColor="#80ffb600"  
android:centerY="0.75"  
android:endColor="#a0ffcb00"  
android:angle="270"/>  
</shape>  
</clip>  
</item>
<item android:id="@android:id/progress">  
<clip>  
<shape>  
<corners android:radius="5dip" />  
<gradient  
android:startColor="#ffffd300"  
android:centerColor="#ffffb600"  
android:centerY="0.75"  
android:endColor="#ffffcb00"  
android:angle="270"/>  
</shape>  
</clip>  
</item>
</layer-list>
```