---
title : 'DeepSeek 论文课（十一）：AI 怎么"看"图：从认图到读文档'
date : 2026-09-30T16:46:00+08:00
categories : ["DeepSeek"]
tags : ["deepseek","ai论文","系列","多模态"]
---

# AI 怎么"看"图：从认图到读文档

**先给结论：** 多模态就是让同一个模型同时处理文字和图片，而这条线十几年的工程史其实只在回答两个问题——**一张图要花掉多少个"词元"**，以及**这些词元该按什么顺序读**。2025 年最反直觉的一步是：把文字先渲染成图片再送进模型，真的能省下大量词元；但省下来的每一分，都标着精度和顺序的价钱。

（**词元**就是 token，第三篇讲过——模型处理的最小单位。视觉词元就是"由图片产生的词元"，它和文字词元在模型眼里是同一种东西，都占上下文的位置。）

## 一、多模态不是"又造了一个模型"，而是让同一个模型多认一门"输入语言"

最早的做法叫"外接"：拿一个已经会读书的文本模型，旁边挂一个只会看图的视觉模型，中间加一层翻译，把图的信息转成文本模型能读的词元。DeepSeek-VL（2024 年 3 月）走的就是这条路——语言部分直接沿用自家的 DeepSeek LLM，视觉部分是另外两个编码器。

这条路内部有个长期矛盾：**"看懂图"和"画出图"对图片的表示要求是相反的。** 理解要连续的、像"语感"一样的向量；生成要离散的、像"色号"一样的编号。Janus 系列的答案是同一个模型里放两套视觉编码、两个输出头，只在文本侧共享。这些名字不用记，但这个矛盾解释了为什么多模态架构换了一代又一代。

## 二、图片进门的第一步是切块，576 个视觉词元就能装下 1024×1024 的画面

