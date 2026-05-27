# System Design cho Marketing: Từ Nguyên Tắc Đến Thực Hành

## Giới Thiệu

Tài liệu này hướng dẫn **cách suy nghĩ như một engineer** khi thiết kế hệ thống phần mềm. Bạn sẽ hiểu **tại sao** kỹ sư lại chọn cách này thay vì cách khác, **khi nào** cần nâng cấp hệ thống, và **làm thế nào** để đề xuất tính năng mà không làm sập server.

Mục đích: Trang bị cho bạn **tư duy System Design** để giao tiếp hiệu quả với tech team và đưa ra quyết định business thông minh.

---

## Phần 1: Ba Trụ Cột Cơ Bản

### 1.1 Scalability (Khả Năng Mở Rộng)

**Định nghĩa:** Hệ thống có thể xử lý được nhu cầu tăng lên (người dùng, dữ liệu, tải) mà không cần viết lại code từ đầu.

**Ví dụ thực tế:**
- Tháng trước: 1,000 leads/ngày
- Tháng này: 10,000 leads/ngày (campaign quốc gia)
- Hệ thống có **chịu được** không?

**Hai cách scale:**

**a) Vertical Scaling (Mua máy mạnh hơn)**
- Upgrade server từ 4GB RAM → 16GB RAM
- Nhanh, đơn giản, nhưng **giới hạn** (máy không thể to vô tận)
- Chi phí: cao từ một điểm nào đó

**b) Horizontal Scaling (Mua nhiều máy nhỏ hơn)**
- 1 máy xử lý 1,000 leads → 10 máy xử lý 10,000 leads
- Linh hoạt hơn, nhưng **phức tạp** (quản lý 10 máy cùng lúc)
- Chi phí: rẻ hơn, nhưng cần kiến trúc tốt

**Nguyên tắc suy nghĩ:**
> "Nếu hôm nay 1,000 leads/ngày gây sập server, thì chiến dịch quốc gia 100,000 leads sẽ là thảm họa. Phải thiết kế từ đầu với tâm thế: 'Hôm nay small, hôm nay có thể big'."

### 1.2 Reliability (Độ Tin Cậy)

**Định nghĩa:** Hệ thống hoạt động ổn định, ngay cả khi có lỗi xảy ra. Người dùng có thể tin tưởng.

**Ví dụ thực tế:**
- Database bị lỗi → Lead vẫn được lưu (không mất)
- Server 1 down → Server 2 tự động nhận việc (failover)
- Mất điện → Dữ liệu vẫn an toàn (backup)

**Ba nguyên tắc:**

**a) Redundancy (Dự Phòng)**
- Không tin tưởng 1 chiếc máy → dùng 2-3 chiếc
- Nếu 1 cái chết, cái khác thay thế
- Chi phí: gấp đôi, nhưng giảm risk từ 100% → 1%

**b) Monitoring (Giám Sát)**
- Luôn theo dõi: Server có chạy? Database có phản hồi? API có timeout?
- Phát hiện sớm → sửa nhanh
- Nếu không monitor, chỉ biết sập khi khách hàng gọi

**c) Graceful Degradation (Giảm Chất Lượng Có Kiểm Soát)**
- Khi quá tải, hệ thống không sập → thay vào đó nó **chậm lại**
- Ví dụ: Lead form vẫn hoạt động nhưng mất 5 giây để submit thay vì 1 giây
- Tốt hơn sập hoàn toàn

### 1.3 Maintainability (Khả Năng Bảo Trì)

**Định nghĩa:** Code dễ hiểu, dễ sửa, dễ thêm tính năng mới mà không làm hỏng cái cũ.

**Ví dụ thực tế:**
- Bạn muốn thêm validation: "Lead từ tỉnh Đắk Nông bắt buộc có SĐT"
- Nếu code tốt: 5 phút sửa xong
- Nếu code tệ: 2 ngày tìm chỗ thay đổi + test + lo sợ sập hệ thống

