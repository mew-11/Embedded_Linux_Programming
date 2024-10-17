Dưới đây là một số ví dụ cụ thể về cách sử dụng `perf stat` và `perf top` để phân tích hiệu suất trên hệ thống Linux:

### 1. **Ví dụ sử dụng `perf stat`**

`perf stat` được sử dụng để thu thập thống kê hiệu suất chung cho một lệnh hoặc chương trình. Nó cung cấp thông tin về thời gian thực thi, số chu kỳ CPU, số lỗi bộ nhớ cache, và nhiều thông tin hữu ích khác.

#### Ví dụ 1: Thu thập số liệu cho lệnh `ls`

```bash
perf stat ls /usr/share
```

Trong ví dụ này, `perf stat` sẽ thu thập và hiển thị số liệu thống kê khi bạn liệt kê các tệp trong thư mục `/usr/share`. Đầu ra sẽ bao gồm:

- Số chu kỳ CPU.
- Số lần cache bị lỗi.
- Số lần chuyển đổi ngữ cảnh (context switches).
- Thời gian thực hiện lệnh.

Kết quả mẫu:

```bash
 Performance counter stats for 'ls /usr/share':

      0.002653      task-clock (msec)         #    0.001 CPUs utilized
             2      context-switches          #    0.755 K/sec
             0      cpu-migrations            #    0.000 K/sec
            61      page-faults               #    23.002 K/sec
       666,224      cycles                    #    251.159 M/sec
       513,029      instructions              #    0.77  insn per cycle
       101,118      branches                  #    38.119 M/sec
         1,177      branch-misses             #     1.16% of all branches

      0.002659039 seconds time elapsed
```

#### Ví dụ 2: Thu thập dữ liệu khi chạy một chương trình C

Giả sử bạn có một chương trình C tên là `my_program`. Bạn có thể sử dụng `perf stat` để thu thập thông tin chi tiết về hiệu suất của chương trình này.

```bash
perf stat ./my_program
```

Kết quả sẽ cung cấp thông tin như số chu kỳ CPU, bộ nhớ cache bị lỗi, thời gian chương trình chạy, v.v.

#### Ví dụ 3: Đo hiệu suất trong một khoảng thời gian nhất định

Nếu bạn muốn đo hiệu suất tổng thể của hệ thống trong một khoảng thời gian nhất định, bạn có thể sử dụng lệnh sau:

```bash
perf stat -a sleep 5
```

- **`-a`**: Ghi lại tất cả các CPU (toàn hệ thống).
- **`sleep 5`**: Thu thập số liệu trong 5 giây.

### 2. **Ví dụ sử dụng `perf top`**

`perf top` là một công cụ phân tích hiệu suất thời gian thực tương tự như lệnh `top` nhưng nó hiển thị các hàm và lệnh chiếm nhiều tài nguyên CPU nhất trong hệ thống.

#### Ví dụ 1: Theo dõi sử dụng CPU thời gian thực

Để chạy `perf top`, bạn chỉ cần thực hiện lệnh sau:

```bash
sudo perf top
```

`perf top` sẽ hiển thị danh sách các hàm hoặc lệnh đang chiếm nhiều tài nguyên CPU nhất trong hệ thống của bạn. Kết quả sẽ trông giống như sau:

```bash
Samples: 23K of event 'cycles', Event count (approx.): 5346209200
Overhead  Symbol
  12.45%  [kernel]      [k] __schedule
   8.12%  [kernel]      [k] finish_task_switch
   5.67%  libc-2.31.so  [.] _int_malloc
   4.23%  [kernel]      [k] do_syscall_64
   3.84%  libc-2.31.so  [.] _int_free
   3.21%  [kernel]      [k] entry_SYSCALL_64_after_hwframe
   2.98%  my_program    [.] main
```

- **Overhead**: Tỷ lệ phần trăm thời gian CPU dành cho một hàm hoặc lệnh cụ thể.
- **Symbol**: Tên của hàm hoặc lệnh.
- **[k]**: Các hàm trong kernel (hạt nhân).
- **[.]**: Các hàm trong ứng dụng người dùng (user-space).

Ví dụ, ở trên, `__schedule` là hàm trong kernel chiếm 12.45% tài nguyên CPU.

#### Ví dụ 2: Theo dõi sử dụng CPU với sự kiện cụ thể

Bạn có thể theo dõi các sự kiện cụ thể như chu kỳ CPU (`cycles`) hoặc lỗi dự đoán nhánh (`branch-misses`) trong thời gian thực:

```bash
sudo perf top -e cycles
```

Lệnh này sẽ chỉ hiển thị các sự kiện liên quan đến chu kỳ CPU.

#### Ví dụ 3: Theo dõi một chương trình cụ thể

Bạn có thể sử dụng `perf top` để theo dõi hiệu suất của một chương trình cụ thể. Ví dụ, để theo dõi chương trình `my_program`, bạn có thể chạy:

```bash
sudo perf top -p $(pgrep my_program)
```

- **`-p`**: Chỉ định PID của quá trình để theo dõi. Bạn có thể dùng `pgrep` để lấy PID của chương trình đang chạy.

### Tổng kết

- **`perf stat`**: Thu thập số liệu thống kê về hiệu suất, hữu ích khi bạn muốn biết tổng quan về số chu kỳ CPU, số lần lỗi cache, chuyển đổi ngữ cảnh, v.v.
- **`perf top`**: Theo dõi thời gian thực, hữu ích để xác định chính xác những hàm hoặc lệnh nào trong hệ thống đang chiếm nhiều CPU nhất.

Bạn có thể kết hợp cả hai công cụ này để tối ưu hóa chương trình của mình, xác định các vấn đề liên quan đến CPU hoặc bộ nhớ, và cải thiện hiệu suất tổng thể.
