<img width="1429" height="766" alt="image" src="https://github.com/user-attachments/assets/edbb713f-8ae8-4fc6-8274-f9c109e2b10e" />

** Tạo 1 Package WHM **

<img width="896" height="859" alt="image" src="https://github.com/user-attachments/assets/1c43ec38-ab00-4657-9850-48fe7568a4b9" />

<img width="1234" height="388" alt="image" src="https://github.com/user-attachments/assets/f23491ed-f94c-4cc8-b455-6c837ca8bd4c" />

Các thông số trên có ý nghĩa như sau : 

Package Name
Tên định danh nội bộ của gói trong WHM, không hiển thị cho khách (khách thấy tên hiển thị ở trang bán hàng riêng, VD "WP Mini 1"). Không dấu, không khoảng trắng.
Resources
MụcÝ nghĩaDisk Space Quota (MB)Tổng dung lượng ổ đĩa account được dùng — gồm file web + email + database + backup cộng dồn. 2048 = 2GB, khớp "2GB SSD".Monthly Bandwidth Limit (MB)Tổng băng thông ra/vào mỗi tháng. Unlimited khớp yêu cầu.Max FTP AccountsSố FTP account con account có thể tạo thêm (ngoài account chính). Unlimited khớp.Max Email AccountsSố hộp mail tối đa có thể tạo. 20 khớp chính xác.Max Mailing ListsSố mailing list (dạng group email qua Mailman) được tạo. Không nằm trong bảng giá, để Unlimited không ảnh hưởng gì.Max SQL DatabasesSố database MySQL/MariaDB tối đa. 3 khớp chính xác.Max Sub DomainsSố subdomain (dùng chung DNS zone domain chính, VD blog.domain.com). Unlimited khớp.Max Parked DomainsSố domain "Alias" — trỏ y hệt nội dung domain chính, không có thư mục riêng. Unlimited khớp "Alias/Parked Domain: Unlimited".Max Addon DomainsSố domain phụ có nội dung/thư mục riêng biệt — thực chất là thêm 1 website mới trên cùng account. Đây là chỗ cần sửa như đã nói ở trên.Max Passenger ApplicationsSố ứng dụng chạy qua Phusion Passenger (Node.js, Python, Ruby app). Không liên quan gói WordPress, để mặc định cũng không sao.Maximum Hourly Email by Domain RelayedGiới hạn số email gửi đi mỗi giờ tính theo domain — cơ chế chống spam/abuse. Nên set giới hạn thay vì Unlimited.Maximum percentage of failed or deferred messages a domain may send per hourNếu % email gửi thất bại/bị trả về vượt ngưỡng này (100 = không giới hạn), Exim sẽ tạm khóa gửi mail của domain đó — cơ chế phát hiện account bị hack gửi spam hàng loạt.Max Quota per Email Address (MB)Giới hạn dung lượng tối đa của từng hộp mail riêng lẻ. Nên set cụ thể để tránh 1 mailbox ăn hết quota chung.
Settings
MụcÝ nghĩaDedicated IPCấp IP riêng cho account thay vì dùng chung IP server (thường dùng khi cần SSL riêng kiểu cũ hoặc yêu cầu đặc biệt). Không cần cho gói cơ bản.Shell AccessCho phép SSH vào account. Tắt là đúng cho gói shared hosting giá rẻ — tránh khách tự ý can thiệp hệ thống.CGI AccessCho phép chạy script CGI (thư mục cgi-bin). Bật để tương thích các ứng dụng cũ, không ảnh hưởng bảo mật đáng kể.Digest Authentication at account creationBật xác thực Digest (thay vì Basic Auth) cho WebDAV — hiếm dùng, để tắt là bình thường.cPanel ThemeGiao diện cPanel hiển thị cho khách. Jupiter là theme mặc định hiện tại, ổn.Feature ListDanh sách bật/tắt các icon/tính năng hiển thị trong cPanel (VD ẩn Shell Access icon, ẩn Addon Domain icon...). Nên tạo Feature List riêng cho gói này thay vì dùng "default" — để ẩn hẳn icon Addon Domain trên giao diện khách, tránh khách thắc mắc tại sao bấm không được dù limit=0.LocaleNgôn ngữ mặc định hiển thị cPanel khi tạo account mới. Vietnamese khớp nhu cầu khách Việt.
Package Extensions — WP Toolkit
Bật plugin WP Toolkit (nếu server có license/cài đặt) → cho phép khách cài WordPress 1-click, quản lý update/backup WP ngay trong cPanel. Đã check đúng, khớp với "Cài đặt: 1-Click WordPress Installation" trong bảng giá.

ở đây, sẽ thử táo các package như các gói wordpress của Nhân Hòa

<img width="1498" height="881" alt="image" src="https://github.com/user-attachments/assets/986cfb3d-add5-46ed-984b-790c3c5b5c27" />

Tạo người dùng, ADmin, Reseller 

User:

