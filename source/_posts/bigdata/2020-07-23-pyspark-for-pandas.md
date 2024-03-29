---
layout: post
title: Pandas UDF and Function Api in Spark
categories: [big data]
description: 官方文档解读
keywords: spark, pandas, arrow
catalog: true
multilingual: false
tags: big data
date: 2020-07-23
---

## Apache Arrow in PySpark
Spark可以使用`Apache Arrow`对python和jvm之间的数据进行传输， 这样会比默认传输方式更加高效。
为了能高效地利用特性和保障兼容性，使用的时候可能需要一点点修改或者配置。

## 为什么使用Arrow作为数据交换中介能够提升性能？
普通的python udf需要经过如下步骤来和jvm交互：
- jvm中一条数据序列化
- 序列化的数据发送到python进程
- 记录被python反序列化
- 记录被python处理
- 结果被python序列化
- 结果被发送到jvm
- jvm反序列化并存储结果到dataframe

所以python udf会比java和scala原生的udf慢。
但是使用pandas udf可以克服数据传输中需要的序列化问题，关键是使用了Arrow. spark使用arrow把JVM中的Dataframe转为可共享的buffer, 然后python也可以把这块共享buffer作为pandas的dataframe, 所以python可以直接在共享内存上操作。
以上，我们总结一下，使用arrow主要有两个好处：
1. 因为直接使用了共享内存，不在需要python和jvm序列化和反序列化数据。
2. pandas有很多使用c实现的方法， 可以直接使用。

## Spark DataFrame和Pandas DataFrame的转化
首先需要配置spark, 设置`spark.sql.execution.arrow.pyspark.enabled`, 默认这个选项是不打开的。
还可以开启`spark.sql.execution.arrow.pyspark.fallback.enabled`来避免如果没有安装`Arrow`或者其它相关错误。
Spark可以使用`toPandas()`方法转化为Pandas DataFrame; 而使用`createDataFrame(pandas_df)`把Pandas DataFrame转为Spark DataFrame.
```
import numpy as np
import pandas as pd

spark.conf.set('spark.sql.execution.arrow.pyspark.enabled', 'true')

pdf = pd.DataFrame(np.random.rand(100,3))

df = spark.createDataFrame(pdf)

# 使用arrow把spark df转化为pandas df
result_pdf = df.select("*").toPandas()
```

## Pandas UDF(矢量UDF)
`Pandas UDF`是用户定义的函数， Spark是用arrow传输数据并用pandas来运行`pandas UDF`， `pandas UDF`使用向量计算，相比于旧版本的`row-at-a-time`python udf, 最多增加100倍的性能. 使用`pandas_udf`修饰器装饰函数，就可以定义一个`pandas UDF`.对spark来说，UDF就是一个普通的pyspark函数。
从spark3.0开始， 推荐使用python类型(`type hint`)来定义pandas udf.
定义类型的时候，`StructType`需要使用`pandas.DataFrame`类型， 其他一律使用`pandas.Series`类型。
```
import pandas as pd
from pyspark.sql.functions import pandas_udf

@pandas_udf("col1 string, col2 long")
def func(s1: pd.Series, s2: pd.Series, s3: pd.DataFrame) -> pd.DataFrame:
  s3['col2'] = s1+s2.str.len()
  s3['col1'] = 'sss'
  return s3
  
df = spark.createDataFrame(
      [[1, "a string", ("a nested string",)]],
      "long_col long, string_col string, struct_col struct<col1:string>")
      
df.printSchema()

df.select(func("long_col", "string_col", "struct_col")).printSchema()

df.select(func("long_col", "string_col", "struct_col")).show()     
+--------------------------------------+
|func(long_col, string_col, struct_col)|
+--------------------------------------+
|                              [sss, 9]|
+--------------------------------------+
```

### Series to Series 类型的UDF
当类型提示可以被表达为`pandas.Series -> pandas.Series`时，称为`Series to Series`UDF
这种类型的`pandas UDF`的输入和输出必须要有相同的长度， PySpark会把数据按列分成多个batch, 然后对每个batch运行`pandas UDF`, 然后组合各自的结果。
```
>>> import pandas as pd
>>> from pyspark.sql.functions import col, pandas_udf
>>> from pyspark.sql.types import LongType
>>> def multiply_func(a: pd.Series, b:pd.Series) -> pd.Series:
...     return a*b
... 
>>> multiply = pandas_udf(multiply_func, returnType=LongType())
>>> x = pd.Series([1,3,4,5])
>>> df = spark.createDataFrame(pd.DataFrame(x, columns=['x']))
>>> df.select(multiply(col('x'),col('x'))).show()
+-------------------+
|multiply_func(x, x)|
+-------------------+
|                  1|
|                  9|
|                 16|
|                 25|
+-------------------+
```

### Series迭代器 -> Series迭代器 类型的UDF
当类型提示可以被表达为`Iterator[pandas.Series] -> Iterator[pandas.Series]`时，称为`Iterator[Series] to Iterator[Series]`UDF.
```
from typing import Iterator
import pandas as pd
from pyspark.sql.functions import pandas_udf

>>> pdf = pd.DataFrame([1,2,3], columns=['x'])
>>> df = spark.createDataFrame(pdf)
>>> df
DataFrame[x: bigint]
>>> @pandas_udf('long')
... def plus_one(iterator: Iterator[pd.Series]) -> Iterator[pd.Series]:
...     for x in iterator:
...             yield x+1
... 
>>> df.select(plus_one('x')).show()
+-----------+
|plus_one(x)|
+-----------+
|          2|
|          3|
|          4|
+-----------+
```

### 多个Series迭代器 -> Series迭代器 类型的UDF
当类型提示可以被表达为`Iterator[Tuple[pandas.Series,...]] -> Iterator[pandas.Series]`时，称为`Iterator[Tuple[pandas.Series,...]] to Iterator[Series]`UDF.

