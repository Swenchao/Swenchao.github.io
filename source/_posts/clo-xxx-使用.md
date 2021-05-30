---
title: clo-xxx-*使用
top: false
cover: false
toc: true
mathjax: true
date: 2020-05-17 12:08:13
password:
summary: bootstrap网格使用
tags:
- 前端
categories:
- 前端
---

# Bootstrap —— 网格系统

## 分类

.col-xs-* 超小屏幕 手机 (<768px)

.col-sm-* 小屏幕 平板 (≥768px)

.col-md-* 中等屏幕 桌面显示器 (≥992px)

.col-lg-* 大屏幕 大桌面显示器 (≥1200px)

## 关键字说明

1. col —— column：列；

2. xs —— maxsmall：超小； sm —— small：小；  md —— medium：中等；  lg —— large：大；

3. -*  表示占列，即占自动每行 row 分12列栅格系统比；

4. 不管在哪种屏幕上，栅格系统都会自动将每 row 行分12列 col-xs-* 和 col-sm-* 和 col-md-*后面跟的参数表示在当前的屏幕中每个div所占列数。例如 <div class="col-xs-6 col-md-3"> 这个div在屏幕中占的位置是：在超小屏幕中这个div占6列也就是屏幕的一半；在中等屏幕占3列也就是1/4。

6、反推，如果我们要在小屏幕上并排显示3个div(12/3个=每个占4 列 )，则用 col-xs-4；在大屏幕上显示6个div(12/6个=每个占2列 ) ，则用 col-md-2；这样我们就可以控制我们自己想要的什么排版了。

## 案列

### 样式单一使用

```html
    <div class="container">
        <div class="row">
            <div class="col-md-4">col-md-4</div>
            <div class="col-md-4">col-md-4</div>
            <div class="col-md-4">col-md-4</div>
            <!-- 说明：每row行共12列，分个3div，每个div平占4列，即3个*4列=12列 -->
        </div>
        <div class="row">
            <div class="col-md-4">col-md-4</div>
            <div class="col-md-8">col-md-8</div>
            <!-- 说明：每row行共12列，分个2div，第1个div占4列，第2个div则占8列，即4列+8列=12列 -->
        </div>
        <div class="row">
            <div class="col-md-3">col-md-3</div>
            <div class="col-md-6">col-md-6</div>
            <div class="col-md-3">col-md-3</div>
            <!-- 说明：每row行共12列，分个3div，每1,3个div占3列，第2个div则占6列，即3列+6列+3列=12列 -->
        </div>
    </div>
```

### 混合使用

```html
<!-- 说明：当屏幕尺寸不同时，调用不同的样式-->

<div class="col-xs-12 col-sm-9 col-md-6 col-lg-3">测试</div>
```