# 美国运通信用卡逾期风险预测

## 写在前言
>> 该项目来源于202205美国运通在kaggle发布的信用卡风险逾期预测竞赛。该项目较为贴近Fintech实际业务内容，是对于从事金融科技工作和试图寻找该类工作的同学们一个很好的选择，大家感兴趣可以看看。<br>
>> 接下来，我会用较长的篇幅介绍该信用卡逾期预测竞赛，里面包含了我对该比赛的思考以及所学知识，大家若想进行更多深入地讨论可以添加我的微信： lhao29902978.

## 一、背景：
>>美国运通Kaggle举办信用卡逾期竞赛，周期3个月（202205-202208。<br>
>>附上比赛链接：https://www.kaggle.com/competitions/amex-default-prediction/overview <br>
>> * 数据描述：数据集为包含时间序列行为的客户匿名档案信息，目的在于用所提供数据集预测客户未来是否存在逾期风险，提高客户信用卡批准率以及现有持卡人更好的客户体验。<br>
>> * 数据量： 
>>>>>> * train：16.39GB，55w+样本，458913 独立客户ID，<br>
>>>>>> * test：33.82GB，150w+，924621 独立客户ID，<br>
>> 脱敏变量来源：拖欠、支出、付款、银行流水、风险变量，共190。<br>
>> * 参赛人数：6003，团队 4874。<br>

## 二、 数据探索与可视化
>>数据集过大，本地机器无法运行，需进行数据压缩，kaggle 论坛提供了诸多大佬压缩完毕的文件和代码，均可参考以及直接使用，该部分并非竞赛关注重点，感兴趣的同学可以研究相应论坛，<br>
附上链接：https://www.kaggle.com/competitions/amex-default-prediction/discussion/328054 <br>
### 数据分数据源统计：<br>
>> * 本次竞赛中可做：各数据源覆盖率，逾期率，时间分布等；<br>
>> * 结合实际业务可选做：除上述部分外，可对各数据源建模，查看单数据源 iv，ks。<br>
>> * 实际业务中除模型效果外还需考虑数据源适配度，价格等诸多问题，考察单数据源也是为了降低成本。（小厂谋生不易，拥有无限资金、时间与精力的大厂可忽略成本问题）<br>
### 数据分类型统计：<br>
>> 该部分也是Kaggle论坛中诸多大佬必做一项。为什么呢？既然为竞赛，则可以不考虑业务，以结果为导向进行编程。分类型处理数据可以更快的了解数据结构，以及做一个初步建模变量筛选，例如：删除部分缺失率较多的变量。对categorical 变量进行encoding等操作，总之，便是最快的让变量能够入模。<br>
>> 顺着大佬们思路，也是我的处理逻辑，大家可以对着以下的问题进行逐步解决：<br>
>>	* 哪些变量是基本三要素（name，id，applydate）？ 这些变量是绝对不要入模的嗷。找到，剔除！<br>
>>	* 哪些变量是categorical变量？找到做转换 or 删除。 模型看不懂字符型变量嗷。<br>
>>	* 哪些变量缺失率过高？缺失率过高的变量入模会导致什么问题？这些变量本身没啥意思，其次入模还增加不少noise。百害而无一利，不要不要！<br>
>>	* 那单一值率，零值率，iv值过低，psi过高？大家就仁者见仁智者见智，具体情况解决具体问题。特别是算iv和psi啊。至少本竞赛数据集，psi怕是得跑个一天哟。时间宝贵，全职打工人+简直竞赛人，钢铁侠看了都要流泪！<br>
### 数据分时间统计：<br>
>> 既然样本是包含时间序列的客户行为特征，大家难道就对于一个人短暂而又漫长的信用卡还款旅途不感兴趣么，IF感兴趣：咱就抽出一个人（customer_id）看看他在样本中的xin还suan款sheng历huo程。Else：也得看。<br>
>> 看了一个人，要不看看全部？ groupby starting …… <br>


