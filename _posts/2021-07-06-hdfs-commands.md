---
title: Các câu lệnh thao tác với file và thư mục trên HDFS 
author: trannguyenhan
date: 2021-07-06 16:00:00 +0700
categories: [Hadoop & Spark]
tags: [Hadoop, Apache Hadoop, Bigdata, HDFS]
math: true
mermaid: true
image:
  src: https://i.pinimg.com/originals/de/61/7d/de617d1ce71f621bbeba8b293996e9fc.jpg
---
Các câu lệnh trên HDFS nhìn chung khá là giống với các câu lệnh trên Linux kể cả về chức năng lẫn tên của chúng, nếu bạn nào đã quen với Linux/Ubuntu rồi thì chắc cũng không cần phải học gì nhiều đâu, cứ thế áp dụng vào thôi.

### help : 
Muốn xin sự trợ giúp về common line trong HDFS : 
```bash
hdfs dfs -help
hdfs dfs -usege <utility_name>
```

Xin sự trợ giúp về 1 lệnh cụ thể nào đó : 
```bash
hdfs dfs -help <statement>
VD: hdfs dfs -help ls
```

### mkdir
Tạo folder trong HDFS : 
```bash
hdfs dfs -mkdir /newfolder
```	

Nếu muốn tạo thư mục ngay cả khi nó đã tồn tại thì thêm tham số : 
```bash
hdfs dfs -mkdir -p /newfolder
```

### ls
Kiểm tra các thư mục có trong hệ thống : 
```bash
hdfs dfs -ls /
```

Muốn kiểm tra tất cả các thư mục con chứa trong hdfs thì sử dụng câu lệnh : 
```bash
hdfs dfs -ls -R /
```

### put 
Sử dụng lệnh put để copy 1 file từ thư mục local vào hdfs : 
```bash
hdfs dfs -put ~/test.txt /newfolder/
hdfs dfs -copyFromLocal ~/test.txt /newfolder/
```

### get
Sử dụng lệnh để lấy 1 file từ hdfs về thư mục local : 
```
hdfs dfs -get /newfolder/test.txt /copyfromhdfs/
hdfs dfs -copyToLocal /newfolder/test.txt /copyfromhdfs/
```

### cat
Kiểm tra nội trung trong file : 
```bash
hdfs dfs -cat /newfolder/test.txt
```

### mv
Chuyển file hoặc cả thư mục từ thư mục này sang thư mục khác : 
```bash
hdfs dfs -mv /newfolder /DR
```

### cp
Giống mv nhưng là copy :
```bash
hdfs dfs -cp /DR/newfolder/test.txt /DR
```

### rm 
Xóa 1 file trong hdfs : 
```bash
hdfs dfs -rm /DR/test.txt	
```

### getmerge
Gộp các file trên hdfs và tải về local : 
```bash
hdfs dfs -getmerge /usr/trannguyenhan /home/trannguyenhan01092000/output.dat
```
	
