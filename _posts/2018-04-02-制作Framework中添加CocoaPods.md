---
layout: post
title: 制作Framework中添加CocoaPods
date: 2018-04-02 13:05:00.000000000 +09:00
---





### CoCall IMLib
由于需要把CoCall中IM部分制作成SDK提供给第三方公司使用所以接触了需要给Framework添加Pods依赖库，总结一些需要注意的地方。

### 制作中可能遇见的问题

1.新建 Podfile，如下编写：

![](/assets/images/2018/PodFile.png)

使用 dynamic frameworks，必须要在Podfile文件中添加 use_frameworks!

2.选择 PROJECR/TARGET -> Build Settings -> Allow non-modular includes in Framework Modules -> YES

![](/assets/images/2018/FrameworkModules.png)

3.如果是静态库 category不能被识别

找到主工程的 target －－Build Setting－－Linking－－更改其 Other Linker Flags 为： -all_load 或 -force_load 即可

### 使用Framework方式
1.将framework导入工程
A.动态库使用方式

  ![](/assets/images/2018/setting.jpg)
  
B.静态库无需上述步骤  

  