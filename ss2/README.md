# Bài 2: Cấu hình phân quyền Nhóm và sudoers bằng visudo

## 1. Mục tiêu

* Tạo nhóm người dùng `devops-admin`.
* Tạo tài khoản người dùng `deployer`.
* Thêm tài khoản `deployer` vào nhóm `devops-admin`.
* Cấu hình `sudoers` để nhóm `devops-admin` được phép sử dụng các lệnh `systemctl`.
* Các lệnh `systemctl` được cấp quyền theo yêu cầu mà không cần nhập mật khẩu (`NOPASSWD`).
* Kiểm tra quyền bằng lệnh `sudo -l`.

---

## 2. Tạo nhóm devops-admin

### Lệnh thực hiện

```bash
sudo groupadd devops-admin
```

### Kiểm tra nhóm

```bash
getent group devops-admin
```

### Kết quả

```bash
azureuser@mayao2:~$ sudo groupadd devops-admin

azureuser@mayao2:~$ getent group devops-admin
devops-admin:x:1002:
```

---

## 3. Tạo tài khoản deployer

### Lệnh thực hiện

```bash
sudo adduser deployer
```

Trong quá trình tạo tài khoản, nhập mật khẩu cho `deployer`.

Ví dụ:

```text
Adding user `deployer' ...
Adding new group `deployer' (1003) ...
Adding new user `deployer' (1003) with group `deployer' ...
Creating home directory `/home/deployer' ...
Copying files from `/etc/skel' ...

New password:
Retype new password:
passwd: password updated successfully
Changing the user information for deployer
Enter the new value, or press ENTER for the default
        Full Name []: Deployer
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] y
```

---

## 4. Thêm deployer vào nhóm devops-admin

### Lệnh thực hiện

```bash
sudo usermod -aG devops-admin deployer
```

### Kiểm tra

```bash
groups deployer
```

### Kết quả

```bash
azureuser@mayao2:~$ groups deployer
deployer : deployer devops-admin
```

Như vậy tài khoản `deployer` đã thuộc nhóm `devops-admin`.

---

## 5. Cấu hình sudoers bằng visudo

Mở file cấu hình sudoers:

```bash
sudo visudo
```

Thêm dòng sau vào cuối file:

```text
%devops-admin ALL=(ALL) NOPASSWD: /usr/bin/systemctl start *, /usr/bin/systemctl stop *, /usr/bin/systemctl restart *, /usr/bin/systemctl status *
```

Sau khi thêm cấu hình, lưu và thoát `visudo`.

---

## 6. Kiểm tra cấu hình sudoers

Có thể kiểm tra trực tiếp bằng:

```bash
sudo visudo -c
```

### Kết quả

```bash
azureuser@mayao2:~$ sudo visudo -c
/etc/sudoers: parsed OK
/etc/sudoers.d/README: parsed OK
```

Kết quả `parsed OK` cho biết file sudoers có cú pháp hợp lệ.

---

## 7. Chuyển sang tài khoản deployer

### Lệnh

```bash
su - deployer
```

### Kết quả

```bash
azureuser@mayao2:~$ su - deployer
Password:

deployer@mayao2:~$
```

Sau khi đăng nhập, kiểm tra nhóm:

```bash
groups
```

### Kết quả

```bash
deployer@mayao2:~$ groups
deployer devops-admin
```

---

## 8. Kiểm tra quyền sudo

### Lệnh

```bash
sudo -l
```

### Kết quả

```bash
deployer@mayao2:~$ sudo -l
Matching Defaults entries for deployer on mayao2:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

User deployer may run the following commands on mayao2:

    (ALL) NOPASSWD: /usr/bin/systemctl start *
    (ALL) NOPASSWD: /usr/bin/systemctl stop *
    (ALL) NOPASSWD: /usr/bin/systemctl restart *
    (ALL) NOPASSWD: /usr/bin/systemctl status *
```

Kết quả cho thấy tài khoản `deployer` được phép sử dụng các lệnh `systemctl` đã được khai báo trong sudoers mà không cần nhập mật khẩu.

---

## 9. Kiểm tra lệnh systemctl status

Thử kiểm tra trạng thái dịch vụ `cron`:

```bash
sudo systemctl status cron
```

### Kết quả

```bash
deployer@mayao2:~$ sudo systemctl status cron
● cron.service - Regular background program processing daemon
     Loaded: loaded (/usr/lib/systemd/system/cron.service; enabled)
     Active: active (running)
