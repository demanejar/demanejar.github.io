---
title: Window function, pivot trong Spark SQL
author: trannguyenhan 
date: 2021-07-26 08:52:00 +0700
categories: [Apache, Spark]
tags: [Spark SQL, Bigdata, Spark, pivot spark, window function]
math: true
mermaid: true
image:
  src: https://i.pinimg.com/originals/cc/38/cd/cc38cddba42b235e5e23638e9473b7d8.jpg
---
*Window aggregate functions (hay thường được gọi tắt là window functions hoặc windowed aggregates) là hàm giúp hỗ trợ tính toán trên 1 nhóm các bản ghi được gọi là cửa sổ mà có liên quan tới bản ghi hiện tại.*

Nhìn chung thì những phần này khá trìu tượng về mặt lý thuyết nên đọc lý thuyết càng gây ra các cảm giác khó hiểu nên thế mình sẽ giới thiệu _window function_ và _pivot_ qua những ví dụ để dễ dàng hình dung hơn nha.

## Example 1: Simple select 
### Yêu cầu 
Cho tập dữ liệu dưới dạng file csv như sau với tên file là `input.csv`: 
```
id,name,population
0,Warsaw,1 764 615
1,Villeneuve-Loubet,15 020
2,Vranje,83 524
3,Pittsburgh,1 775 634
```
_Yêu cầu_: 
-   Load dữ liệu từ csv file
-   Query population lớn nhất

### Lời giải 
Bài này mang tính chất khởi động thôi chứ chưa hề động gì tới _window function_ hay _pivot_ cả, mọi người làm như bình thường: 
```java
package part1;

import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;

public class Main {
	public static void main(String[] args) {
		SparkSession spark = SparkSession.builder().appName("Bai_1").master("local").getOrCreate();
		Dataset<Row> data = spark.read().option("header", true).option("inferSchema", true).csv("src/part1/input.csv");
		
		data.createOrReplaceTempView("data");
		Dataset<Row> result = spark.sql("select population from data order by population desc limit 1");
		
		
		result.show();
	}
}
```

- _option_ _header_ giúp việc đọc vào từ file csv sẽ mặc định dòng đầu tiên là dòng tiêu đề cho các hàng 
- _option_ _inferSchema_ sẽ mặc định cho Spark tự nhận dạng dữ liệu đầu vào mà không cần chúng ta phải tạo _schema_ nữa

Chúng ta sẽ được kết quả của bài 1 như sau: 
```bash
+----------+
|population|
+----------+
|   1775634|
+----------+
```

## Example 2: group function 
### Yêu cầu 
Cho tập dữ liệu sau: 

| id|group|
|---|-----|
| 0| 0|
| 1| 1|
| 2| 0|
| 3| 1|
| 4| 0|

Nhóm lại các _id_ cùng _group_ và cho ra output như sau: 

|group| ids|
|-----|---------|
| 0|[0, 2, 4]|
| 1| [1, 3]|

Lưu ý: [0,2,4] là biểu diễn mảng

### Lời giải 
Bài này cũng chưa phải _window function_ hay _pivot_ gì, chúng ta nhóm bằng việc sử dụng hàm _group_ như sau: 
```java
package part2;

import java.util.ArrayList;
import java.util.List;

import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.RowFactory;
import org.apache.spark.sql.SparkSession;
import org.apache.spark.sql.functions;
import org.apache.spark.sql.types.StructType;

public class Main {
	public static void main(String[] args) {
		SparkSession spark = SparkSession.builder().appName("Bai_1").master("local").getOrCreate();
		
		StructType schema = new StructType().add("id", "integer")
				.add("group", "integer");

        List<Row> listOfdata = new ArrayList<Row>();
        listOfdata.add(RowFactory.create(0,0));
        listOfdata.add(RowFactory.create(1,1));
        listOfdata.add(RowFactory.create(2,0));
        listOfdata.add(RowFactory.create(3,1));
        listOfdata.add(RowFactory.create(4,0));
        
        Dataset<Row> data = spark.createDataFrame(listOfdata,schema);
        
        Dataset<Row> result = data.groupBy("group")
        		.agg(functions.collect_list("id").as("ids")).sort("group");
        
        result.show();
	}
}
```

