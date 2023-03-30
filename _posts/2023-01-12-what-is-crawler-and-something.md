---
title: Crawler, một số điều mình chia sẻ về crawler và loạt bài viết về crawler sắp tới?
author: trannguyenhan 
date: 2023-01-12 20:52:00 +0700
categories: [Crawler]
tags: [Crawler, Scrapy, Selenium]
math: true
mermaid: true
image:
  src: https://raw.githubusercontent.com/demanejar/image-collection/main/WelcomeSeriesCrawler/What-is-Web-Scraping-and-How-to-Use-It.png
---

Crawler, Web Scrape, Web Scraping, thu thập dữ liệu, cào dữ liệu,... chắc là các từ ngữ mà chúng ta hay sử dụng nhất để nói về các công việc tạo ra những chương trình đi phân tích và lấy dữ liệu từ một website. Chắc chắn là có sự khác biệt giữa hai khái niệm `web crawler` và `web scraping`, tuy nhiên về cơ bản là chúng giống nhau và công việc bình thường của chúng ta khi làm thường là sự kết hợp của hai khái niệm này, vậy sau này để cho đơn giản và dễ gọi thì khi nhắc tới hai khái niệm này chúng ta có thể coi chúng là một.

Có một điều là hầu hết các chủ nhân của website đều không thích chúng ta đi cào dữ liệu web của họ. Đầu tiên là vì với những người làm nội dung chân chính thì chắc chắn họ không muốn có người khác sao chép những công sức của họ. Tiếp theo là khi dùng bot thường xuyên để crawl có thể khiến cho website quá tải. Một số website vừa và nhỏ họ thuê những máy chủ không quá lớn để đáp ứng cho một lượng nhỏ các đọc giả của họ, và nếu bot của chúng ta hoạt động quá thường xuyên trên website đó sẽ khiến cho máy chủ hết tài nguyên. Và còn nhiều điều khác nữa...

## Sự phát triển của crawl dữ liệu từ website

Với sự bùng nổ của trí tuệ nhân tạo trong những năm gần đây khiến cho nhu cầu về dữ liệu trở nên cần thiết hơn bao giờ hết. Và nguồn dữ liệu rất phong phú không đâu khác chính là internet. Ví dụ bạn muốn tạo ra một mô hình dự đoán giá nhà, thì việc đầu tiên chắc chắn là việc phải thu thập các dữ liệu về giá nhà trên các trang giao bán bất động sản, sau đó xử lý dữ liệu để làm đầu vào cho mô hình học. Hay việc nếu bạn muốn tạo ra một mô hình dịch máy, ngoài việc thuê người để làm dữ liệu đầu vào sẽ tốn rất nhiều chi phí thì có thể nghĩ tới một hướng khác như là crawl dữ liệu từ các trang báo song ngữ (trong series lần này mình sẽ có một bài viết về chủ đề crawl báo song ngữ),...

## Các cách website thường dùng để phòng tránh việc bị crawl

Series này nói về việc làm sao để crawl được dữ liệu, tuy vậy thì ít nhất chúng ta cũng cần phải biết một số cách cơ bản mà các website dùng để phòng tránh bị crawl thì mới có thể tìm được cách crawl dữ liệu từ các website đó. Nói chung, để crawl được dữ liệu thì phải biết các website phòng chống việc bị crawl như nào. 

### Sử dụng file `robots.txt` 

