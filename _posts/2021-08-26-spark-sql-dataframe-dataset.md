---
title: Spark SQL, Dataframe và Dataset
author: trannguyenhan 
date: 2021-08-26 20:52:00 +0700
categories: [Hadoop & Spark, Spark]
tags: [Spark SQL, Bigdata, Spark]
math: true
mermaid: true
image:
  src: https://i.pinimg.com/originals/50/64/13/5064132e69821108d0c4fff3c6afd9ce.jpg
---
*Spark SQL là một mô hình để xử lý dữ liệu có cấu trúc của Spark rất phổ biến. Interfaces cung cấp bởi Spark SQL có thêm các thông tin về cấu trúc của dữ liệu và các tính toán đang được thực hiện. Với những thông tin bổ sung này Spark SQL có thể thực hiện các tối ưu hóa bổ sung*

## SQL
Spark SQL được thiết kế cực kì gần gũi với các hệ quản trị cơ sở dữ liệu có quan hệ, nhưng không phải về mặt kiến trúc bên trong mà là về mặt tương tác với dữ liệu từ bên ngoài.

Chúng ta có thể truy vấn dữ dữ liệu bằng những câu truy vấn SQL cực kì quen thuộc trong MySQL hay MS SQL Server. Khi đi sâu vào ví dụ mình sẽ giải thích kĩ hơn phần này để mọi người có thể thấy rõ ràng Spark SQL dễ sử dụng như nào khi bạn là master bộ môn cơ sở dữ liệu tại trường.

## Dataframe và Dataset
**Dataframe** là tập dữ liệu phân tán, có cấu trúc là một tập dữ liệu 2 chiều, hay chúng ta có thể hiểu đơn giản nó là một cái bảng tính hay là bảng trong SQL (nhưng phải là một bảng được tối ưu hóa tốt).

**Dataset** thì rộng hơn dataframe, chúng ta có thể nhìn vào ngôn ngữ Java để dễ hiểu hơn, khi khai báo trong Java dataframe được khai báo là `Dataset<Row>`, vậy ta hiểu đơn giản dataframe như là dataset của các hàng, còn ngoài ra dataset có thể chứa thêm nhiều kiểu dữ liệu nữa như là `Dataset<String>`, `Dataset<Integer>`. Dataset còn có thể chứa một đối tượng bất kì mà chúng ta định nghĩa ra nữa.

Trong bài viết này hay là sau có làm bài tập nếu có nhắc tới `Dataframe` thì chúng ta hãy hiểu ngầm rằng đó chính là `Dataset<Row>` và ngược lại nha.

Giống với RDD, Dataset cũng có các phép biến đổi như `map`, `flatmap`, `filter`,...

**Riêng ý kiến của mình về việc sử dụng ngôn ngữ nào thì mình cho rằng**, mặc dù khi sử dụng Java bạn sẽ phải viết code dài hơn các ngôn ngữ khác như Scala hay Python. Tuy nhiên bạn sẽ hiểu được chi tiết từng câu lệnh mà bạn viết ra, thậm chí sau 1-2 tháng bạn đọc lại bạn vẫn có thể hình dung ra code của bạn đang viết gì vì Java viết code rất có quy tắc, các kiểu dữ liệu đều được khai báo, định nghĩa rõ ràng. Tất nhiên là mỗi người thì có một cách nhìn khác nhau và lựa chọn của riêng mình, và dù là sử dụng ngôn ngữ nào trong Spark đi nữa thì về cơ bản đều giống nhau mà thôi, mỗi bước trong ngôn ngữ này hoàn toàn có thể chuyển qua mỗi bước trong ngôn ngữ khác.

## Spark DataFrames Operations
Dĩ nhiên dataframe là chúng ta sẽ sử dụng chủ yếu vì thế nên đôi lúc dataframe sẽ chiếm nhiều phần hơn trong bài viết này của mình. Dataset như là một phần mở rộng của dataframe tuy nhiên nó được sử dụng như là trung gian để chuyển đổi mà thôi. 

### read 
Spark SQL có thể đọc được dữ liệu từ rất nhiều loại file như là csv, json, parquet,... bằng cách sử dụng hàm `read()`.
Ví dụ: 
```java
Dataset<Row> data = spark.read().csv("resources/input.csv");
```
```java
Dataset<Row> data = spark.read().json("resources/input.json");
```
```java
Dataset<Row> data = spark.read().parquet("resources/input.parquet");
```

Ngoài ra bạn có thể thêm một số option để có thể đọc dữ liệu theo ý muốn ví dụ như để tự động nhận schema khi đọc vào file csv thì sử dụng: 
```java
Dataset<Row> data = spark.read().option("inferSchema", true).csv("resources/input.csv");
```

