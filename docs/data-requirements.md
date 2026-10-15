# ĐẶC TẢ YÊU CẦU DỮ LIỆU (DATA REQUIREMENT SPECIFICATION)
**Chuyên ngành:** Data Analytics (DA)  
**Luồng nghiệp vụ:** Luồng L6 – Kho dữ liệu bán hàng và báo cáo doanh thu  
**Dự án:** Mekong Mobile Smart CRM  
**Sinh viên thực hiện:** Mai Tuấn Kiệt (MSSV: 2374802010259)  

---

## 1. DANH SÁCH CÂU HỎI PHÂN TÍCH NGHIỆP VỤ

Mỗi câu hỏi phân tích đều gắn liền với nhu cầu thực tế của ban lãnh đạo chuỗi Mekong Mobile và được truy vết trực tiếp về các User Story tương ứng:

| Mã CH | Câu hỏi phân tích nghiệp vụ | Nguồn phát sinh yêu cầu | Mức chi tiết cần thiết (Granularity) | User Story truy vết |
| :---: | :--- | :--- | :--- | :---: |
| **CH1** | Doanh thu của từng cửa hàng theo từng ngày và từng tháng là bao nhiêu? | Ông Minh (Giám đốc): *"Tôi cần biết mỗi tháng doanh thu từng cửa hàng để theo dõi tiến độ kinh doanh"* | Cửa hàng + Ngày / Tháng | **US3** |
| **CH2** | Những cửa hàng nào có doanh thu sụt giảm liên tiếp trong 3 tháng gần nhất? | Ban giám đốc: *"...và cần cảnh báo sớm những cửa hàng nào đang đi xuống để can thiệp"* | Cửa hàng + 3 tháng liên tiếp | **US4** |
| **CH3** | Tỷ lệ khách hàng quay lại mua hàng lần thứ hai (Repeat Purchase Rate) là bao nhiêu? | Ban tiếp thị & CSKH: *"Bao nhiêu phần trăm khách hàng quay lại mua lần hai sau đơn đầu tiên?"* | Khách hàng (theo SĐT) + Lịch sử đơn | **US6** |
| **CH4** | Nhóm danh mục sản phẩm nào đóng góp tỷ trọng doanh thu lớn nhất theo từng quý? | Nhu cầu quản lý chuỗi cung ứng & phân loại sản phẩm bán chạy theo mùa vụ | Nhóm sản phẩm (Category) + Quý | **US5** |

---

## 2. NGUỒN DỮ LIỆU VÀ ĐÁNH GIÁ HIỆN TRẠNG

Dữ liệu đầu vào được thu thập từ hệ thống bán lẻ phân tán của 24 cửa hàng Mekong Mobile trong giai đoạn 2024 – 2026:

| Tệp dữ liệu nguồn | Định dạng | Tần suất cập nhật | Khối lượng dữ liệu | Vấn đề chất lượng dữ liệu đã ghi nhận |
| :--- | :--- | :--- | :--- | :--- |
| `orders_2024_2026.csv` | CSV, mã hóa UTF-8 | Hàng tháng (tổng hợp 24 cửa hàng) | ~26.000 dòng (26 tháng) | Ngày tháng bị lẫn lộn 3 định dạng (`dd/mm/yyyy`, `d-m-yy`, `yyyy.mm.dd`); Có 137 mã đơn hàng bị trùng lặp giữa các cửa hàng do tự phát sinh mã. |
| `order_items.csv` | CSV, mã hóa UTF-8 | Hàng tháng | ~48.000 dòng | Thành tiền (`thanh_tien`) đôi khi không bằng `so_luong * don_gia` do lỗi nhập liệu hoặc giảm giá không ghi nhận rõ. |
| `products.csv` | CSV, mã hóa UTF-8 | Cập nhật khi có sản phẩm mới | 420 dòng | Cùng một model máy nhưng có 4–5 cách viết tên khác nhau (thừa khoảng trắng, lẫn ký tự hoa/thường). |
| `stores.csv` | CSV, mã hóa UTF-8 | Cập nhật khi mở chi nhánh | 24 dòng | Dữ liệu danh mục chuẩn của 24 chi nhánh, cấu trúc ổn định. |

---

## 3. TỪ ĐIỂN DỮ LIỆU NGUỒN (DATA DICTIONARY)

> **Nguyên tắc kỹ thuật bắt buộc:** Toàn bộ tỷ lệ thiếu (null rate) dưới đây là **con số đo đạc thực tế** trên tập dữ liệu lịch sử bằng lệnh Python `df.isna().mean()`, không phải số ước lượng cảm tính.