<div style="margin:22px 0">
<svg viewBox="0 0 800 360" width="100%" style="max-width:800px;margin:0 auto" role="img" xmlns="http://www.w3.org/2000/svg"><defs><marker id="ah-d11" markerWidth="9" markerHeight="9" refX="7.2" refY="3.2" orient="auto"><path d="M0,0 L8,3.2 L0,6.4 z" fill="currentColor"/></marker></defs><text x="400.0" y="20.0" text-anchor="middle" font-size="14" font-weight="bold" fill="currentColor">两条路：看图 与 读文档</text><rect x="40.0" y="34.0" width="720.0" height="140.0" rx="10" fill="currentColor" fill-opacity="0.025" stroke="currentColor" stroke-width="1" stroke-dasharray="3 4" opacity="0.9"/><text x="50.0" y="50.0" text-anchor="start" font-size="11" opacity="0.72" fill="currentColor">路一：把图变成「视觉词元」</text><rect x="25.0" y="75.0" width="130.0" height="50.0" rx="9" fill="currentColor" fill-opacity="0.06" stroke="currentColor" stroke-width="1.4"/><text x="90.0" y="104.7" text-anchor="middle" font-size="13" fill="currentColor">图像</text><rect x="167.5" y="75.0" width="130.0" height="50.0" rx="9" fill="currentColor" fill-opacity="0.06" stroke="currentColor" stroke-width="1.4"/><text x="232.5" y="97.0" text-anchor="middle" font-size="13" fill="currentColor">切成小块</text><text x="232.5" y="114.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">patch</text><rect x="310.0" y="75.0" width="130.0" height="50.0" rx="9" fill="currentColor" fill-opacity="0.06" stroke="currentColor" stroke-width="1.4"/><text x="375.0" y="97.0" text-anchor="middle" font-size="13" fill="currentColor">视觉编码器</text><text x="375.0" y="114.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">各自算特征</text><rect x="452.5" y="75.0" width="130.0" height="50.0" rx="9" fill="currentColor" fill-opacity="0.13" stroke="currentColor" stroke-width="1.4"/><text x="517.5" y="97.0" text-anchor="middle" font-size="13" fill="currentColor">576 视觉词元</text><text x="517.5" y="114.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">装下整张图</text><rect x="595.0" y="75.0" width="130.0" height="50.0" rx="9" fill="currentColor" fill-opacity="0.06" stroke="currentColor" stroke-width="1.4"/><text x="660.0" y="97.0" text-anchor="middle" font-size="13" fill="currentColor">语言模型</text><text x="660.0" y="114.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">和文字一起读</text><line x1="156.0" y1="100.0" x2="166.5" y2="100.0" stroke="currentColor" stroke-width="1.4" marker-end="url(#ah-d11)"/><line x1="298.5" y1="100.0" x2="309.0" y2="100.0" stroke="currentColor" stroke-width="1.4" marker-end="url(#ah-d11)"/><line x1="441.0" y1="100.0" x2="451.5" y2="100.0" stroke="currentColor" stroke-width="1.4" marker-end="url(#ah-d11)"/><line x1="583.5" y1="100.0" x2="594.0" y2="100.0" stroke="currentColor" stroke-width="1.4" marker-end="url(#ah-d11)"/><rect x="40.0" y="186.0" width="720.0" height="160.0" rx="10" fill="currentColor" fill-opacity="0.025" stroke="currentColor" stroke-width="1" stroke-dasharray="3 4" opacity="0.9"/><text x="50.0" y="202.0" text-anchor="start" font-size="11" opacity="0.72" fill="currentColor">路二：把文字印成图片，反而更省</text><rect x="55.0" y="224.0" width="150.0" height="52.0" rx="9" fill="currentColor" fill-opacity="0.13" stroke="currentColor" stroke-width="1.4"/><text x="130.0" y="247.0" text-anchor="middle" font-size="13" fill="currentColor">文档文字</text><text x="130.0" y="264.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">先变成文本词元</text><rect x="218.3" y="224.0" width="150.0" height="52.0" rx="9" fill="currentColor" fill-opacity="0.13" stroke="currentColor" stroke-width="1.4"/><text x="293.3" y="247.0" text-anchor="middle" font-size="13" fill="currentColor">渲染成图片</text><text x="293.3" y="264.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">排好版的一页图</text><rect x="381.7" y="224.0" width="150.0" height="52.0" rx="9" fill="currentColor" fill-opacity="0.1" stroke="currentColor" stroke-width="1.4" stroke-dasharray="5 4"/><text x="456.7" y="247.0" text-anchor="middle" font-size="13" fill="currentColor">视觉词元</text><text x="456.7" y="264.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">比文本词元少得多</text><rect x="545.0" y="224.0" width="150.0" height="52.0" rx="9" fill="currentColor" fill-opacity="0.13" stroke="currentColor" stroke-width="1.4"/><text x="620.0" y="247.0" text-anchor="middle" font-size="13" fill="currentColor">语言模型</text><text x="620.0" y="264.0" text-anchor="middle" font-size="11" opacity="0.72" fill="currentColor">读回来</text><line x1="206.0" y1="250.0" x2="217.3" y2="250.0" stroke="currentColor" stroke-width="1.4" marker-end="url(#ah-d11)"/><line x1="369.3" y1="250.0" x2="380.7" y2="250.0" stroke="currentColor" stroke-width="1.4" marker-end="url(#ah-d11)"/><line x1="532.7" y1="250.0" x2="544.0" y2="250.0" stroke="currentColor" stroke-width="1.4" marker-end="url(#ah-d11)"/><text x="400.0" y="300.0" text-anchor="middle" font-size="12" opacity="0.82" fill="currentColor">压缩比的定义：原文文本词元数 ÷ 实际视觉词元数 —— 字符级像不像，不等于意思对不对</text><text x="400.0" y="330.0" text-anchor="middle" font-size="12" opacity="0.82" fill="currentColor">第三代把问题从「多少个词元够」换成「这些词元该按什么顺序读」</text></svg>
<p style="text-align:center;font-size:13px;opacity:.7;margin:8px 0 0">多模态两条路：把图编码成词元，或把文字渲染成图再压缩</p>
</div>



模型不认识像素，只认识一串串数字。所以视觉部分第一件事是把图**切成小方块**（patch，直译"小块"），每块转成一串数字。一张高 H、宽 W 的图，方块边长取 p，块数就是（宽÷p）乘（高÷p）。

DeepSeek-VL 的关键数字是 **576**，它是一个"预算"：它的混合编码器让两路并行工作，低分辨率那路用 SigLIP-L，输入 384×384、方块边长 16，得到 24×24 的网格；高分辨率那路用 SAM-B，输入 1024×1024，先得到 64×64×256 的特征，插值放大到 96×96，再经过两层"每次把边长减半"的卷积，也得到 24×24。两路的网格必须对齐，才能逐块沿通道拼起来，最后是 576×2048 维——也就是 **576 个视觉词元**（576 正好等于 24×24）。进 7B 语言模型后再投到 576×4096。

> **一个容易混的细节：** DeepSeek-VL 不是"把原图切成很多块分别编码"，而是整图缩放到固定尺寸、两路各编码一次再融合。常被提到的 "S² 切分" 出自另一篇工作（InternLM-XComposer2-4KHD）：**DeepSeek-VL 论文全文 0 次出现 "S²"**（全文检索 0 命中）。

## 三、真正的难关有两个：块太多装不下，顺序又未必是语义顺序

第一道难关是**数量**。

