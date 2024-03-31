---
title: Đa luồng và đa tiến trình trong Python
author: viethoang
date: 2021-09-20 14:52:00 +0700
categories: [Bigdata, Share]
tags: [Python]
math: true
mermaid: true
---

Trong một lần phỏng vấn mình có được hỏi về các khái niệm này, lúc đó do kiến thức mình hiểu có phần bị sai vì thế mình quyết định là tìm hiểu lại và viết lại một số vấn đề trong lập trình đa luồng và đa tiến trình trong Python

# 1. Một số khái niệm cơ bản
## Tiến trình (Process)
Tiến trình đơn giản có thể hiểu là một chương trình đang được thực thi. Khi chương trình được đưa vào bộ nhớ nó thành tiến trình và được chia thành các 4 phần stack, heap,text,data. Các tiến trình chạy trong không gian riêng biệt lẫn nhau

Mỗi tiến trình trong hệ điều hành được biểu diễn bằng 1 cấu trúc dữ liệu gọi là PCB bao gồm tất cả các thông tin của tiến trình

| ![](https://1.bp.blogspot.com/-hRTr_-Aqkd0/W4PLcYiaDcI/AAAAAAAAGv0/B25xARlDeEQojZeOsJDPdHTBSQek4-RpQCLcBGAs/s1600/process-control-block.png)| 
|:--:| 
| *Các thông tin chứa trong PCB* |

## Luồng (Thread)
Luồng là một đơn vị thực thi trong một tiến trình. Một tiến trình có thể có một hay nhiều luồng. Các luồng không độc lập với nhau mà có thể chia sẻ phần tài nguyên và dữ liệu cho nhau.

| ![](https://s3.ap-south-1.amazonaws.com/afteracademy-server-uploads/what-is-a-thread-in-os-and-what-are-the-differences-between-a-process-and-a-thread-single-threaded-and-multiple-threaded-process-83d094ff508aaf5f.jpg)| 
|:--:| 
| *Luồng trong tiến trình* |

## Đồng bộ hóa (Synchronization)
Vấn đề đồng bộ hóa được chia thành hai dạng:
* Đồng bộ hóa tài nguyên: Xác định việc truy cập vào tài nguyên dùng chung có an toàn không.
* Đồng bộ hóa hoạt động: Đảm bảo việc thực thi các tác vụ chính xác khi phối hợp cùng nhau

# 2. Lập trình đa luồng và đa tiến trình trong Python
## Phân biệt đồng thời (concurrency) và song song (parallelism)

![](https://luminousmen.com/media/concurrency-and-parallelism-are-different.jpg)

* Lập trình đồng thời chỉ việc 2 hay nhiều tiến trình được xử lý xen kẽ thông qua cơ chế context switch trên 1 core CPU
* Lập trình song song tức là 2 hay nhiều tiến trình được xử lý song song với nhau trên nhiều core CPU

### IO và CPU bound
* CPU-bound chỉ việc chương trình sử dụng CPU phần lớn (tính toán). Việc tăng tốc những chương trình dạng này tức là làm sao để có thể thực hiện nhiều phép tính toán hơn trong cùng 1 khoảng thời gian
* IO-bound chỉ việc thời gian thực hiện chương trình phần lớn là do thời gian đợi các tác vụ IO hoàn tất  ví dụ đọc ghi từ ổ đĩa, bàn phím hay mạng. Khi đó CPU sẽ không được sử dụng

![](https://slidetodoc.com/presentation_image_h/dedc387b49807501e7f539cbeebcda69/image-20.jpg)

## Cơ chế GIL trong Python
Cơ chế GIL hay gọi là Global Intepreter Lock là một cơ chế trong python khóa toàn bộ trình thông dịch chỉ cho phép một luồng duy nhất được thực hiện. Do đó tại một thời điểm chỉ có một luồng được thực thi. Mỗi luồng muốn được thực thi thì GIL phải giải phóng luồng trước đó.

![](https://ichi.pro/assets/images/max/724/0*-byBEYjCl3q0yLZ-.png)

Vậy nên cơ chế này sẽ khiến các mô hình lập trình đa luồng không tăng hiệu suất mà có thể bị giảm hiệu suất.
# 3. Thử nghiệm đánh giá các mô hình
Chạy chương trình python dưới đây bao gồm 2 hàm là IO-bound và CPU-bound thực hiện tác vụ tương ứng với tên hàm

```python
import time, os
from threading import Thread, current_thread
from multiprocessing import Process, current_process


COUNT = 200000000
SLEEP = 10

def io_bound(sec):

	pid = os.getpid()
	threadName = current_thread().name
	processName = current_process().name

	print(f"{pid} * {processName} * {threadName} \
		---> Start sleeping...")
	time.sleep(sec)
	print(f"{pid} * {processName} * {threadName} \
		---> Finished sleeping...")

def cpu_bound(n):

	pid = os.getpid()
	threadName = current_thread().name
	processName = current_process().name

	print(f"{pid} * {processName} * {threadName} \
		---> Start counting...")

	while n>0:
		n -= 1

	print(f"{pid} * {processName} * {threadName} \
		---> Finished counting...")

if __name__=="__main__":
	start = time.time()

	# YOUR CODE SNIPPET HERE

	end = time.time()
	print('Time taken in seconds -', end - start)

```
## Thử nghiệm mô hình với các tác vụ IO-bound
Gọi 2 lần hàm io_bound() 

```python
io_bound(SLEEP)
io_bound(SLEEP)
```

Chương trình sẽ có thời gian chạy hơn 20s với 20s là sleep thực hiện tuần tự. Kết quả như dưới đây:

```bash
18795 * MainProcess * MainThread         ---> Start sleeping...
18795 * MainProcess * MainThread         ---> Finished sleeping...
18795 * MainProcess * MainThread         ---> Start sleeping...
18795 * MainProcess * MainThread         ---> Finished sleeping...
Time taken in seconds - 20.015657424926758
```

### Đa luồng
Ở đây mình sử dụng 2 luồng để tăng tốc việc xử lý. 2 luồng đều hoàn thành công việc trong 10s đồng thời và việc này giảm 50% thời gian thực thi chương trình

```python
t1 = Thread(target = io_bound, args =(SLEEP, ))

t2 = Thread(target = io_bound, args =(SLEEP, ))

t1.start()

t2.start()

t1.join()


t2.join()
```

Kết quả:

```bash
19327 * MainProcess * Thread-1         ---> Start sleeping...
19327 * MainProcess * Thread-2         ---> Start sleeping...
19327 * MainProcess * Thread-2         ---> Finished sleeping...
19327 * MainProcess * Thread-1         ---> Finished sleeping...
Time taken in seconds - 10.010849714279175
```

### Đa tiến trình

Trong trường hợp này mình sẽ tạo ra 2 tiến trình thực hiện song song hàm io_bound() trên và kết quả cũng tương tự phần đa luồng

```python
p1 = Process(target = io_bound, args =(SLEEP, ))

p2 = Process(target = io_bound, args =(SLEEP, ))

p1.start()

p2.start()

p1.join()

p2.join()
```

```bash
19967 * Process-1 * MainThread         ---> Start sleeping...
19968 * Process-2 * MainThread         ---> Start sleeping...
19967 * Process-1 * MainThread         ---> Finished sleeping...
19968 * Process-2 * MainThread         ---> Finished sleeping...
Time taken in seconds - 10.01881742477417
```

## Thử nghiệm mô hình với các tác vụ CPU-bound
Tương tự với phần trên mình gọi 2 lần hàm cpu_bound()

```
cpu_bound(COUNT)
cpu_bound(COUNT)
```

Kết quả của việc thực hiện tuần tự

```bash
20153 * MainProcess * MainThread         ---> Start counting...
20153 * MainProcess * MainThread         ---> Finished counting...
20153 * MainProcess * MainThread         ---> Start counting...
20153 * MainProcess * MainThread         ---> Finished counting...
Time taken in seconds - 20.83522891998291
```

### Đa luồng 
Mình sẽ thử thực hiện chương trình trên với hai luồng để kiểm tra xem đối với các công việc cpu-bound thì liệu việc đa luồng trong Python có được cải thiện hay không

```python
t1 = Thread(target = cpu_bound, args =(COUNT, ))

t2 = Thread(target = cpu_bound, args =(COUNT, ))

t1.start()

t2.start()

t1.join()

t2.join()
```

Kết quả cho thấy đối với các tác vụ CPU-bound thì việc lập trình đa luồng không cải thiện được tốc độ. Giải thích ở đây là do cơ chế GIL được trình bày ở trên chỉ cho phép trong một thời điểm chỉ có một luồng được chạy. Ở đây sau khi luồng 1 chạy thì GIL mới được giải phóng và luồng 2 mới được phép chạy. 

```
20338 * MainProcess * Thread-1         ---> Start counting...
20338 * MainProcess * Thread-2         ---> Start counting...
20338 * MainProcess * Thread-2         ---> Finished counting...
20338 * MainProcess * Thread-1         ---> Finished counting...
Time taken in seconds - 20.36056351661682
```

### Đa tiến trình

Đối với trường hợp đa tiến trình mình cũng sẽ thử nghiệm tương tự

```python
p1 = Process(target = cpu_bound, args =(COUNT, ))

p2 = Process(target = cpu_bound, args =(COUNT, ))

p1.start()

p2.start()

p1.join()

p2.join()

end = time.time()
```

Ở đây kết quả đã được cải thiện đáng kể, tốc độ thực thi giảm gần 50%. Tiến trình chính chia thành 2 tiến trình con chạy song song trên các core CPU. Mỗi tiến trình có một luồng chính riêng là MainThread

```bash
20556 * Process-1 * MainThread         ---> Start counting...
20557 * Process-2 * MainThread         ---> Start counting...
20557 * Process-2 * MainThread         ---> Finished counting...
20556 * Process-1 * MainThread         ---> Finished counting...
Time taken in seconds - 11.20688509941101
```

# 4. Kết luận
Trong Python thì việc sử dụng lập trình đa tiến trình sẽ giúp cải thiện tốc độ của chương trình đối với cả CPU-bound và IO-bound tuy nhiên cũng cần phải chú ý vì đa tiến trình tức là các tiến trình riêng biệt sử dụng thêm tài nguyên ram và CPU.

Đối với lập trình đa luồng do cơ chế GIL thì các tác vụ CPU-bound sẽ không cải thiện được vấn đề gì vì thực chất chỉ có 1 luồng được phép chạy và việc sử dụng có khi còn giảm hiệu quả. Tuy nhiên cũng có thể sử dụng trong trường hợp IO-bound, ở đây có thể lấy ví dụ đa luồng trong việc crawl web. Khi gửi request đến một trang thì thời gian IO luôn chiếm phần lớn, khi đó CPU nhàn rỗi có thể cho luồng khác sử dụng vì lúc này luồng phía trước không còn GIL nữa, luồng mới sẽ được sử dụng. Thực chất đây là lập trình đồng thời (concurrency)

![](https://files.realpython.com/media/Threading.3eef48da829e.png)
# Tham khảo
[Difference between multithreading and multiprocessing in Python](https://www.geeksforgeeks.org/difference-between-multithreading-vs-multiprocessing-in-python/)
[Beginers guide to concurrency and parallellism in Python](https://www.toptal.com/python/beginners-guide-to-concurrency-and-parallelism-in-python)
[Cơ chế GIL trong Python](https://nguyentruonglong.net/co-che-global-interpreter-lock-trong-python.html)
[Introduction to infamous Python GIL](https://granulate.io/introduction-to-the-infamous-python-gil/)
[Multithreading in Python](https://www.scaler.com/topics/multithreading-in-python/)
