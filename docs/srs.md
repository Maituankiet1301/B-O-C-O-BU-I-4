# BẢN ĐẶC TẢ YÊU CẦU PHẦN MỀM RÚT GỌN (SRS)
**Dự án:** Hệ thống Smart CRM – Mekong Mobile  
**Phân hệ / Luồng nghiệp vụ:** L6 – Kho dữ liệu bán hàng và báo cáo doanh thu  
**Sinh viên thực hiện:** Mai Tuấn Kiệt | **MSSV:** 2374802010259 | **Track:** Data Analytics (DA)  
**Tiêu chuẩn tham chiếu:** Rút gọn theo tinh thần chuẩn ISO/IEC/IEEE 29148  
**Link GitHub Repo:** [https://github.com/Maituankiet1301/B-O-C-O-BU-I-4](https://github.com/Maituankiet1301/B-O-C-O-BU-I-4)

---

## 1. GIỚI THIỆU VÀ PHẠM VI (Introduction & Scope)

### 1.1. Bối cảnh doanh nghiệp
Mekong Mobile là chuỗi bán lẻ thiết bị di động và phụ kiện với quy mô 24 chi nhánh trải rộng tại khu vực Đồng bằng sông Cửu Long. Hiện tại, dữ liệu bán hàng được kết xuất từ hệ thống bán lẻ cục bộ của 24 cửa hàng dưới dạng các file bảng tính (Excel/CSV) độc lập hàng tháng. Ban giám đốc và các quản lý cửa hàng đang gặp khó khăn lớn do dữ liệu phân tán, không đồng nhất về định dạng ngày tháng, xuất hiện mã đơn trùng lặp giữa các chi nhánh, sai lệch thành tiền và không có công cụ trực quan hóa tập trung để theo dõi doanh thu kịp thời.

### 1.2. Luồng nghiệp vụ lựa chọn
- **Tên luồng:** Luồng L6 – Xây dựng kho dữ liệu bán hàng tập trung và dashboard báo cáo doanh thu đa chiều.
- **Mục tiêu hệ thống:** Tự động hóa quá trình trích xuất dữ liệu bán hàng từ 24 cửa hàng, làm sạch và xử lý các lỗi chất lượng dữ liệu, mô hình hóa dữ liệu theo lược đồ hình sao (Star Schema) và xây dựng dashboard phân tích kinh doanh trực quan hỗ trợ ra quyết định.

### 1.3. Phạm vi và các điều chủ ý KHÔNG làm (WON'T Have - MoSCoW)
- **Trong phạm vi:**
  - Trích xuất dữ liệu từ các file nguồn lịch sử (`orders_2024_2026.csv`, `order_items.csv`, `products.csv`, `stores.csv` - quy mô ~26.000 đơn hàng qua 26 tháng).
  - Làm sạch, chuẩn hóa ngày tháng, số điện thoại, tên sản phẩm và kiểm tra tính toàn vẹn nghiệp vụ.
  - Xây dựng kho dữ liệu Dimensional Model (bảng `fact_sales` và các bảng chiều `dim_date`, `dim_store`, `dim_product`, `dim_customer`).
  - Xây dựng dashboard phân tích doanh thu theo ngày/tháng/cửa hàng, cảnh báo cửa hàng suy giảm doanh thu 3 tháng liên tiếp, và phân tích nhóm sản phẩm đóng góp doanh thu lớn nhất.
- **Chủ ý KHÔNG làm ở phiên bản này (Out of Scope / WON'T):**
  - Không xây dựng hệ thống thanh toán hay tính tiền tại quầy POS trực tiếp.
  - Không quản lý tồn kho vật lý chi tiết tại từng kệ hàng (thuộc phân hệ quản lý kho).
  - Không tự động gửi email/tin nhắn SMS thông báo doanh thu đến từng nhân viên (ghi nhận ở mức WON'T để tránh phân tán nguồn lực).

### 1.4. Bảng thuật ngữ dùng trong toàn tài liệu (Glossary)
| Thuật ngữ | Tên tiếng Anh | Định nghĩa & Ý nghĩa thống nhất |
| :--- | :--- | :--- |
| **Đơn hàng** | Order / Sales Order | Giao dịch mua hàng đã hoàn tất tại một cửa hàng của Mekong Mobile. |
| **Chi tiết đơn hàng** | Order Item | Một dòng sản phẩm cụ thể nằm trong một đơn hàng. |
| **Cửa hàng** | Store | Một trong 24 chi nhánh bán lẻ trực thuộc chuỗi Mekong Mobile. |
| **Sản phẩm** | Product | Thiết bị, điện thoại hoặc phụ kiện kinh doanh tại Mekong Mobile. |
| **Khách hàng** | Customer | Người mua hàng, định danh qua số điện thoại duy nhất 10 chữ số. |
| **Kho dữ liệu** | Data Warehouse (DWH) | Cơ sở dữ liệu phân tích tổ chức theo mô hình chiều (Star Schema). |
| **Bảng sự kiện** | Fact Table (`fact_sales`) | Bảng chứa các số đo định lượng (doanh thu, số lượng, đơn giá). |
| **Bảng chiều** | Dimension Table | Bảng chứa thuộc tính ngữ cảnh (thời gian, cửa hàng, sản phẩm, khách). |
| **Vùng dữ liệu lỗi** | Reject Table | Nơi lưu các bản ghi không đạt chuẩn chất lượng dữ liệu để kiểm toán. |
| **Dashboard** | Dashboard | Bảng hiển thị giao diện báo cáo trực quan các chỉ số kinh doanh chính. |

---

## 2. CÁC BÊN LIÊN QUAN VÀ VAI TRÒ NGƯỜI DÙNG (Stakeholders & User Roles)

| Mã Actor | Vai trò người dùng | Trách nhiệm và Quyền hạn trong hệ thống | Giới hạn quyền hạn (Không được làm gì) |
| :--- | :--- | :--- | :--- |
| **ACT-01** | **Kỹ sư dữ liệu** *(Data Engineer)* | Vận hành pipeline ETL, kiểm soát chất lượng dữ liệu, theo dõi bảng reject, bảo trì kho dữ liệu DuckDB/PostgreSQL. | Không can thiệp sửa trực tiếp số liệu doanh thu đã chốt; không có quyền xem thông tin cá nhân chưa mã hóa. |
| **ACT-02** | **Quản lý cửa hàng** *(Store Manager)* | Xem báo cáo doanh thu theo ngày/tháng của cửa hàng mình quản lý; theo dõi nhân viên bán hàng xuất sắc tại chi nhánh. | Không xem được số liệu doanh thu và chi tiết khách hàng của 23 cửa hàng khác. |
| **ACT-03** | **Giám đốc kinh doanh / Ban giám đốc** *(Executive / Business Director)* | Xem báo cáo doanh thu tổng hợp toàn bộ 24 cửa hàng; nhận diện cửa hàng có doanh thu giảm 3 tháng liên tiếp; phân tích tỷ trọng nhóm sản phẩm. | Không chỉnh sửa cấu hình hệ thống ETL và cơ sở dữ liệu nền tảng. |
| **ACT-04** | **Hệ thống lập lịch tự động** *(Cron Scheduler / System Timer)* | Kích hoạt tự động quá trình trích xuất, làm sạch và tổng hợp dữ liệu vào lúc 02:00 sáng hàng ngày (Actor hệ thống). | Không tự ý can thiệp logic nghiệp vụ tính toán doanh thu. |

---

## 3. YÊU CẦU CHỨC NĂNG VÀ USER STORY (Functional Requirements & User Stories)

### 3.1. Danh sách Yêu cầu chức năng (Functional Requirements - FR)
- **FR1 (Trích xuất dữ liệu thô):** Hệ thống phải có khả năng đọc và nạp dữ liệu từ 24 tệp dữ liệu bán hàng định dạng CSV/Excel của các cửa hàng vào vùng tạm (staging) mà không làm mất mát bản ghi.
- **FR2 (Làm sạch và chuẩn hóa dữ liệu):** Hệ thống phải tự động chuẩn hóa định dạng ngày tháng về ISO-8601 (`YYYY-MM-DD`), chuẩn hóa số điện thoại về 10 chữ số, loại bỏ ký tự lạ khỏi số tiền, và chuyển hướng các dòng lỗi vào bảng `reject_orders`.
- **FR3 (Khử trùng lặp và xác thực nghiệp vụ):** Hệ thống phải khử trùng lặp đơn hàng dựa trên khóa nghiệp vụ kết hợp (`ma_cua_hang`, `ma_don`) và kiểm tra điều kiện thành tiền khớp `so_luong * don_gia` với sai lệch $\le 1$ VNĐ.
- **FR4 (Mô hình hóa và nạp kho dữ liệu):** Hệ thống phải nạp dữ liệu sạch vào bảng `fact_sales` và các bảng chiều (`dim_date`, `dim_store`, `dim_product`, `dim_customer`) theo cấu trúc lược đồ hình sao Star Schema.
- **FR5 (Dashboard phân tích doanh thu cửa hàng):** Hệ thống phải cung cấp giao diện trực quan cho phép lọc và hiển thị doanh thu theo cửa hàng, theo khoảng thời gian, so sánh kết quả đạt được so với chỉ tiêu.
- **FR6 (Cảnh báo suy giảm và phân tích đa chiều):** Hệ thống phải tự động tính toán phát hiện các cửa hàng có doanh thu sụt giảm 3 tháng liên tiếp và phân tích nhóm sản phẩm đóng góp doanh thu lớn nhất theo quý.

---

### 3.2. Danh sách User Story chuẩn INVEST và phân loại MoSCoW

#### US1: Trích xuất dữ liệu bán hàng vào vùng Staging (Ưu tiên: MUST)
- **User Story:** Là **Kỹ sư dữ liệu**, tôi muốn hệ thống tự động trích xuất các tệp dữ liệu đơn hàng từ 24 cửa hàng vào vùng staging để có nguồn dữ liệu tập trung phục vụ xử lý mà không phải thu thập thủ công từng tệp.
- **Tiêu chí chấp nhận (Acceptance Criteria):**
  - **AC1.1 (Luồng chính):** *GIVEN* 24 file dữ liệu đơn hàng tồn tại đúng định dạng trong thư mục nguồn, *WHEN* quy trình trích xuất được kích hoạt, *THEN* toàn bộ ~26.000 dòng được đọc vào bảng `stg_orders` và `stg_order_items` với trạng thái thành công.
  - **AC1.2 (Ngoại lệ):** *GIVEN* một file cửa hàng bị mất hoặc không thể mở được (lỗi I/O), *WHEN* pipeline chạy, *THEN* hệ thống ghi nhận lỗi vào log kiểm toán, bỏ qua file lỗi, tiếp tục xử lý 23 file còn lại và gửi cảnh báo đỏ cho kỹ sư dữ liệu.

#### US2: Chuẩn hóa và làm sạch dữ liệu chất lượng (Ưu tiên: MUST)
- **User Story:** Là **Kỹ sư dữ liệu**, tôi muốn tự động chuẩn hóa các dị biệt về định dạng ngày tháng, số điện thoại, và khoảng trắng tên sản phẩm để dữ liệu đạt chất lượng đồng nhất trước khi nạp vào kho.
- **Tiêu chí chấp nhận (Acceptance Criteria):**
  - **AC2.1 (Luồng chính):** *GIVEN* các cột ngày chứa các định dạng `dd/mm/yyyy`, `d-m-yy`, `yyyy.mm.dd`, *WHEN* hàm chuẩn hóa thực thi, *THEN* 100% dòng hợp lệ được chuyển đổi sang chuẩn `YYYY-MM-DD`.
  - **AC2.2 (Ngoại lệ ngày lỗi):** *GIVEN* dòng dữ liệu chứa ngày không hợp lệ (ví dụ: ngày 31/02 hoặc chuỗi ký tự text rỗng), *WHEN* thực hiện parse ngày thất bại, *THEN* dòng đó bị loại khỏi tập dữ liệu sạch, ghi vào bảng `reject_orders` kèm lý do lỗi `INVALID_DATE_FORMAT`.
  - **AC2.3 (Ngoại lệ thành tiền không khớp):** *GIVEN* một dòng trong `stg_order_items` có `thanh_tien != so_luong * don_gia` với độ lệch vượt quá 1 VNĐ, *WHEN* kiểm tra quy tắc toàn vẹn, *THEN* dòng đó được đưa vào bảng `reject_order_items` với mã lỗi `INCONSISTENT_TOTAL_AMOUNT`.

#### US3: Xem dashboard doanh thu theo ngày và tháng của cửa hàng (Ưu tiên: MUST)
- **User Story:** Là **Quản lý cửa hàng**, tôi muốn xem biểu đồ doanh thu theo ngày/tháng của cửa hàng mình trên dashboard để đánh giá tiến độ kinh doanh và so sánh với chỉ tiêu được giao.
- **Tiêu chí chấp nhận (Acceptance Criteria):**
  - **AC3.1 (Luồng chính):** *GIVEN* quản lý đăng nhập chọn cửa hàng được phân công và chọn khoảng ngày trong tháng, *WHEN* xem dashboard, *THEN* hệ thống hiển thị biểu đồ đường doanh thu theo ngày, tổng doanh thu tháng, số lượng đơn hoàn tất trong thời gian dưới 2.5 giây.
  - **AC3.2 (Ngoại lệ không có dữ liệu):** *GIVEN* cửa hàng mới khai trương chưa phát sinh giao dịch trong khoảng ngày đã chọn, *WHEN* người dùng chọn lọc, *THEN* dashboard hiển thị thông báo "Không có giao dịch phát sinh trong khoảng thời gian này" mà không bị crash hay hiển thị giá trị NaN/Undefined.

#### US4: Báo cáo so sánh doanh thu và cảnh báo sụt giảm giữa các chi nhánh (Ưu tiên: SHOULD)
- **User Story:** Là **Giám đốc kinh doanh**, tôi muốn xem bảng so sánh doanh thu giữa 24 chi nhánh và danh sách cửa hàng bị sụt giảm doanh thu 3 tháng liên tiếp để chỉ đạo điều chỉnh kế hoạch bán hàng kịp thời.
- **Tiêu chí chấp nhận (Acceptance Criteria):**
  - **AC4.1 (Luồng chính):** *GIVEN* ban giám đốc truy cập mục Báo cáo Hiệu quả Chi nhánh, *WHEN* hệ thống truy vấn kho dữ liệu, *THEN* hiển thị xếp hạng doanh thu 24 cửa hàng kèm danh sách cảnh báo các cửa hàng có $DoanhThu_{Thang\_N} < DoanhThu_{Thang\_N-1} < DoanhThu_{Thang\_N-2}$.
  - **AC4.2 (Ngoại lệ):** *GIVEN* các tháng đầu kỳ lịch sử (tháng 1, tháng 2 năm 2024 chưa đủ 3 tháng), *WHEN* tính toán chỉ số sụt giảm, *THEN* hệ thống hiển thị trạng thái "Chưa đủ dữ liệu kỳ trước" thay vì cảnh báo sai.

#### US5: Phân tích doanh thu đa chiều theo nhóm sản phẩm (Ưu tiên: SHOULD)
- **User Story:** Là **Ban giám đốc**, tôi muốn phân tích cơ cấu doanh thu theo từng nhóm danh mục sản phẩm theo quý để nắm bắt thị hiếu người dùng và tối ưu cơ cấu nhập hàng.
- **Tiêu chí chấp nhận (Acceptance Criteria):**
  - **AC5.1 (Luồng chính):** *GIVEN* người dùng chọn quý và năm phân tích, *WHEN* bấm áp dụng bộ lọc, *THEN* hệ thống hiển thị biểu đồ hình tròn (pie chart) hoặc thanh (bar chart) phân bổ doanh thu theo nhóm thiết bị (điện thoại cao cấp, tầm trung, phụ kiện cáp sạc, tai nghe...).
  - **AC5.2 (Ngoại lệ mã sản phẩm lạ):** *GIVEN* sản phẩm bán ra không có tên trong danh mục `products.csv`, *WHEN* nạp vào kho dữ liệu, *THEN* sản phẩm được gán vào nhóm `Unknown (Mã: -1)` và vẫn tính đầy đủ vào doanh thu tổng để không làm lệch số liệu tiền tệ.

#### US6: Phân tích tỷ lệ khách hàng quay lại mua lần 2 (Ưu tiên: COULD)
- **User Story:** Là **Chuyên viên phân tích dữ liệu**, tôi muốn tính toán tỷ lệ phần trăm khách hàng phát sinh đơn hàng thứ hai (Repeat Purchase Rate) theo thời gian để đo lường mức độ giữ chân khách hàng của chuỗi.
- **Tiêu chí chấp nhận (Acceptance Criteria):**
  - **AC6.1 (Luồng chính):** *GIVEN* kho dữ liệu đã làm sạch số điện thoại khách hàng, *WHEN* chạy truy vấn phân tích tần suất mua sắm, *THEN* hệ thống hiển thị tỷ lệ khách hàng có từ 2 đơn hàng trở lên trên tổng số khách duy nhất.
  - **AC6.2 (Ngoại lệ số điện thoại rỗng):** *GIVEN* các đơn hàng khách vãng lai không để lại số điện thoại (null SDT), *WHEN* tính toán retention, *THEN* hệ thống nhóm các đơn này vào nhóm "Khách vãng lai chưa định danh" và loại khỏi mẫu số tính tỷ lệ khách quay lại để bảo đảm độ chính xác.

---

## 4. YÊU CẦU PHI CHỨC NĂNG CÓ NGƯỠNG ĐO ĐƯỢC (Non-Functional Requirements - NFR)

| Mã NFR | Phân loại | Câu hỏi cần trả lời | Phát biểu yêu cầu có NGƯỠNG ĐO ĐƯỢC cụ thể |
| :--- | :--- | :--- | :--- |
| **NFR1** | **Hiệu năng** *(Performance)* | Pipeline xử lý mất bao lâu? Dashboard tải trong bao lâu? | Toàn bộ pipeline ETL làm sạch và nạp 26.000 dòng đơn hàng phải hoàn tất trong **dưới 45 giây** trên máy tính tiêu chuẩn (CPU 4 nhân, 8 GB RAM). Giao diện Dashboard Streamlit phải phản hồi và render biểu đồ trong **dưới 2.5 giây** đối với truy vấn dữ liệu 12 tháng. |
| **NFR2** | **Chất lượng & Tin cậy** *(Data Quality & Reliability)* | Độ chính xác dữ liệu sau làm sạch đạt mức nào? | Tỷ lệ giá trị null của các trường cốt lõi (`ngay_dat`, `thanh_tien`, `ma_cua_hang`) trong bảng `fact_sales` phải đạt **bằng đúng 0%**. Tỷ lệ khớp tính toán `thanh_tien = so_luong * don_gia` đạt **$\ge 99.5\%$**. Các dòng vi phạm được cách ly 100% vào bảng reject. |
| **NFR3** | **Bảo mật & Quyền riêng tư** *(Security & Privacy)* | Ai được thấy dữ liệu gì? Dữ liệu nhạy cảm được bảo vệ ra sao? | Mọi số điện thoại khách hàng hiển thị trên giao diện Dashboard phân tích phải được **che (mask) 4 chữ số cuối** (ví dụ: `090123****`). Phân quyền truy cập đảm bảo Quản lý cửa hàng chỉ xem được số liệu của đúng **01 cửa hàng phụ trách**. |
| **NFR4** | **Khả chuyển & Dễ triển khai** *(Portability)* | Hệ thống chạy ở đâu, dựng môi trường mất bao lâu? | Toàn bộ kho dữ liệu DuckDB/PostgreSQL và ứng dụng Dashboard Streamlit phải đóng gói và khởi động thành công trên cả Windows, macOS, Linux bằng Docker Compose với **đúng 2 câu lệnh**: `docker-compose build` và `docker-compose up`. |

---

## 5. RÀNG BUỘC VÀ QUY TẮC NGHIỆP VỤ (Business Rules & Constraints)

Dựa trên tài liệu nghiệp vụ chuỗi Mekong Mobile (Bảng 9.1 và đặc thù xử lý dữ liệu bán hàng):
1. **QT-01 (Khóa nghiệp vụ đơn hàng):** Do các cửa hàng đánh mã đơn độc lập dẫn đến trùng mã giữa các chi nhánh, khóa nghiệp vụ duy nhất của một đơn hàng bắt buộc phải là cặp giá trị kết hợp: `(ma_cua_hang, ma_don)`.
2. **QT-02 (Chuẩn hóa ngày giao dịch):** Ngày đặt hàng chấp nhận đầu vào từ các định dạng `dd/mm/yyyy`, `d-m-yy`, `yyyy.mm.dd`, nhưng khi lưu trữ vào kho dữ liệu bắt buộc phải quy chuẩn về định dạng ISO-8601 (`YYYY-MM-DD`). Mọi đơn hàng có ngày tương lai ($> Ngày\_Hiện\_Tại$) hoặc trước năm thành lập chuỗi ($< 2020$) phải bị từ chối nạp.
3. **QT-03 (Toàn vẹn số tiền):** Trường `thanh_tien` của chi tiết đơn hàng phải là số nguyên dương (đơn vị: VNĐ). Quy tắc đối soát nghiệp vụ: $|thanh\_tien - (so\_luong \times don\_gia)| \le 1$ VNĐ (cho phép sai số làm tròn tối đa 1 đồng). Nếu chênh lệch lớn hơn, dòng dữ liệu bắt buộc phải bị cách ly vào bảng `reject_order_items`.
4. **QT-04 (Định danh khách hàng qua số điện thoại):** Số điện thoại khách hàng phải được chuẩn hóa về định dạng viễn thông Việt Nam gồm đúng 10 chữ số (bắt đầu bằng đầu số `0`). Các số có tiền tố quốc tế `84` phải đổi thành `0`. Các ký tự khoảng trắng, dấu gạch nối phải bị loại bỏ hoàn toàn.
5. **QT-05 (Tính toàn vẹn tham chiếu cửa hàng và sản phẩm):** Mọi đơn hàng hợp lệ bắt buộc phải có `ma_cua_hang` tồn tại trong danh mục 24 cửa hàng chính thức (`stores.csv`). Đối với sản phẩm không tìm thấy mã trong `products.csv`, hệ thống gán vào mã dự phòng `product_key = -1` (Tên: `Chưa rõ danh mục`) để không làm mất số liệu tổng doanh thu.

---

## 6. BẢNG TRUY VẾT YÊU CẦU (Requirements Traceability Matrix)

> **Ghi chú kiểm chứng bắt buộc:** Mọi yêu cầu chức năng (FR) đều được kết nối chặt chẽ tới ít nhất 1 User Story, 1 Use Case, mức ưu tiên MoSCoW và mã Test Case kiểm thử tương ứng. Tuyệt đối **không có ô trống**.

| Mã FR | Yêu cầu chức năng | User Story | Use Case | MoSCoW | Test Case dự kiến (BT3) |
| :---: | :--- | :---: | :---: | :---: | :---: |
| **FR1** | Trích xuất dữ liệu từ 24 file cửa hàng vào vùng Staging | US1 | UC1 | MUST | TC_ETL_01, TC_ETL_02 |
| **FR2** | Làm sạch định dạng ngày, số điện thoại, tiền tệ, chuyển dòng lỗi vào reject | US2 | UC2 | MUST | TC_CLN_01, TC_CLN_02, TC_CLN_03 |
| **FR3** | Khử trùng lặp khóa nghiệp vụ và đối soát thành tiền `SL × Đơn giá` | US2 | UC2 | MUST | TC_VAL_01, TC_VAL_02 |
| **FR4** | Mô hình hóa dữ liệu và nạp vào bảng Fact và các bảng Dimension | US1, US2 | UC3 | MUST | TC_DWH_01, TC_DWH_02 |
| **FR5** | Hiển thị dashboard doanh thu theo ngày/tháng có bộ lọc cửa hàng | US3 | UC4 | MUST | TC_DSH_01, TC_DSH_02 |
| **FR6** | Báo cáo so sánh cửa hàng, cảnh báo giảm 3 tháng và phân tích nhóm sản phẩm | US4, US5 | UC5, UC6 | SHOULD | TC_ANL_01, TC_ANL_02, TC_ANL_03 |