**Ba nguyên tắc:**

**a) Separation of Concerns (Tách Biệt Trách Nhiệm)**
- Mỗi phần code chỉ làm MỘT việc
- Database chỉ lưu dữ liệu
- Business logic chỉ xử lý quy tắc
- API chỉ giao tiếp
- Khi sửa Database, không ảnh hưởng Business logic

**b) DRY - Don't Repeat Yourself**
- Nếu code giống nhau xuất hiện 3 lần → ghép lại 1 chỗ
- Thay đổi sau này chỉ cần sửa 1 chỗ

**c) Documentation (Tài Liệu)**
- Giải thích **tại sao** chọn cách này (không phải **cái gì**)
- Giúp engineer mới onboard nhanh
- Tránh vấn đề: "Tại sao code lại kiểu này? Có bug không?"

---

## Phần 2: Kiến Trúc Cơ Bản (Layered Architecture)

**Ý tưởng:** Chia hệ thống thành các tầng, mỗi tầng có công việc riêng.

```
┌─────────────────────────────────┐
│   Presentation Layer (UI)       │  ← Form Lead, Dashboard
│   (Người dùng nhìn thấy)        │
├─────────────────────────────────┤
│   API Layer (Gateway)           │  ← Tiếp nhận request từ UI
│   (Kiểm soát cửa ra vào)        │
├─────────────────────────────────┤
│   Business Logic Layer          │  ← Quy tắc: validate, scoring, assign
│   (Não của hệ thống)            │
├─────────────────────────────────┤
│   Data Access Layer             │  ← Nói chuyện với Database
│   (Cầu nối dữ liệu)             │
├─────────────────────────────────┤
│   Database Layer                │  ← Lưu trữ dữ liệu
│   (Kho chứa)                    │
└─────────────────────────────────┘
```

**Ví dụ: Lead từ form được xử lý**

1. **UI:** Người dùng điền form lead (tên, SĐT, địa chỉ) → Click "Gửi"
2. **API Layer:** Nhận request, kiểm tra: "Dữ liệu đầy đủ không? Format đúng không?"
3. **Business Logic:** Apply rules: "Lead từ Hà Nội và có bảo hiểm sức khỏe → Assign cho sale A"
4. **Data Access:** "Lưu lead vào database"
5. **Database:** Dữ liệu được lưu trữ

Sau đó, **ngược chiều**: Database → Data Access → Business Logic → API → UI (Dashboard cập nhật lead mới).

**Lợi ích:**
- Thay đổi UI không ảnh hưởng Database
- Thay đổi quy tắc scoring không cần touch Database
- Dễ test từng layer riêng

---

## Phần 3: Các Mẫu Thiết Kế Thường Dùng

### 3.1 Repository Pattern (Tổ Chức Dữ Liệu)

**Ý tưởng:** Thay vì code truy vấn database rải rác khắp nơi, tập trung vào một "Repository" (kho lưu trữ).

**Ví dụ:**

```python
# ❌ KHÔNG NÊN: Truy vấn database ở nhiều chỗ
class LeadForm:
    def submit_lead(self, data):
        # Database connection ở đây
        db.execute("INSERT INTO leads ...")
        # Nếu thay đổi cách lưu, phải sửa ở 10 chỗ

# ✅ NÊN: Tập trung vào Repository
class LeadRepository:
    """Đây là nơi duy nhất quản lý lead trong database"""
    def save(self, lead):
        db.execute("INSERT INTO leads ...")
    
    def get_by_phone(self, phone):
        return db.query("SELECT * FROM leads WHERE phone = ...", phone)
    
    def update_status(self, lead_id, status):
        db.execute("UPDATE leads SET status = ...", lead_id, status)

class LeadForm:
    def __init__(self, repo: LeadRepository):
        self.repo = repo
    
    def submit_lead(self, data):
        # Chỉ cần gọi Repository
        self.repo.save(data)
```

