---
title: Kafka In Depth
author: viethoang
date: 2021-07-08 20:52:00 +0700
categories: [Blogging, Share]
tags: [Big data,Data Ingestion,Apache Kafka]
math: true
mermaid: true
---

*Apache Kafka là một hệ thống được tạo ra bởi linkedin nhằm phục vụ cho việc xử lý dữ liệu theo luồng (stream process) sau đó được open-source. Ban đầu nó được nhìn nhận dưới dạng một message queue nhưng sau này được phát triển thành một nền tảng xử lý phân tán (distributed streamming platform - nhiều cái thuật ngữ dịch ra tiếng Việt khó quá !!).*

## Vấn đề trong dữ liệu lớn
3V trong big data: dữ liệu với khối lượng lớn được sinh ra theo từng giây và định dạng khác nhau ()có cấu trúc, bán cấu trúc, không có cấu trúc ). Để xử lý được vấn đề nêu trên ta cần phải tập hợp được dữ liệu từ các nguồn như hdfs, nosql db, rdbms,... Tuy nhiên làm thế nào mà ta có thể xử lý đồng thời và không bị phụ thuộc lẫn nhau (looosely couple) đối với những nguồn dữ liệu kia -> Kafka là một giải pháp.

![](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSCLS0931rmRrA28steooUgvAJo1sMOc_kPqVrFxLNmV4fydAmGWYNuEY8M5kBR8aA3nfk&usqp=CAU)


## Một số thành phần trong Kafka 