### 3.1. Bảng dữ liệu đơn hàng nguồn (`orders_2024_2026.csv`)
| Cột nguồn | Kiểu dữ liệu thực tế | Ý nghĩa nghiệp vụ | Giá trị hợp lệ / Quy ước | Tỷ lệ thiếu (Đo được) | Kiểu dữ liệu đích (DWH) |
| :--- | :--- | :--- | :--- | :---: | :--- |
| `Ngay` | Chuỗi văn bản (String) | Ngày phát sinh đơn hàng | 3 định dạng: `dd/mm/yyyy`, `d-m-yy`, `yyyy.mm.dd` | **0.4%** | `DATE` (ISO-8601: `YYYY-MM-DD`) |
| `Ma don` | Chuỗi văn bản (String) | Mã hóa đơn do cửa hàng tự cấp | Chuỗi ký tự (VD: `HD0129`); Trùng mã giữa các cửa hàng | **0.0%** | `VARCHAR(50)` (Ghép thành Composite Key) |
| `SDT` | Chuỗi văn bản (String) | Số điện thoại liên hệ của khách | Dạng: `0xxxxxxxxx`, `84xxxxxxxxx`, chứa khoảng trắng | **6.2%** | `VARCHAR(15)` (Chuẩn hóa về 10 chữ số) |
| `Thanh tien` | Số / Chuỗi văn bản | Tổng tiền thanh toán của đơn | Số nguyên VNĐ; Có 218 dòng bị lưu kèm chữ "đ" phía sau | **1.1%** | `BIGINT` (Số nguyên VNĐ) |
| `Ma cua hang`| Số nguyên / Văn bản | Mã định danh cửa hàng phát sinh đơn | Từ `CH01` đến `CH24` (hoặc số 1 đến 24) | **0.0%** | `INTEGER` (Tham chiếu `dim_store`) |
| `Nhan vien` | Chuỗi văn bản (String) | Tên nhân viên đứng quầy bán | Tên gọi thân mật, không có mã nhân viên chuẩn hóa | **3.8%** | `VARCHAR(100)` |

### 3.2. Bảng chi tiết đơn hàng nguồn (`order_items.csv`)
| Cột nguồn | Kiểu dữ liệu thực tế | Ý nghĩa nghiệp vụ | Giá trị hợp lệ / Quy ước | Tỷ lệ thiếu (Đo được) | Kiểu dữ liệu đích (DWH) |
| :--- | :--- | :--- | :--- | :---: | :--- |
| `ma_don` | Chuỗi văn bản | Mã đơn hàng đối soát | Trùng khớp với mã đơn trong bảng orders | **0.0%** | `VARCHAR(50)` |
| `ma_san_pham`| Chuỗi văn bản | Mã hoặc tên sản phẩm | Tham chiếu danh mục sản phẩm | **0.2%** | `INTEGER` (Tham chiếu `dim_product`) |
| `so_luong` | Số nguyên | Số lượng sản phẩm mua | Số nguyên dương $> 0$ | **0.0%** | `INTEGER` |
| `don_gia` | Số nguyên | Đơn giá niêm yết một đơn vị | Số nguyên dương $> 0$ (VNĐ) | **0.0%** | `BIGINT` |
| `thanh_tien` | Số nguyên | Thành tiền của dòng hàng | $= so\_luong \times don\_gia$ | **0.0%** | `BIGINT` |

---

## 4. QUY TẮC CHẤT LƯỢNG DỮ LIỆU CÓ NGƯỠNG ĐO ĐƯỢC

| Mã CL | Chiều chất lượng (Quality Dimension) | Quy tắc nghiệp vụ và NGƯỠNG ĐO ĐƯỢC | Phương pháp và Câu truy vấn kiểm chứng (Validation Query) |
| :---: | :--- | :--- | :--- |
| **CL1** | **Tính đầy đủ** *(Completeness)* | Sau khi làm sạch, tỷ lệ khuyết thiếu (NULL) của các cột cốt lõi `ngay_dat` và `thanh_tien` trên bảng đích phải đạt **bằng đúng 0%**. | `SELECT COUNT(*) FROM fact_sales WHERE date_key IS NULL OR total_amount IS NULL;` $\rightarrow$ Kết quả kỳ vọng: **0**. |
| **CL2** | **Tính hợp lệ** *(Validity)* | **100%** giá trị `ngay_dat` nạp vào kho phải quy đổi thành công về chuẩn `YYYY-MM-DD`. Dòng sai định dạng phải đưa vào bảng `reject_orders`. | `SELECT COUNT(*) FROM fact_sales WHERE TRY_STRPTIME(CAST(date_key AS VARCHAR), '%Y%m%d') IS NULL;` $\rightarrow$ Kết quả kỳ vọng: **0**. |
| **CL3** | **Tính duy nhất** *(Uniqueness)* | Khóa nghiệp vụ kết hợp `(store_id, order_id)` phải là duy nhất **100%** trong bảng đơn hàng sạch. | So sánh: `COUNT(*)` và `COUNT(DISTINCT store_id \|\| '_' \|\| order_id)` trong bảng đơn hàng phải **bằng nhau**. |
| **CL4** | **Tính nhất quán** *(Consistency)* | Quy tắc logic: $|thanh\_tien - (so\_luong \times don\_gia)| \le 1$ VNĐ phải đạt tỷ lệ thỏa mãn **$\ge 99.5\%$** trên toàn bộ các dòng chi tiết. | `SELECT (COUNT(CASE WHEN ABS(thanh_tien - so_luong * don_gia) <= 1 THEN 1 END) * 100.0 / COUNT(*)) FROM stg_order_items;` $\ge 99.5\%$. |
| **CL5** | **Tính chính xác tham chiếu** *(Accuracy & Referential Integrity)* | **100%** mã cửa hàng `store_id` trong bảng đích phải tồn tại và ánh xạ thành công vào danh mục chuẩn `dim_store`. | `SELECT COUNT(*) FROM fact_sales f LEFT JOIN dim_store s ON f.store_key = s.store_key WHERE s.store_key IS NULL;` $\rightarrow$ Kết quả: **0**. |

