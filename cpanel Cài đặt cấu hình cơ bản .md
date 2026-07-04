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



