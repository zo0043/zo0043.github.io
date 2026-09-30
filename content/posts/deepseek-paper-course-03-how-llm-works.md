---
title : 'DeepSeek 论文课（三）：大模型到底在干什么'
date : 2026-09-30T16:30:00+08:00
categories : ["DeepSeek"]
tags : ["deepseek","ai论文","系列","基础"]
---

# 大模型到底在干什么

**先给结论：** 大模型做的事情只有一件——**根据前面的内容，猜下一个词**。所有看起来神奇的"能力"，都是从这一件事里长出来的。

理解了这一点，后面所有论文都会变得好读得多。

## 一、它的唯一任务：猜下一个词

<div style="margin:22px 0">
<svg viewBox="0 0 700 300" width="100%" style="max-width:700px;margin:0 auto" role="img" xmlns="http://www.w3.org/2000/svg"><defs><marker id="ah-d03" markerWidth="9" markerHeight="9" refX="7.2" refY="3.2" orient="auto"><path d="M0,0 L8,3.2 L0,6.4 z" fill="currentColor"/></marker></defs><rect x="15.0" y="52.0" width="150.0" height="56.0" rx="9" fill="currentColor" fill-opacity="0.06" stroke="currentColor" stroke-width="1.4"/><text x="90.0" y="77.0" text-anchor="middle" font-size="13" fill="currentColor">读入已有内容</text><text x="90.0" y="94.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">前文 + 已写出的字</text><rect x="188.3" y="52.0" width="150.0" height="56.0" rx="9" fill="currentColor" fill-opacity="0.06" stroke="currentColor" stroke-width="1.4"/><text x="263.3" y="77.0" text-anchor="middle" font-size="13" fill="currentColor">算出「可能性表」</text><text x="263.3" y="94.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">每个候选词一个分数</text><rect x="361.7" y="52.0" width="150.0" height="56.0" rx="9" fill="currentColor" fill-opacity="0.06" stroke="currentColor" stroke-width="1.4"/><text x="436.7" y="77.0" text-anchor="middle" font-size="13" fill="currentColor">挑一个词</text><text x="436.7" y="94.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">通常挑高的，加一点随机</text><rect x="535.0" y="52.0" width="150.0" height="56.0" rx="9" fill="currentColor" fill-opacity="0.06" stroke="currentColor" stroke-width="1.4"/><text x="610.0" y="77.0" text-anchor="middle" font-size="13" fill="currentColor">接到末尾</text><text x="610.0" y="94.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">句子变长一个字</text><line x1="166.0" y1="80.0" x2="187.3" y2="80.0" stroke="currentColor" stroke-width="1.4" marker-end="url(#ah-d03)"/><line x1="339.3" y1="80.0" x2="360.7" y2="80.0" stroke="currentColor" stroke-width="1.4" marker-end="url(#ah-d03)"/><line x1="512.7" y1="80.0" x2="534.0" y2="80.0" stroke="currentColor" stroke-width="1.4" marker-end="url(#ah-d03)"/><polyline points="610.0,108.0 670.0,108.0 670.0,108.0 90.0,108.0" fill="none" stroke="currentColor" stroke-width="1.4" stroke-dasharray="6 4" marker-end="url(#ah-d03)"/><text x="670.0" y="112.0" text-anchor="end" font-size="11" opacity="0.78" fill="currentColor">重复 · 直到写完</text><rect x="40.0" y="170.0" width="620.0" height="100.0" rx="10" fill="currentColor" fill-opacity="0.025" stroke="currentColor" stroke-width="1" stroke-dasharray="3 4" opacity="0.9"/><text x="50.0" y="186.0" text-anchor="start" font-size="11" opacity="0.72" fill="currentColor">训练 vs 使用：两件不同的事</text><rect x="80.0" y="206.0" width="200.0" height="48.0" rx="9" fill="currentColor" fill-opacity="0.13" stroke="currentColor" stroke-width="1.4"/><text x="180.0" y="227.0" text-anchor="middle" font-size="13" fill="currentColor">训练</text><text x="180.0" y="244.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">猜错就改内部数字 · 极贵</text><rect x="365.0" y="206.0" width="230.0" height="48.0" rx="9" fill="currentColor" fill-opacity="0.03" stroke="currentColor" stroke-width="1.4" stroke-dasharray="5 4"/><text x="480.0" y="227.0" text-anchor="middle" font-size="13" fill="currentColor">使用</text><text x="480.0" y="244.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">数字不再变 · 但要存「读书笔记」</text></svg>
<p style="text-align:center;font-size:13px;opacity:.7;margin:8px 0 0">「猜下一个词」是一个循环；训练改数字，使用只存笔记</p>
</div>



给它一句话的开头：

> 今天天气很好，我想去公园……

它会输出一个"可能性表"，大致是这样：

