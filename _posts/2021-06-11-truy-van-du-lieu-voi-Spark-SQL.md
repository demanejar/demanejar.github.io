---
title: Truy vấn dữ liệu với Spark SQL sử dụng Java 
author: trannguyenhan 
date: 2021-06-11 20:52:00 +0700
categories: [Blogging, Share]
tags: [Spark SQL, Big data, Hadoop HDFS, Parquet file]
math: true
mermaid: true
---

***Spark SQL là một module Spark để xử lý dữ liệu có cấu trúc, mọi dữ liệu mà Spark SQL xư lý đều ở dưới dạng DataFrame và có thể hoạt động như một công cụ truy vấn SQL phân tán. Spark SQL hỗ trợ rất nhiều loại dữ liệu đầu vào như : parquet, json, csv,...***

Trong bài viết này mình sẽ viết về lần lượt các vấn đề: 
- Chuyển từ 1 file text sang file parquet và lưu trữ tất cả trên Hadoop HDFS
- Sử dụng Spark SQL lấy dữ liệu input là file parquet từ HDFS vừa chuyển từ file text và thực hiện truy vấn

Dữ liệu được sử dụng trong bài viết này mọi người có thể lấy tại [SAMPLE DATA](https://github.com/demanejar/parquet-io-spark/tree/master/sample_text) (file model log là file mô tả các trường của đối tượng trong file text)

# Chuyển dữ liệu từ 1 file text sang parquet file 
Đầu tiên là chúng ta có một tập [SAMPLE DATA](https://github.com/demanejar/parquet-io-spark/tree/master/sample_text) ở đây và tất cả đang được lưu trên HDFS ở đường dẫn `/sample_text`, chúng ta phải tạo ra 1 class phù hợp với các trường của đố i tượng này trước ( để biết đối tượng có những trường nào xem trong file model log.txt), việc tạo class để lưu trữ đối tượng này cũng khá đơn giản, các bạn có thể tham khảo tại đây : [ModelLog.class](https://github.com/demanejar/parquet-io-spark/blob/master/src/model/ModelLog.java).

Tiếp theo, chúng ta phải chuyển toàn bộ dữ liệu text kia sang parquet file, bằng cách đọc vào từng dòng trong file text, lọc ra từng thuộc tính và gán lại cho object `ModelLog` mà chúng ta vừa tạo ở trên, công việc cũng khá đơn giản: 
```
String[] tokenizer = line.split("\t");

Date timeCreate = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss").parse(tokenizer[0]);
Date cookieCreate = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss").parse(tokenizer[1]);
int browserCode = Integer.parseInt(tokenizer[2]);
String browserVer = tokenizer[3];
int osCode = Integer.parseInt(tokenizer[4]);
String osVer = tokenizer[5];
long ip = Long.parseLong(tokenizer[6]);
int locId = Integer.parseInt(tokenizer[7]);
String domain = tokenizer[8];
int siteId = Integer.parseInt(tokenizer[9]);
int cId = Integer.parseInt(tokenizer[10]);
String path = tokenizer[11];
String referer = tokenizer[12];
long guid = Long.parseLong(tokenizer[13]);
String flashVersion = tokenizer[14];
String jre = tokenizer[15];
String sr = tokenizer[16];
String sc = tokenizer[17];
int geographic = Integer.parseInt(tokenizer[18]);
String category = tokenizer[23];
String osName = tokenizer[5];

ModelLog tmpModelLog = new ModelLog(timeCreate, browserCode, browserVer, osName, osCode, osVer, ip, domain,
        path, cookieCreate, guid, siteId, cId, referer, geographic, locId, flashVersion, jre, sr, sc, category);

return tmpModelLog;
```

Theo dõi chi tiết hàm chuyển đổi từ file text sang parquet file [TẠI ĐÂY](https://github.com/demanejar/parquet-io-spark/blob/master/src/fileservices/ReadFileText.java)

Sau bước trên, chúng ta đã đọc ra được 1 `List` các `ModelLog` từ file text, và công việc bây giờ là khi chúng ra file parquet và lưu vào HDFS như sau: 
```
SparkSession spark = SparkSession.builder().appName("Write file parquet to HDFS").master("local").getOrCreate();
		
Dataset<Row> listModelLogDF = spark.createDataFrame(listModelLog, ModelLog.class);
listModelLogDF.write().parquet("hdfs://127.0.0.1:9000/usr/trannguyenhan/pageviewlog");
```

- Chúng ta tạo ra 1 biến SparkSession kết nối tới Spark của bạn và cấu hình tên và master như trên. 
- `spark.createDataFrame` sẽ giúp chuyển đổi đối tượng ModelLog thành `DataFrame` (`Dataset<Row>` chúng ta hiểu đơn giản đây chính là `DataFrame`)
- `listModelLogDF.write().parquet` sẽ giúp bạn lưu lại `DataFrame` này ra file parquet với đường dẫn được chỉ định như trên, các bạn có thể lưu ra file json hoặc csv bằng cách thay đổi phương thức `.parquet` thành `.json` hoặc `.csv` tương ứng.

# Sử dụng Spark SQL để truy vấn dữ liệu 
Như đã nói ở đầu bài là spark SQL sẽ sử lý dữ liệu dưới dạng `DataFrame`, thế nên chúng ta phải đọc toàn bộ dữ liệu dưới dạng file parquet kia thành `DataFrame` như sau: 
```
SparkSession spark = SparkSession.builder().appName("Read file parquet to HDFS").master("local").getOrCreate();
Dataset<Row> parquetFile = spark.read().parquet(
        "hdfs://127.0.0.1:9000/usr/trannguyenhan/pageviewlog");
```

### printSchema
Sử dụng `parquetFile.printSchema();` để in ra hình dạng của dữ liệu (gồm các trường nào, mỗi trường được định nghĩa kiểu dữ liệu gì), kết quả sẽ giống như hình dưới đây: <br /><br />
![](https://i.pinimg.com/564x/20/8c/55/208c554c407ca929eef33d50b25be0fa.jpg)
<br />

### Truy vấn sử dụng thuộc tính của DataFrame
Trước khi sử dụng Spark SQL với những câu lệnh khá gần gũi với SQL (Spark SQL cố xây dựng một công cụ với các sử dụng quen thuộc nhất để cho người sử dụng dễ dàng sử dụng mà gần như không phải học thêm những kiến thức mới), thì chúng ta sẽ thử sử dụng một số phương thức được cung cấp bởi `DataFrame` trước nha: 
- select, lấy ra những cột có định danh là `cId` và in ra màn hình bởi phương thức `show()`
```
parquetFile.select("cId").show();
```
Nếu muốn lấy ra nhiều cột hơn thì đơn giản là cứ điền thêm tên của các trường vào: 
```
parquetFile.select("cId", "locID").show();
```
Kết quả sẽ được như sau (Kết quả in ra terminal sẽ chỉ hiện 20 kết quả, muốn lấy toàn bộ kết quả các bạn hãy write ra file csv hoặc json đễ tiện theo dõi nha): <br /><br />
![](https://i.pinimg.com/564x/30/f9/9a/30f99a8edb7b6838653196056c9ad3d4.jpg)
<br />
Ngoài ra còn những phương thức khác như `group by`, `count` có chức năng giống hệt trong SQL:
```
parquetFile.select("cId", "locID").groupBy("cId").count().show();
```
Tuy nhiên việc sử dụng như này khiến chúng ta khó hơn trong việc phát hiện các logic của vấn đề, lời khuyên mình dành cho mọi người vẫn là nên sử dụng Spark SQL như phần dưới đây.

### Truy vấn dữ liệu sử dụng Spark SQL
Việc sử dụng Spark SQL với những câu lệnh SQL quen thuộc được học trong cơ sở dữ liệu sẽ giúp chúng ta quen thuộc và dễ dàng xử lý các bài toán được đặt ra hơn.
Đầu tiên phải tạo ra 1 bảng (gọi là 1 bảng nhưng thực chất không phải nha, chúng ta có thể hiểu đơn giản là 1 bảng giả cũng được): 
```
parquetFile.createOrReplaceTempView("data");
```
Sau khi tạo như trên thì coi như là giờ chúng ta đã có 1 bảng là data, giờ chúng ta sẽ thực hiện các truy vấn trên bảng này như sau: 
```
spark.sql("select cId from data").show();
spark.sql("select cId, locID from data").show();
```
Kết quả sẽ cho tương tự như bên trên.
Chúng ta cũng có thể sử dụng group by, count, where như ở trong SQL để thực hiện các truy vấn như sau: 
```
spark.sql("select cId, count(locID) from data group by cID").show();
```
(à nhớ là phải có .show() ở cuối để nó in ra màn hình nha, không làm xong rồi không in ra lại cứ tưởng mình làm sai gì thì hỏng)

Các bạn có thể xem thêm các hướng dẫn về Spark SQL tại trang chủ của Spark SQL : [https://spark.apache.org/docs/latest/sql-programming-guide.html](https://spark.apache.org/docs/latest/sql-programming-guide.html)

Mã nguồn của bài này các bạn có thể tham khảo tại : [https://github.com/demanejar/parquet-io-spark](https://github.com/demanejar/parquet-io-spark)
