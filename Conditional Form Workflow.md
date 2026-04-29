# Conditional Form Workflow

🟢 Xanh lá / Green = Luồng chính (Happy Path)　｜　🔴 Đỏ / Red = Rẽ nhánh điều kiện (Conditional Branch)

---

## 1. Sơ đồ tóm tắt (Summary)

```mermaid
graph TD
    START["🎯 BẮT ĐẦU<br/>(Start)"]:::start --> Q1

    Q1["❓ Q1: Độ tuổi người được bảo vệ?<br/>(Age of insured?)"]:::question

    Q1 -->|"15 ngày - 60 tuổi ✅"| Q2
    Q1 -->|"Dưới 15 ngày ⚠️<br/>(Under 15 days)"| R1["🔴 Chưa đủ tuổi<br/>(Underage)"]:::rejected
    Q1 -->|"Trên 65 tuổi ⚠️<br/>(Over 65)"| R2["🔴 Quá tuổi<br/>(Overage)"]:::rejected

    Q2["❓ Q2: Quyền lợi quan tâm?<br/>(Interested benefit?)"]:::question

    Q2 -->|"Khám/nằm viện, Toàn diện, Tư vấn ✅"| Q3
    Q2 -->|"Thai sản ⚠️<br/>(Maternity)"| R3["🔴 Chờ 635 ngày<br/>(635-day wait)"]:::rejected
    Q2 -->|"Nhân thọ / tích lũy ⚠️<br/>(Life / Savings)"| R5["🔴 Không tích lũy<br/>(No savings)"]:::rejected

    Q3["❓ Q3: Tình trạng sức khỏe?<br/>(Health status?)"]:::question

    Q3 -->|"Khỏe mạnh / Đã chữa khỏi ✅<br/>(Healthy / Cured)"| COLLECT
    Q3 -->|"Bệnh nền / mãn tính ⚠️<br/>(Pre-existing)"| R4["🔴 Chờ 365 ngày<br/>(365-day wait)"]:::rejected

    COLLECT["📋 Thu thập thông tin<br/>(Collect info)"]:::happy
    --> SUBMIT["✅ Gửi form<br/>(Submit)"]:::happy
    --> THANK["🎉 Cảm ơn<br/>(Thank you)"]:::success
    --> END_NODE["🏁 KẾT THÚC<br/>(End)"]:::start

    R1 -->|"Đồng ý ✅ (Agree)"| COLLECT
    R2 -->|"Đồng ý ✅ (Agree)"| COLLECT
    R3 -->|"Đồng ý ✅ (Agree)"| COLLECT
    R4 -->|"Đồng ý ✅ (Agree)"| COLLECT
    R5 -->|"Đồng ý ✅ (Agree)"| COLLECT

    R1 -.->|"Từ chối ❌ (Decline)"| END_NODE
    R2 -.->|"Từ chối ❌ (Decline)"| END_NODE
    R3 -.->|"Từ chối ❌ (Decline)"| END_NODE
    R4 -.->|"Từ chối ❌ (Decline)"| END_NODE
    R5 -.->|"Từ chối ❌ (Decline)"| END_NODE

    classDef start fill:#fff3cd,stroke:#ff9800,stroke-width:3px,color:#333
    classDef question fill:#e3f2fd,stroke:#1976d2,stroke-width:2px,color:#333
    classDef happy fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px,color:#333
    classDef success fill:#a5d6a7,stroke:#1b5e20,stroke-width:3px,color:#333
    classDef rejected fill:#ffcdd2,stroke:#c62828,stroke-width:2px,color:#333
```

---

## 2. Chi tiết từng bước (Detailed)

### 2.1. Luồng chính (Main Happy Path) 🟢

