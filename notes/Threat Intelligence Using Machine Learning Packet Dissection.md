# Threat Intelligence Using Machine Learning Packet Dissection

> 基于机器学习的包分析威胁情报

## 1. 背景

### （1）期刊/会议级别

20th International Conference on Security & Management (SAM'21), 26 - 29 July 2021, Las Vegas, USA 

CCF None

### （2）作者信息

（英国）伦敦城市大学网络安全研究中心：Viktor Sowinski-Mydlarz，Karim Ouazzane， Vassil Vassilev

（英国）克兰菲尔德大学计算工程科学中心：Jun Li

谷歌学术找不到几位作者的信息，在Aminer上找到Karim Ouazzane的信息如下：

![image-20210524083537313](Threat Intelligence Using Machine Learning Packet Dissection.assets/image-20210524083537313.png)

### （3）文章背景

因特网上的每一个活动都会使用流量包进行通信，每个流量包包含以下信息：

* IP地址（源IP地址，目的IP地址）
* 使用某种网络协议

恶意软件可以在操作系统级别隐藏其活动(rootkits)，但它通常会在网络活动上留下痕迹，无论它是否加密。

现有的入侵检测系统无法解决网络威胁的复杂性和适应性问题。自适应方法的机器学习(ML)提供更好的检测率。它们还能降低误报率，节省处理和通信成本。

IDS（intrusion detection systems）通常使用两种方法来检测入侵：

* 基于行为：行为检测应用一个正常活动的配置文件，并将新的流量与该配置文件进行比较
* 基于特征：基于特征的检测很容易应用，但是必须创建一组特征

机器学习方法基于特征，可以提供更好的攻击向量和入侵分析，自动化分析减轻安全分析人员的工作压力。

本文作者的工作主要是比较几种流行的机器学习算法在入侵检测种的效果：

* 回归分析
* 逻辑回归
* 人工神经网络
* 支持向量机

## 2. 相关工作



1. Mukkamala, S., Sung, A. H. (2002) “Intrusion Detection: Support Vector Machines  and  Neural  Networks

   > Mukkamala从计算威胁情报和数据准备的潜力的角度对支持向量机和神经网络进行了比较。研究表明，支持向量机在训练时间和检测精度上都超过了神经网络

2.  Zander, S., Nguyen, T., Armitage, G. (2005) “Automated Traffic Classification and Application Identification using Machine Learning”

   > 提出了分类器Autoclass，可以通过朴素贝叶斯算法对未标注的样本进行聚类

3. Moore, A. W., Zuev, D. (2005) “Internet traffic classification using bayesian analysis techniques

   > 有监督朴素贝叶斯算法对人工标注的包进行处理，优点是精度高，但算法成本较大

4. Bernaille, L., Teixeira, R., Akodkenou, I., Soule, A. (2006) “Traffic classification on the fly

   > 提出了一种仅检查前5个TCP包来识别应用程序的方法。它只分析协商阶段的报文，而不分析控制报文。

5. Shon, T., Moon, J. (2007) “A hybrid machine learning approach to network anomaly detection

   > 提出了一种增强支持向量机(Enhanced SVM)方法来降低无监督学习中的误报率

## 3.问题描述



网络攻击的第一步往往是通过数据包嗅探，电子邮件，恶意软件和社会工程的方式来实现的。作者提出的模型**对于威胁的检测，识别和分类都是通过数据包分析实现的**。

从威胁情报的角度来看，数据包可以分为一些有意义的类型：

1. TCP RST报文：RST表示复位，用来异常的关闭连接
2. TCP FIN报文：用来表示正常关闭连接
3. TCP SYN报文：请求连接标志位

> 如果计算机上的恶意软件可以通过局域网传播，它肯定会试图发起与其他计算机的连接。
>
> * 被感染主机首先会发送SYN标志的请求连接数据包给其他主机
> * 扫描其他主机接收的SYN包的数量，根据阈值确定是否受到扫描
> * 被感染主机会接收到很多RST包

## 4. 方法论

### （1）数据

数据来源：Netresec(公共数据包捕获库，https://www.netresec.com/?page=PcapFiles)

数据类型：PCAP和CSV文件，每个文件大小从6MB到318MB不等

作者使用的数据：7个CSV文件和10个PCAP文件，其中包括受Ursnif和Trickbot感染的流量。每条记录都有数字数据，描述数据包的编号、数据包的时间和长度。分类变量是源IP地址和目的IP地址，使用的协议和包信息。

数据文件中不同协议间的报文分布如下：

![image-20210524123044566](Threat Intelligence Using Machine Learning Packet Dissection.assets/image-20210524123044566.png)

每个PCAP文件大约包含47个不同的源IP，51个不同的目的IP。

数据用Pandas（python用于处理数据的库）表示，这是一种二维，大小可变，可以异构的表格数据（下图）：

![image-20210524123442795](Threat Intelligence Using Machine Learning Packet Dissection.assets/image-20210524123442795.png)

### （2）机器学习模型实现

* 线性回归：输出的因变量是连续的
* 逻辑回归：输出的因变量是分离的
* 神经网络：使用超参数训练，缺点是容易困在局部最优解处
* SVM：通过使用不同的核函数，可以处理多种问题

本文对于数据包的分类可以分为四类：可疑数据包，恶意软件下载（两种），正常

神经网络的基本模型：

![image-20210524124933041](Threat Intelligence Using Machine Learning Packet Dissection.assets/image-20210524124933041.png)

## 5. 实验结果

### （1）不同模型的表现

神经网络模型：训练集和测试集的分类精度都为88%

![image-20210524125357194](Threat Intelligence Using Machine Learning Packet Dissection.assets/image-20210524125357194.png)

支持向量机：可以看出，支持向量机的预测精度优于神经网络。事实上，它的准确率是92%，模型在28秒内执行。该模型对噪声容忍度也很有弹性，因为在数据集中引入噪声似乎对精度没有任何影响

![image-20210524125725952](Threat Intelligence Using Machine Learning Packet Dissection.assets/image-20210524125725952.png)

逻辑回归：训练速度很快，但精度只有78%，对噪声也不敏感

线性回归：速度同样很快，精度不高

### （2）数据不平衡

由于数据集中只有少量的RST、SYN和FIN报文，因此实验数据是不平衡的。

> 正常报文占总报文的比例为74.7%，ACK报文占总报文的比例为22.8%。ACK报文分类权重按规则报文的百分比除以ACK报文的百分比计算，为32.7。SYN报文仅占所有报文的2.45%。SYN报文的权重是常规报文的百分比除以SYN报文的百分比，SYN报文的权重为305.4。（**增加数目较少的类别的包的权重**）

### （3）输入变量数对预测的影响

一些输入可能对预测没有重大贡献，作者采用控制变量法研究输入变量对于结果的影响。

包含所有输入的结果（IP，协议等）：

![image-20210524130631722](Threat Intelligence Using Machine Learning Packet Dissection.assets/image-20210524130631722.png)

从输入中删除源IP地址和目的IP地址，这就留下了“否”、“协议”和“长度”作为预测词，结果如下：

![image-20210524130756342](Threat Intelligence Using Machine Learning Packet Dissection.assets/image-20210524130756342.png)

删除'Protocol'和'Length'输入，留下'No'， 'Source'和'Destination' IP：

![image-20210524130824798](Threat Intelligence Using Machine Learning Packet Dissection.assets/image-20210524130824798.png)

上述结果表明：

**协议和数据包的长度对预测是至关重要的。源IP地址和目的IP地址似乎对准确性没有多大影响**

## 6. 容器化和增量学习

使用Docker创建五个容器：

* Airflow创建模型和处理任务
* PostgreSQL存储元数据
* Kafka用于数据流
* Zookeeper用于管理会话和主题
* MLflow用于显示模型统计，比较，并支持增量学习

增量学习是机器学习的一种方法，依赖于新例子的出现和模型的适应。

## 7. 结论

本文作者使用了基于分类数据的监督学习，使用四种模型进行测试。得到以下结论：

* 多元线性回归有许多解释变量(由研究者操纵，代表输入)与标量响应(因变量代表输出)相关
* 逻辑回归可以用来构造数据流中可疑数据包的概率
* 支持向量机算法能够揭示因变量和自变量之间复杂的非线性关系，有助于数据中的异常检测
* 使用神经网络的增量学习具有很大的潜力，因为仅经过20次模型更新DAG(高达88%)，性能就有了显著提高
* 对于类别的权重再分配也可以提高精度
