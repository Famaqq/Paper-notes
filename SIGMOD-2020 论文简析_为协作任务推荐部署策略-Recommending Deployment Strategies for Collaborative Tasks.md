@[TOC](SIGMOD-2020 论文简析:为协作任务推荐部署策略-Recommending Deployment Strategies for Collaborative Tasks)
## 研究背景
大家都知道的，众包任务的流程一般都是由请求者发布，然后上 传到众包平台中的，然后众包工人再从平台上接受任务、提交任务答 案给回平台，平台再进行审核（通过/拒绝），最后汇总给请求者,如下图所示。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201221184948474.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZhbWFfUQ==,size_16,color_FFFFFF,t_70)
目前，众包任务请求者没有对任务的成本、延迟和质量参数上的部署策略进行算法研究及解决方案，仅限于实证研究。典型的参数查询优化问题只关注一个目标来优化。而本文作者从多维度的部署策略建议上去研究，该问题与“推荐关系数据库中的最佳查询计划，其中连接、选择和预测可以多次组合”问题相似。

## 研究目标

 - 帮助请求者部署众包协作任务，研究如何向请求者推荐任务的部署策略，以尽可能实现短时间内以低成本获得高质量的目标；
 -当现有的策略不能满足所制定的部署，则探讨替代部署参数的建议，这样就有可能推荐k个策略可用的不同部署参数，从而进一步指导请求者进行任务部署。

## 问题挑战

 - 首先，如何估计不同类型任务的工人可用性是一个具有挑战性的问题，需要深入研究其自身的优点； 
 - 然后，如何提出原则性但实用的模型来建立部署参数和策略参数之间的关系，或如何现实地对这些职能进行建模，从不同类型任务的历史数据中学习这些职能；
 - 最后，如果请求不能满足所制定的请求时，该怎么进行下一步的策略推荐或任务部署。 
 
## 作者贡献
 - 提出了一个通用的 StratRec 框架，用于在考虑员工可用性的策略进行部署时对一组协作任务的质量，成本和延迟进行建模，以向请求者推荐与其部署任务要求参数相对应的多种策略。
 - 提出了一个备选部署参数算法（ADPAR），未满足的请求将发送到备用参数推荐模块进行处理。
 - 验证了不同策略对不同协作任务（例如文本摘要和文本翻译）的有效性，并为需要指导请求者选择正确策略提供证据。
 - 设计了 BatchStrat，一个统一算法框架来解决批部署推荐问题。 BatchStrat 本质上是贪婪的，它为吞吐量最大化问题提供了精确的结果，并为支付最大化问题(NP-Hard)提供了 1/2 近似因子。
 
## 总体模型
## 1 基本概念
在学习模型之前有几个概念需要先了解一下：
 - ①部署策略**s**：就是我们如何安排工人去完成任务，一般用三个维度去衡量（**结构**-“顺序/并行”；**组织**-“合作/独立”及**风格**-“仅依赖人群/将其与机器算法相结合”）；如下图所示，例如有一个 从英语翻译到法语的任务。有四种策略风格，图(A)中要求工人依次、独立完成任务，并且没有算法的帮助。在图(B)中，工作人员是并行、 协作完成任务，而不需要算法的帮助。最后一种策略规定了一种混合工作风格，其中工作人员与算法相结合。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201221193558733.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZhbWFfUQ==,size_16,color_FFFFFF,t_70)
 - ②工人的可用性**w**：通过利用平台上的历史数据将工人的可利用性捕获为概率分布函数，假设可以捕捉到有70%的机会拥有7%适合承担某种类型的任务的工人，30%的机会拥有 2%的工人，在预期中，这产生了5.5%的可用工人。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201221193705294.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZhbWFfUQ==,size_16,color_FFFFFF,t_70)
