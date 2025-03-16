---
title: Giới thiệu Scrapy Shell
author: trannguyenhan 
date: 2023-04-01 20:52:00 +0700
categories: [Crawler]
tags: [Crawler, Scrapy, Scrapy Shell]
math: true
mermaid: true
image:
  src: https://raw.githubusercontent.com/demanejar/image-collection/main/scrapy/scrapyshell.png
---

Mỗi lần viết 1 spider chúng ta phải viết nhiều các đoạn css selector, xpath để phân tích thông tin mà nhiều lúc không biết nó đúng hay sai. Mỗi lần như vậy thì lại phải chạy project rồi in ra thông tin mình crawl được xem có đúng hay không. Làm như vậy thì rất mất thời gian, vì thế Scrapy có cung cấp một công cụ rất hay để kiểm tra trước xem css selector hay xpath chúng ta viết đã đúng hay chưa.

## Khởi chạy shell

```bash
scrapy shell <url>
```

Với `url` là URL website chúng ta muốn kiểm tra.

Ví dụ mình muốn kiểm tra một trang tin tức của kenh14:

```bash
scrapy shell https://kenh14.vn/thi-vao-lop-10-tai-ha-noi-chinh-thuc-bat-dau-20230610083632015.chn
```

## Sử dụng shell

Khi vào shell thì có một biến response được khai báo sẵn và sử dụng nó như là các biến response trong các `Spider` mà chúng ta thường viết.

Mình check thử URL của website: 

![](https://raw.githubusercontent.com/demanejar/image-collection/main/scrapy/scrapy_shell.png)

Tương tự khi lấy title của website đó: 

```bash
>>> response.css("h1::text").extract_first()
'\r\n                                    Thi vào lớp 10 tại Hà Nội chính thức bắt đầu'
>>> 
```

Ngay trong shell này, nếu muốn chuyển sang một URL khác mà không cần phải thoát shell chúng ta hãy sử dụng lệnh `fetch`:

```bash
>>> fetch("http://www.google.com", True)
2023-06-10 10:55:17 [filelock] DEBUG: Attempting to acquire lock 140238303771136 on /home/trannguyenhan/.cache/python-tldextract/3.8.10.final__usr__7d8fdf__tldextract-3.4.0/publicsuffix.org-tlds/de84b5ca2167d4c83e38fb162f2e8738.tldextract.json.lock
2023-06-10 10:55:17 [filelock] DEBUG: Lock 140238303771136 acquired on /home/trannguyenhan/.cache/python-tldextract/3.8.10.final__usr__7d8fdf__tldextract-3.4.0/publicsuffix.org-tlds/de84b5ca2167d4c83e38fb162f2e8738.tldextract.json.lock
2023-06-10 10:55:17 [filelock] DEBUG: Attempting to release lock 140238303771136 on /home/trannguyenhan/.cache/python-tldextract/3.8.10.final__usr__7d8fdf__tldextract-3.4.0/publicsuffix.org-tlds/de84b5ca2167d4c83e38fb162f2e8738.tldextract.json.lock
2023-06-10 10:55:17 [filelock] DEBUG: Lock 140238303771136 released on /home/trannguyenhan/.cache/python-tldextract/3.8.10.final__usr__7d8fdf__tldextract-3.4.0/publicsuffix.org-tlds/de84b5ca2167d4c83e38fb162f2e8738.tldextract.json.lock
2023-06-10 10:55:17 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.google.com> (referer: None)
>>> 
```

Scrapy shell cũng dùng được trong các file local bằng cách thay vị trí URL bằng đường dẫn tới file tương ứng: 

```bash
scrapy shell ./path/to/file.html
```

Scrapy Shell là một công cụ debug hữu hiệu, còn một số tính năng nữa mà Scrapy Shell cung cấp, nhưng trong bài viết này hay là series này mình chỉ xin phép nói tới những thứ thực sự hữu ích mà mình hay dùng. Kiến thức là rất nhiều thế nên chúng ta nên chắt lọc những thứ hay sử dụng, thường xuyên sử dụng để tránh bị quá tải, những thứ còn lại biết cách search hoặc chỗ để xem lại là được.

Xem thêm về Scrapy shell tại: [https://docs.scrapy.org/en/latest/topics/shell.html](https://docs.scrapy.org/en/latest/topics/shell.html).