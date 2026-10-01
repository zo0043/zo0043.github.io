---
title : 'DeepSeek 论文课（十三）：同样一万张卡，怎么花出两倍效果'
date : 2026-09-30T16:50:00+08:00
categories : ["DeepSeek"]
tags : ["deepseek","ai论文","系列","系统与成本"]
description : "557.6 万美元怎么算出来的、又为什么不等于『训练成本』：2,788K 卡时分项相加、通信藏进计算，以及被排除在外的部分。"
---

# 同样一万张卡，怎么花出两倍效果

**先给结论：** 训练大模型最贵的东西不是"卡好不好"，而是**每一块钱买到了多少有效计算**。DeepSeek 在这件事上做了两件事——把"每美元的效率"当成仪表盘来管，把一次旗舰训练的账**逐项列出来**。后者比前者更罕见：整个行业几乎没人公布成本明细。

## 一、先讲一个反直觉的事实：他们用的是更便宜的卡

大模型集群通常用"卡间直连"的顶级方案（运算卡之间用专用高速通道互联）。而 DeepSeek 2021 年建的集群用了 **10,000 张 PCIe 版本的 A100**——也就是**走普通主板通道、不是顶级互联**的版本。

为什么选便宜方案？因为 2021 年做这个决定时，非大语言模型的负载普遍不到十亿参数量级、显存要求也低，对"卡间通信"的要求没那么变态。省下的钱和更稳的机房，在当时是划算的。

后来大语言模型来了，这个选择就变成了负债：语言模型训练时，卡与卡之间要频繁交换梯度，通信一慢，卡就闲置——**买的算力在排队**。

**他们的应对不是推倒重来，而是在同一个物理集群上把软件重做一遍。** 这才是"系统与成本"这条线的主题。

## 二、CPERF：把"每美元买到多少计算"当仪表盘

论文提出一个记账指标，大意是：**同样的钱，你能拿到多少真实算力**。它的算法是把"计算效率"除以"成本占比"（都以一台参考机器的规格为基准）。

论文给出 **1.38** 这个数字（用表格里四舍五入后的输入复算，得到 1.3833）。翻译成大白话：**在相同预算下，这套方案能买到约 1.38 倍的有效计算。**

这里有一个很重要的口径细节：**这个比值高度依赖当时的显卡价格。** 论文自己也说，结论绑定 2021 年的硬件行情。换个年份、换张卡价，这个数字就不成立了。

> 这就是本系列反复强调的：任何"我们便宜 X%"都自带一个时间戳和价格前提。看到具体数字，先问"哪一年、什么价"。

## 三、通信库：为什么自己写一个

集群里最费时的操作叫 **allreduce**——把一万张卡各自算出的梯度汇总起来，让所有卡更新到同一个模型。

通用的做法（业界标准库）假设卡之间有高速直连通道。但在 PCIe 集群上，这个假设不成立，它会绕远路、消耗大量带宽。DeepSeek 自己写了一个方案，核心思路是：**让 CPU 异步地去搬运这些数据**，而不是让昂贵的运算卡停下来等通信。

他们把这一层和上层的并行策略（怎么切模型、怎么切数据、怎么排队错峰）成套重做。论文声称的结果是三条，**要分开看**：

| 论文的说法 | 具体数字 | 口径 |
|---|---|---|
| 算力表现约为对手的八成 | **83.0%** | 两项矩阵运算性能的合并比值 |
| 成本约为一半 | 整机含网络 **50.4%**（单机 **59.2%**）| 采购价格比，非运行成本 |
| 省约四成能耗 | 每节点功耗 **59.5%** | 峰值功耗比 |

**注意这三条不是同一件事**：不是"只有 80% 的速度"，而是"用一半的钱拿到约八成的性能分"——这是一笔性价比账。**而且这些比值绑死 2021 年的硬件价格**，换个年份就不成立。

## 四、训练一次旗舰到底花多少钱：一张能逐项相加的账

这是全系列里我最推荐高中生看的一段，因为它展示了**科研论文可以诚实到什么程度**。

DeepSeek-V3 的论文直接给了一张成本表：

| 阶段 | H800 卡时 | 金额 |
|---|---|---|
| 预训练 | 2,664K | $5.328M |
| 上下文延长（把窗口从 4K 拉到 128K）| 119K | $0.238M |
| 后训练（对齐阶段）| 5K | $0.01M |
| **合计** | **2,788K** | **$5.576M** |

**这张表可以自己验算**：2,664K + 119K + 5K = 2,788K ✓；三个金额相加也是 557.6 万美元 ✓；而 2,788K × $2（论文假设的每卡时租金）= 557.6 万美元，同样吻合 ✓。

**"卡时"就是"一张卡跑一小时"**，是最朴素的计费单位。整个预训练用了 2048 张卡、54.2 天。

