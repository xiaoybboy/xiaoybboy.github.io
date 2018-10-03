---
title: 第N个丑数
date: 2017-03-20 15:18:42
tags: [算法]
categories: 算法
---
# 题目描述
{% note primary %}&emsp;&emsp;把只包含因子2、3和5的数称作丑数（Ugly Number）。例如6、8都是丑数，但14不是，因为它包含因子7。习惯上
我们把1当做是第一个丑数。求按从小到大的顺序的第N个丑数。 {% endnote %}
<!--more-->
# 思路
{% note success %}&emsp;&emsp;暴力求解思路。首先我们要搞清楚怎么求丑数。ugly[]表示丑数数组。{% endnote %}
1. ugly[0]=1;
2. ugly[1]=1*2;
3. ugly[3]=1*3;
4. ugly[4]=1*2*2;
5. ugly[5]=1*5;
6. ugly[6]=1*2*3;
7. ...
&emsp;&emsp;是不是可以找出规律来，我们需要记录 丑数中因子 2,3,5 出现的次数。具体用语言不好描述，可以看出规律。
# 代码
```java
/**
 * 把只包含因子2、3和5的数称作丑数（Ugly Number）。例如6、8都是丑数，但14不是，因为它包含因子7。
 * 习惯上我们把1当做是第一个丑数。求按从小到大的顺序的第N个丑数。
 * 
 * @author XIAO
 *
 */

public class GetUglyNumber {
	public int GetUglyNumber_Solution(int index) {
		if (index <= 0) {
			return 0;
		}
		int[] result = new int[index];
		result[0] = 1;
		int i2 = 0, i3 = 0, i5 = 0;// i2,i3,i5分别记录丑数的因子中2,3,5的个数
		for (int i = 1; i < index; i++) {
			result[i] = min(result[i2] * 2, min(result[i3] * 3, result[i5] * 5));
			if (result[i] == result[i2] * 2) {
				i2++;
			}
			if (result[i] == result[i3] * 3) {
				i3++;
			}
			if (result[i] == result[i5] * 5) {
				i5++;
			}
		}
		return result[index - 1];
	}

	private int min(int i, int j) {
		return i > j ? j : i;
	}
}
```

