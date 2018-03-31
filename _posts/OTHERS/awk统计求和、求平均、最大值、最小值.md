---
title: awk统计求和、求平均、最大值、最小值
date: 2017-06-28 16:38:58
tags: [linux,awk]
category: [OTHERS]
---

awk统计求和、求平均、最大值、最小值

<!--more-->

## 一、求和
```
cat data|awk '{sum+=$1} END {print "Sum = ", sum}'

```


## 二、求平均
```
cat data|awk '{sum+=$1} END {print "Average = ", sum/NR}'

```


## 三、求最大值
```
cat data|awk 'BEGIN {max = 0} {if ($1+0>max+0) max=$1 fi} END {print "Max=", max}'

```


## 四、求最小值
```
awk 'BEGIN {min = 1999999} {if ($1+0<min+0) min=$1 fi} END {print "Min=", min}'

```