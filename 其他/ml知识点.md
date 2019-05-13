常用的数据挖掘&机器学习知识(点)

Basis(基础)：

MSE(MeanSquare Error 均方误差)，LMS(Least MeanSquare 最小均方)，LSM(Least Square Methods 最小二乘法)，MLE(Maximum LikelihoodEstimation最大似然估计)，QP(QuadraticProgramming 二次规划)， CP(ConditionalProbability条件概率)，JP(Joint Probability 联合概率)，MP(Marginal Probability边缘概率)，Bayesian Formula(贝叶斯公式)，L1 /L2Regularization(L1/L2正则，以及更多的，现在比较火的L2.5正则等)，GD(Gradient Descent 梯度下降)，SGD(Stochastic GradientDescent 随机梯度下降)，Eigenvalue(特征值)，Eigenvector(特征向量)，QR-decomposition(QR分解)，Quantile (分位数)，Covariance(协方差矩阵)。



Common Distribution(常见分布)：

Discrete Distribution(离散型分布)：Bernoulli Distribution/Binomial(贝努利分步/二项分布)，Negative BinomialDistribution(负二项分布)，Multinomial Distribution(多式分布)，Geometric Distribution(几何分布)，Hypergeometric Distribution(超几何分布)，Poisson Distribution (泊松分布)

ContinuousDistribution (连续型分布)：Uniform Distribution(均匀分布)，Normal Distribution/GaussianDistribution(正态分布/高斯分布)，Exponential Distribution(指数分布)，Lognormal Distribution(对数正态分布)，Gamma Distribution(Gamma分布)，Beta Distribution(Beta分布)，Dirichlet Distribution(狄利克雷分布)，Rayleigh Distribution(瑞利分布)，Cauchy Distribution(柯西分布)，Weibull Distribution (韦伯分布)

  Three Sampling Distribution(三大抽样分布)：Chi-square Distribution(卡方分布)，t-distribution(t-distribution)，F-distribution(F-分布)



Data Pre-processing(数据预处理)：

MissingValue Imputation(缺失值填充)，Discretization(离散化)，Mapping(映射)，Normalization(归一化/标准化)。

Sampling(采样)：

SimpleRandom Sampling(简单随机采样)，Offline Sampling(离线等可能K采样)，Online Sampling(在线等可能K采样)，Ratio-based Sampling(等比例随机采样)，Acceptance-rejection Sampling(接受-拒绝采样)，Importance Sampling(重要性采样)，MCMC(Markov Chain MonteCarlo 马尔科夫蒙特卡罗采样算法：Metropolis-Hasting& Gibbs)。



Clustering(聚类)：

K-Means，K-Mediods，二分K-Means，FK-Means，Canopy，Spectral-KMeans(谱聚类)，GMM-EM(混合高斯模型-期望最大化算法解决)，K-Pototypes，CLARANS(基于划分)，BIRCH(基于层次)，CURE(基于层次)，DBSCAN(基于密度)，CLIQUE(基于密度和基于网格)，2014年Science上的密度聚类算法等



Clustering EffectivenessEvaluation(聚类效果评估)：

    Purity(纯度)，RI(Rand Index，芮氏指标)，ARI(Adjusted Rand Index，调整的芮氏指标)，NMI(NormalizedMutual Information，规范化互信息)，F-meaure(F测量)等。



Classification&Regression(分类&回归)：

LR(LinearRegression 线性回归)，LR(Logistic Regression逻辑回归)，SR(SoftmaxRegression 多分类逻辑回归)，GLM(Generalized LinearModel 广义线性模型)，RR(Ridge Regression 岭回归/L2正则最小二乘回归)，LASSO(Least AbsoluteShrinkage and Selectionator Operator L1正则最小二乘回归)， RF(随机森林)，DT(Decision Tree决策树)，GBDT(Gradient BoostingDecision Tree 梯度下降决策树)，CART(Classification AndRegression Tree 分类回归树)，KNN(K-Nearest Neighbor K近邻)，SVM(Support Vector Machine，支持向量机，包括SVC（分类）&SVR（回归）)，KF(Kernel Function 核函数Polynomial KernelFunction 多项式核函数、Guassian Kernel Function 高斯核函数/Radial Basis Function RBF径向基函数、String Kernel Function 字符串核函数)、 NB(Naive Bayes 朴素贝叶斯)，BN(BayesianNetwork/Bayesian Belief Network/Belief Network 贝叶斯网络/贝叶斯信度网络/信念网络)，LDA(Linear DiscriminantAnalysis/Fisher Linear Discriminant 线性判别分析/Fisher线性判别)，EL(Ensemble Learning集成学习Boosting，Bagging，Stacking)，AdaBoost(AdaptiveBoosting 自适应增强)，MEM(Maximum Entropy Model最大熵模型)



Classification EffectivenessEvaluation(分类效果评估)：

ConfusionMatrix(混淆矩阵)，Precision(精确度)，Recall(召回率)，Accuracy(准确率)，F-score(F得分)，ROC Curve(ROC曲线)，AUC(AUC面积)，Lift Curve(Lift曲线) ，KS Curve(KS曲线)。



PGM(ProbabilisticGraphical Models概率图模型)：

BN(BayesianNetwork/Bayesian Belief Network/ Belief Network 贝叶斯网络/贝叶斯信度网络/信念网络)，MC(Markov Chain 马尔科夫链)，HMM(Hidden MarkovModel 马尔科夫模型)，MEMM(Maximum EntropyMarkov Model 最大熵马尔科夫模型)，CRF(Conditional RandomField 条件随机场)，MRF(Markov RandomField 马尔科夫随机场)。



