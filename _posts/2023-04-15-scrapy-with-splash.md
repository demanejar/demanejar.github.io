---
title: Crawl báo song ngữ với Scrapy và Splash
author: trannguyenhan 
date: 2023-04-15 20:52:00 +0700
categories: [Crawler]
tags: [Crawler, Scrapy, Splash]
math: true
mermaid: true
image:
  src: https://raw.githubusercontent.com/demanejar/image-collection/main/ScrapySplash/scrapy_splash_795857e959.png
---

Tìm hiểu thêm về khái niệm, ưu nhược điểm của Client Side và Server Side Rendering qua bài viết [Client-Side và Server-Side Rendering](https://hacerweb.github.io/server-side-rendering-and-client-side-rendering/).

Chúng ta sẽ thấy có một số website client side rendering, các đoạn mã HTML của nó sẽ được gen ra ở phía trình duyệt người dùng, vì thế khi crawl mặc dù F12 thấy đầy đủ các phần tử của website nhưng đoạn mã tải về lại toàn mã Javascript và không tìm thấy các phần tử cần tìm ở đâu.

Trình duyệt là được như vậy là do nó có một trình dịch mã Javascript, khi tải về đoạn mã của website, trình dịch sẽ dịch đoạn mã Javascript trước để đoạn mã này sinh và tải về các đoạn HTML cần thiết và render lại cho người dùng. Đối với những website được render phía người dùng như này có thể sử dụng 2 cách sau để thu thập dữ liệu từ website: 

- 1. Sử dụng API. Các website render động thường sử dụng API để lấy dữ liệu. Ấn F12 và theo dõi tab network để tìm ra API mà website lấy dữ liệu
- 2. Chúng ta sẽ làm giống với những gì mà trình duyệt đang làm, sẽ có một trình biên dịch mã Javascript để render ra mã HTML trước khi phân tích. 2 công cụ phổ biến nhất đó là Selenium và Splash. Với Scrapy nó thường được sử dụng kèm với Splash, để so sánh sự khác biệt giữa Scrapy và Selenium có thể xem tại [https://webscraping.fyi/lib/compare/python-selenium-vs-python-splash/](https://webscraping.fyi/lib/compare/python-selenium-vs-python-splash/)

Trong bài viết này chúng ta sẽ đi thu thập dữ liệu từ một số website song ngữ và được gen động bởi Javascript. Website minh họa là [https://toomva.com](https://toomva.com). Toàn bộ project các bạn có thể xem tại [https://github.com/trannguyenhan/bilingualcrawl-vietnamese-english](https://github.com/trannguyenhan/bilingualcrawl-vietnamese-english).

## Cài đặt thư viện scrapy-splash

```bash
pip install scrapy scrapy-splash
```

## Cài đặt và chạy splash

```bash
sudo docker pull scrapinghub/splash
```

```bash
sudo docker run -p 8050:8050 scrapinghub/splash
```

## Thêm Splash middleware trong settings.py

```py
SPIDER_MIDDLEWARES = {
#    'bilingualcrawl.middlewares.BilingualcrawlSpiderMiddleware': 543,
    'scrapy_splash.SplashDeduplicateArgsMiddleware': 100,
}

# Enable or disable downloader middlewares
# See https://docs.scrapy.org/en/latest/topics/downloader-middleware.html
DOWNLOADER_MIDDLEWARES = {
#    'bilingualcrawl.middlewares.BilingualcrawlDownloaderMiddleware': 543,
    'scrapy_splash.SplashCookiesMiddleware': 723,
    'scrapy_splash.SplashMiddleware': 725,
}
```

## Thêm cấu hình bắt buộc của Splash

```py
SPLASH_URL="http://localhost:8050"
DUPEFILTER_CLASS = 'scrapy_splash.SplashAwareDupeFilter'
HTTPCACHE_STORAGE = 'scrapy_splash.SplashAwareFSCacheStorage'
SPLASH_COOKIES_DEBUG = False
```

## Sử dụng Scrapy Splash trong Spider

Trong các spider thay vì sử dụng `scrapy.Request` chúng ta sẽ sử dụng `SplashRequest` để chúng nó thể được sử dụng Splash render ra mã HTML trước khi phân tích:

```py
def parse_website(self, response):
  lst = response.css('.grid-search-video').css('a').xpath('@href').extract()
  for itm in lst: 
      yield SplashRequest(url=itm, callback=self.parse)
```

## Test Splash thông qua postman

```bash
curl --location --request GET 'http://localhost:8050/render.html?url=https://demanejar.github.io/posts/add-proxy-to-scrapy-project/'
```

Đọc chi tiết hơn về Splash tại [https://splash.readthedocs.io/](https://splash.readthedocs.io/).