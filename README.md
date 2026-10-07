# Bài 3 — MySQL và Systemd cho Spring Boot

## Tạo database và user MySQL

Mở MySQL bằng tài khoản quản trị và thực hiện:

```sql
CREATE DATABASE springboot_db CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci;
CREATE USER 'spring-admin'@'localhost' IDENTIFIED BY '<MAT_KHAU_THEO_DE_BAI>';
GRANT ALL PRIVILEGES ON springboot_db.* TO 'spring-admin'@'localhost';
FLUSH PRIVILEGES;
```

Thay placeholder bằng mật khẩu được yêu cầu trong đề khi thao tác trực tiếp trên VPS. Không đưa mật khẩu thật vào repository công khai.

## Cài ứng dụng và cấu hình môi trường

```bash
sudo useradd --system --user-group --no-create-home --shell /usr/sbin/nologin spring-runner
sudo install -d -o spring-runner -g spring-runner -m 0750 /opt/spring-app
sudo install -o spring-runner -g spring-runner -m 0640 ./app.jar /opt/spring-app/app.jar
sudo install -d -o root -g root -m 0750 /etc/spring-app
sudo install -o root -g root -m 0600 spring-app.env.example /etc/spring-app/spring-app.env
sudoedit /etc/spring-app/spring-app.env
```

Trong `/etc/spring-app/spring-app.env`, thay giá trị placeholder bằng mật khẩu database theo đề bài. Tệp này chỉ được đọc bởi root trong lúc systemd nạp cấu hình, và không được commit.

## Cài đặt service

```bash
sudo install -o root -g root -m 0644 spring-app.service /etc/systemd/system/spring-app.service
sudo systemctl daemon-reload
sudo systemctl enable --now spring-app.service
sudo systemctl status spring-app.service --no-pager
sudo ss -tlnp | grep ':8082'
```

Service chạy bằng `spring-runner`, gọi đúng `/usr/bin/java -jar /opt/spring-app/app.jar`, đặt cổng `8082` và tự khởi động lại sau 10 giây nếu thoát do lỗi. `app.jar` cần có Spring Boot MySQL datasource/driver phù hợp với các biến môi trường trên.

## Kết quả cần xác nhận

`systemctl status` cần hiển thị `active (running)`. `ss` cần hiển thị Java lắng nghe trên `8082`; quyền tiến trình là `spring-runner`. Nếu service lỗi, xem `sudo journalctl -u spring-app.service -n 100 --no-pager`.
