# PHỤ LỤC: BẢNG KHAI BÁO SỬ DỤNG CÔNG CỤ AI
*(Bắt buộc đính kèm theo quy định học phần Chuyên đề tốt nghiệp 1 – Buổi 4 & BT1)*

**Học phần:** Chuyên đề tốt nghiệp 1 – Học kỳ 1, Năm học 2026 – 2027  
**Sinh viên thực hiện:** Mai Tuấn Kiệt  
**MSSV:** 2374802010259  
**Track:** Data Analytics (DA)  
**Luồng bài tập:** L6 – Kho dữ liệu bán hàng và báo cáo doanh thu  

---

### BẢNG KHAI BÁO CHI TIẾT CÁC CÔNG CỤ AI ĐÃ SỬ DỤNG

| Công cụ AI sử dụng | Dùng vào việc gì cụ thể | Áp dụng ở phần nào của bài tập | Đã kiểm chứng kết quả như thế nào? |
| :--- | :--- | :--- | :--- |
| **Claude / ChatGPT / Gemini** | Gợi ý cấu trúc các câu lệnh kiểm tra chất lượng dữ liệu (Data Quality Validation Queries) và biểu thức chính quy (Regex) làm sạch ngày tháng. | `docs/data-requirements.md` (Mục 4 và Mục 6) | Đã chạy thử nghiệm trực tiếp các câu lệnh SQL và Regex trên môi trường Python Jupyter Notebook với tập dữ liệu mẫu `orders_2024_2026.csv`; điều chỉnh biểu thức để không bỏ sót các trường hợp số điện thoại đầu `84`. |
| **Draw.io AI Assistant** | Gợi ý bố cục sắp xếp hình học các thành phần Actor và Use Case để sơ đồ trực quan và không bị chồng chéo đường nối. | `docs/usecase.drawio` | Tự tay kiểm tra và vẽ lại ranh giới hệ thống, bổ sung bảng chú thích ký hiệu (Legend) và khung cảnh báo ngoài phạm vi (Out of Scope) theo đúng chuẩn bài giảng. |
| **— Không dùng — (Tự làm hoàn toàn)** | Phân tích bài toán nghiệp vụ, phỏng vấn giả định các bên liên quan, xây dựng bảng thuật ngữ, xác định 6 User Story, viết 14 tiêu chí Given-When-Then, lập bảng truy vết 100% không ô trống. | `docs/srs.md` (Mục 1, Mục 2, Mục 3, Mục 5, Mục 6) và đặc tả chi tiết Use Case UC2 | Tự đọc và bám sát tài liệu đặc tả case study Mekong Mobile (Bảng 9.1 và dữ liệu 24 cửa hàng); đối chiếu với 7 câu hỏi tự kiểm tra của bản SRS tốt. |
| **GitHub Copilot (Copilot SDK trong VS Code)** | Hỗ trợ lập cấu trúc báo cáo, đề xuất cách biểu diễn kiến trúc, grain và các wireframe; sinh bản nháp sơ đồ để người học rà soát. | `BAO_CAO_BUOI_5.md`, `docs/architecture.drawio`, `docs/erd.drawio`, `docs/wireframe.drawio`, `README.md` | Đối chiếu từng yêu cầu với đề BT1, SRS, Data Requirement và Use Case hiện có; tự kiểm tra quan hệ, khóa, kiểu dữ liệu, trace Use Case, và mở/kiểm tra các tệp sơ đồ trước khi nộp. Các giả định chưa có dữ liệu nguồn được nêu rõ trong báo cáo. |

---

### LỜI CAM KẾT LIÊM CHÍNH HỌC THUẬT

> **"Tôi xác nhận đã đọc, hiểu và chịu trách nhiệm về toàn bộ nội dung nộp trong báo cáo và kho mã nguồn này."**

- **Họ và tên sinh viên:** Mai Tuấn Kiệt  
- **Mã số sinh viên (MSSV):** 2374802010259  
- **Ngày khai báo:** 15/10/2026  
