---
title: python求list的平均数、众数、最大值、最小值、极值、方差等
date: 2017-06-07 17:03:36
tags: [python]
categories: python
---
数据挖掘中经常会用到一组数据(list)的相关特征。
<!--more-->
```python
'''
feature.py
'''
#最大数
def Get_Max(list):
    return max(list)

#最小数
def Get_Min(list):
    return min(list)

#极差
def Get_Range(list):
    return max(list) - min(list)

#中位数
def get_median(data):
   data = sorted(data)
   size = len(data)
   if size % 2 == 0: # 判断列表长度为偶数
       median = (data[size//2]+data[size//2-1])/2
   if size % 2 == 1: # 判断列表长度为奇数
       median = data[(size-1)//2]
   return median

#众数(返回多个众数的平均值)
def Get_Most(list):
    most=[]
    item_num = dict((item, list.count(item)) for item in list)
    for k,v in item_num.items():
        if v == max(item_num.values()):
           most.append(k)
    return sum(most)/len(most)

#获取平均数
def Get_Average(list):
	sum = 0
	for item in list:
		sum += item
	return sum/len(list)

#获取方差
def Get_Variance(list):
	sum = 0
	average = Get_Average(list)
	for item in list:
		sum += (item - average)**2
	return sum/len(list)

#获取n阶原点距
def Get_NMoment(list,n):
    sum=0
    for item in list:
        sum += item**n
    return sum/len(list)
```