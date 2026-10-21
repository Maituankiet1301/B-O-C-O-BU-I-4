# BÁO CÁO BUỔI 5 — THIẾT KẾ KIẾN TRÚC VÀ MÔ HÌNH DỮ LIỆU

**Học phần:** Chuyên đề tốt nghiệp 1 — Học kỳ 1, Năm học 2026–2027
**Họ tên:** Mai Tuấn Kiệt
**MSSV:** 2374802010259
**Track:** Data Analytics (DA)
**Luồng nghiệp vụ:** L6 — Kho dữ liệu bán hàng và báo cáo doanh thu
**Case study:** Smart CRM — Mekong Mobile
**Link GitHub repo:** [Maituankiet1301/B-O-C-O-BU-I-4](https://github.com/Maituankiet1301/B-O-C-O-BU-I-4)

---

## 1. Mục tiêu và phạm vi buổi 5

Buổi 4 đã hoàn thành SRS, Use Case và đặc tả yêu cầu dữ liệu. Buổi 5 tiếp tục bằng ba sản phẩm thiết kế còn thiếu của track DA: sơ đồ kiến trúc có luồng dữ liệu, mô hình kho dữ liệu có phát biểu GRAIN, và ba wireframe dashboard. Các thiết kế được truy vết về FR, US, UC và NFR đã có trong [docs/srs.md](docs/srs.md), [docs/usecase-specification.md](docs/usecase-specification.md) và [docs/data-requirements.md](docs/data-requirements.md).

**Phạm vi nộp BT1:** Đây là hồ sơ phân tích và thiết kế, không phải phần triển khai của BT2. Vì vậy báo cáo này không thêm mã ETL, truy vấn chạy được hay DDL thực thi. Các tệp sơ đồ `.drawio` là nguồn gốc có thể chỉnh sửa.

## 2. Các sản phẩm đã hoàn thiện

| Thành phần | Tệp | Nội dung |
|---|---|---|
| Thiết kế kiến trúc và data flow | [docs/architecture.drawio](docs/architecture.drawio) | Nguồn dữ liệu, lịch chạy, ETL, Staging, kiểm tra chất lượng, vùng Reject, kho sao, dashboard, tác nhân, chú thích và ngoài phạm vi. |
| Mô hình dữ liệu track DA | [docs/erd.drawio](docs/erd.drawio) | Một fact, bốn dimension; khóa, kiểu dữ liệu, mô tả, quan hệ và GRAIN. |
| Wireframe ba màn hình | [docs/wireframe.drawio](docs/wireframe.drawio), [docs/wireframe.png](docs/wireframe.png) | Dashboard chi nhánh, phân tích nhóm sản phẩm theo quý và so sánh cửa hàng/cảnh báo giảm doanh thu; mỗi trang ghi Use Case và nguồn dữ liệu. |
| Báo cáo buổi 5 | [BAO_CAO_BUOI_5.md](BAO_CAO_BUOI_5.md) | Lập luận lựa chọn kiến trúc, mô hình dữ liệu, quy tắc truy vết, quyết định và điểm cần xác nhận. |

## 3. Thiết kế kiến trúc

### 3.1. Các lớp và cách trao đổi dữ liệu

1. **Nguồn:** Các tệp CSV/Excel bán hàng, chi tiết đơn, danh mục sản phẩm và cửa hàng. Khi nạp dữ liệu, pipeline gắn mã cửa hàng từ tệp/thư mục nguồn vào từng dòng chi tiết để giữ định danh đơn hàng xuyên suốt.
2. **Điều phối và nạp:** Cron Scheduler kích hoạt pipeline theo lịch 02:00; Data Engineer có thể vận hành/quản lý các lần nạp. Tệp nguồn được đọc theo lô vào Staging, giữ thông tin nguồn để đối soát.
3. **Chuẩn hóa và kiểm soát chất lượng:** Chuẩn hóa ngày, mã cửa hàng, số điện thoại và tiền tệ; đối chiếu khóa, tính toàn vẹn đơn hàng/chi tiết và danh mục. Bản ghi hợp lệ được chuyển tiếp; bản ghi không hợp lệ được ghi vào Reject kèm mã lỗi và thông tin kiểm toán.
4. **Kho dữ liệu:** Dữ liệu hợp lệ được ánh xạ thành `dim_date`, `dim_store`, `dim_product`, `dim_customer` và `fact_sales` trong DuckDB. Các truy vấn phân tích đọc kho theo các khóa chiều.
5. **Truy vấn và trình bày:** Dashboard Streamlit truy vấn kho qua lớp truy cập dữ liệu. Lớp này giới hạn Quản lý cửa hàng ở chi nhánh được phân quyền; số điện thoại chỉ được trả ra giao diện sau khi che bốn chữ số cuối. Ban giám đốc xem báo cáo tổng hợp.

### 3.2. Lập luận kiến trúc gắn với NFR và đánh đổi

1. **NFR1 — hiệu năng ETL dưới 45 giây cho khoảng 26.000 đơn trên máy 4 nhân/8 GB:** Chọn pipeline ETL xử lý theo lô trong một tiến trình và nạp dữ liệu theo lô vào DuckDB thay vì tách thành nhiều microservice. Cách này giảm chi phí giao tiếp và vận hành cho quy mô bài tập; đánh đổi là các bước ETL dùng chung tiến trình, nên lỗi nghiêm trọng có thể ảnh hưởng cả lần chạy và cần được kiểm soát bằng log, reject và khả năng chạy lại.
2. **NFR3 — che 4 số cuối điện thoại và giới hạn Quản lý cửa hàng đúng một chi nhánh:** Chọn áp dụng lọc quyền tại lớp truy cập dữ liệu theo danh tính/vai trò và mã cửa hàng đã cấp, đồng thời chỉ tạo trường `phone_masked` cho lớp hiển thị. Cách này giảm nguy cơ lộ dữ liệu cá nhân qua dashboard; đánh đổi là cần quản lý ánh xạ người dùng–cửa hàng và kiểm thử mọi đường truy vấn, không chỉ ẩn bộ lọc trên giao diện.
3. **NFR4 — triển khai đa nền tảng bằng Docker Compose với hai lệnh build/up:** Chọn đóng gói ứng dụng, ETL và cấu hình dịch vụ trong cùng bộ triển khai Docker Compose; dùng DuckDB dạng tệp cho nguyên mẫu kho. Cách này đơn giản hóa cài đặt và tái lập môi trường; đánh đổi là DuckDB phù hợp tải phân tích đơn nút, không phải lựa chọn mặc định cho nhiều tiến trình ghi đồng thời, nên tải ghi/đồng thời cao hơn cần đánh giá phương án PostgreSQL.
4. **NFR2 — null 0% ở các trường cốt lõi và cách ly đầy đủ dòng lỗi:** Chọn cổng kiểm tra chất lượng trước khi nạp Fact, với bảng Reject và nhật ký nguồn riêng thay vì âm thầm thay dữ liệu lỗi bằng giá trị mặc định. Cách này bảo toàn khả năng kiểm toán; đánh đổi là bản ghi lỗi chưa thể hiện trong báo cáo cho tới khi được xử lý và nạp lại.

### 3.3. Ngoài phạm vi

POS/thanh toán tại quầy, quản lý tồn kho vật lý, gửi SMS/email và triển khai hạ tầng production nhiều nút không thuộc phạm vi thiết kế của luồng L6 trong BT1.

## 4. Mô hình dữ liệu phân tích

### 4.1. GRAIN và quy tắc khóa

> **Một dòng `fact_sales` đại diện cho một mặt hàng (một dòng chi tiết sản phẩm) trong một đơn hàng, tại một cửa hàng vào một ngày cụ thể.**

- `sales_fact_key` là khóa thay thế của dòng Fact.
- Dòng nguồn được nhận diện/đối soát bằng bộ `(store_key, order_id, order_line_number)`. Mã đơn riêng lẻ không duy nhất trên toàn chuỗi: 137 mã trùng đã được ghi nhận giữa các cửa hàng.
- `order_line_number` là số thứ tự dòng chi tiết được gán khi nạp; phải giữ ổn định theo bản ghi nguồn để hỗ trợ chạy lại có tính lặp an toàn.
- `order_items.csv` trong từ điển hiện có chỉ nêu `ma_don`, chưa nêu cột cửa hàng. Do đó, khi nhập từng tệp cửa hàng, pipeline phải gắn `store_id` từ ngữ cảnh tệp trước khi ghép với orders. Không ghép chỉ bằng `ma_don`. Nếu dữ liệu chi tiết không có thông tin đủ để xác định cửa hàng, cần dừng đối soát và xác nhận nguồn thay vì đoán cửa hàng.
- `orders_2024_2026.csv` có tổng tiền cấp đơn; Fact ở mức dòng sản phẩm nên không cộng lặp tổng tiền đơn vào mỗi dòng. `line_total` lấy từ chi tiết đơn; tổng cấp đơn dùng cho đối soát.
- Bản ghi khách không có/không hợp lệ số điện thoại ánh xạ tới thành viên khách vãng lai dự phòng (`customer_key = 0`); các số điện thoại chuẩn vẫn là dữ liệu nhạy cảm và không được đưa ra giao diện dạng rõ.

### 4.2. Bảng và thuộc tính

Mô tả đầy đủ từng cột, kiểu dữ liệu, khóa và quan hệ nằm trong [docs/erd.drawio](docs/erd.drawio). Tóm tắt:

| Bảng | Khóa và thuộc tính chính | Vai trò |
|---|---|---|
| `fact_sales` | `sales_fact_key BIGINT` PK; `date_key INTEGER`, `store_key BIGINT`, `product_key BIGINT`, `customer_key BIGINT` FK; `order_id`, `order_line_number`; `quantity`, `unit_price`, `line_total`, `discount_amount`; nguồn tệp/dòng. | Số đo doanh thu ở grain dòng sản phẩm; lưu định danh đơn và lineage để đối soát. |
| `dim_date` | `date_key INTEGER` PK (`YYYYMMDD`), `full_date DATE`, ngày/tháng/quý/năm. | Lọc và tổng hợp theo ngày, tháng, quý, năm. |
| `dim_store` | `store_key BIGINT` PK, `store_id INTEGER` mã nghiệp vụ, `store_code`, `store_name`. | Chi nhánh phát sinh giao dịch. |
| `dim_product` | `product_key BIGINT` PK, `product_id VARCHAR(50)`, `product_name`, `normalized_name`, `category_name`. | Sản phẩm và nhóm danh mục để phân tích doanh thu theo quý. |
| `dim_customer` | `customer_key BIGINT` PK, `normalized_phone VARCHAR(15)` (nhạy cảm), `phone_masked VARCHAR(15)`, `is_valid_phone BOOLEAN`. | Phân tích khách quay lại; khách không định danh có hàng dự phòng khóa 0. |

**Quy tắc số đo:** `quantity` là số nguyên dương; `unit_price` và `line_total` là số nguyên VNĐ; áp dụng CL4 với sai số tối đa 1 VNĐ. `discount_amount` là số tiền giảm được xác định từ dữ liệu nguồn khi có cơ sở; nếu nguồn không cung cấp giảm giá tường minh thì không suy diễn khoản giảm từ chênh lệch không giải thích được—dòng vượt quy tắc chất lượng phải vào Reject. Trong bản dữ liệu hiện mô tả, thành tiền chuẩn bằng số lượng × đơn giá.

**Dimension chưa có bản ghi danh mục:** Cần có thành viên dự phòng cho sản phẩm chưa rõ danh mục (`product_key = -1`) và khách vãng lai (`customer_key = 0`) để giữ giao dịch hợp lệ. Không dùng thành viên dự phòng để che lỗi khóa cửa hàng hoặc ngày giao dịch; các lỗi này phải bị từ chối theo quy tắc hiện hành.

## 5. Wireframe và truy vết Use Case

Wireframe ở [docs/wireframe.drawio](docs/wireframe.drawio) gồm ba màn hình:

| Màn hình | Use Case | Bộ lọc, chỉ số và thông tin hiển thị | Bảng/thuộc tính nguồn |
|---|---|---|---|
| Doanh thu chi nhánh theo thời gian | UC4 / US3 | Khoảng ngày, cửa hàng theo quyền; tổng doanh thu, số đơn, AOV; đường doanh thu ngày; chi tiết mã đơn, thành tiền và SĐT đã che. | `dim_date`, `dim_store`, `dim_customer`, `fact_sales`; các số tổng hợp được tính từ Fact. |
| Cơ cấu doanh thu theo quý | UC5 / US5 | Năm, quý; tổng doanh thu, danh mục, doanh thu/tỷ trọng danh mục. | `dim_date`, `dim_product.category_name`, `fact_sales.line_total`. |
| So sánh cửa hàng và cảnh báo giảm ba tháng | UC6 / US4 | Cửa hàng, tháng báo cáo; doanh thu các tháng, % thay đổi và cờ giảm ba tháng liên tiếp. | `dim_store`, `dim_date`, `fact_sales.line_total`; tỷ lệ và cờ cảnh báo là phép tính trên dữ liệu Fact. |

Các khung không đưa thuộc tính ngoài mô hình vào giao diện. Wireframe thể hiện bố cục và dữ liệu dự kiến, không phải giao diện ứng dụng đã triển khai.

## 6. Quyết định và điểm cần xác nhận trước khi BT2

1. Giữ khóa nghiệp vụ đơn hàng theo cửa hàng + mã đơn, không dùng riêng mã đơn để ghép hoặc khử trùng lặp.
2. Nạp Fact ở grain dòng sản phẩm để trả lời phân tích theo nhóm sản phẩm; không nhân bản tổng tiền cấp đơn.
3. Tên/cấu trúc chi tiết của một số cột trong `stores.csv` và `products.csv`, cũng như cách đánh số dòng chi tiết, chưa được liệt kê đầy đủ trong Data Requirement hiện có. Khi có tệp nguồn thực tế, đối chiếu schema trước khi viết ETL; không tự giả định vùng/tỉnh, mã sản phẩm hoặc thứ tự dòng.
4. Yêu cầu phân quyền cửa hàng được biểu diễn trong kiến trúc; cần xác định nguồn danh tính/ánh xạ người dùng–cửa hàng trước khi triển khai dashboard.
5. Chưa có CSV thực tế trong repo để kiểm chứng số liệu hay hiệu năng. Các ngưỡng NFR và chất lượng là tiêu chí nghiệm thu đã nêu ở SRS, chưa phải kết quả benchmark của buổi này.

## 7. Checklist buổi 5

- [x] Sơ đồ kiến trúc có thành phần, trách nhiệm, luồng trao đổi, chú thích và ngoài phạm vi.
- [x] Có ít nhất ba lập luận kiến trúc, mỗi lập luận gắn với NFR và nêu đánh đổi.
- [x] Mô hình track DA có một Fact, bốn Dimension, GRAIN rõ ràng, khóa và mô tả kiểu dữ liệu.
- [x] Có ba wireframe, mỗi màn hình truy vết ít nhất một Use Case và chỉ dùng dữ liệu có trong mô hình.
- [x] Ghi nhận điểm mơ hồ về khóa ghép order–store và cách xử lý có kiểm soát.
- [ ] Xác nhận schema thực tế của các tệp CSV, quyền truy cập collaborator và kết quả peer review trước khi nộp BT1.

**Tự đánh giá:** Hoàn thành các đầu mục thiết kế buổi 5 trên cơ sở tài liệu hiện có. Các mục chưa đánh dấu chỉ có thể hoàn tất khi đối chiếu dữ liệu/thông tin triển khai thực tế; báo cáo không khẳng định đã benchmark hoặc thực hiện peer review.
