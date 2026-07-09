# 📊 Business Rules - MediScan AI

Tài liệu này định nghĩa các quy tắc nghiệp vụ (Business Rules) cốt lõi cho dự án **MediScan AI**, được suy luận và tổng hợp từ tài liệu yêu cầu (README) và bối cảnh y tế.

---

## 1. Quản lý Đơn thuốc & Số hóa (Prescription Management & Digitization)

*   **BR-PR-01 (Tải lên hình ảnh):** Hệ thống phải cho phép người dùng (bệnh nhân/bác sĩ) tải lên hình ảnh đơn thuốc (viết tay hoặc bản in) qua Mobile App.
*   **BR-PR-02 (Nhận diện OCR):** Quá trình quét bằng AI (OCR) phải bóc tách thành công các trường thông tin y tế quan trọng bao gồm: Tên thuốc, Thành phần hoạt chất, Liều lượng, và Cách dùng/Tần suất.
*   **BR-PR-03 (Chuẩn hóa HL7 FHIR):** Mọi dữ liệu phi cấu trúc bóc tách từ ảnh chụp đơn thuốc phải được ánh xạ (mapping) thành định dạng JSON tuân thủ nghiêm ngặt tiêu chuẩn dữ liệu Y tế Quốc tế HL7 FHIR trước khi lưu trữ.
*   **BR-PR-04 (Lưu trữ Hồ sơ):** Đơn thuốc sau khi số hóa thành công phải được lưu tự động vào "Sổ y bạ điện tử" (Digital Health Log) của người dùng trên Cloud, cho phép truy xuất và tra cứu theo thời gian.

## 2. Cảnh báo Tương tác Thuốc (Smart DDI Alert)

*   **BR-DDI-01 (Trích xuất mã ATC):** Với mỗi loại thuốc được thêm vào hệ thống (qua quét đơn hoặc nhập tay), hệ thống phải tự động trích xuất mã ATC (Anatomical Therapeutic Chemical) của thuốc đó.
*   **BR-DDI-02 (Đối chiếu tương tác chéo):** Hệ thống phải tự động quét và đối chiếu chéo mã ATC của thuốc mới với toàn bộ danh sách thuốc hiện có trong "Tủ thuốc số" của người dùng.
*   **BR-DDI-03 (Đánh giá mức độ rủi ro):** Kết quả phân tích tương tác thuốc (DDI) phải được phân loại và hiển thị cảnh báo theo 3 cấp độ:
    *   🔴 **Nghiêm trọng (Severe):** Cấm/khuyến cáo mạnh mẽ không sử dụng chung.
    *   🟡 **Thận trọng (Caution):** Có thể sử dụng nhưng cần theo dõi hoặc điều chỉnh liều.
    *   🟢 **An toàn (Safe):** Không phát hiện tương tác nguy hiểm.
*   **BR-DDI-04 (Cơ sở dữ liệu gốc):** Mọi cảnh báo tương tác phải được truy vấn dựa trên đồ thị tương tác (DDI Graph) bằng Neo4j, đối chiếu với cơ sở dữ liệu chuẩn từ DrugBank Quốc gia.
*   **BR-DDI-05 (Chống Hallucination bằng RAG):** Hệ thống LLM tư vấn y khoa tuyệt đối không được phép tự do suy luận (sinh text) liên quan đến liều lượng hoặc loại thuốc. Việc tư vấn bắt buộc phải tuân theo kiến trúc RAG, chỉ truy xuất và trả lời dựa trên vùng dữ liệu y khoa đã được xác thực an toàn.

## 3. Nhắc nhở Uống thuốc (Medication Reminders)

*   **BR-MR-01 (Tạo lịch tự động):** Khi một đơn thuốc được số hóa và xác nhận thành công, hệ thống phải tự động sinh ra lịch nhắc uống thuốc dựa trên "Liều lượng" và "Cách dùng".
*   **BR-MR-02 (Tùy chỉnh lịch trình):** Người dùng có quyền xem, bật/tắt, hoặc chỉnh sửa lại thời gian của các lịch nhắc uống thuốc này cho phù hợp với sinh hoạt cá nhân.
*   **BR-MR-03 (Thông báo thời gian thực):** Hệ thống Notification Hub phải đảm bảo gửi thông báo đẩy (Push Notification/Websocket) nhắc lịch uống thuốc tới thiết bị người dùng đúng giờ (Real-time).

## 4. Bảo mật & Tuân thủ Pháp lý (Security & Compliance)

*   **BR-SEC-01 (Vô danh hóa dữ liệu):** Thông tin định danh cá nhân (PII - Tên, SĐT, CCCD...) phải được lưu trữ tách biệt và vô danh hóa (Anonymization) so với hồ sơ bệnh lý của người dùng.
*   **BR-SEC-02 (Mã hóa lưu trữ - At rest):** Toàn bộ dữ liệu y tế nhạy cảm (Đơn thuốc, Lịch sử khám bệnh) phải được mã hóa theo tiêu chuẩn AES-256 khi lưu trữ trong Database (MySQL/Neo4j).
*   **BR-SEC-03 (Mã hóa truyền tải - In transit):** Mọi dữ liệu trao đổi giữa Client (Mobile/Web) và Server phải được mã hóa qua giao thức SSL/TLS.
*   **BR-SEC-04 (Tuân thủ pháp lý):** Hệ thống phải đáp ứng các quy định, tiêu chuẩn về lưu trữ hồ sơ bệnh án theo Nghị định 117/2020/NĐ-CP để phòng khám có thể áp dụng hợp pháp.

## 5. Quản lý Người dùng & Phân quyền (User Roles) (Giả định thêm)

*   **BR-UM-01 (Vai trò người dùng):** Hệ thống phân định tối thiểu 3 vai trò:
    *   **Bệnh nhân (Patient):** Quản lý hồ sơ cá nhân, nhận nhắc nhở, xem cảnh báo thuốc.
    *   **Bác sĩ/Dược sĩ (Doctor/Pharmacist):** (Giai đoạn 4) Xem hồ sơ của bệnh nhân (khi được cấp quyền), thực hiện tư vấn trực tuyến.
    *   **Quản trị viên (Admin):** Quản lý tài khoản, cấu hình hệ thống, quản lý cơ sở dữ liệu thuốc.
*   **BR-UM-02 (Xác thực chuyên môn):** Bác sĩ/Dược sĩ tham gia vào hệ thống phải trải qua quy trình xác minh chứng chỉ hành nghề hợp lệ trước khi được cấp quyền tư vấn (Áp dụng cho roadmap Giai đoạn 4).
