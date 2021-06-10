---
title: Cài đặt SQL Server trên Ubuntu
author: trannguyenhan
date: 2021-04-11 20:52:00 +0700
categories: [Blogging, Share]
tags: [ms sql, sql server, ubuntu]
math: true
mermaid: true
image:
  src: https://i.pinimg.com/564x/47/5c/70/475c7097cc8e5a339d12412ad2434c95.jpg
---

*Một cơ sở dữ liệu cần phải có 1 server và 1 công cụ giúp kết nối và truy vấn dữ liệu từ server, ngày xưa người ta thường dùng các cửa sổ dòng lệnh (command line), hiện tại đã có các công cụ khác trực quan hơn để giúp việc truy vấn dữ liệu một cách dễ dàng hơn thông qua các cửa sổ UI. Microsoft SQL Server là cơ sở dữ liệu rất nổi tiếng của Microsoft. Nó được sử dụng rộng rãi trong trường học, doanh nghiệp vì dễ cài đặt và dễ sử dụng. Việc cài đặt Microsoft SQL Server trên Windows là khá dễ dàng . Bạn chỉ cần tải về cài đặt SQL Server Management Studio (SSMS) là sẽ được tự động cài đặt những thứ cần thiết . Tuy nhiên, trên Ubuntu muốn sử dụng SQL Server lại phức tạp hơn một tí.*

SQL Server Management Studio (SSMS) là công cụ dành riêng cho hệ điều hành Windows. Nếu bạn sử dụng MacOS hay Ubuntu muốn hay bắt buộc sử dụng SQL Server vì lý do học tập hay làm việc mà không muốn sử dụng cửa sổ dòng lệnh để chạy những câu lệnh SQL thì phải sử dụng một trong hai công cụ sau đây : Azure Data Studio hoặc cài thêm phần mở rộng mssql extension trên VS Code. Hôm nay, mình sẽ hướng dẫn các bạn cài đặt và sử dụng Azure Data Studio trên Ubuntu. <br />
Đầu tiên, các bạn tải về file .deb của Azure Data Studio tại : [https://docs.microsoft.com/en-us/sql/azure-data-studio/download-azure-data-studio?view=sql-server-linux-2017](https://docs.microsoft.com/en-us/sql/azure-data-studio/download-azure-data-studio?view=sql-server-linux-2017) <br />
![Tải về azure data studio](https://images.viblo.asia/a6569083-b49d-42d3-b6c7-b3ff2efe429d.png) <br />
Các bạn nhấn vào download .deb trong phần platform dành cho Linux như hình vẽ trên, trình duyệt sẽ tự động tải về một file cài đặt. <br />
Mở termial lên rồi dùng lệnh cd để đi tới thư mục chứa file cài đặt được tải về (hoặc bạn có thể đi tới thư mục chứa file cài đặt trước sau đó nhấn chuột phải chọn open terminal). <br />
Cài đặt phần mềm bằng lệnh sau : 
```
sudo dpkg -i azuredatastudio-linux-<version string>.deb
```
(thay `<version string>` bằng phiên bảng tương ứng mà bạn tải về hoăc bạn chỉ cần nhấn `sudo dpkg -i azu` rồi nhấn tab, máy tính sẽ tự động gợi ý file cài đặt cho bạn).  <br />
Chờ sau khi cài đặt xong các bạn sử dụng lệnh `azuredatastudio` để mở Azure Data Studio. Tuy nhiên, như vậy vẫn chưa xong. Bạn mới chỉ cài xong phần mềm trực quan hóa giúp bạn xử lý các truy vấn tốt hơn bằng công cụ UI mà không phải sử dụng các cửa sổ dòng lệnh ( command line). Bây giờ các bạn phải cài thêm SQL Server sau đó kết nối Azure Data Studio với SQL Server. <br /><br />

Để tải về SQL Server các bạn làm theo mình như sau nha, mở terminal lên sử dụng các lệnh sau : <br />
```
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
```
```
sudo add-apt-repository "$(wget -qO- https://packages.microsoft.com/config/ubuntu/18.04/mssql-server-2017.list)"
```
```
sudo apt-get update
sudo apt-get install -y mssql-server
```
Sau khi cài xong, các bạn cấu hình lại SQL Server bằng câu lệnh sau (mọi thứ rất đơn giản bạn chứ chọn Y là được, tuy nhiên nhớ để ý đến phần mật khẩu, mật khẩu phải có chữ in hoa, chữ cái thường, chữ số và ký tự đặc biệt. Và quan trọng hơn nữa là các bạn phải ghi nhớ mật khẩu này để có thể kết nối với server)
```
sudo /opt/mssql/bin/mssql-conf setup
```

Sau khi cài xong sử dụng lệnh sau để kiểm tra, nếu nó hiện như hình bên dưới này tức là bạn đã cài đặt thành công SQL Server rồi đó : 
```
systemctl status mssql-server.service
```
![SQL Server](https://images.viblo.asia/7814fd7e-6720-4475-9079-a3e107a1d4be.png) <br /><br/>
Bây giờ là kết nối SQL Server với Azure Data Studio. Bạn mở Azure Data Studio lên, ấn vào **new connection** trên cửa sổ Welcome hoặc ấn vào **add connection** màu xanh xanh bên trái. Một cửa sổ connection hiện ra để bạn nhập thông tin. <br />

Bạn để thông tin server là localhost, User name mặc định là sa, còn password chính là password mà bạn vừa đặt khi cấu hình. Mọi thứ còn lại bạn để mặc định, sau đó ấn Connect. <br />
![connection](https://images.viblo.asia/d7aad68e-9ef2-4a4f-b135-827e81d9951f.png) <br />

OK! bây giờ bạn sử dụng nó bình thường như các công cụ khác. Bạn có thể tạo cơ sở dữ liệu, bảng hay cột bằng công cụ UI hoặc bằng các câu lệnh truy vấn đều giống như SQL Server Management Studio. Cửa sổ sau khi kết nối thành công sẽ như sau : <br />
![ok](https://images.viblo.asia/5ee89bcd-e201-4d39-9529-65f083172d10.png)

<br />Tham khảo : [https://docs.microsoft.com/](https://docs.microsoft.com/)
