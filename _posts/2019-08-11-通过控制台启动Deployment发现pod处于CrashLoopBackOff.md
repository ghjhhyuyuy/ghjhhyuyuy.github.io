---
layout: post
title: "通过控制台启动Deployment发现pod处于CrashLoopBackOff"
date: 2019-08-11 
description: "通过控制台启动Deployment发现pod处于CrashLoopBackOff"
tag: 博客 
---  

之前通过控制台启动过nginx镜像的deployment，所以没觉得启动一个ubuntu镜像的deployment会有什么难度。但是在实际启动的时候遇到很多问题。
# 序
首先是之前的k8s集群出现了问题，应该是其他成员重置过了，导致6443端口根本得不到响应，我后面换了一个好的集群才能正常操作。因为控制台代码是我写的，所以我开始一直以为创建不成功是我代码的问题，后面排除了各种情况，怀疑集群状态有问题。就像柯南说的，排除所有不可能的，剩下的那个即使再不可思议，那也是事实。真的是太坑了。

# 正文
### 遇到问题
之后遇到的问题是pod一直处于CrashLoopBackOff状态，通过kubectl describe po命令查看详情后发现pod一直在尝试重启失败的容器。在这里我**忽略了错误返回码**，直接按直觉判断是容器有问题，启动失败，这是我犯的一个严重的错误。因为我启动容器的镜像是从中央仓库拉取的，所以不该有问题，后面我手动启动过，确认镜像没有问题，我长时间卡在这里。最后发现**返回码为0**，也就是说容器是运行起来了的，只是快速执行完后就关闭了，所以要做的只是让容器保持运行。看到这里问题总算是要解决了。
### 真正结束
但是，对于我来说仅仅是个开始，在之后我又使用kubectl describe po、kubectl logs、journalctl -e这些状态和日志查询命令尝试并捣鼓了半天才成功使pod运行起来。为什么呢？前面提到了，控制台的后台是我写的，但是在创建deployment这个方法里我没有传入deployment的参数项，所以为了实现ubuntu容器持续运行，我必须得修改后台代码，新增两个参数，command和arg,添加了这两个参数后，我又要尝试如何传这两个参数才能得到我要的效果，因为之前我没添加过这两个参数创建pod，不过这两个参数就和他们的名称一样，一个是命令，一个是参数，command传/bin/bash这些内容，arg可以传-it或者前面命令需要的参数，到这里，问题就都解决了。
![我启动起来的pod.png] (https://upload-images.jianshu.io/upload_images/13735327-ff041193ff1910d7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

转载请注明原地址，王作文的博客：[https://ghjhhyuyuy.github.io](https://ghjhhyuyuy.github.io) 谢谢！