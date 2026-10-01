---
title : 'DeepSeek 论文课（十四）：让模型"查表"记住知识：Engram 记忆'
date : 2026-09-30T16:52:00+08:00
categories : ["DeepSeek"]
tags : ["deepseek","ai论文","系列","Engram","记忆"]
description : "把一部分知识『查出来』而不是『算出来』：哈希查表、门控融合，代价是查表系统占约 3%；它不解决多步推理。"
---

# 让模型"查表"记住知识：Engram 记忆

**先给结论：** 模型的知识一直存在**参数**里——要用就得"算出来"，很贵。Engram 提出一种补充：把一部分知识放成**一张可以查的表**，用的时候直接"查出来"，几乎不花计算。代价是表越大越占内存，而且查错了会引入噪声。

## 一、先分清两种"记住"

人记东西有两种方式：

- **算出来**：别人问你"23 × 17 等于几"，你在心里一步步算。
- **查出来**：别人问你"中国的首都是哪"，你根本不用算，直接想起来——本质上是从记忆里"查"到的。

现在的大模型全靠第一种。所有知识都压缩在参数里，每次要用都得走一遍完整的网络计算。

**这意味着：知道"中国的首都是北京"，和知道一个复杂的推理步骤，花的计算量是一样的。** 这在效率上很浪费——明明有些知识是死记硬背型的。

Engram 的想法就是：**给模型配一本"参考书"，让它把一部分死记硬背的知识直接查出来。**

## 二、具体怎么查：用"词组"当地址

