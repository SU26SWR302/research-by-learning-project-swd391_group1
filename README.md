[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/rBNuyfdi)

# 🩺 MediScan AI – Medical Record Digitization & Drug Interaction Warning

> **"Bridging the gap between messy ink and medical safety."**
> Dự án đạt mục tiêu tối ưu hóa an toàn sức khỏe thông qua việc số hóa đơn thuốc và cảnh báo rủi ro tương tác dược lý bằng Trí tuệ nhân tạo.

---

## 📌 Tổng quan dự án (Project Overview)

Trong bối cảnh ngành y tế Việt Nam, việc bác sĩ kê đơn thuốc viết tay vẫn còn phổ biến, dẫn đến rủi ro đọc sai liều lượng và không kiểm soát được tương tác thuốc giữa các chuyên khoa khác nhau. **MediScan AI** ra đời như một giải pháp hỗ trợ bệnh nhân và phòng khám tuân thủ **Nghị định 117/2020/NĐ-CP**, biến áp lực pháp lý thành công cụ bảo vệ sức khỏe người dùng.

### 🚀 Tính năng cốt lõi

- **OCR Intelligent Scanning:** Nhận diện chữ viết tay từ ảnh chụp đơn thuốc với độ chính xác cao nhờ mô hình Transformer.
- **Smart DDI Alert:** Cảnh báo tương tác thuốc (Drug-Drug Interaction) theo 3 cấp độ (Nghiêm trọng - Thận trọng - An toàn) dựa trên cơ sở dữ liệu DrugBank quốc gia.
- **Digital Health Log:** Lưu trữ lịch sử bệnh án trên Cloud, hỗ trợ truy xuất nhanh khi cần thiết.
- **Medication Reminders:** Tự động thiết lập lịch nhắc uống thuốc từ dữ liệu đã số hóa.

---

## 🛠 Công nghệ sử dụng (Tech Stack)

Với đội ngũ 3 kỹ sư phần mềm, dự án được xây dựng dựa trên kiến trúc Microservices hiện đại:

| Layer                   | Technology                                            |
| :---------------------- | :---------------------------------------------------- |
| **Frontend**            | React, Vite, Tailwind CSS, Redux Toolkit              |
| **Backend**             | Java Spring Boot, Hibernate, Spring Security          |
| **AI/Machine Learning** | Python, TensorFlow/PyTorch, Tesseract OCR, NLP (BERT) |
| **Database**            | MySQL (User data), Neo4j (Drug Interaction Graph)     |
| **DevOps**              | Docker, AWS, GitHub Actions (CI/CD)                   |

---

## 🏗 Kiến trúc hệ thống (System Architecture)

Hệ thống được thiết kế theo nguyên lý **SOLID** và **Clean Architecture** để đảm bảo khả năng mở rộng:

1.  **Mobile App:** Thu thập dữ liệu hình ảnh đơn thuốc.
2.  **OCR Service:** Xử lý ảnh và bóc tách thực thể (NER - Named Entity Recognition).
3.  **Knowledge Engine:** Truy vấn cơ sở dữ liệu Neo4j để phát hiện các mối quan hệ tương tác dược lý phức tạp.
4.  **Notification Hub:** Quản lý thông báo và nhắc lịch theo thời gian thực (Websocket).

---

## ⚖️ Tính pháp lý & Bảo mật (Compliance & Security)

Dự án cam kết tuân thủ chặt chẽ các quy định về dữ liệu y tế:

- **Nghị định 117/2020/NĐ-CP:** Hỗ trợ bác sĩ và phòng khám số hóa hồ sơ, tránh các án phạt về hành chính do viết đơn thuốc không rõ ràng.
- **Vô danh hóa dữ liệu (Anonymization):** Thông tin định danh cá nhân (PII) được tách biệt hoàn toàn với hồ sơ bệnh lý.
- **Mã hóa:** Dữ liệu được mã hóa chuẩn AES-256 khi lưu trữ và SSL/TLS khi truyền tải.

---

## 📈 Lộ trình phát triển (Roadmap)

- [x] **Giai đoạn 1:** Xây dựng MVP (Số hóa đơn thuốc cơ bản).
- [ ] **Giai đoạn 2:** Hoàn thiện cơ sở dữ liệu tương tác thuốc chuyên sâu (DDI Graph).
- [ ] **Giai đoạn 3:** Hợp tác với các chuỗi nhà thuốc lớn để tích hợp API.
- [ ] **Giai đoạn 4:** Mở rộng tính năng tư vấn dược sĩ trực tuyến qua Video Call.

---

## 📦 Cài đặt (Installation for Devs)

```bash
# Clone the repository
git clone [https://github.com/Group1/mediscan-ai.git](https://github.com/your-team/mediscan-ai.git)

# Install dependencies for Frontend
cd client && npm install

# Run Spring Boot Backend
cd server && ./mvnw spring-boot:run
```