| 下一个词 | 可能性 |
|---|---|
| 散步 | 高 |
| 玩 | 较高 |
| 睡觉 | 低 |
| 蓝色 | 极低 |

然后它挑一个（通常挑概率高的，但会加一点随机性，不然每次回答都一样），把选中的词接到句子后面，再重复这个过程。**一个字一个字往外写，每一步都在猜下一个。**

> **你可以自己试试：** 拿一篇文章，随便遮住一个词，猜猜原文是什么。如果你猜得挺准，说明你也做了一次"语言模型"的动作。差别只在于：模型读过的东西比你多几亿倍。

**词元（token）** 就是它处理的最小单位。它**既不是"字"也不是"单词"**，而是由词表切出来的：一个英文单词可能被切成好几段，两个常用汉字也可能合并成一个词元。**切法取决于模型的词表**，所以"2T 词元"不能直接换算成"多少本书"或"多少字"——想知道换算关系，得先看词表。

## 二、"猜下一个词"为什么会变聪明

关键在这里：**要想猜得准，你必须先理解。**

举例，下面这句话：

> 小明把球扔给小红，然后 ___ 又把它扔了回去。

要填对空，模型必须知道"它"指的是球、要分清小明和小红、还要理解"扔回去"的主语会切换。**语法、指代、常识、甚至简单的推理，全都是为了猜准而不得不学会的东西。**

这就是为什么"预测下一个词"这么简单的目标，能训练出会写代码、会解数学题的模型——**不是因为它被专门教过这些，而是因为这些能力有助于它把下一个词猜得更准。**

## 三、训练和"使用"是两件不同的事

这两个阶段常被混淆，但论文里的数字有时指前者、有时指后者，分不清就会读错。

**训练（学习阶段）**：给它看海量文本，让它猜，猜错了就调整内部的那几十亿个数字。这个过程极其昂贵——需要成千上万张显卡跑几个月。DeepSeek 的第一篇论文，用的数据量是 **2 万亿个词元**。

**推理（使用阶段）**：就是你平时用它的过程。模型内部数字**不再变化**——它只是根据输入算出输出。

| | 训练 | 使用（推理） |
|---|---|---|
| 内部参数 | 不断调整 | 固定不变 |
| 需要保存 | 大量中间数据（为了纠正） | 只需模型本身 + 读书笔记 |
| 成本 | 极高，一次性的 | 较低，但每次都要花 |

**"读书笔记"正式叫 KV cache**——模型读长文章时把算过的内容存下来，避免重复计算。这份笔记的大小随文章变长而线性增长，是长文本处理最贵的部分（系列第九篇专讲）。

## 四、为什么它会"一本正经地胡说"

知道它只是"猜下一个词"，幻觉就好解释了。

模型输出的是**"看起来最像样的下一个词"**，不是**"查证过的事实"**。当它没见过某个知识点时，"像样"和"正确"就会分家——于是它编出一个格式完美、语气自信、但完全错误的内容。

所以正规用法里有两道保险：

1. **联网检索**：把真实资料放进它的输入里，让它"照着材料猜"，而不是凭记忆猜。
2. **工具验证**：让它写代码跑一遍、查一次数据库，用真实结果校正。

这也是本系列反复强调"证据"的原因：**模型说得流畅，和它说得对，是两件独立的事。**

## 五、还有一个坑：变的到底是什么

论文里比较两个模型时，有三个完全不同的"变化"会被混在一起谈：

- **知识和技能变了**：真的学会了新东西
- **回答格式变了**：还是那些知识，但更会按要求输出
- **考试条件变了**：题目、提示词、评分方式不一样了

比如有的模型经过选择题训练后，选择题成绩大涨，但自由问答题没进步——**那就是格式变化，不是能力变化。** 分不清这三者，就会把"训练技巧"误读成"模型变强"。

## 六、别把三个"大小"搞混

刚接触时最容易犯的错，是把下面三个词当成一件事：

- 参数量（模型有多少可调数字）——比如 7B、67B
- 数据量（训练时读了多少词元）——比如 2T
- 计算量（总共做了多少次运算）——用 FLOPs 衡量

**它们单位不同、含义不同，而且互相制约：** 算力预算固定时，模型做大一倍，能读的数据就得减半。这个"怎么分配最划算"的问题，正是 DeepSeek 第一篇论文要研究的——也是下一篇的主题。

---

**一句话总结：** 大模型就是个"猜下一个词"的机器，为了猜得准，它不得不学会理解；而幻觉是因为它优化的是"像样"，不是"正确"。

**记住这个数字：** 2 万亿——DeepSeek 第一篇论文用的训练数据规模（词元）。

**下一篇：** [注意力：模型怎么"看"上下文](/posts/deepseek-paper-course-04-attention/)——猜下一个词时，它是怎么知道该看哪里的。