<div style="margin:22px 0">
<svg viewBox="0 0 800 340" width="100%" style="max-width:800px;margin:0 auto" role="img" xmlns="http://www.w3.org/2000/svg"><defs><marker id="ah-d14" markerWidth="9" markerHeight="9" refX="7.2" refY="3.2" orient="auto"><path d="M0,0 L8,3.2 L0,6.4 z" fill="currentColor"/></marker></defs><text x="400.0" y="20.0" text-anchor="middle" font-size="14" font-weight="bold" fill="currentColor">Engram：把一部分知识「查出来」而不是「算出来」</text><rect x="7.0" y="52.0" width="126.0" height="56.0" rx="9" fill="currentColor" fill-opacity="0.06" stroke="currentColor" stroke-width="1.4"/><text x="70.0" y="77.0" text-anchor="middle" font-size="13" fill="currentColor">上下文</text><text x="70.0" y="94.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">最近几个字</text><rect x="139.0" y="52.0" width="126.0" height="56.0" rx="9" fill="currentColor" fill-opacity="0.06" stroke="currentColor" stroke-width="1.4"/><text x="202.0" y="77.0" text-anchor="middle" font-size="13" fill="currentColor">组成词组</text><text x="202.0" y="94.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">2 阶 / 3 阶</text><rect x="271.0" y="52.0" width="126.0" height="56.0" rx="9" fill="currentColor" fill-opacity="0.06" stroke="currentColor" stroke-width="1.4"/><text x="334.0" y="77.0" text-anchor="middle" font-size="13" fill="currentColor">哈希算地址</text><text x="334.0" y="94.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">几个函数各算一次</text><rect x="403.0" y="52.0" width="126.0" height="56.0" rx="9" fill="currentColor" fill-opacity="0.13" stroke="currentColor" stroke-width="1.4"/><text x="466.0" y="77.0" text-anchor="middle" font-size="13" fill="currentColor">查表取向量</text><text x="466.0" y="94.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">命中即取出</text><rect x="535.0" y="52.0" width="126.0" height="56.0" rx="9" fill="currentColor" fill-opacity="0.06" stroke="currentColor" stroke-width="1.4"/><text x="598.0" y="77.0" text-anchor="middle" font-size="13" fill="currentColor">门控融合</text><text x="598.0" y="94.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">决定用多少</text><rect x="667.0" y="52.0" width="126.0" height="56.0" rx="9" fill="currentColor" fill-opacity="0.06" stroke="currentColor" stroke-width="1.4"/><text x="730.0" y="77.0" text-anchor="middle" font-size="13" fill="currentColor">注入隐藏状态</text><text x="730.0" y="94.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">参与后续计算</text><line x1="134.0" y1="80.0" x2="138.0" y2="80.0" stroke="currentColor" stroke-width="1.4" marker-end="url(#ah-d14)"/><line x1="266.0" y1="80.0" x2="270.0" y2="80.0" stroke="currentColor" stroke-width="1.4" marker-end="url(#ah-d14)"/><line x1="398.0" y1="80.0" x2="402.0" y2="80.0" stroke="currentColor" stroke-width="1.4" marker-end="url(#ah-d14)"/><line x1="530.0" y1="80.0" x2="534.0" y2="80.0" stroke="currentColor" stroke-width="1.4" marker-end="url(#ah-d14)"/><line x1="662.0" y1="80.0" x2="666.0" y2="80.0" stroke="currentColor" stroke-width="1.4" marker-end="url(#ah-d14)"/><rect x="40.0" y="150.0" width="720.0" height="170.0" rx="10" fill="currentColor" fill-opacity="0.025" stroke="currentColor" stroke-width="1" stroke-dasharray="3 4" opacity="0.9"/><text x="50.0" y="166.0" text-anchor="start" font-size="11" opacity="0.72" fill="currentColor">容量算术（可以自己验算）</text><text x="400.0" y="186.0" text-anchor="middle" font-size="13" fill="currentColor">记忆表参数 = 阶数 × 每阶槽位数 × 每槽维度</text><text x="400.0" y="210.0" text-anchor="middle" font-size="13" opacity="0.85" fill="currentColor">2 × 2,262,400 × 1280 ≈ 57.9 亿 ≈ 5.7B 参数</text><rect x="95.0" y="237.0" width="250.0" height="50.0" rx="9" fill="currentColor" fill-opacity="0.13" stroke="currentColor" stroke-width="1.4"/><text x="220.0" y="259.0" text-anchor="middle" font-size="13" fill="currentColor">查表是常数时间</text><text x="220.0" y="276.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">表变大，每词计算不变</text><rect x="435.0" y="237.0" width="290.0" height="50.0" rx="9" fill="currentColor" fill-opacity="0.1" stroke="currentColor" stroke-width="1.4" stroke-dasharray="5 4"/><text x="580.0" y="259.0" text-anchor="middle" font-size="13" fill="currentColor">代价：占内存 + 查错带噪声</text><text x="580.0" y="276.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">「不慢」不等于「更快」</text><text x="400.0" y="318.0" text-anchor="middle" font-size="12" opacity="0.8" fill="currentColor">它不解决推理：查得到「首都→北京」，查不出多步逻辑</text></svg>
<p style="text-align:center;font-size:13px;opacity:.7;margin:8px 0 0">Engram 的查表流程与容量算术；它是补充而非替代计算</p>
</div>



查表的关键是**地址怎么定**。

如果直接为所有可能的 N-gram 组合各建一个条目，组合空间会大到不可行；而且单个词的 key 抓不住"地名 + 序号"这种多词实体。Engram 的做法是**按连续的词组查**：

- 取当前位置往前 2 个字、3 个字组成的"词组"（论文里叫 **2-gram / 3-gram**，就是 2 个或 3 个连续的文字单位）
- 把这个词组通过几个不同的**哈希函数**（一种把任意内容压成固定编号的方法，好比给每本书编一个书架号）算出编号
- 拿编号去表里取出对应的向量

**为什么用好几个哈希函数？** 这是哈希表的老技巧：一个函数容易"撞车"（不同词组算出同一个编号），几个函数各查一次再拼起来，撞车概率就低了。

## 三、为什么这样"不花计算"

这是全篇最漂亮的一点。

正常的网络计算，成本随"模型多大"增长。而**查表**：算地址 + 取内存，是常数时间——**不管表有多大，查一次的开销几乎不变。**

所以**"计算量不随表变大"是这个架构的性质，不是实验结论**。真正被实验测的是另一件事：固定同一个 3B 骨干，把表的槽位从约 25.8 万一路扫到 1000 万，**验证损失在 log 空间呈线性下降**（幂律）。这条才是有用的——它说明"表容量"是个可预测的旋钮：你知道想要多少改进，就能算出要开多大的表。

