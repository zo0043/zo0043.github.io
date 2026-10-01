---
title : 'DeepSeek 论文课（二）：AI 论文里那些看不懂的词'
date : 2026-09-30T16:25:00+08:00
categories : ["DeepSeek"]
tags : ["deepseek","ai论文","系列","术语表"]
description : "把 AI 论文里最常见、也最容易望文生义的术语，按『模型长什么样、怎么训练、怎么用、怎么衡量』四层归好类，每个都说明它真正的含义。"
---

# AI 论文里那些看不懂的词

**先给结论：** 读懂 AI 论文不需要智商，只需要知道大约三十个词在说什么。这一篇把它们全部解决掉——**它是整个系列的地基页，任何时候读不懂某个词，回到这里查。**

用法建议：不用背。先扫一遍有个印象，然后去读后面的文章，遇到卡住的地方再回来查。

## 一、模型本身：它是什么，有多大

<div style="margin:22px 0">
<svg viewBox="0 0 800 380" width="100%" style="max-width:800px;margin:0 auto" role="img" xmlns="http://www.w3.org/2000/svg"><defs><marker id="ah-d02" markerWidth="9" markerHeight="9" refX="7.2" refY="3.2" orient="auto"><path d="M0,0 L8,3.2 L0,6.4 z" fill="currentColor"/></marker></defs><text x="400.0" y="20.0" text-anchor="middle" font-size="14" font-weight="bold" fill="currentColor">术语地图：先归类，再理解</text><rect x="40" y="60" width="720" height="56" rx="10" fill="currentColor" fill-opacity="0.1" stroke="currentColor" stroke-width="1.2"/><text x="60.0" y="84.0" text-anchor="start" font-size="12.5" font-weight="bold" fill="currentColor">模型长什么样</text><rect x="258.0" y="74.0" width="84.0" height="28.0" rx="9" fill="currentColor" fill-opacity="0.13" stroke="currentColor" stroke-width="1.4"/><text x="300.0" y="92.1" text-anchor="middle" font-size="11.5" fill="currentColor">参数</text><rect x="365.5" y="74.0" width="84.0" height="28.0" rx="9" fill="currentColor" fill-opacity="0.13" stroke="currentColor" stroke-width="1.4"/><text x="407.5" y="92.1" text-anchor="middle" font-size="11.5" fill="currentColor">层</text><rect x="473.0" y="74.0" width="84.0" height="28.0" rx="9" fill="currentColor" fill-opacity="0.13" stroke="currentColor" stroke-width="1.4"/><text x="515.0" y="92.1" text-anchor="middle" font-size="11.5" fill="currentColor">注意力</text><rect x="580.5" y="74.0" width="84.0" height="28.0" rx="9" fill="currentColor" fill-opacity="0.13" stroke="currentColor" stroke-width="1.4"/><text x="622.5" y="92.1" text-anchor="middle" font-size="11.5" fill="currentColor">专家</text><rect x="688.0" y="74.0" width="84.0" height="28.0" rx="9" fill="currentColor" fill-opacity="0.13" stroke="currentColor" stroke-width="1.4"/><text x="730.0" y="92.1" text-anchor="middle" font-size="11.5" fill="currentColor">词表</text><rect x="40" y="132" width="720" height="56" rx="10" fill="currentColor" fill-opacity="0.06" stroke="currentColor" stroke-width="1.2"/><text x="60.0" y="156.0" text-anchor="start" font-size="12.5" font-weight="bold" fill="currentColor">怎么训练出来</text><rect x="258.0" y="146.0" width="84.0" height="28.0" rx="9" fill="currentColor" fill-opacity="0.13" stroke="currentColor" stroke-width="1.4"/><text x="300.0" y="164.1" text-anchor="middle" font-size="11.5" fill="currentColor">预训练</text><rect x="365.5" y="146.0" width="84.0" height="28.0" rx="9" fill="currentColor" fill-opacity="0.13" stroke="currentColor" stroke-width="1.4"/><text x="407.5" y="164.1" text-anchor="middle" font-size="11.5" fill="currentColor">微调</text><rect x="473.0" y="146.0" width="84.0" height="28.0" rx="9" fill="currentColor" fill-opacity="0.13" stroke="currentColor" stroke-width="1.4"/><text x="515.0" y="164.1" text-anchor="middle" font-size="11.5" fill="currentColor">强化学习</text><rect x="580.5" y="146.0" width="84.0" height="28.0" rx="9" fill="currentColor" fill-opacity="0.13" stroke="currentColor" stroke-width="1.4"/><text x="622.5" y="164.1" text-anchor="middle" font-size="11.5" fill="currentColor">奖励</text><rect x="688.0" y="146.0" width="84.0" height="28.0" rx="9" fill="currentColor" fill-opacity="0.13" stroke="currentColor" stroke-width="1.4"/><text x="730.0" y="164.1" text-anchor="middle" font-size="11.5" fill="currentColor">消融</text><rect x="40" y="204" width="720" height="56" rx="10" fill="currentColor" fill-opacity="0.06" stroke="currentColor" stroke-width="1.2"/><text x="60.0" y="228.0" text-anchor="start" font-size="12.5" font-weight="bold" fill="currentColor">怎么用起来</text><rect x="258.0" y="218.0" width="84.0" height="28.0" rx="9" fill="currentColor" fill-opacity="0.13" stroke="currentColor" stroke-width="1.4"/><text x="300.0" y="236.1" text-anchor="middle" font-size="11.5" fill="currentColor">词元</text><rect x="365.5" y="218.0" width="84.0" height="28.0" rx="9" fill="currentColor" fill-opacity="0.13" stroke="currentColor" stroke-width="1.4"/><text x="407.5" y="236.1" text-anchor="middle" font-size="11.5" fill="currentColor">上下文</text><rect x="473.0" y="218.0" width="84.0" height="28.0" rx="9" fill="currentColor" fill-opacity="0.13" stroke="currentColor" stroke-width="1.4"/><text x="515.0" y="236.1" text-anchor="middle" font-size="11.5" fill="currentColor">KV cache</text><rect x="580.5" y="218.0" width="84.0" height="28.0" rx="9" fill="currentColor" fill-opacity="0.13" stroke="currentColor" stroke-width="1.4"/><text x="622.5" y="236.1" text-anchor="middle" font-size="11.5" fill="currentColor">推理</text><rect x="688.0" y="218.0" width="84.0" height="28.0" rx="9" fill="currentColor" fill-opacity="0.13" stroke="currentColor" stroke-width="1.4"/><text x="730.0" y="236.1" text-anchor="middle" font-size="11.5" fill="currentColor">延迟</text><rect x="40" y="276" width="720" height="56" rx="10" fill="currentColor" fill-opacity="0.03" stroke="currentColor" stroke-width="1.2"/><text x="60.0" y="300.0" text-anchor="start" font-size="12.5" font-weight="bold" fill="currentColor">怎么衡量好坏</text><rect x="258.0" y="290.0" width="84.0" height="28.0" rx="9" fill="currentColor" fill-opacity="0.13" stroke="currentColor" stroke-width="1.4"/><text x="300.0" y="308.1" text-anchor="middle" font-size="11.5" fill="currentColor">基准</text><rect x="365.5" y="290.0" width="84.0" height="28.0" rx="9" fill="currentColor" fill-opacity="0.13" stroke="currentColor" stroke-width="1.4"/><text x="407.5" y="308.1" text-anchor="middle" font-size="11.5" fill="currentColor">准确率</text><rect x="473.0" y="290.0" width="84.0" height="28.0" rx="9" fill="currentColor" fill-opacity="0.13" stroke="currentColor" stroke-width="1.4"/><text x="515.0" y="308.1" text-anchor="middle" font-size="11.5" fill="currentColor">口径</text><rect x="580.5" y="290.0" width="84.0" height="28.0" rx="9" fill="currentColor" fill-opacity="0.13" stroke="currentColor" stroke-width="1.4"/><text x="622.5" y="308.1" text-anchor="middle" font-size="11.5" fill="currentColor">基线</text><rect x="688.0" y="290.0" width="84.0" height="28.0" rx="9" fill="currentColor" fill-opacity="0.13" stroke="currentColor" stroke-width="1.4"/><text x="730.0" y="308.1" text-anchor="middle" font-size="11.5" fill="currentColor">消融缺失</text><rect x="40.0" y="348.0" width="720.0" height="28.0" rx="10" fill="currentColor" fill-opacity="0.025" stroke="currentColor" stroke-width="1" stroke-dasharray="3 4" opacity="0.9"/><text x="400.0" y="370.0" text-anchor="middle" font-size="12" opacity="0.85" fill="currentColor">同一个中文词可能在两层里意思不同——「推理」既是「模型运行」也是「解题能力」</text></svg>
<p style="text-align:center;font-size:13px;opacity:.7;margin:8px 0 0">把术语按「结构 / 训练 / 运行 / 评测」四层归类，位置感比死记更重要</p>
</div>


