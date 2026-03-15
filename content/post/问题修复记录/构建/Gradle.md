# Gradle

#  2026/02

## 1

### 问题/现象

Execution failed for task ':app:ijDownloadArtifact19962fe3-559 
Could not resolve all files for configuration ':app:downloadArtifact_6ae37e31-57ec-435c-b708-ba79d5b34faf'

![image-20260206183547593](D:\study\Images\image-20260206183547593.png)

### 解决方案

- 找不到这个依赖就把这个依赖注释一下然后重新同步下载，在编译会报错，缺少依赖，然后我又注释回来就好了，正常下载依赖打包成功了，可能是IDE的问题

## 2

### 问题/现象

![image-20260206185625787](D:\study\Images\image-20260206185625787.png)



## 3.

![image-20260209143318849](C:\Users\TSCQ\AppData\Roaming\Typora\typora-user-images\image-20260209143318849.png)





## 4.NDK路径下存在还是显示的NDK不存在

![image-20260226134840493](C:\Users\TSCQ\AppData\Roaming\Typora\typora-user-images\image-20260226134840493.png)

- 解决：在哪个模块下面报的错误就在哪个模块下面再配置该地址即可