> 而"系统侧不慢"的证据是另一张表：把 1000 亿参数的表卸载到主机内存，开销 1.92%～2.78%。两张表说的是两件事，别混。

翻译成人话：**这是"用内存换计算"的路子。** 内存（存储）比计算便宜，而且可以扩充；算力（运算）贵且难扩充。

**但这里有个必须说清的边界。** "计算量不增加"指的是**查表这个动作本身**，不代表整个模型的运行完全不受影响：

- 表里的向量最终还是要**参与后面的计算**，所以它会增加一点工作量（论文报告这部分开销很小）。
- 表本身占的内存是真的实物——放进运算卡就挤占了别的东西，所以论文才要把它**卸载到主机内存**（见第五节）。
- 查得准不准，取决于哈希设计，**表大不等于记得好**。

**所以准确的说法是："查表路径的边际计算成本极低，但不是零，而且它的瓶颈从算力换成了内存带宽。"** 这是第十二篇那个纪律的又一次应用：**同一个数字换个口径就变成另一句话。**

## 四、一个能自己验算的容量算术

论文里有个 27B 规模的配置（总参数 26.7B），它的记忆表是这样算的：

- 2 个阶（2-gram 和 3-gram）
- 每阶 2,262,400 个槽位〔**作者报告**（论文配置表披露值）；"它是 8 个哈希头合计的索引空间"此一解释为本指南推断，单头素数表大小论文未给〕
- 每个槽位存 1280 维的向量

**参数量的算法和普通网络一样：槽位数 × 每个槽存的数字个数。**

2 × 2,262,400 × 1280 ≈ **57.9 亿**〔槽数为作者报告，乘法为本指南重建〕 ≈ 5.7B 参数。

> ⚠️ 这一行能自己复算（2 × 2,262,400 × 1,280 = 5,791,744,000），占 26.7B 的 21.7%。但**它是 Engram 论文 27B 配置下的数**，别和 V4.1 旗舰的"约 1600 万条/头"混。

所以 26.7B 的总参数里，**约 5.7B 是"查表"的记忆**，剩下的约 21B 是正常的网络骨干。**这就是"条件记忆"这个名字的由来**：记忆是被条件触发的（查到了就用），不是每次都参与计算。

## 五、他们做了什么实验证明有用

论文用了**四个模型、三个规模**（4.1B / 26.7B / 39.5B）做对照，用同样的数据、同样的训练预算，比较"加记忆"和"不加记忆"。其中 27B 档位其实有两个模型——一个参数量对齐、一个训练计算量对齐（iso-parameter 与 iso-FLOPs），是两组独立对照。

**结果（MMLU，一个综合知识测试）：加了 Engram 的版本更好。**

这里必须说清一个**论文自身的不一致**：摘要是 "MMLU +3.4"，正文写 "+3.0"，表格里给出的是具体的两个分数——用表格相减是 **+3.0**。**摘要和正文对不上，是论文常见的小毛病。**

（顺带解开一个谜：那个 +3.4 其实和另一个测试（MMLU-Redux）的分数差 64.0 − 60.6 = 3.4 精确对应——**很可能摘要把两个测试的数字串了行。** 这是我们自己的推断，论文没解释。）

**另一个实验更实用**：他们把一个 **1000 亿参数**的表**卸载到主机内存**（不放在运算卡上），发现速度几乎不掉（开销 **1.92%～2.78%**）。这是**保守估计**：全走 PCIe、用的是 nano-vLLM 原型，不是生产推理栈。原理是他们用了一种"提前预取"的技巧——趁前面的计算还在跑，先把后面要查的东西搬过来，把搬运时间藏起来。

## 六、代价与失效场景（论文自己列的）

没有白拿的好处：

