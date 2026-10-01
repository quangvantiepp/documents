Dưới đây là bản thiết kế kỹ thuật chi tiết, bao gồm **Lựa chọn Công nghệ** và **Danh sách Functions (Hàm/API/Chức năng chi tiết)** để triển khai trọn gói dự án theo đúng ngân sách, tối ưu cho 1.000 user, chạy trên nền tảng **Vercel + Render + Tên miền .VN** (~6 triệu/năm tiền hạ tầng).

---

## PHẦN 1: LỰA CHỌN CÔNG NGHỆ (TECH STACK)

Để đảm bảo hiệu năng cao, tối ưu chi phí vận hành (~6 triệu/năm) và chuẩn Responsive (Mobile & Desktop):

* **Frontend (Giao diện người dùng & Admin):**
* **Framework:** **Next.js (React)** hoặc **Vite + React**.
* *Lý do:* Next.js hỗ trợ SSR/SSG cực tốt cho SEO và tốc độ tải trang trên Mobile, deploy hoàn toàn miễn phí hoặc giá rẻ trên **Vercel**.
* **UI Library:** **Tailwind CSS** + **shadcn/ui** (hoặc Ant Design). Giúp build giao diện lịch booking dạng lưới (grid) cực kỳ mượt mà trên cả PC lẫn điện thoại.


* **Backend (API & Xử lý nghiệp vụ):**
* **Language/Runtime:** **Node.js (Express.js)** hoặc **Python (FastAPI)**.
* *Lý do:* Nhẹ, xử lý bất đồng bộ tốt, dễ deploy lên gói cơ bản của **Render**.


* **Database & Storage (Cơ sở dữ liệu):**
* **Database chính:** **PostgreSQL** (chạy trên Render). Lưu trữ thông tin User, Phòng, Ca booking, Lịch sử, Tài chính.
* **Lưu trữ file/upload:** **Supabase Storage** hoặc **Cloudinary** (Gói free hoặc phí thấp để lưu ảnh/tài liệu phần "lịch sử upload").


* **Infrastructures (Hạ tầng theo yêu cầu):**
* **Frontend Hosting:** **Vercel** (Tối ưu CDN toàn cầu).
* **Backend & DB Hosting:** **Render** (Web Service + PostgreSQL Database).
* **Domain:** Tên miền quốc gia `.vn` (~600k/năm).



---

## PHẦN 2: CHI TIẾT CÁC FUNCTIONS & MODULES THEO ROLE

---

### I. MODULE HỆ THỐNG CHUNG (CORE RULES)

* **Khung giờ:** 09:00 - 24:00 mỗi ngày.
* **Cấu trúc ca:** Mỗi ca kéo dài 60 phút, nghỉ giữa các ca 15 phút (Ví dụ: Ca 1: 09:00 - 10:00, Ca 2: 10:15 - 11:15, v.v.).

---

### II. PHÍA USER (KHÁCH HÀNG)

#### 1. Module Xác thực & Định danh (`auth.service`)

* `api_user_login_or_register(phone_number)`
* **Input:** Số điện thoại.
* **Process:** Kiểm tra trong bảng `Users`. Nếu chưa có -> Tạo mới bản ghi với role `user`. Nếu có -> Lấy thông tin. Trả về Token xác thực (JWT) lưu vào LocalStorage/Cookie.


* `api_user_check_session()`
* Kiểm tra tính hợp lệ của token khi user truy cập lại.



#### 2. Module Truy cập lần đầu & Lịch sử ngắn hạn (`onboarding.service`)

* `api_client_build_local_history(device_id)`
* Ghi nhận và lưu vết các thao tác/lịch sử tạm thời trên thiết bị của user trước khi họ chính thức đăng nhập bằng số điện thoại.



#### 3. Module Lịch Booking & Đặt phòng (`booking.service`)

* `api_get_calendar_matrix(date, room_id)`
* **Output:** Trả về danh sách tất cả các phòng và trạng thái các ca (Trống / Đã đặt / Đang bảo trì) trong khung 9h00 - 24h00 của ngày được chọn. Tối ưu hiển thị dạng lưới cho cả Mobile và Desktop.


* `api_user_create_booking(payload)`
* **Input:** `user_id` (hoặc SĐT), `room_id`, `list_slot_ids` (danh sách các ca chọn), `player_count` (số lượng người chơi).
* **Validation quan trọng:**
* Kiểm tra số lượng ca: **User chỉ được phép book tối đa 2 ca liên tiếp**. Nếu `length(list_slot_ids) > 2`, hệ thống chặn và trả về lỗi thông báo vượt quá giới hạn cho phép.
* Kiểm tra xem các ca đó đã bị người khác đặt chưa (tránh trùng lặp).


* **Process:** Lưu vào bảng `Bookings`, cập nhật trạng thái ca thành `Booked`.



#### 4. Module Lịch sử cá nhân (`history.service`)

* `api_user_get_played_history(user_id)`
* Lấy toàn bộ danh sách các phòng và ca chơi mà user đã hoàn thành hoặc đang đặt (Lịch sử toàn bộ chỗ đã chơi).


* `api_user_get_upload_history(user_id)`
* Lấy danh sách các tài liệu, hình ảnh hoặc file mà user đã từng upload lên hệ thống (truy vấn từ bảng `UserUploads`).



---

### III. PHÍA ADMIN (QUẢN TRỊ HỆ THỐNG)

Phân hệ Admin được chia thành 3 nhóm quyền rõ rệt:

#### 1. System-Admin (Quản trị viên hệ thống tối cao)

* `api_sysadmin_manage_slots_and_pricing(action, data)`
* Thêm, sửa, xóa hoặc thay đổi cấu hình giá tiền cho từng ca chơi trong khung 9h00 - 24h00.


* `api_sysadmin_manage_rooms(action, room_data)`
* Chức năng thêm phòng mới, sửa tên/mô tả phòng, hoặc xóa/khóa phòng.


* `api_sysadmin_financial_dashboard(filter_date_range)`
* Kiểm soát tài chính, tính toán tổng doanh thu theo ngày, tuần, và **tổng hợp doanh thu hàng tháng** dưới dạng biểu đồ hoặc bảng số liệu.



#### 2. Admin-View (Quản lý thông tin & Lịch trình)

* `api_adminview_update_customer_info(customer_id, new_data)`
* Cho phép thay đổi một số thông tin cơ bản của khách hàng (Tên, SĐT, ghi chú, trạng thái).


* `api_adminview_get_weekly_schedule()`
* Hiển thị lịch trình toàn bộ hệ thống dưới dạng **lịch theo tuần** trực quan, giúp theo dõi công suất lấp đầy phòng của quán.



#### 3. Admin Quầy (Receptionist Admin)

* `api_admin_reception_modify_booking(booking_id, new_room_id, new_slots)`
* Thay đổi lịch booking của user (đổi giờ, đổi phòng) **với điều kiện khung giờ mới còn trống**.


* `api_admin_reception_force_book_slots(user_id, room_id, list_slots)`
* Đặc quyền của Admin Quầy: **Được phép book nhiều ca liên tiếp (trên 2 ca)** cho user theo yêu cầu trực tiếp tại quầy, vượt qua giới hạn chặn của user thông thường.