```
>>> from typing import Iterator, Tuple
>>> @pandas_udf('long')
... def multiply_two_cols(
...     iterator: Iterator[Tuple[pd.Series, pd.Series]]) -> Iterator[pd.Series]:
...     for a,b in iterator:
...             yield a*b
... 

>>> df.select(multiply_two_cols('x','x')).show()
+-----------------------+
|multiply_tow_cols(x, x)|
+-----------------------+
|                      1|
|                      4|
|                      9|
+-----------------------+
```

### Series -> Scalar 类型的UDF
当类型提示可以被表达为`pandas.Series -> Scalar`时，称为`Series to Scalar`UDF.
Scalar具体的类型必须是原生python类型如int, float等等， 或者是numpy的数据类型如numpy.int64, numpy.float64
这种UDF可以被用于`groupBy(), agg(), pyspark.sql.Window`.
```
>>> from pyspark.sql import Window

>>> df = spark.createDataFrame([(1,1.0), (1,2.0),(2,3.0),(2,4.0),(2,10.0)], ('id','v'))
>>> df
DataFrame[id: bigint, v: double]
>>> @pandas_udf('double')
... def mean_udf(v: pd.Series) -> float:
...     return v.mean()
... 
>>> df.select(mean_udf('v')).show()
+-----------+
|mean_udf(v)|
+-----------+
|        4.0|
+-----------+

>>> df.groupby('id').agg(mean_udf('v')).show()
+---+-----------------+
| id|      mean_udf(v)|
+---+-----------------+
|  1|              1.5|
|  2|5.666666666666667|
+---+-----------------+

>>> df
DataFrame[id: bigint, v: double]
>>> df.show()
+---+----+
| id|   v|
+---+----+
|  1| 1.0|
|  1| 2.0|
|  2| 3.0|
|  2| 4.0|
|  2|10.0|
+---+----+
>>> w = Window.partitionBy('id').rowsBetween(Window.unboundedPreceding, Window.unboundedFollowing)
>>> df.withColumn('mean_v', mean_udf('v').over(w)).show()
+---+----+-----------------+                                                    
| id|   v|           mean_v|
+---+----+-----------------+
|  1| 1.0|              1.5|
|  1| 2.0|              1.5|
|  2| 3.0|5.666666666666667|
|  2| 4.0|5.666666666666667|
|  2|10.0|5.666666666666667|
+---+----+-----------------+
```

## Spark的Pandas函数API
Spark有一些函数可以让python的函数通过pandas实例直接用在spark dataframe上。内部机制上类似`pandas udf`, jvm把数据转成arrow的buffer, 然后pandas可以直接在buffer上操作。但是区别是，这些函数api使用起来就像普通pyspark api一样是作用在dataframe上的, 而不像udf那样作用于一个`column`. 实际使用的时候，一般是`DataFrame.groupby().applyInPandas()`或者`DataFrame.groupby().mapInPandas()`

### Grouped Map api
Spark的dataframe在`groupby`后使用普通的pandas函数， 如`df.groupby().applyInPandas(func, schema))`， 普通的pandas函数需要输入是pandas dataframe, 返回普通的pandas dataframe. 上面这写法会把每个分组group映射到pandas dataframe.
`df.groupby().applyInPandas(func, schema))`过程其实分为三步， 典型的`split-apply-combine`模式：
- `DataFrame.groupBy`分组数据
- 分组的数据映射到pandas dataframe后，apply到传入的函数
- 组合结果成一个新的pyspark Dataframe
使用groupBy().applyInPandas(), 用户需要做两件事：
- 写好pandas函数
- 定义好pyspark dataframe结果的schema
```
>>> def subtract_mean(pdf):
...     v = pdf.v
...     return pdf.assign(v=v-v.mean())
... 
>>> df.groupby('id').applyInPandas(subtract_mean,schema='id long, v double').show()
+---+------------------+                                                        
| id|                 v|
+---+------------------+
|  1|              -0.5|
|  1|               0.5|
|  2|-2.666666666666667|
|  2|-1.666666666666667|
|  2| 4.333333333333333|
+---+------------------+
```

### Map api
也可以对pyspark dataframe和pandas dataframe做map操作，`DataFrame.mapInPandas()`是对当前的DataFrame的取一个迭代器映射普通到pandas函数。这个普通pandas函数必须是输入输出都是pdf.
```
>>> def filter_func(iterator):
...     for pdf in iterator:
...             yield pdf[pdf.id == 1]
... 
>>> df.mapInPandas(filter_func, schema=df.schema).show()
+---+---+
| id|  v|
+---+---+
|  1|1.0|
|  1|2.0|
+---+---+
```

### Co-grouped Map api
这个api可以使两个pyspark dataframe组合后使用pandas函数
```
import pandas as pd

df1 = spark.createDataFrame(
    [(20000101, 1, 1.0), (20000101, 2, 2.0), (20000102, 1, 3.0), (20000102, 2, 4.0)],
    ("time", "id", "v1"))

df2 = spark.createDataFrame(
    [(20000101, 1, "x"), (20000101, 2, "y")],
    ("time", "id", "v2"))

def asof_join(l, r):
    return pd.merge_asof(l, r, on="time", by="id")

df1.groupby("id").cogroup(df2.groupby("id")).applyInPandas(
    asof_join, schema="time int, id int, v1 double, v2 string").show()
# +--------+---+---+---+
# |    time| id| v1| v2|
# +--------+---+---+---+
# |20000101|  1|1.0|  x|
# |20000102|  1|3.0|  x|
# |20000101|  2|2.0|  y|
# |20000102|  2|4.0|  y|
# +--------+---+---+---+
```
