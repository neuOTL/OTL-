# 在线迁移学习实现
## 1.在线迁移学习概述
 &emsp;&emsp;在线迁移学习（OTL），是一种在在线环境下将离线状态下学习到的模型利用迁移学习的技术与在线模型相融合的一种新式的机器学习方法。其有两个基本假设：（i）目标域的训练数据是以序列模式到来的 (ii) 已经有一些模型在源域上训练完成.

 &emsp;&emsp;OTL可以解决在线环境下的数据漂移问题（简单的说就是数据分布随时间发生变化），这是单纯利用迁移学习所无法解决的。同时OTL也比直接使用在线学习效果要更高，因为OTL利用了离线模型学习到的知识。所以，OTL是一种在特定情境下应用的兼具在线学习与迁移学习优点的方法。首次提出在线迁移学习算法的是Peilin Zhao和Steven Hoi，Peilin Zhao在阿里巴巴做研究，Steven Hoi是新加坡管理大学(SMU)的副教授。以下是论文连接：

[《Online Transfer Learning》](https://ac.els-cdn.com/S0004370214000800/1-s2.0-S0004370214000800-main.pdf?_tid=66004ea4-e571-49a5-9ca5-d9f34e33359d&acdnat=1552293438_54e27b877ccfe21927468e2d7f931966)

 &emsp;&emsp;这篇论文针围绕对两种不同的情况分别提出了有针对性的OTL算法。第一种情况是源域（离线模型数据的分布域）和目标域（在线数据的分布域）是同质的（homogeneous），即我们认为源域和目标域的特征空间相同；第二种情况是源域和目标域是异质的（heterogeneous），即目标域和源域的特征空间不同。很明显，第二种情况下学习更加困难。


 &emsp;&emsp;由于我的试验项目的背景主要是围绕同质域学习，所以我只实现了该论文中的同质域OTL算法。如果我还有时间的话我会把异质域的算法再实现出来

## 2. 同质域OTL算法
 &emsp;&emsp;论文中提出了两种同质域OTL算法，分别是HomOTL-I和HomOTL-II。这两种算法的核心思想是相同的，只不过看问题的角度不一样，HomOTL-I是从损失的角度来更新模型，而HomOTL-II是从错误率来更新模型。该算法的基本表示如下所示：


![同质域OTL算法](https://github.com/neuOTL/OTL-/blob/master/pictures/Hom_OTL1.png)

 &emsp;&emsp;该算法的核心思想来自于集成学习（ensemble learning）,OTL的预测结果由一个在线学习模型（比如PA算法）得到的结果，和一个离线学习模型（比如svm）的预测结果，加权平均得到的。OTL算法最核心的地方就是该如何根据线上预测时到来的实例序列来跟新这两个模型的组合权重。为此作者精心设计了一套有效的更新方法，解决了“协变量转移”的挑战，并给出算法收敛性的数学证明。

 论文中提出的算法是用来解决线性分类问题的，而我在实验中实现的是非线性回归问题的OTL算法，其主要变化是：

1. 将原始算法的损失函数从hinge损失变为 ε-不敏感损失。并且删除了源算法中的归一化部分（我统一在OTL使用之前归一化）

![svr损失函数](https://github.com/neuOTL/OTL-/blob/master/pictures/SVR%E6%8D%9F%E5%A4%B1%E5%87%BD%E6%95%B0.png)

2. 对于在线学习算法PA,我将线性多项式扩展为了径向基核多项式，使其具有非线性映射的能力。


3. 由于非线性PA算法每次预测时如果产生损失都要保存支持向量，随着支持向量的数量的增加，其模型会变的越来越复杂和庞大，为了避免支持向量无限的增加下去，我采用了固定缓冲器的核在线学习方法，如果支持向量的数量超过阈值则会随机选择一个支持向量剔除出支持向量集合。

## 3. 实验
 &emsp;&emsp;实验的数据集来自为[household_power_consumption_days](https://archive.ics.uci.edu/ml/datasets/individual+household+electric+power+consumption)。该数据集是一个多变量时间序列数据集，用于描述单个家庭四年的用电量。该数据是在2006年12月至2010年11月之间收集的，并且每分钟收集家庭内的能耗观察结果。

它是一个多变量系列，由七个变量组成（除日期和时间外）; 他们是：

（1） global_active_power：家庭消耗的总有功功率（千瓦）。

（2）global_reactive_power：家庭消耗的总无功功率（千瓦）。

（3）voltage：平均电压（伏特）。

（4）global_intensity：平均电流强度（安培）。

（5）sub_metering_1：厨房的有功电能（瓦特小时的有功电能）。

（6）sub_metering_2：用于洗衣的有功能量（瓦特小时的有功电能）。

（7）sub_metering_3：气候控制系统的有功电能（瓦特小时的有功电能）

 &emsp;&emsp;实验结果采用回归问题常用的评价指标：平均平方误差（MSE）和平均绝对值误差（MAE）来衡量预测结果，预测结果如下所示:其中图1是PA预测值，svm预测值和OTL预测值与实际值的曲线变化趋势；图2，3是svm和OTL在每一轮训练时累计MSE和MAE误差曲线；图4，5是PA，svm和OTL在每一轮训练时累计MSE和MAE误差曲线。

![各时刻训练结果比较（归一化后）](https://github.com/neuOTL/OTL-/blob/master/pictures/akk.png)
 &emsp;&emsp;&emsp; &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;图1： 各时刻训练结果比较（归一化后）

![SVR和OTL算法的MSE变化趋势](https://github.com/neuOTL/OTL-/blob/master/pictures/mse2.png)
  &emsp;&emsp;&emsp;  &emsp;&emsp;&emsp;      &emsp;&emsp;&emsp;        图2：SVR和OTL算法的MSE变化趋势

![SVR和OTL算法的MAE变化趋势](https://github.com/neuOTL/OTL-/blob/master/pictures/mae2.png)
   &emsp;&emsp;&emsp;   &emsp;&emsp;&emsp;     &emsp;&emsp;&emsp;      图3：SVR和OTL算法的MAE变化趋势

![PA,SVR和OTL算法的MSE变化趋势](https://github.com/neuOTL/OTL-/blob/master/pictures/mse3.png)
   &emsp;&emsp;&emsp;    &emsp;&emsp;&emsp;    &emsp;&emsp;&emsp;      图4：PA,SVR和OTL算法的MSE变化趋势

![PA,SVR和OTL算法的MAE变化趋势](https://github.com/neuOTL/OTL-/blob/master/pictures/mse4.png)
     &emsp;&emsp;&emsp;        &emsp;&emsp;&emsp;   &emsp;&emsp;&emsp;  图5：PA,SVR和OTL算法的MAE变化趋势
     
## 4.实验结论
 &emsp;&emsp;可以看出，OTL算法在200轮之后的误差已经低于本地训练的svr模型，说明OTL算法是优于PA算法和svr算法的简单组合的，这体现了集成学习的威力，即若干个弱分类器组合在一起的表现要更好。PA算法由于初始是从零开始学习，所以需要一段时间才能收敛，如果数据集发生了很严重的数据漂移的话，PA算法的表现应该会强于离线学习的svr。无论如何，OTL算法的预测误差上界是在线和离线模型中误差最小的那个。


## 5.异构域与数据漂移OTL
 &emsp;&emsp;作者在论文的第二部分分别针对异构域与数据漂移的情况提出了3个算法，其中HetOTL是针对异构域情况的，CDOL和OWA是针对数据漂移情况的。

### 5.1 HetOTL算法，即异构算法的特点：

（1） 数据的目标域由两部分组成，一部分是和源域一样的特征，一部分是其特有的特征，相应的，我们的模型也分成两个部分，一个模型对应于源域的特征，一个模型对应于目标域新出现的特征。

（2） 源域分类器初始化为离线模型，目标域分类器初始化为零

（3） 源域分类器和目标分类器要同时进行更新，所以说源域的分类器不能随意的挑选了

（4） 在HetOTL算法中，如果要讲其从分类问题扩展到回归问题的话需要把Proposistion2 的推导换成回归问题的损失函数再推导一遍，两个模型的更新的形式应该和分类问题不同。

![好好hemtol](https://github.com/neuOTL/OTL-/blob/master/pictures/HetOTL%E7%AE%97%E6%B3%95.png)

### 5.2 CDOL算法,即进一步针对数据飘移提出的OTL算法：
 &emsp;&emsp;CDOL算法的基本观点是：如果数据漂移十分严重，说明源域的模型已经不再适合继续预测了，所以我们需要动态的改变源域模型。CDOL算法其实是Hom-OTL算法的改进，与之相比只是增加了一个窗口更新策略。该算法的思路是讲训练过程换分为各个时期即窗口，每一个窗口训练一个在线的模型，在这个窗口结束之后，该算法会从目标域和源域这两个模型中挑选表现最好的模型赋给源域模型，同时将目标与的模型置为零，在下一个周期（窗口）中重新训练一个新的目标域模型。
。
![经济CDOL](https://github.com/neuOTL/OTL-/blob/master/pictures/CDOL%E7%AE%97%E6%B3%95.png)

### 5.3 OWA算法，即增加了可动态更新窗口大小的算法的特点
 &emsp;&emsp;OWA算法是CDOL算法的改进。其主要思想是：每次到到达窗口更新时机的时候，计算错误率，如果源域分类器的错误率大于目标域错误率，则跟换新的窗口周期i，将窗口大小置为初始值P，如果反之，则不更新窗口周期i，继续增大窗口的大小（通过增大窗口大小使在线模型学习到更多的信息，使其准确率增加）。

![看看OWA](https://github.com/neuOTL/OTL-/blob/master/pictures/OWA%E7%AE%97%E6%B3%95.png)
