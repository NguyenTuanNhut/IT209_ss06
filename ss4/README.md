# Bài 4: Quản lý tiến trình nền với nohup và tín hiệu Kill

## 1. Mục tiêu

* Tạo và chạy shell script dưới nền.
* Sử dụng `nohup` để tiến trình tiếp tục chạy khi đóng Terminal hoặc ngắt kết nối SSH.
* Sử dụng `ps` và `pgrep` để tìm PID của tiến trình.
* Kiểm tra dữ liệu được ghi liên tục vào file log.
* Sử dụng tín hiệu `SIGTERM (15)` để dừng tiến trình an toàn.
* Hiểu cách sử dụng `SIGKILL (9)` khi tiến trình không thể dừng bằng `SIGTERM`.

---

## 2. Tạo shell script loop-monitor.sh

Tạo file `loop-monitor.sh`:

```bash
cat << 'EOF' > loop-monitor.sh
#!/bin/bash

while true; do
    echo "System time: $(date)" >> /tmp/monitor.log
    sleep 5
done
EOF
```

### Kiểm tra nội dung file

```bash
cat loop-monitor.sh
```

### Kết quả

```bash
azureuser@mayao2:~$ cat loop-monitor.sh
#!/bin/bash

while true; do
    echo "System time: $(date)" >> /tmp/monitor.log
    sleep 5
done
```

---

## 3. Gán quyền thực thi

### Lệnh

```bash
chmod +x loop-monitor.sh
```

### Kiểm tra quyền

```bash
ls -l loop-monitor.sh
```

### Kết quả

```bash
azureuser@mayao2:~$ ls -l loop-monitor.sh
-rwxr-xr-x 1 azureuser azureuser 118 Oct  7 20:40 loop-monitor.sh
```

Ký tự `x` cho biết file đã được cấp quyền thực thi.

---

## 4. Khởi chạy script bằng nohup

Sử dụng `nohup` kết hợp với `&` để chạy script dưới nền và không phụ thuộc vào Terminal hiện tại.

### Lệnh

```bash
nohup ./loop-monitor.sh > /dev/null 2>&1 &
```

### Kết quả

```bash
azureuser@mayao2:~$ nohup ./loop-monitor.sh > /dev/null 2>&1 &
[1] 2458
```

Trong đó:

* `[1]` là số thứ tự job trong shell hiện tại.
* `2458` là PID của tiến trình `loop-monitor.sh`.

PID thực tế có thể khác tùy từng máy.

---

## 5. Tìm PID của tiến trình

Sử dụng `pgrep`:

```bash
pgrep -f loop-monitor.sh
```

### Kết quả

```bash
azureuser@mayao2:~$ pgrep -f loop-monitor.sh
2458
```

PID của tiến trình là:

```text
2458
```

---

## 6. Kiểm tra tiến trình bằng ps

### Lệnh

```bash
ps aux | grep loop-monitor.sh
```

### Kết quả

```bash
azureuser@mayao2:~$ ps aux | grep loop-monitor.sh
azureuser   2458  0.0  0.1   7236  2048 pts/0    S    20:40   0:00 /bin/bash ./loop-monitor.sh
azureuser   2480  0.0  0.0   6544   736 pts/0    S+   20:41   0:00 grep --color=auto loop-monitor.sh
```

Dòng chứa:

```text
/bin/bash ./loop-monitor.sh
```

là tiến trình đang chạy.

Dòng:

```text
grep --color=auto loop-monitor.sh
```

chỉ là chính lệnh `grep`, không phải tiến trình cần quản lý.

---

## 7. Kiểm tra file log

Script sẽ ghi thời gian hiện tại vào `/tmp/monitor.log` sau mỗi 5 giây.

### Lệnh

```bash
tail -n 10 /tmp/monitor.log
```

### Kết quả

```bash
azureuser@mayao2:~$ tail -n 10 /tmp/monitor.log
System time: Wed Oct  7 20:40:12 +07 2026
System time: Wed Oct  7 20:40:17 +07 2026
System time: Wed Oct  7 20:40:22 +07 2026
System time: Wed Oct  7 20:40:27 +07 2026
System time: Wed Oct  7 20:40:32 +07 2026
System time: Wed Oct  7 20:40:37 +07 2026
System time: Wed Oct  7 20:40:42 +07 2026
System time: Wed Oct  7 20:40:47 +07 2026
System time: Wed Oct  7 20:40:52 +07 2026
System time: Wed Oct  7 20:40:57 +07 2026
```

Các dòng log cách nhau khoảng 5 giây, chứng minh script đang chạy liên tục.

---

## 8. Kiểm tra script vẫn chạy sau khi đăng nhập lại

Sau khi thoát SSH:

```bash
exit
```

Đăng nhập lại máy chủ và kiểm tra:

```bash
pgrep -f loop-monitor.sh
```

### Kết quả

```bash
azureuser@mayao2:~$ pgrep -f loop-monitor.sh
2458
```

Kiểm tra log:

```bash
tail -n 5 /tmp/monitor.log
```

### Kết quả

```bash
azureuser@mayao2:~$ tail -n 5 /tmp/monitor.log
System time: Wed Oct  7 20:45:12 +07 2026
System time: Wed Oct  7 20:45:17 +07 2026
System time: Wed Oct  7 20:45:22 +07 2026
System time: Wed Oct  7 20:45:27 +07 2026
System time: Wed Oct  7 20:45:32 +07 2026
```