```mermaid
graph TD
    START["🎯 BẮT ĐẦU<br/>(Start)"]:::start
    --> Q1["❓ Người cần bảo vệ sức khỏe<br/>năm nay bao nhiêu tuổi?<br/><br/>(How old is the person<br/>who needs health protection?)"]:::question
    --> Q2["❓ Anh/chị đang quan tâm đến<br/>quyền lợi nào nhất?<br/><br/>(Which benefit are you<br/>most interested in?)"]:::question
    --> Q3["❓ Tình trạng sức khỏe hiện tại<br/>của người được bảo vệ?<br/><br/>(Current health status<br/>of the insured person?)"]:::question
    --> COLLECT["📋 THU THẬP THÔNG TIN<br/>Họ tên ｜ SĐT ｜ Tỉnh/Thành<br/><br/>(Collect: Name, Phone, Province)"]:::happy
    --> SUBMIT["✅ GỬI FORM<br/>(Submit)"]:::happy
    --> THANK["🎉 GỬI THÀNH CÔNG!<br/><br/>Cảm ơn anh/chị đã tin chọn<br/>Bảo Việt An Gia cùng chúng tôi!<br/>Chuyên viên sẽ liên hệ trong 24h.<br/><br/>(Success! Advisor contacts within 24h.)"]:::success
    --> END_NODE["🏁 KẾT THÚC<br/>(End)"]:::start

    classDef start fill:#fff3cd,stroke:#ff9800,stroke-width:3px,color:#333
    classDef question fill:#e3f2fd,stroke:#1976d2,stroke-width:2px,color:#333
    classDef happy fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px,color:#333
    classDef success fill:#a5d6a7,stroke:#1b5e20,stroke-width:3px,color:#333
```

#### Bảng tuỳ chọn (Options Table):

| Câu hỏi (Question) | Đi tiếp ✅ (Pass) | Rẽ nhánh ⚠️ (Branch) |
|---|---|---|
| **Q1** Độ tuổi (Age) | 15 ngày–10 tuổi, 10–30, 31–50, 51–60 | Dưới 15 ngày → 2.2 ｜ Trên 65 → 2.3 |
| **Q2** Quyền lợi (Benefit) | Khám/nằm viện, Toàn diện, Tư vấn | Thai sản → 2.4 ｜ Nhân thọ/tích lũy → 2.6 |
| **Q3** Sức khỏe (Health) | Khỏe mạnh, Đã chữa khỏi | Bệnh nền/mãn tính → 2.5 |

---

### 2.2. Chưa đủ tuổi (Underage) 🔴

```mermaid
graph TD
    Q1["❓ Độ tuổi?<br/>(Age?)"]:::question -->|"Dưới 15 ngày tuổi<br/>(Under 15 days)"| NOTICE["🔴 Cảm ơn anh/chị đã quan tâm!<br/><br/>Để tham gia gói bảo vệ này,<br/>bé yêu cần đủ 15 ngày tuổi.<br/><br/>Trong thời gian chờ bé cứng cáp,<br/>chúng tôi có những giải pháp tuyệt vời<br/>bảo vệ toàn diện cho ba mẹ.<br/><br/>Anh/chị có muốn chuyên viên liên hệ<br/>tư vấn giải pháp khác không?<br/><br/>(Baby must be at least 15 days old.<br/>Alternative solutions for parents available.)"]:::rejected

    NOTICE -->|"Đồng ý ✅ (Agree)"| COLLECT["📋 Thu thập thông tin"]:::happy
    NOTICE -.->|"Từ chối ❌ (Decline)"| END_NODE["🏁 Kết thúc"]:::start

    classDef question fill:#e3f2fd,stroke:#1976d2,stroke-width:2px,color:#333
    classDef rejected fill:#ffcdd2,stroke:#c62828,stroke-width:2px,color:#333
    classDef happy fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px,color:#333
    classDef start fill:#fff3cd,stroke:#ff9800,stroke-width:3px,color:#333
```

---

### 2.3. Quá tuổi (Overage) 🔴

```mermaid
graph TD
    Q1["❓ Độ tuổi?<br/>(Age?)"]:::question -->|"Trên 65 tuổi<br/>(Over 65 years)"| NOTICE["🔴 Cảm ơn anh/chị đã quan tâm!<br/><br/>Theo quy định, sản phẩm này chỉ dành cho<br/>khách hàng từ 65 tuổi trở xuống.<br/><br/>Tuy nhiên, chúng tôi có nhiều gói bảo vệ<br/>ưu việt cho các thành viên khác trong gia đình.<br/><br/>Anh/chị có muốn tham khảo cho người thân không?<br/><br/>(Product only available for age 65 and under.<br/>Packages for other family members available.)"]:::rejected

    NOTICE -->|"Đồng ý ✅ (Agree)"| COLLECT["📋 Thu thập thông tin"]:::happy
    NOTICE -.->|"Từ chối ❌ (Decline)"| END_NODE["🏁 Kết thúc"]:::start

    classDef question fill:#e3f2fd,stroke:#1976d2,stroke-width:2px,color:#333
    classDef rejected fill:#ffcdd2,stroke:#c62828,stroke-width:2px,color:#333
    classDef happy fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px,color:#333
    classDef start fill:#fff3cd,stroke:#ff9800,stroke-width:3px,color:#333
```

---