1. **查错会带噪声**：哈希"撞车"会取到不相关的向量，混进正常计算里。而且静态嵌入对多义词完全没有区分能力——"苹果"是水果还是公司，表里存的是同一个向量。
2. **需要预取窗口**：卸载到主机内存的那套技巧，依赖"前面的计算还没跑完"这段时间来搬数据。如果计算太快或者预取来不及，就会卡住。
3. **"不慢"不等于"更快"**：论文证明的是"加了大表速度不掉"，**不是"比纯 MoE 快"**。论文没有给出这个对照。这是两件不同的事，容易被混为一谈。
4. **它不解决推理问题**：表里存的是"关联"，不是"逻辑"。查得到"首都→北京"，查不出多步推理。

## 七、从论文到产品：它进了旗舰模型

一个很少见的后续：**这个概念后来真的进了产品。**

在 V4.1-Flash 旗舰模型里，Engram 作为记忆模块被正式采用：**196B 的 Engram 参数**，做成 2 个模块分别插在**第 1 层和第 14 层**，而语言骨干是 552B。

论文还给了它的配置细节：用 2、3、4 阶词组（比论文原版的 2、3 阶更宽）、8 个头、每头约 1600 万条表项、精度是 FP8。

**注意这是两笔独立的账**：552B 是骨干，196B 是记忆表，加起来才是完整体量。**引用时千万不要把 196B 算进骨干**，也不要把它们合并成一个数字去和其他模型比——**分母不同，比出来的结论就是错的**（这正是第十二篇六道检查里的"分母结构"那一关）。

> 顺带一个容易被夸大的点：V4.1 报告在 **§2.1** 专门澄清，它是"第一个**从语言预训练第一天起就原生多模态**的旗舰"，而**不是"第一次做多模态"**——多模态这条线前面已经走了好几代（第十一篇讲过）。**"第一个 X"这类说法，一定要看清 X 的完整定语。**

## 八、这条线教给我们什么

Engram 的价值不在于某个具体数字，而在于它给出了一个**明确的分工思路**：

- **该"算"的**：需要逻辑、组合、推理的部分——交给网络。
- **该"查"的**：死记硬背、模式固定的部分——交给表。

这和第七篇的 MoE（按需激活专家）、第九篇的 KV 压缩（只存摘要）是**同一个思路的三次应用**：**不要平均地花钱，要按需花。**

**效率的本质不是"少做"，而是"对不同的内容用不同贵的方式处理"。**

第十五篇，我们看这条路走到推理服务上的样子：模型回答你的时候，怎么用类似的思路把速度提上来。




## 原文出处

本系列的数字都来自下面这些报告（**已逐篇核验**）。编号后的 vN 是本系列核对时对应的版本——如果你打开的另一版数字不一样，先看版本号。

- Engram 条件记忆论文：[arXiv:2601.07372v2](https://arxiv.org/abs/2601.07372)
- 第七节"它进了旗舰模型"的 V4.1-Flash 数字：[arXiv:2609.19969v1](https://arxiv.org/abs/2609.19969)
- 第五节对比的基线来自 DeepSeekMoE：[arXiv:2401.06066v1](https://arxiv.org/abs/2401.06066)

> **数字血统提醒（本系列口径纪律的自我要求）：** 本篇出现了四个"表规模"数字，分属两篇论文、多个实验，引用时必须带上出处与口径——**2,262,400** 是 Engram 论文里每个压缩阶的槽数（跨 8 个哈希头，单头素数表大小论文未披露，这是**推断**值）；**约 1600 万条/头** 是 **V4.1 旗舰**的口径；**5.79B** 是 Engram 论文 27B 配置下的记忆参数量；而"不到 3%"属于 1000 亿参数表的**卸载实验**，与 5.79B 不是同一个实验。

---

**一句话总结：** Engram 把部分知识从"算出来"改成"查出来"——用哈希词组当地址查表，计算不随表变大，换来省算力，代价是查错的噪声和内存占用。

**记住这个数字：** 2 × 2,262,400 × 1280 ≈ 5.7B——一个 27B 模型的记忆表容量就是这么算出来的，占它总参数的两成多。

**下一篇：** [让模型说话更快：投机解码与 DSpark](/posts/deepseek-paper-course-15-dspark/)——一次不只猜一个词，还得猜得准。