Chúng ta sẽ được kết quả của bài 2 như sau: 
```bash
+-----+---------+
|group|      ids|
+-----+---------+
|    0|[0, 2, 4]|
|    1|   [1, 3]|
+-----+---------+
```

## Example 3: pivot
### Yêu cầu 
Cho tập dữ liệu như sau: 

| id|day|price|units|
|---|---|-----|-----|
|100| 1| 23| 10|
|100| 2| 45| 11|
|100| 3| 67| 12|
|100| 4| 78| 13|
|101| 1| 23| 10|
|101| 2| 45| 13|
|101| 3| 67| 14|
|101| 4| 78| 15|
|102| 1| 23| 10|
|102| 2| 45| 11|
|102| 3| 67| 16|
|102| 4| 78| 18|

Sử dụng _pivot_ và cho ra kết quả như sau: 

| id|price_1|price_2|price_3|price_4|unit_1|unit_2|unit_3|unit_4|
|---|-------|-------|-------|-------|------|------|------|------|
|100| 23| 45| 67| 78| 10| 11| 12| 13|
|101| 23| 45| 67| 78| 10| 13| 14| 15|
|102| 23| 45| 67| 78| 10| 11| 16| 18|

### Lời giải 
Đầu tiên là chúng ta sẽ đi tạo tập dữ liệu theo yêu cầu đề bài (các bạn cũng có thể tạo 1 file csv và đọc vào cũng được nha, ở đây mình muốn tạo dữ liệu bởi nhiều cách để có thể giới thiệu hết 1 lượt những cách tạo dữ liệu input trong _spark_): 
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

Nhìn vào yêu cầu output chúng ta sẽ thấy là sẽ có 2 yêu cầu rõ rệt là sẽ chia _price_ ra trước sau đó là chia _unit_. Vậy chúng ta cũng sẽ làm từng bước một, ***đầu tiên*** là chia _price_ theo _id_ như sau: 
```java
Dataset<Row> result_1 = data.withColumn("concat_day", functions.concat(functions.lit("price_"), data.col("day")))
				.groupBy("id").pivot("concat_day").agg(functions.first("price"));
```
- `functions.concat()` giúp tạo ra tên mới cho hàng là _price__ cộng với đuôi giá trị _day_ 
- `pivot` giúp xoay ngang lại bảng dữ liệu ban đầu của chúng ta

Kết quả của bước đầu tiên sẽ là: 
```bash
+---+-------+-------+-------+-------+
| id|price_1|price_2|price_3|price_4|
+---+-------+-------+-------+-------+
|101|     23|     45|     67|     78|
|100|     23|     45|     67|     78|
|102|     23|     45|     67|     78|
+---+-------+-------+-------+-------+
```

Tương tự ta nhóm lại dữ liệu đầu bài theo _unit_ ở ***bước 2*** như sau: 
```java
Dataset<Row> result_2 = data.withColumn("concat_day", functions.concat(functions.lit("unit_"), data.col("day")))
				.groupBy("id").pivot("concat_day").agg(functions.first("units"));
```

Kết quả của bước 2 là: 
```bash
+---+------+------+------+------+
| id|unit_1|unit_2|unit_3|unit_4|
+---+------+------+------+------+
|101|    10|    13|    14|    15|
|100|    10|    11|    12|    13|
|102|    10|    11|    16|    18|
+---+------+------+------+------+
```

Công việc ***còn lại*** là đơn giản rồi, chỉ cần nối 2 bảng lại là xong: 
```java
Dataset<Row> result = result_1.join(result_2, "id").sort("id");
```

