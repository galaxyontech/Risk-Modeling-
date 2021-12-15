Access to the HDFS file to adm209,nks8839, and sj3549
yl5278: finalproject/PAKDD.txt
ys3234: finalprojectdata/raw_data.csv
Zj637: Final_Project/input/final_project_data.txt

Directory: /profiling_code
/yl5287:
In HDFS, the folder finalproject is shared. 
Run all java files in profiling folder on input data at finalproject/PAKDD.txt(shared), results are saved in related folders:
NullLines - finalproject/nulllines: count the number of lines that is null for each selected columns
MaxMin - finalproject/maxmin: find the min and max age as well as min and max monthly income
CountRecs - finalproject/originalcount: count the total number of lines in PAKDD.txt
CountGroup - finalproject/countgroup: count the number of occurence for all possible values in each column. i.e. for gender, find the number of times that F, M, N occurs
Avg - finalproject/Avg: calculate the average age and income

/ys3234:
Run all java files in profiling folder on input data at finalprojectdata/raw_data.csv(shared), results are saved in related folders:
NARecords.java - finding all NA in raw_data.txt
NullRecords.java - finding all Null in raw_data.txt
CountRecords.java - finding total lines in raw_data.txt
avgincome.java - finding average income (column 46,index 45) in raw_data.csv (ignore all NAs)
Maxincome.java - finding max income in raw_data.txt
Maxincome.java - finding min income in raw_data.txt

/zj637:
Run all java files in profiling folder on input data at Final_Project/input/final_project_data.txt(shared), results are saved in related folders:
avg - find the average value of a numeric column 
CountGroup - find the number of different values of columns 
Max - find the maximum record of a numerical column 
CountRec- Count the number of records in the data 
Mode- Find the mode of a column, to see the mode of different columns, simply change the index of the code 

Directory: /etl_code

/yl5287:
Run CleanMapper.java CleanReducer.java Clean.java on input data at finalproject/PAKDD.txt(shared), result is in folder finalproject/output as well as /etl_code/yl5287/cleandata.txt
In HDFS, the complications of all three dataset at finalproject/finalcleandata.txt
In foder, the complications of all three dataset at /etl_code/yl5287/finalcleandata.txt

/ys3234:
Run CleanData.java on input data at finalprojectdata/raw_data.csv(shared), result is in folder /etl_code/ys3234/finalcleandata_Song.txt
In HDFS, the complications of all three dataset at finalprojectdata/finaldata.txt
In foder, the complications of all three dataset at /etl_code/ys3234/finaldata_total_three_dataset_together.txt

/zj637:
Run c1.java c2.java and c3.java or directly run cleaned.jar on input data, result is in folder Final_Rroject/output, 
In HDFS, the complications of all three dataset at /user/zj637/Final_Project/output/cleaned_data_3/part-r-0000
In foder, the complications of all three dataset at /etl_code/zj637/finalcleandata.txt




Directory: /ana_code
Datas are concatenated together.
for yl5287: it's in finalproject/input with filename finalcleandata.txt (also in /etl_code/yl5287/finalcleandata.txt)
for ys3234: it's in finalprojectdata/finaldata.txt (also in /etl_code/ys3234/finaldata_total_three_dataset_together.txt)
For zj637:  it's in Final_Project/all_input/finalcleandata.txt

The compiled data follows the following schema for analytics:
Housing: Owned - 1, Not-owned - 0
Age: <25 - 1, 25-34 - 2, 35-44 - 3, 45-54 - 4, 55-64 - 5, 65 -74 - 6, >74 - 7
Sex: Male - 1, Female - 0
Employment: Employed - 1, Unemployed - 0
Monthly income: Income/Avg(income) 
Credit: Good -1, Bad - 0

Hive: 
1.access hive
	beeline --silence
	!connect jdbc:hive2://hm-1.hpc.nyu.edu:10000/
	Enter username for jdbc:hive2://hm-1.hpc.nyu.edu:10000/: <net_id>
	Enter password for jdbc:hive2://hm-1.hpc.nyu.edu:10000/: **********
	0: jdbc:hive2://hm-1.hpc.nyu.edu:10000/> use <net_id>;
2.Run each command in Hive.txt file on input data /user/yl5287/finalproject/input/finalcleandata.txt. Explanation for each line is in the Hive.txt file. Output is visible in /screenshots/Hive.pdf with explanations of each query.


Mapreduce:
Run TotalMapper.java TotalReducer.java Total.java on finalcleandata.txt at finalproject/input, output is in folder finalproject/totalscore. Screenshot is in screenshots folder with name Screenshot total_score.png

Correlation_Spark:
1.connnect to peel cluster
   ssh <net id>@peel.hpc.nyu.edu
2.connect to Spark Shell
   spark-shell --deploy-mode client
3.Import package from correlation matrix
   import org.apache.spark.mllib.stat.Statistics
   import org.apache.spark.mllib.linalg._
4.Create input from the cleaned total data file
   val data= sc.textFile("finalprojectdata/finaldata.txt")
