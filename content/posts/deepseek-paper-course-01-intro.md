---
title : 'DeepSeek 论文课（一）：为什么值得花时间读 DeepSeek 的论文'
date : 2026-09-30T16:20:00+08:00
categories : ["DeepSeek"]
tags : ["deepseek","ai论文","系列","学习方法"]
---

# 为什么值得花时间读 DeepSeek 的论文

**先给结论：** 因为 DeepSeek 的论文是公开资料里，少数能把"一个好主意"和"它到底花了多少钱"同时讲清楚的材料。读它们，你不只是知道 AI 在变强，还能开始看懂**它凭什么强、代价是什么**。

## 一、这套论文讲了两年半的连续故事

<div style="margin:22px 0">
<svg viewBox="0 0 780 300" width="100%" style="max-width:780px;margin:0 auto" role="img" xmlns="http://www.w3.org/2000/svg"><defs><marker id="ah-d01" markerWidth="9" markerHeight="9" refX="7.2" refY="3.2" orient="auto"><path d="M0,0 L8,3.2 L0,6.4 z" fill="currentColor"/></marker></defs><text x="390.0" y="20.0" text-anchor="middle" font-size="14" font-weight="bold" fill="currentColor">四段演化与三条主线</text><rect x="32.5" y="44.0" width="155.0" height="52.0" rx="9" fill="currentColor" fill-opacity="0.06" stroke="currentColor" stroke-width="1.4"/><text x="110.0" y="67.0" text-anchor="middle" font-size="13" fill="currentColor">2024 上半</text><text x="110.0" y="84.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">先算账：钱该花在哪</text><rect x="215.8" y="44.0" width="155.0" height="52.0" rx="9" fill="currentColor" fill-opacity="0.06" stroke="currentColor" stroke-width="1.4"/><text x="293.3" y="67.0" text-anchor="middle" font-size="13" fill="currentColor">2024 下半</text><text x="293.3" y="84.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">拼便宜零件</text><rect x="399.2" y="44.0" width="155.0" height="52.0" rx="9" fill="currentColor" fill-opacity="0.06" stroke="currentColor" stroke-width="1.4"/><text x="476.7" y="67.0" text-anchor="middle" font-size="13" fill="currentColor">2025</text><text x="476.7" y="84.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">让模型学会「想」</text><rect x="582.5" y="44.0" width="155.0" height="52.0" rx="9" fill="currentColor" fill-opacity="0.06" stroke="currentColor" stroke-width="1.4"/><text x="660.0" y="67.0" text-anchor="middle" font-size="13" fill="currentColor">2026</text><text x="660.0" y="84.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">效率极限 + 开新方向</text><line x1="188.0" y1="70.0" x2="215.3" y2="70.0" stroke="currentColor" stroke-width="1.4" marker-end="url(#ah-d01)"/><line x1="371.3" y1="70.0" x2="398.7" y2="70.0" stroke="currentColor" stroke-width="1.4" marker-end="url(#ah-d01)"/><line x1="554.7" y1="70.0" x2="582.0" y2="70.0" stroke="currentColor" stroke-width="1.4" marker-end="url(#ah-d01)"/><rect x="40" y="134" width="700" height="32" rx="6" fill="currentColor" fill-opacity="0.1" stroke="currentColor" stroke-width="1" /><text x="58.0" y="154.0" text-anchor="start" font-size="12" fill="currentColor">省：用更少计算做同样的事</text><rect x="40" y="180" width="700" height="32" rx="6" fill="currentColor" fill-opacity="0.06" stroke="currentColor" stroke-width="1" /><text x="58.0" y="200.0" text-anchor="start" font-size="12" fill="currentColor">判断对错：从对错 → 自己当裁判</text><rect x="40" y="226" width="700" height="32" rx="6" fill="currentColor" fill-opacity="0.06" stroke="currentColor" stroke-width="1" /><text x="58.0" y="246.0" text-anchor="start" font-size="12" fill="currentColor">读数字：同一件事不同口径能差几倍</text><text x="390.0" y="288.0" text-anchor="middle" font-size="12" opacity="0.7" fill="currentColor">三条线在最新一代模型上汇合</text></svg>
<p style="text-align:center;font-size:13px;opacity:.7;margin:8px 0 0">四段演化：同一时期并行推进的三条主线</p>
</div>



从 2024 年 1 月到 2026 年 9 月，本文系列收录并逐篇核验了 DeepSeek 的 31 篇论文报告（范围说明见第五节）。它们不是零散的成果，而是一条能连起来的线：

