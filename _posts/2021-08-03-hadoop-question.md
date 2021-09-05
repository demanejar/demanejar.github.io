---
title: Tổng hợp các câu hỏi về Apache Hadoop 
author: trannguyenhan
date: 2021-08-03 20:52:00 +0700
categories: [Apache, Hadoop]
tags: [Hadoop, Apache Hadoop, Bigdata, HDFS, Hadoop Yarn]
math: true
mermaid: true
image:
  src: https://i.pinimg.com/originals/4a/3c/14/4a3c144fa89a85fd6dbccc07bdb8509a.jpg
---
#### Mục tiêu chính của Apache Hadoop 
Lưu trữ dữ liệu khả mở và xử lý dữ liệu mạnh mẽ. Tiết kiệm chi phí khi lưu trữ và xử lý lượng dữ liệu lớn.

Bạn có thể xem thêm chi tiết mục tiêu của Hadoop [TẠI ĐÂY](https://demanejar.github.io/posts/hdfs-introduction/#m%E1%BB%A5c-ti%C3%AAu-c%E1%BB%A7a-hdfs)

#### Hadoop giải quyết bài toán chịu lỗi thông qua kỹ thuật gì
- Hadoop chịu lỗi thông qua kỹ thuật dư thừa
- Các tệp tin được phân mảnh, các mảnh được nhân bản ra các node khác trên cụm
- Các công việc cần tính toán được phân mảng thành các tác vụ độc lập 

#### Mô tả cách thức 1 client được dữ liệu trên HDFS
Client truy vấn namenode để biết được vị trí các chunks. Namenode trả về vị trí các chunks. Client kết nối song song với các datanode để đọc các chunk.

Bạn có thể xem chi tiêt quá trình đọc dữ liệu này [TẠI ĐÂY](https://demanejar.github.io/posts/hdfs-introduction/#read-data)

#### Mô tả cách thức 1 client ghi dữ liệu trên HDFS
Client kết nối tới namenode để chỉ định khối lượng dữ liệu cần ghi. Namnode chỉ định vị trí các chunk cho client. Client khi chunk tới datanode đầu tiền, sau đó các datanode tự động thực thi nhân bản. Quá trình kết thúc khi tất cả các chunk và nhân bản đã được thực thi thành công. 

Bạn có thể xem chi tiêt quá trình ghi dữ liệu này [TẠI ĐÂY](https://demanejar.github.io/posts/hdfs-introduction/#write-data)

#### Các thành phần chính trong Hadoop Ecosystem
Hadoop Ecosytem là một nền tảng cung cấp các giải pháp để lưu trữ và xử lý lượng lớn dữ liệu. 
Các thành phần chính trong Hadoop Ecosytem là: 
- HDFS 
- Mapreduce framework 
- YARN
- Zookeeper

#### Cơ chế chịu lỗi của datanode trong HDFS
*Sử dụng cơ chế heartbeat*, định kì datanode sẽ gửi thông báo về trạng thái cho namenode. Khoảng thời gian mặc định mà datanode gửi heartbeat về cho namenode là 3s, sau 3s mà datanode không có gửi thông tin về cho namenode thì mặc định namenode coi là node đó đã chết và nhiệm vụ chưa hoàn thành của node đó sẽ được trao lại cho node mới.

Để cấu hình lại thời gian heartbeat thì bạn có thể thêm đoạn XML sau vào trong file `hdfs-site.xml` như sau: 
```xml
<property>
<name>dfs.heartbeat.interval</name>
<value>3</value>
</property>

<property>
<name>dfs.namenode.heartbeat.recheck-interval</name>
<value>300000</value>
</property>
```

Bạn có thể xem các cấu hình mặc định của `hdfs-site.xml` [TẠI ĐÂY](https://hadoop.apache.org/docs/r2.7.0/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml)

#### Cơ chế tổ chức dữ liệu của datanode trong HDFS
Files trong HDFS được chia thành các khối có kích thước cố định được ( block-sized chunks) gọi là data block. Các block được lưu trữ như các đơn vị độc lập (kích thước của 1 block mặc định là 128MB).
 
 Các chunk là các tập tin hệ thống trong tập tin cục bộ của máy chủ datanode
 
 *Xem thêm:* [*Vấn đề gì xảy ra nếu lưu trữ các file nhỏ trên HDFS?*](https://demanejar.github.io/posts/hdfs-introduction/#blocks)
 
#### Cơ chế nhân bản dữ liệu trong HDFS
Các block của file được nhân bản để tăng khả năng chịu lỗi. Namenode là node đưa ra tất cả các quyết định đến việc nhân rộng các khối. 

#### HDFS giải quyêt bài toán single-point-of-failure cho namenode bằng cách nào 
Sử dụng Secondary Namenode theo cơ chế active-passive. Secondary Namenode chỉ hoạt động khi có vấn đề với Namenode

Các bạn có thể xem chi tiết về Secondary Namenode [TẠI ĐÂY](https://demanejar.github.io/posts/hdfs-introduction/#secondary-namenode)

#### 3 chế độ mà Hadoop có thể chạy? 
- *Standalone mode*: đây là chế độ mặc định, Hadoop sử dụng local FileSystem và 1 tiến trình Java duy nhất để chạy các dịch vụ Hadoop.
- *Pseudo-distributed mode*: triển khai Hadoop trên 1 node để thực thi tất cả các dịch vụ.
- *Fully-distributed mode*: triển khai Hadoop trên 1 cụm máy với namenode và datanode.

#### Giải thích Bigdata và tiêu chí 5V của Bigdata
Bigdata là thuật ngữ chỉ tập dữ liệu lớn và phức tạp và rất khó để xử lý bằng các công cụ dữ liệu quan hệ, các ứng dụng xử lý dữ liệu truyền thống. 

5V trong Bigdata là: 
- *Volume*: Volume thể hiện lượng dữ liệu đang tăng với tốc độ cấp số nhân, tức là bằng Petabyte và Exabyte.
- *Velocity*: Velocity đề cập đến tốc độ dữ liệu đang phát triển, rất nhanh. Hôm nay, dữ liệu của ngày hôm qua được coi là dữ liệu cũ. Ngày nay, mạng xã hội là một yếu tố đóng góp lớn vào tốc độ phát triển của dữ liệu.
- *Variety*: Variety đề cập đến sự không đồng nhất của các kiểu dữ liệu. Nói cách khác, dữ liệu được thu thập có nhiều định dạng như video, âm thanh, csv, v.v. Vì vậy, các định dạng khác nhau này đại diện cho nhiều loại dữ liệu.
- *Veracity*: Veracity đề cập đến dữ liệu bị nghi ngờ hoặc không chắc chắn của dữ liệu có sẵn do dữ liệu không nhất quán và không đầy đủ. Dữ liệu có sẵn đôi khi có thể lộn xộn và khó tin cậy. Với nhiều dạng dữ liệu lớn, chất lượng và độ chính xác rất khó kiểm soát. Khối lượng thường là lý do đằng sau sự thiếu chất lượng và độ chính xác của dữ liệu.
- *Value*: Tất cả đều tốt và tốt khi có quyền truy cập vào dữ liệu lớn nhưng trừ khi chúng ta có thể biến nó thành một giá trị.
