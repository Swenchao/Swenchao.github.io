---
title: Flink-Window
top: false
cover: false
toc: true
mathjax: true
date: 2021-05-09 20:20:36
password:
summary: Flink中窗口使用
tags:
  - Flink
categories:
  - 大数据
---

# Flink

## Flink 中的 window

### Window

#### Window 概述

streaming 流式计算是一种被设计用于处理无限数据集的数据处理引擎，而无限数据集是指一种不断增长的本质上无限的数据集，而 window 是一种切割无限数据为有限块进行处理的手段。

Window 是无限数据流处理的核心，Window 将一个无限的 stream 拆分成有限大小的”buckets”（桶），我们可以在这些桶上做计算操作。

#### Window 类型

##### TimeWindow（时间窗口）

按照时间生成 Window。对于 TimeWindow，可以根据窗口实现原理的不同分成三类:

1. 滚动窗口(Tumbling Windows)

将数据依据固定的窗口长度对数据进行切片。

特点：时间对齐，窗口长度固定，没有重叠。

滚动窗口分配器将每个元素分配到一个指定窗口大小的窗口中，滚动窗口有一个固定的大小，并且不会出现重叠。

例如:如果你指定了一个 5 分钟大小的滚动窗口，窗口的创建如下图所示

![6-1 滚动窗口](6-1滚动窗口.png)

适用场景：适合做 BI 统计等(做每个时间段的聚合计算)。

2. 滑动窗口(Sliding Windows)

滑动窗口是固定窗口的更广义的一种形式，滑动窗口由固定的窗口长度和滑动间隔组成。

特点：时间对齐，窗口长度固定，可以有重叠。

滑动窗口分配器将元素分配到固定长度的窗口中，与滚动窗口类似，窗口的大小由窗口大小参数来配置，另一个窗口滑动参数控制滑动窗口开始的频率。因此，滑动窗口如果滑动参数小于窗口大小的话，窗口是可以重叠的，在这种情况下元素会被分配到多个窗口中。

例如，你有 10 分钟的窗口和 5 分钟的滑动，那么每个窗口中 5 分钟的窗口里包含着上个 10 分钟产生的数据，如下图所示:

![6-2 滑动窗口](6-2滑动窗口.png)

适用场景：对最近一个时间段内的统计(求某接口最近 5min 的失败率来决定是否要报警)。

**注：**滚动窗口可看成是特殊的滑动窗口（滑动步长等于窗口长度）

3. 会话窗口(Session Windows)

由一系列事件组合一个指定时间长度的 timeout 间隙（两个窗口之间的最小时间间隔）组成，类似于 web 应用的 session，也就是一段时间没有接收到新数据就会生成新的窗口。

特点：时间无对齐。

session 窗口分配器通过 session 活动来对元素进行分组，session 窗口跟滚动窗口和滑动窗口相比，不会有重叠和固定的开始时间和结束时间的情况，相反，当它在一个固定的时间周期内不再收到元素，即非活动间隔产生，那个这个窗口就会关闭。一个 session 窗口通过一个 session 间隔来配置，这个 session 间隔定义了非活跃周期的长度，当这个非活跃周期产生，那么当前的 session 将关闭并且后续的元素将被分配到新的 session 窗口中去。

![6-3 会话窗口](6-3会话窗口.png)

##### CountWindow（计数窗口）

按照指定的数据条数生成一个 Window，与时间无关。

1. 滚动计数窗口

将滚动时间窗口中的时间换成了数据条数（每几条数据是一个窗口）

2. 滑动计数窗口

将滑动时间窗口中的时间换成了数据条数（每 x 条数据一个窗口，每个窗口间隔 y 条数据）

### Window API

#### 窗口分配器 —— window() 方法

我们可以用 .window() 来定义一个窗口，然后基于这个 window 去做一些聚合或者其它处理操作。注意 window() 方法必须在 keyBy 之后才能用。

Flink 提供了更加简单的 .timeWindow 和 .countWindow 方法，用于定义时间窗口和计数窗口。

window() 方法接收的输入参数是一个 WindowAssigner

- WindowAssigner 负责将每条输入的数据分发到正确的 window 中

- Flink 提供了通用的 WindowAssigner

  滚动窗口(tumbling window)

  滑动窗口(sliding window)

  会话窗口(session window)

  全局窗口(global window)

#### 创建不同类型窗口

- 滚动时间窗口(tumbling time window)

```java
.timeWindow(Time.seconds(15))
```

- 滑动时间窗口(sliding time window)

```java
// 窗口大小 滑动步长
.timeWindow(Time.seconds(15), Time.seconds(5))
```

- 会话窗口(session window)

```java
// 间隔1分钟的会话窗口
.window(EventTimeSessionWindows.withGap(Time.minutes(1)))
```

- 滚动计数窗口（10 个数据为一个窗口）

```java
.countWindow(10);
```

- 滑动计数窗口（10 个数据为一个窗口，滑动步长为 2 个数据）

```java
.countWindow(10, 2);
```

**样例**

```java
// 窗口测试
// 基于dataStream可以调windowAll方法
// Note: This operation is inherently non-parallel since all elements have to pass through
//	 * the same operator instance.  （源码注解：因为所有元素都被传递到下游相同的算子中，所以本质上是非并行的）
//         mapStream.windowAll();
// 开窗口前要先进行keyBy
mapStream.keyBy("id")
        // 时间窗口
        // .window(TumblingProcessingTimeWindows.of(Time.seconds(15)))
        // 根据传参，判断开的是什么窗口（滚动 滑动）
        // .timeWindow(Time.seconds(15))
        // .timeWindow(Time.seconds(15), Time.seconds(5))
        // 会话窗口（1分钟间隔）
        // .window(EventTimeSessionWindows.withGap(Time.minutes(1)))
        // 计数窗口
        // 窗口大小 滑动步长
        .countWindow(10, 2);
```