**Lợi ích:**
- Khi thay đổi cách lưu database, chỉ sửa LeadRepository (1 chỗ)
- LeadForm không cần biết chi tiết database hoạt động như nào
- Dễ test: có thể mock Repository để test LeadForm mà không cần database thực

### 3.2 Service Layer (Xử Lý Logic)

**Ý tưởng:** Tách logic xử lý phức tạp ra một layer riêng, tái sử dụng ở nhiều chỗ.

```python
class LeadScoringService:
    """Xử lý logic scoring lead"""
    def __init__(self, repo: LeadRepository):
        self.repo = repo
    
    def score_lead(self, lead_data):
        """Tính điểm lead dựa trên các quy tắc"""
        score = 0
        
        # Rule 1: Lead từ TP.HCM, Hà Nội → +20 điểm
        if lead_data['province'] in ['HCMC', 'Hanoi']:
            score += 20
        
        # Rule 2: Đã có bảo hiểm → +30 điểm
        if lead_data['has_insurance']:
            score += 30
        
        # Rule 3: SĐT được xác minh → +15 điểm
        if lead_data['phone_verified']:
            score += 15
        
        return score
    
    def assign_to_sales(self, lead_id):
        """Tìm sale phù hợp và assign"""
        lead = self.repo.get_by_id(lead_id)
        score = self.score_lead(lead)
        
        # Rule: Nếu score > 50 → assign cho sale A (chuyên handle hot leads)
        # Nếu 20-50 → assign cho sale B
        # Nếu < 20 → assign cho sale C (training)
        
        if score > 50:
            assigned_to = 'Sale A'
        elif score > 20:
            assigned_to = 'Sale B'
        else:
            assigned_to = 'Sale C'
        
        self.repo.update_status(lead_id, f'assigned_to_{assigned_to}')
        return assigned_to

# Sử dụng
scoring_service = LeadScoringService(repo)
assigned_to = scoring_service.assign_to_sales(lead_id=123)
print(f"Lead 123 được assign cho {assigned_to}")
```

**Logic giải thích:**
1. `score_lead()`: Tính điểm lead dựa trên 3 quy tắc
   - Nếu từ TP.HCM hoặc Hà Nội → cộng 20 điểm
   - Nếu đã có bảo hiểm → cộng 30 điểm
   - Nếu SĐT được xác minh → cộng 15 điểm
2. `assign_to_sales()`: Dựa trên điểm để assign cho sales phù hợp
   - Điểm cao (>50) → Sales A (chuyên gia)
   - Điểm trung bình (20-50) → Sales B
   - Điểm thấp (<20) → Sales C (learning)

**Lợi ích:**
- Khi muốn thay đổi quy tắc scoring (ví dụ: thêm +10 cho lead từ Đà Nẵng), chỉ cần sửa `score_lead()`
- `assign_to_sales()` có thể được dùng ở nhiều chỗ (form submit, bulk import, admin panel)
- Dễ test logic: mock dữ liệu rồi kiểm tra scoring có đúng không

### 3.3 Caching Pattern (Lưu Tạm)

**Ý tưởng:** Dữ liệu hay được dùng → lưu vào bộ nhớ nhanh, giảm query database.

```python
class LeadRepository:
    def __init__(self, db, cache):
        self.db = db
        self.cache = cache  # Redis hoặc memory cache
    
    def get_by_phone(self, phone):
        """Lấy lead theo SĐT (có cache)"""
        
        # Bước 1: Kiểm tra cache (rất nhanh, milliseconds)
        cached_lead = self.cache.get(f'lead:{phone}')
        if cached_lead:
            return cached_lead  # Trả về ngay
        
        # Bước 2: Nếu không có trong cache, query database (chậm, hàng trăm ms)
        lead = self.db.query(f"SELECT * FROM leads WHERE phone = {phone}")
        
        # Bước 3: Lưu vào cache để lần sau nhanh hơn
        self.cache.set(f'lead:{phone}', lead, expire_in=3600)  # Lưu 1 giờ
        
        return lead
    
    def update_lead(self, phone, new_data):
        """Cập nhật lead (phải xóa cache)"""
        
        # Bước 1: Cập nhật database
        self.db.execute(f"UPDATE leads SET ... WHERE phone = {phone}")
        
        # Bước 2: Xóa cache cũ (vì dữ liệu đã thay đổi)
        self.cache.delete(f'lead:{phone}')
```

