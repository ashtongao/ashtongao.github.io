---
layout:     post
title:      Stark Spark
subtitle:   Spark的安装以及部署
date:       2018-12-18
author:     Ashton
header-img: img/post-bg-os-metro.jpg
catalog: true
tags:
    - Spark
    - Real-Time application
    - Ditributed System
---

## 安装Spark

[Spark官方安装包](https://www.apache.org/dyn/closer.lua/spark/spark-2.4.0/spark-2.4.0-bin-hadoop2.7.tgz)

下载spark-2.4.0版本

```
wget http://mirrors.tuna.tsinghua.edu.cn/apache/spark/spark-2.4.0/spark-2.4.0-bin-hadoop2.7.tgz

tar -xvf spark-2.4.0-bin-hadoop2.7.tgz
```

![](img/2018-12-19-Start%20Spark/2018-12-18-Start%20Spark-20181218112052.png)

Spark依赖Hadoop client库，主要是要使用到HDFS以及YARN，下载的包里面已经预先打包了对应模块

![](img/2018-12-19-Start%20Spark/2018-12-18-Start%20Spark-20181220093131.png)

## 通过命令行Shell运行

Spark自带了很多example，在目录`examples/src/main`中，可以使用`bin/run-example <class>`来运行对应的example

```
./bin/run-example SparkPi 10
```

![](img/2018-12-19-Start%20Spark/2018-12-18-Start%20Spark-20181218112819.png)


也可以通过Spark-Shell来运行Spark

```
./bin/spark-shell --master local[2]

选项说明
--master 用于指定分布式集群的URL
local[2] 代表本地运行，同时开启2个线程
```

运行后会看到Spark-Shell接口，通过Scala语言来交互

![](img/2018-12-19-Start%20Spark/2018-12-18-Start%20Spark-20181218113335.png)

## 实践：提交一个Spark任务

目标：分别统计a、b字母在文件中的出现的次数

```
mkdir -p src/main/java
vi src/main/java/SimpleApp.java
```

```java
/* SimpleApp.java */
import org.apache.spark.sql.SparkSession;
import org.apache.spark.sql.Dataset;

public class SimpleApp {
  public static void main(String[] args) {
    // YOUR_SPARK_HOME需要更换成上面下载后的spark目录，如
    // /root/report/spark-2.4.0-bin-hadoop2.7/README.md
    String logFile = "YOUR_SPARK_HOME/README.md";
    SparkSession spark = SparkSession.builder().appName("Simple Application").getOrCreate();
    Dataset<String> logData = spark.read().textFile(logFile).cache();

    // 注意lambda这里需要使用JDK 8
    long numAs = logData.filter(s -> s.contains("a")).count();
    long numBs = logData.filter(s -> s.contains("b")).count();

    System.out.println("Lines with a: " + numAs + ", lines with b: " + numBs);

    spark.stop();
  }
}

```

使用Maven构建

```xml
<project>
  <groupId>edu.berkeley</groupId>
  <artifactId>simple-project</artifactId>
  <modelVersion>4.0.0</modelVersion>
  <name>Simple Project</name>
  <packaging>jar</packaging>
  <version>1.0</version>
  <dependencies>
    <dependency> <!-- Spark dependency -->
      <groupId>org.apache.spark</groupId>
      <artifactId>spark-sql_2.11</artifactId>
      <version>2.4.0</version>
    </dependency>
  </dependencies>
</project>
```

最终的目录结构

```
$ find .
./pom.xml
./src
./src/main
./src/main/java
./src/main/java/SimpleApp.java
```

执行打包

```
mvn package
...
[INFO] Building jar: {..}/{..}/target/simple-project-1.0.jar
```



```shell
如果提示：

error: lambda expressions are not supported in -source 1.5

给Maven指定更高版本的JDK即可（JDK 8），没有则忽略
```



接下来启动我们的Spark程序

```
YOUR_SPARK_HOME/bin/spark-submit \
  --class "SimpleApp" \
  --master local[4] \
  target/simple-project-1.0.jar
...
Lines with a: 46, Lines with b: 23
```

可以看到最终的运行结果

![](img/2018-12-19-Start%20Spark/2018-12-18-Start%20Spark-20181220091455.png)


## Reference

[Spark Overview](https://spark.apache.org/docs/latest/index.html)
[Submitting Applications](https://spark.apache.org/docs/latest/submitting-applications.html)
[Quick Start](https://spark.apache.org/docs/latest/quick-start.html)




















































































