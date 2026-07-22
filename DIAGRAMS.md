# 📐 Biểu đồ Hệ thống - MediScan AI

Tài liệu này chứa các bản nháp biểu đồ cốt lõi cho dự án MediScan AI, được thiết kế bằng chuẩn mã nguồn **Mermaid**. 

> [!TIP]
> Bạn có thể xem trực tiếp các biểu đồ này bằng cách copy mã nguồn bên dưới (phần nằm trong block `mermaid`) và dán vào [Mermaid Live Editor](https://mermaid.live/), hoặc cài đặt extension hỗ trợ Mermaid (ví dụ: Markdown Preview Mermaid Support) ngay trên VS Code / GitHub.

---

## 1. Biểu đồ Use Case (Tổng quan tính năng)

Thể hiện các tác nhân (Actors) và các hành động chính mà họ có thể thực hiện trong hệ thống.

```mermaid
flowchart LR
    %% Định nghĩa các tác nhân
    subgraph Actors [Các đối tượng người dùng]
        P(Bệnh nhân / Patient)
        D(Bác sĩ & Dược sĩ / Doctor & Pharmacist)
        A(Quản trị viên / Admin)
    end
    
    %% Định nghĩa các tính năng hệ thống
    subgraph System [Hệ thống MediScan AI]
        UC1(Tải lên hình ảnh đơn thuốc)
        UC2(Xem cảnh báo tương tác thuốc - DDI)
        UC3(Quản lý lịch nhắc nhở uống thuốc)
        UC4(Xem Sổ y bạ điện tử)
        UC5(Tư vấn sức khỏe trực tuyến)
        UC6(Quản lý cơ sở dữ liệu Thuốc & User)
    end
    
    %% Phân quyền Patient
    P --> UC1
    P --> UC2
    P --> UC3
    P --> UC4
    P --> UC5
    
    %% Phân quyền Doctor
    D --> UC4
    D --> UC5
    
    %% Phân quyền Admin
    A --> UC6
```
![img.png](img.png)
---

## 2. Biểu đồ ERD (Mô hình Dữ liệu Quan hệ - RDS)

Thiết kế cơ sở dữ liệu cơ bản cho **MySQL**.
> [!IMPORTANT]
> - Bảng `USER_PROFILES` được tách riêng để mã hóa chuẩn AES (chuẩn hóa PII).
> - Trường `atc_code` trong bảng `PRESCRIPTION_ITEMS` chính là "chìa khóa" (tham chiếu logic) để Backend dùng đi query tiếp sang CSDL Đồ thị **Neo4j** nhằm tìm tương tác thuốc.
> - Bảng `PRESCRIPTIONS` có một cột `fhir_json_data` để lưu trữ dạng thô chuẩn Y tế Quốc tế.

```mermaid
erDiagram
    USERS ||--o| USER_PROFILES : "1:1 (Sở hữu)"
    USERS ||--o{ PRESCRIPTIONS : "1:N (Tải lên)"
    PRESCRIPTIONS ||--|{ PRESCRIPTION_ITEMS : "1:N (Bao gồm)"
    PRESCRIPTION_ITEMS ||--o{ MEDICATION_REMINDERS : "1:N (Lên lịch)"
    
    USERS {
        bigint user_id PK
        string email
        string password_hash
        string role "PATIENT, DOCTOR, ADMIN"
        datetime created_at
    }
    
    USER_PROFILES {
        bigint profile_id PK
        bigint user_id FK
        string full_name "Mã hóa (PII)"
        string identity_card "Mã hóa (PII)"
        string phone "Mã hóa (PII)"
        date dob
    }
    
    PRESCRIPTIONS {
        bigint prescription_id PK
        bigint patient_id FK
        bigint doctor_id FK "Nullable (Nếu bác sĩ kê)"
        string image_url "Đường dẫn ảnh AWS S3"
        json fhir_json_data "Dữ liệu JSON chuẩn HL7 FHIR"
        datetime created_at
    }
    
    PRESCRIPTION_ITEMS {
        bigint item_id PK
        bigint prescription_id FK
        string drug_name "Tên thuốc"
        string atc_code "Mã ATC (Dùng truy vấn Neo4j)"
        string dosage "Liều lượng"
        string frequency "Tần suất uống"
    }
    
    MEDICATION_REMINDERS {
        bigint reminder_id PK
        bigint item_id FK
        bigint patient_id FK
        string cron_expression "Thời gian nhắc (VD: 0 8 * * *)"
        string status "ACTIVE, INACTIVE"
    }
```
![img_3.png](img_3.png)

## 3. Biểu đồ Tuần tự (Sequence Diagram) - Luồng cốt lõi

Biểu diễn luồng xử lý phức tạp nhất: **Bệnh nhân tải đơn thuốc -> Nhận diện OCR -> Kiểm tra tương tác (DDI) -> Trả về kết quả.**

```mermaid
sequenceDiagram
    autonumber
    actor Patient as Bệnh nhân
    participant App as Mobile App
    participant API as API Gateway (Spring Boot)
    participant OCR as OCR & NLP Service (Python)
    participant Neo4j as Knowledge Engine (Neo4j)
    participant DB as MySQL DB
    
    Patient->>App: Chụp & Tải ảnh đơn thuốc
    activate App
    App->>API: POST /api/v1/prescriptions/upload (Kèm file ảnh)
    activate API
    
    API->>OCR: Gọi gRPC/REST: Phân tích hình ảnh
    activate OCR
    OCR-->>OCR: 1. OCR (Transformer) nhận diện chữ viết
    OCR-->>OCR: 2. NLP (BERT) bóc tách thực thể (Tên thuốc, Liều)
    OCR-->>OCR: 3. Ánh xạ dữ liệu sang JSON (Chuẩn HL7 FHIR)
    OCR-->>API: Trả về FHIR JSON Payload (Kèm mã ATC)
    deactivate OCR
    
    API->>DB: Lưu thông tin đơn thuốc & chi tiết thuốc
    
    API->>Neo4j: Truy vấn DDI bằng các mã ATC vừa nhận
    activate Neo4j
    Neo4j-->>API: Trả về mức độ rủi ro (Nghiêm trọng/Thận trọng/An toàn)
    deactivate Neo4j
    
    API->>DB: Sinh lịch nhắc nhở (Reminders) & Lưu kết quả DDI
    
    API-->>App: Trả về kết quả: Dữ liệu bóc tách + Cảnh báo DDI
    deactivate API
    
    App-->>Patient: Hiển thị giao diện Số hóa & Cảnh báo an toàn
    deactivate App
```
![img_2.png](img_2.png)

---

## 4. Mô hình Dữ liệu Đồ thị (Neo4j Graph Schema)

Biểu đồ này mô tả cấu trúc dữ liệu lưu trong **Neo4j** để phục vụ việc truy vấn nhanh tương tác thuốc (Drug-Drug Interaction - DDI).
- **Node (Thực thể):** `Drug` (Thuốc), `ActiveIngredient` (Hoạt chất).
- **Relationship (Mối quan hệ):** `CONTAINS` (Chứa thành phần), `INTERACTS_WITH` (Tương tác với).
- Cấu trúc này giúp Backend truy vấn cực nhanh: *"Tìm tất cả tương tác giữa các hoạt chất của thuốc A với thuốc B"*.

```mermaid
erDiagram
    DRUG ||--o{ ACTIVE_INGREDIENT : "CONTAINS (Chứa thành phần)"
    ACTIVE_INGREDIENT }|--|{ ACTIVE_INGREDIENT : "INTERACTS_WITH (Có tương tác)"
    
    DRUG {
        string drug_id "ID (Khớp với CSDL Thuốc QG)"
        string name "Tên biệt dược (VD: Panadol)"
    }
    
    ACTIVE_INGREDIENT {
        string atc_code PK "Mã ATC cốt lõi (VD: N02BE01)"
        string name "Tên hoạt chất (VD: Paracetamol)"
    }
    
    %% Đây là thuộc tính (properties) nằm trên đường nối INTERACTS_WITH của Neo4j
    INTERACTS_WITH_PROPERTIES {
        string severity "SEVERE, CAUTION (Mức độ nguy hiểm)"
        string description "Mô tả cơ chế tương tác"
        string reference "Nguồn (DrugBank)"
    }
```

---

## 5. Kiến trúc Hệ thống Tổng thể (System / Deployment Architecture)

Sơ đồ mô tả kiến trúc **Microservices** tổng thể, thể hiện rõ cách Spring Boot (Backend) và Python (AI OCR) phối hợp với các cơ sở dữ liệu (MySQL, Neo4j, S3).

```mermaid
flowchart TD
    %% Khối Client
    subgraph Client_Tier [Client / Frontend]
        App[Mobile App]
        Web[Web Portal cho Bác sĩ]
    end
    
    %% Gateway
    subgraph Gateway_Tier [API Gateway]
        AG[Spring Cloud Gateway + JWT Auth]
    end
    
    %% Khối Microservices (Java Spring Boot)
    subgraph Backend_Tier [Core Microservices - Spring Boot]
        UserSvc[User & Auth Service]
        PrescriptionSvc[Prescription & DDI Service]
        NotifSvc[Notification / WebSocket Service]
    end
    
    %% Khối AI (Python)
    subgraph AI_Tier [AI Services - Python]
        OCR[OCR & NLP Engine (FastAPI)]
        RAG[RAG Consulting Engine]
    end
    
    %% Khối Database & Storage
    subgraph Data_Tier [Polyglot Persistence Layer]
        MySQL[(MySQL - Hồ sơ, Lịch nhắc)]
        Neo4j[(Neo4j - Đồ thị tương tác thuốc)]
        Redis[(Redis - Message Broker / Cache)]
        S3[(AWS S3 - Lưu ảnh đơn thuốc)]
    end

    %% Routing
    Client_Tier -->|REST / HTTPS| AG
    AG --> UserSvc
    AG --> PrescriptionSvc
    
    %% Spring Boot giao tiếp DB
    UserSvc --> MySQL
    PrescriptionSvc --> MySQL
    PrescriptionSvc -->|Truy vấn Cypher| Neo4j
    PrescriptionSvc -->|Upload File| S3
    
    %% Spring Boot gọi Python
    PrescriptionSvc <-->|REST API / gRPC| OCR
    PrescriptionSvc <-->|Tư vấn an toàn| RAG
    
    %% AI Query
    RAG -.->|Chống ảo giác (RAG)| Neo4j
    
    %% Event & Notification
    PrescriptionSvc -->|Publish Event nhắc thuốc| Redis
    Redis -->|Consume Event| NotifSvc
    NotifSvc -->|WebSocket / Push Notif| App
```