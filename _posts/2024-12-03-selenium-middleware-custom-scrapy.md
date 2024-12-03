---
title: Cài đặt Selenium Middleware cho Scrapy 
author: trannguyenhan 
date: 2024-12-03 20:52:00 +0700
categories: [Crawler]
tags: [Crawler, Selenium, Scrapy]
math: true
mermaid: true
image:
  src: https://raw.githubusercontent.com/demanejar/image-collection/refs/heads/main/Scrapy-Selenium/scrapy_selenium_middleware.png
---

Scrapy có cung cấp thư viện [scrapy-selenium](https://pypi.org/project/scrapy-selenium/) cho phép việc sử dụng Seleinum để lấy dữ liệu trang web trước khi trả về cho spdier xử lý. Tuy nhiên trong 1 số trường hợp ví dụ mình không muốn dùng selenium bình thường mà mình muốn dùng undetected-chromedriver vì nó giúp bypass Cloudflare trên website, muốn add proxy thông qua extension của chrome, muốn scroll lên xuống và một số hành động trước khi vào spider,... thì việc sử dụng thư viện sẽ là rất hạn chế. việc xử dụng thư viện cũng rất tiện lợi trong 1 số trường hợp đơn giản và mình sẽ giới thiệu trong bài viết sau. Trong bài viết này mình sẽ nói về việc sử dụng [undetected-chromedriver](https://pypi.org/project/undetected-chromedriver/) xây dựng SeleniumMiddleware bypass Cloudflare website.

## Khởi tạo project

Khởi tạo một project scrapy: 

```bash
scrapy startproject seleniummiddlewarescrapydemo
```

Tạo một spider để crawl dữ liệu, ở đây mình crawl dữ liệu từ trang `vinpearl.com/`:


```bash
scrapy genspider vinpearl_dot_com https://vinpearl.com/vi/news/ha-noi
```

Cài đặt thư viện cần thết:

```bash
pip install scrapy
pip install scrapy-selenium
```

## SeleniumMiddleware

Tạo class `SeleniumBaseMiddleware` như sau:

```python
class SeleniumBaseMiddleWare(object):
    @classmethod
    def from_crawler(cls, crawler):
        middleware = cls()
        crawler.signals.connect(middleware.spider_opened, signals.spider_opened)
        crawler.signals.connect(middleware.spider_closed, signals.spider_closed)
        return middleware

    def get_chrome_options(self):
        options = uc.ChromeOptions()
        return options

    def spider_opened(self, spider):
        try:
            options = self.get_chrome_options()

            self.driver = uc.Chrome(
                options=options,
                version_main=127
            )
        except Exception as e:
            logging.error(f"Failed to open spider and create driver: {e}")
            raise

    def process_request(self, request, spider):
        logging.info('SELENIUM FETCH URL {}'.format(request.url))
        self.driver.get(request.url)

        request.meta.update({'driver': self.driver})
        content = self.driver.page_source
        return HtmlResponse(
            request.url, encoding="utf-8", body=content, request=request
        )

    def process_response(self, request, response, spider):
        return response
    
    def spider_closed(self, spider):
        self.driver.quit()
```

- `spider_opened()` khởi tạo `driver` và các tham số cần thiết, ví dụ như là `options` hay nếu có thêm proxy chúng ta cũng sẽ thêm proxy tại đây.

- `spider_closed()` tắt `driver` sau khi kết thúc crawl.

- `process_request()` xử lý việc lấy về dữ liệu, gán `request.meta.update({'driver': self.driver})` để sử dụng driver phía trong spider nếu cần thiêt.

Trong spider `vinpearl_dot_com` nếu có những tác vụ mà cần sử dụng `driver` như scroll lên xuống, click button thì sẽ lấy ra `driver` như sau: 

```python
driver = response.request.meta["driver"]
driver.execute_script(
    "window.scrollTo(0, document.body.scrollHeight);"
)
```

Với điều kiện là `driver` đã được update vào trong request của middleware trước đó `request.meta.update({'driver': self.driver})` như trong SeleniumBaseMiddleware.

Trong file `settings` thêm SeleniumBaseMiddleware vào trong DOWNLOADER_MIDDLEWARES:

```python
DOWNLOADER_MIDDLEWARES = {
   "seleniummiddlewarescrapydemo.middlewares.SeleniumBaseMiddleWare": 543,
}
```

Bây giờ việc crawl website rất là dễ dàng khi không còn bị chặn bởi Cloudflare và đã gen toàn bộ mã Javascript của website động thành HTML để dễ dàng phân tích.

Toàn bộ project xem tại: [https://github.com/demanejar/selenium-middleware-scrapy-demo](https://github.com/demanejar/selenium-middleware-scrapy-demo)