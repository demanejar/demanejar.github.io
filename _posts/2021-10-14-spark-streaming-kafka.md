---
title: Spark Streaming với Kafka
author: trannguyenhan 
date: 2021-10-14 20:52:00 +0700
categories: [Hadoop & Spark]
tags: [Bigdata, Spark, Kafka, Kafka Consumer]
math: true
mermaid: true
image:
  src: https://raw.githubusercontent.com/demanejar/image-collection/main/SparkStreamingKafka/streaming-arch.png
---
Trong 2 bài ví dụ về [Spark Streaming](https://demanejar.github.io/posts/socket-stream/) trước thì mình đã minh họa về Spark Streaming nhận dữ liệu qua socket và xử lý chúng. Tuy nhiên, trong thực tế thì ít khi chúng ta sử dung socket để truyền và xử lý dữ liệu thay vào đó chúng ta sẽ thường sử dụng các hàng đợi tin nhắn (Message Queue) mà tiêu biểu nhất ở đây chính là Kafka.

Để tìm hiểu nhanh về Kafka cũng như các cách sử dụng của chúng (tạo topic, xem topic, tạo producer, tạo consumer) thì bạn có thể xem nhanh tại đây: [https://kafka.apache.org/quickstart](https://kafka.apache.org/quickstart)

Trong bài viết này chúng ta sẽ đi làm một project nhỏ để hiểu luồng hoạt động của Kafka-Spark sử dụng Spark Streaming nhận dữ liệu từ Kafka và in chúng ra màn hình.

## Chuẩn bị Kafka

Trước hết chúng ta cần bật Kafka và tạo 1 topic đặt tên là "topic-1" bằng câu lệnh: 

```bash
bin/kafka-topics.sh --create --topic topic-1 --bootstrap-server localhost:9092
```

Tạo một _producer_ để sẵn sàng gửi dữ liệu với câu lệnh:

```bash
bin/kafka-console-producer.sh --topic topic-1 --bootstrap-server localhost:9092
```

## Chuẩn bị project

Đã xong kafka giờ chúng ta sẽ chuẩn bị một project bằng Spark Streaming để đọc dữ liệu được lưu trong kafka. 

Cấu trúc project gồm 2 package `demo.sparkstreaming` chứa hàm `main` và `demo.sparkstreaming.properties` chứa các cấu hình của Spark Streaming Consumer:

```
- SparkStreamingDemo: 
	- demo.sparkstreaming
		- StreamingConsumer.java
	- demo.sparkstreaming.properties
		- SSKafkaProperties.java
```

File `SSKafkaProperties` chứa những cấu hình của Spark Streaming Consumer: 

```java
package demo.sparkstreaming.properties;

import java.util.HashMap;
import java.util.Map;

import org.apache.kafka.common.serialization.StringDeserializer;

public class SSKafkaProperties {
	/**
	 * Define configuration kafka consumer with spark streaming
	 * @return
	 */
	public static Map<String, Object> getInstance() {
        Map<String, Object> kafkaParams = new HashMap<String, Object>();
        kafkaParams.put("bootstrap.servers", "localhost:9092");
        kafkaParams.put("key.deserializer", StringDeserializer.class);
        kafkaParams.put("value.deserializer", StringDeserializer.class);
        kafkaParams.put("group.id", "0");
        kafkaParams.put("auto.offset.reset", "earliest");
        kafkaParams.put("enable.auto.commit", false);
        
        return kafkaParams;
	}
}
```

File `StreamingConsumer` chứa hàm `main`: 

```java
package demo.sparkstreaming;

import java.util.Arrays;
import java.util.Collection;
import java.util.List;

import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.function.Function;
import org.apache.spark.streaming.Durations;
import org.apache.spark.streaming.api.java.JavaDStream;
import org.apache.spark.streaming.api.java.JavaInputDStream;
import org.apache.spark.streaming.api.java.JavaStreamingContext;
import org.apache.spark.streaming.kafka010.ConsumerStrategies;
import org.apache.spark.streaming.kafka010.KafkaUtils;
import org.apache.spark.streaming.kafka010.LocationStrategies;

import demo.sparkstreaming.properties.SSKafkaProperties;

public class StreamingConsumer {
	public static void main(String[] args) throws InterruptedException {
		SparkConf conf = new SparkConf().setMaster("local").setAppName("Spark Streaming Consumer");
		JavaStreamingContext jssc = new JavaStreamingContext(conf, Durations.seconds(1));
		
		// Define a list of Kafka topic to subscribe
		Collection<String> topics = Arrays.asList("topic-1");
		
		// Create an input DStream which consume message from Kafka topics
		JavaInputDStream<ConsumerRecord<String, String>> stream;
		stream = KafkaUtils.createDirectStream(jssc, 
				LocationStrategies.PreferConsistent(), 
				ConsumerStrategies.Subscribe(topics, SSKafkaProperties.getInstance()));
		
		JavaDStream<String> lines = stream.map((Function<ConsumerRecord<String,String>, String>) kafkaRecord -> kafkaRecord.value());
		lines.cache().foreachRDD(line -> {
			List<String> list = line.collect();
			if(line != null) {
				for(String l: list) {
					System.out.println(l);
				}
			}
		});
		
		// Start the computation
        jssc.start();
        jssc.awaitTermination();
	}
}
```

## Chạy project

Build project thành file `.jar`: 

```bash
mvn clean package
```

Chạy file `.jar` vừa build trên Spark với `spark-submit`: 

```bash
spark-submit --class demo.sparkstreaming.StreamingConsumer target/SparkStreamingDemo-V1-jar-with-dependencies.jar
```

Truyền dữ liệu vào thông qua `producer` mà bạn vừa tạo ở phần 1 và xem kết quả ở màn hình terminal bạn vừa chạy `spark-submit`.

Toàn bộ project bạn có thể xem tại [https://github.com/demanejar/spark-streaming-kafka-demo](https://github.com/demanejar/spark-streaming-kafka-demo)
