

> 假设如下dex

class.dex |class1.dex |
---|---
a.class |d.class |
b.class[有bug] |e.class|
c.class |f.class|
.... |...|
 
1：下面分析android是如何加载class的，比如b.class
> 核心代码
> https://android.googlesource.com/platform/libcore-snapshot/+/ics-mr1/dalvik/src/main/java/dalvik/system/DexPathList.java

```
  public Class findClass(String name) {
        //dexElements [class.dex,class1.dex.....]
        for (Element element : dexElements) {
            DexFile dex = element.dexFile;
            if (dex != null) {
                Class clazz = dex.loadClassBinaryName(name, definingContext);
                if (clazz != null) {
                    return clazz;
                }
            }
        }
        return null;
    }
    //可以看到是按照dexElements数组里面的dex文件顺序循环，第一个成功找到并返回
```
那么我们hack上面这个查找过程，新增补丁dex

补丁.dex |class.dex |class1.dex |
---|---|---
无 |a.class |d.class |
b.class[补丁]  |b.class[有bug] |e.class|
无  |c.class |f.class|
无 |...|...|

```
        ....
  //dexElements [补丁.dex,class.dex,class1.dex.....]
  //加入补丁.dex，并放到数组最前面，那么就会首先查找补丁.dex的类
        for (Element element : dexElements) {
        ....
```
2：成功插入了补丁的class,但是下面的情况会出问题

```
报错：java.lang.IllegalAccessError: Class ref in pre-verified class
场景：c.class调用b.class
现在是 c.class[class.dex]---->b.class[补丁.dex]  不同dex中，报错
原先是c.class[class.dex]---->b.class[class.dex]  同一个dex中,不报错
```

```
错误原因：被打上CLASS_ISPREVERIFIED标签的类会进行校验。
报错源代码：
https://android.googlesource.com/platform/dalvik.git/+/51801371a9b0f829303d326a2300518177dde3e8/vm/oo/Resolve.cpp
if (!fromUnverifiedConstant &&
            IS_CLASS_FLAG_SET(referrer, CLASS_ISPREVERIFIED))
        {
            ClassObject* resClassCheck = resClass;
            if (dvmIsArrayClass(resClassCheck))
                resClassCheck = resClassCheck->elementClass;
            if (referrer->pDvmDex != resClassCheck->pDvmDex &&
                resClassCheck->classLoader != NULL)
            {
                ALOGW("Class resolved by unexpected DEX:"
                     " %s(%p):%p ref [%s] %s(%p):%p",
                    referrer->descriptor, referrer->classLoader,
                    referrer->pDvmDex,
                    resClass->descriptor, resClassCheck->descriptor,
                    resClassCheck->classLoader, resClassCheck->pDvmDex);
                ALOGW("(%s had used a different %s during pre-verification)",
                    referrer->descriptor, resClass->descriptor);
                dvmThrowIllegalAccessError(
                    "Class ref in pre-verified class resolved to unexpected "
                    "implementation");
                return NULL;
            }
        }

解决办法：防止类被打上CLASS_ISPREVERIFIED标签

1：那么类是什么时候被打上CLASS_ISPREVERIFIED标签的
https://android.googlesource.com/platform/dalvik/+/froyo-release/vm/analysis/DexVerify.c
if (dvmVerifyClass(clazz, VERIFY_DEFAULT)) {
    //校验通过则写入CLASS_ISPREVERIFIED标签
                    assert((clazz->accessFlags & JAVA_FLAGS_MASK) ==
                        pClassDef->accessFlags);
                    ((DexClassDef*)pClassDef)->accessFlags |=
                        CLASS_ISPREVERIFIED;//加入标签
                }
                
2：如何防止被打上标签
原理：让校验不通过
如果a.class 引用了不在同一个dex文件中的class,那么a.class就不会有CLASS_ISPREVERIFIED
上面c.class开始和b.class在同一个dex,所以c.class被打上标签
 后面补丁的b.class和c.class不在同一个dex,所以报错，为了解决这个问题
 我们要事先就要防止a.class被打上标签

3 方法：
使用gralde插件，编译阶段在a.class的构造函数中插入(通过字节码修改库asm)
Hack.class的引用，hack.class在独立的dex文件中，那么a.class的标签就不会打上
```


```
https://github.com/dodola/RocooFix 实现了上面方案的开源库
```
```
rocoo 解决类加载问题核心代码
   Field pathListField = RocooUtils.findField(loader, "pathList");
            Object dexPathList = pathListField.get(loader);
            Field dexElement = RocooUtils.findField(dexPathList, "dexElements");
            Class<?> elementType = dexElement.getType().getComponentType();
            Method loadDex = RocooUtils.findMethod(dexPathList, "loadDexFile", File.class, File.class, ClassLoader.class, dexElement.getType());
            loadDex.setAccessible(true);

            Object dex = loadDex.invoke(null, additionalClassPathEntries.get(0), optimizedDirectory, loader, dexElement.get(dexPathList));
            Constructor<?> constructor = elementType.getConstructor(File.class, boolean.class, File.class, DexFile.class);
            constructor.setAccessible(true);
            Object element = constructor.newInstance(new File(""), false, additionalClassPathEntries.get(0), dex);

            Object[] newEles = new Object[1];
            newEles[0] = element;
            RocooUtils.expandFieldArray(dexPathList, "dexElements", newEles);
```


```
rocoofix 插件，解决CLASS_ISPREVERIFIED 问题 核心代码
v.visitFieldInsn(Opcodes.GETSTATIC, "java/lang/Boolean", "FALSE", "Ljava/lang/Boolean;");
        v.visitMethodInsn(Opcodes.INVOKEVIRTUAL, "java/lang/Boolean", "booleanValue", "()Z", false);
        v.visitJumpInsn(Opcodes.IFEQ, l1);
        v.visitFieldInsn(Opcodes.GETSTATIC, "java/lang/System", "out", "Ljava/io/PrintStream;");
        v.visitLdcInsn(Type.getType("Lcom/dodola/rocoo/Hack;"));
        v.visitMethodInsn(Opcodes.INVOKEVIRTUAL, "java/io/PrintStream", "println", "(Ljava/lang/Object;)V", false);
        v.visitLabel(l1);
```



> 参考
<br>  <a href="https://mp.weixin.qq.com/s?__biz=MzI1MTA1MzM2Nw==&mid=400118620&idx=1&sn=b4fdd5055731290eef12ad0d17f39d4a&scene=1&srcid=1106Imu9ZgwybID13e7y2nEi#wechat_redirect">空间的文章<a/>  感谢qqzone的无私奉献
<br>  <a href="https://github.com/dodola/RocooFix">RocooFix</a> 感谢dodola的无私奉献
