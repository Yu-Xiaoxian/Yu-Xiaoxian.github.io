---
title: Bayes_theorem_and_its_Application
tags:
- SLAM
- Probability Theory
- Robotic
mathjax: true
---

# 贝叶斯法则及其应用

## 贝叶斯法则

贝叶斯法则，其公式如下
$$
p(x|y)=\frac{p(y|x)p(x)}{p(y)}
$$
在贝叶斯推论中，p(x)称为先验概率，因为p(x)不考虑任何条件y的因素；p(x|y)称为后验概率，是在y发生的条件下，x发生的概率。

## 贝叶斯法则应用

### 兴奋剂检测

已知：某兴奋剂检测结果为阳性和阴性的概率分别为99%和1%，而运动员中服用该兴奋剂的概率为0.5%

问：在随机抽查中检测结果为阳性的运动员，有多大概率服用了该兴奋剂

答
$$
\begin{align}p(user|+)
=&\frac{p(+|user)p(user)}{p(+)} \\
=&\frac{p(+|user)p(user)}{p(+|user)p(user)+p(+|non-user)p(non-user)}\\
=&\frac{0.99\times0.005}{0.99\times0.005+0.01\times0.995}\\
\approx&0.3322
\end{align}
$$
当服用兴奋剂的运动员占比较低时，很容易出现假阳，假阳概率高达66.8%