### Window Function（窗口函数）

在开窗之前先做一次 keyBy（分组）操作，然后再进行开窗（分桶操作：哪些数据归位一组），最后还需要一次聚合操作进行统计（在分桶之后，要对桶内数据做操作）

window function 就定义了要对窗口中收集的数据做的计算操作

#### 不同窗口函数使用

1. 增量聚合函数(incremental aggregation functions)

每条数据到来就进行计算，保持一个简单的状态（ReduceFunction, AggregateFunction）

2. 全窗口函数(full window functions)

先把窗口所有数据收集起来，等到计算的时候会遍历所有数据（ProcessWindowFunction，WindowFunction）

可用于统计数据中位数一类操作

**需求：**统计 15 秒内，每个传感器传过来的数据条数（从增量聚合函数和全窗口函数最后结果可以看出，虽然全窗口函数效率会低一些，但是能拿到更多的信息）

```java
// ...
// 增量聚合函数
SingleOutputStreamOperator<Integer> resultStream = mapStream.keyBy("id")
    .timeWindow(Time.seconds(15))
    // 第一个Integer为累加器类型，第二个为输出
    .aggregate(new AggregateFunction<SensorReading, Integer, Integer>() {
        @Override
        // 创建累加器
        public Integer createAccumulator() {
            // 初始值为 0
            return 0;
        }

        @Override
        // sensorReading：传过来的传感器数据   accumulator：累加器
        public Integer add(SensorReading sensorReading, Integer accumulator) {
            // 来一条数据就加1
            return accumulator + 1;
        }

        @Override
        // 返回结果
        public Integer getResult(Integer accumulator) {
            return accumulator;
        }

        @Override
        // 一般用于会话窗口中（合并分区）
        public Integer merge(Integer a, Integer b) {
            return a + b;
        }
    });

//        resultStream.print();

// 全窗口函数
SingleOutputStreamOperator<Tuple3<String, Long, Integer>> resultStream2 = mapStream.keyBy("id")
    .timeWindow(Time.seconds(15))
    /*
        * @param <IN> The type of the input value.
        * @param <OUT> The type of the output value.
        * @param <KEY> The type of the key.
        * @param <W> The type of {@code Window} that this window function can be applied on.
        */
    .apply(new WindowFunction<SensorReading, Tuple3<String, Long, Integer>, Tuple, TimeWindow>() {
        @Override
        public void apply(Tuple tuple, TimeWindow window, Iterable<SensorReading> input, Collector<Tuple3<String, Long, Integer>> out) throws Exception {
            String id = tuple.getField(0);
            Long timeEnd  = window.getEnd();
            int count = IteratorUtils.toList(input.iterator()).size();
            out.collect(new Tuple3<>(id, timeEnd, count));
        }
    });

resultStream2.print();
// ...
```

[demo 地址](https://github.com/Swenchao/FlinkCode/blob/main/src/main/java/com/swenchao/apitest/window/WindowTest1_TimeWindow.java)

**需求**统计传来 10 个数据的平均值（10 个数据为一个窗口，2 个数据为移动步伐）

```java
public static class MyAvgTemp implements AggregateFunction<SensorReading, Tuple2<Double, Integer>, Double> {

    @Override
    public Tuple2<Double, Integer> createAccumulator() {
        return new Tuple2<>(0.0, 0);
    }

    @Override
    public Tuple2<Double, Integer> add(SensorReading sensorReading, Tuple2<Double, Integer> accu) {
        return new Tuple2<>(accu.f0 + sensorReading.getTemperature(), accu.f1 + 1);
    }

    @Override
    public Double getResult(Tuple2<Double, Integer> accu) {
        return accu.f0 / accu.f1;
    }

    @Override
    public Tuple2<Double, Integer> merge(Tuple2<Double, Integer> a, Tuple2<Double, Integer> b) {
        return new Tuple2<>(a.f0 + b.f0, a.f1 + b.f1);
    }
}
```

[demo 地址](https://github.com/Swenchao/FlinkCode/blob/main/src/main/java/com/swenchao/apitest/window/WindowTest2_CountWindow.java)

### 其它可选 API

- .trigger() —— 触发器

定义 window 什么时候关闭，触发计算并输出结果

- .evitor() —— 移除器

定义移除某些数据的逻辑

- .allowedLateness() —— 允许处理迟到的数据（传一个时间数：允许的最大延迟时间）

若此窗口原先九点关闭，那么其在九点会先输出一个结果，然后在以后的一分钟内，若来一个此窗口的数据则更新一下之前输出的结果，直到 9:01

- .sideOutputLateData() —— 将迟到的数据放入侧输出流

- .getSideOutput() —— 获取侧输出流

```java
// 延迟数据侧输出流
OutputTag<SensorReading> outputTag = new OutputTag<>("late");

// 正常数据
SingleOutputStreamOperator<SensorReading> sumStream = mapStream.keyBy("id")
        .timeWindow(Time.seconds(15))
        .allowedLateness(Time.minutes(1))
        .sideOutputLateData(outputTag)
        .sum("temp");

// 拿到延迟数据
sumStream.getSideOutput(outputTag).print("late");

// 拿到延迟数据与正常数据之后，再做一个合并就可以取到所有数据的和了
```

![6-4 其他可选api.png](6-4其他可选api.png)

**注：**什么样的数据才算是迟到数据？

根据事件时间来进行判断，而不是数据到来时间。比如：有一条数据产生于 8:57，但是他 9:01 才到，那么其就是 8:00-9:00 窗口的迟到数据。

![6-5](6-5转换.png)

> 断断续续终于完成这部分了，接下来顺顺利利啊～
