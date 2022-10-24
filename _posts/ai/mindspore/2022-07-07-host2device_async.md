---
layout: post
title: "内存同步模式"
author: dazuo
date: 2020-07-02 20:19:00 +0800
categories: [MindSpore]
tags: [内存同步]
math: true
mermaid: true
---



原始：

```cpp
AscendSession::RunOpImplOrigin()
			↓
AscendSession::LoadInputData()  // load input data to device
			↓
for (size_t i = 0; i < inputs.size(); ++i) {
    AscendDeviceAddress::SyncHostToDevice()
}
			↓
		SyncStream()
		SyncMemory()
			↓
		rtMemcpy()
```



修改后：

```cpp
AscendSession::RunOpImplOrigin()
			↓
AscendSession::LoadInputData()  // load input data to device
			↓
for (size_t i = 0; i < inputs.size(); ++i) {
    AscendDeviceAddress::SyncHostToDevice()
}
			↓
		SyncMemory()
			↓
		MemcpyAsync()
			↓
		rtMemcpy()
```



对比图如下：

![image-20220801150230828](../../../img/memsync/memsync.png){: width="1086" height="542"}



LoadInputData里按照input的数量循环调用SyncHostToDevice->CopyHostMemToDeviceAsync->cudaMemcpyAsync进行Host2Device的内存同步操作，但是最后加了SyncStream操作，GPU上底层调用的就是cudaStreamSynchronize，使得相当于调用的是同步内存拷贝，影响性能。