Ta sẽ có kết quả cuối cùng là: 
```bash
+---+-------+-------+-------+-------+------+------+------+------+
| id|price_1|price_2|price_3|price_4|unit_1|unit_2|unit_3|unit_4|
+---+-------+-------+-------+-------+------+------+------+------+
|100|     23|     45|     67|     78|    10|    11|    12|    13|
|101|     23|     45|     67|     78|    10|    13|    14|    15|
|102|     23|     45|     67|     78|    10|    11|    16|    18|
+---+-------+-------+-------+-------+------+------+------+------+
```

## Example 4: window function
### Yêu cầu 
Cho tập dữ liệu sau: 
```
time,department,items_sold
1,IT,15
2,Support,81
3,Support,90
4,Support,25
5,IT,40
6,IT,24
7,Support,31
8,Support,1
9,HR,27
10,IT,75
```

Yêu cầu:
-   Load dữ liệu vào spark từ file csv
-   Tính running_total hay ***tổng tích lũy*** số item đã bán được đến thời điểm time.

Output có dạng như sau: 

|time|department|items_sold|running_total|
|----|----------|----------|-------------|
| 9| HR| 27| 27|
| 1| IT| 15| 15|
| 5| IT| 40| 55|
| 6| IT| 24| 79|
| 10| IT| 75| 154|
| 2| Support| 81| 81|
| 3| Support| 90| 171|
| 4| Support| 25| 196|
| 7| Support| 31| 227|
| 8| Support| 1| 228|


### Lời giải 
Phân tích output tí ta có thể thấy: 
- Các bản ghi được nhóm lại theo từng _department_ và sắp xếp theo _time_
- Cột _running_total_ được tính bằng tổng tích lũy các _items_sold_ ở phía trước nó mà cùng _department_

Phần nhóm và sắp xếp thì ta có thể sử dụng _group_ và _sort_, còn phần tính tổng tích lũy thì sao? Giờ phải có cách nào lấy ra các bản ghi cùng _department_ và ở phía trước của bản ghi hiện tại, _window function_ là cách tốt nhất để giải quyết vấn đề này.

```java
package part4;

import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;
import org.apache.spark.sql.functions;
import org.apache.spark.sql.expressions.Window;
import org.apache.spark.sql.expressions.WindowSpec;

public class Main {
	public static void main(String[] args) {
		SparkSession spark = SparkSession.builder().appName("Bai_4").master("local").getOrCreate();
		Dataset<Row> data = spark.read().option("header", true).option("inferSchema", true).csv("src/part4/input.csv");
				
		WindowSpec wins = Window.partitionBy("department").orderBy("time").rowsBetween(Window.unboundedPreceding(), Window.currentRow());
		Dataset<Row> result = data.withColumn("running_total", functions.sum("items_sold").over(wins));
		
		result.show();
	}
}

```

- `partitionBy` chia tách dữ liệu ban đầu ra theo _partition_
- `rowsBetween(Window.unboundedPreceding(), Window.currentRow())` giới hạn lại cửa sổ là từ đầu cho tới vị trí hiện tại
- `functions.sum("items_sold").over(wins)` là giá trị của cột `running_total` được tạo bằng tổng của cột _items_sold_ giới hạn trong cửa sổ `wins`

Kết quả cuối cùng của ví dụ 4 sẽ là: 
```bash
+----+----------+----------+-------------+
|time|department|items_sold|running_total|
+----+----------+----------+-------------+
|   9|        HR|        27|           27|
|   1|        IT|        15|           15|
|   5|        IT|        40|           55|
|   6|        IT|        24|           79|
|  10|        IT|        75|          154|
|   2|   Support|        81|           81|
|   3|   Support|        90|          171|
|   4|   Support|        25|          196|
|   7|   Support|        31|          227|
|   8|   Support|         1|          228|
+----+----------+----------+-------------+
```

Còn 1 số ví dụ nữa có lẽ mình sẽ viết trong phần 2 nha vì bài này cũng khá là dài rồi. Xem tiếp phần 2 [TẠI ĐÂY](#)
