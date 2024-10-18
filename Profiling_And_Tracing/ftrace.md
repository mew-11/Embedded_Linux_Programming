### **ftrace**: Công cụ Tracing mạnh mẽ trên Linux

`ftrace` là một công cụ tracing cấp kernel mạnh mẽ được tích hợp trong hệ điều hành Linux. Nó được thiết kế để theo dõi và ghi lại các hoạt động trong nhân (kernel) và các tiến trình người dùng tương tác với kernel. `ftrace` cung cấp khả năng theo dõi các cuộc gọi hàm trong kernel, các sự kiện ngắt (interrupt), chuyển đổi tiến trình (context switch), và nhiều hơn nữa.

### **Mục tiêu của ftrace**

Mục tiêu chính của `ftrace` là giúp lập trình viên và quản trị hệ thống hiểu cách kernel hoạt động và tương tác với các tiến trình người dùng. Nó giúp phát hiện và chẩn đoán các vấn đề về hiệu suất, lỗi trong kernel, và các điểm nghẽn trong hệ thống. `ftrace` có thể:

- Theo dõi các **cuộc gọi hàm** trong kernel.
- Theo dõi các **ngắt** (interrupt) và **context switches**.
- Giám sát các hoạt động **I/O**, **hệ thống tệp** (file system), và **mạng**.
- Giúp phát hiện **điểm nghẽn** về hiệu suất trong kernel hoặc ứng dụng.

### **Các thành phần chính của ftrace**

1. **Tracers (Bộ theo dõi)**:

   - **Function tracer**: Theo dõi và ghi lại tất cả các cuộc gọi hàm trong kernel.
   - **Sched tracer**: Theo dõi hoạt động của lịch biểu (scheduler), như việc chuyển đổi tiến trình (context switches), ngắt (interrupts), và hoạt động của CPU.
   - **IRQ tracer**: Theo dõi các sự kiện ngắt trong hệ thống.
   - **Wakeup tracer**: Theo dõi thời gian một tiến trình đang ngủ (sleep) và thời gian nó được đánh thức.

2. **Tracepoints**: Là các điểm được đặt trong mã nguồn kernel tại những vị trí mà các sự kiện quan trọng xảy ra. `ftrace` có thể ghi lại các thông tin tại các tracepoints này.

3. **Ring buffer**: Là nơi lưu trữ các sự kiện mà `ftrace` thu thập được. Dữ liệu tracing sẽ được ghi vào bộ nhớ đệm vòng (ring buffer) và có thể được truy xuất sau đó.

### **Cách sử dụng cơ bản của ftrace**

`ftrace` có thể được truy cập và điều khiển thông qua hệ thống tệp ảo **`/sys/kernel/debug/tracing`**, nơi lưu trữ các tệp cấu hình và kết quả tracing.

#### **1. Kiểm tra trạng thái hiện tại của ftrace**

Trước khi bắt đầu, bạn có thể kiểm tra thư mục `ftrace`:

```bash
cd /sys/kernel/debug/tracing
ls
```

Thư mục này chứa nhiều tệp và thư mục con để cấu hình và kiểm soát quá trình tracing.

#### **2. Bật chế độ tracing hàm (function tracing)**

Một trong những tính năng mạnh nhất của `ftrace` là **function tracer**, cho phép bạn theo dõi tất cả các cuộc gọi hàm trong kernel.

Để bật function tracing:

```bash
echo function > current_tracer
```

Lệnh này sẽ chỉ định sử dụng `function` tracer, ghi lại tất cả các cuộc gọi hàm.

#### **3. Bắt đầu và dừng quá trình tracing**

- **Bắt đầu tracing**:

```bash
echo 1 > tracing_on
```

- **Dừng tracing**:

```bash
echo 0 > tracing_on
```

#### **4. Xem kết quả tracing**

Kết quả của quá trình tracing được ghi vào tệp `trace`:

```bash
cat trace
```

Tệp này chứa danh sách các cuộc gọi hàm đã xảy ra trong kernel. Ví dụ, kết quả có thể trông như thế này:

```
# tracer: function
#
# CPU  DURATION                  FUNCTION
# |     |    |                     |
  0)   0.123 us    |  trace_function_entry();
  0)   0.234 us    |  another_kernel_function();
  1)   1.002 us    |  some_other_kernel_function();
```

#### **5. Xóa dữ liệu trace**

Để xóa sạch dữ liệu trong `trace`, bạn có thể sử dụng:

```bash
echo > trace
```

#### **6. Sử dụng `set_ftrace_filter` để lọc các hàm cần theo dõi**

