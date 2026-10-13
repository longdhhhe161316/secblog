---
title: 'XSS và CSRF: hai cơ chế khác nhau, một điểm giao nhau'
description: 'Ghi chú hệ thống về Cross-Site Scripting và Cross-Site Request Forgery: cơ chế, phân loại OWASP, các nhầm lẫn phổ biến theo trình độ, và điểm hai lỗ hổng này giao nhau.'
date: 2026-10-12T15:07:00
categories:
  - OWASP
tags:
  - XSS
  - CSRF
  - OWASP Top 10
image: ''
draft: false
---

> **Tóm tắt.** Bài viết hệ thống hóa cơ chế của hai lỗ hổng web thường bị nhầm lẫn — Cross-Site Scripting (XSS) và Cross-Site Request Forgery (CSRF) — bắt đầu từ chính những nhầm lẫn phổ biến khi mới tiếp cận, sau đó làm rõ căn cứ phân loại của OWASP, đối chiếu điểm khác biệt cốt lõi, và trình bày điểm giao nhau giữa hai cơ chế: trường hợp XSS vô hiệu hóa biện pháp chống CSRF.

## Đặt vấn đề

Nhiều tài liệu nhập môn trình bày XSS và CSRF bằng ví dụ minh họa gần giống nhau — một đoạn mã được chèn vào bình luận trên diễn đàn, nạn nhân mở trang và "dính" lỗi. Cách trình bày này dễ khiến người đọc đồng nhất hai lỗ hổng, trong khi chúng khác nhau ở cả vị trí lỗi lẫn cơ chế khai thác.

## Những nhầm lẫn thường gặp khi mới tiếp cận

Trước khi đi vào cơ chế chi tiết, phần này nêu ra các ngộ nhận phổ biến nhất — để người đọc tự kiểm tra hiểu biết hiện tại của mình trước khi đọc tiếp.

**Nhầm lẫn 1: Nghĩ hai lỗ hổng giống nhau vì cùng cần nạn nhân đăng nhập.**
Đúng là cả XSS và CSRF đều khai thác điều kiện nạn nhân đang có phiên đăng nhập hợp lệ, nhưng đó chỉ là điều kiện cần, không phải cơ chế gây ra lỗ hổng. Hai lỗi nằm ở hai vị trí hoàn toàn khác nhau trong ứng dụng (xem mục 5).

**Nhầm lẫn 2: Đồng nhất "chèn được nội dung hiển thị" với "chèn được mã thực thi".**
Đây là nhầm lẫn phổ biến nhất. Một thẻ HTML như `<img src="...">` xuất hiện trong nội dung do người dùng nhập vào **không phải là XSS**, dù nó cũng "chèn được" vào trang. Thẻ này không chứa mã lệnh — nó chỉ là một chỉ dẫn bình thường để trình duyệt tải ảnh, và chính hành vi tải ảnh "vô hại" này lại là thứ bị lợi dụng trong CSRF (giải thích chi tiết ở mục 4 và 5).

**Nhầm lẫn 3: Nhầm tên viết tắt.**
CSRF và XSRF là cùng một khái niệm, chỉ khác tên gọi. Nhưng SSRF (Server-Side Request Forgery) là một lỗ hổng **hoàn toàn khác** — liên quan đến việc server tự gửi request đi nơi khác, không liên quan đến trình duyệt của nạn nhân. Ba chữ viết tắt gần giống nhau này là nguồn nhầm lẫn thường gặp.

**Nhầm lẫn 4: Nghĩ `HttpOnly` chặn được hoàn toàn hậu quả của XSS.**
`HttpOnly` chỉ chặn việc **đọc** cookie bằng JavaScript. Nó không chặn script độc hại **thực hiện hành động** thay người dùng — chi tiết ở mục 3.4.

## Cross-Site Scripting (XSS)

### Cơ chế

XSS phát sinh khi ứng dụng đưa dữ liệu từ người dùng — có thể gọi là **đầu vào**, **input**, **dữ liệu nhập**, hay **payload** (các từ này được dùng thay thế cho nhau trong tài liệu bảo mật) — vào HTML hoặc JavaScript của trang mà không qua xử lý mã hóa (encoding) phù hợp. Hệ quả là trình duyệt diễn giải nhầm dữ liệu đó thành **mã cần thực thi**, thay vì chỉ là nội dung cần hiển thị.

### Phân loại

Ba loại XSS được phân biệt theo **nơi dữ liệu độc hại được đặt vào** (có thể gọi là điểm chèn, điểm tiêm, hay target của cuộc tấn công) và **thời điểm nó được thực thi**:

| Loại | Dữ liệu độc hại nằm ở đâu (điểm chèn / input) | Thời điểm thực thi |
|---|---|---|
| Reflected | Trong tham số của request — ví dụ phần sau dấu `?` trên URL | Server phản chiếu trực tiếp trong response ngay lập tức, không lưu lại |
| Stored | Được lưu trữ lâu dài trên server — ví dụ trong bình luận, hồ sơ cá nhân, tên hiển thị | Mỗi lần dữ liệu đó được hiển thị cho bất kỳ người dùng nào khác |
| DOM-based | Hoàn toàn phía trình duyệt (client), không đi qua server | JavaScript trong trang tự đọc và chèn dữ liệu không an toàn — ví dụ từ phần `#` của URL |

