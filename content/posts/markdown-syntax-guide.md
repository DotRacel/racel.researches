---
author: "racel & hugo"
title: "Markdown 语法模板"
date: "2024-03-02"
description: "Markdown 基础语法模板和 HTML 格式模板"
tags: ["markdown", "css", "html", "themes"]
categories: ["中文", "English", "templates"]
series: ["Themes Guide"]
aliases: ["migrate-from-jekyl"]
ShowToc: true
TocOpen: true
---

这篇文章提供了基础 Markdown 的语法，一般用于 Hugo 的内容编写，同时还展示了基础 HTML 元素是怎么在 Hugo 主题中表现的。

<!--more-->

## Headings 标题

下列 HTML `<h1>`—`<h6>` 标签表示了六层标题的展示. `<h1>` 为最高一级标题, `<h6>` 是最低级标题.

# H1

## H2

### H3

#### H4

##### H5

###### H6

## Paragraph 段落

Xerum, quo qui aut unt expliquam qui dolut labo. Aque venitatiusda cum, voluptionse latur sitiae dolessi aut parist aut dollo enim qui voluptate ma dolestendit peritin re plis aut quas inctum laceat est volestemque commosa as cus endigna tectur, offic to cor sequas etum rerum idem sintibus eiur? Quianimin porecus evelectur, cum que nis nust voloribus ratem aut omnimi, sitatur? Quiatem. Nam, omnis sum am facea corem alique molestrunt et eos evelece arcillit ut aut eos eos nus, sin conecerem erum fuga. Ri oditatquam, ad quibus unda veliamenimin cusam et facea ipsamus es exerum sitate dolores editium rerore eost, temped molorro ratiae volorro te reribus dolorer sperchicium faceata tiustia prat.

Itatur? Quiatae cullecum rem ent aut odis in re eossequodi nonsequ idebis ne sapicia is sinveli squiatum, core et que aut hariosam ex eat.

## Blockquotes 块引用

块引用经常用于标志引用得到的内容，一般会在 `页脚` 或 `引用` 元素中，或者是用在注释或者缩写中使用。

#### Blockquote without attribution 无出处的块引用

> Tiam, ad mint andaepu dandae nostion secatur sequo quae.
> **注意** 块引用中仍然遵循 Markdown 的基本语法规则

#### Blockquote with attribution 有出处的块引用

> Don't communicate by sharing memory, share memory by communicating.
>
> — Rob Pike[^1]

[^1]: The above quote is excerpted from Rob Pike's [talk](https://www.youtube.com/watch?v=PAAkCSZUG1c) during Gopherfest, November 18, 2015.

## Tables 表格

基础的 Markdown 并不支持表格，但是 Hugo 一开始便支持了这个功能。

| Name  | Age |
| ----- | --- |
| Bob   | 27  |
| Alice | 23  |

#### 表格中可以继续使用 Markdown 语法

| Italics   | Bold     | Code   |
| --------- | -------- | ------ |
| _italics_ | **bold** | `code` |

## 列表类型

#### 有序列表

1. First item
2. Second item
3. Third item

#### 无序列表

- List item
- Another item
- And another item

#### 嵌套的无序列表

- Fruit
  - Apple
  - Orange
  - Banana
- Dairy
  - Milk
  - Cheese

#### 嵌套的有序列表

1. Fruit
    - Apple
    - Orange
    - Banana
2. Dairy
    1. Milk
    2. Cheese
3. Third item
    1. Sub One
    2. Sub Two

## 其他元素 — abbr, sub, sup, kbd, mark

<abbr title="Graphics Interchange Format">GIF</abbr> is a bitmap image format.

H<sub>2</sub>O

X<sup>n</sup> + Y<sup>n</sup> = Z<sup>n</sup>

Press <kbd><kbd>CTRL</kbd>+<kbd>ALT</kbd>+<kbd>Delete</kbd></kbd> to end the session.

Most <mark>salamanders</mark> are nocturnal, and hunt for insects, worms, and other small creatures.