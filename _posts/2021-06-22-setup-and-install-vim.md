---
title: Cấu hình và cài đặt Vim từ A-Z 
author: trannguyenhan
date: 2021-06-22 20:52:00 +0700
categories: [Blogging, Share]
tags: [Ubuntu, Vim, NeoVim, IDE, Text Editor]
math: true
mermaid: true
---

***Vim là một bản sao, có bổ sung, của chương trình soạn thảo văn bản vi của Bill Joy cho Unix. Tác giả của Vim, Bram Moolenaar, đã dựa trên mã nguồn của một cổng trình soạn thảo Stevie sang Amiga và phát hành một phiên bản cho công chúng vào năm 1991, NeoVim là một siêu mở rộng của Vim. Tại sao chúng ta lại chọn Vim? Vim có cấu hình cao và đi kèm với các tính năng đáng chú ý như tô sáng cú pháp, hỗ trợ chuột, phiên bản đồ họa, chế độ trực quan, nhiều lệnh chỉnh sửa mới và một lượng lớn tiện ích mở rộng cùng nhiều hơn nữa. Bài viết này mình sẽ hướng dẫn các bạn cài đặt Vim từ a-z để có một trình soạn thảo mạnh mẽ sử dụng cho tất cả các ngôn ngữ***

## Tải Vim và 1 số package điều kiện  
Trước hết để dùng Vim thì bạn phải có Vim, mình sẽ sử dụng NeoVim vì NeoVim là một phiên bản cải tiến của Vim và nó ưu việt hơn Vim.
Để tải NeoVim các bạn cài đặt qua câu lệnh sau: 
```
sudo apt-get install neovim
```
Một số điều kiện để có thể sử dụng NeoVim là máy bạn phải có python2, 3 ( python 2,3 thì đã được cài đặt sẵn trên Ubuntu chúng ta chỉ cần cài thêm thư viện neovim vào là được), Ruby và Nodejs.

**Cài Ruby**: 
```
sudo apt-get install ruby-full
```

**Cài Nodejs**:
```
sudo apt-get install nodejs
sudo apt-get install npm
```
**Cài package neovim trên python3**: 
```
sudo apt-get install python3-pip ( Cài pip nếu máy bạn chưa cài)
pip install neovim
pip install pynvim
```
**Cài package neovim trên Nodejs**: 
```
sudo npm install -g neovim
```
**Cài package neovim trên Ruby**: 
```
gem install neovim 
```
**Cài package neovim trên python2**:
Đối với python2 thì khó khăn hơn chút, nếu như bạn nào sử dụng Ubuntu 20.10 thì đã không còn hỗ trợ pip2, để tải về pip2 thì các bạn làm theo mình như dưới đây: 

Sử dụng curl để tải về file get-pip.py
```
curl https://bootstrap.pypa.io/pip/2.7/get-pip.py --output get-pip.py
```
Sau đó sử dụng python2 để chạy file vừa download về: 
```
sudo python2 get-pip.py
```
Kiểm tra phiên bản của pip2 xem bạn đã cài đặt thành công chưa: 
```
pip2 --version
```
Cài package neovim cho python2
```
pip2 install neovim
pip2 install pynvim
```

## Cài đặt và cấu hình Vim 
Mở terminal và gõ :
```
nvim
```
để truy cập vào vim. Vim sẽ có nhiều chế độ khác nhau, cái này mình sẽ không nói ở đây. Kiểm tra xem tình trạng cài đặt bạn gõ : 
```
:checkhealth
```
Nếu bạn đã cài đặt đầy đủ các gói bên trên thì tất cả sẽ là `OK` hêt, và nhớ là phải `OK` hết thì mới làm bước tiếp theo nha: 

![](https://i.pinimg.com/564x/bd/20/ec/bd20ec8a3916b02e7bd88b0cb09593d0.jpg)

Sau khi checkhealth hiển thị mọi thứ `OK` hết thì sẽ tới bước tiếp theo, mọi người truy cập vào https://vim-bootstrap.com/ để tạo file cấu hình khởi tạo cho Vim, tải về sau đó đổi tên thành `init.vim`.

Tạo một thư mục `nvim` tại `~/.config` và copy file `init.vim` vào thư mục này, khi khởi động vim thì nó sẽ đọc thư mục này là khởi tạo các gói cần thiết.
Truy cập vào vim một lân nữa bằng lệnh: 
```
nvim
```
Lúc này vim sẽ tự động tải về các gói mà bạn chọn ở trên web https://vim-bootstrap.com/ và tải về, các package mở rộng tải về sẽ đặt tại cùng đường dẫn với file `init.vim`
Một số plugin mặc định đã được cài đặt với file `init.vim` được tải từ `vim-boootstrap` như là `nerdtree`, plugin này giúp tạo một cây thư mục như một số IDE hay Text Editor để. Để kiểm tra mọi người có thể tạo 1 file bất kì, ví dụ mình tạo 1 file `demo_python.py` và mở file này bằng vim như sau: 
```
vim demo_python.py
hoặc 
nvim demo_python.py
```
Ấn `F3` để mở cây thư mục hoặc `F2` để mở cây thư mục tại vị trí file đang mở: 

![](https://i.pinimg.com/564x/f5/6d/a3/f56da3c3132f38c5887a186b51354b7e.jpg)

Các bạn có thể tham khảo tại https://vimawesome.com/ để tải thêm nhiều plugin nữa phù hợp nha.

Phần này giờ cũng hơi dài rồi, phần sau mình sẽ giới thiệu thêm tới mọi người một số extension hay dùng và hữu ích nha.

Tham khảo [https://stackoverflow.com/](https://stackoverflow.com/questions/61981156/unable-to-locate-package-python-pip-ubuntu-20-04)