③请求的任务**d**：下表 中则是 3 个请求的任务 d 和 4 种部署策略以及他们相应的质量、成本和延迟参数值。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201221194000694.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZhbWFfUQ==,size_16,color_FFFFFF,t_70)
## 2 StratRec 框架
了解完这几个主要的概念后，来看一下作者本文提出的一个框架 “StratRec”，这是一个优化驱动的中间层，位于请求者、工作人员和平台之间。StratRec有两个主要模块：聚合器和替代参数推荐 (ADPAR)。聚合器负责将 k 个策略推荐给一批传入的部署请求，同时考虑工人的可用性。如果平台没有足够资质的编辑人员来满足所有请求，聚合器将通过优化以平台为中心的目标来对它们进行测试，例如最大限度地提高吞吐量或成本。未满足的请求将发送到ADPaR，ADPaR会建议可使用k个策略的不同部署参数。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201221194058363.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZhbWFfUQ==,size_16,color_FFFFFF,t_70)
## 3 StratRec 运行流程
对于该框架有一个策略模型：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201221194946330.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZhbWFfUQ==,size_16,color_FFFFFF,t_70)

 - ①首先，将部署请求集 d、工人的可用性 w 以 及从策略数据库中选择满足请求条件的策略作为输入。
 - ②然后，先执行部署策略建模，以估计给定部署请求 d 的策略 s 的质量、成本和延迟。因此，如果使用策略 s 部署 d， 则将此部署的质量参数建模为：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201221194541414.png)
通过将历史数据拟合到此线性模型，可针对每个 s，d 和参数（质量，成本，等待 时间）组合获得模型参数α和β。一旦这些参数已知，则再次使用上述方程来估计员工需求，以满足使用策略 s 部署 d 的质量阈值(成本和延迟）。最后，生成一个由|S|个可用部署策略作为列及 m 个不同的部署请求作为行映射而成的二维矩阵 W，作为输出。

 - ③这个矩阵中的单元 Wij 代表使用 j 策略部署及第 i 个请求估计所需的劳动力。需要去进行劳动力计算，该值利用策略 j 去部署请求 i 超过这三个需求（质量、成本和延迟）的最大值。从形式上讲，它们可以表述如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201221194451306.png)

 - ④批部署推荐：为满足条件的请求分配策略，要是不满足条件则将不 满足的请求传入 ADPAR 中。

## 4 备选部署参数算法（ADPAR）
它以部署 d 和策略集 S 作为输入，并被设计为推荐替代部署参数 d’和 k 个策略，然后满足 d 与 d’之间的欧几里得距离最小。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201221195454869.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZhbWFfUQ==,size_16,color_FFFFFF,t_70)
**ADPAR-Exact 算法步骤：**该算法处理是几何的，提出了一种精确的离散技术 ADPaRExact， 采用三条扫线，每个参数、质量、成本和延迟都有一个参数，并逐渐放宽参数，以产生允许 k 个策略的最紧的替代参数。通过其独特的设 计选择，ADPaR-Exact 被授权选择最适合优化目标函数的参数，从而产生 ADPaR 的精确解。
 - **步骤一**：首先，每个策略 s 都是三维空间中的一个点，以 d2 策略为例， 则 d2 是一个超矩形。 第一步：计算部署需要满足每个部署参数与策略之间的松弛(增量）。松弛计算这类似于 si.cost-d2.cost（同样适用于质量和延迟）， 当策略成本小于部署阈值时，它显示不需要松弛，因此我们将其转换为 0。以 d2 策略为例，由此得出下表所示的松弛值：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222122656215.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZhbWFfUQ==,size_16,color_FFFFFF,t_70)
 - **步骤二**：根据从步骤 1 计算出的松弛值对所有参数进行排序， 并跟踪策略的索引和松弛值的参数。另一个数据结构，一个大小|S|×3 的布尔矩阵 M，用于跟踪列表 R 中游标 r 当前移动所涵盖的策略的数量。这个矩阵被初始化为 0，随着 r 的推进，条目被更新为 1。下表4为所有参数松弛值排序，我们按照该顺序去扫描策略，每前进一步，矩阵M对应策略的Q、L、C记为1，如下表2所示。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222122931615.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZhbWFfUQ==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222123213758.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZhbWFfUQ==,size_16,color_FFFFFF,t_70)
 - **步骤三**：涉及沿 Q、C 和 L 设计三条扫描线。扫线是一条假想的垂直线，它向右扫过平面。Q 扫描线按 Q 的递增顺序对 C L 平 面中的 S 进行排序，当 ADPaR-Exact 遇到策略时，它会扫描这条线， 游标 r 指向 R 矩阵 M 中这三个值中最小的值，以查看到目前为止涵盖哪些策略的参数。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222123455451.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZhbWFfUQ==,size_16,color_FFFFFF,t_70)
 - **步骤四**：ADPaR-Exact 检查当前 d’是否涵盖 k 个策略，并在相应的二维平面上为第三个参数的固定值创建投影。图显示了(Q，L) 平面中固定成本的示例，它寻找在这 Q、L 两个参数中最大的扩展， 这个新的 d”涵盖了 k 个策略，这就产生了三个新的部署参数， dC”,dQ”,dL”。它选择这三个最好的，并更新 d’。此时，它检 查 M 是否包含 k 个策略。如果有，它停止处理并返回新的 d’和 k 个策略。如果没有，它将光标 r 向右推进。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222123713512.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZhbWFfUQ==,size_16,color_FFFFFF,t_70)
