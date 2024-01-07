# Spark Big Data Analytics
Download the search term data set for the e-commerce web server and run analytic queries on it using pyspark and JupyterLab. You will then load a pretrained sales forecasting model and predict the sales forecast for a future year.


**Objectives**
- Load data into a dataframe
- Perform analytics on the data
- Predict future outcomes using a trained forecasting model

**Tools / Software Used**
- Apache Spark
- JupyterLab

## 1 - Create a Spark session and populate a dataframe
We first intall python packages needed for Apache Spark
```
!pip install pyspark
!pip install findspark
```

Now we can initialize a Spark session
```
import findspark
findspark.init()

from pyspark import SparkContext, SparkConf
from pyspark.sql import SparkSession

# Creating a spark context class
sc = SparkContext()

# Creating a spark session
spark = SparkSession \
    .builder \
    .appName("E-commerce web server search term analysis").getOrCreate()
```

Download the search term dataset from the below url. Then load the dataset into a spark dataframe.
```
!wget https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DB0321EN-SkillsNetwork/Bigdata%20and%20Spark/searchterms.csv
```
```
searchTermsdf = spark.read.option("header",True).csv("searchterms.csv")
```

## 2 - Analyze the data
We can display the shape of our dataframe by printing the number of rows and columns.
```
numRows = searchTermsdf.count()
numColumns = len(searchTermsdf.columns)
```
```
Rows: 10001
Columns: 4
```

Let's see what the data in our dataframe looks like by displaying the first 5 rows.
```
searchTermsdf.show(5)
```
```
+---+-----+----+--------------+
|day|month|year|    searchterm|
+---+-----+----+--------------+
| 12|   11|2021| mobile 6 inch|
| 12|   11|2021| mobile latest|
| 12|   11|2021|   tablet wifi|
| 12|   11|2021|laptop 14 inch|
| 12|   11|2021|     mobile 5g|
+---+-----+----+--------------+
only showing top 5 rows

```
We can now perform analytics on our data. For example, how many times was term `gaming laptop` searched?

```
searchTermsdf.createOrReplaceTempView('searchTerms')
spark.sql("SELECT COUNT(*) FROM searchTerms WHERE searchterm='gaming laptop'").show()
```
```
+--------+
|count(1)|
+--------+
|     499|
+--------+
```

Or what are the top 5 most frequently used search terms?
```
spark.sql('SELECT searchterm, COUNT(searchterm) FROM searchTerms GROUP BY searchterm ORDER BY COUNT(searchterm) DESC').show(5)
```
```
+-------------+-----------------+
|   searchterm|count(searchterm)|
+-------------+-----------------+
|mobile 6 inch|             2312|
|    mobile 5g|             2301|
|mobile latest|             1327|
|       laptop|              935|
|  tablet wifi|              896|
+-------------+-----------------+
only showing top 5 rows
```

## 3 - Prediction Model
We use a pretrained sales forecasting model to make predictions about future sales. First we need to download the model.
```
!wget https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DB0321EN-SkillsNetwork/Bigdata%20and%20Spark/model.tar.gz
!tar -xvzf model.tar.gz
```

Load the downloaded model into Spark
```
from pyspark.ml.regression import LinearRegressionModel
salesPredictionModel = LinearRegressionModel.load('sales_prediction.model')
```

Using the loaded model, we make a prediction for the amount of sales in the year 2023.
```
from pyspark.ml.feature import VectorAssembler

# This function converts a scalar number into a dataframe that can be used by the model to predict.
def predict(year):
    assembler = VectorAssembler(inputCols=["year"],outputCol="features")
    data = [[year,0]]
    columns = ["year", "sales"]
    _ = spark.createDataFrame(data, columns)
    __ = assembler.transform(_).select("features","sales")
    predictions = salesPredictionModel.transform(__)
    predictions.select("prediction").show()

predict(2023)
```
```
+------------------+
|        prediction|
+------------------+
|175.16564294006457|
+------------------+
```