1. **2024 年上半年：先搞清楚钱该花在哪。** 与其急着造大模型，他们先做小规模实验，研究"参数、数据、算力怎么分配最划算"。
2. **2024 下半年到年底：把便宜的零件拼起来。** 用更省显存的注意力机制、更省的专家结构，堆出一个很强但成本可控的模型。
3. **2025 年：让模型学会"想"。** 重点转向推理能力——怎么用奖励机制让模型自己学会一步步解题。
4. **2026 年：把效率推到极限，同时开辟新方向。** 压缩、低精度计算、记忆结构、多模态，几乎同时换代。

**这条线上最显眼的一条主线是"省"**——不是省钱本身，而是"用更少的计算，做到一样甚至更好"。但它**不是唯一的主线**：另外还有两条同样贯穿全程的线，一条是**"怎么判断对错"**（从简单对错判定，一路升维到能自己当裁判），另一条是**"怎么读数字"**（同一件事在不同口径下能差出几倍）。三条线会在最新一代模型上汇合。

## 二、为什么"省"比"更强"更值得学

大多数新闻只告诉你模型考了多少分。分数会过时，**方法不会**。

举个具体的例子。"KV cache"（可以理解成模型读长文章时做的读书笔记）占用的内存，会随着文章变长而暴涨。DeepSeek 的一系列论文就在解决这个问题：2024 年他们的方案是每一层、每一个字都要存 576 个数字；到 2026 年最新报告里，全模型只有 4 层需要存主笔记，平均到每个字只要约 2.5 条。

> **注意这个数字的前提：** "2.5 条"是相对 DeepSeek 自己的最新模型而言，不是所有模型通用。换个模型、换个比较对象，数字就不同。**这是本系列会反复强调的一件事：论文里任何一个数字，都必须问"跟谁比、在什么条件下"。**

看懂这一串演化，你就理解了 AI 里最核心的一种思维方式：**把一个昂贵的东西，拆开，找到真正贵的环节，然后只在那里省钱。**

## 三、这套论文有一个罕见的优点：敢说自己不够好

2026 年的旗舰报告里，作者明确写了一段话，大意是：**我们比最强的对手落后 3 到 6 个月。**

很少有公司会在自己的论文里这么写。同样的报告里，测试表格显示新模型在某些项目上**输给**了自己上一代；官方新闻稿的措辞比论文正文更激进——论文说"相当，部分任务更好"，新闻稿说"领先"。

对读者来说这是好事：**当一份材料愿意暴露自己的弱点时，它的其他数字也更值得信任。** 本系列会专门用一篇讲"怎么读论文里的数字"，把这种判断方法教给你。

## 四、需要什么基础？其实很少

网上讲 AI 的文章常常两条路：要么全是名词听不懂，要么全是比喻学不到东西。

这个系列走中间路线：**不用线性代数、不用微积分、不用编程**。我们会用"查字典""分诊台""读书折角"这样的比喻讲清每个机制，但比喻之后一定跟上真实的数字和条件——因为只有数字能让你分辨，哪个说法是真的。

需要的前置知识，本系列会用三篇讲完：

- 大模型到底在干什么（第三篇）
- 注意力是怎么工作的（第四篇）
- 参数、数据、算力这三个"大小"为什么不能混（第五篇）

之后是主线故事，再之后是可以挑着读的支线。

## 五、一个提醒：这不是"读完全部论文"

要诚实说明：这个系列覆盖的是**核验过的 31 篇主目录论文**。DeepSeek 还有一些论文藏在没有署名的合作研究里，检索本身就有难度——比如有一篇讲智能体训练沙箱的论文，作者栏写着 DeepSeek，但官方代码仓库里找不到对应入口，所以目前无法完全确认。

所以本系列不会说"DeepSeek 的全部论文都在这里"。它说的是：**这 31 篇是能被交叉验证、能查到来源、能追到官方入口的部分。** 学知识的第一条纪律，是知道自己知道的范围到哪儿。

---

**一句话总结：** DeepSeek 的论文是一条"如何用更少算力做到更强"的连续故事，而且作者愿意承认自己的不足——这两点让它比单纯的成绩通报更值得读。

**记住这个数字：** 31 篇，2024 年 1 月到 2026 年 9 月。

**下一篇：** [AI 论文里那些看不懂的词（术语表）](/posts/deepseek-paper-course-02-glossary/)——先把拦路的词全部解决掉。