按照表4中的顺序去前进并记录表2，可以发现最终输出的策略为{s1,s2,s3}。
## 数据集
 作者自己在AMT上设置的文本翻译实验收集数据：
 - ①选择了三个流行的英语童谣进行句子翻译。每个押韵由 4-5 行组成，将从英语翻译成印地语。
 - ②对于文本创作，要求工人写 5 个与婚礼主题相关的句子。对于句子翻译，质量测试由 5 个样本句子组成，从英语翻译到印地语，由专家审核。一周中不同的三天，每天雇用了80 名工人。
## 实验分析
- ①首先的观察是工人的可用性可以被估计，并且确实随着时间的推移而变化。观察到，对于这两种任务类型，在窗口 2（星期一 到星期四）期间，与其他两个窗口相比，工作人员更可用，如下图实验所示。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222124431844.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZhbWFfUQ==,size_16,color_FFFFFF,t_70)
- ②每个部署参数与文本编辑任务的工作人员可用性具有线性关系。质量和成本随着工人的可用性而线性增加。延迟随着工人可用性的增加而降低，如下图实验所示。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222124558306.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZhbWFfUQ==,size_16,color_FFFFFF,t_70)
- ③当任务部署时，考虑到 Strat Rec 的推荐，使用统计显著性，它们在固定成本阈值下平 均达到更高的质量和更低的延迟，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222124723250.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZhbWFfUQ==,size_16,color_FFFFFF,t_70)
## 困惑/思考
 - 本篇论文作者主要是设计了一个任务部署策略推荐框架StartRec，以及一个备选部署参数算法（ADPAR）。在框架设计中引入了工人的可用性去与策略的参数进行关联建模，去分析合适的推荐策略，要是没分配到合适策略的请求则会传入ADPAR中进行参数替代，推荐具有新的参数的部署请求d' 及备选策略集。
 - 这篇文章亮点我认为是：作者主要在众包任务部署策略选择上的整体框架流程的分析，一个大框架解决方案的提出；以及前面引入了工人的可用性与策略的参数（质量、成本、延迟）进行建模分析。论文作者的证明证据及公式都比较全面，SIGMOD的论文真的十分硬核，论文阅读下来不得不佩服作者的考虑全面及严谨，但是文章阅读理解起来没有那么形象，有些点还是有跳跃，对于我来说还是不能很快很好的理解，嗯，我继续加油吧！😢
## 附件
- 作者ppt讲解视频地址：https://vimeo.com/447854983
- 作者原文（有积分的支持一下吖，没有的也可以私信我邮箱）：https://download.csdn.net/download/Fama_Q/13635308
- 论文中文翻译版：https://download.csdn.net/download/Fama_Q/13754421
