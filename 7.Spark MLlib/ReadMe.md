### Spark MLlib
- 内容
    - [ ] 算法工具:
        1. 分类
        2. 回归
        3. 聚类
        4. 协同过滤
    - [ ] 特征工具:
        1. 特征提取
        2. 转换
        3. 降维
        4. 选择工具
    - [ ] 流水线工具:构建，评估，调整工作流
    - [ ] 持久性工具:用来保存算法，加载算法、模型、管道
    - [ ] 实用性工具:线性代数，统计，数据处理
- 类别:
    - [ ] spark.mllib:基于RDD数据对象
    - [ ] spark.ml:基于DataFrame数据抽象
- 支持算法:
    |  | 离散数据 | 连续数据 |
    |:------:|:--------:|:--------:|
    | 监督学习 | Classification;LogisiticRegression(with Elastic-Net);SVM;DecisionTree;RandomForest;GBDT;NaiveBayes;MultilayerPerceptron;OneVsRest | Regression;LinearRegression(with Elastic-Net);Decision Tree;RandomFores;GBDT;AFTSurvivalRegression;IsotonicRegression |
    | 无监督学习 | Clustering;KMeans;GaussianMixture;LDA;PoweriterationClustering;BisectingKMeans | Dimensionality Reduction'matrix factorization;PCA;SVD;ALS;WLS |

