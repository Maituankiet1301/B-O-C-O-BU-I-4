# BÁO CÁO BUỔI 4 (THỰC HÀNH)
**Học phần:** Chuyên đề tốt nghiệp 1 – Học kỳ 1, Năm học 2026 – 2027  
**Họ tên:** Mai Tuấn Kiệt  
**MSSV:** 2374802010259  
**Track:** DA (Data Analytics)  
**Luồng nghiệp vụ:** L6 – Kho dữ liệu bán hàng và báo cáo doanh thu (Case study: Smart CRM – Mekong Mobile)  
**Link GitHub Repo:** [https://github.com/Maituankiet1301/B-O-C-O-BU-I-4](https://github.com/Maituankiet1301/B-O-C-O-BU-I-4)

---

### BẢNG ĐIỀN THÔNG TIN BÁO CÁO BUỔI 4

**1. Link commit cuối buổi (docs/srs.md + file .drawio):**  
[https://github.com/Maituankiet1301/B-O-C-O-BU-I-4/commit/06331c3](https://github.com/Maituankiet1301/B-O-C-O-BU-I-4/commit/06331c3) (Đã commit đầy đủ `docs/srs.md`, `docs/data-requirements.md`, `docs/usecase.drawio`, `docs/ai-disclosure.md` lên branch `main`).

**2. User Story:**  
**06** story | **03** MUST (US1, US2, US3) | **14** tiêu chí GWT (**06** tiêu chí ngoại lệ/validation).

**3. Use Case:**  
**04** actor (Kỹ sư dữ liệu, Quản lý cửa hàng, Giám đốc kinh doanh / Ban giám đốc, Hệ thống định kỳ Cron/Timer) | **06** use case | UC đặc tả chi tiết: **UC2 – Làm sạch, chuẩn hóa và kiểm tra chất lượng dữ liệu** (**03** luồng ngoại lệ: 3a Ngày sai định dạng không parse được -> reject, 4a Chênh lệch thành tiền != SL × Đơn giá -> reject, 5a Mã cửa hàng không tồn tại trong danh mục stores -> gán cảnh báo).

**4. SRS:**  
**6/6** mục đầy đủ theo chuẩn ISO/IEC/IEEE 29148 rút gọn | **06** FR có mã (FR1 đến FR6) | **04** NFR có ngưỡng số đo được | Truy vết còn **0** ô trống (100% ánh xạ đầy đủ FR ↔ US ↔ Use Case ↔ MoSCoW ↔ Test case dự kiến).

**5. Mẫu track (Data requirement spec):**  
**Xong** (Đạt chuẩn 6/6 mục: 4 câu hỏi phân tích CH1–CH4, 4 tệp nguồn dữ liệu, Từ điển dữ liệu với tỷ lệ thiếu đo đạc thực tế, 5 quy tắc chất lượng dữ liệu CL1–CL5 có ngưỡng đo và phương pháp kiểm chứng, Phát biểu GRAIN chuẩn Kimball, Bảng quy tắc biến đổi dữ liệu chi tiết).

**6. Giờ thực tế xong:**  
User Story **09:40** | Use Case **10:35** | SRS + track **11:30**

**7. Checklist đạt 10/10 – các mục chưa đạt (ghi số mục):**  
**Đạt 10/10 mục** theo Bảng tiêu chí kỹ thuật tối thiểu Buổi 4 (Không có mục nào chưa đạt).

**8. Peer review với:**  
**Trần Văn Nam** (Track **SE**) – tóm tắt phạm vi: **Đúng** (Bạn tóm tắt chính xác: *"Hệ thống xây dựng pipeline ETL thu thập dữ liệu bán hàng từ 24 file cửa hàng của Mekong Mobile, xử lý làm sạch các lỗi sai định dạng, nạp vào mô hình kho dữ liệu sao Star Schema và trực quan hóa dashboard đa chiều phục vụ quản lý cửa hàng và ban giám đốc"*).  
*Góp ý sẽ sửa:* Bạn Nam góp ý phần NFR về bảo mật nên làm rõ cách ẩn thông tin cá nhân khách hàng (che 4 số cuối số điện thoại trên dashboard phân tích), và bổ sung rõ kiểu dữ liệu đích (Target Data Type) trong từ điển dữ liệu. Em đã tiếp thu và cập nhật ngay vào mục 4 SRS và file `docs/data-requirements.md`.

**9. Chỗ chưa xong/còn phân vân – đã thử gì, kết quả ra sao:**  
- *Vấn đề phân vân:* 137 mã đơn trùng lặp giữa các cửa hàng khác nhau trong `orders_2024_2026.csv`.  
- *Đã thử:* Thử nghiệm tạo Surrogate Key tự tăng đơn lẻ (UUID/Auto-increment ID) so với việc dùng Khóa tự nhiên kết hợp (`ma_cua_hang` + `ma_don`).  
- *Kết quả:* Quyết định giữ Khóa nghiệp vụ là cặp (`ma_cua_hang` + `ma_don`) ở tầng Staging/ODS để dễ dàng đối soát ngược về cửa hàng khi có lệch số liệu kế toán, đồng thời tạo khóa đại diện `order_key` (surrogate key) ở tầng kho dữ liệu Dimensional Model để tối ưu hiệu năng join bảng Fact.

**10. Một quyết định em đưa ra hôm nay – dựa trên tiêu chí nào đã học:**  
- *Quyết định:* Xác định Mức chi tiết (GRAIN) của bảng Fact ở mức chi tiết nhất: *"Một dòng trong bảng fact_sales đại diện cho MỘT SẢN PHẨM trong MỘT ĐƠN HÀNG tại MỘT CỬA HÀNG vào MỘT NGÀY"* thay vì tổng hợp ở mức đơn hàng.  
- *Tiêu chí đã học:* Dựa trên nguyên lý thiết kế chiều của Ralph Kimball và yêu cầu trả lời câu hỏi phân tích CH4 ("Nhóm sản phẩm nào đóng góp doanh thu lớn nhất theo quý?"). Nếu chọn grain ở mức đơn hàng thì mất thông tin sản phẩm và không thể phân tích danh mục hàng bán chạy.

**11. Việc còn dở – hạn em tự đặt (trước buổi 5):**  
- Viết file script tạo schema DDL DuckDB và dựng thử nghiệm pipeline làm sạch dữ liệu mẫu bằng Pandas/DuckDB; vẽ phác thảo sơ đồ kiến trúc hệ thống và lược đồ Star Schema trên dbdiagram.io để chuẩn bị cho Buổi 5.  
- *Hạn tự đặt:* **22:00 ngày 18/10/2026**.

**12. Tự đánh giá:**  
**Đạt hết mục tiêu** (Đã hoàn thiện trọn vẹn bản SRS 6 mục, Use Case Diagram chuẩn ký pháp kèm chú thích, đặc tả UC2 chi tiết có luồng ngoại lệ, mẫu Data Spec track DA đầy đủ, khắc phục toàn bộ các lỗi thường gặp từ phản hồi BT1).
