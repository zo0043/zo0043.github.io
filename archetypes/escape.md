---
title: "{{ .File.ContentBaseName }}"
date: {{ .Date }}
aliases: []
description: "" # 必填：留空会让 og/twitter/JSON-LD 回退到正文摘要，泄漏字面 [[…]]
chapter: "01 先认清你在跟谁玩" # 所属章节，决定列表页分组；章节名即排序键
tags: ["A股", "逃生指南"]
cost: { money: "0", time: "少", will: "否" } # 钱=0|少|多  时间=少|中|多  毅力=否|些|是
gain: "中" # 大|中|小
metric: "亏损" # 亏损|收益|时间|自由|处罚
evidence: "B" # A|B|C
disputed: false # 可选：A/B 级存在反方证据时置 true
checked: 2026-10-08 # 来源核对日期；引用监管文件的条目必填
---

正文固定五节；缺任何一节、或上面任何字段取值非法，构建会直接失败
（校验器 `layouts/partials/guide-guard.html` 在构建期运行，绕不过去）。

## 怎么做

一次能做完的具体动作。

## 说人话

用大白话讲清为什么。**这一节不许出现阿拉伯数字**——它只负责解释，不引入「收益」栏之外的新数字。这条由 `guide-guard.html` 机检。

## 收益

按 `metric` 口径给量级。

## 来源

- 只写原始出处，逐条附核对日期。例如：中国投资者网《维权路径与机构》栏目 <http://www.investor.org.cn/qybh/wqfw/wqljyjg/>（核对于 2026-10-08）

## 备注

边界条件、适用前提、已知反例。

想引用别的条目就写 `[[条目名]]`；想写字面的方括号，用行内代码 `[[这样]]` 或在源文件里写 \\[[这样]]。