```

Không xuất hiện yêu cầu nhập mật khẩu.

---

## 10. Kiểm tra lệnh restart

Thực hiện restart dịch vụ `cron`:

```bash
sudo systemctl restart cron
```

### Kết quả

```bash
deployer@mayao2:~$ sudo systemctl restart cron
deployer@mayao2:~$
```

Lệnh thực hiện thành công và không yêu cầu nhập mật khẩu.

Có thể kiểm tra lại:

```bash
sudo systemctl status cron
```

### Kết quả

```bash
deployer@mayao2:~$ sudo systemctl status cron
● cron.service - Regular background program processing daemon
     Loaded: loaded (/usr/lib/systemd/system/cron.service; enabled)
     Active: active (running)
```

---

## 11. Kiểm tra cấu hình sudoers

Dòng cấu hình đã thêm:

```text
%devops-admin ALL=(ALL) NOPASSWD: /usr/bin/systemctl start *, /usr/bin/systemctl stop *, /usr/bin/systemctl restart *, /usr/bin/systemctl status *
```

Trong đó:

| Thành phần            | Ý nghĩa                                                     |
| --------------------- | ----------------------------------------------------------- |
| `%devops-admin`       | Áp dụng cho tất cả user thuộc nhóm `devops-admin`           |
| `ALL`                 | Áp dụng trên tất cả máy chủ                                 |
| `(ALL)`               | Cho phép thực thi với quyền của các user khác, bao gồm root |
| `NOPASSWD`            | Không yêu cầu nhập mật khẩu                                 |
| `systemctl start *`   | Cho phép khởi động dịch vụ                                  |
| `systemctl stop *`    | Cho phép dừng dịch vụ                                       |
| `systemctl restart *` | Cho phép khởi động lại dịch vụ                              |
| `systemctl status *`  | Cho phép xem trạng thái dịch vụ                             |

---

## 12. Kiểm tra cuối cùng

### Kiểm tra nhóm

```bash
groups deployer
```

### Kết quả

```bash
deployer : deployer devops-admin
```

### Kiểm tra sudo

```bash
sudo -l
```

### Kết quả

```bash
User deployer may run the following commands on mayao2:

    (ALL) NOPASSWD: /usr/bin/systemctl start *
    (ALL) NOPASSWD: /usr/bin/systemctl stop *
    (ALL) NOPASSWD: /usr/bin/systemctl restart *
    (ALL) NOPASSWD: /usr/bin/systemctl status *
```

### Kiểm tra restart cron

```bash
sudo systemctl restart cron
```

### Kết quả

```bash
deployer@mayao2:~$ sudo systemctl restart cron
deployer@mayao2:~$
```

Lệnh thực hiện thành công mà không yêu cầu nhập mật khẩu.

---

## 13. Tổng kết

| Nội dung                           | Kết quả    |
| ---------------------------------- | ---------- |
| Tạo group `devops-admin`           | Hoàn thành |
| Tạo user `deployer`                | Hoàn thành |
| Thêm `deployer` vào `devops-admin` | Hoàn thành |
| Cấu hình sudoers bằng `visudo`     | Hoàn thành |
| `systemctl start`                  | Được phép  |
| `systemctl stop`                   | Được phép  |
| `systemctl restart`                | Được phép  |
| `systemctl status`                 | Được phép  |
| Không yêu cầu mật khẩu             | `NOPASSWD` |
| Restart `cron`                     | Thành công |

## 14. Kết luận

Đã hoàn thành cấu hình phân quyền nhóm và `sudoers`. Tài khoản `deployer` thuộc nhóm `devops-admin` và chỉ được cấp quyền thực hiện các lệnh quản lý dịch vụ `systemctl start`, `stop`, `restart` và `status` theo cấu hình sudoers. Các lệnh này có thể được thực hiện mà không cần nhập mật khẩu nhờ tùy chọn `NOPASSWD`.
