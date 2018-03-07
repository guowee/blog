---
title: Android热补丁动态修复技术
date: 2018-02-23 17:57:08
tags: Android
---
# 一、Dex分包方案的由来
+ Dalvik限制
当apk解压后里面是只有一个classes.dex文件，而这个dex文件里面就包含了项目的所有.classes文件。但是当一个app的功能越来越复杂，可能会出现两个问题：
    1. 编译失败，因为一个DVM 中存储方法id 用的是short类型，导致dex中方法不能超过65536个
    2. apk在Android 2.3之前的机器无法安装，因为dex文件过大（用来执行dexopt的内存只分配了5M）
+ 解决方案
针对以上问题，研究出了dex分包方案。原理就是将编译好的class文件拆分打包成两个dex,绕过dex方法数量的限制以及安装时的检查。在运行时再动态的加载第二个dex文件。
除了第一个dex文件（即正常apk包唯一包含的Dex文件），其它dex文件都以资源的方式放在安装包中，并在Application的onCreate回调中被注入到系统的ClassLoader。因此，对于那些在注入之前已经引用到的类（以及它们所在的jar）,必须放入第一个Dex文件中。



# 二、Dex分包的原理——ClassLoader