###### 65536
```
原因1:dex文件能够包含的最大方法数是65536
原因2:dexopt溢出,即使dex文件方法数没有超出65536
```

###### Multidex
```
引发问题:
应用启动速度会降低
Dalvik linearAlloc 的bug,可能导致使用了multidex的应用无法在Android4.0以前的手机上运行
```
