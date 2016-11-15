

```
目的
1android中使用矢量图片
2png icon转矢量图片
```


```
1:android中使用矢量图片
    1 build.gralde
     defaultConfig {
        ...
        vectorDrawables.useSupportLibrary = true
        ...
     }

   2 布局
    
   xmlns:app="http://schemas.android.com/apk/res-auto"
   
   <android.support.v7.widget.AppCompatImageView
   ...
    app:srcCompat="@drawable/ic5_home_lawserver" //替代 android:src
    ...
    />
```

```
2:png icon转矢量图片
 
 软件:vector magic(可以搜破解版)
 步骤:
      1:导入png
      2:保存为svg
      3:android studio:new vector asset (选择导出的svg即可)
 
 

```
