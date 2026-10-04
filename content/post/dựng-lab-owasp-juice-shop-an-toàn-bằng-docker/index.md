---
title: Dựng lab OWASP Juice Shop an toàn bằng Docker
description: Hướng dẫn chạy OWASP Juice Shop trên máy cá nhân bằng Docker Compose, chỉ cho truy cập nội bộ, làm nền tảng cho loạt bài OWASP Top 10.
date: 2026-10-04T22:06:00
categories:
  - OWASP
tags:
  - Juice Shop
  - OWASP Top 10
  - Docker
image: pasted-image-1791126534938.png
draft: false
---

Mình bắt đầu loạt bài về \*\*OWASP Top 10\*\* bằng việc dựng một môi trường để thực hành. 

Bài này hướng dẫn chạy \*\*OWASP Juice Shop\*\*, một ứng dụng web cố tình chứa nhiều lỗ hổng, build bằng Docker trên máy cá nhân, theo cách an toàn để không ảnh hưởng đến máy hay mạng của bạn. 

## Vì sao cần tự dựng lab Các nền tảng như PortSwigger Web Security Academy rất tốt để học cách khai thác từng lỗi, nhưng một số mục trong OWASP Top 10:2025 như cấu hình sai, logging hay xử lý lỗi thì cần môi trường mình tự kiểm soát mới quan sát được. Juice Shop phủ phần lớn các mục Top 10, có bảng điểm và gợi ý, nên rất hợp để bắt đầu. ## Juice Shop là gì Đây là dự án chính thức của OWASP: một cửa hàng nước ép trực tuyến viết bằng Node.js, cố tình chứa hàng chục lỗ hổng từ dễ đến khó. Chính vì cố tình dễ bị tấn công, \*\*tuyệt đối không đưa nó ra Internet\*\*. 

## Chuẩn bị - Máy cài \*\*Docker Desktop\*\* (mình dùng Windows). - Một trình duyệt. Về sau mình sẽ dùng thêm Burp Suite. - Khoảng vài GB dung lượng trống cho image.
