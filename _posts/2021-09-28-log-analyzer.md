---
title: Project Log Analyzer với Spark Streaming
author: trannguyenhan 
date: 2021-09-28 20:52:00 +0700
categories: [Hadoop & Spark]
tags: [Spark Streaming, Bigdata, Spark]
math: true
mermaid: true
image:
  src: https://raw.githubusercontent.com/demanejar/image-collection/main/SparkStreaming/streaming-arch.png
---
*Bài viết trước chúng ta đã làm quen với Spark Streaming với một project đơn giản về lọc từ, trong bài viết này chúng ta sẽ xem xét project phức tạp hơn một tí về phân tích log.*

## Chuẩn bị Project

Toàn bộ project các bạn có thể xem tại: [https://github.com/demanejar/logs-analyzer](https://github.com/demanejar/logs-analyzer), mọi người clone project về để chuẩn bị chạy nha.

Mình sẽ đi qua giải thích từng file để mọi người hiểu hơn về project, đầu tiên là file đầu vào `log.txt`, file này chứa 1546 dòng log, các log đều có cấu trúc giống nhau, công việc của project này chính là nhận đầu vào từ file log này và đưa ra các phân tích, thống kê về trên tập log này.

File `stream.sh` là một file shell script với nhiệm vụ là đọc dữ liệu từ file `log.txt` và đẩy chúng qua cổng 9999.

File `build.sbt` thì giống với project trước, file này để khai báo các thư viện, phụ thuộc để có thể build project thành một file `.jar`

Cuối cùng là project của chúng ta viết bằng `scala` với 2 file `ApacheAccessLog.scala` và `LogAnalyzerStreaming.scala`. File `LogAnalyzerStreaming.scala` sẽ lắng nghe ở cổng 9999, lấy dữ liệu và tiến hành tổng hợp, phân tích chúng.

Lưu ý: nếu bạn chạy cụm nhiều máy, hãy đổi địa chỉ hostname trong file `LogAnalyzerStreaming.scala` thành địa chỉ máy master nha.

## Chạy project và xem kết quả

Khởi động Spark và kiểm tra địa chỉ của master thông qua cổng 8080 (`spark://PC0628:7077`):

![](https://raw.githubusercontent.com/demanejar/image-collection/main/LogAnalyzer/8080.png)

Build project vừa rồi thành file `.jar` với câu lệnh dưới đây, lần đầu build có thể các bạn sẽ phải đợi khá lâu vì nó phải tải xuống các thư viện, trong các lần build sau sẽ nhanh hơn:

```bash
sbt clean package
```

![](https://raw.githubusercontent.com/demanejar/image-collection/main/LogAnalyzer/sbt_clean_package.png)

Chạy spark-submit với file jar vừa bulid được:

```
spark-submit --master spark://PC0628:7077 --class LogAnalyzerStreaming target/scala-2.12/log-analyzer_2.12-0.0.1.jar
```

-   Tham số  `master`  là địa chỉ của master mà bạn vừa lấy ở bên trên
-   Tham số  `class`  là đường dẫn tới hàm main của project

Trong bài viết trước, chúng ta đã gửi dữ liệu thông qua socket bằng tay, trong bài này vẫn là socket tuy nhiên  chúng ta sẽ viết một chương trình để chúng tự động gửi dữ liệu và chúng ta chỉ việc ngồi chờ kết quả thôi.

Như mình giải thích ở ban đầu, file `stream.sh` là một file shell script với nhiệm vụ là đọc dữ liệu từ file `log.txt` và đẩy chúng qua cổng 9999. Sử dụng câu lệnh sau để bắt đầu chạy: 

```
./stream.sh log.txt
```

- Với `log.txt` viết ở phía sau thể hiện `log.txt` là một tham số đầu vào của chương trình viết trong file `.stream.sh`

Kết quả in ra màn hình cũng sẽ vụt qua rất nhanh, các bạn có thể kéo terminal lên để nhìn 

![](https://raw.githubusercontent.com/demanejar/image-collection/main/LogAnalyzer/start_project.png)

Mở cổng 4040 để xem lại chi tiết các job vừa thực hiện (`localhost:4040`):

![](https://raw.githubusercontent.com/demanejar/image-collection/main/LogAnalyzer/4040.png)

Tham khảo: [databricks](https://github.com/databricks/reference-apps/tree/master/logs_analyzer)