## 五、这 557.6 万美元不包含什么（最重要的一段）

论文明确说明：**这只是"正式训练"的直接租金。**

不包含的至少有这么几类：

- 前期研究和大量失败的实验（探索阶段往往比正式训练更烧钱）
- 算法消融实验（"这么改到底有没有用"的对照实验）
- 人员工资
- 机房、电力、网络等基础设施的摊销

**所以"557.6 万美元训出 V3"是一个被截断了口径的说法。** 媒体常把它当作"全部成本"来报道，这是典型的**口径截断**——数字是真的，但少说了它只是总账里的一格。

**一个行业的对照**：别的公司几乎从不公布这类明细。公布出来会被拿去比较、会被质疑，但**比较的前提正是透明**。这也是这个系列想说的一点：**最值得信任的论文，是那种肯把账摊开让你算的论文。**

## 六、为什么一定要把"通信"藏起来

<div style="margin:22px 0">
<svg viewBox="0 0 800 340" width="100%" style="max-width:800px;margin:0 auto" role="img" xmlns="http://www.w3.org/2000/svg"><defs><marker id="ah-d13" markerWidth="9" markerHeight="9" refX="7.2" refY="3.2" orient="auto"><path d="M0,0 L8,3.2 L0,6.4 z" fill="currentColor"/></marker></defs><text x="400.0" y="20.0" text-anchor="middle" font-size="14" font-weight="bold" fill="currentColor">把通信藏在计算下面</text><text x="70.0" y="58.0" text-anchor="start" font-size="12" fill="currentColor">没有重叠</text><rect x="150" y="44" width="88" height="30" rx="4" fill="currentColor" fill-opacity="0.1" stroke="currentColor" stroke-width="1"/><text x="194.0" y="63.0" text-anchor="middle" font-size="12" fill="currentColor">算</text><rect x="242" y="44" width="88" height="30" rx="4" fill="currentColor" fill-opacity="0.03" stroke="currentColor" stroke-width="1" stroke-dasharray="4 3"/><text x="286.0" y="63.0" text-anchor="middle" font-size="12" fill="currentColor">等</text><rect x="334" y="44" width="88" height="30" rx="4" fill="currentColor" fill-opacity="0.1" stroke="currentColor" stroke-width="1"/><text x="378.0" y="63.0" text-anchor="middle" font-size="12" fill="currentColor">算</text><rect x="426" y="44" width="88" height="30" rx="4" fill="currentColor" fill-opacity="0.03" stroke="currentColor" stroke-width="1" stroke-dasharray="4 3"/><text x="470.0" y="63.0" text-anchor="middle" font-size="12" fill="currentColor">等</text><rect x="518" y="44" width="88" height="30" rx="4" fill="currentColor" fill-opacity="0.1" stroke="currentColor" stroke-width="1"/><text x="562.0" y="63.0" text-anchor="middle" font-size="12" fill="currentColor">算</text><rect x="610" y="44" width="88" height="30" rx="4" fill="currentColor" fill-opacity="0.03" stroke="currentColor" stroke-width="1" stroke-dasharray="4 3"/><text x="654.0" y="63.0" text-anchor="middle" font-size="12" fill="currentColor">等</text><text x="70.0" y="118.0" text-anchor="start" font-size="12" fill="currentColor">有重叠</text><rect x="150" y="104" width="92" height="30" rx="4" fill="currentColor" fill-opacity="0.10" stroke="currentColor" stroke-width="1"/><rect x="180" y="104" width="30" height="30" rx="4" fill="currentColor" fill-opacity="0.03" stroke="currentColor" stroke-width="1" stroke-dasharray="4 3"/><text x="196.0" y="123.0" text-anchor="middle" font-size="12" fill="currentColor">算</text><rect x="246" y="104" width="92" height="30" rx="4" fill="currentColor" fill-opacity="0.10" stroke="currentColor" stroke-width="1"/><rect x="276" y="104" width="30" height="30" rx="4" fill="currentColor" fill-opacity="0.03" stroke="currentColor" stroke-width="1" stroke-dasharray="4 3"/><text x="292.0" y="123.0" text-anchor="middle" font-size="12" fill="currentColor">算</text><rect x="342" y="104" width="92" height="30" rx="4" fill="currentColor" fill-opacity="0.10" stroke="currentColor" stroke-width="1"/><rect x="372" y="104" width="30" height="30" rx="4" fill="currentColor" fill-opacity="0.03" stroke="currentColor" stroke-width="1" stroke-dasharray="4 3"/><text x="388.0" y="123.0" text-anchor="middle" font-size="12" fill="currentColor">算</text><rect x="438" y="104" width="92" height="30" rx="4" fill="currentColor" fill-opacity="0.10" stroke="currentColor" stroke-width="1"/><rect x="468" y="104" width="30" height="30" rx="4" fill="currentColor" fill-opacity="0.03" stroke="currentColor" stroke-width="1" stroke-dasharray="4 3"/><text x="484.0" y="123.0" text-anchor="middle" font-size="12" fill="currentColor">算</text><rect x="534" y="104" width="92" height="30" rx="4" fill="currentColor" fill-opacity="0.10" stroke="currentColor" stroke-width="1"/><rect x="564" y="104" width="30" height="30" rx="4" fill="currentColor" fill-opacity="0.03" stroke="currentColor" stroke-width="1" stroke-dasharray="4 3"/><text x="580.0" y="123.0" text-anchor="middle" font-size="12" fill="currentColor">算</text><rect x="630" y="104" width="92" height="30" rx="4" fill="currentColor" fill-opacity="0.10" stroke="currentColor" stroke-width="1"/><rect x="660" y="104" width="30" height="30" rx="4" fill="currentColor" fill-opacity="0.03" stroke="currentColor" stroke-width="1" stroke-dasharray="4 3"/><text x="676.0" y="123.0" text-anchor="middle" font-size="12" fill="currentColor">算</text><text x="400.0" y="152.0" text-anchor="middle" font-size="12" opacity="0.8" fill="currentColor">虚线是通信时间：重叠后它落在计算的时间段内，卡不再空等</text><rect x="40.0" y="170.0" width="720.0" height="150.0" rx="10" fill="currentColor" fill-opacity="0.025" stroke="currentColor" stroke-width="1" stroke-dasharray="3 4" opacity="0.9"/><text x="50.0" y="186.0" text-anchor="start" font-size="11" opacity="0.72" fill="currentColor">一次旗舰训练的成本表（可以逐项相加）</text><rect x="52.5" y="214.0" width="155.0" height="52.0" rx="9" fill="currentColor" fill-opacity="0.06" stroke="currentColor" stroke-width="1.4"/><text x="130.0" y="237.0" text-anchor="middle" font-size="13" fill="currentColor">预训练</text><text x="130.0" y="254.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">2,664K 卡时</text><rect x="227.5" y="214.0" width="155.0" height="52.0" rx="9" fill="currentColor" fill-opacity="0.06" stroke="currentColor" stroke-width="1.4"/><text x="305.0" y="237.0" text-anchor="middle" font-size="13" fill="currentColor">上下文延长</text><text x="305.0" y="254.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">119K</text><rect x="402.5" y="214.0" width="155.0" height="52.0" rx="9" fill="currentColor" fill-opacity="0.06" stroke="currentColor" stroke-width="1.4"/><text x="480.0" y="237.0" text-anchor="middle" font-size="13" fill="currentColor">后训练</text><text x="480.0" y="254.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">5K</text><rect x="577.5" y="214.0" width="155.0" height="52.0" rx="9" fill="currentColor" fill-opacity="0.13" stroke="currentColor" stroke-width="1.4"/><text x="655.0" y="237.0" text-anchor="middle" font-size="13" fill="currentColor">合计</text><text x="655.0" y="254.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">2,788K ≈ 557.6 万美元</text><text x="400.0" y="296.0" text-anchor="middle" font-size="12" opacity="0.82" fill="currentColor">但这只是「正式训练」的直接租金：不含研究、消融、工资与机房</text></svg>
<p style="text-align:center;font-size:13px;opacity:.7;margin:8px 0 0">流水线重叠让通信不再浪费时间；成本表可逐项相加，但口径仅限正式训练</p>
</div>



