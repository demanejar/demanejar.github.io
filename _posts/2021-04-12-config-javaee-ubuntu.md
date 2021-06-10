---
title: Cài đặt và cấu hình Java EE cho Eclipse
author: trannguyenhan
date: 2021-04-11 20:52:00 +0700
categories: [Blogging, Share]
tags: [Jave EE, Java, Java web, Eclipse]
math: true
mermaid: true
image:
---
*Java EE giúp tạo các trang web. Java EE là một API vì thế để dùng Java EE bạn cần một máy chủ tương thích với Java EE như là Glassfish được phát triển bởi Oracle hay là Tomcat,… Bạn chỉ việc thêm nó vào thư viện và sử dụng như bình thường.* <br /><br />
Bài viết này mình sẽ hướng dẫn mọi người cài đặt Java EE trên Eclipse với máy chủ Tomcat. <br /><br />
-  **Đầu tiên**, nếu máy bạn chưa có JDK thì tải ngay JDK bản mới nhất tại : [https://www.oracle.com/java/technologies/javase-downloads.html ](https://www.oracle.com/java/technologies/javase-downloads.html ) ( Tải về bản phân phối mới nhất của Java, hoặc bản tương thích tương ứng với thiết bị của bạn). <br /><br />
-  **Tải về** Eclipse for JavaEE tại : [https://www.eclipse.org/downloads/packages/release/2020-06/r/eclipse-ide-enterprise-java-developers ](https://www.eclipse.org/downloads/packages/release/2020-06/r/eclipse-ide-enterprise-java-developers ) ( Là bản thứ 2 trong Eclipse packages). <br /><br />
- **Tải về** một server tương thích với Java EE và thêm nó vào Eclipse, ở đây mình chọn Tomcat, tải Tomcat tại : [https://tomcat.apache.org/download-90.cgi ](https://tomcat.apache.org/download-90.cgi ) ( Hãy tải bản mới nhất tại thời điểm bạn tải để có những trải nghiệm tốt nhất). <br /><br />


Sau đó bạn mở Eclipse lên, chọn `Window --> Preferences --> Server --> Runtime Environment --> Add --> Chọn loại server sau đó thêm đường dẫn và kết thúc` ( Ở đây mình chọn là ***Tomcat 9.0*** và mình để Tomcat tại ổ D, lưu ý là server mà bạn tải về chỉ là bản zip, không cần cài đặt, chỉ cần giải nén và đặt tại một thư mục bất kì và chọn đường dẫn của server tới nơi mà bạn để thư mục là được). <br /><br />
    ![](https://images.viblo.asia/d3088636-8d97-4b82-8560-26ad6b8d5676.png) <br /><br />
    ![](https://images.viblo.asia/3df0ab08-926d-4e28-94c5-00244f61512b.png) <br /><br />
Tạo 1 project **Dynamic web Project** trong Eclipse <br /><br />
    ![](https://images.viblo.asia/c9c42e2b-cb21-4bfc-bde8-cd066b3f0709.png) <br />
- Hiện tại thì Dynamic Web Project ít được dùng, người ta ưa chuộng **Maven Project** hơn vì sự vượt trội của nó, để chuyển sang Maven Project từ project Dynamic Web Project trong Eclipse, nhấn chuột phải vào project vừa tạo chọn `Configure --> Convert to Maven Project`. <br /><br />
- Sau đó thêm server runtime vào Libraries để có thể dùng các thư viện của ***Java EE*** ( Bạn có thể chọn bất kì Server nào tương thích với Java EE, ở đây mình chọn Tomcat 9.0).<br /><br />
- Chuột phải vào `Project` chọn `Properties ( Alt + Enter) --> Java Build Path --> Add Library --> Server Runtime`.<br /><br />
    ![](https://images.viblo.asia/fcf7d69e-da08-4414-ba3b-5f26559719df.png) <br />
- Next và chọn `Server --> Finish` và lưu lại. OKE! Đã xong. Giờ bạn đã có thể tạo các ứng dụng web dựa trên Java EE bằng Eclispe.<br /><br />
