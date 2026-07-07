# Báo Cáo: Cơ Chế Và Cấu Hình Chặn Gửi/Nhận Email Trong Exim (DirectAdmin)

**Người thực hiện:** Nguyễn Tiến Đông
**Môi trường:** DirectAdmin CustomBuild — Exim + Dovecot
**Ngày:** Tháng 7/2026

---

## Mục Lục

1. [Mục đích](#1-mục-đích)
2. [Kiến trúc tổng quan](#2-kiến-trúc-tổng-quan)
3. [Cơ chế hoạt động của Exim ACL](#3-cơ-chế-hoạt-động-của-exim-acl)
4. [Vai trò của Dovecot trong SMTP-AUTH](#4-vai-trò-của-dovecot-trong-smtp-auth)
5. [Phân loại 4 tình huống chặn](#5-phân-loại-4-tình-huống-chặn)
6. [Cài đặt và cấu hình chi tiết](#6-cài-đặt-và-cấu-hình-chi-tiết)
7. [Kiểm tra sau khi cấu hình](#7-kiểm-tra-sau-khi-cấu-hình)
8. [Các lỗi thường gặp và cách xử lý](#8-các-lỗi-thường-gặp-và-cách-xử-lý)
9. [Tổng kết](#9-tổng-kết)

---

## 1. Mục Đích

Tài liệu này ghi lại cơ chế hoạt động và các bước cấu hình để:

- Chặn **1 tài khoản email nội bộ** (nhân viên) gửi và/hoặc nhận thư.
- Chặn **1 địa chỉ email bên ngoài** gửi vào hệ thống, hoặc chặn nhân viên nội bộ gửi ra cho địa chỉ đó.

Mục tiêu là hiểu rõ **vì sao** một cấu hình hoạt động, không chỉ chép lệnh, để có thể tự điều chỉnh khi gặp tình huống khác.

---

## 2. Kiến Trúc Tổng Quan

Trong DirectAdmin, việc gửi/nhận mail liên quan đến 2 dịch vụ chính, phối hợp qua 1 database mật khẩu chung:

```
                    ┌─────────────────────────────┐
                    │   /etc/virtual/<domain>/     │
                    │         passwd (mã hoá)      │
                    └───────────┬─────────┬────────┘
                                │         │
                    tra cứu khi│         │tra cứu khi
                    IMAP/POP3  │         │SMTP-AUTH
                                │         │
                    ┌───────────▼──┐   ┌──▼────────────┐
                    │   Dovecot     │   │     Exim      │
                    │ (nhận mail về)│   │ (gửi/nhận mail)│
                    └───────────────┘   └───────────────┘
```

- **Dovecot**: phục vụ IMAP/POP3 để client (Outlook, webmail...) **lấy mail về**, đồng thời đóng vai trò **backend xác thực (SASL)** cho Exim.
- **Exim**: xử lý toàn bộ luồng SMTP — nhận mail từ ngoài vào, và nhận mail nhân viên gửi ra ngoài. Khi cần xác thực (SMTP-AUTH), Exim hỏi ngược lại Dovecot qua socket nội bộ.

---

## 3. Cơ Chế Hoạt Động Của Exim ACL

Exim xử lý một phiên SMTP qua các giai đoạn tuần tự, mỗi giai đoạn được kiểm soát bởi 1 khối ACL (Access Control List) riêng trong `exim.conf`:

| Giai đoạn SMTP | ACL tương ứng | Thời điểm chạy |
|---|---|---|
| Kết nối TCP | `acl_smtp_connect` | Ngay khi client kết nối |
| Lệnh `HELO`/`EHLO` | `acl_check_helo` | Client tự giới thiệu hostname |
| Lệnh `AUTH` (nếu có) | `acl_check_auth` | Client đăng nhập username/password |
| Lệnh `MAIL FROM` | `acl_smtp_mail` | Client khai báo người gửi |
| Lệnh `RCPT TO` | `acl_check_recipient` | Client khai báo người nhận |
| Lệnh `DATA` | `acl_check_message` / `acl_check_data` | Client gửi nội dung thư |

**Nguyên tắc chạy ACL:** mỗi ACL gồm nhiều rule, chạy tuần tự từ trên xuống dưới:

- `accept` — nếu điều kiện khớp, dừng ACL ngay, cho phép đi tiếp.
- `deny` — nếu điều kiện khớp, từ chối và trả mã lỗi SMTP (VD: `550`), nhưng cho phép client gửi lệnh `QUIT` sạch sẽ.
- `drop` — giống `deny` nhưng ngắt kết nối TCP ngay lập tức, không cho gửi thêm lệnh nào.
- Nếu không rule nào khớp, ACL mặc định `deny` ở cuối (trừ khi có rule accept cuối cùng).

Việc chặn gửi/nhận 1 email **chủ yếu can thiệp vào `acl_check_recipient`**, vì đây là nơi Exim biết chính xác ai đang nhận thư — trước cả khi nội dung `DATA` được truyền, giúp tiết kiệm tài nguyên khi từ chối sớm.

### Hook mở rộng (.include_if_exists)

DirectAdmin thiết kế sẵn các "điểm neo" trong `exim.conf` dạng:

```
.include_if_exists /etc/exim.acl_check_recipient.pre.conf
```

File này được Exim nạp **nếu tồn tại**, chạy **ngay tại vị trí include** — tức trước toàn bộ rule mặc định phía sau nó. Đây là cách chuẩn để thêm rule tùy chỉnh mà:

- Không cần sửa `exim.conf` gốc (tránh bị `./build exim_conf` ghi đè khi DirectAdmin update).
- Không cần chạy lại `./build exim_conf` sau khi sửa — chỉ cần `service exim restart`.

---

## 4. Vai Trò Của Dovecot Trong SMTP-AUTH

Bản thân Exim **không tự kiểm tra mật khẩu**. Khi client gửi lệnh `AUTH LOGIN`/`AUTH PLAIN` kèm username/password, Exim chuyển tiếp thông tin đó cho Dovecot qua 1 socket nội bộ (auth socket), Dovecot tra trong file `passwd` của domain tương ứng và trả lời đúng/sai.

Nếu xác thực thành công, Exim gán biến `$authenticated_id` = địa chỉ email vừa đăng nhập, và biến này tồn tại **suốt phiên kết nối đó**. Đây là cơ sở để phân biệt:

- **Mail không xác thực** — đến từ mail server khác trên Internet, `$authenticated_id` không tồn tại.
- **Mail đã xác thực** — nhân viên gửi qua Outlook/webmail dùng đúng user/pass, `$authenticated_id` = email nhân viên.

> ⚠️ Lưu ý thực tế: một số webmail (VD: cấu hình gửi qua `sendmail`/socket cục bộ thay vì kết nối SMTP thật) có thể **không đi qua bước AUTH**, khiến `$authenticated_id` không được set — làm các rule dựa vào `authenticated = *` không phát huy tác dụng. Cần kiểm tra log để xác nhận trước khi kết luận rule "không hoạt động".

---

## 5. Phân Loại 4 Tình Huống Chặn

| # | Tình huống | Cơ chế đúng | File/vị trí |
|---|---|---|---|
| 1 | Chặn **hoàn toàn** 1 mailbox nội bộ (cả gửi lẫn nhận) | Suspend account qua DirectAdmin | User Level → E-mail Accounts → Suspend |
| 2 | Chặn **gửi** của 1 mailbox nội bộ, vẫn cho nhận | Blacklist theo `$authenticated_id` | `/etc/virtual/blacklist_smtp_usernames` |
| 3 | Chặn **nhận** — không ai gửi được đến 1 địa chỉ (nội bộ hoặc để bảo vệ khỏi spam từ 1 nguồn cụ thể) | ACL `deny recipients =` trong `acl_check_recipient` | Custom ACL (`.pre.conf`) |
| 4 | Chặn **1 địa chỉ ngoài** gửi vào hệ thống bạn | Blacklist theo `MAIL FROM` (Return-Path) | `/etc/virtual/blacklist_senders` |

Bảng trên là điểm mấu chốt: **không có 1 rule duy nhất giải quyết mọi tình huống** — mỗi loại chặn cần đúng cơ chế tương ứng với giai đoạn SMTP mà nó diễn ra.

---

## 6. Cài Đặt Và Cấu Hình Chi Tiết

### 6.1. Tình huống 1 — Chặn hoàn toàn 1 nhân viên (khuyến nghị, đơn giản nhất)

Vào **DirectAdmin → User Level → E-mail Accounts**, chọn tài khoản, bấm **Suspend**.

Cơ chế: khoá truy cập vào file `passwd` của mailbox đó → Dovecot từ chối cả IMAP/POP3 (nhận) lẫn SMTP-AUTH (gửi), vì cả hai đều tra cùng 1 nguồn xác thực.

### 6.2. Tình huống 2 — Chặn gửi của 1 nhân viên, vẫn giữ nhận được mail

```bash
echo "nhanvien@domain.com" >> /etc/virtual/blacklist_smtp_usernames
service exim restart
```

Cơ chế: rule có sẵn trong `exim.conf` mặc định của DirectAdmin tra cứu `$authenticated_id` trong file này ngay khi AUTH thành công, nếu khớp thì `drop` kết nối trước khi cho gửi bất kỳ thư nào.

### 6.3. Tình huống 3 — Chặn nhận đến 1 địa chỉ cụ thể (áp dụng cho mọi người gửi, kể cả nội bộ)

```bash
nano /etc/exim.acl_check_recipient.pre.conf
```

Nội dung:

```
deny
    message     = 550 Mailbox unavailable
    recipients  = nhanvien@domain.com
```

```bash
exim -bV
service exim restart
```

### 6.4. Tình huống 4 — Chặn 1 địa chỉ ngoài gửi vào hệ thống

```bash
echo "external@domain.com" >> /etc/virtual/blacklist_senders
service exim restart
```

Cơ chế: rule mặc định trong `acl_check_recipient` có `senders = +blacklist_senders`, so khớp giá trị `MAIL FROM` của kết nối đến — áp dụng cho mail **chưa authenticated** (tức từ ngoài Internet).

### 6.5. Tình huống 4b — Chặn nhân viên nội bộ gửi ra cho 1 địa chỉ ngoài cụ thể

```bash
nano /etc/exim.acl_check_recipient.pre.conf
```

Thêm:

```
deny
    message       = 550 Sending to this recipient is not permitted
    recipients    = external@domain.com
    authenticated = *
```

```bash
exim -bV
service exim restart
```

Điều kiện `authenticated = *` đảm bảo rule chỉ áp dụng cho phiên đã đăng nhập SMTP-AUTH (nhân viên gửi ra), không ảnh hưởng đến việc nhận mail bình thường.

---

## 7. Kiểm Tra Sau Khi Cấu Hình

```bash
# Kiểm tra cú pháp exim.conf không lỗi trước khi restart
exim -bV

# Theo dõi log thời gian thực khi test gửi/nhận thử
tail -f /var/log/exim/mainlog

# Kiểm tra routing của 1 địa chỉ
exim -bt user@domain.com
```

Khi test, tìm trong log các trường:
- `authenticated_id=...` — xác nhận phiên đã qua SMTP-AUTH.
- `A=dovecot_plain:` hoặc tương tự — xác nhận cơ chế xác thực nào được dùng.
- Dòng `<= ` (nhận) hoặc `=> ` (gửi thành công) hoặc thông báo lỗi 550 — xác nhận rule có chạy đúng không.

---

## 8. Các Lỗi Thường Gặp Và Cách Xử Lý

| Hiện tượng | Nguyên nhân khả dĩ | Cách xử lý |
|---|---|---|
| Đã thêm vào `blacklist_senders` nhưng nhân viên vẫn gửi được | File này chỉ chặn mail **chưa authenticated**, không áp dụng cho người đã login | Dùng `blacklist_smtp_usernames` thay vì `blacklist_senders` |
| Rule `authenticated = *` không có tác dụng dù nhân viên gửi qua webmail | Webmail có thể gửi qua socket nội bộ, bỏ qua bước AUTH SMTP thật sự | Kiểm tra log xem có `authenticated_id` không; nếu không, cần chặn ở tầng khác (VD: chặn trong cấu hình webmail hoặc router local delivery) |
| Sửa `exim.conf` xong bị mất sau khi update DirectAdmin | Sửa trực tiếp file gốc thay vì dùng custom template hoặc `.pre.conf` | Luôn dùng `/usr/local/directadmin/custombuild/custom/exim.conf` (cần `./build exim_conf`) hoặc các file `.pre.conf` (không cần build lại) |
| Exim không khởi động lại được sau khi sửa | Lỗi cú pháp ACL (thiếu thụt lề, thiếu dòng trống giữa các rule) | Chạy `exim -bV` trước khi restart để bắt lỗi cú pháp |

---

## 9. Tổng Kết

- Việc chặn gửi/nhận 1 email trong Exim **không phải một cơ chế duy nhất**, mà tùy thuộc: đối tượng là ai (nội bộ hay bên ngoài), và chiều nào (gửi hay nhận).
- Mấu chốt kỹ thuật là hiểu rõ 2 khái niệm: **giai đoạn ACL** (`acl_check_recipient` là nơi phần lớn việc chặn diễn ra) và **`$authenticated_id`** (do Dovecot xác thực, dùng để phân biệt mail nội bộ đã login với mail từ ngoài).
- Với mailbox nội bộ, cách an toàn và đơn giản nhất vẫn là **Suspend** qua giao diện DirectAdmin; chỉ nên dùng cấu hình Exim thủ công khi cần độ chi tiết cao hơn (chặn 1 chiều, chặn theo địa chỉ ngoài, hoặc không thể xoá/suspend tài khoản mặc định của hệ thống).