要理解这笔账为什么能这么算，得先知道训练时最大的浪费在哪里。

第七篇讲过 MoE：每次处理一个词，模型只叫醒少数几个"专家"。但这里有个系统的麻烦——**专家分布在不同机器上**。于是每处理一步，就要把词元通过机器间的网络送到对应专家那里，算完再送回来。这一来一回叫**全对全通信**。

**问题在于：通信的时候，运算卡是闲着的。**

如果通信占 30% 的时间，就等于**每买 10 张卡，有 3 张在干等**——钱在空气里烧。所以训练系统的核心 KPI 不是"卡跑多快"，而是**"卡有多少时间真的在算"**。

DeepSeek 的做法叫**双向流水线**：把一整套训练流程切成很多小段，让不同的机器同时处理不同的小段——**当 A 组在通信时，B 组正在计算**。这样通信的时间就被计算的时间盖住了。

**打个比方**：一个人洗菜、一个人切菜、一个人炒菜，如果每个人都必须等前一个人完全做完才动手，锅就得空着。流水线就是让所有人在同一时刻都有活干。

> 但这里有个前提：**如果专家被分配得不均衡**（某个专家被叫醒太多次，另一台机器闲着），流水线就会在某一段卡住，重叠失效。**所以"均衡"不只是模型质量问题，还是系统效率问题**——这也正是第七篇"无辅助损失均衡"存在的原因。**模型设计和系统设计，从来不是两件事。**

