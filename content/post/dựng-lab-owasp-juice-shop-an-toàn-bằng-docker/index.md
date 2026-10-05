---
title: "Dựng lab OWASP Juice Shop an toàn bằng Docker"
description: "Triển khai OWASP Juice Shop trên máy cá nhân bằng Docker Compose, chỉ cho truy cập nội bộ, làm nền tảng thực nghiệm cho loạt bài OWASP Top 10."
slug: "dung-lab-owasp-juice-shop"
date: 2026-10-04T10:00:00+07:00
image: pasted-image-1791126534938.png
categories:
    - OWASP
tags:
    - Juice Shop
    - OWASP Top 10
    - Docker
draft: false
---

> **Tóm tắt.** Bài viết trình bày cách triển khai OWASP Juice Shop, một ứng dụng web cố tình chứa lỗ hổng, bằng Docker Compose trên máy cá nhân. Môi trường được cấu hình để chỉ chấp nhận kết nối từ chính máy chủ (localhost) nhằm tránh phơi bày ứng dụng ra mạng. Bài viết cũng ghi lại một sự cố khởi động thường gặp và thử thách đầu tiên của hệ thống.
>
> **Từ khóa:** OWASP Top 10, Juice Shop, Docker, môi trường thực nghiệm.

## Giới thiệu

Loạt bài này đi theo danh sách OWASP Top 10:2025. Nền tảng như PortSwigger Web Security Academy rất phù hợp để học cách khai thác từng lỗi, nhưng một số mục như cấu hình sai, ghi log và xử lý lỗi cần một môi trường tự kiểm soát để quan sát. Juice Shop phủ phần lớn các mục Top 10 và có bảng điểm kèm gợi ý, nên được chọn làm nền thực nghiệm.

Juice Shop là dự án chính thức của OWASP: một cửa hàng nước ép trực tuyến viết bằng Node.js, cố tình chứa hàng chục lỗ hổng từ dễ đến khó. Vì cố tình dễ bị tấn công, **tuyệt đối không đưa nó ra Internet**.

## Môi trường thực nghiệm

### Yêu cầu

- Docker Desktop (hệ điều hành Windows).
- Một trình duyệt web. Về sau bổ sung Burp Suite.
- Vài GB dung lượng trống cho image.

### Cấu hình

Tập tin `docker-compose.yml` trong thư mục `owasp-lab`:

```yaml
services:
  juice-shop:
    image: bkimminich/juice-shop:latest
    container_name: juice-shop
    ports:
      - "127.0.0.1:3000:3000"
    restart: "no"
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
```

Ba lựa chọn cấu hình đáng chú ý:

| Tùy chọn | Tác dụng |
|---|---|
| `127.0.0.1:3000:3000` | Chỉ mở cổng cho chính máy, không cho mạng nội bộ truy cập |
| `no-new-privileges` | Ngăn tiến trình trong container tự nâng quyền |
| `cap_drop: ALL` | Bỏ các đặc quyền Linux mà ứng dụng không cần |

Nếu viết `3000:3000` thay vì `127.0.0.1:3000:3000`, Docker sẽ mở cổng cho cả mạng Wi-Fi, và bất kỳ ai cùng mạng đều truy cập được một ứng dụng đầy lỗ hổng.

## Triển khai

### Khởi động

```powershell
docker compose up -d
docker ps
```

Sau khi chạy, cột `PORTS` phải hiển thị `127.0.0.1:3000->3000/tcp`. Nếu hiển thị `0.0.0.0:3000` thì cổng đang mở ra ngoài và cần sửa lại tập tin cấu hình.

### Sự cố ERR_EMPTY_RESPONSE

Truy cập `http://localhost:3000` ngay sau khi khởi động, trình duyệt báo lỗi `ERR_EMPTY_RESPONSE` dù container ở trạng thái `Up`. Nguyên nhân là Docker đã mở cổng nhưng ứng dụng bên trong chưa khởi động xong. Chờ khoảng một phút rồi tải lại là được. Có thể kiểm tra bằng:

```powershell
docker logs juice-shop --tail 30
curl.exe -I http://localhost:3000
```

Kết quả `HTTP/1.1 200 OK` cho biết ứng dụng đã sẵn sàng. Trong PowerShell cần gõ `curl.exe`, vì `curl` trơn là bí danh của một lệnh khác.

## Kết quả và thảo luận

Thử thách đầu tiên là tìm trang bảng điểm (Score Board). Trang này tồn tại trong ứng dụng nhưng không có liên kết nào dẫn tới. Đây là mô phỏng của **bảo mật bằng sự che giấu**: ẩn một chức năng khỏi giao diện không bảo vệ được nó, vì mã JavaScript phía client nằm trong tay người dùng. Cách tiếp cận là mở công cụ phát triển của trình duyệt (F12) và đọc mã nguồn phía client.

Về lưu tiến độ, Juice Shop dùng chuỗi **continue code** thay vì lưu dữ liệu bằng Docker volume. Chuỗi này được lưu lại sau mỗi buổi thực hành và nhập lại khi dựng môi trường mới.

## Kết luận

Môi trường Juice Shop chạy trên Docker đã sẵn sàng cho các bài thực hành tiếp theo. Bước kế tiếp là đặt một WAF (ModSecurity với bộ luật OWASP CRS) phía trước ứng dụng để so sánh kết quả khi có và không có lớp bảo vệ này.

Lưu ý an toàn: chỉ thực hành trên môi trường của mình, không mở cổng ra ngoài, và không dùng các kỹ thuật này lên hệ thống thật khi chưa được phép bằng văn bản.

## Tài liệu tham khảo

1. OWASP Juice Shop: https://owasp.org/www-project-juice-shop/
2. OWASP Top 10: https://owasp.org/Top10/
3. Docker Compose: https://docs.docker.com/compose/