<img width="1920" height="1034" alt="image" src="https://github.com/user-attachments/assets/e53a744d-5ed2-426f-867c-a2846b0227fb" />

<img width="1920" height="1034" alt="image" src="https://github.com/user-attachments/assets/6eee843a-3c8a-4655-8748-e752c63137ea" />

<img width="774" height="576" alt="image" src="https://github.com/user-attachments/assets/60e13bb4-d58e-4b15-b970-45e58ba308c3" />

1. Tạo User (cPanel Account) — khách hàng cuối
# Tạo Tài Khoản Mới (WHM) — Cài Đặt Định Tuyến Thư và Cài Đặt DNS

## 1. Cài Đặt Định Tuyến Thư (Mail Routing)

Mục này quyết định server có tự xử lý mail đến cho domain hay không, hoặc chuyển tiếp mail đi đâu. Cấu hình sai tại đây là nguyên nhân phổ biến gây mất email hoặc mail bị dội ngược (bounce).

### Tự Động Phát Hiện Cấu Hình (Automatically Detect Configuration)

Server tự kiểm tra bản ghi MX của domain để quyết định chế độ phù hợp:

- Nếu MX trỏ về chính server này, hệ thống tự chọn Local Mail Exchanger.
- Nếu MX trỏ ra dịch vụ email bên ngoài, hệ thống tự chọn Remote Mail Exchanger.

Đây là lựa chọn an toàn nhất khi tạo tài khoản mới, tránh trường hợp cấu hình tay bị bỏ sót khi khách đổi nhà cung cấp email sau này.

### Bộ Trao Đổi Thư Cục Bộ (Local Mail Exchanger)

Server được cấu hình luôn chấp nhận thư và xử lý mail nội bộ trên chính máy chủ, dù thư gửi từ trong hay ngoài server. Đây là lựa chọn đúng khi:

- Domain sử dụng hộp mail dạng tên_miền được tạo trực tiếp trong cPanel (công cụ Email Accounts).
- Domain không sử dụng dịch vụ email ngoài như Google Workspace hoặc Microsoft 365.

Nếu domain đang dùng Google Workspace nhưng vẫn để chế độ này, sẽ xảy ra xung đột: mail vừa cố gắng vào hộp thư nội bộ trên server, vừa có bản ghi MX trỏ ra Google, dẫn đến thất lạc thư hoặc mail vào sai nơi.

### Sao Lưu Bộ Trao Đổi Thư (Backup Mail Exchanger)

Server đóng vai trò MX dự phòng, nhận và giữ tạm mail khi mail server chính đang gián đoạn, sau đó chuyển tiếp lại khi mail server chính hoạt động trở lại. Ít dùng trong mô hình shared hosting thông thường, chủ yếu áp dụng cho hạ tầng mail có kiến trúc phức tạp.

### Bộ Trao Đổi Thư Từ Xa (Remote Mail Exchanger)

Áp dụng khi domain sử dụng dịch vụ email bên ngoài (Google Workspace, Microsoft 365, Zoho Mail...). Khi chọn chế độ này, server không cố nhận mail cho domain nữa; các hộp mail tạo trong cPanel (nếu có) sẽ không còn hoạt động. Đây là cấu hình bắt buộc phải chuyển sang khi domain dùng dịch vụ email ngoài, để tránh mail bị kẹt do cố gắng gửi vào hộp thư nội bộ trong khi MX thực tế trỏ ra ngoài.

### Mail Child Node

Áp dụng trong kiến trúc cluster, khi mail server được tách riêng thành một node độc lập khỏi web server. Không áp dụng cho mô hình một server duy nhất thông thường.

### Bảng lựa chọn theo tình huống thực tế

| Tình huống khách hàng | Chế độ nên chọn |
|---|---|
| Dùng hộp mail tên_miền tạo trong cPanel | Local Mail Exchanger (hoặc Automatically Detect) |
| Dùng Google Workspace / Microsoft 365 | Remote Mail Exchanger |
| Chưa xác định, để hệ thống tự quyết theo MX record | Automatically Detect Configuration |
| Không sử dụng email theo domain này | Local Mail Exchanger, không ảnh hưởng vì không phát sinh hộp thư |

Khuyến nghị khi tạo tài khoản mới: nên chọn Automatically Detect Configuration làm mặc định thay vì cố định Local, nhằm tránh phải chỉnh tay khi khách chuyển sang dùng dịch vụ email ngoài sau này.

## 2. Cài Đặt DNS

Mục này quyết định các bản ghi DNS được sinh tự động khi tạo tài khoản, chủ yếu phục vụ bảo mật email (chống giả mạo gửi thư) và quyền quản lý vùng DNS (DNS zone).

### Kích hoạt DKIM trên tài khoản này

DKIM (DomainKeys Identified Mail) giúp server ký điện tử vào mỗi email gửi đi bằng khóa riêng (private key); bên nhận xác thực chữ ký qua khóa công khai (public key) nằm trong bản ghi TXT của DNS. Mục đích là chứng minh email thực sự được gửi từ server này và không bị chỉnh sửa trên đường truyền. Nếu tắt, email gửi đi dễ bị các dịch vụ như Gmail, Outlook đánh dấu là thư rác.

