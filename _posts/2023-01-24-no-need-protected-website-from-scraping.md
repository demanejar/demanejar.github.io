---
title: Không cần thiết phải bảo vệ website của bạn khỏi bị cào 100%?
author: trannguyenhan 
date: 2023-01-24 20:52:00 +0700
categories: [Crawler]
tags: [Crawler, Scrapy, Selenium, Protected Website]
math: true
mermaid: true
image:
  src: https://raw.githubusercontent.com/demanejar/image-collection/main/WelcomeSeriesCrawler/What-is-Web-Scraping-and-How-to-Use-It.png
---

Các website hiện nay thì đã không còn dễ lấy dữ liệu như ngày trước nữa vì cấu trúc của các website bây giờ cũng khác xưa rất là nhiều, nó không có các phần được định nghĩa rõ ràng để  phân tích nhanh chóng, và một phần lớn khác là vì một số website vừa và lớn cũng đã áp dụng một số các biện pháp ngăn chặn việc crawl dữ liệu từ website của họ. 

Tuy nhiên chắc chắn rồi, không cách gì có thể  tránh được hoàn toàn cả, một phần lý do cũng là do những người làm website cũng không hoàn toàn muốn bảo vệ website của họ khỏi bị crawl 100%, một số lý do được nêu ra như sau:

- Không có cách nào có thể ngăn chặn hoàn toàn việc quét web nên nhiều người hiểu được cũng không quá cố gắng cho việc ngăn chặn: nói chung là trừ việc tắt server chứ web của bạn vẫn còn công khai trên internet mà nhiều người có thể truy cập được thì sẽ không thể ngăn chặn được việc bị quét.

- Ảnh hưởng tới trải nghiệm người dùng: website có thể dùng cách rất phổ biến để ngăn chặn bot đó là dùng capcha, mỗi lần vào web là sẽ bắt phải người dùng phải vượt capcha trước. Điều này chắc chắn là giảm được lượng lớn bot vào web tuy nhiên cũng ảnh hưởng tới người dùng rất nhiều. Bạn thử nghĩ nếu website nào bắt bạn phải nhập capcha quá nhiều thì bạn cảm giác ra sao, với mình thì mình tắt nó đi ngay lập tức.

- Việc bảo vệ tốn nhiều chi phí: việc sử dụng thêm các bên thứ 3 như Cloudflare để bảo vệ website của mình khỏi các con bot cũng sẽ mất một khoản phí tương đối mà không phải ai cũng sẵn sàng chi trả. Còn nếu không sử dụng các dịch vụ của bên thứ 3 thì bạn lại phải bỏ một sô tiền tương đối lớn ra để xây dựng các chức năng, rồi sau đó là kiểm thử chúng.

- Ảnh hưởng tới việc lên TOP tìm kiếm: như mình đã nói là hầu hết chủ nhân trang web đều không thích chúng ta đi cào dữ liệu của họ, nhưng có một con bot mà họ cực kì thích mà còn phải dọn đường để nó vào nữa là các con bot của Search Engine Google (và các Search Engine khác). Vì thế nếu dùng quá nhiều kỹ thuật ngăn chặn khiến cho Google không thể lấy được dữ liệu website của bạn thì chắc chắn website không thể lên TOP.

- Vấn đề pháp lý: mình cũng không muốn nói quá nhiều về vấn đề này, cũng chỉ muốn nói một ý chung chung là giống như nhiều hoạt động khác trên Internet, không có câu trả lời đơn giản nào về các khía cạnh pháp lý của việc crawling data từ website.


Tham khảo: [https://finddatalab.com/](https://finddatalab.com/protection)