**Logic giải thích:**
1. **First call:** Không có cache → query DB (chậm) → lưu cache
2. **Second call:** Có cache → trả về ngay (nhanh)
3. **Update:** Phải xóa cache để lần sau lấy dữ liệu mới

**Ví dụ thực tế:**
- Lead A gọi 100 lần trong 1 giờ → Query DB 1 lần + trả về từ cache 99 lần
- Tiết kiệm 99 × 200ms = ~20 giây response time
- Khi update lead A → xóa cache → call tiếp theo query DB lại → cache dữ liệu mới

**Lợi ích:**
- Giảm tải database
- Response time nhanh hơn
- Hệ thống chịu được traffic spike tốt hơn

### 3.4 Asynchronous Processing (Xử Lý Không Đợi)

**Ý tưởng:** Không phải mọi việc đều cần làm ngay. Một số việc có thể làm sau khi user nhận được phản hồi.

```python
# ❌ KHÔNG NÊN: Làm tất cả đồng thời (lead form bị hang)
class LeadForm:
    def submit_lead(self, data):
        # 1. Lưu lead (nhanh: 100ms)
        lead = self.repo.save(data)
        
        # 2. Gửi email (chậm: 5 giây)
        self.email_service.send_confirmation(data['email'])
        
        # 3. Gửi SMS (chậm: 3 giây)
        self.sms_service.send(data['phone'])
        
        # 4. Gọi API bên thứ 3 (chậm: 10 giây)
        self.crm_service.sync_lead(lead)
        
        # Tổng cộng: 100 + 5000 + 3000 + 10000 = 18 giây
        # User phải chờ 18 giây để form trả về! 😞
        
        return {'status': 'success'}

# ✅ NÊN: Tách việc đơn giản ra, việc phức tạp xử lý sau
class LeadForm:
    def submit_lead(self, data):
        # 1. Lưu lead (nhanh: 100ms)
        lead = self.repo.save(data)
        
        # 2. Đẩy các task phức tạp vào queue (nhanh: 10ms)
        self.task_queue.add_job('send_confirmation_email', data['email'])
        self.task_queue.add_job('send_sms', data['phone'])
        self.task_queue.add_job('sync_to_crm', lead.id)
        
        # Trả về ngay (tổng: 100 + 10 + 10 + 10 = 130ms)
        return {'status': 'success', 'lead_id': lead.id}

# Một worker chạy ở background, xử lý các job từ queue
class BackgroundWorker:
    def process_jobs(self):
        while True:
            job = self.task_queue.get_next_job()
            
            if job.type == 'send_confirmation_email':
                self.email_service.send_confirmation(job.data['email'])
            
            elif job.type == 'send_sms':
                self.sms_service.send(job.data['phone'])
            
            elif job.type == 'sync_to_crm':
                self.crm_service.sync_lead(job.data['lead_id'])
```

**Logic giải thích:**
1. User submit form → lưu database (phải làm ngay)
2. Sau khi lưu database, đẩy các task phức tạp vào queue:
   - Gửi email confirmation
   - Gửi SMS
   - Sync với CRM bên ngoài
3. Form trả về ngay cho user (130ms thay vì 18 giây)
4. Worker chạy ở background xử lý các task từ queue (các job sẽ hoàn thành trong vài giây tiếp theo)