**大模型 / LLM**：用海量文字训练出来的程序，核心功能是"根据前面的内容，预测下一个词"。它的"知识"来自训练，而不是查数据库。

**参数（权重）**：模型内部的可调数字，相当于它的"脑细胞连接强度"。训练就是不断调整这些数字。
> 7B 表示 70 亿个参数，671B 表示 6710 亿个。这是**数量**，不是能力评分。

**总参数 vs 激活参数**：有些模型每次只动用一部分参数（见"混合专家"）。总参数是全部家当，激活参数是每次实际动用的那一小部分。
> 看到"1.6T 总参数、49B 激活"，意思是家当很大，但每次只用其中约 3%。

**Transformer**：2017 年提出、目前几乎所有大模型都在用的基础架构（这是全行业常识，不是本系列核验的结论）。它由许多层叠起来，每层主要做两件事：注意力 + 前馈网络。

**Decoder（解码器）**：只往后看、一个字一个字往外写的结构。现在的主流大模型基本都是解码器。

**嵌入（embedding）**：把文字变成一串数字的过程或结果。模型不认识字，只认识数字。
> 可以想象成给每个词一个坐标，"猫"和"狗"的坐标距离比较近。

**注意力（attention）**：模型处理一个词时，回头看看前面哪些词跟自己有关，并按相关程度加权。
> 比喻：查字典时，每个字都互相瞟一眼，看谁该影响谁。