NN(Neural Network神经网络)：

ANN(ArtificialNeural Network 人工神经网络)，BP(Error Back Propagation 误差反向传播)，HN（Hopfield Network），
RNN(Recurrent Neural Network，循环神经网络），SRN（Simple Recurrent Network，简单的循环神经网络），ESN（Echo State Network，回声状态网络），LSTM（Long Short Term Memory 长短记忆神经网络），CW-RNN（Clockwork

 Recurrent Neural Network，时钟驱动循环神经网络，2014ICML）等。



Deep Learning(深度学习)：

Auto-encoder(自动编码器)，SAE(Stacked Auto-encoders堆叠自动编码器：Sparse Auto-encoders稀疏自动编码器、Denoising Auto-encoders去噪自动编码器、ContractiveAuto-encoders 收缩自动编码器)，RBM(Restricted BoltzmannMachine 受限玻尔兹曼机)，DBN(Deep BeliefNetwork 深度信念网络)，CNN(Convolutional NeuralNetwork 卷积神经网络)，Word2Vec(词向量学习模型)。



Dimensionality Reduction(降维)：                 

LDA(LinearDiscriminant Analysis/Fisher Linear Discriminant 线性判别分析/Fish线性判别)，PCA(Principal ComponentAnalysis 主成分分析)，ICA(Independent ComponentAnalysis 独立成分分析)，SVD(Singular ValueDecomposition 奇异值分解)，FA(Factor Analysis 因子分析法)。



Text Mining(文本挖掘)：

VSM(Vector SpaceModel向量空间模型)，Word2Vec(词向量学习模型)，TF(Term Frequency词频)，TF-IDF(TermFrequency-Inverse Document Frequency 词频-逆向文档频率)，MI(Mutual Information 互信息)，ECE(Expected CrossEntropy 期望交叉熵)，QEMI(二次信息熵)，IG(Information Gain 信息增益)，IGR(InformationGain Ratio 信息增益率)，Gini(基尼系数)，x2 Statistic(x2统计量)，TEW(Text EvidenceWeight文本证据权)，OR(OddsRatio 优势率)，N-Gram Model，LSA(LatentSemantic Analysis 潜在语义分析)，PLSA(ProbabilisticLatent Semantic Analysis 基于概率的潜在语义分析)，LDA(Latent DirichletAllocation 潜在狄利克雷模型)，SLM(StatisticalLanguage Model，统计语言模型)，NPLM(NeuralProbabilistic Language Model，神经概率语言模型)，CBOW(Continuous Bag of Words Model，连续词袋模型)，Skip-gram(Skip-gramModel)等。



Association Mining(关联挖掘)：

Apriori，FP-growth(FrequencyPattern Tree Growth 频繁模式树生长算法)，AprioriAll，Spade。



Recommendation Engine(推荐引擎)：

DBR(Demographic-basedRecommendation 基于人口统计学的推荐)，CBR(Context-based Recommendation 基于内容的推荐)，CF(Collaborative Filtering协同过滤)，UCF(User-based CollaborativeFiltering Recommendation 基于用户的协同过滤推荐)，ICF(Item-based CollaborativeFiltering Recommendation 基于项目的协同过滤推荐)。



SimilarityMeasure&Distance Measure(相似性与距离度量)：

EuclideanDistance(欧式距离)，Manhattan Distance(曼哈顿距离)，Chebyshev Distance(切比雪夫距离)，Minkowski Distance(闵可夫斯基距离)，Standardized EuclideanDistance(标准化欧氏距离)，Mahalanobis Distance(马氏距离)，Cos(Cosine 余弦)，Hamming Distance/EditDistance(汉明距离/编辑距离)，Jaccard Distance(杰卡德距离)，Correlation CoefficientDistance(相关系数距离)，Information Entropy(信息熵)，KL(Kullback-LeiblerDivergence KL散度/Relative Entropy 相对熵)。



Optimization(最优化)：

Non-constrained Optimization(无约束优化)：Cyclic Variable Methods(变量轮换法)，Pattern Search Methods(模式搜索法)，Variable Simplex Methods(可变单纯形法)，Gradient Descent Methods(梯度下降法)，Newton Methods(牛顿法)，Quasi-Newton Methods(拟牛顿法)，Conjugate GradientMethods(共轭梯度法)。

ConstrainedOptimization(有约束优化)：Approximation ProgrammingMethods(近似规划法)，Feasible DirectionMethods(可行方向法)，Penalty Function Methods(罚函数法)，Multiplier Methods(乘子法)。

HeuristicAlgorithm(启发式算法)，SA(Simulated Annealing，模拟退火算法)，GA(genetic algorithm遗传算法)



Feature Selection(特征选择)：

MutualInformation(互信息)，Document Frequence(文档频率)，Information Gain(信息增益)，Chi-squared Test(卡方检验)，Gini(基尼系数)。



Outlier Detection(异常点检测)：

Statistic-based(基于统计)，Distance-based(基于距离)，Density-based(基于密度)，Clustering-based(基于聚类)。



Learning to Rank(基于学习的排序)：

Pointwise：McRank；

Pairwise：RankingSVM，RankNet，Frank，RankBoost；

Listwise：AdaRank，SoftRank，LamdaMART；



Tool(工具)：

MPI，Hadoop生态圈，Spark，BSP，Weka，Mahout，Scikit-learn，PyBrain…
1、熟悉一个或多个常见的神经网络开源工具库，例如：Tensorflow, Caffe, MXNet 等； 
2、熟悉常用的机器学习/科学计算库，例如：SciKit-Learn, Numpy 等；