**Ví dụ timeline:**
```
User submit form (t=0ms)
  ↓
Save to DB (t=0-100ms) ✓
  ↓
Add jobs to queue (t=100-130ms) ✓
  ↓
Return success to user (t=130ms) ✓ ← User nhất toàn bộ trong 130ms
  ↓
[Background worker xử lý]
Send email (t=130-5130ms) ✓
Send SMS (t=5130-8130ms) ✓
Sync CRM (t=8130-18130ms) ✓
```

**Lợi ích:**
- User experience tốt (form response nhanh)
- Hệ thống handle được traffic spike (jobs được xử lý từ từ)
- Nếu email service bị down, form vẫn hoạt động (chỉ email không gửi được)

---

## Phần 4: Red Flags - Dấu Hiệu Cảnh Báo

### 🚩 Signal 1: "Response Time Bắt Đầu Chậm"

**Dấu hiệu:**
- Form lead submit: từ 200ms → 2 giây → 10 giây
- Dashboard cập nhật lead từ real-time → delay 30 phút

**Nguyên nhân có thể:**
- Database bị quá tải (query chậm)
- Không dùng cache
- Background jobs xếp hàng dài (worker không đủ)
- Code có query DB nhiều lần trong 1 request

**Hành động:**
- Kiểm tra database logs: query nào chậm?
- Thêm cache cho dữ liệu hay dùng
- Tăng số workers xử lý background jobs
- Profile code: tìm bottleneck

### 🚩 Signal 2: "Lỗi Ngẫu Nhiên / Đôi Khi Lead Bị Mất"

**Dấu hiệu:**
- 99/100 lần form submit thành công, nhưng 1 lần lead biến mất
- Database log không thấy lead đó

**Nguyên nhân có thể:**
- Không có transaction (nếu process bị interrupt ở giữa)
- Network timeout khi gửi dữ liệu (user nghĩ gửi rồi nhưng thực tế không)
- Exception xảy ra khi lưu database nhưng code không handle

**Hành động:**
- Thêm transaction: "Hoặc lưu hết, hoặc không lưu gì"
- Thêm error logging: ghi lại khi có exception
- Thêm retry logic: nếu lần đầu fail, thử lại 3 lần
- Gửi warning email khi error xảy ra

### 🚩 Signal 3: "Không Thể Scale - Thêm Máy Không Giúp"

**Dấu hiệu:**
- Traffic tăng 2x, máy tăng từ 1 → 3, nhưng performance chỉ tăng 1.2x (thay vì 2x)

**Nguyên nhân có thể:**
- Database chỉ có 1 instance → ngợn cổ chai
- Tất cả máy đọc/ghi vào 1 database → tranh chấp resource
- Code không thread-safe (xung đột dữ liệu khi chạy song song)

**Hành động:**
- Replicate database (1 write + N read)
- Shard database (phân chia dữ liệu theo quy tắc)
- Review code để tìm race conditions

### 🚩 Signal 4: "Mỗi Lần Deploy Là Một Sứ Mệnh"

**Dấu hiệu:**
- Deploy feature nhỏ → phải backup toàn bộ database
- Deploy phải down hệ thống 30 phút
- Deploy phiên bản mới → mất 3 giờ rollback vì bug

**Nguyên nhân:**
- Không có versioning hoặc database migration plan
- Tests không đủ
- Deploy script quá phức tạp / manual

**Hành động:**
- Tự động hóa deploy (CI/CD pipeline)
- Viết comprehensive tests
- Database migration: plan trước, rollback nhanh
- Blue-green deployment: deploy song song, switch khi ready

---

## Phần 5: Case Study - Lead Generation System cho Asahi Life

### Bối Cảnh

**Hiện tại:**
- 1,000 - 5,000 leads/ngày
- 1 database server
- Web form, API cho mobile

**Tham vọng:**
- Chiến dịch quốc gia Q3 2025: 50,000+ leads/ngày
- Real-time sync với CRM Salesforce
- Lead scoring + auto assign sales
- Analytics dashboard

