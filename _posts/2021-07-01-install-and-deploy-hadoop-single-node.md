---
title: Cài đặt và triển khai Hadoop single node 
author: trannguyenhan
date: 2021-07-01 16:00:00 +0700
categories: [Hadoop & Spark, Hadoop]
tags: [Hadoop, Apache Hadoop, Bigdata, HDFS, Hadoop Yarn]
math: true
mermaid: true
image:
  src: https://raw.githubusercontent.com/demanejar/image-collection/refs/heads/main/HadoopIntro/hadoop_intro_banner.jpg
---

*Mỗi ngành công nghiệp lớn đang triển khai Apache Hadoop là khung tiêu chuẩn để xử lý và lưu trữ dữ liệu lớn. Hadoop được thiết kế để được triển khai trên một mạng lưới hàng trăm hoặc thậm chí hàng ngàn máy chủ chuyên dụng. Tất cả các máy này làm việc cùng nhau để đối phó với khối lượng lớn và nhiều bộ dữ liệu khổng lồ*

**_Xem thêm_**: [Giới thiệu tổng quan Hadoop](https://demanejar.github.io/posts/hadoop-introduction/)

![](https://raw.githubusercontent.com/demanejar/image-collection/refs/heads/main/HadoopInstallAndDeploy/hadoop_logo_intro.jpg)

Hadoop mạnh mẽ và hữu dụng chỉ khi được cài đặt và khai thác nó trên nhiều node, tuy nhiên với người bắt đầu thì Hadoop Single node là sự khởi đầu tuyệt vời để làm quen với hadoop. Bài viết này mình sẽ hướng dẫn các bạn triển khai Hadoop trên 1 node ( Hadoop Single node).

#### Điều kiện trước khi cài

*   Máy bạn phải có jdk ( bản 8, 11 hay 15 đều được, lưu ý nếu bạn dùng hadoop 3.1.4 thì có thể dùng bản jdk8, còn nếu dùng hadoop 3.2.2 trở lên thì hãy sử dụng java11 trở lên), nếu chưa có thì bạn có thể cài theo câu lệnh sau: 

```bash
sudo apt-get install openjdk-11-jdk -y
```

*   Máy có SSH client và SSH server, nếu chưa có thì bạn có thể cài theo câu lệnh sau: 

```bash
sudo apt-get install openssh-server openssh-client -y
```

## Thiết lập User cho Hadoop

Thường mình thấy một nhiều trang hướng dẫn các bạn tạo một user mới trên Ubuntu và cài Hadoop trên đó nhưng qua trải nhiệm của mình thì cài Single Node chúng ta có thể cài trên bất cứ tài khoản nào, bạn có thể cài đặt ngay trên tài khoản admin mà bạn đang dùng hiện tại.  

Tạo cặp khóa SSH và xác định vị trí sẽ được lưu trữ: 

```bash
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
```

Hệ thống sẽ tiến hành tạo và lưu cặp khóa SSH: 

![](https://raw.githubusercontent.com/demanejar/image-collection/refs/heads/main/HadoopInstallAndDeploy/ssh_gen_key.png)

Sử dụng lệnh **_cat_** để lưu **_public key_** vào **_authorized\_keys_** trong thư mục của SSH:

```bash
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

Phân quyền cho người dùng với lệnh _**chmod**_:

```bash
chmod 0600 ~/.ssh/authorized_keys
```

Xác minh mọi thứ được thiết lập chính xác bằng cách ssh đến localhost: 

```bash
ssh localhost
```

## Download và Cài đặt Hadoop trên Ubuntu

Tải về một phiên bản Hadoop trên trang phân phối chính thức của Hadoop tại : [https://hadoop.apache.org/releases.html](https://hadoop.apache.org/releases.html)

![](https://raw.githubusercontent.com/demanejar/image-collection/refs/heads/main/HadoopInstallAndDeploy/version_download_hadoop.png)

Ấn vào phần **_binary_** trong **_Binary download_**

Bây giờ để file nén mà bạn vừa tải về vào bất kì chỗ nào và giải nén nó ra bằng lệnh:

```bash
tar xvzf hadoop-3.2.2.tar.gz
```

## Cấu hình và triển khai Hadoop Single Node (Pseudo-Distributed Mode)

Để cấu hình Hadoop cho chế độ phân phối giả chúng ta sẽ chỉnh sửa các tập file cấu hình của Hadoop trong đường dẫn **_etc/hadoop_** và trong **_file cấu hình môi trường_** gồm các file sau:

*   `.bashrc`
*   `hadoop-env.sh`
*   `core-site.xml`
*   `hdfs-site.xml`
*   `mapred-site.xml`
*   `yarn-site.xml`

**_Lưu ý_**: Trong phần cài đặt dưới đây, Hadoop của mình được đặt trong thư mục `/opt/myapp`, các bạn có thể để Hadoop ở bất cứ đâu cũng được, không nhất thiết phải để giống như mình. 

### Cấu hình biến môi trường Hadoop ( file .bashrc)

Mở file của .bashrc của bạn bằng trình soạn thảo nano:

```bash
sudo nano ~/.bashrc
```

Xác định biến mỗi trường **_Hadoop_** bằng cách thêm các biến sau vào cuối file ( nhớ chỉnh sửa đường dẫn **_$HADOOP\_HOME_** cho đúng với đường dẫn mà bạn đã đặt hadoop) 

```bash
#Hadoop Related Options 
JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64 
export HADOOP_HOME=/opt/myapp/hadoop-3.2.2 
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME 
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME 
export HADOOP_YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native" 
```

Áp dụng các thay đổi trên bằng cách thực hiện lệnh sau:

```bash
source ~/.bashrc
```

### Chỉnh sửa file hadoop-env

Mở file `hadoop-env.sh` bằng trình soạn thảo nano:

```bash
sudo nano $HADOOP_HOME/etc/hadoop/hadoop-env.sh
```

Tìm tới vị trí như hình bên dưới, bỏ comment ( bỏ dấu #) phần **JAVA\_HOME** và thêm vào đầy đủ đường dẫn openjdk trên máy của bạn: 

![](https://raw.githubusercontent.com/demanejar/image-collection/refs/heads/main/HadoopInstallAndDeploy/config_openjdk_hadoop.png)

### Chỉnh sửa file core-site.xml

Mở file `core-site.xml` bằng trình soạn thảo nano:

```bash
sudo nano $HADOOP_HOME/etc/hadoop/core-site.xml
```

Thêm vào giữa 2 thẻ `configuration` để được nội dung đầy đủ như sau:

```xml
<configuration>
<property>
	<name>hadoop.tmp.dir</name>
	<value>/opt/myapp/hadoop-3.2.2/tmpdata</value>
</property>
<property> 
	<name>fs.default.name</name> 
	<value>hdfs://localhost:9000</value>
</property>
</configuration>
```

`fs.default.name` cấu hình địa chỉ của HDFS, nếu không cấu hình mặc định nó sẽ được đặt tại cổng 9000, nếu bị trùng cổng thì hãy thay đổi nó sang cổng khác để Hadoop có thể hoạt động bình thường. 

### Chỉnh sửa file hdfs-site.xml

Mở file `hdfs-site.xml` bằng trình soạn thảo nano: 

```bash
sudo nano $HADOOP_HOME/etc/hadoop/hdfs-site.xml
```

Thêm vào giữa 2 thẻ `configuration` để được nội dung đầy đủ như sau:

```xml
<configuration>
<property>
	<name>dfs.data.dir</name> 
	<value>/opt/myapp/hadoop-3.2.2/dfsdata/namenode</value>
</property>
<property>
	<name>dfs.data.dir</name>
	<value>/opt/myapp/hadoop-3.2.2/dfsdata/datanode</value>
</property> 
<property>
	<name>dfs.replication</name> 
	<value>2</value>
</property>
</configuration>
```

`dfs.replication` cấu hình số bản sao, thường thì 3 là con số thường được chọn tuy nhiên khi cài single node chỉ mang tính chất học là chủ yếu thì bạn để bao nhiêu cũng được.

### Chỉnh sửa file mapred-site.xml

Mở file `mapred-site.xml` bằng trình soạn thảo nano:

```bash
sudo nano $HADOOP_HOME/etc/hadoop/mapred-site.xml
```

Thêm vào giữa 2 thẻ `configuration` để được nội dung đầy đủ như sau:

```xml
<configuration>
<property>
	<name>mapreduce.framework.name</name>
	<value>yarn</value>
</property>
</configuration>
```

### Chỉnh sửa file yarn-site.xml

Mở file `yarn-site.xml` bằng trình soạn thảo nano: 

```bash
sudo nano $HADOOP_HOME/etc/hadoop/yarn-site.xml
```

Thêm vào giữa 2 thẻ configuration để được nội dung đầy đủ như sau:

```xml
<configuration> 
<property>
	<name>yarn.nodemanager.aux-services</name>
	<value>mapreduce_shuffle</value>
</property>
<property> 
	<name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
	<value>org.apache.hadoop.mapred.ShuffleHandler</value>
</property>
<property>
	<name>yarn.resourcemanager.hostname</name>
	<value>127.0.0.1</value>
</property>
<property>
	<name>yarn.acl.enable</name>
	<value>0</value>
</property> 
<property>
	<name>yarn.nodemanager.env-whitelist</name>  
	<value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PERPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
</property>
</configuration>
```

### Format HDFS namenode

Các bạn phải định dạng lại namenode trước khi bắt đầu các dịch vụ đầu tiên:

```bash
hdfs namenode -format
```

### Start Hadoop Cluster

Tại thư mục sbin, thực hiện các lệnh sau để chạy và khởi động Hadoop:

```bash
./start-all.sh
```

Chạy lệnh jsp để kiểm tra các trình daemon đang chạy: 

```bash
jps
```

Nếu kết quả ra 6 trình daemon như sau thì bạn đã cấu hình đúng (các bạn để ý tới 6 daemon bên trên thôi nha, XMLServerLauncher các bạn không cần quan tâm tới nó): 

![](https://raw.githubusercontent.com/demanejar/image-collection/refs/heads/main/HadoopInstallAndDeploy/check_daemon_hadoop.png)

### Truy cập Hadoop UI từ trình duyệt 

Các bạn có thể kiểm tra Hadoop đã được cài đặt thành công hay chưa tại cổng mặc định của namenode là `9870`: 

```bash
localhost:9870
```

![](https://raw.githubusercontent.com/demanejar/image-collection/refs/heads/main/HadoopInstallAndDeploy/localhost_9870_hadoop.png)

Kiểm tra datanode tại cổng mặc định `9864`: 

```bash
localhost:9864
```

![](https://raw.githubusercontent.com/demanejar/image-collection/refs/heads/main/HadoopInstallAndDeploy/localhost_9864_hadoop.png)

Kiểm tra Yarn resource manager tại cổng `8088`: 

```bash
locahost:8088
```

![](https://raw.githubusercontent.com/demanejar/image-collection/refs/heads/main/HadoopInstallAndDeploy/localhost_8088_hadoop.png)

Tham khảo: [https://phoenixnap.com/](https://phoenixnap.com/kb/install-hadoop-ubuntu)
