---
title: sklearn PCA降维
date: 2017-06-01 18:01:31
tags: [机器学习,sklearn]
categories: sklearn学习
---
对于太多的特征，一般需要进行降维处理。PCA是最常用的降维的方法，sklearn提供了PCA降维的方法。
# 函数原型
```
class sklearn.decomposition.PCA(n_components=None, copy=True, whiten=False, svd_solver='auto', tol=0.0, iterated_power='auto', random_state=None)
```
<!--more-->
# 主要参数
1. n_components:
意义：PCA算法中所要保留的主成分个数n，也即保留下来的特征个数n
类型：int 或者 string，缺省时默认为None，所有成分被保留。
	赋值为int，比如n_components=1，将把原始数据降到一个维度。
	赋值为string，比如n_components='mle'，将自动选取特征个数n，使得满足所要求的方差百分比。
2. copy:
类型：bool，True或者False，缺省时默认为True。
意义：表示是否在运行算法时，将原始训练数据复制一份。若为True，则运行PCA算法后，原始训练数据的值不            会有任何改变，因为是在原始数据的副本上进行运算；若为False，则运行PCA算法后，原始训练数据的              值会改，因为是在原始数据上进行降维计算。
3. whiten:
类型：bool，缺省时默认为False
意义：白化，使得每个特征具有相同的方差。

