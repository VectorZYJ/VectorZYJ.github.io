---
title: test
date: 2024-09-28 20:33:43
tags:
---

### This is a test file

`gcloud compute ssh nyu-dataproc-m --project hpc-dataproc-19b8 --zone us-central1-f`

`spark-shell --deploy-mode client`
# Lab1
```
gcloud compute ssh nyu-dataproc-m --project hpc-dataproc-19b8 --zone us-central1-f
hadoop fs -ls /
hadoop fs -ls
hadoop fs -ls /user/yz10317_nyu_edu
hadoop fs -mkdir lab1
hadoop fs -ls
hadoop fs -getfacl lab1
wget https: //www.gutenberg.org/files/1661/1661-0.txt
hadoop fs -put 1661-0.txt lab1/data.txt
hadoop fs -ls lab1
hadoop fs -get lab1/data.txt
diff 1661-0.txt data.txt
hadoop fs -rm -r lab1
hadoop fs -ls
spark-shell --deploy-mode client
```
```Scala
:help
sc.\tab
sc.version
val myConstant: Int = 2437
myConstant
myConstant.\tab
myConstant.to\tab
myConstant.toFloat
myConstant
myConstant.toFloat.toInt
val myString = myConstant.toString
:type myString
:q
```
# Lab 2
```
gcloud compute ssh nyu-dataproc-m --project hpc-dataproc-19b8 --zone us-central1-f
spark-shell --deploy-mode client
```
```Scala
val exchangeRate: Double = 1.04
val dollars: Int = 100.00
val dollars: Double = 100.00
var euros = 0.00
euros = dollars * exchangeRate
dollars = 500
val dollars = 500
var eurosInt: Int = 0
eurosInt = dollars * exchangeRate
eurosInt = (dollars * exchangeRate).toInt
exchangeRate.getClass
dollars.getClass
euros.getClass
eurosInt.getClass
println(f"September 2023: $$$dollars = $eurosInt euros")


val record = "2023-09-25:19:10:00, 12345678-aaaa-1000-gggg-000111222333, 58, TRUE, enabled, disabled, 37.819722,-122.478611"
record.length
record.contains("disabled")
record.indexOf(',')
record.toLowerCase.indexOf("true")
record
var record2 = record
record == record2
record2 = "something else"
record == record2
```
```
hdfs dfs -mkdir loudacre
hdfs dfs -mkdir loudacre/weblog
//download 2014-03-15.log and upload it to nyu-dataproc-hdfs-ingest
hadoop distcp gs://nyu-dataproc-hdfs-ingest/2014-03-15.log loudacre/weblog
spark-shell --deploy-mode client
```
```Scala
val logfile: String = "/user/yz10317_nyu_edu/loudacre/weblog/2014-03-15.log"
val originalRdd = sc.textFile(logfile)
originalRdd.take(5)
val jpgRdd = originalRdd.filter(line => line.contains("jpg"))
jpgRdd.take(5)
originalRdd.filter(line => line.contains("jpg")).count()
val ipRdd = originalRdd.map(line => line.substring(0, line.indexOf('-')-1))
ipRdd.saveAsTextFile("loudacre/iplist")
:q
```
```
hdfs dfs -ls loudacre/iplist
hdfs dfs -cat loudacre/iplist/part-00000
```

# Lab 3
```Scala
val data = sc.textFile("loudacre/devstatus/devicestatus.txt")
val splitedData = data.map(line => line.split(line(19)))
val filteredData = splitedData.filter(line => line.length == 14)
val extractedData = filteredData.map(line => Array(line(0), line(1), line(2), line(12), line(13)))
def modify(line: Array[String]) = {
	line(1) = line(1).split(' ')(0)
	line
}
val modifiedData = extractedData.map(modify)
val finalData = modifiedData.map(line => line.mkString(","))
finalData.saveAsTextFile("loudacre/devstatus/devicestatus_etl")
```

# Lab 4
```Scala
val requestCountsRdd = (
	sc.textFile("loudacre/weblog/2014-03-15.log")
	.map(_.split(' ')(2).toInt -> 1)
	.reduceByKey(_ + _)
	.sortByKey()
)

requestCountsRdd.take(10)

val visitFrequencyRdd = (
	sc.textFile("loudacre/weblog/2014-03-15.log")
	.map(_.split(' ')(2).toInt -> 1)
	.reduceByKey(_ + _)
	.map(_._2 -> 1)
	.reduceByKey(_ + _)
	.sortByKey()
)

visitFrequencyRdd.collect()

val userIpList = (
	sc.textFile("loudacre/weblog/2014-03-15.log", 4)
	.map(line => line.split(' ')(2).toInt -> line.split(' ')(0))
	.distinct()
	.groupByKey()
	.sortByKey()
	.mapValues(_.toArray)
	.map(tuple => tuple._1.toString + ": " + tuple._2.mkString(" "))
)

userIpList.saveAsTextFile("loudacre/useriplist")

:q
```
```
hdfs dfs -ls loudacre/useriplist
hdfs dfs -cat loudacre/useriplist/part-00000 | head -5
hdfs dfs -get loudacre/useriplist .

./google-cloud-sdk/bin/gcloud init
gcloud compute scp --recurse nyu-dataproc-m:~/useriplist ~
```

# Lab 5
```Scala
import scala.xml._

val resultRdd = {
	sc.wholeTextFiles("loudacre/activations/*.xml") // RDD[(String, String)]
	.map(x => XML.loadString(x._2) \ "activation") // RDD[scala.xml.NodeSeq]
	.flatMap(x => x) // RDD[scala.xml.Node]
	.map(x => (x \ "account-number").text -> (x \ "model").text)
	.map(x => x._1.toString + ":" + x._2.toString)
}

resultRdd.saveAsTextFile("loudacre/account-models")
spark-submit --master yarn --class AccountModel lab5/target/scala-2.12/accountmodel_2.12-1.0.0.jar
```

# Lab 6
```
hadoop distcp gs://nyu-dataproc-hdfs-ingest/iot_devices.json /user/yz10317_nyu_edu
```
```Scala
case class DeviceIoTData(
  battery_level: Long,
  co2_level: Long,
  cca2: String,
  cca3: String,
  cn: String,
  device_id: Long,
  device_name: String,
  humidity: Long,
  ip: String,
  latitude: Double,
  lcd: String,
  longitude: Double,
  scale: String,
  temp: Long,
  timestamp: Long
)
val ds = spark.read.json("iot_devices.json").as[DeviceIoTData]
ds.printSchema
ds.show(5, false)
ds.first()

val failDevices = {
	ds.select("device_id", "battery_level")
	.where($"battery_level" === 0)
}
failDevices.count()
failDevices.show(5, false)

val co2Emission = {
	ds.select("cn","co2_level")
	.groupBy("cn")
	.agg(avg("co2_level"))
	.withColumnRenamed("avg(co2_level)", "average_co2_level")
	.sort($"average_co2_level".desc)
}
co2Emission.show(5, false)
```

# Lab 7
```Scala
val data = Seq(("Alice", "111-11-1111"), ("Bob", "123-45-6789"), ("Carol", "987-65-4321"))
val df = data.toDF("name", "ssn")
df.show()

df.createOrReplaceTempView("users")
spark.sql("SELECT * FROM users").show()

val hide = (ssn: String) => { 
	"***-**-".concat(ssn.substring(7, 11))
}
spark.udf.register("hide", hide)
spark.sql("SELECT name, hide(ssn) As ssn FROM users").show()
```