### Thiết Kế

```
┌─────────────────────────────────────────────────┐
│             Web Form / Mobile App               │
│              (Presentation Layer)               │
└────────────────────┬────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────┐
│             API Gateway (Load Balancer)         │
│  - Distribute requests across multiple servers  │
│  - Rate limiting: max 100 req/sec per IP        │
└────────────────────┬────────────────────────────┘
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
┌──────────────┬──────────────┬──────────────┐
│  API Server  │  API Server  │  API Server  │
│  (Node 1)    │  (Node 2)    │  (Node 3)    │  ← Horizontal scale
│              │              │              │
│ - Validate   │              │              │
│ - Business   │              │              │
│   logic      │              │              │
└──────────────┴──────┬───────┴──────────────┘
                      │
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
   ┌─────────┐  ┌────────┐  ┌──────────┐
   │  Cache  │  │ Queue  │  │ Database │
   │ (Redis) │  │(Kafka) │  │(PostgreSQL)│
   └─────────┘  └────────┘  └──────────┘
        │             │             │
        │  (Fast)  (Async)     (Reliable)
        └─────────────┴─────────────┘
                      │
        ┌─────────────┼──────────────┐
        ▼             ▼              ▼
   ┌─────────┐  ┌─────────┐  ┌────────┐
   │ Email   │  │   SMS   │  │  CRM   │
   │ Service │  │ Service │  │Salesforce│
   └─────────┘  └─────────┘  └────────┘
```

### Giải Thích Chi Tiết

**1. API Gateway (Load Balancer)**
```
Tác dụng: Chia đều request từ 50,000 leads/ngày cho 3 API servers

Ví dụ:
- Lead từ web form gửi request → API Gateway
- API Gateway phân tích: "Server 1 đã xử lý 200 leads, Server 2 xử lý 180, Server 3 xử lý 190"
- Gửi lead tới Server 2 (ít tải nhất)

Lợi ích: Không server nào quá tải, tăng throughput từ 5K → 50K leads/ngày
```

**2. Cache (Redis)**
```
Tác dụng: Lưu dữ liệu hay dùng, tránh query database mỗi lần

Ví dụ:
- Lead query "Danh sách sale ở HN" → 1000 leads/h × 5ms = 5 giây
- Nếu cache: 1000 leads/h × 0.5ms = 500ms (nhanh 10x)

Quy tắc cache:
- Lưu config (danh sách sale, province list): expire 1 ngày
- Lưu lead detail: expire 1 giờ (vì có thể bị update)
- Xóa cache ngay khi cập nhật dữ liệu
```

**3. Queue (Kafka)**
```
Tác dụng: Xử lý các task phức tạp asynchronously

Ví dụ workflow:
  t=0s: Lead submit form
  t=0.1s: Save DB + push 3 jobs vào queue (email, SMS, CRM sync)
  t=0.15s: Return success to user
  t=0.5-5s: Workers xử lý jobs từ queue (user không chờ)

Lợi ích:
- Form response nhanh
- Nếu email service down, form vẫn hoạt động
- Nếu queue overflow, system gracefully degrade (chấp nhận lead, delay notification)
```

**4. Database (PostgreSQL) - Primary + Replicas**
```
Tác dụng: Lưu trữ tin cậy + horizontal read scaling

Setup:
- Primary: Nhận tất cả writes (INSERT, UPDATE lead)
- Replica 1, 2: Read-only copies, dùng cho queries

Ví dụ:
  Write (cập nhật lead status): → Primary
  Read (lấy lead detail): → Replica 1
  Read (analytics: bao nhiêu lead hôm nay): → Replica 2

Nếu Primary down:
- Promote Replica 1 thành Primary
- Failover tự động, downtime < 1 phút

Nếu Replica down:
- Vẫn có Replica 2 để đọc
- Không ảnh hưởng writes
```

