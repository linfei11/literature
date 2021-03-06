# Holmes: An Efficient and Lightweight Semantic Based Anomalous Email Detector

> Holmes:一种高效、轻量级的基于语义的异常电子邮件检测器

## 1.背景

### （1）期刊/会议级别

arXiv预印本网站 2021.4.25

### （2）作者团队

悉尼新南威尔士大学计算机科学与工程学院（Shiyi Yang，Hui Guo）

深圳市桑福科技有限公司网络安全部门（Peilun Wu）

北京水利部信息中心（Feng Qian）

![image-20210513092055819](Holmes一种高效、轻量级的基于语义的异常电子邮件检测器.assets/image-20210513092055819.png)

![image-20210513092141904](Holmes一种高效、轻量级的基于语义的异常电子邮件检测器.assets/image-20210513092141904.png)

### （3）文章背景

尽管Facebook，微信等即时通信软件越来越流行，但是电子邮件对于企业来说仍是必不可少的。电子邮件也是骇客入侵内部网络的入口，恶意邮件的类型：

1. 网络钓鱼
2. 欺诈
3. 勒索邮件
4. 恶意广告

这些恶意邮件主要是通过看似合法的内容来伪装自己，这些邮件的正文中通常包含：（用于劫持用户系统）

* 包含恶意软件的附件
* 恶意的嵌入式URL

为了解决恶意邮件问题，许多企业都部署了过滤系统，使用的技术包括：**灰名单和主题分析**

