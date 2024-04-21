# LLM推理优化

用户关注的推理性能指标：

- 吞吐量：系统在单位时间内能处理的token数量。=系统处理完的token个数(输入和输出序列长度之和)/对应耗时。

  从系统角度来看，吞吐量越高，系统的资源利用率越高，系统成本越低。

- 时延：用户平均受到每个token所需时间。=用户发出请求到受到完整回复的耗时/生成序列的长度。

  从用户角度来看，时延越小，用户体验越好。

[LLM推理优化系统工程概述](https://zhuanlan.zhihu.com/p/680635901)

[LLM（十八）：LLM 的推理优化技术纵览](https://zhuanlan.zhihu.com/p/642412124)

[LLM大模型推理部署优化技术综述](https://zhuanlan.zhihu.com/p/655557420)

![image-20240421213447234](/home/zuo/.config/Typora/typora-user-images/image-20240421213447234.png)

为了提高llm的推理性能，当前业界常用的优化手段如下所示：

## 显存

#### KV cache

#### Paged Attention

[LLM推理优化技术综述：KVCache、PageAttention、FlashAttention、MQA、GQA](https://zhuanlan.zhihu.com/p/655325832)



## 计算

#### 算子融合

#### 高性能算子

## 量化

#### 权重量化

#### KVcache量化

## 分布式

#### 张量并行

#### 流水线并行

#### 通信优化

## 服务

#### Continuous Batching

#### Dynamic Batching

#### Async Serving

## 其他









