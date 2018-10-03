---
title: 用sklearn做数据预处理
date: 2017-06-06 14:57:48
tags: [机器学习,sklearn]
categories: sklearn学习
---
用sklearn做机器学习的时候，一直都是用的时候再去网上找资料，结果每次都要重新找，很麻烦。终于下决心好好总结一下，平时经常用的一些sklearn的东西（或其他相关知识），希望能写成一个系列的笔记。
# 概要
这次就总结下sklearn数据预处理。主要是别人博客的知识，汇总一下。
sklearn是一个常用的机器学习库，其中的sklearn.preprocessing模块包含了常用的预处理函数，包括数据的清洗，如缺失值和零值的填充，数据标准化，二值化和哑编码等。
# 数据预处理
## 1.标准化（均值去除和按方差比例缩放）
将数据转化为均值为零，方差为一的数据，形如标准正态分布（高斯分布）。实际操作中，经常忽略特征数据的分布形状，移除每个特征均值，划分离散特征的标准差，从而等级化，进而实现数据中心化。在利用机器学习算法（例如SVM）的过程中，如果目标函数中的一个特征的方差的阶数的量级高于其他特征的方差，那么这一特征就会在目标函数中占主导地位，从而“淹没”其他特征的作用。
数据标准化的意义：
1. 消除量纲影响和变量自身变异大小和数值大小的影响。
2. 数据同趋化，主要解决不同性质数据问题，对不同性质指标直接加总不能正确反映不同作用力的综合结果
### Z-score标准化
基于原始数据的均值（mean）和标准差（standard deviation）进行数据的标准化。
公式为：(X-mean)/std  （mean为均值，std为标准差）
计算时对每个属性/每列分别进行。
将数据按期属性（按列进行）减去其均值，并处以其方差。得到的结果是，对于每个属性/每列来说所有数据都聚集在0附近，方差为1。
用sklearn实现有两种方式：
1.使用sklearn.preprocessing.scale()函数，可以直接将给定数据进行标准化。
```python
>>> from sklearn import preprocessing
>>> import numpy as np
>>> X = np.array([[ 1., -1.,  2.],
...               [ 2.,  0.,  0.],
...               [ 0.,  1., -1.]])
>>> X_scaled = preprocessing.scale(X)
 
>>> X_scaled                                          
array([[ 0.  ..., -1.22...,  1.33...],
       [ 1.22...,  0.  ..., -0.26...],
       [-1.22...,  1.22..., -1.06...]])
 
>>>#处理后数据的均值和方差
>>> X_scaled.mean(axis=0)
array([ 0.,  0.,  0.])#零均值
 
>>> X_scaled.std(axis=0)
array([ 1.,  1.,  1.])#单位方差
```
2.使用sklearn.preprocessing.StandardScaler类，使用该类的好处在于可以保存训练集中的参数（均值、方差）直接使用其对象转换测试集数据。
```python
>>> scaler = preprocessing.StandardScaler().fit(X)
>>> scaler
StandardScaler(copy=True, with_mean=True, with_std=True)
 
>>> scaler.mean_                                      
array([ 1. ...,  0. ...,  0.33...])
 
>>> scaler.std_                                       
array([ 0.81...,  0.81...,  1.24...])
 
>>> scaler.transform(X)                               
array([[ 0.  ..., -1.22...,  1.33...],
       [ 1.22...,  0.  ..., -0.26...],
       [-1.22...,  1.22..., -1.06...]])
 
 
>>>#可以直接使用训练集对测试集数据进行转换
>>> scaler.transform([[-1.,  1., 0.]])                
array([[-2.44...,  1.22..., -0.26...]])
```
### MinMax标准化(最小最大值标准化)
将数据缩放至给定的最小值与最大值之间，通常是０与１之间，可用MinMaxScaler实现。或者将最大的绝对值缩放至单位大小，可用MaxAbsScaler实现。
对原始数据的线性变换，使结果落到[0,1]区间，转换函数如下：
x ＝ (x - min)/(max - min)
max: 样本数据的最大值
min: 为样本数据的最小值
使用这种方法的目的包括：
1、对于方差非常小的属性可以增强其稳定性。
2、维持稀疏矩阵中为0的条目。
sklearn实现MinMax标准化：
```python
>>> X_train = np.array([[ 1., -1.,  2.],
...                     [ 2.,  0.,  0.],
...                     [ 0.,  1., -1.]])
...
>>> min_max_scaler = preprocessing.MinMaxScaler()
>>> X_train_minmax = min_max_scaler.fit_transform(X_train)
>>> X_train_minmax
array([[ 0.5       ,  0.        ,  1.        ],
       [ 1.        ,  0.5       ,  0.33333333],
       [ 0.        ,  1.        ,  0.        ]])
 
>>> #将相同的缩放应用到测试集数据中
>>> X_test = np.array([[ -3., -1.,  4.]])
>>> X_test_minmax = min_max_scaler.transform(X_test)
>>> X_test_minmax
array([[-1.5       ,  0.        ,  1.66666667]])
 
 
>>> #缩放因子等属性
>>> min_max_scaler.scale_                             
array([ 0.5       ,  0.5       ,  0.33...])
 
>>> min_max_scaler.min_                               
array([ 0.        ,  0.5       ,  0.33...])  
```
MinMaxScaler()默认的缩放范围是（0,1）。在构造类对象的时候也可以直接指定最大最小值的范围：feature_range=(min, max)
例如：min_max_scaler = preprocessing.MinMaxScaler(feature_range=(min, max))
### MaxAbsScaler（绝对值最大标准化）
与上述标准化方法相似，但是它通过除以最大值将训练集缩放至[-1,1]
```python
X_train = np.array([[ 1., -1.,  2.],  
                     [ 2.,  0.,  0.],  
                    [ 0.,  1., -1.]])  
max_abs_scaler = preprocessing.MaxAbsScaler()  
X_train_maxabs = max_abs_scaler.fit_transform(X_train)  
# doctest +NORMALIZE_WHITESPACE^, out: array([[ 0.5, -1.,  1. ], [ 1. , 0. ,  0. ],       [ 0. ,  1. , -0.5]])  
X_test = np.array([[ -3., -1.,  4.]])  
X_test_maxabs = max_abs_scaler.transform(X_test) #out: array([[-1.5, -1. ,  2. ]])  
max_abs_scaler.scale_  #out: array([ 2.,  1.,  2.])  
```
## 正则化（Normalization）规范化
这个本人理解的不是很透，先把别人的结论放上来。而且从找到的资料来看，这个翻译比较模糊。
文档上说：Normalization is the process of scaling individual samples to have unit norm. 
正则化的过程是将每个样本缩放到单位范数（每个样本的范数为1），如果后面要使用如二次型（点积）或者其它核方法计算两个样本之间的相似性这个方法会很有用。
将样本缩放成单位向量，标准化数据是针对特征来说的，而现在正则化是对样本来做的，是用样本数据除以他的范式。
sklearn实现使用preprocessing.normalize(x, norm = 'l1')方法，具体的参数说明详见sklearn文档
http://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.normalize.html#sklearn.preprocessing.normalize
1.使用preprocessing.normalize()函数对指定数据进行转换：
```python
>>> X = [[ 1., -1.,  2.],
...      [ 2.,  0.,  0.],
...      [ 0.,  1., -1.]]
>>> X_normalized = preprocessing.normalize(X, norm='l2')
 
>>> X_normalized                                      
array([[ 0.40..., -0.40...,  0.81...],
       [ 1.  ...,  0.  ...,  0.  ...],
       [ 0.  ...,  0.70..., -0.70...]])
```
2.使用processing.Normalizer()类实现对训练集和测试集的拟合和转换：
```python
>>> normalizer = preprocessing.Normalizer().fit(X)  # fit does nothing
>>> normalizer
Normalizer(copy=True, norm='l2')
 
>>>
>>> normalizer.transform(X)                            
array([[ 0.40..., -0.40...,  0.81...],
       [ 1.  ...,  0.  ...,  0.  ...],
       [ 0.  ...,  0.70..., -0.70...]])
 
>>> normalizer.transform([[-1.,  1., 0.]])             
array([[-0.70...,  0.70...,  0.  ...]])
```
对于l2 norm,变换后每个样本的各维特征的平方和为1。类似地，L1 norm则是变换后每个样本的各维特征的绝对值和为1。还有max norm，则是将每个样本的各维特征除以该样本各维特征的最大值.
## 二值化（Binarization）
将数值型数据转化为布尔型的二值数据，可以设置一个阈值（threshold）
在sklearn中，sklearn.preprocessing.Binarizer函数可以实现
```python
>>> X = [[ 1., -1.,  2.],
...      [ 2.,  0.,  0.],
...      [ 0.,  1., -1.]]

>>> binarizer = preprocessing.Binarizer().fit(X)  # fit does nothing
>>> binarizer
Binarizer(copy=True, threshold=0.0)

>>> binarizer.transform(X)
array([[ 1.,  0.,  1.],
       [ 1.,  0.,  0.],
       [ 0.,  1.,  0.]])
```
而且还可以调整阈值
```python
>>> binarizer = preprocessing.Binarizer(threshold=1.1)
>>> binarizer.transform(X)
array([[ 0.,  0.,  1.],
       [ 1.,  0.,  0.],
       [ 0.,  0.,  0.]])
```
## 标签预处理（Label preprocessing）