Thay vì ghi lại tất cả các hàm, bạn có thể chỉ định một danh sách các hàm cụ thể cần theo dõi bằng tệp `set_ftrace_filter`. Điều này giúp thu hẹp phạm vi tracing và chỉ tập trung vào các hàm quan trọng.

Ví dụ, để chỉ theo dõi các hàm liên quan đến `schedule`, bạn có thể làm như sau:

```bash
echo 'schedule' > set_ftrace_filter
```

#### **7. Theo dõi chuyển đổi tiến trình (context switch)**

Một cách khác để sử dụng `ftrace` là theo dõi các sự kiện liên quan đến lịch biểu (scheduler), bao gồm chuyển đổi tiến trình (context switch). Để làm điều này, bạn có thể sử dụng bộ theo dõi `sched`:

```bash
echo sched > current_tracer
echo 1 > tracing_on
```

Khi bật `sched` tracer, bạn có thể theo dõi các sự kiện như khi nào tiến trình bị chuyển đổi, hoặc khi nào một tiến trình bị đặt vào hàng đợi.

#### **8. Theo dõi hoạt động I/O**

Bạn cũng có thể sử dụng `ftrace` để theo dõi các hoạt động liên quan đến I/O và hệ thống tệp bằng cách sử dụng các tracepoints hoặc bộ theo dõi I/O tích hợp.

---

### **Ví dụ thực tế với ftrace**

#### **1. Theo dõi tất cả các hàm liên quan đến `write()` trong kernel**

Bạn muốn theo dõi các cuộc gọi hàm xảy ra trong kernel khi một tiến trình gọi hệ thống `write()` để ghi dữ liệu vào một tệp. Bạn có thể làm như sau:

```bash
# Bật function tracing
echo function > current_tracer

# Bật tracing
echo 1 > tracing_on

# Chạy một lệnh ghi dữ liệu vào tệp
echo "Hello ftrace" > /tmp/testfile

# Dừng tracing
echo 0 > tracing_on

# Xem kết quả
cat trace
```

Kết quả sẽ hiển thị các hàm kernel được gọi trong quá trình thực hiện lệnh `write()`.

#### **2. Theo dõi thời gian tiến trình bị đánh thức**

Bạn có thể sử dụng `ftrace` để theo dõi bao lâu thì một tiến trình đang ngủ bị đánh thức. Sử dụng bộ theo dõi `wake_up` để biết thời gian giữa khi tiến trình ngủ và khi nó được đánh thức.

```bash
echo wakeup_rt > current_tracer
echo 1 > tracing_on
# Sau đó kiểm tra kết quả trong tệp trace
cat trace
```

---

### **Các tùy chọn cấu hình khác của ftrace**

- **`trace_options`**: Bạn có thể bật hoặc tắt các tùy chọn bổ sung để tùy chỉnh quá trình tracing, chẳng hạn như bật/tắt hiển thị thời gian CPU, dấu thời gian, v.v.

  Ví dụ, để bật dấu thời gian:

  ```bash
  echo no_funcgraph_abstime > trace_options
  ```

- **`set_event`**: Cho phép bạn bật hoặc tắt việc ghi lại các sự kiện cụ thể, chẳng hạn như các sự kiện liên quan đến I/O, mạng, hoặc bộ nhớ.

  Ví dụ, để bật các sự kiện liên quan đến lịch biểu (sched):

  ```bash
  echo sched:sched_switch > set_event
  ```

---

### **Ưu điểm của ftrace**

- **Linh hoạt**: `ftrace` cung cấp nhiều bộ theo dõi (tracers) khác nhau, từ việc theo dõi các cuộc gọi hàm đơn giản đến theo dõi các sự kiện phức tạp như chuyển đổi tiến trình và ngắt.
- **Độ sâu phân tích cao**: Bạn có thể theo dõi chi tiết từng hàm trong kernel, giúp phát hiện lỗi hoặc tối ưu hóa hệ thống.
- **Tích hợp sẵn trong kernel**: Không cần cài đặt thêm phần mềm, `ftrace` là một phần của kernel Linux.

### **Nhược điểm của ftrace**

- **Khó sử dụng cho người mới**: `ftrace` có nhiều tính năng và tùy chọn cấu hình phức tạp, đòi hỏi người dùng phải có hiểu biết về kernel và hệ thống Linux.
- **Chi phí tài nguyên**: Tracing với `ftrace` có thể tiêu tốn nhiều tài nguyên, đặc biệt khi theo dõi nhiều hàm hoặc sự kiện cùng lúc.