## 三、 变量衍生与特征工程
>> 经过一番数据探索性分析，我想大家对该样本有了一定的理解，那么我就可以进行变量衍生了。俺是百分百相信就这自带变量绝不可能有一个好的结果，不然比赛就变成一个调参比赛了。最终胜者可能不一定是大神，但一定是最幸运的人。<br>
>> 
>> 正常的变量衍生，我们一定是根据变量的实际含义进行衍生，或加减乘除，或做比率同比环比，或做数学公式 收入-成本=利润，但我们实际拿到的是脱敏变量，那就得发挥想象力，只要想得多，变量就衍生的多。模型好不好不知道，但肯定差不到哪去。<br>
### 变量衍生：
>> First，我们从容易思考的角度做衍生（描述性统计），对于两种变量，分别做：<br>
>> * Categorical features: mean, std, sum, last <br>
>> * Numerical features: mean, std, sum, min, max, last.<br>
>> 
>> Second, 我们可以对临近变量做diff工作，也就是记录临近变量的差值，具体做法就是 dataframe - （dataframe 向下平移一层）。<br>
>> 
>> Third，keep considering …… <br>
>> 变量中‘S_2’记录了时间，针对时间又可进行拆分成年月日周，针对不同的时间我们可以统计某一个时间区间内的 mean，std，sum 等，再进行 diff工作。（在比赛中我们可尽可能多的联想衍生特征，最终特征能不能用只有入模看效果才会知道）<br>
>> Attention：在实际业务建模中，慎重使用时间列直接入模，该变量可能导致模型效果在train中虚高，也就是对未来未知数据集的泛化能力不足。<br>
### 特征工程：
>> * Categorical features 各种处理方式，最常使用的则是encoding；<br>
>> * 缺失变量的处理，对于缺失率超过95%，选择直接剔除；缺失率小于 5%的，选择mean/median 填充，其余全部填为0，当然还有别的处理方式。在该竞赛中，我觉得此处的处理影响不大。具体实际业务则应具体情况具体考虑，当然本人经历业务中，一般对于缺失率99%+才会剔除，（小厂数据囊中羞涩，将就用~）。<br>
>> * 异常值处理：本人在竞赛中并没有处理异常值，看 kaggle论坛中的大神们也并没有处理，赛后重新思考，觉得是不是可以针对每一个customer id 剔除掉异常样本呢。大家仁者见仁智者见智，敢想敢拼。<br>

