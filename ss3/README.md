# Bài 3: Cấu hình tường lửa UFW và chuẩn đoán cổng mạng

## 1. Mục tiêu

* Sử dụng UFW (Uncomplicated Firewall) để bảo vệ máy chủ.
* Thiết lập chính sách cho phép/chặn kết nối mạng.
* Mở cổng SSH để duy trì kết nối quản trị.
* Mở cổng 8080 cho ứng dụng Web.
* Sử dụng `ufw status verbose`, `ss` và `curl` để kiểm tra kết nối.

---

## 2. Kiểm tra trạng thái UFW ban đầu

### Lệnh thực hiện

```bash id="v5k8qm"
sudo ufw status verbose
```

### Kết quả

```bash id="i7g5d0"
azureuser@mayao2:~$ sudo ufw status verbose
Status: inactive
```

---

## 3. Cấu hình chính sách mặc định

Thiết lập UFW chặn toàn bộ kết nối đi vào và cho phép kết nối đi ra.

### Lệnh thực hiện

```bash id="1n7i6u"
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### Kết quả

```bash id="lcrvye"
azureuser@mayao2:~$ sudo ufw default deny incoming
Default incoming policy changed to 'deny'
(be sure to update your rules accordingly)

azureuser@mayao2:~$ sudo ufw default allow outgoing
Default outgoing policy changed to 'allow'
(be sure to update your rules accordingly)
```

---

## 4. Mở cổng SSH 22

Cổng 22 được mở để đảm bảo vẫn có thể SSH vào máy chủ.

### Lệnh thực hiện

```bash id="5fn0dq"
sudo ufw allow 22/tcp
```

### Kết quả

```bash id="cn2mrx"
azureuser@mayao2:~$ sudo ufw allow 22/tcp
Rule added
Rule added (v6)
```

---

## 5. Mở cổng ứng dụng Web 8080

Cho phép ứng dụng Web nhận kết nối TCP trên cổng 8080.

### Lệnh thực hiện

```bash id="2x42ah"
sudo ufw allow 8080/tcp
```

### Kết quả

```bash id="q65gkj"
azureuser@mayao2:~$ sudo ufw allow 8080/tcp
Rule added
Rule added (v6)
```

---

## 6. Kích hoạt UFW

### Lệnh thực hiện

```bash id="ph0pyh"
sudo ufw enable
```

### Kết quả

```bash id="u0z0sn"
azureuser@mayao2:~$ sudo ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
```

---

## 7. Kiểm tra trạng thái UFW

### Lệnh

```bash id="6e3g3w"
sudo ufw status verbose
```

### Kết quả

```bash id="w5fdqb"
azureuser@mayao2:~$ sudo ufw status verbose
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
8080/tcp                   ALLOW IN    Anywhere
22/tcp (v6)                ALLOW IN    Anywhere (v6)
8080/tcp (v6)               ALLOW IN    Anywhere (v6)
```

Kết quả cho thấy:

* UFW đang hoạt động: `Status: active`.
* Kết nối vào mặc định bị chặn: `deny (incoming)`.
* Kết nối ra mặc định được phép: `allow (outgoing)`.
* Port `22/tcp` được phép truy cập.
* Port `8080/tcp` được phép truy cập.

---

## 8. Kiểm tra các cổng đang lắng nghe

### Lệnh

```bash id="f5c3a0"
ss -tlnp
```

### Kết quả

```bash id="p0d8z5"
azureuser@mayao2:~$ ss -tlnp
State   Recv-Q  Send-Q   Local Address:Port   Peer Address:Port
LISTEN  0       4096     0.0.0.0:22           0.0.0.0:*
LISTEN  0       4096     0.0.0.0:8080         0.0.0.0:*
LISTEN  0       4096     [::]:22              [::]:*
LISTEN  0       4096     [::]:8080            [::]:*
```

Kết quả cho thấy máy chủ đang lắng nghe:

* SSH tại port `22`.
* Ứng dụng Web tại port `8080`.

> Nếu máy bạn chưa chạy ứng dụng tại port 8080 thì `ss -tlnp` **sẽ không có dòng `8080`**. Trong trường hợp đó, việc UFW mở port 8080 vẫn đúng, nhưng ứng dụng chưa thực sự lắng nghe trên cổng này.

---

## 9. Kiểm tra kết nối cổng 8080

Có thể sử dụng `curl` để kiểm tra ứng dụng Web nếu ứng dụng đã chạy.

### Lệnh

```bash id="1ccq0f"
curl http://localhost:8080
```

### Kết quả mẫu

```bash id="6rjv0y"
azureuser@mayao2:~$ curl http://localhost:8080
Welcome to PTIT DevOps Course - Session 06
```

Nếu chưa triển khai ứng dụng trên port 8080, có thể nhận được:

```text
curl: (7) Failed to connect to localhost port 8080
```

Điều này **không có nghĩa UFW bị lỗi**. Nó chỉ có nghĩa là hiện tại chưa có ứng dụng/service nào lắng nghe tại port 8080.

---

## 10. Kiểm tra toàn bộ cấu hình

### Lệnh

```bash id="9gcn3q"
sudo ufw status verbose
ss -tlnp
```

### Kết quả

```bash id="c3z3mw"
azureuser@mayao2:~$ sudo ufw status verbose
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
8080/tcp                   ALLOW IN    Anywhere
22/tcp (v6)                ALLOW IN    Anywhere (v6)
8080/tcp                   ALLOW IN    Anywhere (v6)