**Q / K / V**：注意力里的三个角色。Q 是"我在找什么"，K 是"我能提供什么"，V 是"我实际的内容"。模型用 Q 和 K 算相关度，再按相关度取出 V。

**KV cache（键值缓存）**：模型读长文章时，把算过的内容存下来避免重算，就是它的"读书笔记"。
> 这份笔记会随文章变长线性增长，是长文本处理最贵的地方——系列第九篇专讲这个。

**MLA**：DeepSeek 提出的一种省笔记的技术，把笔记压缩后再存。2026 年的新旗舰（从 DeepSeek-V4 起）已经不用它了，改成了"在序列方向上合并条目"的路子。

**FFN（前馈网络）**：注意力之后的那一层，做逐字的"思考"。混合专家技术主要替换的就是它。

**归一化 / RMSNorm**：训练时防止数字变得过大或过小的调节装置，类似音量自动平衡。

**位置编码 / RoPE**：让模型知道谁在前谁在后的机制，通过"旋转"数字来实现。

## 二、训练：模型是怎么学会的

**预训练**：让模型读海量文字、学习"下一个词是什么"的第一阶段。最烧钱，也决定了底子。

**上下文窗口**：模型一次能"看到"多少内容。128K 表示约 13 万个词元（这里 1K 按 1024 算）。
> 注意两件事：**窗口大不等于能有效利用**；而且这个 128K 常常是从更短的长度**拉伸**出来的（YaRN 一类技术），不是一上来就按这么长训练的。

**微调 / SFT**：预训练之后，用"问答范例"教模型按人类想要的方式回答。是从"会接话"变成"会听指令"。

**强化学习（RL）**：不直接告诉模型正确答案，而是给它的输出打分，让它自己往高分方向调整。

**奖励（reward）**：给模型输出的分数。可以来自规则（答案对不对），也可以来自另一个模型。

**GRPO**：DeepSeek 用过的一种强化学习方法。特点是**不需要单独训练一个"预估分数"的助理网络**（其他方法需要一个），而是让模型对同一个问题写一组答案，先用这组的平均水平当基准（比平均好就奖励、差就惩罚），**再按这组答案的离散程度归一化**——除以标准差会实际改变更新步长，不只是缩了个数值。
> 比喻：不请助教估分，让同一个学生把同一道题做几十遍（DeepSeekMath 实际用的是每组 **64** 条），比一比哪几遍比自己平均水平好，就往那个方向练。
> 还有一条边界要说清：GRPO **不是**把约束全扔了。它仍然拉着一条橡皮筋，不许模型跑得离训练前的旧版本太远（这条约束叫 KL，其他方法通常把它放在奖励里，GRPO 把它挪进了损失函数）。所以它省掉的是那个同规模的助理网络，**没省**奖励信号，也没省这条约束。

**思维链（CoT）**：让模型把推理步骤写出来，而不是直接给答案。写步骤往往能提高正确率。

**蒸馏**：把一个大模型的能力"教"给一个小模型，让小模型用更少的资源做类似的事。

**灾难性遗忘**：模型换着学新东西时，容易把旧本事忘了。这是**换数据继续训练**时要小心的问题——比如从代码模型转去学数学，就可能把代码能力忘掉，办法是把旧数据混回来一起训。

## 三、效率：怎么省钱（本系列最多的内容）

**稠密模型**：每个字都要经过全部参数计算。稳但贵。