- 机器学习流水线
    - 概念: 
        - [ ] Transformer:转换器，把DataFrame转换成另一种DataFrame
        - [ ] Estimator:相当于一个算法,实现了.fit()
        - [ ] Parameter:设置Transformer或Estimator的参数
        - [ ] Pipline:多个工作流连在一起输出结果
    - 构建：<br>
        定义各个阶段piplinestage并有序组织在一起
        - 小例子
            - [] 特征提取TF-IDF<br>
                相关概念可以参考[我的推荐算法学习归纳](https://github.com/LeslieZhoa/My_Recommendation_System)
                ```python
                # 开启spark环境
                cd /usr/local/spark
                sbin/start-all.sh
                # 打开pyspark
                bin/pyspark --master local[2]
                
                # 在pyspark下运行
                from pyspark.ml.feature import HashingTF,IDF,Tokenizer # 导入相关包
                
                # 创建一个dataframe，toDF为定义列名
                sentenceData = spark.createDataFrame([(0, "I heard about Spark and I love Spark"),(0, "I wish Java could use case classes"),(1, "Logistic regression models are neat")]).toDF("label", "sentence")
                
                # Tokenizer作用是分词
                tokenizer = Tokenizer(inputCol="sentence", outputCol="words")
                
                wordsData = tokenizer.transform(sentenceData)
                
                # 显示分词效果
                wordsData.show()
                
                # HashingTF是把分词句子映射成index，返回映射index和单词频次
                hashingTF = HashingTF(inputCol="words", outputCol="rawFeatures", numFeatures=2000)
                featurizedData = hashingTF.transform(wordsData)
                featurizedData.select("words","rawFeatures").show(truncate=False)
                
                # IDF得到单词权重
                idf = IDF(inputCol="rawFeatures", outputCol="features")
                idfModel = idf.fit(featurizedData)
                rescaledData = idfModel.transform(featurizedData)
                rescaledData.select("features", "label").show(truncate=False)
                ```
            - [ ] Logistics回归<br>
                下载[iris数据](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2017/03/iris.txt)到node-1的/usr/local/spark/下
                ```python
                # 依然在pyspark上运行
                # 导入各种类
                from pyspark.ml.linalg import Vector,Vectors
                from pyspark.sql import Row,functions
                from pyspark.ml.evaluation import MulticlassClassificationEvaluator
                from pyspark.ml import Pipeline
                from pyspark.ml.feature import IndexToString, StringIndexer,VectorIndexer,HashingTF, Tokenizer
                from pyspark.ml.classification import LogisticRegression,LogisticRegressionModel,BinaryLogisticRegressionSummary, LogisticRegression
                
                # 定制函数读取特征和label
                >>> def f(x):
                ...     rel = {}
                ...     rel['features']=Vectors. \
                ...     dense(float(x[0]),float(x[1]),float(x[2]),float(x[3]))
                ...     rel['label'] = str(x[4])
                ...     return rel
                
                # 读取 数据
                >>> data = spark.sparkContext. \
                ... textFile("file:///usr/local/spark/iris.txt"). \
                ... map(lambda line: line.split(',')). \
                ... map(lambda p: Row(**f(p))). \
                ... toDF()
                >>> data.show()
                
                # 获取特征
                
                # StringIndexer()-->按频次编码成index
                
                >>> labelIndexer = StringIndexer(). \
                ... setInputCol("label"). \
                ... setOutputCol("indexedLabel"). \
                ... fit(data)
                # VectorIndexer()-->有超参maxCategoties，
                # 如果vector某一列取值种类超过maxCategoties，则保持原状不转换
                
                >>> featureIndexer = VectorIndexer(). \
                ... setInputCol("features"). \
                ... setOutputCol("indexedFeatures"). \
                ... fit(data)
                
                # 设置LogisticRegression算法的参数
                >>> lr = LogisticRegression(). \
                ... setLabelCol("indexedLabel"). \
                ... setFeaturesCol("indexedFeatures"). \
                ... setMaxIter(100). \
                ... setRegParam(0.3). \
                ... setElasticNetParam(0.8)
                >>> print("LogisticRegression parameters:\n" + lr.explainParams())
                
                # 把预测结果转回字符
                >>> labelConverter = IndexToString(). \
                ... setInputCol("prediction"). \
                ... setOutputCol("predictedLabel"). \
                ... setLabels(labelIndexer.labels)
                
                # 练成pipline
                >>> lrPipeline = Pipeline(). \
                ... setStages([labelIndexer, featureIndexer, lr, labelConverter])
                
                # 拆分数据集，训练
                >>> trainingData, testData = data.randomSplit([0.7, 0.3])
                >>> lrPipelineModel = lrPipeline.fit(trainingData)
                >>> lrPredictions = lrPipelineModel.transform(testData)
                
                # 输出预测结果
                >>> preRel = lrPredictions.select( \
                ... "predictedLabel", \
                ... "label", \
                ... "features", \
                ... "probability"). \
                ... collect()
                >>> for item in preRel:
                ...     print(str(item['label'])+','+ \
                ...     str(item['features'])+'-->prob='+ \
                ...     str(item['probability'])+',predictedLabel'+ \
                ...     str(item['predictedLabel']))
                
                # 评估模型，输出准确率
                >>> evaluator = MulticlassClassificationEvaluator(). \
                ... setLabelCol("indexedLabel"). \
                ... setPredictionCol("prediction")
                >>> lrAccuracy = evaluator.evaluate(lrPredictions)
                >>> lrAccuracy
                
                # 获取模型参数
                >>> lrModel = lrPipelineModel.stages[2]
                >>> print ("Coefficients: \n " + str(lrModel.coefficientMatrix)+ \
                ... "\nIntercept: "+str(lrModel.interceptVector)+ \
                ... "\n numClasses: "+str(lrModel.numClasses)+ \
                ... "\n numFeatures: "+str(lrModel.numFeatures))
            
                ```
            - [ ] 决策树
                ```python
                # 在pyspark下运行，步骤基本与逻辑回归一致
                
                # 导包
                >>> from pyspark.ml.classification import DecisionTreeClassificationModel
                >>> from pyspark.ml.classification import DecisionTreeClassifier
                >>> from pyspark.ml import Pipeline,PipelineModel
                >>> from pyspark.ml.evaluation import MulticlassClassificationEvaluator
                >>> from pyspark.ml.linalg import Vector,Vectors
                >>> from pyspark.sql import Row
                >>> from pyspark.ml.feature import IndexToString,StringIndexer,VectorIndexer
                
                # 特征定义
                >>> def f(x):
                ...     rel = {}
                ...     rel['features']=Vectors. \
                ...     dense(float(x[0]),float(x[1]),float(x[2]),float(x[3]))
                ...     rel['label'] = str(x[4])
                ...     return rel
                >>> data = spark.sparkContext. \
                ... textFile("file:///usr/local/spark/iris.txt"). \
                ... map(lambda line: line.split(',')). \
                ... map(lambda p: Row(**f(p))). \
                ... toDF()
                
                # 处理数据
                >>> labelIndexer = StringIndexer(). \
                ... setInputCol("label"). \
                ... setOutputCol("indexedLabel"). \
                ... fit(data)
                >>> featureIndexer = VectorIndexer(). \
                ... setInputCol("features"). \
                ... setOutputCol("indexedFeatures"). \
                ... setMaxCategories(4). \
                ... fit(data)
                >>> labelConverter = IndexToString(). \
                ... setInputCol("prediction"). \
                ... setOutputCol("predictedLabel"). \
                ... setLabels(labelIndexer.labels)
                >>> trainingData, testData = data.randomSplit([0.7, 0.3])
                
                # 创建决策树
                >>> dtClassifier = DecisionTreeClassifier(). \
                ... setLabelCol("indexedLabel"). \
                ... setFeaturesCol("indexedFeatures")
                
                # 构建流水线，拟合参数
                >>> dtPipeline = Pipeline(). \
                ... setStages([labelIndexer, featureIndexer, dtClassifier, labelConverter])
                >>> dtPipelineModel = dtPipeline.fit(trainingData)
                >>> dtPredictions = dtPipelineModel.transform(testData)
                >>> dtPredictions.select("predictedLabel", "label", "features").show(20)
                
                # 评估效果
                >>> evaluator = MulticlassClassificationEvaluator(). \
                ... setLabelCol("indexedLabel"). \
                ... setPredictionCol("prediction")
                >>> dtAccuracy = evaluator.evaluate(dtPredictions)
                >>> dtAccuracy
                
                # 查看参数
                >>> treeModelClassifier = dtPipelineModel.stages[2]
                >>> print("Learned classification tree model:\n" + \
                ... str(treeModelClassifier.toDebugString))
                ```