### Vị trí trong OWASP Top 10

XSS thuộc nhóm **Injection** (tạm dịch: "chèn mã độc hại"): A03:2021 theo bản 2021, chuyển thành A05:2025 theo bản cập nhật gần nhất. Căn cứ phân loại là nguyên lý chung giữa các lỗi injection: ranh giới giữa **dữ liệu** và **mã lệnh** bị phá vỡ. SQL injection khiến dữ liệu bị hiểu nhầm thành câu lệnh truy vấn; XSS khiến dữ liệu bị hiểu nhầm thành mã chạy trong trình duyệt.

### Vì sao `HttpOnly` không phải "lá chắn toàn diện"

Thuộc tính `HttpOnly` trên cookie chỉ chặn việc **đọc** giá trị cookie bằng JavaScript (lệnh `document.cookie` sẽ không trả về giá trị cookie có cờ này). Tuy nhiên, nó **không** ngăn một đoạn script độc hại **thực hiện hành động** thay người dùng — vì trình duyệt vẫn tự động đính kèm cookie vào mọi request mà script gửi đi, bất kể script có đọc được giá trị cookie đó hay không.

**Ví dụ minh họa bằng `fetch`:** `fetch` là một hàm JavaScript dùng để **chủ động gửi một request HTTP đi**, tương tự như khi trình duyệt tự gửi request khi bạn bấm vào một nút hay tải một trang — chỉ khác là lần này request được gửi đi **theo lệnh của đoạn mã**, không cần người dùng bấm gì cả. Giả sử một trang có lỗ hổng XSS, và script sau được chạy trong ngữ cảnh trang đó:

```js
fetch('https://vi-du-webapp.com/account/delete-project?id=1', {
  method: 'POST',
  credentials: 'include'
});
```

Lệnh này yêu cầu trình duyệt gửi một request POST tới địa chỉ đã cho. Tham số `credentials: 'include'` chỉ định trình duyệt **đính kèm cookie của trang đó** vào request — kể cả khi cookie có `HttpOnly`, vì đây là hành vi mặc định của trình duyệt khi gửi request tới cùng domain, không phải việc JavaScript tự tay đọc giá trị cookie. Nếu server không có cơ chế xác minh nào khác ngoài cookie, request này được xử lý y như thể chính người dùng chủ động bấm nút xóa.

Đây là lý do `HttpOnly` được xem là biện pháp **giảm thiểu hậu quả**, không phải biện pháp ngăn chặn XSS xảy ra.

### Biện pháp phòng ngừa

| Biện pháp | Nguyên lý |
|---|---|
| Mã hóa đầu ra theo đúng ngữ cảnh (output encoding) | Dữ liệu chèn vào HTML, vào thuộc tính HTML, vào JavaScript, vào URL — mỗi nơi có quy tắc mã hóa khác nhau |
| Content Security Policy (CSP) | Giới hạn nguồn script được phép chạy, giảm thiểu tác động nếu lỗ hổng vẫn tồn tại |
| `HttpOnly` và `SameSite` cho cookie | Không ngăn XSS phát sinh, nhưng giới hạn hậu quả nếu bị khai thác |
| Framework tự động mã hóa (React, Vue) | Giảm rủi ro mặc định; vẫn có điểm yếu nếu dùng các API như `dangerouslySetInnerHTML` hoặc `v-html` không cẩn trọng |

[Liên hệ thực nghiệm: thử thách XSS đã thực hiện trên OWASP Juice Shop — loại nào, trường input nào bị khai thác, điều chỉnh hướng tiếp cận ra sao nếu lần thử đầu không thành công.]

## Cross-Site Request Forgery (CSRF)

### Cơ chế

CSRF phát sinh khi ứng dụng xử lý một hành động thay đổi trạng thái (ví dụ: xóa dữ liệu, đổi mật khẩu, chuyển tiền) chỉ dựa vào việc **có cookie hợp lệ đi kèm**, mà không xác minh request đó có thực sự xuất phát từ chính giao diện của ứng dụng hay không.

Khác với XSS, CSRF **không cần chèn mã thực thi nào cả**. Nó chỉ cần lợi dụng một hành vi hoàn toàn bình thường của trình duyệt: mỗi khi trình duyệt gặp một thẻ cần tải tài nguyên (như `<img>`), nó tự động gửi request tới địa chỉ trong thẻ đó — không cần hỏi ý kiến người dùng, không cần chạy JavaScript.

