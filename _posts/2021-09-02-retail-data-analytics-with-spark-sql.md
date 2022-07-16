---
title: Phân tích dữ liệu bán lẻ với Spark SQL
author: trannguyenhan 
date: 2021-09-02 20:52:00 +0700
categories: [Hadoop & Spark, Spark]
tags: [Spark SQL, Bigdata, Spark]
math: true
mermaid: true
image:
  src: https://i.pinimg.com/originals/50/64/13/5064132e69821108d0c4fff3c6afd9ce.jpg
---
_Như đã tìm hiểu ở bài viết trước về [Spark SQL, Dataframe và Dataset](https://demanejar.github.io/posts/spark-sql-dataframe-dataset/), Spark SQL là một mô hình để xử lý dữ liệu có cấu trúc của Spark rất phổ biến. Trong bài viết này chúng ta sẽ sử dụng Spark SQL để đi phân tích, xử lý một dữ liệu bán lẻ có cấu trúc cho trước_

## Dữ liệu và chuẩn bị dữ liệu
### Dữ liệu

Cho dataset đính kèm về dữ liệu bán lẻ online [TẠI ĐÂY](https://github.com/demanejar/retails/blob/master/resources/retails.csv), hãy phân tích dữ liệu trên để trả lời 5 câu hỏi sau đây: 

1. Có tổng bao nhiêu giao dịch, sản phẩm và khách hàng khác nhau?
2. Tỉ lệ khách hàng không có thông tin
3. Đâu là nước có số lượng đơn hàng (Quantity) nhiều thứ 3?
4. Từ nào xuất hiện ít nhất trong phần Description?
5. Sản phẩm nào bán được số lượng (Quantity) lớn nhất ở United Kingdom?

### Chuẩn bị dữ liệu

Để trực quan thì dữ liệu chúng ta sẽ đẩy lên HDFS và Spark SQL sẽ lấy dữ liệu trực tiếp từ HDFS để xử lý, để đẩy dữ liệu lên HDFS chúng ta sử dụng câu lệnh: 
```bash
hdfs dfs -copyFromLocal resources/retails.csv /
```

Kiểm tra dữ liệu đã có trên HDFS chưa:
```bash
hdfs dfs -ls /
```

### Thư viện và ngôn ngữ sử dụng

Mình là fan cứng của Java vì vậy mình sẽ sử dụng Java để giải quyết bài toán này thay vì Scala. Vì Java không phải là ngôn ngữ triển khai của Spark nên những hỗ trợ về Java cũng không nhiều và cũng có rất ít tài liệu về Spark viết với Java nên mình cũng muốn đóng góp thêm chút ít phần tài liệu về Spark với Java tới cộng đồng.

Các bạn tạo một maven project và thêm vào 2 phụ thuộc sau (phụ thuộc Spark Core và Spark SQL): 
```xml
<!-- https://mvnrepository.com/artifact/org.apache.spark/spark-core -->
<dependency>
	<groupId>org.apache.spark</groupId>
	<artifactId>spark-core_2.12</artifactId>
	<version>3.1.2</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.apache.spark/spark-sql -->
<dependency>
	<groupId>org.apache.spark</groupId>
	<artifactId>spark-sql_2.12</artifactId>
	<version>3.1.2</version>
	<scope>provided</scope>
</dependency>
```

## Phân tích và xử lý dữ liệu bán lẻ
 Nếu bạn mới tìm hiểu về Spark SQL thì hãy xem lại bài viết [Spark SQL, Dataframe và Dataset](https://demanejar.github.io/posts/spark-sql-dataframe-dataset/) để hiểu rõ hơn về SparkSession, Dataframe, Dataset<Row>, các `option` `inferSchema` và `header`. Trong 5 bài tập này mình sẽ giải thích về logic của vấn đề và giải thích những cái mới.

### Part 1
*Câu hỏi: Có tổng bao nhiêu giao dịch, sản phẩm và khách hàng khác nhau?*

Với câu hỏi thứ nhất, chúng ta lấy ra từng loại giao dịch, sản phẩm và khách hàng và đếm số lượng của từng loại: 

```java
package com.spark.part_1;

import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;

public class Main {
	public static void main(String[] args) {
		SparkSession spark = SparkSession.builder()
				.appName("Part-1")
				.master("local")
				.getOrCreate();
		
		Dataset<Row> data = spark.read()
				.option("inferSchema", true)
				.option("header", true)
				.csv("hdfs://localhost:9000/retails.csv");
				
		// number of customer distinct
		// except 1 because there are value is null of information customer ID
		long cntCustomers = data.select("CustomerID").distinct().count() - 1; 
		
		// number of product distinct
		long cntProdcts = data.select("StockCode").distinct().count();
		
		// number of invoice distinct
		long cntInvoices = data.select("InvoiceNo").distinct().count();
		
		// print 
		System.out.println("Number of customer distinct: " + cntCustomers); 
		System.out.println("Number of product distinct: " + cntProdcts);
		System.out.println("Number of invoice distinct: " + cntInvoices);
		
		// output: 
		// Number of customer distinct: 4372
		// Number of product distinct: 4070
		// Number of invoice distinct: 25900
	}
}
```
- Chúng ta nhìn vào bản ghi có thể thấy là các giá trị khách hàng, sản phẩm hay hóa đơn đều có những bản ghi lặp lại, vì vậy trước khi đếm chúng ta phải sử dụng `distinct()` để lọc các giá trị trùng.
- Nhìn vào câu 2 chúng ta có thể thấy là có những khách hàng sẽ không có thông tin, vì thế trường `CustomerID` sẽ có những trường `NULL` và chúng ta phải bỏ chúng đi, vì thế `cntCustomers` phải trừ đi 1.

### Part 2
*Câu hỏi: Tỉ lệ khách hàng không có thông tin*

Câu hỏi 2 này cũng gần giống với câu hỏi đầu tiên. Chúng ta đếm những khách hàng không có thông tin và chia cho tổng số lượng khách hàng: 

```java
package com.spark.part_2;

import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;

public class Main {
	public static void main(String[] args) {
		SparkSession spark = SparkSession.builder().appName("Part-2").master("local").getOrCreate();
		Dataset<Row> data = spark.read()
				.option("inferSchema", true)
				.option("header", true)
				.csv("hdfs://localhost:9000/retails.csv");
		
		// get number of customer (done part 1)
		long cntCustomers = data.select("CustomerID").count();
		
		// get number of customer no information
		long cntCustomersNoInfor = data.select("CustomerID").filter(data.col("CustomerID").isNull()).count();
		
		double ratio = (double) cntCustomersNoInfor / cntCustomers * 100;
		System.out.printf("Ratio no information: %f \n", ratio);
	}
}
```
- Sử dụng `isNull()` để lấy ra các dòng NULL chính xác hơn

### Part 3
*Câu hỏi: Đâu là nước có số lượng đơn hàng (Quantity) nhiều thứ 3?*

Với câu hỏi này, chúng ta nhóm lại các đơn hàng theo từng quốc gia và tính tổng số lượng theo Quantity: 

```java
package com.spark.part_3;

import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;

public class Main {
	public static void main(String[] args) {
		SparkSession spark = SparkSession.builder().appName("Part-3").master("local").getOrCreate();
		Dataset<Row> data = spark.read()
				.option("inferSchema", true)
				.option("header", true)
				.csv("hdfs://localhost:9000/retails.csv");
		
		data.createOrReplaceTempView("data");
		spark.sql("select Country, sum(Quantity) as count from data group by Country order by count desc").show();
	}
}
```
- Trong câu này chúng ta truy vấn trực tiếp bằng câu lệnh SQL với `Spark.sql()`. Kết quả đầu ra khi sử dụng `Spark.sql` là một `Dataset<Row>` và sử dụng bình thường như những `Dataset<Row>` được khai báo khác khác.

### Part 4
*Câu hỏi: Từ nào xuất hiện ít nhất trong phần Description?*

Câu hỏi này có lẽ là câu hỏi phức tạp nhất trong phần này, chúng ta phải lấy ra phần `Description` và xử lý chúng. 

Mình sẽ lấy ví dụ để rõ ràng hơn nha, giả sử chúng ta có 1 bản ghi của trường `Description` là "WHITE HANGING HEART T-LIGHT HOLDER", chúng ta sẽ sử dụng flatMap để tách từ 1 dòng này ra thành 5 dòng, mỗi dòng chứa một từ "WHITE", "HANGING", "HEART", "T_LIGHT", "HOLDER". Sau khi tách toàn bộ các dòng của tập dữ liệu chúng ta sẽ làm công việc như các câu hỏi bên trên là nhóm chúng lại theo các từ giống nhau và thực hiện đếm.

```java
package com.spark.part_4;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.Iterator;
import java.util.List;

import org.apache.spark.api.java.function.FlatMapFunction;
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.RowFactory;
import org.apache.spark.sql.SparkSession;
import org.apache.spark.sql.catalyst.encoders.RowEncoder;
import org.apache.spark.sql.types.StructType;

public class Main {
	public static void main(String[] args) {
		SparkSession spark = SparkSession.builder().appName("Part-4").master("local").getOrCreate();
		Dataset<Row> data = spark.read()
				.option("inferSchema", true)
				.option("header", true)
				.csv("hdfs://localhost:9000/retails.csv");
		
		data.where("Description is not null").flatMap(new FlatMapFunction<Row, Row>() {
			private static final long serialVersionUID = 1L;
			private int cnt = 0;
			
			@Override
			public Iterator<Row> call(Row r) throws Exception {
				List<String> listItem = Arrays.asList(r.getString(2).split(" "));
				
				List<Row> listItemRow = new ArrayList<Row>();
				for (String item : listItem) {
					listItemRow.add(RowFactory.create(cnt, item, 1));
					cnt++;
				}
				
				return listItemRow.iterator();
			}
		}, RowEncoder.apply(new StructType().add("number", "integer").add("word", "string").add("lit", "integer"))).createOrReplaceTempView("data");
		
		spark.sql("select word, count(lit) as count from data group by word order by count desc").show();
	}
}
```
- `data.where("Description is not null")` để bỏ đi những trường NULL, vì chúng vừa không mang lại gì mà còn hay gây ra lỗi `NullPointerException`.
- `RowEncoder` sẽ nói cho Spark biết `StructType` của dữ liệu mới sau khi biến đổi của bạn có dạng như nào.

### Part 5
*Câu hỏi: Sản phẩm nào bán được số lượng (Quantity) lớn nhất ở United Kingdom?*

```java
package com.spark.part_5;

import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;

public class Main {
	public static void main(String[] args) {
		SparkSession spark = SparkSession.builder().appName("Part-4").master("local").getOrCreate();
		Dataset<Row> data = spark.read()
				.option("inferSchema", true)
				.option("header", true)
				.csv("hdfs://localhost:9000/retails.csv");
		
		data.filter(data.col("Country").equalTo("United Kingdom")).createOrReplaceTempView("data");
		spark.sql("select Description, sum(Quantity) as count from data group by Description order by count desc").show();
	}
}
```

Xem toàn bộ mã nguồn của Project tại [https://github.com/demanejar/retails](https://github.com/demanejar/retails)
