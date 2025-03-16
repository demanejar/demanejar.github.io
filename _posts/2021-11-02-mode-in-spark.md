---
title: Giải thích về các chế độ chính khi chạy Spark
author: trannguyenhan 
date: 2021-11-02 20:52:00 +0700
categories: [Hadoop & Spark, Spark]
tags: [Bigdata, Spark, Yarn, Mesos, Apache Mesos]
math: true
mermaid: true
image:
  src: https://raw.githubusercontent.com/demanejar/image-collection/main/ModeInSpark/run_spark_mode.png
---

Khi sử dụng Spark các bạn có thể thấy có rất là nhiều các chế độ khác nhau như _local_, _standalone_, _yarn_,... chắc hẳn rất nhiều người còn chưa hiểu rõ về các chế độ này nhất là khi mình sử dụng các chế độ trong từng bài viết của mình trên blog, trong bài viết này mình sẽ nói  rõ về các chế độ và tại sao trong các bài viết có lúc mình minh họa bằng chế độ _local_, lúc thì lại minh họa bằng chế độ _standalone_, mục đích chính của bài viết là để các bạn hiểu rõ hơn khi đọc các bài viết trên blog của mình.

## Các chế độ chính 

Chúng ta có 5 chế độ chính trong Spark đó là:

-   `local[*]`: là chế độ _local_, chế độ này chúng ta có thể chạy mà không cần bật Spark
-   `spark://master:7077`: chuyển lại `master` thành tên master node của bạn, ví dụ như của mình là `spark://PC0628:7077`, đây là chế độ _standalone_ cluster.
-   `yarn-client` là chế độ _yarn-client_
-   `yarn-cluster` là chế độ _yarn-cluster_
-   `mesos://host:5050` là _mesos cluster_

## Cách sử dụng

Trong blog của mình các bạn có thể thấy khi thì mình sử dụng chế độ _local_ ( là lúc mình set master là `local`, `local[2]` hoặc `local[*]`), lúc thì mình sử dụng chế độ _standalone_ ( là lúc mình set master là  `spark://master:7077` với `master` tương ứng là tên master node của cụm Spark), và trong phần này mình sẽ giải thích kĩ càng tại sao trong từng bài viết mình lại sử dụng các chế độ này, không chỉ blog của mình mà rất nhiều các blog khác kể cả các blog tiếng Việt hay tiếng Anh đều đang sử dụng như vậy.

Các bạn có thể thấy với những bài viết mang tính chất cài đặt hay là kiểu thiên về quy trình như các bài viết: 

- [Cài đặt Apache Spark standalone](https://demanejar.github.io/posts/install-apache-spark-ubuntu/)
- [Chương trình Word Count với spark-submit và spark-shell](https://demanejar.github.io/posts/word-count-with-spark-submit-and-spark-shell/)
- ...

thì mình thường sử dụng chế độ _standalone_ tức là mình sẽ set master tương ứng là  `spark://master:7077`, còn một số bài viết kiểu ví dụ minh họa hay các bài viết hướng dẫn cách làm tức là chúng ta chú trọng tới cách giải quyết bài toán đó như các bài viết: 

- [Phân tích dữ liệu bán lẻ với Spark SQL](https://demanejar.github.io/posts/retail-data-analytics-with-spark-sql/)
- [Window function, pivot trong Spark SQL](https://demanejar.github.io/posts/spark-sql-window-function-pivot/)
- ...

thì mình lại thường sử dụng chế độ _local_ tức là mình sẽ set master là `local` (`local[2]`  hoặc `local[*]`).

### Chế độ _local_

Các bạn để ý thấy là ở chế độ _local_ khi chạy `spark-shell`, `spark-submit` hay bất cứ gì trên môi trường spark các bạn đều không phải bật Spark lên, và cũng vì thế nên các thông tin của job, của cụm tại các cổng UI của Spark như `8080` đều sẽ không có gì.

Với những tính chất như mình đề cập ở trên thì rõ ràng chế độ _local_ này phù hợp với những trường hợp mà các bạn làm bài tập và muốn chú ý tới kết quả hoặc là cách làm tức là nói về hướng giải quyết của bài toán. Ví dụ như bạn có một bài toán phân tích, và bạn làm và muốn kiểm tra xem kết quả với cách làm của mình đúng chưa thì sử dụng chế độ _local_ là hợp lí nhất, các bạn sẽ chả cần phải bật cụm mà có thể chạy ngay, thấy kết quả và kiểm tra được ngay.

### Chế độ _standalone_

Muốn chạy ở chế độ này bạn phải bật cụm Spark của bạn lên và mỗi khi chạy `spark-shell` hay `spark-submit` thì bạn phải set master của bạn tương ứng là `spark://master:7077`.

Khi chạy ở chế độ này các bạn có thể thoải mái xem các cấu hình cụm, thông tin job qua các cổng UI của Spark như là cổng `8080`.

Thế nên trong các trường hợp như hướng dẫn cài đặt hay kiểu các bài viết về quy trình thì _standalone_ là chế độ hợp lí để lựa chọn. Ví dụ bây giờ bạn có 1 job WordCount, các bạn cũng biết WordCount là một job kinh điển và nó cực kì dễ, nếu như job này các bạn muốn hiểu về các tính toán của nó, giải thuật của nó như nào, cách làm của nó ra sao thì đơn giản các bạn chỉ cần viết nó và chạy nó trên chế độ _local_ và kiểm tra kết quả chứ không nhất thiết phải bật cụm Spark lên làm gì. Nhưng nếu các bạn muốn dùng job WordCount này để minh họa cách sử dụng Spark từ A-Z, chạy trên cụm thì cấu hình như nào, sau khi chạy xong xem job ở đâu, thông tin job ra sao thì bạn nên sử dụng _standalone_ thay vì _local_.

Tất nhiên là ở chế độ _standalone_ thì các bạn có thể làm được những gì mà _local_ làm được, việc lựa chọn chỉ là sự phù hợp hơn trong từng trường hợp mà thôi.

### Chọn chế độ nào

Qua từng phần mình cũng đã giải thích rõ ràng nên chắc các bạn cũng biết là mình sử dụng các chế độ như nào rồi. Trong các bài viết, khi tới phần minh họa mà mình set master là `local` thì các bạn hiểu là bài viết đó mình muốn truyền tải tới các bạn là cách làm, cách giải quyết bài toán đó. Còn khi mình set master là `spark://master:7077` (chế độ _standalone_) thì thường (mình nhấn mạnh là thường nha, vì đơn giản những gì _local_ làm được thì _standalone_ vẫn làm được) là các bài viết kiểu hướng dẫn cài đặt, quy trình tổng quát. 

### Chế độ _yarn_ và _mesos_

![](https://raw.githubusercontent.com/demanejar/image-collection/main/ModeInSpark/mesos_vs_yarn.png)

Trên thực thế thì Spark hay Hadoop đều là các framework sinh ra để chạy phân tán trên nhiều máy vì thế các chương trình và tài nguyên đều phải được chạy và lưu trữ trên các máy trong cụm. Hadoop có một trình quản lý tài nguyên riêng được gọi là YARN. Và khi chúng ta chạy Spark với tài nguyên được lưu trữ trên HDFS thì có thể tận dụng YARN làm trình quản lý tài nguyên. 

Apache Mesos thì cũng giống yarn vậy, cả yarn và mesos đều là quản lý tài nguyên phân tán có mục đích chung và hỗ trợ nhiều loại công việc như MapReduce, Spark, Flink, Storm,... Chúng là rất tốt và phù hợp cho các cụm quy mô lớn tới rất lớn.

