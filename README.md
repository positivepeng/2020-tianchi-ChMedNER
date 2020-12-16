# 2020-tianchi-ChMedNER
[2020 “万创杯”中医药天池大数据竞赛——中药说明书实体识别挑战](https://tianchi.aliyun.com/competition/entrance/531824/introduction)  复盘  初赛:17 复赛死于docker(泪)

参赛过程中主要参考的链接：

 - [一等奖团队分享 | 2019 CCF BDCI 《互联网金融新实体发现》](https://mp.weixin.qq.com/s/SgkQB7t0j2_kqHeotspHBQ)
 - [2019BDCI互联网金融实体识别（单模第一，综合第二名思路与代码框架傻瓜式分享）](https://zhuanlan.zhihu.com/p/100884995)
 - [“达观杯”冠军分享：预训练模型彻底改变了NLP，但也不能忽略传统方法带来的提升](https://zhuanlan.zhihu.com/p/84717061)
 - [2019BDCI互联网金融实体识别复赛A榜第七，B榜第五](https://github.com/light8lee/2019-BDCI-FinancialEntityDiscovery)

## 赛后前排分享总结：
#### 1. 数据预处理
#### 2. 模型设计
#### 3. 模型融合


### 小贤贤nice团队_亚军比赛攻略[ 原文链接](https://tianchi.aliyun.com/forum/postDetail?spm=5176.12586969.1002.6.25a33e5b7SqxIw&postId=154948)
方案总结：
1. 数据去噪:
&nbsp;&nbsp;&nbsp;&nbsp;本赛题数据集噪声较大，使用Roberta-CRF-FGM模型对训练集进行10折交叉验证对标注实体进行过滤，具体过滤规则为：10折训练得到10份预测结果，如果在10份预测结果中都有则保留，如果一个也没有但是原始训练集中标注了则去除。
2. 模型：
&nbsp;&nbsp;&nbsp;&nbsp;Roberta-CRF，BERT和CRF设置不同学习率，BERT的输出采用最后三层加权融合，权重为可学习参数。
3. 改进策略：
&nbsp;&nbsp;&nbsp;&nbsp;提高泛化，引入对抗训练，在BERT的word.embedding层增加扰动，对抗训练的方法有FGM、PGD、FreeLB等
&nbsp;&nbsp;&nbsp;&nbsp;在Bert和CRF中间加入BILSTM / IDCNN等，把Bert学习到的token向量作为BILSTM / IDCNN的输入，让模型进一步学习特征编码。BILSTM作为序列模型，推断时间会较慢，IDCNN预测速度更快，但实际使用时，IDCNN在长文本的表现不如BiLSTM
&nbsp;&nbsp;&nbsp;&nbsp;采用更大的模型，把RoBEATa_base替换成RoBERTa_large
&nbsp;&nbsp;&nbsp;&nbsp;引入词汇信息，BERT是基于字符，对于NER任务来说，基于字符能避免分词错误带来的不可逆误差，但专用领域的词典也有十分重要的作用，如Lattice LSTM、Soft-Lexicon、FLAT等都是考虑融合字和词信息的模型
4. 模型融合策略：
&nbsp;&nbsp;&nbsp;&nbsp;融合方法一：对不同模型的Bert输出层概率、CRF的转移概率进行权重平均
&nbsp;&nbsp;&nbsp;&nbsp;融合方法二：在每一折中，保留 f1值最高的模型，在输出的单元标签中，进行服从多数投票，如10折的预测结果，其中7折预测为B-eff、其它三折预测为O，则取B-eff
&nbsp;&nbsp;&nbsp;&nbsp;融合方法三（最优）：在每一折中，保留 recall最高的模型，在输出的实体标签中，进行阈值保留，如10折的预测结果，其中7折预测出“脾虚”这一实体，另外3折没有预测出来，假如阈值取≥7，则保留，否则丢弃
5. 经验总结：
&nbsp;&nbsp;&nbsp;&nbsp;在本地调试的时候，一定要划分好训练集、验证集和测试集，把实验记录的数据做好记录，提高效率（建议新手应该先搭建pipeline，记得设置随机种子！不然连划分数据集都难以复现）
&nbsp;&nbsp;&nbsp;&nbsp;多积累，多看前沿论文和解决方法，合理安排时间。

### 亚军比赛攻略[ 原文链接](https://tianchi.aliyun.com/forum/postDetail?spm=5176.12586969.1002.12.25a33e5b7SqxIw&postId=154826)
方案总结：
1. 实体补全：匹配相同的文本，补上标注
2. 特征挖掘：发现实体漏标比例和样本长度正相关
3. 模型融合：10折基于置信度阈值调整的投票策略

### NewBee季军比赛攻略[ 原文链接](https://tianchi.aliyun.com/forum/postDetail?spm=5176.12586969.1002.3.25a33e5b7SqxIw&postId=155024)

方案总结：
1. 数据预处理：长句子切分，生成伪标签数据增强
2. 模型构建：{BERT|Roberta|NEZHA}-{BiLSTM}-{CRF|SPAN}，BERT多层输出动态融合，标签平滑，对抗训练
3. 模型融合与后处理：舍弃过长的实体，舍弃带特殊字符的实体，融合的实体进行分割
4. 模型融合：单模融合：SWA，多模融合：对在训练集中出现过的和未出现过的分开处理。

### Theanswer-比赛攻略[原文链接](https://tianchi.aliyun.com/forum/postDetail?spm=5176.12586969.1002.9.25a33e5bTKPiNR&postId=154762)
方案总结：
1. 数据预处理：长度分割，Lattice词表：加入专业词汇
2. 模型构建，Roberta-{Flat-Lattice-layer}-{Softmax|CRF|Span}，对预训练模型进行再训练