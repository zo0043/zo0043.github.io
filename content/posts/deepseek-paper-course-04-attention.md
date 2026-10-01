---
title : 'DeepSeek 论文课（四）：注意力：模型怎么"看"上下文'
date : 2026-09-30T16:32:00+08:00
categories : ["DeepSeek"]
tags : ["deepseek","ai论文","系列","注意力"]
description : "注意力机制的三步（打分、归一、加权）讲清楚，再看 DeepSeek 如何把『读书笔记』从每头 64 份压到 8 份共用。"
---

# 注意力：模型怎么"看"上下文

**先给结论：** 模型猜下一个词时，靠的是**让前文里每个词互相"看一眼"，再按相关程度加权取用**。这套机制叫注意力。它的好处是精准，代价是**文章变长时开销按平方爆炸**——后面所有的省算力技术，几乎都在跟这笔账搏斗。

## 一、问题：猜词的时候，该听谁的

先回到上一篇的结论：模型做的事是"猜下一个词"。

现在看这句话：

> 小明把球扔给小红，然后 ___ 又把它扔了回来。

要猜对空，模型必须知道：主语应该是小红（不是小明），"它"指的是球。这些信息都在前面——但前文可能很长，模型怎么知道**该重点看哪几个词**？

如果它对前面每个词都一视同仁地平均处理，就会把"扔""球""小明""小红"混在一起，猜不出谁是主语。

**答案：让每个词提出一个"我想找什么"，再跟前文每个词的"我能提供什么"逐一比对，按相关程度加权。** 这就是注意力。

## 二、三个角色：Q、K、V

<div style="margin:22px 0">
<svg viewBox="0 0 780 340" width="100%" style="max-width:780px;margin:0 auto" role="img" xmlns="http://www.w3.org/2000/svg"><defs><marker id="ah-d04" markerWidth="9" markerHeight="9" refX="7.2" refY="3.2" orient="auto"><path d="M0,0 L8,3.2 L0,6.4 z" fill="currentColor"/></marker></defs><text x="390.0" y="20.0" text-anchor="middle" font-size="14" font-weight="bold" fill="currentColor">Q、K、V 与共享机制</text><rect x="40.0" y="34.0" width="700.0" height="130.0" rx="10" fill="currentColor" fill-opacity="0.025" stroke="currentColor" stroke-width="1" stroke-dasharray="3 4" opacity="0.9"/><text x="50.0" y="50.0" text-anchor="start" font-size="11" opacity="0.72" fill="currentColor">第一步：每个词提出「我在找什么」并比对</text><rect x="45.0" y="67.0" width="110.0" height="46.0" rx="9" fill="currentColor" fill-opacity="0.06" stroke="currentColor" stroke-width="1.4"/><text x="100.0" y="87.0" text-anchor="middle" font-size="13" fill="currentColor">当前词</text><text x="100.0" y="104.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">的词元向量</text><rect x="195.0" y="39.0" width="110.0" height="42.0" rx="9" fill="currentColor" fill-opacity="0.13" stroke="currentColor" stroke-width="1.4"/><text x="250.0" y="57.0" text-anchor="middle" font-size="13" fill="currentColor">Q 查询</text><text x="250.0" y="74.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">我在找什么</text><rect x="190.0" y="99.0" width="120.0" height="42.0" rx="9" fill="currentColor" fill-opacity="0.13" stroke="currentColor" stroke-width="1.4"/><text x="250.0" y="117.0" text-anchor="middle" font-size="13" fill="currentColor">K / V</text><text x="250.0" y="134.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">别人能提供什么</text><line x1="156.0" y1="90.0" x2="188.0" y2="90.0" stroke="currentColor" stroke-width="1.4" marker-end="url(#ah-d04)"/><rect x="330.0" y="67.0" width="140.0" height="46.0" rx="9" fill="currentColor" fill-opacity="0.06" stroke="currentColor" stroke-width="1.4"/><text x="400.0" y="87.0" text-anchor="middle" font-size="13" fill="currentColor">相关度分数</text><text x="400.0" y="104.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">Q 与前文每个 K 比对</text><line x1="312.0" y1="90.0" x2="328.0" y2="90.0" stroke="currentColor" stroke-width="1.4" marker-end="url(#ah-d04)"/><rect x="485.0" y="67.0" width="130.0" height="46.0" rx="9" fill="currentColor" fill-opacity="0.06" stroke="currentColor" stroke-width="1.4"/><text x="550.0" y="87.0" text-anchor="middle" font-size="13" fill="currentColor">归一成权重</text><text x="550.0" y="104.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">该听谁的百分比</text><line x1="472.0" y1="90.0" x2="482.0" y2="90.0" stroke="currentColor" stroke-width="1.4" marker-end="url(#ah-d04)"/><rect x="630.0" y="67.0" width="120.0" height="46.0" rx="9" fill="currentColor" fill-opacity="0.06" stroke="currentColor" stroke-width="1.4"/><text x="690.0" y="87.0" text-anchor="middle" font-size="13" fill="currentColor">加权取 V</text><text x="690.0" y="104.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">混合出结果</text><line x1="618.0" y1="90.0" x2="628.0" y2="90.0" stroke="currentColor" stroke-width="1.4" marker-end="url(#ah-d04)"/><rect x="40.0" y="180.0" width="700.0" height="140.0" rx="10" fill="currentColor" fill-opacity="0.025" stroke="currentColor" stroke-width="1" stroke-dasharray="3 4" opacity="0.9"/><text x="50.0" y="196.0" text-anchor="start" font-size="11" opacity="0.72" fill="currentColor">省钱关键：多个提问者共用一份资料</text><rect x="75.0" y="215.0" width="170.0" height="50.0" rx="9" fill="currentColor" fill-opacity="0.03" stroke="currentColor" stroke-width="1.4" stroke-dasharray="5 4"/><text x="160.0" y="237.0" text-anchor="middle" font-size="13" fill="currentColor">MHA：各配一套</text><text x="160.0" y="254.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">64 份笔记 · 最贵</text><rect x="310.0" y="215.0" width="180.0" height="50.0" rx="9" fill="currentColor" fill-opacity="0.13" stroke="currentColor" stroke-width="1.4"/><text x="400.0" y="237.0" text-anchor="middle" font-size="13" fill="currentColor">GQA：一组共用</text><text x="400.0" y="254.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">8 份笔记 · 省下 7/8</text><rect x="555.0" y="215.0" width="170.0" height="50.0" rx="9" fill="currentColor" fill-opacity="0.03" stroke="currentColor" stroke-width="1.4" stroke-dasharray="5 4"/><text x="640.0" y="237.0" text-anchor="middle" font-size="13" fill="currentColor">MQA：全部共用</text><text x="640.0" y="254.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">1 份 · 最省但最弱</text><text x="390.0" y="308.0" text-anchor="middle" font-size="12" opacity="0.78" fill="currentColor">DeepSeek 67B：64 个查询头共享 8 份笔记（每 8 个 Q 共用一份）</text></svg>
<p style="text-align:center;font-size:13px;opacity:.7;margin:8px 0 0">注意力的三步：打分 → 归一 → 加权；GQA 让多个查询头共用笔记</p>
</div>