## 四、模型选择与训练
>> 对于金融类数据分类预测模型，可选 LR，XGB, Lightgbm, catboost, DNN。对于本样本而言，LR可以排除，样本量极大，线性关系可能性较小，LR大概率不会得到一个好的模型。（此处，博主在唠叨一句，实际业务中，一些银行更倾向 LR，除了LR适合建评分卡之外，LR模型可解释性较强，在金融领域，模型可解释性所占比重较高。）<br>
>> 
>> 比赛周期较长，所以大家在前期可以使用单个不同模型进行建模，其实只要特征衍生做得好，建模则相对容易。<br>
>>>> 模型搭建的模板大体一样，设定模型参数，搭建 CVfolds，输出/保存模型，预测分数，检验分数，查看特征重要性，筛选特征，优化模型 ……,博主这里就举一个 lightgbm的例子，也是我诸多单个模型中表现最好的模型。<br>
>>>> 
>> 首先是参数选择：以下为博主建模选择参数，（其实在该竞赛中特征衍生和模型融合做得好，参数全部默认都可以）<br>
>>>   params = {<br>
>>>>         'objective': 'binary',<br>
>>>>        'metric': 'binary_logloss',<br>
>>>>         'boosting':'dart',<br>
>>>>         'seed': 42,<br>
>>>>         'num_leaves': 100,<br>
>>>>         'learning_rate': 0.01,<br>
>>>>         'feature_fraction': 0.20,<br>
>>>>         'bagging_fraction': 0.50,<br>
>>>>         'max_depth': -1, <br>
>>>>         'bagging_freq': 10,<br>
>>>>         'n_jobs': -1,<br>
>>>>         'lambda_l1': 0.1,<br>
>>>>         'lambda_l2': 2,<br>
>>>>         'min_data_in_leaf': 40<br>
>>>>        }<br>
>>以上参数，也是从论坛中浏览大佬分享的参数总结出来的。最有意思的是 申请的42，seed玄学设置为42总比别的好，原因嘛，玄学，好像还有论文专门总结过这个，此处不是我们研究的重点，感兴趣的小伙伴可以google scholar。<br>
>>调参并不是我们主要内容，大家随便看看即可。此处也粘贴该竞赛排名第一的大佬参数，（感谢大佬无私分享，大佬具体代码链接也会在附录中粘贴，供感兴趣小伙伴浏览）。<br>
>>> 'lgb_params':{ 'objective' : 'binary', 'metric' : 'binary_logloss', 'boosting': 'dart', 'max_depth' : -1, 'num_leaves' : 64, 'learning_rate' : 0.035, 'bagging_freq': 5, 'bagging_fraction' : 0.75, 'feature_fraction' : 0.05, 'min_data_in_leaf': 256, 'max_bin': 63, 'min_data_in_bin': 256, #'min_sum_heassian_in_leaf': 10, 'tree_learner': 'serial', 'boost_from_average': 'false', 'lambda_l1' : 0.1, 'lambda_l2' : 30, 'num_threads': 24, 'verbosity' : 1, }, <br>
>>> 
>>数据集的庞大，也不适合大家在参数上花费较多的时间去调整，但lgbm还是有诸多参数调节的小技巧。例如 boosting选择 gbdt 和dart（带dropout的gbdt）最后得出来的分数差异还比较大; >>learning_rate选择也比较重要，一般大家选择 < 0.05即可，num_leaves 控制树上叶子数，默认31，max_depth 控制树深，此处可选择默认值-1，即无限制。还有诸多注意事项，若之后做小数据集>>的分类预测，可以多做做lgbm调参，再给大家具体分享。（该数据集已经耗费我100+租服务器费用了！！赚钱不易）。<br>

>>模型交叉验证也是大数据集模型搭建的重点，sklearn 则提供了 Kfold，StratifiedKfold，GroupKfold诸多选择，此处我们选择StratifiedKfold，因为该函数尽可能保证划分后数据集与原始数据集近似。<br>
>>最后即可以开始 lgb.train，model.save_model，model.predict。当开始预测 test数据集时候，我们可以选择最优模型直接预测也可以对不同fold预测结果求均值。这样一个完整的lgb模型搭建完毕。<br>
>>Xgb，catgboost等别的模型则万变不离其宗，皆可举一反三。<br>

## 五、融合模型，提升预测分数与模型鲁棒性
>> 本着一切皆可融合的理念，接下来则是对不同模型的融合，融合过程也简单，则是对不同模型预测分数进行函数式汇总，具体如 Ax1 + Bx2 + Cx3 = score。
单个模型可能实力有限，多个模型分的融合能够给准确率带来一个飞跃。当然我们也应遵循奥卡姆剃刀原理，“在其他一切同等的情况下，较简单的解释比复杂的好”。如果强行融合 n+模型，最终模型可能陷入过拟合，得不偿失。



## 六、总结
>> 美国运通的信用卡逾期风险预测，我们可以看成是一个贷前或贷中的风险评估。该竞赛和Fintech实际业务比较贴近，对于金融科技的同学绝对是受益匪浅。在比赛中，我们可以着重强化训练变量衍生和数据挖掘的思维逻辑和代码能力，同时也对于我们在做模型有一定的帮助。论坛中，各类大佬的分享交流对于我们来说都是一次思维的碰撞，我们会惊叹于大佬们层出不穷的想法，受益匪浅。比赛的结果不是重点，赛后的复盘和学习同样重要，期待未来能够再次参与同类竞赛，也期待能够再次观赏大佬层出不穷的想法。

## 七、实际业务的反思
>>尽管比赛让我们受益匪浅，但并不可以完全复刻于实际业务。实际业务去开发一个模型或是数据产品，除了考虑变量与模型的质量，还应考虑开发周期与成本。事实上，我们在业务中做数据衍生，除了基础的描述性统计衍生外，还应较多的考虑变量实际关系的组合和衍生。在模型搭建中，我们还应考虑使用场景，如银行数据，还是各种现金贷、消费贷等，我们所面临的客户主题是谁？我们应保证模型的泛化能力。在我实际业务处理中，一个xgb模型，并不会过多的敢于 plenty，正则化等参数，这类参数容易导致模型不稳定，对未来数据集可能会导致效果极速下降。对于模型结果的评估，该竞赛使用系统提供的评估方式，实际业务会去关注 KS值，同时辅助性浏览模型十分组图，ROP，ROC等。以上讲给还在读书的同学们听，给未入职场的朋友们闲聊一些业务与比赛的区别。
>>
>>以上分享到此结束，若您对分享内容有别的见解或想要交流分类模型可随时与我联系或发送至我的邮箱 hao425871387@163.com，也可添加我的微信lhao29902978.

## 附录
>> 该竞赛优秀作品：<br>
https://www.kaggle.com/competitions/amex-default-prediction/discussion/348961  








