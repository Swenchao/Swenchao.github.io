---
title: R-work1
top: false
cover: false
toc: true
mathjax: true
date: 2020-04-05 23:16:39
password:
summary: 大数据项目作业（R）
tags:
- R
categories:
- R
---

# R作业

## 题目描述

1. 请使用R 软件进行以下编程要求：

- 请在R 中，执行以下代码（注意：请将studID 数字置换成你的学号）：
  studID <- 2017120102
  set.seed(studID)
  n <- 10000
  num <- sample(n, 1:100000, replace=TRUE)

- 编写函数，将其命名为findPrime，可以用来查验以上num 物件中质数的数量，
并由大到小进行排序，返回前第17,153,2349 个质数（注意：函数要同时能返回
质数的数量和第17,153,2349 个质数质数的物件）。

2. 请使用layout() 将三幅图画在同一个图面上，图面大小安排如下（注意：为了图
    面美观，请进行必要的par 参数调整，例如：在layout 指令后，键入以下代码
    par(mar=c(3,3,1,1), mgp=c(2,0.2,0), tcl=-0.2)）


![](1586072291(1).png)


- 第一幅图——条状图（hist()）。
数据：
y <- rnorm(100, mean=2000, sd=50)


- 第二幅图——气泡图
数据：
SAT <- data.frame(
”studentID” = c(”A”, ”B”, ”C”, ”D”, ”E”, ”F”, ”G”, ”H”),
”improvement” = c(100, 57, 80, 191, 5, 10, 25, 123),
”origGrades” = c(712, 1105, 690, 687, 725, 1200, 470, 752),
”weekInSch” = c(18, 4, 7, 27, 2, 25, 19, 10))


- 第三幅图——中国GDP 地图
数据：
set.seed(1)
GDP <- rnorm(34, mean=5000, sd=50)

## 代码

```R
# 1

studID <- 19202025
set.seed(studID)
n <- 10000
num <- sample(size = n, 1:100000, replace=TRUE)
findPrime<-function(num){
  res <- numeric(length =  0)  #存放返回质数
  index <- 1  #质数个数
  for(i in num){
    j <- 2
    flag <- 0  #是否为质数标志
    #  筛选质数
    while (j*j <= i) {
      if(i %% j == 0){
        flag <- 1
        break
      }
      j = j + 1
    }
    #  是质数存到有序数组中
    if(flag == 0){
      # browser()
      i_temp <- index - 1
      #  原本返回数组为空
      if(i_temp == 0){
        res[i_temp + 1] <- i
        index <- index + 1
      }
      else{
        # 不为空进行插入排序，找到合适位置
        while(res[i_temp] > i & i_temp >= 1){
          res[i_temp + 1] <- res[i_temp]
          i_temp <- i_temp - 1
          if(i_temp == 0)
            break
        }
        res[i_temp+1] <- i
        index = index + 1
      }
    }
  }
  l <- c(res, res[17], res[153])
  return(l)
}
res <- findPrime(num)


# 2
layout(matrix(c(1,3,2,3),2,2))
par(mar=c(3,3,1,1), mgp=c(1.5,-0.2,0), tcl=-0.2)


y <- rnorm(100, mean=2000, sd=50)
hist(y)



SAT <- data.frame(
  'studentID' = c('A', 'B', 'C', 'D', 'E', 'F', 'G', 'H'),
  'improvement' = c(100, 57, 80, 191, 5, 10, 25, 123),
  'origGrades' = c(712, 1105, 690, 687, 725, 1200, 470, 752),
  'weekInSch' = c(18, 4, 7, 27, 2, 25, 19, 10))
#  当前默认处理数据
attach(SAT)
plot(x = seq(0,250,50), y = seq(200,1600,250), xlab = "improvement", ylab = "origGrades", type = "n")
points(x = improvement, 
       y = origGrades,
       cex = weekInSch/2,
       col = studentID, pch = 16)
text(improvement, origGrades, studentID)


#  调整地图样式
par(mar=c(1,3,3,1), mgp=c(1.5,-0.2,0), tcl=-0.2)

library(raster)
library(rgdal)
library(maps)

set.seed(1)
GDP <- rnorm(34, mean = 5000, sd = 50)
#  空间数据导入
CN.map <- readOGR(dsn="d:/scwri/Desktop/R/bou2_4p.shp",layer="bou2_4p")
projection(CN.map)<- CRS("+proj=longlat +ellps=WGS84 +towgs84=0,0,0,0,0,0,0 +no_defs")

chinamaps <- function (mapObj, a, grayscale=FALSE, lo, hi,...){
  if (length(a)!=34){
    stop ("wrong number of provinces")
  }
  #  颜色获取
  getColor <- function(mapdata, provname, provcol, othercol){
    FUN <- function(x, y){
      ifelse(x %in% y, which(y==x), 0)
    }
    colIndex <- sapply(mapdata@data$NAME, FUN=FUN, provname)
    cols <- c(othercol, provcol)[colIndex+1];
    return(cols)
  }
  a.long <-  getColor(mapObj, levels(mapObj$NAME), a, "white")
  b.long <- ifelse(is.na(a.long), "black", "gray85")
  a.long <- ifelse(is.na(a.long), "white", a.long)
  #  画图
  plot(mapObj, col=a.long, border=b.long, lwd=0.5)
}
#  颜色主题
cols1 <- terrain.colors(34, alpha = 1)
chinamaps(mapObj=CN.map, a=rev(cols1)[order(GDP)])
```


## 总结

1. 感觉r中while循环执行好像跟之前接触的有些不同

```R
# 不为空进行插入排序，找到合适位置
while(res[i_temp] > i & i_temp >= 1){
    res[i_temp + 1] <- res[i_temp]
    i_temp <- i_temp - 1
    if(i_temp == 0)
    break
}
```

以上代码中，如果  res[i_temp] > i   条件中，res数组越界，则会直接报错而不会进行判断，此处纠结好久（其实以上代码就是一个插入排序）

2. num <- sample(size = n, 1:100000, replace=TRUE)

sample随机抽样语句，size是样本大小，1:100000是范围，replace是是否有放回

3. layout(matrix(c(1,3,2,3),2,2))

画布布局。其中matrix中的c(1,3,2,3)表示画布分成四格，第一格是第一幅图；第二个格和第四格是第三幅图；第三格是第二幅图。下面是格子分布：


![](1586075348(1).png)


4. ifelse(x %in% y, which(y==x), 0)

- %in%：判断前面一个向量内的元素是否在后面一个向量中，返回布尔值

```R
a <- c(1,3,13,1443,43,43,4,34,3,4,3)
b <- c(1,13,11,1313,434,1)
a %in% b
# 返回内容
# [1]  TRUE FALSE  TRUE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
```

- ifelse(a,b,c)：如果a成立，则b，否则c

- 颜色主题

heat.colors() 从红色渐变到黄色再变到白色（以体现“高温”、“白热化”）。
terrain.colors() 从绿色渐变到黄色再到棕色最后到白色（这些颜色适合表示地理地形）。
cm.colors() 从青色渐变到白色再到粉红色。
topo.colors() 从蓝色渐变到青色再到黄色最后到棕色。