![](https://raw.githubusercontent.com/demanejar/image-collection/main/WelcomeSeriesCrawler/robottxt.png)

Chủ sở hữu trang web thường sử dụng các tệp “robots.txt” để truyền đạt ý định của họ về việc crawl dữ liệu từ website của họ. Hiểu đơn giản là file này định nghĩa những gì mà chúng ta được phép crawl trên website. Tuy nhiên, chắc chắn là chả mấy con bot nào tuân theo cái file này cả. 

Nếu bạn dùng qua `Scrapy` sẽ thấy từ phiên bản scrapy 1.1 (2016-05-11) dowloader sẽ tải về tập robots.txt trước khi bắt đầu crawling, để xem những url nào là những url được phép crawl. Nhưng cũng rất dễ để thay đổi điều này, vào trong setting.py và đặt lại `ROBOTSTXT_OBEY = False`, mọi thứ lại trở lại như ban đầu.

### Gen website động bằng Javascript

![](https://raw.githubusercontent.com/demanejar/image-collection/main/WelcomeSeriesCrawler/javascript.jpeg)

Bây giờ thì các website được gen bằng Javascript đã không còn là khó khăn trong việc crawl nữa. Các trình duyệt như Chrome, Firefox,... nó có thể hiển thị được nội dung trang web là vì nó có một trình dịch Javascript trên các trình duyệt, và trước khi hiển thị cho người dùng nó đã phải chạy các đoạn mã Javascript để thu về nội dung HTML sau đó hiển thị nên cho người dùng. 

Vậy tư tưởng của chúng ta khi crawl các trang web gen bằng Javascript cũng như vậy, trước khi lấy mã để phân tích chúng ta sẽ cho qua một tool chung gian để nó chạy các đoạn mã Javascript và trả về cho chúng ta mã HTML cuối cùng. Với `Scrapy` có thể sử dụng `Splash`, hay có thể sử dụng luôn `Selenium` để giả lập trình duyệt,... (Mình sẽ nói rõ hơn trong các bài viết cụ thể về từng cách).

Muốn check xem website mà bạn muốn crawl dữ liệu có gen bằng Javascript hay không có thể sử dụng extension: [Disable JavaScript
](https://chrome.google.com/webstore/detail/disable-javascript/jfpdlihdedhlmhlbgooailmfhahieoem)

### Bảo vệ qua cloudflare

![](https://raw.githubusercontent.com/demanejar/image-collection/main/WelcomeSeriesCrawler/Cloudflare_Logo.svg.png)

Có rất nhiều cách để phát hiện xem một trình duyệt hay một request tới website là của bot hay của một người dùng thật. Cloudflare sẽ phát hiện và bắt những con bot phải vượt qua capcha trước khi muốn truy cập tiếp vào nội dung của website đang được bảo vệ.

Cách để vượt cloudflare mình cũng sẽ nói ở trong một bài viết cụ thể sau trong series này.

### Ban theo IP dựa trên lưu lượng truy cập quá nhiều từ một IP

![](https://raw.githubusercontent.com/demanejar/image-collection/main/WelcomeSeriesCrawler/your-ip-has-been-banned-thumbnail.jpg)

Điều này có thể được khắc phục bằng cách sử dụng proxy, bạn tạo ra một pools (bể chứa) chứa hàng nghìn proxy tùy theo nhu cầu của bạn, mỗi lần request vào website sẽ lấy ra một proxy, như vậy những người sở hữu website sẽ không thể biết các request vào website của họ là tới từ các con bot do bạn tạo ra. 

Ngoài ra thì khi crawl cũng nên đặt cho những lần truy cập website một thời gian nghỉ, một phần để tránh quá tải cho website đang crawl và cũng để  tránh những người sở hữu website phát hiện ra các request liên tiếp trong khoảng thời gian liên tục từ một nơi.

### Sử dụng đăng nhập để bảo vệ nội dung

![](https://raw.githubusercontent.com/demanejar/image-collection/main/WelcomeSeriesCrawler/loginmessage.png)

Bắt người dùng phải đăng nhập để có thể xem nội dung trên website của họ. Dĩ nhiên là chúng chỉ áp dụng nếu website đó đang thực sự có chỗ đứng, với một website mới tạo mà bắt người dùng cứ phải đăng nhập để xem nội dung thì chưa chắc đã thu hút được người dùng mới.

Và để vượt qua được cách bảo vệ này thì chúng ta chỉ cần đăng nhập thông tin lưu lại session cho mỗi lần request sau vào website, điển hình nhất cho việc này chính là crawl Facebook và mình cũng sẽ nói cụ thể hơn ở bài viết sau.

### Sử dụng capcha

![](https://raw.githubusercontent.com/demanejar/image-collection/main/WelcomeSeriesCrawler/unnamed.jpg)

Việc sử dụng capcha có thể chặn được kha khá bot nhưng lại rất ít được sử dụng vì nó gây một trải nghiệm tồi tệ cho người dùng.