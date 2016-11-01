
```
youdao note
```
#### preview

```
sequenceDiagram
AppShell-->Applition:ignore
Exopackage-->Applition:AppShell() > Exopackage()
Applition-->>Exopackage: onCreate() start
Exopackage->>Exopackage: ensureDelegate()
Exopackage-->>Exopackage_createDelegate: createDelegate()
Exopackage_createDelegate->>Exopackage_createDelegate:xx
Exopackage_createDelegate-->>ExopackageDexLoader:ExopackageDexLoader.loadExopackageJars
ExopackageDexLoader->>ExopackageDexLoader:加载jar
Exopackage_createDelegate-->>ExopackageSoLoader:ExopackageSoLoader.init(this)
ExopackageSoLoader->>ExopackageSoLoader:加载so
Exopackage-->>ApplicationLike: delegate.onCreate()
ApplicationLike->>ApplicationLike:onCreate()
Exopackage-->Applition:onCreate() end
```
#### loadExopackageJars

```
sequenceDiagram
SystemClassLoaderAdder-->>SystemClassLoaderAdderB: installDexJars
SystemClassLoaderAdderB->>SystemClassLoaderAdderB:new SystemClassLoaderAdder()
SystemClassLoaderAdder-->>SystemClassLoaderAdderB:for dexJars[M]
SystemClassLoaderAdderB->>SystemClassLoaderAdderB:new DexClassLoader
SystemClassLoaderAdderB-->>SystemClassLoaderAdderC:addPathsOfClassLoaderToSystemClassLoader(newClassLoader)
SystemClassLoader-->SystemClassLoader:systemClassLoader.dexElements(A)
NewDexClassLoader-->SystemClassLoader:+mergedElementsArray(A+B=C)
NewDexClassLoader-->NewDexClassLoader:newClassLoader.dexElements(B=>dexJar[m])
SystemClassLoader-->SystemClassLoader:setDexElementsArray(C) to systemClassLoader
SystemClassLoaderAdder-->>SystemClassLoaderAdderB:end for dexJars[M]
```
 
```
 private void addNewClassLoaderToSystemClassLoaderWithBaseDex(
      DexClassLoader newClassLoader,
      PathClassLoader systemClassLoader) throws NoSuchFieldException, IllegalAccessException {
    Object currentElementsArray = getDexElementsArray(getDexPathList(systemClassLoader));
    Object newElementsArray = getDexElementsArray(getDexPathList(newClassLoader));
    Object mergedElementsArray = mergeArrays(currentElementsArray, newElementsArray);
    setDexElementsArray(getDexPathList(systemClassLoader), mergedElementsArray);
  }
 private Object getDexPathList(BaseDexClassLoader classLoader)
      throws NoSuchFieldException, IllegalAccessException {
    return getField(classLoader, BaseDexClassLoader.class, "pathList");
  }
```


#### ExopackageSoLoader

```
sequenceDiagram
ExopackageSoLoader->>ExopackageSoLoader: verifyMetadataFile
ExopackageSoLoader->>ExopackageSoLoader: preparePrivateDirectory
ExopackageSoLoader->>ExopackageSoLoader: parseMetadata

ExopackageSoLoader-->>ExopackageSoLoaderB:loadLibrary(when app need load lib)
ExopackageSoLoaderB-->>ExopackageSoLoaderC:copySoFileIfRequired()
ExopackageSoLoaderC->>ExopackageSoLoaderC:data/local/tmp/exopackage/" + context.getPackageName() + "/native-libs =>> exo-libs(path)

ExopackageSoLoaderB->>ExopackageSoLoaderB: System.load(path)
```

```
  public static void loadLibrary(String shortName) throws UnsatisfiedLinkError {
    if (!initialized) {
      Log.d(TAG, "ExopackageSoLoader not initialized, falling back to System.loadLibrary()");
      System.loadLibrary(shortName);
      return;
    }

    String libname = shortName.startsWith("lib") ? shortName : "lib" + shortName;

    File libraryFile = copySoFileIfRequired(libname);

    if (libraryFile == null) {
      throw new UnsatisfiedLinkError("Could not find library file for either ABIs.");
    }

    String path = libraryFile.getAbsolutePath();

    Log.d(TAG, "Attempting to load library: " + path);
    System.load(path);
    Log.d(TAG, "Successfully loaded library: " + path);
}
```

