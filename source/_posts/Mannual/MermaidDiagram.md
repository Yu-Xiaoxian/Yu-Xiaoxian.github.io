---
title: 配置 Hexo 支持 Mermaid（流程图、甘特图）
tags:
  - Hexo
  - Mermaid
  - 前端
categories:
  - Mannual
date: 2019-05-03 23:43:16

updated:: 2019-05-03 23:43:16
---

# Mermaid 示例
## 流程图
```mermaid
graph LR;
A[Talker] --Hello World!--> B[Listener]
```

## 甘特图
```mermaid
gantt
dateFormat  YYYY-MM-DD
title Test GANTT diagram

section A section
Completed work  :done,    des1, 2014-01-06,2014-01-08
Active work     :active,  des2, 2014-01-09, 3d
Future work     :         des3, after des2, 5d
```

## 序列图
```mermaid
sequenceDiagram
    participant Alice
    participant John
    Alice->>John: Hello John, how are you?
	John->>Alice: Fine, thanks, and you?
	Alice->>John: I'm fine, too.
```

# 参考链接
1. [Hexo中引入Mermaid流程图](https://tyloafer.github.io/posts/7790/)
2. [GitHub:hexo-filter-mermaid-diagrams](https://github.com/webappdevelp/hexo-filter-mermaid-diagrams)
3. [mermaid GitBook](https://mermaidjs.github.io/)