注意力给每个词准备了三样东西，用开会打比方最直观：

| 角色 | 通俗说法 | 开会比喻 |
|---|---|---|
| **Q**（查询） | 我在找什么 | 你举手问："有没有人知道预算的事？" |
| **K**（键） | 我能提供什么 | 每个人桌上的名牌，写着"我负责预算 / 我负责排期" |
| **V**（值） | 我实际的内容 | 真正开口说的那段话 |

流程分三步：

1. 拿你的 Q，去和在场每个词的 K 比一比，算出一个**相关度分数**（越匹配分越高）。
2. 把这些分数变成一组"权重"——就像把分数折算成"我该听谁的百分比"。
3. 按权重把所有人的 V 取出来、加权混合，得到这一轮的结果。

**关键点：权重是"按相关度分配注意力"，不是"取平均"。** 如果一个词和前文所有词都无关，它的输出会退化成"一锅平均"——因为所有权重被迫摊平；如果只和某一个词强相关，它几乎就只取那一个词的内容。

### 但 Q、K、V 到底从哪来？（这一节是全篇的地基，缺了它后面都悬空）

上面的比喻容易让人以为 Q、K、V 是三样现成的东西。**不是。它们三个来自同一个词的同一个向量，被三张不同的"透镜"分别照出来的。**

具体说：模型读到"扔"这个字，先得到一条向量（可以理解成一串数字，记作 x）。然后：

- 用矩阵 **W_Q** 乘一次 → 得到"我想找什么"（Q）
- 用矩阵 **W_K** 乘一次 → 得到"我能提供什么"（K）
- 用矩阵 **W_V** 乘一次 → 得到"我实际的内容"（V）

**三张矩阵里存的数字，就是模型学出来的东西。** 训练改的主要是它们。所以"训练一个 Transformer"里很大一部分，就是在学这三张透镜。

> 这也解释了一个常被问到的问题：**为什么同一个字，在不同句子里 K 不一样？** 因为透镜只负责把 x 变成 K，x 本身才是内容——同一个字在不同上下文里得到的 x 不同（前面的层已经把它加工过了），乘同一张 W_K，出来的 K 就不同。

