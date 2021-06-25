---
title: Mô hình lập trình Mapreduce cho Bigdata 
author: trannguyenhan
date: 2021-06-25 08:00:00 +0700
categories: [Blogging, Share]
tags: [Ubuntu, Mapreduce, Bigdata, Java]
math: true
mermaid: true
---
*Mapreduce là một mô hình lập trình để triển khai và xử lý dữ liệu lớn. Lập trình viên sẽ phải viết 2 hàm là map và reduce, trong đó hàm map xử lý các cặp key/value đầu vào cho ra kết quả là các cặp key/value trung gian, và hàm reduce sẽ gộp tất cả value trung gian có cùng key trung gian lại với nhau. WordCount là một ví dụ điển hình cho Mapreduce mà sẽ được mình minh họa trong bài viết này*

## Tại sao Mapreduce lại ra đời?
Như phần giới thiệu thì ai cũng biết là mapreduce là viết gộp lại của map và reduce là 2 hàm chính trong phương pháp lập trình này mà lập trình viên phải viết. Mô hình lập trình này bắt nguồn chính là từ Google, hay rõ ràng hơn là từ 1 bài báo của Google. Vấn đề đặt ra là cần phải song song hóa các tính toán, phân tán dữ liệu và phải có khả năng chịu lỗi cao. 

Mapreduce đơn giản và mạnh mẽ cho phép tự động song song hóa và phân phối các phép tính quy mô lớn, kết hợp với việc triển khai mô hình lập trình này để đạt được hiệu suất cao trên các cụm máy tính lớn hàng trăm, hàng nghìn máy tính.

## Mô hình lập trình 
Mô hình của mapreduce phải nói là cực kì đơn giản và dễ hình dung. Chương trình mapreduce nhận đầu vào là 1 cặp key/value và cho ra kết quả là 1 cặp key/value. Lập trình viên sẽ chỉ phải viết lại 2 hàm trong mô hình lập trình mapreduce là hàm map và hàm reduce.

Map, nhận đầu vào là cặp key/value tính toán và cho ra một cặp key/value trung gian. Các thư viện cung cấp bởi các framework Mapreduce sẽ làm những việc tiếp theo là tổng hợp những value có cùng key lại và chuyển qua bước reduce tiếp theo.

Reduce, nhận đầu vào là cặp key và 1 danh sách value có cùng key đã được xử lý ở bước trên và đưa ra kết quả theo yêu cầu của bài toán.

Mô hình lập trình mapreduce được viết giống như 2 đoạn mã giả sau với ví dụ bài toán WordCount:
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

Các bạn có thể theo dõi chương trình WordCount được viết bằng Java của mình [TẠI ĐÂY](https://github.com/demanejar/word-count-hadoop)

## Kết luận 
Mapreduce là không phải là mô hình duy nhất giúp song song hóa tốt mà còn rất nhiều các mô hình lập trình khác giúp xử lý dữ liệu lớn bằng cách song song hóa. Tuy nhiên, mapreduce là đơn giản, dễ tiếp cận. Để hiểu hơn nữa về mapreduce bạn có thể tìm hiểu và đọc thêm bài báo của Google [Mapreduce : Simplified Data Processing on Large Clusters](https://github.com/demanejar/download-folder/blob/main/mapreduce-osdi04.pdf)
