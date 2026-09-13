---
title: Quartz 写作特性速览
date: 2026-09-13
tags:
  - meta
  - 教程
draft: false
---

这篇演示 Quartz 原生支持的 Obsidian 特性，用来验证发布后是否正常显示。

## 双链与反向链接

可以引用 [[欢迎来到 James 的投资笔记]]，发布后会变成站内链接，并自动生成反向链接（backlinks）。

## Callout 提示框

> [!warning] 风险提示
> 任何标的讨论都先讲风险，再谈收益。小资金也不等于无风险。

> [!tip] 小技巧
> 在笔记 frontmatter 里写 `draft: true`，可以让这篇留在本地、不参与发布。

## 代码块

```python
def sharpe(returns, rf=0.02):
    excess = returns - rf
    return excess.mean() / excess.std()
```

## 数学公式

夏普比率：$S = \frac{\mathbb{E}[R_p - R_f]}{\sigma_{R_p}}$

## 列表

- 确定性优先
- 分批买卖纪律
- 不碰看不懂的标的
