---
title: Window function, pivot trong Spark SQL (Part 2)
author: trannguyenhan 
date: 2021-09-14 08:52:00 +0700
categories: [Hadoop & Spark, Spark]
tags: [Spark SQL, Bigdata, Spark, pivot spark, window function]
math: true
mermaid: true
image:
  src: https://i.pinimg.com/originals/cc/38/cd/cc38cddba42b235e5e23638e9473b7d8.jpg
---
*Nếu bạn chưa xem phần 1 thì có thể xem lại [TẠI ĐÂY](https://demanejar.github.io/posts/spark-sql-window-function-pivot/) nha, bài viết hôm nay mình sẽ giới thiệu tiếp tới mọi người một số ví dụ về window function và pivot sâu hơn để mọi người có thể hiểu rõ hơn về window function và pivot trong Spark*

## Example 5: Window function 
### Yêu cầu 
Cho tập dữ liệu dạng _.csv_ như sau: 
```
time,department,items_sold,running_total
1,IT,15,15
2,Support,81,81
3,Support,90,171
4,Support,25,196
5,IT,40,55
6,IT,24,79
7,Support,31,227
8,Support,1,228
9,HR,27,27
10,IT,75,154
```
-   Đọc dữ liệu vào spark từ file csv chứa dữ liệu.
-   Dùng SparkSQL query độ lệch giữa các running_total liên tiếp nhau theo thời gian của các department

INPUT: 

|time|department|items_sold|running_total|
|----|----------|----------|-------------|
| 1| IT| 15| 15|
| 2| Support| 81| 81|
| 3| Support| 90| 171|
| 4| Support| 25| 196|
| 5| IT| 40| 55|
| 6| IT| 24| 79|
| 7| Support| 31| 227|
| 8| Support| 1| 228|
| 9| HR| 27| 27|
| 10| IT| 75| 154|

OUTPUT: 

|time|department|items_sold|running_total|diff|
|----|----------|----------|-------------|----|
| 9| HR| 27| 27| 27|
| 1| IT| 15| 15| 15|
| 5| IT| 40| 55| 40|
| 6| IT| 24| 79| 24|
| 10| IT| 75| 154| 75|
| 2| Support| 81| 81| 81|
| 3| Support| 90| 171| 90|
| 4| Support| 25| 196| 25|
| 7| Support| 31| 227| 31|
| 8| Support| 1| 228| 1|

### Lời giải 
Chúng ta sẽ thấy là bài này giống bài tính tổng tích lũy tại [Ví dụ 4](https://demanejar.github.io/posts/spark-sql-window-function-pivot/#example-4-window-function) chỉ khác là tại ví dụ này chúng ta sẽ đi tính hiệu giữa 2 _running_total_ liên tiếp mà không phải là  tổng. 

Để giải quyết bài này thì mình đưa ra một lời giải hơi phức tạp 1 tí như sau: 
```java
package part5;

import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;
import org.apache.spark.sql.functions;
import org.apache.spark.sql.expressions.Window;
import org.apache.spark.sql.expressions.WindowSpec;

public class Main {
	public static void main(String[] args) {
		SparkSession spark = SparkSession.builder().appName("Bai_5").master("local").getOrCreate();
		Dataset<Row> data = spark.read().option("header", true).option("inferSchema", true).csv("src/part5/input.csv");
		
		WindowSpec wins = Window.partitionBy("department")
				.orderBy("time")
				.rowsBetween(Window.currentRow() - 1, Window.currentRow());
		
		Dataset<Row> result_1 = data.withColumn("diff_sum", functions.sum("running_total").over(wins));
		Dataset<Row> result_2 = result_1.withColumn("diff_1", functions.max("running_total").over(wins));
		result_2.createOrReplaceTempView("data_1");
		
		spark.sql("select time, department, items_sold, running_total, diff_1, diff_sum - diff_1 as diff_2 from data_1")
				.createOrReplaceTempView("data_2");
		
		Dataset<Row> result = spark.sql("select time, department, items_sold, running_total, diff_1 - diff_2 as diff from data_2");
		result.show();
		
	}
}
```

Ngắn gọn lại lời giải trên, thì để tính được cột _diff_ ta sẽ lấy tổng của _running_total_ của hàng hiện và hàng trước đó trừ đi _running_total_ của hàng hiện tại để lấy ra _running_total_ của hàng trước đó, gọi cột mới này là cột _diff_2_. Sau đó lấy _running_total_ của mỗi hàng hiện tại trừ đi _running_total_ của hàng trước đó (là cột _diff_2_) thì ta sẽ ra được kết quả. Sở dĩ làm phức tạp như vậy mà không lấy luôn _running_total_ của hàng trước đó qua _window function_ là để tránh đi kết quả `null`, nếu ta lấy luôn giá trị _running_total_ của hàng trước đó bằng _window function_ sau:  
```java
WindowSpec wins = Window.partitionBy("department").orderBy("time").rowsBetween(Window.currentRow() - 1, Window.currentRow()-1);
```
Thì ta sẽ tạo được thêm cột mới là giá trị _running_total_ của hàng trước đó và công việc bây giờ chỉ cần trừ đi sẽ có được kết quả cuối cùng, đơn giản như vậy nhưng kết quả của phương pháp trên bị vướng giá trị `null` như dưới đây: 
```bash
+----+----------+----------+-------------+----+
|time|department|items_sold|running_total|diff|
+----+----------+----------+-------------+----+
|   9|        HR|        27|           27|null|
|   1|        IT|        15|           15|null|
|   5|        IT|        40|           55|  40|
|   6|        IT|        24|           79|  24|
|  10|        IT|        75|          154|  75|
|   2|   Support|        81|           81|null|
|   3|   Support|        90|          171|  90|
|   4|   Support|        25|          196|  25|
|   7|   Support|        31|          227|  31|
|   8|   Support|         1|          228|   1|
+----+----------+----------+-------------+----+
```
Và mình vẫn chưa tìm được cách `replace` giá trị `null` kia thế nên mình giải quyết bài toán này bằng 1 phương pháp hơi phức tạp như trên. 

## Example 6: Window function 
### Yêu cầu
Cho tập dữ liệu đầu vào _.csv_ như sau: 
```
id,name,department,salary
1,Hunter Fields,IT,15
2,Leonard Lewis,Support,81
3,Jason Dawson,Support,90
4,Andre Grant,Support,25
5,Earl Walton,IT,40
6,Alan Hanson,IT,24
7,Clyde Matthews,Support,31
8,Josephine Leonard,Support,1
9,Owen Boone,HR,27
10,Max McBride,IT,75
```
-   Đọc dữ liệu vào spark từ file csv
-   Tìm sự chênh lệch salary giữa nhân viên có lương cao nhất với các nhân viên còn lại trong từng phòng ban

INPUT:

| id| name|department|salary|
|---|-----------------|----------|------|
| 1| Hunter Fields| IT| 15|
| 2| Leonard Lewis| Support| 81|
| 3| Jason Dawson| Support| 90|
| 4| Andre Grant| Support| 25|
| 5| Earl Walton| IT| 40|
| 6| Alan Hanson| IT| 24|
| 7| Clyde Matthews| Support| 31|
| 8|Josephine Leonard| Support| 1|
| 9| Owen Boone| HR| 27|
| 10| Max McBride| IT| 75|

OUTPUT: 

| id| name|department|salary|diff|
|---|-----------------|----------|------|----|
| 9| Owen Boone| HR| 27| 0|
| 1| Hunter Fields| IT| 15| 60|
| 5| Earl Walton| IT| 40| 35|
| 6| Alan Hanson| IT| 24| 51|
| 10| Max McBride| IT| 75| 0|
| 2| Leonard Lewis| Support| 81| 9|
| 3| Jason Dawson| Support| 90| 0|
| 4| Andre Grant| Support| 25| 65|
| 7| Clyde Matthews| Support| 31| 59|
| 8|Josephine Leonard| Support| 1| 89|

### Lời giải 
Bài này khá là đơn giản hơn so với các bài bên trên, chúng ta chỉ cần tìm ra giá trị `max` của _department_ và trừ đi là xong: 
```java
package part6;

import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;
import org.apache.spark.sql.functions;
import org.apache.spark.sql.expressions.Window;
import org.apache.spark.sql.expressions.WindowSpec;

public class Main {
	public static void main(String[] args) {
		SparkSession spark = SparkSession.builder().appName("Bai_6").master("local").getOrCreate();
		Dataset<Row> data = spark.read().option("header", true).option("inferSchema", true).csv("src/part6/input.csv");
		
		WindowSpec wins = Window.partitionBy("department").orderBy("id")
				.rowsBetween(Window.unboundedPreceding(), Window.unboundedFollowing());
		Dataset<Row> result_1 = data.withColumn("diff_max", functions.max("salary").over(wins));
		
		result_1.createOrReplaceTempView("data_1");
		Dataset<Row> result = spark.sql("select id, name, department, salary, diff_max - salary as diff from data_1");
		
		result.show();
	}
}
```

## Example 7: Window function 
### Yêu cầu
Cho tập dữ liệu dưới dạng đầu vào _.csv_ như sau: 
```
Employee,Salary
Tony,50
Alan,45
Lee,60
David,35
Steve,65
Paul,48
Micky,62
George,80
Nigel,64
John,42
```

- Load dữ liệu từ file csv
- Thêm cột Percentage với giá trị được quy định như sau
-   30% số nhân viên đầu có lương cao nhất nhận giá trị “High”
-   40% tiếp theo nhận giá trị “Average”
-   Phần còn lại nhận giá trị “Low”

### Lời giải 
Thoạt nhìn thì ta sẽ thấy nó đơn giản nhưng nhìn chung nó cũng không được đơn giản cho lắm. 
Hướng làm cho bài này mà mình đưa ra sẽ là: 
- Sắp xếp lại giá trị lương theo thứ tự từ cao tới thấp
- Đánh số thứ tự cho các nhân viên vừa sắp xếp theo chỉ số giảm dần
- Tính giá trị nhân viên nằm ở top bao nhiêu phần trăm bằng cách lấy số thự tự của nhân viên vừa được đánh dấu chia cho số thứ tự cao nhất (hay là tổng số nhân viên)
- Thêm cột _Percentage_ bằng cách những nhân viên nào có giá trị phần trăm tính ở bước trên từ 0.7 tới 1 thì là _High_, từ 0.3 tới 0.7 thì là _Average_, từ 0 tới 0.3 là _Low_

Mã nguồn lời giải này như sau: 
```java
package part7;

import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;
import org.apache.spark.sql.functions;
import org.apache.spark.sql.expressions.Window;

public class Main {
	public static void main(String[] args) {
		SparkSession spark = SparkSession.builder().appName("Bai_7").master("local").getOrCreate();
		Dataset<Row> data = spark.read().option("header", true).option("inferSchema", true).csv("src/part7/input.csv");
		data = data.sort(data.col("Salary").desc());
		data = data.withColumn("stt", functions.rank().over(Window.orderBy("Salary")));
		data = data.withColumn("Salary_count", functions.count("Salary").over()).sort(data.col("Salary").desc());
		data.createOrReplaceTempView("data");
		
		Dataset<Row> data_1 = spark.sql("select Employee, Salary, stt / Salary_count as per from data");
		data_1.createOrReplaceTempView("data_1");
		
		Dataset<Row> result_1 = spark.sql("select Employee, Salary from data_1 where per > 0.7 and per <= 1");
		result_1 = result_1.withColumn("Percentage", functions.lit("High"));
		
		Dataset<Row> result_2 = spark.sql("select Employee, Salary from data_1 where per > 0.3 and per <= 0.7");
		result_2 = result_2.withColumn( "Percentage", functions.lit("Average"));
		
		Dataset<Row> result_3 = spark.sql("select Employee, Salary from data_1 where per <= 0.3 ");
		result_3 = result_3.withColumn("Percentage", functions.lit("Low"));
		
		Dataset<Row> result = result_1.union(result_2).union(result_3);
		result.show();
	}
}
```

## Example 8: Window function 
### Yêu cầu
Cho tập dữ liệu với đầu vào _.csv_ như sau: 
```
id,title,genre,quantity
1,Hunter Fields,romance,15
2,Leonard Lewis,thriller,81
3,Jason Dawson,thriller,90
4,Andre Grant,thriller,25
5,Earl Walton,romance,40
6,Alan Hanson,romance,24
7,Clyde Matthews,thriller,31
8,Josephine Leonard,thriller,1
9,Owen Boone,sci-fi,27
10,Max McBride,romance,75
```

-   Load dữ liệu từ csv
-   Lọc những tilte có số lượng quantity top 1 và top 2 với mỗi genre

### Lời giải 
Bài này mình có thể giải quyết mà không cần sử dụng _windows function_ như sau: 
```java
package part8;

import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;

public class Main {
	public static void main(String[] args) {
		SparkSession spark = SparkSession.builder().appName("Bai_8").master("local").getOrCreate();
		Dataset<Row> data = spark.read().option("header", true).option("inferSchema", true).csv("src/part8/input.csv");
		
		data.createOrReplaceTempView("data");
		spark.sql("SELECT\n"
				+ "  id,\n"
				+ "  title,\n"
				+ "  genre,\n"
				+ "  quantity,\n"
				+ "  rank\n"
				+ "FROM (\n"
				+ "  SELECT\n"
				+ "    id,\n"
				+ "    title,\n"
				+ "    genre,\n"
				+ "    quantity,\n"
				+ "    dense_rank() OVER (PARTITION BY genre ORDER BY quantity DESC) as rank\n"
				+ "  FROM data) tmp\n"
				+ "WHERE\n"
				+ "  rank <= 2").show();
		
	}
}
```

## Example 9: pivot
### Yêu cầu
Cho tập dữ liệu với đầu vào _.csv_ như sau: 
```
1,Question1Text,Yes,abcde1,0,"(x1,y1)"
2,Question2Text,No,abcde1,0,"(x1,y1)"
3,Question3Text,3,abcde1,0,"(x1,y1)"
1,Question1Text,No,abcde2,0,"(x2,y2)"
2,Question2Text,Yes,abcde2,0,"(x2,y2)"
```

-   Load dữ liệu từ json
-   Tạo báo cáo giống output

INPUT: 

|Qid| Question|AnswerText|ParticipantID|Assessment| GeoTag|
|---|-------------|----------|-------------|----------|-------|
| 1|Question1Text| Yes| abcde1| 0|(x1,y1)|
| 2|Question2Text| No| abcde1| 0|(x1,y1)|
| 3|Question3Text| 3| abcde1| 0|(x1,y1)|
| 1|Question1Text| No| abcde2| 0|(x2,y2)|
| 2|Question2Text| Yes| abcde2| 0|(x2,y2)|

OUTPUT:

|ParticipantID|Assessment| GeoTag|Qid_1|Qid_2|Qid_3|
|-------------|----------|-------|-----|-----|-----|
| abcde1| 0|(x1,y1)| Yes| No| 3|
| abcde2| 0|(x2,y2)| No| Yes| null|

### Lời giải 
Bài này cũng khá tương tự với lại [ví dụ 3](https://demanejar.github.io/posts/spark-sql-window-function-pivot/#example-3-pivot), nên bạn có thể xem lại [ví dụ 3](https://demanejar.github.io/posts/spark-sql-window-function-pivot/#example-3-pivot) để hiểu hơn 1 số chỗ phép biến đổi nha: 
```java
package part9;

import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;
import org.apache.spark.sql.functions;


public class Main {
	public static void main(String[] args) {
		SparkSession spark = SparkSession.builder().appName("Bai_9").master("local").getOrCreate();
		Dataset<Row> data = spark.read().option("header", true).option("inferSchema", true).csv("src/part9/input.csv");
		
		Dataset<Row> result = data.withColumn("concat_Qid", functions.concat(functions.lit("Qid_"), data.col("Qid")))
				.groupBy("ParticipantID", "Assessment", "GeoTag").pivot("concat_Qid").agg(functions.first("AnswerText"));
		
		result.show();
	}
}
```

## Tổng kết
Bạn có thể xem lại toàn bộ mã nguồn của 9 ví dụ trên [TẠI ĐÂY](https://github.com/demanejar/window-function-pivot-spark-sql) (Nếu thấy thú vị thì và bổ ích cho nhóm mình 1 star trong repo nha).

File tổng hợp các ví dụ các bạn có thể xem [TẠI  ĐÂY](https://github.com/demanejar/window-function-pivot-spark-sql/tree/master/resource)

Mong rằng 9 ví dụ này giúp cho các bạn hiểu qua được phần vào về _window function_ và _pivot_ trong Spark SQL hơn.
