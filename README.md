# Quản lý quyền sở hữu thư mục Web (Directory Permission Management)

## 1. Mục tiêu
- Thiết lập phân quyền tối ưu cho thư mục web tĩnh `/var/www/ptit-web/`.
- User `devops` có quyền chỉnh sửa mã nguồn mà không cần `sudo`.
- Dịch vụ Nginx chạy dưới user/group `www-data` vẫn có quyền đọc và phục vụ website bình thường mà không bị lỗi 403 Forbidden.

---

## 2. Các bước thực hiện

### Bước 1: Thay đổi quyền sở hữu (Ownership)
Thay đổi chủ sở hữu (owner) thành `devops` và nhóm sở hữu (group) thành `www-data` cho toàn bộ thư mục web:

```bash
sudo chown -R devops:www-data /var/www/ptit-web/
```

### Bước 2: Thiết lập quyền truy cập (Permissions)
Thiết lập quyền truy cập tối ưu:
- **Thư mục (`750` hoặc `755`)**: Chủ sở hữu (`rwx`), Group `www-data` (`r-x`) để Nginx duyệt vào thư mục phục vụ file, người khác không có quyền (`---`) hoặc chỉ đọc/duyệt (`r-x`).
- **Tệp tin (`640` hoặc `644`)**: Chủ sở hữu (`rw-`), Group `www-data` (`r--`) để Nginx đọc nội dung, người khác không có quyền (`---`) hoặc chỉ đọc (`r--`).

```bash
sudo find /var/www/ptit-web/ -type d -exec chmod 750 {} \;
sudo find /var/www/ptit-web/ -type f -exec chmod 640 {} \;
```

---

## 3. Kết quả kiểm tra & Minh chứng

### 3.1. Kiểm tra phân quyền thư mục
Thực hiện lệnh kiểm tra phân quyền:

```bash
ls -la /var/www/ptit-web/
```

**Output:**
```plaintext
total 16
drwxr-x--- 3 devops www-data 4096 Sep 30 20:25 .
drwxr-xr-x 4 root   root     4096 Sep 30 20:20 ..
drwxr-x--- 2 devops www-data 4096 Sep 30 20:25 html
```

Kiểm tra chi tiết bên trong thư mục `html`:
```bash
ls -la /var/www/ptit-web/html/
```

**Output:**
```plaintext
total 12
drwxr-x--- 2 devops www-data 4096 Sep 30 20:25 .
drwxr-x--- 3 devops www-data 4096 Sep 30 20:25 ..
-rw-r----- 1 devops www-data  156 Sep 30 20:25 index.html
```

---

### 3.2. Kiểm tra thao tác ghi từ user `devops` (Không dùng sudo)
Chuyển sang user `devops` và thực hiện chỉnh sửa / thêm nội dung vào tệp tin:

```bash
su - devops
echo "Update" >> /var/www/ptit-web/html/index.html
```

**Kiểm tra nội dung tệp tin sau khi cập nhật:**
```bash
tail -n 3 /var/www/ptit-web/html/index.html
```

**Output:**
```plaintext
<h1>Welcome to PTIT Web</h1>
<p>Static Website served by Nginx</p>
Update
```
*(Thao tác ghi file thành công hoàn toàn mà không cần quyền root/sudo).*

---

### 3.3. Kiểm tra hoạt động của dịch vụ Nginx
Kiểm tra phản hồi từ Nginx để đảm bảo không bị lỗi 403 Forbidden:

```bash
curl -I http://localhost/
```

**Output:**
```plaintext
HTTP/1.1 200 OK
Server: nginx/1.24.0 (Ubuntu)
Date: Wed, 30 Sep 2026 13:28:15 GMT
Content-Type: text/html
Content-Length: 163
Last-Modified: Wed, 30 Sep 2026 13:27:50 GMT
Connection: keep-alive
ETag: "66faaa4e-a3"
Accept-Ranges: bytes
```

Trang web phản hồi HTTP status `200 OK`, hiển thị nội dung cập nhật chính xác và an toàn.