### 标签二值化（Label binarization）
LabelBinarizer通常用于通过一个多类标签（label）列表，创建一个label指示器矩阵
```python
>>> lb = preprocessing.LabelBinarizer()
>>> lb.fit([1, 2, 6, 4, 2])
LabelBinarizer(neg_label=0, pos_label=1)
>>> lb.classes_
array([1, 2, 4, 6])
>>> lb.transform([1, 6])
array([[1, 0, 0, 0],
       [0, 0, 0, 1]])
```
上例中每个实例中只有一个标签（label），LabelBinarizer也支持每个实例数据显示多个标签：
```python
>>> lb.fit_transform([(1, 2), (3,)]) #(1,2)实例中就包含两个label
array([[1, 1, 0],
       [0, 0, 1]])
>>> lb.classes_
array([1, 2, 3])
```
### 标签编码（Label encoding）
``` python
>>> from sklearn import preprocessing
>>> le = preprocessing.LabelEncoder()
>>> le.fit([1, 2, 2, 6])
LabelEncoder()
>>> le.classes_
array([1, 2, 6])
>>> le.transform([1, 1, 2, 6])
array([0, 0, 1, 2])
>>> le.inverse_transform([0, 0, 1, 2])
array([1, 1, 2, 6])
```
也可以用于非数值类型的标签到数值类型标签的转化：
```python
>>> le = preprocessing.LabelEncoder()
>>> le.fit(["paris", "paris", "tokyo", "amsterdam"])
LabelEncoder()
>>> list(le.classes_)
['amsterdam', 'paris', 'tokyo']
>>> le.transform(["tokyo", "tokyo", "paris"])
array([2, 2, 1])
>>> list(le.inverse_transform([2, 2, 1]))
['tokyo', 'tokyo', 'paris']
```
## 离散变量编码
例如性别有‘男’， ‘女’，然而计算机的许多模型都只能在数值型数据当中进行计算，如果我们简单的将‘男’为1，‘女’为0，虽然也可以完成转换，但是在转换的过程当中我们引入了大小关系，就是‘女’ < ‘男’，这会对后续模型应用造成不必要的困扰。 
解决方法为OneHotEncode，就是将其转化为二进制串，除了当前值所在位置为1，其他全部为0，如[0,0,1,0,0], [0,1,0,0,0].。性别可表示为男为[1,0]，女为[0,1]，这样一个性别特征就转化成了两个特征。
```python
df = pd.DataFrame({'pet': ['cat', 'dog', 'dog', 'fish'],'age': [4 , 6, 3, 3],
'salary':[4, 5, 1, 1]})
#例如有数据比较大，OneHotEncoder会生成非常多的特征，或者为字符串数据，先转化为数字，所以先用LabelEncoder处理。
label = preprocessing.LabelEncoder()
df['pet'] = label.fit_transform(df['pet'])
one_hot = preprocessing.OneHotEncoder(sparse = False)
print one_hot.fit_transform(df[['pet']])
```
```
before
   age  pet  salary
0    4    0       4
1    6    1       5
2    3    1       1
3    3    2       1
after
[[ 1.  0.  0.]
 [ 0.  1.  0.]
 [ 0.  1.  0.]
 [ 0.  0.  1.]]
```
## 缺失值处理
sklearn中的Imputer类提供了一些基本的方法来处理缺失值，如使用均值、中位值或者缺失值所在列中频繁出现的值来替换。
例如使用均值来处理的实例：
```python
>>> import numpy as np  
>>> from sklearn.preprocessing import Imputer  
>>> imp = Imputer(missing_values='NaN', strategy='mean', axis=0)  
>>> imp.fit([[1, 2], [np.nan, 3], [7, 6]])  
Imputer(axis=0, copy=True, missing_values='NaN', strategy='mean', verbose=0)  
>>> X = [[np.nan, 2], [6, np.nan], [7, 6]]  
>>> print(imp.transform(X))                             
[[ 4.          2.        ]  
 [ 6.          3.666...]  
 [ 7.          6.        ]]  
```
Imputer也支持稀疏矩阵作为输入：
```python
>>> import scipy.sparse as sp
>>> X = sp.csc_matrix([[1, 2], [0, 3], [7, 6]])
>>> imp = Imputer(missing_values=0, strategy='mean', axis=0)
>>> imp.fit(X)
Imputer(axis=0, copy=True, missing_values=0, strategy='mean', verbose=0)
>>> X_test = sp.csc_matrix([[0, 2], [6, 0], [7, 6]])
>>> print(imp.transform(X_test))                      
[[ 4.          2.        ]
 [ 6.          3.666...]
 [ 7.          6.        ]]
```
## 维度拓展
考虑复杂化非线性特征，就是生成多项式特征，例如(x1,x2)−>(x1,x2,x21,x1x2,x22)，会使特征数量增加
```python
poly = PolynomialFeatures(2)#参数为阶数
poly.fit_transform(X) 
'''
before 
[[0, 1],
 [2, 3],
 [4, 5]]
after
[[  1.,   0.,   1.,   0.,   0.,   1.],
 [  1.,   2.,   3.,   4.,   6.,   9.],
 [  1.,   4.,   5.,  16.,  20.,  25.]]
```
参考：
http://scikit-learn.org/stable/modules/preprocessing.html#preprocessing-normalization
http://blog.csdn.net/u010787640/article/details/60956164
http://blog.csdn.net/shmily_skx/article/details/52946414
http://www.cnblogs.com/chaosimple/p/4153167.html
http://blog.csdn.net/csmqq/article/details/51461696