## 七、最常坏的零件，不是算力卡

集群运维论文里有一份按故障类型统计的表格（过去一年共 12,970 条故障记录）。结果是：

- **NVLink 桥接器错误占 42.57%**（这是卡间高速互联部件，反而最常坏）
- 另一类错误占 33.48%
- 内存纠错（ECC）相关只占 2.14%

**结论很朴素但很实用：万卡集群里最脆弱的环节是连接，不是计算。** 但要说清他们到底做了什么：对反复出同一错误的桥接部件，办法是**反复压测把它筛出来隔离**，再靠**每 5 分钟一次 checkpoint** 兜底；论文自己承认，对 xid74 这类故障**没有预防手段**。

> 这里有一个**值得学的方法：先复算引文，再判断修辞。** 论文说这类故障"比其他硬件故障高几个数量级"，但拿表格原始计数自己除一遍——xid_87 类 5,521 次，对比 xid_48 ECC 类 277 次，**5521 ÷ 277 ≈ 19.9 倍**，够不上"数量级"（十倍以上）的说法。
>
> **顺带坦白一件事：** 材料库里记的是 22.5 倍，但它和上面的原始计数除不出来（5521 ÷ x = 22.5 会得到约 245 的分母，而表格里没有这一类）。所以本系列**只用能复算的 19.9 倍**——一个教人"每个数字都要自己除一遍"的系列，引用二手复算值时也该落地跑一遍。

## 八、论文自己算错的地方，也值得记下来

这个系列的第十二篇讲过"看数字的六道检查"。这一篇里就有两个真实样本：

1. **并行效率的算术不一致**：论文声称某配置达到 91% 的并行效率，但用它自己给的每步耗时复算，得到约 82.5%。同一个公式在另外两组配置上却精确吻合——说明这不是口径问题，而是这一处数字本身有问题。
2. **存储容量的单位含糊**：论文说存储系统"超过 20 PiB"，但严格按二进制口径（PiB，1 PiB = 2⁵⁰ 字节）核算，镜像后是 19.65 PiB。**PiB 和 PB 混着用，就能让一个数字看起来更大一点。**

**我特意写这两条，不是因为它们重要，而是因为它们证明了：再严谨的论文也需要读者自己动手核。** 这个习惯，比记住任何数字都值钱。

## 九、这条线还没到头

集群和训练成本讲完了，但还有一个问题没答：**模型训练好之后，用户问它一句话，它为什么回得那么慢？**

而且 V3 的论文里埋了一个很有意思的模块——**MTP（多词元预测）**，它训练时的目标是"一次猜不止一个词"。这个当初为了"提高数据效率"设计的模块，后来被拿到了推理服务上，变成了让回答变快的核心武器。

这就是第十五篇的主题：让模型说话更快。而第十四篇，我们先看一个更奇特的想法——**让模型把知识"查出来"，而不是"算出来"**。




## 原文出处

本系列的数字都来自下面这些报告（**已逐篇核验**）。编号后的 vN 是本系列核对时对应的版本——如果你打开的另一版数字不一样，先看版本号。

- Fire-Flyer AI-HPC 论文：[arXiv:2408.14158v2](https://arxiv.org/abs/2408.14158)
- DeepSeek-V3 技术报告：[arXiv:2412.19437v2](https://arxiv.org/abs/2412.19437)
- 正文引用的优化线索来自《Insights into DeepSeek-V3: Scaling Challenges and Reflections on Hardware for AI Architectures》（arXiv:2505.09343），它给出"MoE 推理速度上界由互连带宽而非算力决定"这一判断

> **关于本篇的机制名：** 正文里只讲了五个官方部件的**功能**（CPU 端规约、跨机通信、存储、流水线、NVLink 桥）。它们的正式名字分别是 HFReduce、HaiScale、3FS、DualPipe、NVLink Bridge——本篇按"讲清做什么"取舍，没有逐个报名字。若要查原文，用上面的 arXiv ID + 论文第九节。

---

**一句话总结：** 系统与成本这条线的核心不是堆硬件，而是把"每块钱买到多少有效计算"当仪表盘，并且愿意把一次训练的账逐项公开让读者相加验证。

**记住这个数字：** 2,788K H800 卡时 ≈ 557.6 万美元——而它只包含正式训练，不含研究、消融、工资和机房。

**下一篇：** [让模型"查表"记住知识：Engram 记忆](/posts/deepseek-paper-course-14-engram/)——把一部分记忆从"算出来"改成"查出来"。