Điều này chứng minh tiến trình vẫn hoạt động sau khi phiên SSH cũ kết thúc.

---

## 9. Dừng tiến trình bằng SIGTERM

Sử dụng tín hiệu `SIGTERM (15)` để yêu cầu tiến trình kết thúc một cách an toàn.

### Lệnh

```bash
kill -15 2458
```

### Kết quả

```bash
azureuser@mayao2:~$ kill -15 2458
```

Thông thường lệnh `kill` không hiển thị thông báo nếu thực hiện thành công.

---

## 10. Kiểm tra tiến trình sau khi kill

### Lệnh

```bash
pgrep -f loop-monitor.sh
```

### Kết quả

```bash
azureuser@mayao2:~$ pgrep -f loop-monitor.sh
```

Không có PID được trả về, nghĩa là tiến trình đã dừng.

Có thể kiểm tra thêm:

```bash
ps aux | grep loop-monitor.sh
```

### Kết quả

```bash
azureuser@mayao2:~$ ps aux | grep loop-monitor.sh
azureuser   2590  0.0  0.0   6544   736 pts/0    S+   20:46   0:00 grep --color=auto loop-monitor.sh
```

Chỉ còn tiến trình `grep`, không còn:

```text
/bin/bash ./loop-monitor.sh
```

Do đó tiến trình đã được dừng thành công.

---

## 11. Kiểm tra log sau khi dừng

### Lệnh

```bash
tail -n 5 /tmp/monitor.log
```

### Kết quả

```bash
azureuser@mayao2:~$ tail -n 5 /tmp/monitor.log
System time: Wed Oct  7 20:45:12 +07 2026
System time: Wed Oct  7 20:45:17 +07 2026
System time: Wed Oct  7 20:45:22 +07 2026
System time: Wed Oct  7 20:45:27 +07 2026
System time: Wed Oct  7 20:45:32 +07 2026
```

Sau khi tiến trình bị dừng, file log không tiếp tục được ghi thêm dữ liệu.

---

## 12. Trường hợp SIGTERM không dừng được tiến trình

Nếu tiến trình không phản hồi với `SIGTERM`, có thể sử dụng `SIGKILL (9)`.

### Cú pháp

```bash
kill -9 <PID>
```

Ví dụ:

```bash
kill -9 2458
```

`SIGKILL` buộc hệ thống kết thúc tiến trình ngay lập tức và không cho tiến trình cơ hội xử lý tín hiệu trước khi thoát.

Trong bài thực hành này, `SIGTERM (15)` đã dừng tiến trình thành công nên không cần sử dụng `SIGKILL (9)`.

---

## 13. Giải thích các lệnh

| Lệnh                       | Chức năng                                                   |
| -------------------------- | ----------------------------------------------------------- |
| `chmod +x loop-monitor.sh` | Cấp quyền thực thi cho script                               |
| `nohup`                    | Cho phép tiến trình tiếp tục chạy khi Terminal/SSH kết thúc |
| `&`                        | Chạy tiến trình dưới nền                                    |
| `pgrep -f`                 | Tìm PID dựa trên tên/lệnh của tiến trình                    |
| `ps aux`                   | Liệt kê các tiến trình đang chạy                            |
| `tail -n 10`               | Xem 10 dòng cuối của file                                   |
| `kill -15`                 | Gửi tín hiệu SIGTERM để dừng tiến trình an toàn             |
| `kill -9`                  | Gửi SIGKILL để buộc tiến trình kết thúc                     |

---

## 14. Kiểm tra cuối cùng

### Khởi chạy

```bash
nohup ./loop-monitor.sh > /dev/null 2>&1 &
```

### Tìm PID

```bash
pgrep -f loop-monitor.sh
```

Kết quả:

```bash
2458
```

### Kiểm tra log

```bash
tail -n 10 /tmp/monitor.log
```

Kết quả:

```text
System time: Wed Oct  7 20:40:12 +07 2026
System time: Wed Oct  7 20:40:17 +07 2026
System time: Wed Oct  7 20:40:22 +07 2026
System time: Wed Oct  7 20:40:27 +07 2026
System time: Wed Oct  7 20:40:32 +07 2026
System time: Wed Oct  7 20:40:37 +07 2026
System time: Wed Oct  7 20:40:42 +07 2026
System time: Wed Oct  7 20:40:47 +07 2026
System time: Wed Oct  7 20:40:52 +07 2026
System time: Wed Oct  7 20:40:57 +07 2026
```

### Dừng tiến trình

```bash
kill -15 2458
```

### Kiểm tra sau khi dừng

```bash
pgrep -f loop-monitor.sh
```

Kết quả:

```text
```

Không còn PID, chứng minh tiến trình đã được dừng.

---

## 15. Kết luận

Đã hoàn thành bài thực hành quản lý tiến trình nền.

Script `loop-monitor.sh` được chạy bằng `nohup` kết hợp với `&`, giúp tiến trình tiếp tục hoạt động ngay cả khi phiên SSH bị ngắt. Tiến trình được tìm bằng `pgrep` và `ps`, đồng thời kiểm tra hoạt động thông qua file `/tmp/monitor.log`.

Sau đó sử dụng tín hiệu `SIGTERM (15)` thông qua lệnh `kill -15` để dừng tiến trình một cách an toàn. Trong trường hợp tiến trình không phản hồi, có thể sử dụng `SIGKILL (9)` để buộc tiến trình kết thúc.
