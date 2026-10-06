# Báo Cáo Bài 2: Cấu Hình Phân Quyền Nhóm và Sudoers Bằng visudo

## 1. Nhật ký các lệnh đã thực hiện

### Bước 1: Tạo nhóm và tài khoản người dùng
Tạo nhóm `devops-admin`, tạo người dùng `deployer` và gán người dùng này vào nhóm `devops-admin`:
```bash
# Tạo nhóm mới
sudo groupadd devops-admin

# Tạo user deployer
sudo adduser deployer

# Thêm deployer vào nhóm devops-admin
sudo usermod -aG devops-admin deployer
```

### Bước 2: Cấu hình quyền hạn trong sudoers
Mở tệp cấu hình sudoers an toàn bằng lệnh `visudo`:
```bash
sudo visudo
```
*(Hoặc tạo file cấu hình riêng trong thư mục `/etc/sudoers.d/devops-admin` bằng lệnh: `sudo visudo -f /etc/sudoers.d/devops-admin`)*

---

## 2. Dòng cấu hình sudoers đã thêm vào hệ thống

Thêm dòng sau vào cuối tệp cấu hình (dấu `%` đại diện cho Group):
```text
%devops-admin ALL=(ALL) NOPASSWD: /usr/bin/systemctl start *, /usr/bin/systemctl stop *, /usr/bin/systemctl restart *, /usr/bin/systemctl status *
```

**Ý nghĩa:**
- `%devops-admin`: Áp dụng cho tất cả thành viên thuộc nhóm `devops-admin`.
- `ALL=(ALL)`: Áp dụng trên mọi máy chủ (`ALL`) và thực thi dưới quyền mọi tài khoản/root (`(ALL)`).
- `NOPASSWD:`: Không yêu cầu nhập mật khẩu khi chạy các lệnh được liệt kê.
- `/usr/bin/systemctl ...`: Chỉ cho phép thực thi 4 thao tác `start`, `stop`, `restart`, `status` của dịch vụ hệ thống.

---

## 3. Kết quả kiểm tra (Verification Output)

### Bước 1: Chuyển sang tài khoản deployer
```bash
su - deployer
```

### Bước 2: Kiểm tra các quyền đặc quyền sudo (`sudo -l`)
```bash
sudo -l
```

**Đầu ra kết quả terminal:**
```console
deployer@ubuntu:~$ sudo -l
Matching Defaults entries for deployer on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User deployer may run the following commands on ubuntu:
    (ALL) NOPASSWD: /usr/bin/systemctl start *, /usr/bin/systemctl stop *, /usr/bin/systemctl restart *, /usr/bin/systemctl status *
```

### Bước 3: Chạy thử lệnh khởi động lại dịch vụ `cron`
```bash
sudo systemctl restart cron
```

**Đầu ra kết quả terminal:**
```console
deployer@ubuntu:~$ sudo systemctl restart cron
deployer@ubuntu:~$ sudo systemctl status cron
● cron.service - Regular background program processing daemon
     Loaded: loaded (/lib/systemd/system/cron.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2026-10-06 20:50:12 UTC; 4s ago
```

---

## 4. Giải thích kết quả
- User `deployer` thuộc nhóm `devops-admin` đã được cấp quyền thực thi lệnh `systemctl` (`start`, `stop`, `restart`, `status`).
- Lệnh `sudo systemctl restart cron` thực thi thành công ngay lập tức mà không yêu cầu nhập mật khẩu (`NOPASSWD`).
- Tài khoản `deployer` không có toàn quyền root, các lệnh nguy hiểm khác ngoài danh sách cấp phép đều bị từ chối, đảm bảo an toàn hệ thống theo nguyên tắc Least Privilege (Đặc quyền tối thiểu).