Quy trình điển hình:
1. Kẻ tấn công đặt một request nhắm tới ứng dụng đích (gọi là **target**, hay đích tấn công) trên một trang bất kỳ mà nạn nhân có khả năng ghé qua — có thể dưới dạng thẻ `<img>` (cho request kiểu GET) hoặc một form ẩn tự động submit (cho request kiểu POST).
2. Nạn nhân, đang trong phiên đăng nhập hợp lệ tại ứng dụng đích, mở trang chứa request đó.
3. Trình duyệt tự động gửi request kèm cookie session, vì cookie được gắn theo tên miền đích, không phụ thuộc vào việc request được khởi tạo từ đâu.
4. Ứng dụng đích không kiểm tra nguồn gốc của request, thực hiện hành động như thể chính nạn nhân yêu cầu.

### Vị trí trong OWASP Top 10

CSRF được gộp vào **A01 — Broken Access Control** (Kiểm soát truy cập bị phá vỡ) từ bản 2021, không còn là mục riêng như ở bản 2017. Căn cứ: CSRF về bản chất là một dạng lỗi kiểm soát truy cập — ứng dụng thực hiện hành động mà không xác minh đúng cách request đó có thực sự được người dùng ủy quyền hay không.

### Điều kiện khai thác

- Ứng dụng đích không triển khai CSRF token (một mã xác thực đi kèm mỗi form) hoặc không kiểm tra header `Origin`/`Referer` (hai header cho biết request đến từ trang nào).
- Cookie session không có thuộc tính `SameSite=Strict` (hoặc `Lax` kết hợp điều kiện cho phép).
- Nạn nhân còn phiên đăng nhập hợp lệ tại thời điểm mở trang chứa request giả mạo.

### Biện pháp phòng ngừa

| Biện pháp | Vị trí triển khai |
|---|---|
| CSRF token (mã xác thực đồng bộ theo mỗi phiên hoặc mỗi form) | Server |
| Kiểm tra header `Origin` / `Referer` | Server |
| Thuộc tính `SameSite` cho cookie | Cấu hình cookie |
| Thiết kế hành động thay đổi trạng thái dùng POST/PUT/DELETE, tránh dùng GET | Thiết kế API |

[Liên hệ thực nghiệm: thử thách CSRF trên Juice Shop, nếu đã thực hiện.]

## Đối chiếu

| | XSS | CSRF |
|---|---|---|
| Vị trí lỗi | Nơi hiển thị nội dung — không mã hóa đầu ra | Nơi xử lý hành động — không xác minh nguồn request |
| Cơ chế khai thác | Chèn mã chạy được (script) vào input | Lợi dụng hành vi tự động của trình duyệt để gửi request, không cần script |
| `HttpOnly` có liên quan | Có — chặn đọc cookie, không chặn hành động | Không — không cần đọc cookie |
| `SameSite` có liên quan | Không trực tiếp | Có — là lớp phòng thủ chính |
| Ví dụ minh họa dễ gây nhầm lẫn | `<script>...</script>` đặt trong một ô nhập liệu | `<img src="...">` đặt trong cùng ô nhập liệu đó |

Sự tương đồng về ví dụ minh họa — cùng là một đoạn mã HTML chèn vào ô input của bình luận — là nguyên nhân chính dẫn đến Nhầm lẫn 2 đã nêu ở mục 2: một thẻ `<img>` không chạy mã, nó chỉ khiến trình duyệt thực hiện một hành vi mặc định của giao thức HTTP (tải tài nguyên); một đoạn script thực thi mới là mã lệnh thật sự trong ngữ cảnh của trang.

## Điểm giao nhau: XSS vô hiệu hóa biện pháp chống CSRF

Dù khác nhau về cơ chế, hai lỗ hổng không hoàn toàn độc lập trong thực tế. Nếu một ứng dụng tồn tại lỗ hổng XSS, kẻ tấn công có khả năng dùng script để **đọc trực tiếp giá trị CSRF token hợp lệ** ngay trên trang (vì script chạy trong cùng ngữ cảnh với trang, có quyền truy cập toàn bộ nội dung trang — gọi là DOM), sau đó gắn token đọc được này vào một request giả mạo do chính script gửi đi bằng `fetch`.

Hệ quả: CSRF token — vốn được xem là biện pháp phòng ngừa chuẩn cho CSRF — không còn là lớp phòng thủ độc lập nếu ứng dụng còn tồn tại XSS. Đây là lý do phòng thủ theo chiều sâu (defense in depth — nhiều lớp bảo vệ chồng lên nhau) được khuyến nghị, thay vì phụ thuộc vào một biện pháp duy nhất.

## Kết luận

[2–3 câu tổng kết dựa trên trải nghiệm thực nghiệm cá nhân, không lặp lại nội dung đã trình bày. Có thể nêu hướng tiếp theo của loạt bài — ví dụ nhóm Injection còn lại (SQL injection, Command injection) hoặc nhóm Broken Access Control nói chung.]

## Tài liệu tham khảo

1. OWASP Top 10: https://owasp.org/Top10/
2. OWASP Cheat Sheet Series — Cross Site Scripting Prevention: https://cheatsheetseries.owasp.org/
3. OWASP Cheat Sheet Series — Cross-Site Request Forgery Prevention: https://cheatsheetseries.owasp.org/
4. OWASP Juice Shop: https://owasp.org/www-project-juice-shop/
5. MDN Web Docs — Fetch API: https://developer.mozilla.org/