### Kích hoạt SPF trên tài khoản này

SPF (Sender Policy Framework) là bản ghi TXT khai báo những server được phép gửi mail thay mặt domain. Giá trị mặc định được sinh ra:

```
v=spf1 +a +mx +ip4:103.170.123.59 ~all
```

Ý nghĩa từng thành phần:

- `+a`: cho phép server có bản ghi A trùng domain được gửi mail.
- `+mx`: cho phép các server nằm trong bản ghi MX được gửi mail.
- `+ip4:103.170.123.59`: cho phép rõ địa chỉ IP này gửi mail.
- `~all`: chế độ SoftFail, mail từ nguồn không nằm trong danh sách trên vẫn được nhận nhưng bị đánh dấu nghi ngờ, khác với `-all` (HardFail) là từ chối thẳng.

Nếu khách hàng sử dụng thêm dịch vụ gửi mail bên ngoài (ví dụ nền tảng email marketing hoặc CRM), cần bổ sung thủ công IP hoặc domain của dịch vụ đó vào bản ghi SPF sau khi tạo tài khoản, nếu không mail gửi từ các dịch vụ này dễ bị đánh dấu spam.

### Enable DMARC on this account

DMARC dựa trên kết quả xác thực của cả SPF và DKIM để quyết định cách xử lý mail giả mạo domain. Giá trị mặc định:

```
v=DMARC1; p=none;
```

Tham số `p=none` nghĩa là chỉ giám sát, không chặn: mail không đạt SPF hoặc DKIM vẫn được nhận bình thường, DMARC chỉ ghi nhận để báo cáo. Đây là mức khởi điểm an toàn, tránh trường hợp domain mới cấu hình DMARC nghiêm ngặt khiến mail hợp lệ bị chặn nhầm do SPF/DKIM chưa ổn định.

Với khách hàng cần mức bảo mật cao hơn nhằm chống giả mạo domain phục vụ lừa đảo, có thể tư vấn nâng dần theo lộ trình: `p=none` sang `p=quarantine` (đưa vào thư rác) rồi `p=reject` (chặn hẳn), sau khi đã xác nhận SPF và DKIM hoạt động ổn định trong thời gian đủ dài.

### Dùng máy chủ cấp tên đã xác định tại Tổ Chức Đăng Ký Tên Miền

Khi tick tùy chọn này, server sẽ bỏ qua vùng DNS cục bộ và để domain hoạt động theo đúng nameserver mà khách đã khai báo tại nhà đăng ký domain. Áp dụng khi domain không sử dụng nameserver của server này để quản lý DNS, ví dụ trường hợp dùng Cloudflare DNS hoặc DNS do registrar cung cấp.

Khi không tick (mặc định), domain sẽ sử dụng vùng DNS được tạo cục bộ ngay trên server, theo hai nameserver hiển thị bên dưới.

### Ghi đè mọi vùng DNS hiện có cho tài khoản này

Nếu domain đã tồn tại vùng DNS từ trước trên server (trường hợp tạo lại tài khoản, hoặc domain từng thuộc tài khoản khác), tick tùy chọn này sẽ xóa và tạo lại toàn bộ vùng DNS mới. Mặc định nên để trống; chỉ tick khi chủ động muốn reset lại vùng DNS cũ, vì tick nhầm sẽ làm mất toàn bộ bản ghi tùy chỉnh đã có từ trước như MX ngoài hoặc TXT xác minh dịch vụ bên thứ ba.

### Máy Chủ Cấp Tên (Nameservers)

```
ns1.tdong41.id.vn
ns2.tdong41.id.vn
```

Đây là hai nameserver riêng của server. Khách hàng cần trỏ domain về hai nameserver này tại nơi đăng ký domain nếu muốn toàn bộ DNS được quản lý qua server này. Nếu khách chỉ trỏ bản ghi A mà không đổi nameserver, vùng DNS được tạo trên WHM sẽ không có tác dụng thực tế; khi đó việc quản lý DNS phải thực hiện tại registrar hoặc Cloudflare.

### Bảng lưu ý khi triển khai cho khách hàng

| Tình huống | Xử lý đề xuất |
|---|---|
| Khách dùng nameserver riêng (Cloudflare, registrar DNS) | Cài Đặt DNS trong WHM không có tác dụng thực tế, vùng DNS phải quản lý bên ngoài |
| Khách trỏ hẳn nameserver về server này | Giữ nguyên cấu hình mặc định (DKIM, SPF, DMARC đều bật) |
| Khách sử dụng thêm dịch vụ gửi mail ngoài | Sau khi tạo tài khoản, vào Zone Editor bổ sung thủ công vào bản ghi SPF |
| Khách yêu cầu mức bảo mật domain cao | Sau thời gian vận hành ổn định, nâng dần DMARC từ p=none lên p=quarantine hoặc p=reject |