### 把它写成能自己算的形式

系列第五篇给过参数量公式，这里给注意力的。文字版：

- **相关度分数** = Q 和 K 的点积 = 两个向量"对应分量相乘再全加起来"，得到一个数
- **权重** = softmax(分数 ÷ √d) —— d 是每头多少维，归一成加起来等于 1
- **输出** = 权重加权求和 V

这就是全部三步。**没有任何一步需要超过"乘加"的运算**——这也是为什么注意力在 2017 年之后能立刻铺开：它在硬件上太友好了。

> 这里有个反直觉的地方：softmax 这组权重**加起来永远等于 1**，所以不存在"什么都取不到"。真正发生的是——全部无关时权重趋近均匀、输出接近所有 V 的平均（恰恰是前一句否定的那个"取平均"）；强相关时才可能一边倒。"分配"和"平均"不是两种模式，而是同一件事的两端。

> 一个小小的技术细节：算完相关度分数后会做一个"缩放"处理。点积是 128 个独立分量相加，**方差会随维度增长**，不加缩放时分数会被拉得很大、softmax 把几乎全部重量压给一个词、其他人权重趋近 0，梯度也跟着消失。缩放就是把分数拉回能区分的区间。（这是初始化层面的直觉，不是对任意训练后分布都成立的定理。）

## 三、多头：不是只看一种关系，而是同时看很多种

一次"看"往往不够。句子里的关系有很多种：

- 谁指代谁（"它"→"球"）
- 谁是动作的主语（"扔"→"小明"）
- 前后照应、并列、转折……

所以模型不只做一套注意力，而是**并行做很多套，每套自己学一种关系**，这叫**多头（multi-head）**。就像开完大会之后，还分成语法组、指代组、语气组分别对一遍，最后汇总。

> **接上上一节：这些头就是上一节那三张矩阵被切开的结果。** 假设一条向量是 512 维，要开 8 个注意力头，那么 W_Q/W_K/W_V 各自排成 **8 份小矩阵**，每份只负责 64 维。于是"每头各有自己的一套透镜"，学的自然不会是同一种关系——**这不是巧合，是参数分开的直接后果。**
>
> 所以"几个头"和"每头多少维"是同一件事的两种说法：**512 维切成 8 份，就是 8 个注意力头、每个头 64 维**。篇05 会用这条切分算参数量。

## 四、省钱的关键设计：让多个"提问者"共用一份"资料"

这里出现了第一个真正影响成本的取舍。

- **MHA（多头注意力）**：每个 Q 都配一套自己的 K、V——最灵活，也最占地方。
- **MQA**：所有 Q 共用**一套** K、V。
- **GQA**：折中——一组 Q 共用一套 K、V。

**DeepSeek 67B 用的是 64 个查询头共享 8 个 KV 头**（也就是每 8 个 Q 共用一份 K、V）。可以想象成：8 位老师各自提问（Q 不同），但**共用同一本教材**（K、V 相同）。

为什么这能省？因为在**使用（推理）阶段**，模型必须把 K、V 存下来备查——这份存储叫 **KV cache（键值缓存）**，就是上一篇提到的"读书笔记"。

关键对比是这样的：**如果每个查询头都各配一套 K、V（也就是 MHA），67B 要存 64 份；改成每 8 个查询头共用一份后，只要存 8 份。** 在**层数、长度、每头维度都不变**的前提下，这份笔记本身的存储**降到原来的八分之一**。

（注意边界：只是笔记本身降到 1/8，**不等于整个显存降到 1/8**，更不等于速度快 8 倍——权重、临时数据都还占着地方。）

### 顺带说清楚：头的宽度是怎么分的

上面反复出现"每头维度"，它其实来自一个很简单的切分：**词元向量的总宽度被平均切成若干头**，每个头只管其中一小段。

用 DeepSeek 67B 的真实配置（论文表 2）：

| 配置 | 67B |
|---|---:|
| 隐藏宽度 | 8192 |
| Q 头数 | 64 |
| 每头宽度 | 128 |

64 × 128 = 8192，正好分完。所以"每头宽度 128"不是随便定的，是**总宽度除以头数**得到的。

于是可以把"每层每个词元要存多少笔记"亲手算一遍：

> **笔记数 = 2（K 和 V 两份） × KV 头数 × 每头宽度**
> = 2 × 8 × 128 = **2048 个数字**（一层、一个词元）
> 再乘上层数 95：**2 × 8 × 128 × 95 = 194,560 个数字**

这就是"64 个 Q 共用 8 份笔记"在账面上的样子。

