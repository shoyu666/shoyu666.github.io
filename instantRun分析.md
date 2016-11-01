
```
youdao note
```



```
sequenceDiagram
Application->>BootstrapApplicationA: attachBaseContext() start
BootstrapApplicationA-->>BootstrapApplicationB: !AppInfo.usingApkSplits start
BootstrapApplicationB->>BootstrapApplicationB:createResources
BootstrapApplicationB->>BootstrapApplicationB:setupClassLoaders
BootstrapApplicationA-->>BootstrapApplicationB: !AppInfo.usingApkSplits end
BootstrapApplicationA-->BootstrapApplicationA:super.attachBaseContext(context);
BootstrapApplicationA-->>BootstrapApplicationB:realApplication!=null
BootstrapApplicationB->>BootstrapApplicationB:invoke realApplication attachBaseContext()
Application->>BootstrapApplicationA: attachBaseContext() end
Application->>BootstrapApplicationA: onCreate() start
BootstrapApplicationA-->>BootstrapApplicationB:MonkeyPatcher.monkeyPatchApplication
BootstrapApplicationA-->>BootstrapApplicationB:MonkeyPatcher.monkeyPatchExistingResources
Application->>BootstrapApplicationA: onCreate() end
```

##### createResources

```
sequenceDiagram
createResources-->>FileManager:checkInbox start
FileManager-->>FileManagerB:startUpdate start
FileManagerB-->>FileManagerB:getWriteFolder(wipe=true) start
FileManagerB-->>FileManagerB:a=leftIsActive?left:right/wipe a and return a 
FileManager-->>FileManagerB:startUpdate end
FileManager-->>FileManagerB:writeAaptResources start
FileManager-->>FileManagerB:writeAaptResources end
FileManager->>FileManager:finishUpdate 写入'left' 到active ?
FileManager->>FileManager:resources.delete()
createResources-->>FileManager:checkInbox end
```
leftIsActive:如果active文件不存在返回true,如果active存在，判断文件里面是否存储的是‘left’


##### setupClassLoaders

```
sequenceDiagram
IncrementalClassLoader-->>IncrementalClassLoaderB: inject()
IncrementalClassLoaderB->>IncrementalClassLoaderB:new IncrementalClassLoader
IncrementalClassLoaderB->>IncrementalClassLoaderB:setParent(classLoader, incrementalClassLoader=newParent)
IncrementalClassLoaderB->>DelegateClassLoader:createDelegateClassLoader
DelegateClassLoader->>IncrementalClassLoaderB:new DelegateClassLoader
IncrementalClassLoader-->>IncrementalClassLoaderB: inject() end
```
findClass
######  IncrementalClassLoader extends ClassLoader
######  DelegateClassLoader extends BaseDexClassLoader
######  IncrementalClassLoader 包含 DelegateClassLoader
```
sequenceDiagram
IncrementalClassLoader-->>DelegateClassLoader: findclass
```