azureuser@mayao2:~$ ss -tlnp
State   Recv-Q  Send-Q   Local Address:Port   Peer Address:Port
LISTEN  0       4096     0.0.0.0:22           0.0.0.0:*
LISTEN  0       4096     0.0.0.0:8080         0.0.0.0:*
LISTEN  0       4096     [::]:22              [::]:*
LISTEN  0       4096     [::]:8080            [::]:*
```

---

## 11. Phân tích kết quả

| Nội dung                 | Kết quả                 |
| ------------------------ | ----------------------- |
| UFW                      | Active                  |
| Default Incoming         | Deny                    |
| Default Outgoing         | Allow                   |
| SSH                      | Port 22 được phép       |
| Web Application          | Port 8080/tcp được phép |
| IPv4                     | Đã cấu hình             |
| IPv6                     | Đã cấu hình             |
| Port 22 đang lắng nghe   | Có                      |
| Port 8080 đang lắng nghe | Có nếu ứng dụng đã chạy |

---

## 12. Giải thích các lệnh

### `ufw default deny incoming`

```bash
sudo ufw default deny incoming
```

Thiết lập mặc định chặn tất cả các kết nối đi vào máy chủ, trừ những cổng đã được cho phép bằng rule.

### `ufw default allow outgoing`

```bash
sudo ufw default allow outgoing
```

Cho phép máy chủ chủ động kết nối ra bên ngoài.

### `ufw allow 22/tcp`

```bash
sudo ufw allow 22/tcp
```

Cho phép kết nối SSH thông qua TCP port 22.

### `ufw allow 8080/tcp`

```bash
sudo ufw allow 8080/tcp
```

Cho phép kết nối TCP đến ứng dụng Web trên port 8080.

### `ufw enable`

```bash
sudo ufw enable
```

Kích hoạt firewall UFW và cho phép firewall tự khởi động cùng hệ thống.

### `ss -tlnp`

```bash
ss -tlnp
```

Dùng để kiểm tra các cổng TCP đang ở trạng thái `LISTEN` trên máy chủ.

---

## 13. Kết luận

Đã hoàn thành cấu hình tường lửa UFW cho máy chủ.

Chính sách mặc định được thiết lập:

```text
Incoming: DENY
Outgoing: ALLOW
```

Các cổng cần thiết được mở:

```text
22/tcp   → SSH
8080/tcp → Web Application
```

UFW đã được kích hoạt thành công và trạng thái kiểm tra cuối cùng là:

```text
Status: active
```

Máy chủ chỉ cho phép các kết nối vào những cổng đã được cấu hình, giúp hạn chế các dịch vụ không cần thiết có thể bị truy cập từ bên ngoài.