Hoặc để Spark hiểu hàng đầu tiên của file csv là hàng tiêu đề thì thêm vào option: 
```java
Dataset<Row> data = spark.read().option("header", true).csv("resources/input.csv");
```

### write
Về ý nghĩa thì hàm write sẽ giúp ghi dữ liệu ra một file đầu ra và việc sử dụng hoàn toàn giống với hàm read chỉ thay `read()` bằng `write()` trong các câu lệnh.

### show
Để nhìn hoặc kiểm tra dữ liệu trong 1 dataframe thì sử dụng hàm `show()`, mục đích chính chỉ nhằm kiểm tra mà thôi nên `show()` chỉ hỗ trợ in ra 20 hàng trong dataframe.
```java 
Dataset<Row> data = spark.read().csv("resources/input.csv");
data.show();
```

Kết quả cho ra giống như bên dưới đây:
```bash
+--------+------+---+------------+
|Employee|Salary|stt|Salary_count|
+--------+------+---+------------+
|  George|    80| 10|          10|
|   Steve|    65|  9|          10|
|   Nigel|    64|  8|          10|
|   Micky|    62|  7|          10|
|     Lee|    60|  6|          10|
|    Tony|    50|  5|          10|
|    Paul|    48|  4|          10|
|    Alan|    45|  3|          10|
|    John|    42|  2|          10|
|   David|    35|  1|          10|
+--------+------+---+------------+
```

(Ví dụ này mình sẽ lấy để làm tập dữ liệu cho một số ví dụ bên dưới nha)

### printSchema
`printSchema` in ra cấu trúc của bảng hay là kiểu dữ liệu của từng cột.
```java
Dataset<Row> data = spark.read().csv("resources/input.csv");
data.printSchema();
```

Kết quả cho ra sẽ giống như bên dưới đây: 
```bash
root
 |-- Employee: string (nullable = true)
 |-- Salary: integer (nullable = true)
 |-- stt: integer (nullable = true)
 |-- Salary_count: long (nullable = false)
```

### select
`select` giúp lấy ra các cột theo ý muốn khi mà chúng ta không muốn in hết toàn bộ dữ liệu của dataframe.
```java
Dataset<Row> data = spark.read().csv("resources/input.csv");
data.select("Employee", "Salary").show();
```

Ví dụ trên lấy ra và hiển thị 2 hàng "Employee" và "Salary", kết quả sẽ cho ra giống như bên dưới đây: 
```bash
+--------+------+
|Employee|Salary|
+--------+------+
|  George|    80|
|   Steve|    65|
|   Nigel|    64|
|   Micky|    62|
|     Lee|    60|
|    Tony|    50|
|    Paul|    48|
|    Alan|    45|
|    John|    42|
|   David|    35|
+--------+------+
```

### filter
Giống với `select` tuy nhiên ta có thể hiểu là `select` là lọc cột còn `filter` là lọc hàng.
```java
Dataset<Row> data = spark.read().csv("resources/input.csv");
data.filter(data.col("Salary").$greater(60)).show();
```

Ví dụ trên lọc ra các hàng mà có cột "Salary" có giá trị lớn hơn 60, kết quả cho ra sẽ giống như bên dưới đây: 
```bash
+--------+------+---+------------+
|Employee|Salary|stt|Salary_count|
+--------+------+---+------------+
|  George|    80| 10|          10|
|   Steve|    65|  9|          10|
|   Nigel|    64|  8|          10|
|   Micky|    62|  7|          10|
+--------+------+---+------------+
```

### groupBy
`groupBy` sẽ giúp nhóm lại các hàng mà có cùng giá trị cần nhóm và có thể thực hiện tính toán trên các giá trị mà chúng ta nhóm lại.

Bây giờ chúng ta sẽ tạo ra 1 tập dữ liệu khác để dễ minh họa hơn: 
```java
StructType schema = new StructType().add("id", "integer")
				.add("day", "integer")
				.add("price", "integer")
				.add("units", "integer");

List<Row> listOfdata = new ArrayList<Row>();
listOfdata.add(RowFactory.create(100,1,23,10));
listOfdata.add(RowFactory.create(100,2,45,11));
listOfdata.add(RowFactory.create(100,3,67,12));
listOfdata.add(RowFactory.create(100,4,78,13));
listOfdata.add(RowFactory.create(101,1,23,10));
listOfdata.add(RowFactory.create(101,2,45,13));
listOfdata.add(RowFactory.create(101,3,67,14));
listOfdata.add(RowFactory.create(101,4,78,15));
listOfdata.add(RowFactory.create(102,1,23,10));
listOfdata.add(RowFactory.create(102,2,45,11));
listOfdata.add(RowFactory.create(102,3,67,16));
listOfdata.add(RowFactory.create(102,4,78,18));
```

Kết quả sau khi tạo tập dữ liệu trên là một bảng như sau: 