### Code Example: Lead Submission Pipeline

```python
# === API Server ===
from fastapi import FastAPI
from sqlalchemy import create_engine
import redis
import json

app = FastAPI()
db_engine = create_engine("postgresql://...")
cache = redis.Redis(host='redis-server')
queue = KafkaProducer(bootstrap_servers=['kafka-broker'])

class LeadRepository:
    def __init__(self, engine):
        self.engine = engine
    
    def save_lead(self, data):
        """Save lead to database"""
        with self.engine.connect() as conn:
            conn.execute(
                "INSERT INTO leads (name, phone, email, province, status) "
                "VALUES (:name, :phone, :email, :province, :status)",
                {
                    'name': data['name'],
                    'phone': data['phone'],
                    'email': data['email'],
                    'province': data['province'],
                    'status': 'new'
                }
            )
            conn.commit()
        return True

class LeadScoringService:
    def __init__(self, repo):
        self.repo = repo
    
    def score(self, lead_data):
        """Calculate lead score"""
        score = 0
        if lead_data['province'] in ['HCMC', 'Hanoi', 'Danang']:
            score += 20
        if lead_data.get('has_insurance'):
            score += 30
        return score
    
    def assign_sales(self, lead_id, lead_data):
        """Assign lead to sales based on score"""
        score = self.score(lead_data)
        # Rule: score > 50 → Sales A (senior), else → Sales B (junior)
        assigned_to = 'Sales A' if score > 50 else 'Sales B'
        return assigned_to

@app.post("/api/leads")
def submit_lead(data: dict):
    """Main API endpoint for lead submission"""
    
    # Step 1: Validate input
    if not all(k in data for k in ['name', 'phone', 'email']):
        return {'error': 'Missing fields'}, 400
    
    # Step 2: Save to database
    repo = LeadRepository(db_engine)
    repo.save_lead(data)
    
    # Step 3: Score and assign
    scoring_service = LeadScoringService(repo)
    assigned_to = scoring_service.assign_sales(lead_id=None, lead_data=data)
    
    # Step 4: Push async tasks to queue
    # (Không chờ các tasks này hoàn thành)
    queue.send('send_email', json.dumps({
        'email': data['email'],
        'subject': 'Lead Confirmation',
        'body': f'Hello {data["name"]}, your lead has been received.'
    }))
    queue.send('send_sms', json.dumps({
        'phone': data['phone'],
        'message': f'Hello {data["name"]}, sales {assigned_to} will contact you soon.'
    }))
    queue.send('sync_to_crm', json.dumps({
        'lead_name': data['name'],
        'lead_phone': data['phone'],
        'assigned_to': assigned_to
    }))
    
    # Step 5: Return immediately to user
    return {
        'status': 'success',
        'message': f'Lead received. {assigned_to} will contact you shortly.',
        'assigned_to': assigned_to
    }

# === Background Worker (chạy riêng) ===
class BackgroundWorker:
    def __init__(self):
        self.queue = KafkaConsumer('default', bootstrap_servers=['kafka-broker'])
    
    def process_messages(self):
        for message in self.queue:
            job = json.loads(message.value)
            
            if message.topic == 'send_email':
                # Send email (có thể lỗi, không ảnh hưởng lead submission)
                try:
                    email_service.send(job['email'], job['subject'], job['body'])
                except Exception as e:
                    logger.error(f"Email send failed: {e}")
            
            elif message.topic == 'send_sms':
                # Send SMS
                try:
                    sms_service.send(job['phone'], job['message'])
                except Exception as e:
                    logger.error(f"SMS send failed: {e}")
            
            elif message.topic == 'sync_to_crm':
                # Sync to Salesforce
                try:
                    salesforce_api.create_lead(job)
                except Exception as e:
                    logger.error(f"CRM sync failed: {e}")
                    # Retry or alert

if __name__ == '__main__':
    worker = BackgroundWorker()
    worker.process_messages()
```

