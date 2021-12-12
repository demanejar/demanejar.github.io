---
title: Project Socket Stream với Spark Streaming
author: trannguyenhan 
date: 2021-09-23 20:52:00 +0700
categories: [Hadoop & Spark]
tags: [Spark Streaming, Bigdata, Spark]
math: true
mermaid: true
image:
  src: https://raw.githubusercontent.com/demanejar/image-collection/main/SparkStreaming/streaming-arch.png
---
*Trong bài viết này chúng ta sẽ đi xét một ví dụ nhỏ với Spark Streaming. Công việc của chúng ta là tạo một project với Spark Streaming lắng nghe ở cổng 7777 và lọc những dòng có chứa từ "error" rồi in chúng ra.* 

## Chuẩn bị Project

Project rất đơn giản, được viết bằng Scala. Bạn có thể xem toàn bộ project tại đây: [https://github.com/demanejar/socket-stream](https://github.com/demanejar/socket-stream), mọi người clone project về để chuẩn bị chạy nha.

Trong project có 2 file chính, một là file `SocketStream.scala`: 
```scala
import org.apache.spark._
import org.apache.spark.SparkContext._
import org.apache.spark.streaming.StreamingContext
import org.apache.spark.streaming.StreamingContext._
import org.apache.spark.streaming.dstream.DStream
import org.apache.spark.streaming.Duration
import org.apache.spark.streaming.Seconds

object SocketStream {
	def main(args : Array[String]){
		val conf = new SparkConf().setAppName("Socket-Stream")
		val ssc = new StreamingContext(conf, Seconds(1))
		val lines = ssc.socketTextStream("localhost", 7777)
		val errorLines = lines.filter(_.contains("error"))
		errorLines.print()
		ssc.start()
		ssc.awaitTermination()
	}
}
```
- File này có nhiệm vụ là tạo một `StreamingContext` lắng nghe ở cổng 7777, lọc những dòng có chứa từ `error` và in chúng ra màn hình.
- Đây là lần đầu tiên mình minh họa Spark với Scala, với những lần trước mình đều minh họa bằng mã Java. Các bạn có thể thấy Scala hao hao giống với Java xong lại có cách viết thì giống với Python. Scala nó như là ngôn ngữ sinh ra để cho Spark rồi ấy, thế nên là ngôn ngữ này có rất nhiều những cái hỗ trợ khi bạn dùng nó để code Spark. 

File còn lại là file `build.sbt`: 

```sbt
name := "socket-stream"
version := "0.0.1"
scalaVersion := "2.11.12"
libraryDependencies ++= Seq(
"org.apache.spark" %% "spark-core" % "2.4.1" % "provided",
"org.apache.spark" %% "spark-streaming" % "2.4.1"
)
```

- Để có thể submit job này với spark-submit thì bạn phải build chúng thành một file `.jar`, với scala thì chúng ta sẽ sử dụng `sbt`
- Nếu máy bạn chưa có `sbt` có thể tham khảo cách cài đặt tại đây: [https://www.scala-sbt.org/download.html](https://www.scala-sbt.org/download.html)

## Chạy Project và xem kết quả

Khởi động Spark và kiểm tra địa chỉ của master thông qua cổng 8080 (`spark://PC0628:7077`): 

![](https://raw.githubusercontent.com/demanejar/image-collection/main/SocketStream/socket_stream.png)

Build project vừa rồi thành file `.jar` với câu lệnh dưới đây, lần đầu build có thể các bạn sẽ phải đợi khá lâu vì nó phải tải xuống các thư viện, trong các lần build sau sẽ nhanh hơn: 

```bash
sbt clean package
```

![](https://raw.githubusercontent.com/demanejar/image-collection/main/SocketStream/sbt_clean_package.png)

Chạy spark-submit với file jar vừa bulid được: 
```
spark-submit --master spark://PC0628:7077 --class SocketStream target/scala-2.11/socket-stream_2.11-0.0.1.jar
```

- Tham số `master` là địa chỉ của master mà bạn vừa lấy ở bên trên
- Tham số `class` là đường dẫn tới hàm main của project

Mở một terminal khác lên và chạy lệnh sau để bắt đầu gửi text qua cổng 7777: 

```
nc -l 7777
```

Kết quả in ra màn hình và sẽ vụt qua rất nhanh: 

![](https://raw.githubusercontent.com/demanejar/image-collection/main/SocketStream/result_1.png)

![](https://raw.githubusercontent.com/demanejar/image-collection/main/SocketStream/result_2.png)

Mở cổng 4040 để xem lại chi tiết các job vừa thực hiện (`localhost:4040`): 

![](https://raw.githubusercontent.com/demanejar/image-collection/main/SocketStream/4040.png)

Tham khảo: sách Learning Spark - Zaharia M., et al. (trang 184)