---

## 5. MỨC CHI TIẾT (GRAIN) VÀ THIẾT KẾ CHIỀU KHO DỮ LIỆU

### 5.1. Phát biểu GRAIN chuẩn (Kimball Methodology)
> **"Một dòng trong bảng `fact_sales` đại diện cho MỘT MẶT HÀNG (SẢN PHẨM) nằm trong MỘT ĐƠN HÀNG được bán tại MỘT CỬA HÀNG vào MỘT NGÀY cụ thể."**

### 5.2. Hệ quả thiết kế lược đồ kho dữ liệu (Star Schema)
- **Bảng Fact:** `fact_sales`
  - **Khóa chính đại diện (Surrogate Primary Key):** `sales_fact_key` (hoặc khóa kết hợp `order_id, product_key`)
  - **Khóa ngoại tham chiếu các chiều:**
    - `date_key` (tham chiếu `dim_date`)
    - `store_key` (tham chiếu `dim_store`)
    - `product_key` (tham chiếu `dim_product`)
    - `customer_key` (tham chiếu `dim_customer`)
  - **Chỉ số đo lường (Measures):** `quantity` (số lượng), `unit_price` (đơn giá), `line_total` (thành tiền), `discount_amount` (tiền giảm giá).
- **Lý do lựa chọn mức Grain này:** Câu hỏi nghiệp vụ **CH4** yêu cầu phân tích đóng góp doanh thu theo từng nhóm danh mục sản phẩm. Nếu tổng hợp grain ở mức đơn hàng (`one row per order`), hệ thống sẽ mất thông tin mặt hàng chi tiết và không thể trả lời được câu hỏi CH4.

---

## 6. QUY TẮC BIẾN ĐỔI VÀ XỬ LÝ DỮ LIỆU LỖI (TRANSFORMATION RULES)

| Vấn đề dữ liệu nguồn | Hiện tượng quan sát trên dữ liệu thực tế | Quy tắc biến đổi áp dụng trong pipeline ETL | Xử lý ngoại lệ / Dòng lỗi |
| :--- | :--- | :--- | :--- |
| **Định dạng ngày lẫn lộn** | Tồn tại 3 định dạng song song: `dd/mm/yyyy`, `d-m-yy`, `yyyy.mm.dd` | Sử dụng bộ giải mã ưu tiên thử lần lượt 3 format (`%d/%m/%Y`, `%d-%m-%y`, `%Y.%m.%d`). Ép thành chuỗi ISO `YYYY-MM-DD`. | Dòng không thể parse được ghi vào bảng `reject_orders` kèm lý do `INVALID_DATE_FORMAT`. |
| **Trùng mã đơn hàng** | 137 mã đơn trùng lặp do 24 cửa hàng đánh số độc lập | Tạo khóa nghiệp vụ phức hợp ở staging: `ma_cua_hang || '_' || ma_don`. Tầng Fact sinh Surrogate Key tự tăng `sales_fact_key`. | Không có dòng bị loại; loại bỏ hoàn toàn hiện tượng đè dữ liệu sai lệch. |
| **Số điện thoại dị biệt** | 3 định dạng: `0xxxxxxxxx`, `84xxxxxxxxx`, chứa dấu cách/chấm | Regex loại bỏ toàn bộ ký tự không phải số; nếu bắt đầu bằng `84` thì đổi thành `0`; kiểm tra độ dài đúng 10 số. | SĐT không đúng 10 số được gán cờ `is_valid_phone = FALSE`, số khách vãng lai rỗng gán `customer_key = 0`. |
| **Thành tiền dính chữ "đ"** | 218 dòng chứa chuỗi ký tự text: `"1.500.000 đ"` | Dùng regex xóa bỏ ký tự `"đ"`, khoảng trắng và dấu chấm phân cách hàng nghìn, sau đó cast sang kiểu `BIGINT`. | Nếu không thể cast sang số hợp lệ $\rightarrow$ đưa vào `reject_orders` với lý do `CORRUPTED_CURRENCY_STRING`. |
| **Tên sản phẩm sai lệch** | Cùng model máy nhưng có 4–5 cách viết hoa/thường, khoảng cách thừa | Chuẩn hóa `lower().strip()`, loại bỏ khoảng trắng kép; đối chiếu với bảng ánh xạ chuẩn `products.csv`. | Sản phẩm không khớp từ điển được gán `product_key = -1` (Tên: `Sản phẩm chưa phân loại`) và ghi log cảnh báo. |