### Tổng quan
![](https://www.cloudkarafka.com/img/blog/durable-message-system.png)
Một ứng dụng kết nối với hệ thống kafka này và gửi thông điệp(bản ghi) tới topic. Thông điệp có thể là bất cứ gì ví dụ như log của hệ thống, sự kiện,... Sau đó một ứng dụng khác kết nối và lấy thông điệp(bản ghi) trong topic. Dữ liệu vẫn được lưu trữ tại ổ đĩa cho tới khi hết thời gian đã được định sẵn (retention time).

### Broker
Ở đây ta đề cập đến cụm kafka, trong đây ta sẽ có các server chạy kafka. Tại đây chứa các topic được producer gửi dữ liệu vào và các consumer sẽ lấy dữ liệu ở đây.
![](https://www.cloudkarafka.com/img/blog/kafka-broker-beginner.png)
Việc quản lý các broker sẽ được thực hiện bởi zookeeper. Trong một cụm kafka cũng có thể có nhiều hơn 1 zookeeper 

### Topic
Topic là nơi mà bản ghi dữ liệu được lưu lại và sử dụng.

Các bản ghi được đưa vào topic sẽ có 1 "offset" là chỉ số vị trí của chúng để có thể giúp consumer xác định được và lấy ra. Consumer có thể điều chỉnh cái vị trí này để lấy ra theo thứ tự mong muốn ví dụ như khi ta muốn xử lý từ đầu thì có thể đặt offset sớm nhất.

#### Topic partition
Như đã nói ở phần trước thì mỗi bản ghi đều có 1 offset riêng và trong topic ta có khái niệm partition tức phân mảnh. Một topic có thể có nhiều partition và các bản ghi sẽ được lưu trong partition đó với giá trị offset. Điều này sẽ giúp consumer đọc dữ liệu từ topic song song, ví dụ 1 topic có 4 partitions ta có 1 consumer và consumer này sẽ đọc từ 4 partitions của topic trên.

![](https://miro.medium.com/max/700/0*vJkjcHVs9ZAN42Gt.png)

Các partition của topic có thể nằm trên các broker khác nhau vậy nên điều này sẽ giúp cho topic có thể truy cập được từ nhiều broker.

Việc sao lưu dữ liệu của topic là khi ta cài đặt replication-factor lúc tạo topic. Ví dụ như khi ta tạo một topic với 3 partitions và replication-factor là 3 thì sẽ được kết quả như hình dưới. Điều này đảm bảo tính khả dụng (availability)

![](https://vsudo.net/blog/wp-content/uploads/2019/12/partition-kafka-800x378.jpg)

Cơ chế ở đây sẽ là Leader - Follower (partitions). Khi dữ liệu được đẩy vào leader sẽ chịu trách nhiệm phần đọc ghi. Các follower chỉ đóng vai trò sao chép lại, ta cũng không thể đọc được  vì kafka không cho phép. Khi leader sập thì sẽ bầu ra leader mới từ các follower và đồng bộ lại dữ liệu.
![](https://vsudo.net/blog/wp-content/uploads/2019/12/leader-replica-kafka-800x488.jpg)
Việc lưu trữ Leader - Follower sẽ được lưu vào metadata được quản lý bởi zookeeper
### Producer
Producer có nhiệm vụ gửi dữ liệu vào topic trong broker.
#### Producer ghi một thông điệp như thế nào ?
Trong cụm kafka, producer chỉ ghi dữ liệu vào partition leader của topic. Đối với một topic nói riêng ta không nói đến việc leader - follower ở đây, nếu có 1 dữ liệu mới vào mà topic producer gửi có nhiều hơn 1 topic thì chỉ có 1 partition nhận được dữ liệu đó.

3 cách mà producer có thể ghi vào topic:
* send(key,value,topic,partition): chỉ định rõ partition mà message được ghi vào.
* send(key,value,topic): Sẽ có 1 hàm hash tiến hành tính giá trị của key để đưa vào partition phù hợp.
* send(key,value): Dữ liệu được đưa vào partition theo cơ chế round-robin

### Consumer
Consumer lấy dữ liệu trong các topic trong broker mà nó subcribe. Consumer có thể đọc bất cứ offset nào trong topic vậy nên nó có thể vào trong cụm bất cứ lúc nào
#### Consumer lấy thông điệp như thế nào ?
Giống như producer consumer phải tìm kiếm trước metadata và đọc dữ liệu từ leader-partition. Nếu dữ liệu mà lớn thì có thể consumer sẽ bị lag -> để giải quyết Consumer Group.

Consumer group là một nhóm các consumer cùng id. Trong đó mỗi consumer sẽ chỉ gán với 1 partitions. Nếu số lượng partition lớn hơn số consumer thì consumer có thể nhận được nhiều hơn, nếu nhỏ hơn thì sẽ có consumer không nhận được dữ liệu. Việc gán partition cho consumer sẽ được chịu trách nhiệm bởi Group Coordinator - 1 trong các broker của cụm

![](https://www.oreilly.com/library/view/kafka-the-definitive/9781491936153/assets/ktdg_04in05.png)

## Lưu trữ dữ liệu trong kafka
Tất cả dữ liệu được lưu trên ổ đĩa chứ không phải trên ram và được sắp xếp tuần tự theo 1 cấu trúc dữ liệu là log. Việc này vẫn khiến kafka có thể nhanh như vậy là bởi một số lý do dưới đây:
* Các message được nhóm lại với nhau tạo thành 1 segment -> giảm chi phí sử dụng tài nguyên mạng, consumer sẽ tìm 1 segment -> giảm tải disk
* Dữ liệu lưu dưới dạng nhị phân làm cho nó tối ưu khả năng zero-copy
* Các dữ liệu tại producer đã được nén dùng các thuật toán nén như gzip và sau đó giải nén tại consumer
* Việc đọc/ghi trên cấu trúc dữ liệu log là 0(1) (No random disk access)

![](https://miro.medium.com/max/700/1*E3b9_nT-2PbZERMo0ArspA.png)


## Usecase
Đối với trang thương mại điện tử, mỗi click hay các hành động của người dùng trên trang đó có thể phản án thói quen mua sắm của họ. Các dữ liệu này sẽ được đưa vào kafka để xử lý theo thời gian thực giúp đưa ra gợi ý ngay tại họ thời điểm họ trên trang web. Sau đó việc xử lý theo luồng sẽ được tổng hợp lại với dữ liệu mới để có thể gợi ý với lần vào tiếp theo. Việc sử dụng kafka sẽ không liên quan đến cơ sở dữ liệu hệ thống nên đảm bảo được ổn định.

![](https://www.cloudkarafka.com/img/blog/apache-kafka-web-shop.png)

Còn khá nhiều ứng dụng nữa nhưng mình không muốn tập trung vào phần này, mọi người có thể tìm hiểu thêm.
## Cài đặt và sử dụng Kafka
## Tham khảo
https://www.cloudkarafka.com/blog/part1-kafka-for-beginners-what-is-apache-kafka.html
https://radiant-brushlands-42789.herokuapp.com/betterprogramming.pub/a-simple-apache-kafka-cluster-with-docker-kafdrop-and-python-cf45ab99e2b9
https://sonusharma-mnnit.medium.com/apache-kafka-in-depth-49aae1e844be
https://medium.com/@durgaswaroop/a-practical-introduction-to-kafka-storage-internals-d5b544f6925f#:~:text=Kafka%20stores%20all%20the%20messages,also%20called%20as%20the%20Offset%20.
https://vsudo.net/blog/kafka-la-gi.html