| id|day|price|units|
|---|---|-----|-----|
|100|  1|   23|   10|
|100|  2|   45|   11|
|100|  3|   67|   12|
|100|  4|   78|   13|
|101|  1|   23|   10|
|101|  2|   45|   13|
|101|  3|   67|   14|
|101|  4|   78|   15|
|102|  1|   23|   10|
|102|  2|   45|   11|
|102|  3|   67|   16|
|102|  4|   78|   18|

Nhiệm vụ của chúng ta bây giờ là nhóm lại các hàng có id giống nhau sử dụng groupBy như sau: 
```java
data.groupBy("id").agg(functions.collect_list("price").as("prices"), functions.collect_list("units").as("units")).show();
```

Kết quả sau khi thực hiện là: 
```bash
+---+----------------+----------------+
| id|          prices|           units|
+---+----------------+----------------+
|101|[23, 45, 67, 78]|[10, 13, 14, 15]|
|100|[23, 45, 67, 78]|[10, 11, 12, 13]|
|102|[23, 45, 67, 78]|[10, 11, 16, 18]|
+---+----------------+----------------+
```

Chúng ta cũng có thể thực hiện tính tổng hay tính trung bình, tìm min, tìm max bằng cách thay collect_list bằng sum, min, max,... như dưới đây: 
```java
data.groupBy("id").agg(functions.sum("price").as("prices"), functions.sum("units").as("units")).show();
```

Kết quả cho ra là: 
```bash
+---+------+-----+
| id|prices|units|
+---+------+-----+
|101|   213|   52|
|100|   213|   46|
|102|   213|   55|
+---+------+-----+
```


## Truy vấn bằng các câu lệnh SQL
Việc sử dụng các câu lệnh SQL sẽ là dễ sử dụng hơn, dễ hiểu hơn và gần gũi hơn rất nhiều so với chúng ta. Và vì thế nên mình thấy kể cả học hay đi làm thì việc sử dụng SQL để truy vấn dữ liệu là được sử dụng nhiều hơn khi mình sử dụng các hàm được cung cấp cho các dataframe.

Ví dụ chúng ta cũng sẽ đi thực hiện tính tổng giống như ví dụ bên trên: 
```java
data.createOrReplaceTempView("data");
spark.sql("select id, sum(price) as prices, sum(units) as units from data group by id").show();
```

`data.createOrReplaceTempView("data");` sẽ giúp tạo ra 1 bảng giống với trong các cơ sở dữ liệu quan hệ và chúng ta sẽ thao tác trên đó. 

Kết quả của ví dụ này thu được cũng giống như kết quả của ví dụ bên trên.

Những gì mà bạn làm được trong SQL như join, inner join, group by, select,... các bạn đề có thể làm y hệt với Spark SQL, cách viết cũng là y hệt. Mọi thứ là quá quen thuộc với những câu truy vấn SQL thế nên phần này mình sẽ dừng lại ở đây. 

## Chuyển đổi từ `Dataset<Row>` Sang `Dataset<String>`
Như phần đầu thì mình có nói là chúng ta có thể chuyển sang bất kì loại `Dataset` nào từ `Dataset<Row>` kể cả các object mà các bạn tự định nghĩa. Trong phần này mình nói về việc chuyển từ Dataset<Row> Sang `Dataset<String>` để có thể dễ dàng hình dung hơn.

Chúng ta sẽ sử dụng lại tập dữ liệu bên trên: 

|Employee|Salary|stt|Salary_count|
|--------|------|---|------------|
|  George|    80| 10|          10|
|   Steve|    65|  9|          10|
|   Nigel|    64|  8|          10|
|   Micky|    62|  7|          10|
|     Lee|    60|  6|          10|
|    Tony|    50|  5|          10|
|    Paul|    48|  4|          10|
|    Alan|    45|  3|          10|
|    John|    42|  2|          10|
|   David|    35|  1|          10|

Bây giờ mình sẽ nhóm lại tập dữ liệu trên thành một thông tin dạng `[Employee] : [Salary]` như sau: 
```java
data.map(new MapFunction<Row, String>() {
	private static final long serialVersionUID = 1L;
	
	@Override
	public String call(Row value) throws Exception {
		return value.getString(0) + " : " + value.getInt(1);
	}

}, Encoders.STRING()).show();
```

Kết quả thu được là: 
```bash
+-----------+
|      value|
+-----------+
|George : 80|
| Steve : 65|
| Nigel : 64|
| Micky : 62|
|   Lee : 60|
|  Tony : 50|
|  Paul : 48|
|  Alan : 45|
|  John : 42|
| David : 35|
+-----------+
```


Tham khảo: [https://spark.apache.org/](https://spark.apache.org/docs/2.3.0/sql-programming-guide.html)