5.Split the data by "," and using map to keep all elements as double number 
   val cor = input.map{ case(line:String) => val split = line.split(",");split.map(elem=> elem.toDouble)}.map(arr=>Vectors.dense(arr))
6.Using Statistics.corr to create the correaltion matrix based on the finaldata.txt
   val correlation = Statistics.corr(cor)
7.Shown all data in matrix(6*6)
   println(correlation.toString(6,Int.MaxValue))
Result: screenshot is in /screenshots/correlation_result.jpg


Logistic Regression and Random Foreast:

To executate the below script, please use spark shell in hdfs, to see the exact result of each step, please go analysis.txt file in ana_code folder 

1. Read in files 
scala>  var df = spark.read.option("header", "false").csv("Final_Project/all_input/finalcleandata.txt")
df: org.apache.spark.sql.DataFrame = [_c0: string, _c1: string ... 4 more fields]
2. change colums names 
scala> var df_final_1 =  
df.withColumn("sex",col("_c3").cast(IntegerType)).withColumn("age",col("_c1").cast(IntegerType)).withColumn("credit",col("_c5").cast(IntegerType)).withColumn("Employment",col("_c3").cast(IntegerType)).withColumn("Housing",col("_c0").cast(IntegerType)).withColumn("income",col("_c4").cast(DoubleType))
3. Make input feature columns 
scala> val cols = Array("age","Housing","Employment","income","sex")
cols: Array[String] = Array(age, Housing, Employment, income, sex)
4. make Vector Assembler
scala>  val assembler = new VectorAssembler().setInputCols(cols).setOutputCol("features")
assembler: org.apache.spark.ml.feature.VectorAssembler = vecAssembler_f1b23933f5d3
5 transform df to contain features 
scala>  val featureDf = assembler.transform(df_final_1)
featureDf: org.apache.spark.sql.DataFrame = [_c0: string, _c1: string ... 11 more fields
6. show feature df 
scala> featureDf.show()
7. import string indexer 
scala> import org.apache.spark.ml.feature.StringIndexer
import org.apache.spark.ml.feature.StringIndexer
8. create label index 
scala> val indexer = new StringIndexer().setInputCol("credit").setOutputCol("label")
indexer: org.apache.spark.ml.feature.StringIndexer = strIdx_3a9604d907a0
9. transform into label DF 
scala> val labelDf = indexer.fit(featureDf).transform(featureDf)
labelDf: org.apache.spark.sql.DataFrame = [_c0: string, _c1: string ... 12 more fields]
10. create label index 
scala> val indexer = new StringIndexer().setInputCol("_c5").setOutputCol("label")
indexer: org.apache.spark.ml.feature.StringIndexer = strIdx_9e89edab1515
11. set index of _c5 and transform into labelDF 
scala> val labelDf = indexer.fit(featureDf).transform(featureDf)
labelDf: org.apache.spark.sql.DataFrame = [_c0: string, _c1: string ... 12 more fields]
12. transform into label df 
scala> val labelDf = indexer.fit(featureDf).transform(featureDf)
labelDf: org.apache.spark.sql.DataFrame = [_c0: string, _c1: string ... 12 more fields]
13. create random state 
scala> val seed = 5043
seed: Int = 5043
14. split data into training data and testdata 
scala> val Array(trainingData, testData) = labelDf.randomSplit(Array(0.7, 0.3), seed)
trainingData: org.apache.spark.sql.Dataset[org.apache.spark.sql.Row] = [_c0: string, _c1: string ... 12 more fields]
15. import logistic regression 
scala> import org.apache.spark.ml.classification.LogisticRegression
import org.apache.spark.ml.classification.LogisticRegression
16. set options for logistic regression 
scala> val logisticRegression = new LogisticRegression().setMaxIter(100).setRegParam(0.02).setElasticNetParam(0.8)
logisticRegression: org.apache.spark.ml.classification.LogisticRegression = logreg_8a21048c5297
17. fit the model 
scala> val logisticRegressionModel = logisticRegression.fit(trainingData)
18 create prediction df to view the result 
scala> val predictionDf = logisticRegressionModel.transform(testData)
19 view the result 
scala>  predictionDf.show()
20. important evaluation package 
scala> import org.apache.spark.ml.evaluation.BinaryClassificationEvaluator
import org.apache.spark.ml.evaluation.BinaryClassificationEvaluator
21 create instance 
scala> val evaluator = new BinaryClassificationEvaluator().setLabelCol("label").setRawPredictionCol("prediction").setMetricName("areaUnderROC")
evaluator: org.apache.spark.ml.evaluation.BinaryClassificationEvaluator = binEval_0c158bca1388
22. check the accuracy 
scala> val accuracy = evaluator.evaluate(predictionDf)
accuracy: Double = 0.5   
23 import the random foreast classification 
scala> import org.apache.spark.ml.classification.{RandomForestClassificationModel, RandomForestClassifier}
import org.apache.spark.ml.classification.{RandomForestClassificationModel, RandomForestClassifier}
24 create instance 
scala> val randomForestClassifier = new RandomForestClassifier()
randomForestClassifier: org.apache.spark.ml.classification.RandomForestClassifier = rfc_1627125507a1
25 change options of the instance 
scala> val randomForestClassifier = new RandomForestClassifier().setImpurity("gini").setMaxDepth(10).setNumTrees(20))..setFeatureSubsetStrategy("auto")
26 set random state 
scala> val randomForestClassifier = new RandomForestClassifier().setImpurity("gini").setMaxDepth(10).setNumTrees(20).setFeatureSubsetStrategy("auto").setSeed(seed)
27. fit model into training data
scala> val randomForestModel = randomForestClassifier.fit(trainingData)
28. see predictiondf 
scala> val predictionDf = randomForestModel.transform(testData)
predictionDf: org.apache.spark.sql.DataFrame = [_c0: string, _c1: string ... 15 more fields]
29 view the result 
scala> predictionDf.show(10)
30.create instance 
scala> val randomForestClassifier = new RandomForestClassifier().setImpurity("gini").setMaxDepth(3).setNumTrees(20).setFeatureSubsetStrategy("auto").setSeed(seed)
randomForestClassifier: org.apache.spark.ml.classification.RandomForestClassifier = rfc_72431f480e45
31. fit model into training 
scala> val randomForestModel = randomForestClassifier.fit(trainingData)
randomForestModel: org.apache.spark.ml.classification.RandomForestClassificationModel = RandomForestClassificationModel (uid=rfc_72431f480e45) with 20 trees
32. transform into prediction df 
scala> val predictionDf = randomForestModel.transform(testData)
predictionDf: org.apache.spark.sql.DataFrame = [_c0: string, _c1: string ... 15 more fields]
33. create evaluation matrix 
scala> val evaluator = new BinaryClassificationEvaluator()
evaluator: org.apache.spark.ml.evaluation.BinaryClassificationEvaluator = binEval_b5564a012c86
34. create classification instance 
scala> val evaluator = new BinaryClassificationEvaluator().setLabelCol("label").setMetricName("areaUnderROC")
evaluator: org.apache.spark.ml.evaluation.BinaryClassificationEvaluator = binEval_90f9e6799401
35. see the accuracy 
scala> val accuracy = evaluator.evaluate(predictionDf)
accuracy: Double = 0.6485763787777834
36. notice the accuracy is improved 
scala> println(accuracy)
0.6485763787777834
37. make another model of random foreast 
scala> val randomForestClassifier = new RandomForestClassifier().setImpurity("gini").setMaxDepth(10).setNumTrees(30).setFeatureSubsetStrategy("auto").setSeed(seed)
randomForestClassifier: org.apache.spark.ml.classification.RandomForestClassifier = rfc_f25f8610d04b
38  fit model
scala> val randomForestModel = randomForestClassifier.fit(trainingData)
randomForestModel: org.apache.spark.ml.classification.RandomForestClassificationModel = RandomForestClassificationModel (uid=rfc_f25f8610d04b) with 30 trees
39. create prediction df 
scala> val predictionDf = randomForestModel.transform(testData)
predictionDf: org.apache.spark.sql.DataFrame = [_c0: string, _c1: string ... 15 more fields]
40 create classfication 
scala> val evaluator = new BinaryClassificationEvaluator().setLabelCol("label").setMetricName("areaUnderROC")
evaluator: org.apache.spark.ml.evaluation.BinaryClassificationEvaluator = binEval_3caf0c7e684a
41 print accuracy 
scala> val accuracy = evaluator.evaluate(predictionDf)
accuracy: Double = 0.6577528877401801                                           
42 see the accuracy improved, but not by much 
scala> val randomForestClassifier = new RandomForestClassifier().setImpurity("gini").setMaxDepth(20).setNumTrees(30).setFeatureSubsetStrategy("auto").setSeed(seed)
randomForestClassifier: org.apache.spark.ml.classification.RandomForestClassifier = rfc_cb2e5271ded7
43. fit training data 
scala> val randomForestModel = randomForestClassifier.fit(trainingData)
randomForestModel: org.apache.spark.ml.classification.RandomForestClassificationModel = RandomForestClassificationModel (uid=rfc_cb2e5271ded7) with 30 trees
44 transform prediction df 
scala> val predictionDf = randomForestModel.transform(testData)
predictionDf: org.apache.spark.sql.DataFrame = [_c0: string, _c1: string ... 15 more fields]
45 view result 
scala> predictionDf.show(30)

46. create evaluator 
scala> val evaluator = new BinaryClassificationEvaluator().setLabelCol("label").setMetricName("areaUnderROC")
evaluator: org.apache.spark.ml.evaluation.BinaryClassificationEvaluator = binEval_1fde4d03a88f
47 see the result. 
scala> val accuracy = evaluator.evaluate(predictionDf)
accuracy: Double = 0.6578033471292027 

Result: screenshot is in /screenshots/LR_RF.pdf

