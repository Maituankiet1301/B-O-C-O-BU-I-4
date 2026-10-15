# ĐẶC TẢ CHI TIẾT USE CASE (USE CASE SPECIFICATION)
**Phân hệ / Luồng nghiệp vụ:** Luồng L6 – Kho dữ liệu bán hàng và báo cáo doanh thu  
**Hệ thống:** Mekong Mobile Smart CRM  
**Sinh viên thực hiện:** Mai Tuấn Kiệt (MSSV: 2374802010259) – Track DA  

---

## 1. SƠ ĐỒ USE CASE TỔNG QUAN
- **File sơ đồ gốc:** [docs/usecase.drawio](file:///c:/Users/Admin/OneDrive/B%C3%A1o%20C%C3%A1o%20Bu%E1%BB%95i%204/docs/usecase.drawio) (Vẽ trên Draw.io, tuân thủ chuẩn UML).
- **Ranh giới hệ thống:** Hệ thống Kho dữ liệu bán hàng và Báo cáo doanh thu Mekong Mobile.
- **Tác nhân (Actors):**
  1. *Kỹ sư dữ liệu (Data Engineer):* Người quản trị pipeline ETL và kho dữ liệu.
  2. *Hệ thống lập lịch (Cron Scheduler):* Tác nhân thời gian tự động kích hoạt pipeline lúc 02:00 sáng.
  3. *Quản lý cửa hàng (Store Manager):* Người xem báo cáo doanh thu chi nhánh phụ trách.
  4. *Giám đốc kinh doanh / Ban giám đốc (Business Director):* Người xem báo cáo tổng hợp 24 chi nhánh.
- **Danh sách 6 Use Case:**
  - UC1: Trích xuất và nạp dữ liệu từ 24 cửa hàng vào Staging.
  - UC2: Làm sạch, chuẩn hóa và kiểm tra chất lượng dữ liệu.
  - UC3: Nạp dữ liệu vào mô hình kho dữ liệu (Fact & Dimensions).
  - UC4: Xem dashboard doanh thu theo ngày và tháng của cửa hàng.
  - UC5: Phân tích doanh thu đa chiều theo nhóm sản phẩm theo quý.
  - UC6: Báo cáo so sánh cửa hàng & Cảnh báo sụt giảm 3 tháng liên tiếp.

---

## 2. ĐẶC TẢ CHI TIẾT USE CASE CỐT LÕI (USE CASE SPECIFICATION)

### UC2 – LÀM SẠCH, CHUẨN HÓA VÀ KIỂM TRA CHẤT LƯỢNG DỮ LIỆU
- **Actor chính:** Kỹ sư dữ liệu (Data Engineer) / Kích hoạt tự động bởi Hệ thống lập lịch (Cron Scheduler).
- **Mục tiêu:** Xử lý toàn bộ các dị biệt dữ liệu nguồn (ngày tháng 3 định dạng, mã đơn trùng, số điện thoại, chuỗi tiền tệ lỗi) và cách ly các dòng vi phạm chất lượng vào bảng `reject_orders` / `reject_order_items` trước khi nạp vào kho dữ liệu.
- **Điều kiện trước (Pre-conditions):**
  1. Pipeline trích xuất (UC1) đã hoàn tất đưa toàn bộ ~26.000 dòng đơn hàng thô vào vùng Staging (`stg_orders`, `stg_order_items`).
  2. Các bảng danh mục chuẩn (`stores.csv`, `products.csv`) đã sẵn sàng.
- **Điều kiện sau (Post-conditions):**
  1. Dữ liệu đạt chuẩn chất lượng được chuẩn bị sẵn sàng để nạp vào bảng Fact và các bảng Dimension.
  2. 100% các dòng không hợp lệ được lưu trữ an toàn trong bảng `reject_orders` và `reject_order_items` kèm mã lỗi và dấu thời gian kiểm toán.
- **Liên quan:** US2 | **Mức ưu tiên:** MUST

#### LUỒNG SỰ KIỆN CHÍNH (Happy Path):
1. Hệ thống đọc từng lô bản ghi dữ liệu đơn hàng từ bảng `stg_orders`.
2. Hệ thống thực hiện giải mã định dạng ngày tháng `Ngay` bằng bộ parser ưu tiên và chuẩn hóa về định dạng ISO-8601 (`YYYY-MM-DD`).
3. Hệ thống tạo Khóa nghiệp vụ phức hợp duy nhất bằng cách kết hợp: `ma_cua_hang || '_' || ma_don`.
4. Hệ thống chuẩn hóa số điện thoại khách hàng `SDT` (loại bỏ ký tự lạ, chuyển đầu số `84` thành `0`, kiểm tra đủ 10 chữ số).
5. Hệ thống loại bỏ ký tự tiền tệ `"đ"` và dấu phân cách, chuyển đổi trường `Thanh tien` thành số nguyên chuẩn VNĐ.
6. Hệ thống đối soát tính nhất quán ở bảng chi tiết: kiểm tra $|thanh\_tien - so\_luong \times don\_gia| \le 1$.
7. Hệ thống đối chiếu `ma_cua_hang` với danh mục chuẩn `dim_store`.
8. Hệ thống xuất dữ liệu sạch sang bảng tạm chuẩn bị nạp (Ready-to-Load) và ghi nhận nhật ký làm sạch thành công (Data Quality Log).

#### LUỒNG NGOẠI LỆ (Exceptions):
- **2a. Ngày đặt hàng sai định dạng hoặc giá trị bất hợp lý:**
  - *Điều kiện:* Chuỗi ngày không khớp cả 3 định dạng hoặc có ngày tháng không tồn tại (VD: `31/02/2025`) hoặc ngày trong tương lai.
  - *Hành vi hệ thống:* Hệ thống từ chối nạp bản ghi này vào kho; tự động đẩy bản ghi sang bảng `reject_orders` với lý do `INVALID_DATE_FORMAT`; pipeline tiếp tục xử lý các bản ghi tiếp theo mà không bị dừng đột ngột.
- **4a. Số điện thoại khách hàng không đúng chuẩn:**
  - *Điều kiện:* Chuỗi SĐT ít hơn hoặc nhiều hơn 10 chữ số sau khi chuẩn hóa.
  - *Hành vi hệ thống:* Hệ thống vẫn tiếp nhận đơn hàng nhưng gán cờ `is_valid_phone = FALSE` và gán `customer_key = 0` (Khách vãng lai chưa định danh), không làm gián đoạn việc tính doanh thu.
- **6a. Chênh lệch thành tiền vượt quá sai số cho phép ($> 1$ VNĐ):**
  - *Điều kiện:* Giá trị `thanh_tien != so_luong * don_gia` với mức lệch $> 1$ đồng.
  - *Hành vi hệ thống:* Hệ thống cách ly dòng chi tiết sản phẩm này vào bảng `reject_order_items` với mã lỗi `INCONSISTENT_LINE_TOTAL`; gửi thông báo cảnh báo mức High cho Kỹ sư dữ liệu đối soát lại với kế toán cửa hàng.
- **7a. Mã cửa hàng không tồn tại trong danh mục 24 cửa hàng:**
  - *Điều kiện:* `ma_cua_hang` nằm ngoài dải từ `CH01` đến `CH24`.
  - *Hành vi hệ thống:* Đẩy toàn bộ đơn hàng vào bảng `reject_orders` với mã lỗi `UNKNOWN_STORE_ID`; tạo ticket cảnh báo lỗi nguồn dữ liệu.

---

### UC4 – XEM DASHBOARD DOANH THU THEO CỬA HÀNG VÀ THỜI GIAN
- **Actor chính:** Quản lý cửa hàng (Store Manager) / Ban giám đốc.
- **Mục tiêu:** Cung cấp báo cáo trực quan về doanh số bán hàng, số lượng đơn hoàn tất và so sánh với chỉ tiêu doanh thu theo ngày và theo tháng.
- **Điều kiện trước:** Người dùng đã xác thực vào hệ thống Dashboard Streamlit; kho dữ liệu đã hoàn tất nạp dữ liệu kỳ mới nhất.
- **Điều kiện sau:** Giao diện hiển thị đầy đủ biểu đồ đường biểu diễn doanh thu và các thẻ chỉ số KPI tổng hợp.
- **Liên quan:** US3 | **Mức ưu tiên:** MUST

#### LUỒNG SỰ KIỆN CHÍNH (Happy Path):
1. Quản lý cửa hàng truy cập màn hình "Báo cáo Doanh thu Chi nhánh".
2. Hệ thống tự động xác định chi nhánh mà quản lý được phân quyền phụ trách (hoặc cho phép chọn chi nhánh nếu là Ban giám đốc).
3. Người dùng chọn khoảng thời gian cần xem (Từ ngày ... Đến ngày ...).
4. Người dùng bấm nút "Áp dụng bộ lọc" (Apply Filter).
5. Hệ thống gửi truy vấn SQL tổng hợp đến kho dữ liệu DuckDB/PostgreSQL.
6. Hệ thống nhận tập kết quả và hiển thị:
   - Thẻ chỉ số (KPI Cards): Tổng doanh thu (VNĐ), Tổng số đơn hàng, Giá trị trung bình/đơn (AOV).
   - Biểu đồ đường (Line Chart): Diễn biến doanh thu theo từng ngày trong tháng.
   - Bảng kê chi tiết đơn hàng (đã che 4 số cuối SĐT khách hàng).

#### LUỒNG NGOẠI LỆ (Exceptions):
- **3a. Khoảng thời gian chọn không hợp lệ (Ngày bắt đầu > Ngày kết thúc):**
  - *Hành vi hệ thống:* Hệ thống hiển thị thông báo lỗi màu đỏ ngay dưới bộ lọc: *"Ngày bắt đầu không được lớn hơn ngày kết thúc"*; vô hiệu hóa nút Áp dụng bộ lọc cho đến khi người dùng điều chỉnh lại.
- **5a. Cửa hàng chưa có dữ liệu giao dịch phát sinh trong khoảng thời gian đã chọn:**
  - *Hành vi hệ thống:* Hệ thống hiển thị hộp thông báo trung tính: *"Không tìm thấy dữ liệu giao dịch nào cho cửa hàng này trong kỳ báo cáo đã chọn"*; các thẻ chỉ số hiển thị giá trị `0 VNĐ` thay vì phát sinh lỗi chia cho 0 hoặc lỗi hiển thị `NaN`.
- **6a. Mất kết nối tới cơ sở dữ liệu kho:**
  - *Hành vi hệ thống:* Dashboard hiển thị thông báo: *"Dịch vụ kho dữ liệu tạm thời không phản hồi. Vui lòng thử lại sau ít phút hoặc liên hệ Quản trị viên"*, kèm mã lỗi dịch vụ.