**再往下一步：2024 年的 MLA 把这个数字压到了 (512 + 64) × 60 = 34,560。**

按"一个数字算一个单位"来比：34,560 ÷ 194,560 = **17.76%**，也就是笔记条数少了约 **82%**。

> ⚠️ **这两个数字还有一个容易漏掉的前提：它们来自两个不同的模型。** 34,560 是 V2 的 **60 层**，194,560 是 DeepSeek 67B 的 **95 层**——82% 里既有 MLA 的压缩，也有一层"层数从 95 变成 60"的贡献。如果强行把 MLA 按在 67B 的 95 层上算，是 (512+64) × 95 = 54,720，只减少 **71.9%**。三个数字都对，但分母不同，不能混着引。
>
> 另外，93.3% 这条论文没有给出算式。上面的"6 bit vs 2 字节"是按部署口径复算出来的，结果精确吻合，但属**重建**，不是论文原文的推导——引用时值得带上这半句。

## 五、代价：每个词都要看前面所有词

注意力的强大来自"全局审视"，但它同时带来一笔很贵的账。

**第 1 个词看 0 个、第 2 个词看 1 个……第 n 个词要看前面 n−1 个。** 全部加起来，配对数就是 1 + 2 + … + (n−1) = **n × (n−1) ÷ 2**，所以复杂度大致随长度的**平方**增长。

具体一点，用同一个式子算：

- 长度 1024：1024 × 1023 ÷ 2 ≈ **52 万**对
- 长度 128K（约 13 万个词元，131,072）：131,072 × 131,071 ÷ 2 ≈ **86 亿**对

**这个式子你可以自己在纸上验一遍**——把数字代进去，比记住结论有用。长度翻倍，这个数字约变成 **4 倍**（精确地说是 4 倍减去一个可忽略的量）。

> 顺手说一个口径分歧的来源：有些地方会写成"n² 的一半"、有些写"n²"，差别在**第 t 个词算不算看它自己**。本系列的模型是"看前面 n−1 个"，所以用上面的 n(n−1)/2；换成 n² 的话，1024 就该是约 105 万、128K 就是约 172 亿。两个都对，但同一句话里只能用一套。

同时 KV cache 也在涨，但它是**线性**的——每读进一个词元，就要多存一份笔记。所以长文本有两笔账：

| | 增长方式 | 什么时候最要命 |
|---|---|---|
| 注意力计算量 | 长度的**平方** | 极长上下文 |
| KV cache 存储 | 长度的**线性** | 长对话、长文档、并发多 |

两笔账都要还，而它们的最优解法不一样——这正好引出后面的两条主线。

## 六、一个直接后果：笔记的条数是有公式的

KV cache 的大小由五件事决定：**2 × 层数 × KV 头数 × 序列长度 × 每头维度**，再乘以每个数字占的字节数。

（开头的 **2** 来自 K 和 V 是两份独立的张量——模型每个词要各存一套。）

这就是为什么它随长度线性增长、为什么减少 KV 头能省、以及为什么"压缩笔记"（下一段故事）永远有生意可做。

> 提醒一个容易混的坑：上面这个"2"来自 K 和 V 是两个张量，**不是**"16 位精度占 2 字节"的意思。两个 2 是不同的 2，而且它们会乘在一起——所以 BF16 下的 KV cache 还要再乘一个 2。

## 七、这条路还没走完

注意力让模型能精准取用前文，但它**太认真了**：每个词都要看前面所有词。

于是自然的问题是：**真的每个词都需要看所有词吗？** 大部分时候答案是不需要——远距离的绝大多数内容跟当前词毫无关系。

这就是"稀疏化"的起点，也是 DeepSeek 后面几年最重要的一条线：不是减少笔记的**宽度**（把每条笔记压窄），而是减少要看的**条数**（只挑相关的看）。第九篇会完整讲这条线从 2024 年走到 2026 年的四代方案。




## 原文出处

本系列的数字都来自下面这些报告（**已逐篇核验**）。编号后的 vN 是本系列核对时对应的版本——如果你打开的另一版数字不一样，先看版本号。

- DeepSeek-V2 技术报告：[arXiv:2405.04434v5](https://arxiv.org/abs/2405.04434)

---

**一句话总结：** 注意力让每个词用 Q、K、V 互相打分并按相关度加权取用，效果精准，代价是计算量随长度平方增长、笔记随长度线性增长。

**记住这个数字：** 64 个查询头共用 8 份笔记——DeepSeek 67B 的配比，笔记本身因此降到八分之一。

**下一篇：** [参数、数据、算力：三个"大小"千万别混](/posts/deepseek-paper-course-05-three-sizes/)——同一个数字换个单位，能让你把论文读反。
