1. Building Apache Spark
> https://people.apache.org/~pwendell/spark-nightly/spark-branch-2.0-docs/latest/building-spark.

```
$ export MAVEN_OPTS="-Xmx2g -XX:ReservedCodeCacheSize=512m"                                                                      
# build
$ ./build/mvn -Pyarn -Phadoop-2.4 -Dhadoop.version=2.4.0 -DskipTests clean package
# building a runnable distribution
$ ./dev/make-distribution.sh --name custom-spark --tgz -Psparkr -Phadoop-2.4 -Phive -Phive-thriftserver -Pyarn
```

2. open with IntelliJ(IDEA)
```
git clone git@github.com:2L-knowledgebase/spark-adaptive.git

File => New => Project from Existing Source
Maven
checked "Import Maven projects automaitcally" + Automatically download: "Sources" + "Documentation"
select profiles as you need
"Next" to end

Wait for resolving dependencies "downloading"
```


3. compile with CDH, https://github.com/genghuiluo/spark-adaptive/tree/cdh

```
add cloudera repo in pom.xml under root directory:

<repository>
        <id>cloudera</id>
        <name>cloudera Repository</name>
        <url>https://repository.cloudera.com/artifactory/cloudera-repos/</url>
</repository>

verify your CDH hadoop version:
$ hadoop version
Hadoop 2.6.0-cdh5.14.2
Subversion http://github.com/cloudera/hadoop -r 5724a4ad7a27f7af31aa725694d3df09a68bb213
Compiled by jenkins on 2018-03-27T20:40Z
Compiled with protoc 2.5.0
From source with checksum 302899e86485742c090f626a828b28
This command was run using /opt/cloudera/parcels/CDH-5.14.2-1.cdh5.14.2.p0.3/jars/hadoop-common-2.6.0-cdh5.14.2.jar

./build/mvn -Pyarn -Phadoop-2.6 -Dhadoop.version=2.6.0-cdh5.14.2 -DskipTests clean package
./dev/make-distribution.sh --name custom-spark --tgz -Psparkr -Phadoop-2.6 -Phive -Phive-thriftserver -Pyarn
```

### refs:

1. [CSDN: spark2.3.0 源码编译，一次成功](https://blog.csdn.net/qq_27882063/article/details/79991376)
2. [CSDN: spark2.1源码编译](https://blog.csdn.net/babyhuang/article/details/78656093)
3. [Spark 官方文档翻译——《Spark编译》](http://www.voidcn.com/article/p-yjdepyvk-ma.html)
4. [IntelliJ IDEA 导入 spark 源码 步骤](https://blog.csdn.net/haohaixingyun/article/details/60968776)
