
###### 前提： android 高版本插件开放了dexOptions，方便开发者可以制定dex参数
###### 目的： 指定第一个dex中类
###### 步骤
```
tool版本2.2+
例：com.android.tools.build:gradle:2.2.0-rc1
```


```
build.gralde配置
android{
.......
  multiDexEnabled true
.......
dexOptions 
    {
     additionalParameters = ['--minimal-main-dex','--set-max-idx-number=20000','--main-dex-list=...maindexlist.txt']
    }
}
--minimal-main-dex  第一个dex最小，配合--main-dex-list 表示只有指定的类在第一个dex中
--set-max-idx-number=20000 dex方法数  超过这个dex会分包
--main-dex-list  指定第一个dex中类
.....
```

```
maindexlist.txt内容[可以参考build\intermediates\multi-dex\debug\maindexlist.txt]
com/shoyu666/a/a.class
.....
```

```
结果：a.class会在第一个class.dex中
```
