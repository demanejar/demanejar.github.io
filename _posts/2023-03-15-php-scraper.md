---
title: PHP Scraper
author: trannguyenhan 
date: 2023-03-15 20:52:00 +0700
categories: [Crawler]
tags: [Crawler, PHP, PHP crawler]
math: true
mermaid: true
image:
  src: https://raw.githubusercontent.com/demanejar/image-collection/main/scrapy/scraping-with-php-image-1.png
---

Nói về crawl chắc hẳn mọi thứ đều đổ dồn về Python và các framework xây dựng trên Python như Scrapy, Beautiful Soup hay Selenium sử dụng Python,... Trong bài viết này ngoại truyện một tí, chúng ta sẽ nói về một ngôn ngữ mà không được mạnh lắm về mảng này, PHP.

Sẽ có rất nhiều lúc mà Python sẽ không giải quyết hết được các vấn đề về crawl của bạn mà bạn phải cần tới ngôn ngữ đang sử dụng cho website của bạn để làm điều đó. PHP đang là ngôn ngữ phổ biến nhất cho website và vì thế nên chắc chắn sẽ có lúc bạn phải dùng tới PHP để crawl.

## Cài đặt thư viện

Thư viện sử dụng được mình đề cập tới trong bài viết này là `guzzle_requests`. Tạo file `composer.json` và thêm vào thư viện `guzzle_requests`: 

```json
{
    "require": {
        "guzzlehttp/guzzle": "^7.4"
    }
}
```

Chạy `composer install` để tải xuống thư viện cho project.

## Phân tích website

Tạo file `guzzle_requests.php` để viết mã phân tích website.

Trong bài viết này mình ví dụ phân tích website [https://meridiano.net/beisbol/beisbol-venezolano/249167/lvbp--tigres-de-aragua-aseveran-que-miguel-cabrera-tiene--mucha-posibilidad--de-jugar.html](https://meridiano.net/beisbol/beisbol-venezolano/249167/lvbp--tigres-de-aragua-aseveran-que-miguel-cabrera-tiene--mucha-posibilidad--de-jugar.html), lấy tiêu đề, mô tả, ảnh mô tả bài viết, nội dung và ghi vào một file kết quả đầu ra.

Import thư viện và khai báo một httpClient: 

```php
require 'vendor/autoload.php';
$httpClient = new \GuzzleHttp\Client();
```

Get HTML của trang web về để phân tích: 

```php
$response = $httpClient->get('https://meridiano.net/beisbol/beisbol-venezolano/249167/lvbp--tigres-de-aragua-aseveran-que-miguel-cabrera-tiene--mucha-posibilidad--de-jugar.html');
$htmlString = (string) $response->getBody();
```

Bây giờ để lấy từng thông tin chúng ta sẽ sử dụng xpath để lấy từng thành phần: 

```php
$title = $xpath->evaluate('//meta[contains(@property,"og:title")]/@content');

$description = $xpath->evaluate('//meta[contains(@property,"og:description")]/@content');

$image = $xpath->evaluate('//div[contains(@class,"News Detail")]//img/@src');

$text = $xpath->evaluate('//*[@id="med"]');
```

Sau khi lấy xong thì ghi chúng ra file. Gộp lại ta có một project như sau: 

```php
<?php

require 'vendor/autoload.php';
$httpClient = new \GuzzleHttp\Client();

$response = $httpClient->get('https://meridiano.net/beisbol/beisbol-venezolano/249167/lvbp--tigres-de-aragua-aseveran-que-miguel-cabrera-tiene--mucha-posibilidad--de-jugar.html');
$htmlString = (string) $response->getBody();

// HTML is often wonky, this suppresses a lot of warnings
libxml_use_internal_errors(true);

$doc = new DOMDocument();
$doc->loadHTML($htmlString);
$xpath = new DOMXPath($doc);

$file = fopen("test.txt", "w");

$title = $xpath->evaluate('//meta[contains(@property,"og:title")]/@content');
if(count($title) > 0){
    echo "Title: " . $title[0]->textContent.PHP_EOL;
    fwrite($file, "Title: " . $title[0]->textContent.PHP_EOL);
}

$description = $xpath->evaluate('//meta[contains(@property,"og:description")]/@content');
if(count($title) > 0){
    echo "Description: " . $description[0]->textContent.PHP_EOL;
    fwrite($file, "Description: " . $description[0]->textContent.PHP_EOL);
}

$image = $xpath->evaluate('//div[contains(@class,"News Detail")]//img/@src');
if(count($image) > 0){
    echo "Image link: " . $image[0]->textContent.PHP_EOL;
    fwrite($file, "Image link: " . $image[0]->textContent.PHP_EOL);
}

$text = $xpath->evaluate('//*[@id="med"]');
if(count($image) > 0){
    echo "Text content: " . $text[0]->textContent.PHP_EOL;
    fwrite($file, "Text content: " . $text[0]->textContent.PHP_EOL);
}
fclose($file);
```

Toàn bộ project bạn có thể xem thêm tại [https://github.com/trannguyenhan/php-scraper](https://github.com/trannguyenhan/php-scraper)