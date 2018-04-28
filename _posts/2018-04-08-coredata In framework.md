---
layout: post
title: CoreData In Framework
date: 2018-04-28 16:05:00.000000000 +09:00
---





### CoCall IMLib
我在制作IMLib时使用了CoreData作为本地存储的方式。

### 制作中遇见的问题

由于制作的是静态Framework，资源打包进Framework是读取不了的。只有动态Framework能在.app的Framework文件夹下看到，并读取.framework里的资源文件。使用[NSBundle mainBundle]获取到.app的目录，如果是动态Framework目录下会看到这个动态库以及动态库里面资源文件。静态Framework.app下的Framework目录为空。


```
/// 创建NSManagedObjectModel对象
- (NSManagedObjectModel *)managedObjectModel
{
    if (_managedObjectModel != nil) {
		return _managedObjectModel;
	}
    
    NSURL *modelURL = [[NSBundle mainBundle] URLForResource:self.modelName withExtension:@"momd"];

    _managedObjectModel = [[NSManagedObjectModel alloc] initWithContentsOfURL:modelURL];
    return _managedObjectModel;
}
```

所以通过这段代码是获取不到modelURL的，导致NSManagedObjectModel为nil

### 解决方案
制作bundle  把 .xcdatamodeld文件当做资源制作成bundle。

![](/assets/images/2018/CCData.jpg)

制作后：


![](/assets/images/2018/bundle.jpg)


```
- (NSManagedObjectModel *)managedObjectModel
{
    if (_managedObjectModel != nil) {
		return _managedObjectModel;
	}
    
    NSString *staticLibraryBundlePath = [[NSBundle bundleForClass:[self class]] pathForResource:@"CCData" ofType:@"bundle"];
    
    NSURL *staticLibraryMOMURL = [[NSBundle bundleWithPath:staticLibraryBundlePath]URLForResource:@"CCIMLib" withExtension:@"mom"];
    
    _managedObjectModel = [[NSManagedObjectModel alloc]initWithContentsOfURL:staticLibraryMOMURL];
    
    return _managedObjectModel;
}
```

  