**混合专家（MoE）**：把一个大网络拆成很多"专家"，每次只叫几个相关的来干活。
> 比喻：大厨一个人做所有菜很慢，改成专家团分诊，每道菜只叫对应的人。

**路由（routing）**：决定"这道题该叫哪几位专家"的机制。

**负载均衡**：防止所有任务都挤给同几位专家。有的专家累死，有的闲着，整体效率就下降。

**稀疏 / 稀疏注意力**：只处理一部分，而不是全部。
> 长文章里，"每个字都看全部字"代价是长度的平方——稀疏化的目标是打破这个平方。

**量化**：用更少的位数存数字。常见精度从高到低：FP32（32 位）、FP16/BF16（16 位）、FP8（8 位）、FP4（4 位）——位数越少越省内存和带宽，但数字越粗糙。
> 比喻：本来用精确到分的账本，改成精确到元——省空间，但对算法的要求更高。DeepSeek 从用 FP8 训练，到 2026 年用 FP4 处理专家参数。

**FLOPs**：计算量的单位，可以理解成"要做多少次基本运算"。

**吞吐 / 延迟**：吞吐是单位时间能处理多少，延迟是单个请求要等多久。两者常常此消彼长。

**投机解码**：先用便宜的小模型猜几个词，再让大模型快速核对，猜对了就省时间。

**MTP（多词元预测）**：一次预测多个后续词，而不是只预测下一个，用于加速训练和推理。

## 四、评测与判断：数字怎么看

**基准测试（benchmark）**：一套标准化考题，比如数学题库、代码题、阅读理解。

**消融实验（ablation）**：一次只去掉一个设计，看效果掉多少，用来证明"这个设计确实有用"。**没有消融实验的结论要打问号。**

**困惑度（PPL）**：衡量模型对文字的"意外程度"。越低说明越能预料下一个词。
> 但不同**分词方式**的困惑度不能直接比较——同一句话，不同模型切出的词元个数不同，算出来的"平均意外程度"自然不可比。论文里更严谨的做法是换成**每字节多少比特（BPB）**，分母换成"字节"这个所有模型都一样的单位。

**幻觉**：模型一本正经地编造不存在的事实。根因是它在"预测像样的下一个词"，而不是在"查证事实"。

**多模态**：能处理文字之外的内容（图片、文档截图）。让模型"看图说话"。（本系列只涉及 DeepSeek 的图像/文档线，它的多模态工作里**没有音频**。）

**Agent / 智能体**：不只是回答问题，而是能调用工具、多步执行任务的 AI 系统。

**Harness**：包在模型外面的工程框架，负责工具调用、权限、文件操作等。模型负责生成文字，Harness 负责把事做成。

## 五、两个中文里同名、必须分清的词

**推理（reasoning）**：模型"思考"的过程——推导、分析、一步步解题。比如"这道数学题该怎么算"。

**推理（inference）**：模型"运行"的过程——把输入算成输出。比如"部署一次推理要多少显存"。

> 中文都叫"推理"，英文却是两个完全不同的词。**读到"推理成本"指的是第二个，"推理能力"通常指第一个。** 搞混会完全读错论文。




## 原文出处

本系列的数字都来自下面这些报告（**已逐篇核验**）。编号后的 vN 是本系列核对时对应的版本——如果你打开的另一版数字不一样，先看版本号。

- DeepSeek-V2 技术报告：[arXiv:2405.04434v5](https://arxiv.org/abs/2405.04434)
- DeepSeek-V3 技术报告：[arXiv:2412.19437v2](https://arxiv.org/abs/2412.19437)
- DeepSeek-R1 技术报告：[arXiv:2501.12948v2](https://arxiv.org/abs/2501.12948)
- DeepSeek-V4 技术报告：[arXiv:2606.19348v1](https://arxiv.org/abs/2606.19348)
- Engram 条件记忆论文：[arXiv:2601.07372v2](https://arxiv.org/abs/2601.07372)
- DSpark 论文：[arXiv:2607.05147v1](https://arxiv.org/abs/2607.05147)
- Fire-Flyer AI-HPC 论文：[arXiv:2408.14158v2](https://arxiv.org/abs/2408.14158)

**关于版本：** DeepSeek-R1 的 v1 与 v2 之间存在数字修订，本系列统一以 **v2** 为准。

---

**一句话总结：** 上面这些词覆盖了本系列大部分的阅读障碍，剩下的可以在正文里现场解释。

**记住这个方法：** 遇到不懂的词，先问"它是模型的一部分，还是训练的方法，还是一个衡量的数字"——归类之后，理解会快很多。

**下一篇：** [大模型到底在干什么](/posts/deepseek-paper-course-03-how-llm-works/)——从头建一个最小心智模型。
