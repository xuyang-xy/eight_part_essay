Residual-Network
----------------
01. 概述：输入和输入的非线性变化的叠加
```
    x1+1 = xl + F(xl，w)
```
02. 动机：  
残差网络是为了解决深度学习中隐藏层过多时的网络退化问题而提出。  退化（degradation）问题是指：当网络隐藏层变多时，网络的准确度达到饱和然后急剧退化，而且这个退化不是由于过拟合引起的  
过拟合： 训练集loss下降， 测试集loss上升  
退化：   训练集与测试及上loss同时上升  
ResNet的初衷，就是让网络拥有这种恒等映射的能力，能够在加深网络的时候，至少能保证深层网络的表现至少和浅层网络持平
03. 
备注：  
Deep residual network CVPR 2016