如果图没被压缩，词元数是会爆炸的。同一张 1024×1024 的图，若改用单个方块边长 16 的编码器直接吃，会得到 64×64 = 4096 块，是 576 的 **7.1 倍**（这一行是本系列的教学推算，不是论文原文数字）。 而词元变多不只是"多占几行"，注意力要在词元之间两两比较，开销是超线性涨的——分辨率翻一倍、方块大小不变，词元数是 4 倍，注意力开销量级上是 **16 倍**。

576 这个预算有多大？LLaVA-1.5 用 336×336 的输入、CLIP-ViT-L/336px 编码器，也是 24×24 = 576 个词元。**同样 576 个词元的预算，DeepSeek-VL 装进去的是 1024 分辨率的画面。** 这就是"固定词元预算"：不追"看得更细就多给词元"，而是把预算钉死，逼编码器在有限预算里塞进更多内容。

代价必须写清：把长截图、票据这类"又长又小字"的图一律缩到 1024，小字会被压糊。DeepSeek-VL 自己的表里，OCRBench 得 456 分，同期 GPT-4V 是 659 分（前提：这是该论文表 5 中作者照录的对照成绩）。

第二道难关是**顺序**，它更隐蔽。

图是二维的，而模型读的是一维序列。通用做法是把图块按"光栅序"展平：第一行从左到右，再第二行……然后套上和文字一样的位置编码。

对一张猫的照片，这样没问题。但对一页文档、一张表格、一篇多栏排版，这个顺序**未必是语义顺序**：表头可能被排在表体之后，多栏文章可能从右栏跳到左栏，公式的推导链被打断。论文说这相当于给模型强加了一个"与语义无关的归纳偏置"。换句话说，**我们一直逼模型用"扫描仪的顺序"去理解一个本来按逻辑组织的东西。**

## 四、最反直觉的一步：把文字印成图片再传，确实能换来十倍的词元压缩

2025 年 10 月，DeepSeek-OCR 提出一个反问：既然图能被压成少量视觉词元，那么**把一页文字渲染成图片**，让视觉编码器压一遍，再让语言模型把这些词元还原成原文，行不行？这就是**光学上下文压缩**——不压缩模型内部，而是在**进入模型之前**就把文字换一种载体。

日常类比：把一页书拍成照片，比把它存成纯文字更省地方——一张照片的"每平方厘米信息量"远高于一排排字符。

**先把"压缩比"钉死**：分子是原文的文本词元数，分母是实际用掉的视觉词元数（论文图 1 的图注逐字给出该定义）。分子用谁的切词器、分母算输出词元而非切块数，都会改变这个数字。

| 视觉词元档 | 压缩比 | 字符级精度 | 前提 |
|---|---|---|---|
| 100 词元 | 6.7× | 98.5% | Fox 基准，英文文档 600–1300 词元，共 100 页 |
| 100 词元 | 12.6× | 87.1% | 同上 |
| 64 词元 | 10.5× | 96.5% | 同上 |
| 64 词元 | 19.7× | 59.1% | 同上，但这一档只有 4 页文档撑着 |


论文摘要的概括口径：压缩 9–10 倍时精度 96% 以上，10–12 倍约 90%，20 倍时约 60%。同一批文档在 64 与 100 词元两档之间，压缩比之比恒为 100÷64，约 1.5625，可见表内自洽。

三条代价必须一起说：

1. **这是字符级精度，不是"信息无损"。** 97% 意味着大约 3% 的字符可能出错，而合同、代码、公式对错误的容忍度远低于一次 OCR 演示。
2. **顺序最先垮。** 把"第 1 条……第 100 条"这样的列表渲染成图，条目边界会被卷积抹平。OmniDocBench 里专门有一列测顺序，DeepSeek-OCR 基础档英文 0.064、中文 0.181，比同一张表的文字列差得多。
3. **不可检索。** 文本上下文能被字符串匹配和检索工具索引，视觉词元序列不能——"压完之后还能不能查"归零。

还有一条外部批评必须带上：2025 年 12 月的论文 arXiv:2512.03643 用近乎零参数的均值池化和层级编码器做对照，主张"重建任务上，直接方法在所有压缩比上匹敌或超过视觉路径"，语言建模上视觉路径与"直接丢弃上下文"的截断基线相当，并写道 "The excitement around optical context compression outpaces the evidence"（兴奋超过了证据）。

> ⚠️ **别把两本账合成一本。** "压缩 10 倍"省的是**进入模型之前的词元条数**——每个视觉词元的缓存开销与文本词元同价，所以条数少了，单价没变。 而 V4.1 报告里的"KV 缓存压到 1/4 的 HBM、1/8 的 SSD 存储"（官方新闻页口径，分母是 V4-Flash），省的是**模型内部已经算出来的笔记**。一个是进门时的计数，一个是仓库里的存量，分母不同，不能相加，也不能互相"印证"。