# 属性
和参数差不多，参考[http://scikit-learn.org/stable/modules/generated/sklearn.decomposition.PCA.html](http://scikit-learn.org/stable/modules/generated/sklearn.decomposition.PCA.html)
# 方法
fit(X[, y]) —— Fit the model with X.
fit_transform(X[, y])—— Fit the model with X and apply the dimensionality reduction on X.
get_covariance() —— Compute data covariance with the generative model.
get_params([deep]) —— Get parameters for this estimator.
get_precision() —— Compute data precision matrix with the generative model.
inverse_transform(X) —— Transform data back to its original space, i.e.,
score(X[, y]) —— Return the average log-likelihood of all samples
score_samples(X) —— Return the log-likelihood of each sample
set_params(**params) —— Set the parameters of this estimator.
transform(X) —— Apply the dimensionality reduction on X.
详细说明：
1. fit(X,y=None)
fit()可以说是scikit-learn中通用的方法，每个需要训练的算法都会有fit()方法，它其实就是算法中的“训练”这一步骤。因为PCA是无监督学习算法，此处y自然等于None。
2. fit(X)，表示用数据X来训练PCA模型。
函数返回值：调用fit方法的对象本身。比如pca.fit(X)，表示用X对pca这个对象进行训练。
3. fit_transform(X)
用X来训练PCA模型，同时返回降维后的数据。
newX=pca.fit_transform(X)，newX就是降维后的数据。
4. inverse_transform()
将降维后的数据转换成原始数据，X=pca.inverse_transform(newX)
5. transform(X)
将数据X转换成降维后的数据。当模型训练好后，对于新输入的数据，都可以用transform方法来降维。
此外，还有get_covariance()、get_precision()、get_params(deep=True)、score(X, y=None)等方法，参考上面的英文。
# Example
以一组二维的数据data为例，data如下，一共12个样本（x,y），其实就是分布在直线y=x上的点，并且聚集在x=1、2、3、4上，各3个。
```python
>>> data  
array([[ 1.  ,  1.  ],  
       [ 0.9 ,  0.95],  
       [ 1.01,  1.03],  
       [ 2.  ,  2.  ],  
       [ 2.03,  2.06],  
       [ 1.98,  1.89],  
       [ 3.  ,  3.  ],  
       [ 3.03,  3.05],  
       [ 2.89,  3.1 ],  
       [ 4.  ,  4.  ],  
       [ 4.06,  4.02],  
       [ 3.97,  4.01]])  
```
data这组数据，有两个特征，因为两个特征是近似相等的，所以用一个特征就能表示了，即可以降到一维。下面就来看看怎么用sklearn中的PCA算法包。
（1）n_components设置为1，copy默认为True，可以看到原始数据data并未改变，newData是一维的，并且明显地将原始数据分成了四类。
```python
>>> from sklearn.decomposition import PCA   
>>> pca=PCA(n_components=1)  
>>> newData=pca.fit_transform(data)  
>>> newData  
array([[-2.12015916],  
       [-2.22617682],  
       [-2.09185561],  
       [-0.70594692],  
       [-0.64227841],  
       [-0.79795758],  
       [ 0.70826533],  
       [ 0.76485312],  
       [ 0.70139695],  
       [ 2.12247757],  
       [ 2.17900746],  
       [ 2.10837406]])  
>>> data  
array([[ 1.  ,  1.  ],  
       [ 0.9 ,  0.95],  
       [ 1.01,  1.03],  
       [ 2.  ,  2.  ],  
       [ 2.03,  2.06],  
       [ 1.98,  1.89],  
       [ 3.  ,  3.  ],  
       [ 3.03,  3.05],  
       [ 2.89,  3.1 ],  
       [ 4.  ,  4.  ],  
       [ 4.06,  4.02],  
       [ 3.97,  4.01]])  
```
（2）将copy设置为False，原始数据data将发生改变。
```python
>>> pca=PCA(n_components=1,copy=False)  
>>> newData=pca.fit_transform(data)  
>>> data  
array([[-1.48916667, -1.50916667],  
       [-1.58916667, -1.55916667],  
       [-1.47916667, -1.47916667],  
       [-0.48916667, -0.50916667],  
       [-0.45916667, -0.44916667],  
       [-0.50916667, -0.61916667],  
       [ 0.51083333,  0.49083333],  
       [ 0.54083333,  0.54083333],  
       [ 0.40083333,  0.59083333],  
       [ 1.51083333,  1.49083333],  
       [ 1.57083333,  1.51083333],  
       [ 1.48083333,  1.50083333]])  
```
（3）n_components设置为'mle'，看看效果，自动降到了1维。
```python
>>> pca=PCA(n_components='mle')  
>>> newData=pca.fit_transform(data)  
>>> newData  
array([[-2.12015916],  
       [-2.22617682],  
       [-2.09185561],  
       [-0.70594692],  
       [-0.64227841],  
       [-0.79795758],  
       [ 0.70826533],  
       [ 0.76485312],  
       [ 0.70139695],  
       [ 2.12247757],  
       [ 2.17900746],  
       [ 2.10837406]])  
```
（4）对象的属性值
```python
>>> pca.n_components  
1  
>>> pca.explained_variance_ratio_  
array([ 0.99910873])  
>>> pca.explained_variance_  
array([ 2.55427003])  
>>> pca.get_params  
<bound method PCA.get_params of PCA(copy=True, n_components=1, whiten=False)>  
```
我们所训练的pca对象的n_components值为1，即保留1个特征，该特征的方差为2.55427003，占所有特征的方差百分比为0.99910873，意味着几乎保留了所有的信息。get_params返回各个参数的值。
（5）对象的方法
```python
>>> newA=pca.transform(A)
```  
对新的数据A，用已训练好的pca模型进行降维。
```python
>>> pca.set_params(copy=False)  
PCA(copy=False, n_components=1, whiten=False) 
```
设置参数。
# 示例
在一个论坛上看到的例子
```python
#导入数值计算库
import numpy as np
#导入科学计算库
import pandas as pd
#导入数据预处理库
from sklearn.preprocessing import StandardScaler
#导入PCA算法库
from sklearn.decomposition import PCA

#读取贷款状态数据从创建名为LoanStats3a的数据表
LoanStats3a=pd.DataFrame(pd.read_csv('LoanStats3a.csv'))
 
#查看数据表内容
LoanStats3a.head()

#删除包含空值的特征
LoanStats3a=LoanStats3a.dropna()

#设置特征表X
#将贷款数据表中的贷款特征数据单独提取出来，用于后面的降维操作。
X = np.array(LoanStats3a[['loan_amnt', 'funded_amnt_inv', 'installment',
'annual_inc', 'dti', 'delinq_2yrs', 'inq_last_6mths', 'open_acc',
'pub_rec', 'revol_bal', 'total_acc', 'out_prncp', 'out_prncp_inv',
'total_pymnt', 'total_pymnt_inv', 'total_rec_prncp', 'total_rec_int',
'total_rec_late_fee', 'recoveries', 'collection_recovery_fee',
'last_pymnt_amnt']])

#对特征数据进行标准化处理,去除不同数据的单位限制，将它们转化为无量纲的纯数值。
sc = StandardScaler()
X_std = sc.fit_transform(X)

#创建PCA对象，n_components=3
pca = decomposition.PCA(n_components=3)

#使用PCA对特征进行降维
X_std_pca = pca.fit_transform(X_std)

#下面的写法与上面相同,下面进行了白化变换，数据还原之后进行了方差的归一化
pca=PCA(n_components=6,whiten=True)
pca.fit(X_std)
newData=pca.transform(X_std)
X=pca.inverse_transform(newData)

```
本文参考：http://blog.csdn.net/u012162613/article/details/42192293
		http://www.aboutyun.com/thread-21655-1-1.html