> **灰名单**是一项反垃圾邮件的技术，当邮件来自未列入[白名单](http://www.emailcamel.com/node/141)或先前未知的发件人时，在SMTP 会话期间灰名单将以暂时错误信息拒绝该邮件。 不仅如此，在指定时段内（默认 15 分钟），任何重新投递的企图都将暂时被拒绝。 因为垃圾邮件制造者在邮件被拒绝后通常不会进一步尝试投递，灰名单有助于显著减少用户收到的垃圾邮件数量。

但是上述技术不能检测新的，未知邮件的威胁（利用不同的热门话题，如COVID-19，美国大选等信息）。

这些未知的威胁可以很容易地绕过反垃圾邮件网关，成功地渗透到目标系统，导致一系列的破坏性后果，如管理员帐户被盗、数据库攻击和财务勒索。

为了解决上述问题，作者提出了Holmes——一种基于人工智能技术的异常邮件检测系统。Holmes结合**词嵌入**和**新奇性检测**来发现大型企业环境中大量镜像SMTP通信流中的异常行为。

主要贡献：

1. 提出了一种高效、轻量级的基于语义的异常电子邮件检测器Holmes，该检测器不仅能有效地发现新的电子邮件威胁，而且在现实环境中保持较低的误报率。
2. 通过可视化展示了检测到的可疑电子邮件的相关关系，并根据网络威胁情报和攻击者的地理位置对攻击者进行画像。
3. 通过在真实的企业环境中对Holmes进行评估，结果显示：Holmes不仅可以准确地检测出那些已经被反垃圾邮件网关拦截的邮件威胁，还可以发现大量反垃圾邮件网关不能发现的邮件威胁。
4. 将Holmes与多个商业电子邮件检测器进行对比，发现，Holmes的性能优于其他检测器，具有很高的准确率。

### （4）相关问题和挑战

根据MITRE ATT&CK攻击矩阵，异常邮件可以分为两类：

* 外部威胁：来源为外部的电子邮件
* 内部威胁：来源为被窃取的内部合法账户

大多数先前的工作集中在对于特定攻击类型的探测，比如：基于URL的横向钓鱼检测，在大型网络空间中，利用搜索引擎检测钓鱼网页。

仍有许多悬而未决的问题和挑战需要解决。现将一些问题及现有的解决方案介绍如下：

> 1. 脆弱的身份验证机制：由于缺乏本地身份验证机制的SMTP服务(没有内置的认证方法),攻击者可以很容易地伪造电子邮件头,假冒身份,以欺骗接受者,避免被垃圾邮件过滤系统过滤。
>
>    解决方法：SPF [13] (Sender Policy Framework)、DKIM [14] (Domain Key Identified Mail)和DMARC  [15] (Domain- based Message Authentication, Reporting, and  Conformance)等框架来提高认证机制的可靠性。然而，由于基于组件的软件设计[16]的不一致性，例如邮件转发服务器的不兼容性，这些安全框架仍然可能无效，这使得大量的电子邮件威胁有可能规避防御策略。
>
> 2. 对未知的变种威胁邮件缺乏敏感性：由于SMTP的不可靠性导致的电子邮件威胁已经演变成多种变异类型，传统安全产品很难发现这些变异类型。
>
>    解决方法：基于规则的异常检测方法到基于人工智能的方法的转变
>
> 3. 高误报率：机器学习方法用于异常检测具有误报率高的特点。虽然FPR是显示优越性能的指标，但是作者认为关键是发现了多少有价值的威胁，因此不把FPR作为评估指标。
>
> 4. 高成本和性能瓶颈：**数据收集的高成本和算法性能的瓶颈是基于人工智能检测方法的一个重大挑战**。研究人员可以不断提高算法的复杂度以获得更好的性能;然而，在实际应用中，大多数人工智能模型通常需要**大量的计算和存储资源**，这使得检测器变得不容易使用，并显著延迟了响应时间[22]-[24]。此外，使用监督机器学习的检测器**需要输入带标签的记录**，一旦它们的性能开始下降，通常需要重新训练，这也使得机器学习在自动化方面的作用大大降低。
>
> 5. 缺乏来源分析：很少有探测器考虑将来源分析集成到检测机制中。然而来源分析是电子邮件检测中一个重要且有效的组成部分。来源分析可以揭示攻击事件和电子邮件背后的攻击者画像的细节，例如(1)电子邮件来自哪里;(2)真实发送人是谁;(3)恶意shell代码如何执行;(4)恶意事件之间的潜在相关性。上述信息对于蓝队分析攻击TTPs(技术、战术和程序)非常重要，并进一步帮助安全专家识别单个攻击者或组织.

## 2.模型设计

> Holmes：异常电子邮件检测
>
> 为了解决上述挑战，作者引入了一种高效、轻量级的面向语义的异常电子邮件检测器Holmes，它可以通过分析SMTP头部中电子邮件的收件人/主题来检测电子邮件攻击。Holmes最初是用Python编写的，只有52行代码。基于运行时分析，Holmes可以在不到73秒的时间内完成整个检测，在CentOS虚拟服务器上使用约700  MB数据集(一天的SMTP记录)，消耗127mb内存。

### （1）词嵌入

文本类型的头部信息不能直接输入机器学习模型，所以要转化为机器可以理解的数据类型。

大多数算法工程师使用OneHotEncoder或OrdinalEncoder快速编码分类特性，这可以通过开源库Scikit-Learn简单实现。然而，这两种方法都不能有效地记忆和保持多个**数据项在时间或空间维度上的潜在语义关联**，因此作者使用NLP的词嵌入技术。

使用的SMTP协议头部特征：

![image-20210513135354812](Holmes一种高效、轻量级的基于语义的异常电子邮件检测器.assets/image-20210513135354812.png)

词嵌入关键代码：

![image-20210513135502036](Holmes一种高效、轻量级的基于语义的异常电子邮件检测器.assets/image-20210513135502036.png)

### （2）未知威胁电子邮件检测（以前从未出现）

> LOF算法：
>
> LOF通过计算一个数值score来反映一个样本的异常程度。这个数值的大致意思是：一个样本点周围的样本点所处位置的平均密度比上该样本点所在位置的密度。比值越大于1，则该点所在位置的密度越小于其周围样本所在位置的密度，这个点就越有可能是异常点。关于密度等理论概念，详见下面介绍。
> 　　用视觉直观的感受一下下图，对于C1集合的点，整体间距，密度，分散情况较为均匀一致，可以认为是同一簇；对于C2集合的点，同样可认为是一簇。o1、o2点相对孤立，可以认为是异常点或离散点。现在的问题是，如何实现算法的通用性，可以满足C1和C2这种密度分散情况迥异的集合的异常点识别。LOF可以实现我们的目标。
>
> ![在这里插入图片描述](Holmes一种高效、轻量级的基于语义的异常电子邮件检测器.assets/20191105135119987.png)

原理：异常的电子邮件往往是未知的和新奇的，它们的行为通常偏离正常活动的痕迹。

基于这一原理，作者**使用局部离群因子进行新颖度检测，以发现那些在邮件历史记录中从未见过的邮件**

代码如下：

![image-20210513135908891](Holmes一种高效、轻量级的基于语义的异常电子邮件检测器.assets/image-20210513135908891.png)

从Doc2Vec中学习到的特征向量作为新颖性检测的输入。LOF可以学习历史邮件，然后根据离群因素的阈值预测新发现的邮件。

将LOF应用于新颖性检测有一些巨大的优势:

(1)它允许训练带有污染的数据;

(2)基于其较低的复杂度，可以进行在线学习，避免了性能下降和再训练的成本;

(3)对微调不敏感，保证了参数学习的有效性和稳定性。

### （3）罕见度选择模块

黑客几乎不会将重复的邮件重新发送给相同的收件人，在此基础上，作者设计了一个罕见度选择模块，过滤掉新见邮件中通信频率较高的邮件，这可以降低误报率

> 罕见的发件人→收件人关系

### （4）关系图分析

大多数工作都忽略了一个问题：几个异常事件之间的关系

作者引入一个相关图分析（CGA）模块，；通过关联多个不同的异常事件来提高对攻击者画像能力。

CGA是一个有向图：

节点包括：国家，源IP，发送者，主题

有向图强制具有密集连接的节点更接近，没有连接或连接较少则会相对分散。

> CGA描述了几种不同异常(如相同的srcIp、相同的主题或相同的发送方)的相似性，并根据它们的地理位置将集群集中起来，这样可以显著提高起源分析的可解释性。

![image-20210513141506067](Holmes一种高效、轻量级的基于语义的异常电子邮件检测器.assets/image-20210513141506067.png)

上图显示了CGA的可视化结果，其中突出显示了按照国家源IP集中的子图中的连通组件;连接的组件表明:(1)相同的恶意邮件，但来源不同;(2)同一个源发送多个不同的恶意邮件。

## 3.实验结果

将Holmes部署在企业环境中，从ElasticSearch  (ES)服务器读取镜像SMTP记录。此外，Holmes使用流处理可以学习历史SMTP记录及时检测未知异常。

评估指标如下：

1. 异常检出率
2. 恶意检出率
3. 误报率

并将检测结果与VirusTotal中安全公司提供的几种商用检测器进行了比较。结果表明，Holmes能够发现未知邮件威胁，具有较高的总体检出率，明显优于其他检测器。

### （1）一个月的检测结果

作者在2020年12月运行了Holmes并对其检测性能进行了评估。总体检测结果如图所示：Holmes每天可以发现大约1000封异常邮件，其中约23%是真正恶意的，包括钓鱼链接或被感染的恶意软件。其余的异常大多是垃圾邮件，只有少数是误报的。

![image-20210513142845647](Holmes一种高效、轻量级的基于语义的异常电子邮件检测器.assets/image-20210513142845647.png)

作者从邮件服务器中提取了一些不能被反垃圾邮件网关识别的，但是被Holmes识别，最终被人工确认为恶意的邮件，来说明这些精心设计的邮件的形式。

1）：伪造DHL服务的恶意邮件(DHL：全球著名的物流公司)

![image-20210513143742875](Holmes一种高效、轻量级的基于语义的异常电子邮件检测器.assets/image-20210513143742875.png)

* 电子邮件使用了与发货通知相关的一个非常常见的主题
* 发送者信息被修改为“DHL  Express”，可以通过一些黑客工具实现
* 该邮件包含一个名为invoice.doc的附件，该附件是一个利用CVE-2017-11882[40]漏洞的恶意木马文件
* 这封电子邮件包含一张精美的DHL快递服务图片，以欺骗收件人

2）：New Sign-in Attempt：电子邮件使用一个名为“New Sign-in Attempt”的欺骗主题来欺骗收件人，让他们更改电子邮件帐户密码。

![image-20210513144259420](Holmes一种高效、轻量级的基于语义的异常电子邮件检测器.assets/image-20210513144259420.png)

3）：Warning!!!：该邮件类似于攻击场景IV  -A2，在邮件内容中嵌入了一个用于钓鱼的链接，如图6所示。然而，不同的是，钓鱼链接https://armonaoil.com/admin/images/  npgtr/newsl/potcpanel[。]html是来自合法网站而不是个人网站，这表明合法网站已被劫持。

![image-20210513144455211](Holmes一种高效、轻量级的基于语义的异常电子邮件检测器.assets/image-20210513144455211.png)

### （2）关系图分析

通过分析关系图，得到的一些案例结论：

1. 比特币欺诈:该案件展示了一种集群式行为，数百名来自不同srcIp地址的发件人发送相同的电子邮件来欺骗收件人，邮件内容是:“您的计算机已被控制……将比特币转移到钱包....”。该情况表明受控僵尸网络已被用于电子邮件欺诈，这是一个强大的IOC(妥协指标)的网络威胁情报数据库。
2. 垃圾邮件:这种攻击是一种集群式行为，来自同一个国家的几个srcIp地址发送了大量主题相似的垃圾邮件，涉及敏感的政治话题、赌博和色情信息。这些电子邮件基本上是无害的，但对于调查不正常的电子邮件通信行为仍然很重要，因为一些垃圾邮件可能涉及潜在的间谍活动，这可能导致内部机密文件的信息泄漏。
3. 周期性网络钓鱼行为:作者还从CGA观察到一群来自同一国家的黑客使用不同的srcIp地址定期向其客户发送主题相似的恶意电子邮件，其网络钓鱼链接使用相同的URL重定向技术;这种周期性的异常行为最终被我们的安全分析师认定为一种长期的、有针对性的钓鱼行为

### （3）Holmes与其他商业电子邮件检测器的比较

为了评估检测能力，作者将Holmes与VirusTotal中6个主要安全厂商提供的一些商业邮件检测器进行了比较，并选择了15个恶意邮件作为测试样本，这些恶意邮件要么包含钓鱼链接，要么包含被感染的恶意附件。

![image-20210513145648884](Holmes一种高效、轻量级的基于语义的异常电子邮件检测器.assets/image-20210513145648884.png)

Holmes在评价中可以成功地检测出全部15个测试样本

### （4）一些有趣的电子邮件检测方法

> * 可信社交网络图
>
>   社交网络图可以根据发送者与接收者的历史交流记录建立关联;一旦一个新的关系被观察到，电子邮件应该被标记为需要进一步研究。
>
> * 内容和称呼分析
>
>   内容分析是识别电子邮件是否带有恶意的最简单和直接的方法;一些检测器还可能分析称呼的名称是否与收件人的名称匹配，以确保它不是群发邮件
>
> * 附件检测（黑客有许多技术来加密或混淆恶意软件，因此该项检测技术用处不大）

## 4.结论

本文介绍了一种轻量级的基于语义的异常邮件检测器Holmes，它可以有效地发现真实网络中的恶意邮件，而且在消耗成本和测试性能之间做出了很好的平衡。

思考：

* 文章主要使用LOF（局部离群因子算法）和罕见度模块来对一个邮件进行判断，但是离群点一定是恶意邮件吗？作者根据其所处的公司网络得到的结论不一定适用于其他组织架构不同的公司。
* 最后测试的时候只用了15个样本，测试样本集太小了，可能不具有代表性。

