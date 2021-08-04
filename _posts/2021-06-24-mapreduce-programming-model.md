---
title: Mô hình lập trình MapReduce cho Bigdata 
author: trannguyenhan
date: 2021-06-24 08:00:00 +0700
categories: [Apache, Hadoop]
tags: [Ubuntu, mapreduce, Bigdata, Java]
math: true
mermaid: true
image:
  src: https://i.pinimg.com/originals/24/b3/8d/24b38d9f8ea2a4bddb32c1c95d290378.jpg
---
*MapReduce là một kỹ thuật xử lý và là một mô hình lập trình cho tính toán phân tán để triển khai và xử lý dữ liệu lớn. MapReduce chứa 2 tác vụ quan trọng là map và reduce, trong đó hàm map xử lý các cặp key/value đầu vào cho ra kết quả là các cặp key/value trung gian, và hàm reduce gộp các giá trị đầu ra trung gian có cùng các key trung gian lại với nhau. WordCount là một ví dụ điển hình cho MapReduce mà sẽ được mình minh họa trong bài viết này*

## Tại sao Mapreduce lại ra đời?
Như phần giới thiệu thì ai cũng biết là MapReduce là viết gộp lại của map và reduce là 2 hàm chính trong phương pháp lập trình này. Mô hình lập trình này bắt nguồn chính là từ Google, hay rõ ràng hơn là từ 1 bài báo của Google. Vấn đề đặt ra là cần phải song song hóa các tính toán, phân tán dữ liệu và phải có khả năng chịu lỗi cao. 

MapReduce đơn giản và mạnh mẽ cho phép tự động song song hóa và phân phối các phép tính quy mô lớn, kết hợp với việc triển khai mô hình lập trình này để đạt được hiệu suất cao trên các cụm máy tính lớn hàng trăm, hàng nghìn máy tính.

## Mô hình lập trình 
Mô hình của mapreduce phải nói là cực kì đơn giản và dễ hình dung. Lập trình viên sẽ chỉ phải viết lại 2 hàm trong mô hình lập trình MapReduce là hàm map và hàm reduce.

Map, nhận đầu vào là cặp key/value tính toán và cho ra một cặp key/value trung gian. Các thư viện cung cấp bởi các framework Mapreduce sẽ gộp lại các giá trị trung gian có cùng key trung gian lại và chuyển tới bước kế tiếp.

Reduce, nhận đầu vào là cặp key và 1 danh sách value có cùng key đã được xử lý ở bước trên và kết hợp các giá trị này với nhau thông qua vòng lặp để đưa ra 1 tập giá trị nhỏ hơn vừa trong bộ nhớ và phù hợp với yêu cầu bài toán.

### Ví dụ 
WordCount là một bài toán kinh điển để minh họa cho MapReduce, ý tưởng của MapReduce được viết giống với đoạn mã giả sau: 
```
map(string key, string value){
	// key: document name
	// value: document value
	for each word w in value{
		ComputeIntermediate(w,"1"); // tính toán các giá trị key/value trung gian
	} 
}

reduce(string key, List values){
	int result = 0;
	for each v in values {
		result += ParseInt(v); // tổng hợp giá trị 
	}
	
	Write(key, result); // ghi kết quả
}
```

Để hình dung rõ hơn, bạn có thể xem hình ảnh sau: 

![](https://i.pinimg.com/564x/a2/16/e2/a216e26d3ce4a95ac211fc92fb84710e.jpg)

Các bạn có thể theo dõi chương trình WordCount được viết bằng Java của mình [TẠI ĐÂY](https://github.com/demanejar/word-count-hadoop)

### Types
Mặc dù đoạn mã giả trên cũng phản ánh khá rõ đầu ra và đầu vào của 2 hàm map, reduce. Tuy nhiên để rõ ràng hơn về mối quan hệ giữa input, output của map, reduce sẽ như sau: 
```
map	: (k1, v1) => list(k2, v2)
reduce	: (k2, list(v2)) => list(v2)
```

## Kết luận 
MapReduce là không phải là mô hình duy nhất giúp song song hóa tốt mà còn rất nhiều các mô hình lập trình khác giúp xử lý dữ liệu lớn bằng cách song song hóa. Tuy nhiên, MapReduce là đơn giản, dễ tiếp cận. Để hiểu hơn nữa về MapReduce bạn có thể tìm hiểu và đọc thêm bài báo của Google [Mapreduce : Simplified Data Processing on Large Clusters](https://github.com/demanejar/download-folder/blob/main/mapreduce-osdi04.pdf)
