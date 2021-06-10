---
title: Server side redering và client side redering
author: trannguyenhan
date: 2021-04-11 20:52:00 +0700
categories: [Blogging, Share]
tags: [web developer, server side, client side]
math: true
mermaid: true
image:
  src: https://i.pinimg.com/originals/24/1f/a3/241fa373697135fc0c589ab4c3a7b5b1.jpg
---

# Server side redering và client side redering là gì?
- Server side redering là một cơ chế mà đã xuất hiện từ rất lâu, có lẽ là từ khi web ra đời mà ở đây phần lớn những xử lý logic từ nhỏ cho tới lớn đều được đảm nhận và thực hiện bởi server khi người dùng gửi yêu cầu tới website, sau đó server sẽ gửi lại cho máy khách những mã HTML (hay cả là những mã js và css) để hiển thị.
- Client side redering là một công nghệ mới hơn, nó sinh ra nhằm giảm bớt gánh nặng ở phía server. Với ý tưởng rằng một số logic đơn giản hay là chuyển trang thì sẽ do máy khách xử lý còn những logic phức tạp hơn thì vẫn do server đảm nhiệm.


Client side redering sinh ra đã làm tách biệt khá nhiều giữa back-end và front-end.

# Ưu và nhược điểm của Server side redering và Client side redering
## Server side redering
#### Về việc phát triển 
- Server side redering đã được phát triển từ rất lâu vì thế có rất nhiều công cụ, framework hỗ trợ cơ chế này. 
- Lập trình viên dễ hiểu, dễ viết mã nguồn hơn vì đa số phần logic được xử lý tại một nơi là server. Ngoài ra việc không có sự tách biệt quá lớn giữa back-end và front-end nên lập trình viên sẽ chỉ cần phát triển 1 project, không cần phải tách ra back-end là front-end.

#### Về tốc độ tải trang 
- Ban đầu tải trang rất nhanh vì mọi thứ đã được xử lý tại server, phía máy khách chỉ việc hiển thị.
- Tuy nhiên, với các lần tải sau thì một vòng lặp đầy đủ như lần đầu được lặp lại, nên tốc độ không được nhanh như client side redering.
- Mỗi lần chuyển trang là phải load trang lại một lần gây lên khó chịu và tốn băng thông do gửi nhiều dữ liệu dư thừa, thậm chí nếu nhiều yêu cầu có thể làm server mất đi tính sẵn sàng.

#### Về SEO 
- SEO lên top tìm kiếm tốt hơn vì bot của Google, Yahoo, Bing sẽ vào web và crawl toàn bộ mã html về.

#### Tương thích 
- Chạy được phần lớn trên mọi trình duyệt, kể cả disable javascript vẫn có thể chạy được.

#### Tương tác 
- Tương tác không được tốt như client side redering vì trang phải load đi load lại nhiều lần.

## Client side redering
#### Về việc phát triển : 
- Client side redering khiến việc phát triển web phân chia rõ ràng back-end, front-end và nhiệm vụ của từng bộ phận. Việc viết mã nguồn trở lên phức tạp hơn.
	
#### Tốc độ tải trang : 
- Trong lần đầu tiên tải trang có thể lâu vì website sẽ phải tải toàn bộ javascript về máy khách. Tuy nhiên, trong các lần tiếp theo có thể sẽ không mất nhiều thời gian khi việc xử lý các logic đơn giản được thực hiện trực tiếp trên máy khách và không cần load trang, thậm chí khi không có mạng website vẫn có thể hoạt động như một ứng dụng desktop nếu như không có gửi những truy vấn về phía server.
- Nếu client sử dụng các máy điện thoại hay thiết bị cũ có thể ảnh hưởng tới tốc độ tải trang.
	
#### Về SEO :
- Các máy tìm kiếm thường không tương thích chạy javascript  khi crawler dữ liệu trang web do đó việc có một thứ hạng tốt trên các công cụ tìm kiếm là rất khó khăn mặc dù Google đang thay đổi để phù hợp hơn.
	
#### Tương thích : 
- Không tương thích với các trình duyệt cũ không hỗ trợ javascript hay là javascript bị disable.
	
#### Tương tác : 
- Cung cấp trải nghiệm tốt hơn cho người dùng, giao diện đẹp mắt.
<br />
Tham khảo : [toidicodedao.com](https://toidicodedao.com/)
