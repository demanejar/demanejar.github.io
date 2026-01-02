---
title: Cài đặt Zeppelin Notebook
author: trannguyenhan 
date: 2021-10-01 20:52:00 +0700
categories: [Hadoop & Spark, Spark]
tags: [Bigdata, Spark, Zeppelin]
math: true
mermaid: true
image:
  src: https://raw.githubusercontent.com/demanejar/image-collection/main/zeppelin/thumbnail.png
---

*Chắc chúng ta quen nhiều hơn với Jupyter notebook và Zeppelin notebook có thể còn chưa được nghe tới bao giờ. Zeppelin notebook hay Apache Zeppelin là một ứng dụng dựa trên web cho phép phân tương tác trực tiếp với SQL, Scala, Python, R và hơn thế nữa.*

### Tải về Zeppelin

Trước hết bạn tải về file cài đặt của zeppelin tại: [https://zeppelin.apache.org/](https://zeppelin.apache.org/), sau đó giải nén ra (lưu ý chọn bản full interpreter ở bên trên):

![](https://raw.githubusercontent.com/demanejar/image-collection/main/zeppelin/download.png)

![](https://raw.githubusercontent.com/demanejar/image-collection/main/zeppelin/download1.png)

### Cấu hình

Bây giờ chúng ta sẽ đi cấu hình một số thông số của zeppelin trong thư mục `/conf`.  Các bạn sẽ thấy trong thư mục này có một số file có đuôi `.template`, đây là các file cấu hình mẫu, chúng ta cần cấu hình cho 2 file `zeppelin-env.sh` và `zeppelin-site.xml` (nếu bạn dùng windows thì bạn sẽ sử dụng file `zeppelin-env.cmd` thay cho file `zeppelin-env.sh`).

Tạo file `zeppelin-env.sh` và copy toàn bộ nội dung từ file `zeppelin-env.sh.template`: 

```bash
cp zeppelin-env.sh.template zeppelin-env.sh
```

Cấu hình JAVA_HOME trong file `zeppelin-env.sh` như sau: 

![](https://raw.githubusercontent.com/demanejar/image-collection/main/zeppelin/java_home.png)

Để tìm được đường dẫn trên bạn làm như sau: 

```bash
which java
```
> output: /usr/bin/java

```bash
readlink -f /usr/bin/java
```
> output:  /usr/lib/jvm/java-11-openjdk-amd64/bin/java

Đường dẫn mà chúng ta cần lấy là: `/usr/lib/jvm/java-11-openjdk-amd64` (bỏ đi `/bin/java`)

Tạo file `zeppelin-site.xml` và copy toàn bộ nội dung từ file `zeppelin-site.xml.template`. Zeppelin mặc định sẽ mở tại cổng `8080`, tuy nhiên cổng này là cổng của Spark, vì thế chúng ta cần đổi cổng cho zeppelin để tránh xung đột, bạn có thể chọn một cổng bất kì, ở đây mình chuyển nó sang cổng `15000`: 

![](https://raw.githubusercontent.com/demanejar/image-collection/main/zeppelin/change_port.png)

### Cấu hình interpreter

Bật zeppelin lên thông qua câu lệnh: 

```bash
bin/zeppelin-daemon.sh start
```

Truy cập vào zeppelin bằng trình duyệt của bạn với đường dẫn [http://localhost:15000/](http://localhost:15000/) (nếu bạn không câu hình zeppelin với cổng `15000`, hãy thay lại cổng của bạn cho hợp lý): 

![](https://raw.githubusercontent.com/demanejar/image-collection/main/zeppelin/zeppelin.png)

Để cấu hình interpreter, các bạn nhấn vào phần tài khoản ở góc phải của màn hình (_anonymouse_) sau đó chọn _interpreter_: 

![](https://raw.githubusercontent.com/demanejar/image-collection/main/zeppelin/interpreter.png)

Chuyển tới phần `spark` và chỉnh sửa lại giá trị của `spark.master`: 

![](https://raw.githubusercontent.com/demanejar/image-collection/main/zeppelin/spark_interpreter.png)

Chúng ta có 5 chế độ chính: 

- `local[*]`:  là chế độ local, chế độ này chúng ta có thể chạy mà không cần bật Spark
- `spark://master:7077`: chuyển lại `master` thành tên master node của bạn, ví dụ như của mình là `spark://PC0628:7077`, đây là chế độ standalone cluster.
- `yarn-client`  là chế độ yarn-client 
- `yarn-cluster`  là chế độ yarn-cluster
- `mesos://host:5050`  là Mesos cluster

Kéo xuống bên dưới và cấu hình lại thuộc tính `PYSPARK_DRIVER_PYTHON` thành đường dẫn tới trình thông dịch python của bạn: 

![](https://raw.githubusercontent.com/demanejar/image-collection/main/zeppelin/pyspark_interpreter.png)

Bây giờ mở zeppelin lên và thực hành thôi: 

![](https://raw.githubusercontent.com/demanejar/image-collection/main/zeppelin/create_new_notebook.png)

![](https://raw.githubusercontent.com/demanejar/image-collection/main/zeppelin/code_zeppelin.png)

Tham khảo: [https://zeppelin.apache.org/](https://zeppelin.apache.org/docs/latest/interpreter/spark.html)
