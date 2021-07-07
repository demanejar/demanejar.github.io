---
title: Giới thiệu tổng quan Hadoop 
author: trannguyenhan
date: 2021-07-07 20:52:00 +0700
categories: [Blogging, Share]
tags: [Hadoop, Apache Hadoop, Big data, HDFS, Hadoop Yarn]
math: true
mermaid: true
---

*Hadoop là frameword dựa trên 1 giải pháp tới từ Google để lưu trữ và xử lý dữ liệu lớn. Hadoop sử dụng giải thuật MapReduce sử lý song song dữ liệu đầu vào. Hadoop chạy chương trình dựa vào thuật toán MapReduce. Tóm lại, Hadoop được sử dụng để phát triển các ứng dụng có thể thực hiện phân tích thống kê hoàn chỉnh trên dữ liệu số lượng lớn.*

***Xem thêm***: [***Mô hình lập trình Mapreduce cho Bigdata***](https://demanejar.github.io/posts/mapreduce-programming-model/)
## Kiến trúc Hadoop 
Hadoop gồm 2 tầng chính: 
- Tầng xử lý và tinh toán (MapReduce): MapReduce là một mô hình lập trình song song hóa để xử lý dữ liệu lớn trên một cụm gồm nhiều các máy tính thương mại (commodity hardware)
- Tầng lưu trữ (HDFS): HDFS cung cấp 1 giải pháp lưu trữ phân tán cũng được thiết kế để chạy trên các máy tính thương mại

Ngoài 2 thành phần được đề cập ở trên thì Hadoop framework cũng gồm 2 mô-đun sau: 
- Hadoop Common: là các thư viện, tiện ích viết bằng ngôn ngữ Java
- Hadoop Yarn: lập lịch và quản lý các tài nguyên


![](https://i.pinimg.com/564x/4a/3c/14/4a3c144fa89a85fd6dbccc07bdb8509a.jpg)


## Hadoop làm việc như thế nào?
Hadoop giải quyết vấn đề nếu như bạn hay công ty của bạn không đủ điều kiện để xây dựng các máy chủ siêu mạnh thì chúng ta có thể kết hợp nhiều các máy tính thương mại lại để mang lại một cụm máy có khả năng xử lý một lượng dữ liệu lớn.
Các máy trong cụm có thể đọc và xử lý dữ liệu song song để mang lại những hiệu quả cao.
Bạn có thể chạy code trên một cụm các máy tính, quá trình này thông qua luồng sau đây: 
- Dữ liệu được phân chia vào các thư mục và file. Mỗi file được chứa trong 1 blocks có kích thước cố định được xác định sẵn ( mặc định là 128MB)
- Các tệp này được phân phối trên các nút, cụm khác nhau 
- HDFS nằm ở trên cùm của hệ thống file cục bộ, giám sát quá trình 
- Các block được lưu các bản sao để đề phòng quá trình lỗi xảy ra trên phần cứng 
- Kiểm tra mã được thực hiện thành công chưa
- Thực hiện bước sort diễn ra giữa map và reduce
- Gửi data tới các máy nhất định để thực hiện các bước tiếp theo
- Viết log cho mỗi công việc hoàn thành

## Ưu điểm của Hadoop 
- Hadoop cho phép người dùng viết và kiểm tra nhanh trên hệ thống phân tán. Hadoop sử dụng hiệu quả và tự động phân tán dữ liệu và công việc qua nhiều máy trong cùng cụm.
- Hadoop không yêu cầu phần cứng của các máy trong cụm, bất cứ máy tính nào cũng có thể là 1 phần của cụm Hadoop. Hadoop sẽ phân công công việc hợp lý cho mỗi máy phù hợp với khả năng của mỗi máy.
- Hadoop cun cấp hệ thống có khả năng chịu lỗi và tính sẵn có cao. Thay vào đó thư viện Hadoop đã được thiết kế để xử lý lỗi từ tầng ứng dụng.
- Cụm có thể add thêm hoặc remove đi các cluster trong cụm mà không ảnh hưởng tới các tiến trình đang chạy
- Một lợi thế lớn khác của Hadoop là ngoài là mã nguồn mở, nó tương thích trên tất cả các nền tảng vì nó dựa trên Java
