viterbi & beam-seach
--------------------
01. 概述：  
viterbi： 基于dp实现的序列解码策略算法，最优路径由最优子路径转移而来  
beam_search： 基于贪心的的解码策略算法，每次预测的时候是选取概率最高的topk个路径
02. 比较：  
viterbi： 适合解空间较小的任务，例如序列标注  
beam_search： 适合解空间较大的任务，例如文本生成，机器翻译