## 五、顺序可以被训成机制，但"名字像"绝不等于是同一条技术线

2026 年 1 月，DeepSeek-OCR 2 把问题从"多少个词元够"换成"这些词元该按什么顺序读"，机制名叫**视觉因果流**（Visual Causal Flow）：在编码器里放一组**可学习的因果流查询**（可以理解成一组位置不固定、靠训练学出来的"探照灯"），每个查询能看到整张图，但只能看到排在它之前的查询——于是这串查询自己长成一条"先看标题、再看表格、沿公式往下推"的阅读链。查询数量正好等于视觉词元的数量，输出只取后一半喂给语言模型，词元总数因此不增加。

已核验的效果（每条都带前提）：

- **阅读顺序指标**（R-order，编辑距离，越低越好）从 0.085 降到 0.057，9 类文档全部改善。同一张表里对位的是 DeepSeek-OCR 的 1156 个词元上限，OCR-2 只用 1120。
- **整体指标** 87.36 升到 91.09（同表口径）。注意这只是端到端模型里的最高分——同表还有管线式模型更高，不能说"全场第一"。
- **生产环境的重复率**：图像场景 6.25% 降到 4.17%，PDF 场景 3.69% 降到 2.88%。这是"拿不到标准答案时能观测的质量代理指标"，不是语义正确率。

反例同样要说：**报纸 0.131 变到 0.139、书籍 0.022 变到 0.033，反而更差。** 作者归因于 1120 的上限对超密报纸不够、报纸训练数据只有 250k 条。此外论文没有给注意力图或查询轨迹来证明"查询真的按语义重排了"，也没有消融实验。

**下面是本篇想留给你的一手现场教学。** 网上有一种流传很广的说法：DeepSeek-V4.1 报告里的"因果编解码器"（Causal Encoder-Decoder，CED）和 OCR-2 的"视觉因果流"，是同一条技术线。回源核实的结果正好相反：

- V4.1 全文 **0 次**出现 OCR-2 的论文编号（2601.20552）、"DeepSeek-OCR"、"visual causal flow"；148 条参考文献里没有 OCR-2。
- V4.1 全文 **0 次**出现 "codec" 这个词——"因果编解码器"是传播里的意译，论文自己的写法是 Causal **Encoder-Decoder**。
- 机制也不同：CED 指 40 层被切成 20 层编码器加 20 层解码器，解码器的全局 KV 由编码器末层投影而来（论文自述受 YOCO 启发）；OCR-2 指视觉编码器内部用查询给词元排序（血统是 DETR、BLIP-2 式的可学习查询）。
- 两者共享的，只有"因果掩码"这个到处都在用的通用词。


**所以结论是：名词像，不等于同一条线。** 看到相似的技术名，先别画谱系图，该查三件事——有没有互相引用、原文用词是否相同、机制是不是同一件事。这条纪律比记住任何机制都耐用。

最后收到 2026 年：V4.1-Flash 是 552B 的专家混合模型，40 层，视觉侧是自研的 DeepSeek-ViT（32 层、方块边长 14、3×3 像素重排把词元数降 9 倍、上限 1344×1344、最多 1024 个视觉词元）加两层 MLP 投影；论文写明它是**第一个从语言预训练第一天起就原生多模态的旗舰**（V4 的视觉只是实验线，OCR 线是文档专用小模型）。它的多模态评测只有绝对分、没有基线列（MMMU-Pro 56.5、CVBench 77.9、DocVQA 95.6、RefCOCO-avg 86.0），所以这里只能陈述"考了多少"，不能说"超过了谁"。

为什么说这条线和"省算力"主线在这里汇合？因为**图片会变成大量视觉词元，词元多，KV 缓存压力就大**——省算力和多模态在新旗舰里成了同一个工程问题。但要如实补一句：视觉编码器本身没和 KV 压缩机制耦合，这一代收益来自"共用同一个骨干模型"，而不是"对视觉部分做 KV 压缩"。

---

**一句话总结：** 多模态的工程史就是两个问题——一张图要花多少词元、这些词元按什么顺序读；而 2025 年最反直觉的一步（把文字渲染成图片再压）证明"载体"本身也能省钱，代价是精度、顺序和可检索性。

**记住这个数字：** 10×——一页排版规整的文档渲染成图片后，约 10 个文本词元的内容能压进 1 个视觉词元；代价是字符级精度约 97%（不是信息无损），且顺序敏感的内容最先吃亏。

**下一篇：** [论文里的数字怎么看才不被骗](/posts/deepseek-paper-course-12-numbers/)
