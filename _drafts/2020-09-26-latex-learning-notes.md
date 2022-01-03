---
layout: post
title: LaTex 学习笔记
categories: 论文
tags: LaTex 排版
author: ZekeTian
---





## 1. LaTex 源文件基本结构

### 1.1 导言区

功能：可用于全局设置，如：documentclass、title、author、date

语法：

```latex
\documentclass{类名}
```

注意：一个 LaTex 有且仅有一个导言区

### 1.2 正文区

语法：

```latex
\begin{类名}

$数学模式$ 文本模式
\end{类名}
```

正文区可以

## 2. 基本语法

- 数学公式

  - 行内公式：`$ 公式内容 $`，只有一行
  - 行间公式：`$$ 公式内容 $$`，多行

- 使用包

  ```latex
  \usepackage{包名}
  ```

- 章节

  ```latex
  \section{章节名}
  \subsection{子章节名}
  \subsubsection{子章节名}
  ```

- 换行

  - 第一种方式：插入一个新行（无论插入多少个新行，最终效果都是只换一次行）
  - 第二种方式：`\\`

  



### 