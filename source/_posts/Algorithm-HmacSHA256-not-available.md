---
title: Algorithm HmacSHA256 not available
date: 2016-04-21 16:32:01
categories:
- Development
tags:
- AmazonClientException
- Algorithm
- HmacSHA256
---

# 场景：
> **有一段代码需要访问Amazon，直接执行main方法是没问题的。可是把代码打成jar包，用ProcessBuilder执行java -jar的方式来run的时候，抛出了Exception。**

# Algorithm HmacSHA256 not available 异常信息

    Exception in thread "pool-1-thread-1" com.amazonaws.AmazonClientException: Unable to calculate a 
	request signature: Unable to calculate a request signature: Algorithm HmacSHA256 not available
	at com.amazonaws.auth.AbstractAWSSigner.sign(AbstractAWSSigner.java:81)
	at com.amazonaws.auth.AWS4Signer.computeSignature(AWS4Signer.java:289)
	at com.amazonaws.auth.AWS4Signer.sign(AWS4Signer.java:127)
	at com.amazonaws.http.AmazonHttpClient.executeOneRequest(AmazonHttpClient.java:646)
	at com.amazonaws.http.AmazonHttpClient.executeHelper(AmazonHttpClient.java:454)
	at com.amazonaws.http.AmazonHttpClient.execute(AmazonHttpClient.java:294)
	at com.amazonaws.services.dynamodbv2.AmazonDynamoDBClient.invoke(AmazonDynamoDBClient.java:3106)
	at com.amazonaws.services.dynamodbv2.AmazonDynamoDBClient.batchWriteItem(AmazonDynamoDBClient.java:771)
	at com.amazonaws.services.dynamodbv2.document.internal.BatchWriteItemImpl.doBatchWriteItem(BatchWriteItemImpl.java:111)
	at com.amazonaws.services.dynamodbv2.document.internal.BatchWriteItemImpl.batchWriteItem(BatchWriteItemImpl.java:52)
	at com.amazonaws.services.dynamodbv2.document.DynamoDB.batchWriteItem(DynamoDB.java:159)
	at com.saasbee.webapp.service.search.DocumentService.store(DocumentService.java:79)
	at com.saasbee.webapp.service.search.thread.ParseTaskThread.run(ParseTaskThread.java:97)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source)
	at java.lang.Thread.run(Unknown Source)
	Caused by: com.amazonaws.AmazonClientException: Unable to calculate a request signature: Algorithm HmacSHA256 not available
	at com.amazonaws.auth.AbstractAWSSigner.sign(AbstractAWSSigner.java:91)
	at com.amazonaws.auth.AbstractAWSSigner.sign(AbstractAWSSigner.java:79)
	... 15 more
	Caused by: java.security.NoSuchAlgorithmException: Algorithm HmacSHA256 not available
	at javax.crypto.Mac.getInstance(Mac.java:181)
	at com.amazonaws.auth.AbstractAWSSigner.sign(AbstractAWSSigner.java:87)
	... 16 more
	
<!-- more -->

# Algorithm HmacSHA256 not available 解决办法
>**猜测应该是Amazon的security有关方面的问题。最终我的解决方法如下：**  
>**在pom.xml里面加入${JAVA_HOME}/jre/lib/sunjce_provider.jar的依赖，然后重新package，异常消失。**

	<dependency>
        <groupId>org.sun</groupId>
        <artifactId>sunjce_provider</artifactId>
        <scope>system</scope>
        <version>1.0</version>
        <systemPath>${JAVA_HOME}/jre/lib/sunjce_provider.jar</systemPath>
    </dependency>

>**缺点：要求调用这个jar的程序的jre版本与生成这个jar的jre版本一致。**