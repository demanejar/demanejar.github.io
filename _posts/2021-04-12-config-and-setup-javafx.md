---
title: Cài đặt, cấu hình và sửa một số lỗi khi cài đặt Javafx
author: trannguyenhan
date: 2021-04-11 20:52:00 +0700
categories: [Blogging, Share]
tags: [Java FX, Java, OpenJFX, Java UI]
math: true
mermaid: true
---
*JavaFX là nền tảng để tạo và phân phối các ứng dụng dành cho máy tính để bàn cũng như các ứng dụng RIAs (Rich Internet Applications) có thể chạy trên nhiều thiết bị khác nhau.  JavaFX dự định thay thế hoàn toàn Swing làm thư viện GUI chuẩn cho Java SE. JavaFX hỗ trợ cho các máy tính để bàn và trình duyệt web trên nền tảng Windows, Linux và macOS. Bài này mình sẽ hướng dẫn các bạn cài đặt và sửa một số lỗi khi cấu hình javafx trên eclipse.* <br />
## Với Project bình thường
Đầu tiên, muốn tạo project Javafx thì bạn phải tải về bộ thư viện javafx phù hợp với nền tảng của mình tại : [https://gluonhq.com/products/javafx/](https://gluonhq.com/products/javafx/)
<br />![download openjfx](https://images.viblo.asia/02a7fd45-8cd9-4e05-8af9-73798e251e10.png)<br /><br />
Sau khi tải về, các bạn giải nén thư mục vừa rồi tại một nơi nào đó trong máy tính của bạn (bất cứ nơi nào đều được), ở đây trên máy tính của mình thì thư viện openjfx được lưu tại /media/trannguyenhan01092000/LEARN/openjfx-11.0.2_linux-x64_bin-sdk/javafx-sdk-11.0.2 (ubuntu, các bạn dùng win hay mac cứ làm tương tự nha).
<br />![unzip file openjfx](https://images.viblo.asia/2f988e29-8838-44eb-ae33-d7247b77aa92.png)<br /><br />
Và bây giờ, mở eclipse lên. <br />
Tạo một thư viện người dùng (User Library) mới trong `Window -> Preferences -> Java -> Build Path -> User Libraries -> New`
<br />![create user library](https://images.viblo.asia/d27e183c-736f-46e0-a012-bd413574db41.png)<br /><br />
Đặt tên cho thư viện javafx của bạn rồi OK, sau đó thêm các file JAR vào bộ thư viện này. Nhấn vào `Add External JARs`, chọn tới thư mục mà bạn giải nén file openjfx bạn tải ở về bên trên, trong phần lib chọn tất cả các file có đuôi .jar, xong thì `Apply and Close`
<br />![choose file jar](https://images.viblo.asia/a794ad5a-f3f3-4386-8b49-24a727347355.png)<br /><br />
Bây giờ tạo một Project Java, Chọn `File -> New -> Java Project` ở đây mình tạo một Project và đặt tên cho nó là HelloJavafxDemo.
<br />![create project](https://images.viblo.asia/123b516a-4703-4fea-8f97-4e1c50fa3cd8.png)<br /><br />
Cuối cùng, bạn chỉ cần thêm thư viện mình vừa tạo vào classpath là được, Nhấn chuột phải vào Project của bạn Chọn `Build Path -> Configure Build Path`, tại phần classpath nhấn vào `Add Library -> User Library -> chọn thư viện javafx bạn vừa tạo` rồi nhấn `Finish`, rồi nhấn lần lượt `Apply`, `Apply and Close`.
<br />![add library1](https://images.viblo.asia/499db763-9dbc-4971-9a5e-36bfb51e529f.png)
<br />![add library2](https://images.viblo.asia/8a7aa3cc-87c2-4fd1-8c24-f362c784217d.png)
<br />![add library3](https://images.viblo.asia/901c03b3-3c2e-45eb-a151-c58922f4671c.png)
<br/>![add library4](https://images.viblo.asia/0231930b-31d0-4e09-af6f-a445ccfb90a6.png)<br /><br />
Cuối cùng bạn có thể thấy bên cửa sổ explorer đã có thêm thư viện Javafx mà bạn thêm vào, vậy là đã xong.<br /><br />
**Fix lỗi** `Error: JavaFX runtime components are missing, and are required to run this application`, trên trang chủ của openjfx có nói tới lỗi này và một số bạn có thể sẽ mắc lỗi này trong khi chạy chương trình.
<br />![error](https://images.viblo.asia/bb26fdb2-119e-46e6-a530-2f927db48ea8.png)
<br />![error](https://images.viblo.asia/9982831f-e913-4b45-835b-483051cd10ec.png)<br />
Để sửa, các bạn chọn `Run -> Run  Configurations`, trong phần Arguments nhập `--module-path <path>/lib/  --add-modules=ALL-MODULE-PATH`, trong đó path là đường dẫn tới thư mục mà bạn giải nén ra file openjfx đã tải về, ví dụ của mình sẽ là `--module-path /media/trannguyenhan01092000/LEARN/openjfx-11.0.2_linux-x64_bin-sdk/javafx-sdk-11.0.2/lib/  --add-modules=ALL-MODULE-PATH`, vậy là đã xong, Apply và Run để xem kết quả.
<br />![result](https://images.viblo.asia/23dc80ae-bdea-49b0-a0ca-c60c05cdb6b0.png)<br /><br />

## Với Maven Project
Với maven project thì dễ dàng hơn, các bạn tạo một project java bình thường. Sau khi tạo xong nhấn chuột phải vào project vừa tạo chọn `Configure -> Convert to Maven Project,` các bạn cứ để mặc định cũng được rồi ấn `Finish`. Trong file `pom.xml` thêm phụ thuộc javafx như sau : 
```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>Week3SoftwareEngineering</groupId>
  <artifactId>Week3SoftwareEngineering</artifactId>
  <version>V1</version>
  <dependencies>
  	<dependency>
    <groupId>org.openjfx</groupId>
    <artifactId>javafx-controls</artifactId>
    <version>14</version>
  	</dependency>
  </dependencies>
  <build>
    <sourceDirectory>src</sourceDirectory>
    <plugins>
      <plugin>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.8.1</version>
        <configuration>
          <release>11</release>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
``` 
Nhấn chuột phải vào Project của bạn chọn `Refresh`, nếu như trong phần `Explorer` hiện ra thêm thư viện `Maven Dependencies` thì là OK đã xong. Còn nếu bạn nào Refresh mà không thấy thì tốt nhất tắt Eclipse đi rồi vào lại, lần này chắc chắn là thấy =)). Nếu các bạn cũng mắc lỗi `Error: JavaFX runtime components are missing, and are required to run this application` thì cũng fix y hệt như bên trên nha.<br /><br />
Tham khảo : [https://openjfx.io/openjfx-docs/](https://openjfx.io/openjfx-docs/)