**Timeline cho 50,000 leads/ngày:**

```
t=0s:    Lead A submit form gửi request
t=0.05s: Load balancer route → API Server 1
t=0.08s: Validate input ✓
t=0.12s: Save to database ✓
t=0.15s: Score lead ✓
t=0.16s: Push 3 jobs vào queue (email, SMS, CRM) ✓
t=0.17s: Return success to user ✓

[User nhất toàn bộ trong 170ms, form không hang]

Sau đó (background):
t=1s:    Worker 1 xử lý email job
t=2s:    Worker 2 xử lý SMS job
t=5s:    Worker 3 xử lý CRM sync job

(Tất cả async, không block form)
```

### Monitoring & Alerting

**Metrics cần theo dõi:**
```
1. Response Time
   - Target: 95th percentile < 500ms
   - Alert: If > 1s cho 5 phút liên tục

2. Error Rate
   - Target: < 0.1% (99.9% success)
   - Alert: If > 0.5%

3. Database
   - Query latency: Target < 100ms
   - Connection pool: Alert nếu > 80% usage

4. Queue
   - Job queue depth: Alert nếu > 10,000 pending
   - Processing lag: Alert nếu > 10 phút

5. Cache
   - Hit rate: Target > 80%
   - Memory usage: Alert nếu > 70%
```

---

## Phần 6: Bài Học & Lời Khuyên

### Nguyên Tắc Vàng

**1. "Don't Optimize Too Early"**
- Trước hết: code phải đúng
- Sau đó: code phải rõ ràng
- Cuối cùng (nếu cần): code phải nhanh

**2. "Complexity is the Enemy"**
- Đơn giản > phức tạp
- 1 hệ thống đơn giản dễ bảo trì hơn 1 hệ thống phức tạp mạnh hơn

**3. "Measure Before You Optimize"**
- Không dự đoán bottleneck → đo (profile)
- 80% thời gian thường dùng cho 20% code

**4. "Fail Fast, Learn Faster"**
- Deploy nhỏ, thường xuyên
- Phát hiện bug sớm
- Rollback nhanh nếu cần

### Để Làm Việc Tốt Với Tech Team

**Hỏi đúng câu hỏi:**

❌ Tránh: "Làm sao để system nhanh hơn?"
✅ Hỏi: "Request nào chậm nhất? Chậm bao lâu? Bao nhiêu user bị ảnh hưởng?"

❌ Tránh: "Tôi muốn feature X"
✅ Hỏi: "Feature X giải quyết problem Y cho user Z như thế nào? Timeline & resource cần bao nhiêu?"

❌ Tránh: "Tại sao lại lâu thế?"
✅ Hỏi: "Blocking point là gì? Có thể parallel hóa hay refactor để nhanh hơn không?"

### Red Flags Khi Giao Tiếp Với Engineer

- "Vẫn được, không cần refactor" → Debt tích lũy
- "Deploy lần tới sẽ fix" → Kỳ tới sẽ có bug tệ hơn
- "Không có time để viết test" → QA sẽ tốn time hơn sau
- "User không biết difference" → Người dùng biết, qua latency

---

## Tổng Kết

System Design không phải chỉ về architecture phức tạp. Đó là **cách suy nghĩ**:

1. **Hiểu vấn đề**: Bao nhiêu user? Bao nhiêu dữ liệu? Latency target?
2. **Trade-off**: Không có giải pháp hoàn hảo → chọn cái tốt nhất cho context
3. **Scale từ nhỏ**: Thiết kế để dễ mở rộng sau
4. **Monitor & Iterate**: Measure, phát hiện problem, fix, repeat

Với mindset này, bạn sẽ:
- Nói chuyện thông minh với tech team
- Đề xuất feature mà không sợ system sụp
- Nhận ra bottleneck sớm
- Tăng tốc độ launch campaign

Chúc bạn thành công! 🚀