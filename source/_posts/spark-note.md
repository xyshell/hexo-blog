---
title: spark-note
date: 2020-07-26 22:27:48
tags: spark
toc: true
---

# Data Processing

## Creating a SparkSession Object

```python
from pyspark.sql import SparkSession
spark=SparkSession.builder.appName('data_processing').getOrCreate()
import pyspark.sql.functions as F
from pyspark.sql.types import *
```

## Creating Dataframes

```python
schema=StructType() \
    .add("user_id","string") \
    .add("country","string") \
    .add("browser", "string") \
    .add("OS",'string') \
    .add("age", "integer")
df=spark.createDataFrame([
    ("A203",'India',"Chrome","WIN",33),
    ("A201",'China',"Safari","MacOS",35),
    ("A205",'UK',"Mozilla","Linux",25)
],schema=schema)
df.printSchema()
df.show()
```

## Null Values

```python
df_na=spark.createDataFrame([
    ("A203",None,"Chrome","WIN",33),
    ("A201",'China',None,"MacOS",35),
    ("A205",'UK',"Mozilla","Linux",25)
],schema=schema)
df_na.show()
df_na.fillna('0').show()
df_na.fillna( { 'country':'USA', 'browser':'Safari' } ).show()
df_na.na.drop().show() # dropna row
df_na.na.drop(subset='country').show() # dropna row for column
df_na.replace("Chrome","Google Chrome").show()
df_na.drop('user_id').show() # drop column

df=spark.read.csv("customer_data.csv",header=True,inferSchema=True)
df.count()
len(df.columns)
df.show(3)
df.summary().show() # describe
```

## Subset of a Dataframe

```python
df.select(['Customer_subtype','Avg_Salary']).show()

df.filter(df['Avg_Salary'] > 1000000).count()
df.filter(df['Avg_Salary'] > 1000000).show()
df.filter(df['Avg_Salary'] > 500000).filter(df['Number_of_houses'] > 2).show()

df.where((df['Avg_Salary'] > 500000) & (df['Number_of_houses'] > 2)).show()
```

## Aggregations

```python
df.groupBy('Customer_subtype').count().show()

for col in df.columns:
    if col !='Avg_Salary':
        print(f" Aggregation for {col}")
        df.groupBy(col).count().orderBy('count',ascending=False).show(truncate=False)

df.groupBy('Customer_main_type').agg(F.mean('Avg_Salary')).show()
df.groupBy('Customer_main_type').agg(F.max('Avg_Salary')).show()
df.groupBy('Customer_main_type').agg(F.min('Avg_Salary')).show()
df.groupBy('Customer_main_type').agg(F.sum('Avg_Salary')).show()

df.sort("Avg_Salary", ascending=False).show()

df.groupBy('Customer_subtype') \
    .agg(F.avg('Avg_Salary') \
    .alias('mean_salary')) \
    .orderBy('mean_salary',ascending=False) \
    .show(50,False)
df.groupBy('Customer_subtype') \
    .agg(F.max('Avg_Salary') \
    .alias('max_salary')) \
    .orderBy('max_salary',ascending=False) \
    .show()
```

## Collect

```python
df.groupby("Customer_subtype") \
    .agg(F.collect_set("Number_of_houses")) \
    .show()
df.groupby("Customer_subtype") \
    .agg(F.collect_list("Number_of_houses")) \
    .show()

df=df.withColumn('constant',F.lit('finance'))
df.select('Customer_subtype','constant').show()
```

## User-Defined Functions (UDFs)

```python
from pyspark.sql.functions import udf
df.groupby("Avg_age").count().show()

def age_category(age):
    if age == "20-30 years":
        return "Young"
    elif age== "30-40 years":
        return "Mid Aged"
    elif ((age== "40-50 years") or (age== "50-60 years")) :
        return "Old"
    else:
        return "Very Old"
age_udf=udf(age_category,StringType())
df=df.withColumn('age_category',age_udf(df['Avg_age']))
df.select('Avg_age','age_category').show()
df.groupby("age_category").count().show()

df.select('Avg_Salary').summary().show()
min_sal=1361
max_sal=48919896
from pyspark.sql.functions import pandas_udf, PandasUDFType
def scaled_salary(salary):
    scaled_sal=(salary-min_sal)/(max_sal-min_sal)
    return scaled_sal
scaling_udf = pandas_udf(scaled_salary, DoubleType())
df.withColumn("scaled_salary",scaling_udf(df['Avg_Salary'])) \
    .show(10,False)
```

## Joins

```python
region_data = spark.createDataFrame([
    ('Family with grown ups','PN'),
    ('Driven Growers','GJ'),
    ('Conservative families','DD'),
    ('Cruising Seniors','DL'),
    ('Average Family ','MN'),
    ('Living well','KA'),
    ('Successful hedonists','JH'),
    ('Retired and Religious','AX'),
    ('Career Loners','HY'),('Farmers','JH')
], schema=StructType() \
    .add("Customer_main_type","string") \
    .add("Region Code","string"))
new_df=df.join(region_data,on='Customer_main_type')
new_df.groupby("Region Code").count().show()
```

## Pivoting

```python
df.groupBy('Customer_main_type') \
    .pivot('Avg_age') \ 
    .sum('Avg_Salary') \
    .fillna(0) \ 
    .show()
df.groupBy('Customer_main_type') \
    .pivot('label') \
    .sum('Avg_Salary') \
    .fillna(0) \
    .show()
```

## Window Functions or Windowed Aggregates

```python
from pyspark.sql.window import Window
from pyspark.sql.functions import col,row_number
win = Window.orderBy(df['Avg_Salary'].desc())
df=df.withColumn('rank', row_number().over(win).alias('rank')).show()

win_1=Window.partitionBy("Customer_subtype").orderBy(df['Avg_Salary'].desc())
df=df.withColumn('rank', row_number().over(win_1).alias('rank'))
df.groupBy('rank').count().orderBy('rank').show()
df.filter(col('rank') < 4).show()
```