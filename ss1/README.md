# Bài 1: Khảo sát FHS và Phân quyền File/Folder nâng cao

## 1. Mục tiêu

* Tạo cấu trúc thư mục theo chuẩn FHS.
* Thực hành phân quyền file/folder bằng `chmod`.
* Thay đổi Owner và Group bằng `chown`.
* Kiểm tra quyền truy cập bằng `ls -la`.

---

## 2. Tạo cấu trúc thư mục

Tạo thư mục `/var/www/my-app` gồm hai thư mục con `public` và `logs`.

### Lệnh thực hiện

```bash
sudo mkdir -p /var/www/my-app/public
sudo mkdir -p /var/www/my-app/logs
```

### Kiểm tra

```bash
ls -la /var/www/my-app
```

### Kết quả

```bash
azureuser@mayao2:~$ ls -la /var/www/my-app
total 16
drwxr-xr-x 4 root root     4096 Oct  7 20:25 .
drwxr-xr-x 3 root root     4096 Oct  7 20:25 ..
drwxr-xr-x 2 root root     4096 Oct  7 20:25 logs
drwxr-xr-x 2 root root     4096 Oct  7 20:25 public
```

---

## 3. Thiết lập Owner và Group

Sử dụng tài khoản `devops1` làm Owner và nhóm `www-data` làm Group.

### Lệnh thực hiện

```bash
sudo chown -R devops1:www-data /var/www/my-app
```

### Kiểm tra

```bash
ls -la /var/www/my-app
```

### Kết quả

```bash
azureuser@mayao2:~$ ls -la /var/www/my-app
total 16
drwxr-xr-x 4 devops1 www-data 4096 Oct  7 20:25 .
drwxr-xr-x 3 root    root     4096 Oct  7 20:25 ..
drwxr-xr-x 2 devops1 www-data 4096 Oct  7 20:25 logs
drwxr-xr-x 2 devops1 www-data 4096 Oct  7 20:25 public
```

---

## 4. Phân quyền thư mục public

Yêu cầu:

* Owner: đọc, ghi, thực thi (`rwx`)
* Group: đọc, thực thi (`r-x`)
* Others: không có quyền (`---`)

Tương đương quyền `750`.

### Lệnh thực hiện

```bash
sudo chmod 750 /var/www/my-app/public
```

### Kiểm tra

```bash
ls -ld /var/www/my-app/public
```

### Kết quả

```bash
azureuser@mayao2:~$ ls -ld /var/www/my-app/public
drwxr-x--- 2 devops1 www-data 4096 Oct  7 20:25 /var/www/my-app/public
```

---

## 5. Phân quyền thư mục logs

Yêu cầu:

* Owner: đọc, ghi, thực thi (`rwx`)
* Group: đọc, ghi, thực thi (`rwx`)
* Others: không có quyền (`---`)

Tương đương quyền `770`.

### Lệnh thực hiện

```bash
sudo chmod 770 /var/www/my-app/logs
```

### Kiểm tra

```bash
ls -ld /var/www/my-app/logs
```

### Kết quả

```bash
azureuser@mayao2:~$ ls -ld /var/www/my-app/logs
drwxrwx--- 2 devops1 www-data 4096 Oct  7 20:25 /var/www/my-app/logs
```

---

## 6. Kiểm tra cuối cùng

### Lệnh

```bash
ls -la /var/www/my-app
```

### Kết quả

```bash
azureuser@mayao2:~$ ls -la /var/www/my-app
total 16
drwxr-xr-x 4 devops1 www-data 4096 Oct  7 20:25 .
drwxr-xr-x 3 root    root     4096 Oct  7 20:25 ..
drwxrwx--- 2 devops1 www-data 4096 Oct  7 20:25 logs
drwxr-x--- 2 devops1 www-data 4096 Oct  7 20:25 public
```

## 7. Phân tích kết quả

| Thư mục  | Owner     | Group      | Permission    | Octal |
| -------- | --------- | ---------- | ------------- | ----- |
| `public` | `devops1` | `www-data` | `rwx r-x ---` | `750` |
| `logs`   | `devops1` | `www-data` | `rwx rwx ---` | `770` |

### Giải thích

* `public` có quyền `750`: Owner được đọc, ghi và truy cập; Group được đọc và truy cập; Others không có quyền.
* `logs` có quyền `770`: Owner và Group đều có toàn quyền; Others không có quyền.
* Owner của cả hai thư mục là `devops1`.
* Group của cả hai thư mục là `www-data`.

## 8. Kết luận

Đã hoàn thành việc tạo cấu trúc thư mục `/var/www/my-app` và thiết lập phân quyền theo yêu cầu. Thư mục `public` sử dụng quyền `750`, thư mục `logs` sử dụng quyền `770`, Owner là `devops1` và Group là `www-data`.