### 2.4. Thai sản (Maternity) 🔴

```mermaid
graph TD
    Q2["❓ Quyền lợi?<br/>(Benefit?)"]:::question -->|"Kế hoạch thai sản<br/>(Maternity plan)"| NOTICE["🔴 Chúc mừng kế hoạch của gia đình mình!<br/><br/>Lưu ý nhỏ: Quyền lợi thai sản có<br/>thời gian chờ là 635 ngày<br/>(từ lúc tham gia đến khi sinh bé).<br/><br/>Việc lên lộ trình sớm sẽ mang lại<br/>nhiều quyền lợi tốt hơn.<br/><br/>Anh/chị có muốn nhận tư vấn chi tiết không?<br/><br/>(Maternity benefits require<br/>a 635-day waiting period.)"]:::rejected

    NOTICE -->|"Đồng ý ✅ (Agree)"| COLLECT["📋 Thu thập thông tin"]:::happy
    NOTICE -.->|"Từ chối ❌ (Decline)"| END_NODE["🏁 Kết thúc"]:::start

    classDef question fill:#e3f2fd,stroke:#1976d2,stroke-width:2px,color:#333
    classDef rejected fill:#ffcdd2,stroke:#c62828,stroke-width:2px,color:#333
    classDef happy fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px,color:#333
    classDef start fill:#fff3cd,stroke:#ff9800,stroke-width:3px,color:#333
```

---

### 2.5. Bệnh nền / mãn tính (Pre-existing Conditions) 🔴

```mermaid
graph TD
    Q3["❓ Sức khỏe?<br/>(Health?)"]:::question -->|"Bệnh lý nền / mãn tính<br/>(Pre-existing / chronic)"| NOTICE["🔴 Cảm ơn anh/chị đã chia sẻ chân thành!<br/><br/>Lưu ý nhỏ: Đối với các bệnh lý nền/mãn tính,<br/>quyền lợi bảo hiểm sẽ áp dụng<br/>thời gian chờ là 365 ngày.<br/><br/>Nếu anh/chị đồng ý với quy định này<br/>và muốn thiết kế gói bảo vệ phù hợp nhất,<br/>chuyên viên của chúng tôi luôn sẵn sàng hỗ trợ!<br/><br/>Anh/chị có muốn nhận tư vấn không?<br/><br/>(Pre-existing conditions require<br/>a 365-day waiting period.)"]:::rejected

    NOTICE -->|"Đồng ý ✅ (Agree)"| COLLECT["📋 Thu thập thông tin"]:::happy
    NOTICE -.->|"Từ chối ❌ (Decline)"| END_NODE["🏁 Kết thúc"]:::start

    classDef question fill:#e3f2fd,stroke:#1976d2,stroke-width:2px,color:#333
    classDef rejected fill:#ffcdd2,stroke:#c62828,stroke-width:2px,color:#333
    classDef happy fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px,color:#333
    classDef start fill:#fff3cd,stroke:#ff9800,stroke-width:3px,color:#333
```

---

### 2.6. Nhân thọ / tích lũy (Life Insurance / Savings) 🔴

```mermaid
graph TD
    Q2["❓ Quyền lợi?<br/>(Benefit?)"]:::question -->|"Nhân thọ, tích lũy<br/>(Life insurance / Savings)"| NOTICE["🔴 Cảm ơn anh/chị đã quan tâm!<br/><br/>Sản phẩm này là Thẻ Bảo Hiểm Sức Khỏe<br/>chuyên chi trả chi phí y tế<br/>(không có giá trị tích lũy).<br/><br/>Nếu anh/chị vẫn muốn ưu tiên<br/>bảo vệ sức khỏe, chuyên viên của chúng tôi<br/>luôn sẵn sàng hỗ trợ!<br/><br/>Anh/chị có muốn nhận tư vấn<br/>thẻ sức khỏe không?<br/><br/>(Health insurance card only,<br/>no savings/investment value.)"]:::rejected

    NOTICE -->|"Đồng ý ✅ (Agree)"| COLLECT["📋 Thu thập thông tin"]:::happy
    NOTICE -.->|"Từ chối ❌ (Decline)"| END_NODE["🏁 Kết thúc"]:::start

    classDef question fill:#e3f2fd,stroke:#1976d2,stroke-width:2px,color:#333
    classDef rejected fill:#ffcdd2,stroke:#c62828,stroke-width:2px,color:#333
    classDef happy fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px,color:#333
    classDef start fill:#fff3cd,stroke:#ff9800,stroke-width:3px,color:#333
```
