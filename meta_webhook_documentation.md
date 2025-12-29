# Meta Webhook - Technical Documentation
# Meta Webhook - Tài Liệu Kỹ Thuật

> **Version / Phiên bản**: 2.0  
> **Last Updated / Cập nhật**: 2025-12-29  
> **Author / Tác giả**: Hieu Le ALCV

---

## 📋 Table of Contents / Mục Lục

1. [Overview / Tổng Quan](#1-overview--tổng-quan)
2. [Architecture / Kiến Trúc](#2-architecture--kiến-trúc)
3. [Features / Tính Năng](#3-features--tính-năng)
4. [Installation / Cài Đặt](#4-installation--cài-đặt)
5. [Configuration / Cấu Hình](#5-configuration--cấu-hình)
6. [API Reference / Tham Khảo API](#6-api-reference--tham-khảo-api)
7. [Workflow Diagrams / Sơ Đồ Luồng](#7-workflow-diagrams--sơ-đồ-luồng)
8. [Troubleshooting / Xử Lý Lỗi](#8-troubleshooting--xử-lý-lỗi)
9. [Development Guide / Hướng Dẫn Phát Triển](#9-development-guide--hướng-dẫn-phát-triển)

---

## 1. Overview / Tổng Quan

### English
This Google Apps Script acts as a webhook receiver for Meta (Facebook) Lead Forms. When a user submits a lead form on Facebook, Meta sends the data to this script, which then saves it to a Google Sheet in real-time.

### Tiếng Việt
Google Apps Script này hoạt động như một webhook receiver cho Meta (Facebook) Lead Forms. Khi người dùng submit lead form trên Facebook, Meta gửi dữ liệu đến script này, sau đó lưu vào Google Sheet theo thời gian thực.

### Key Information / Thông Tin Chính

| Item | Value |
|------|-------|
| Script File | `meta_webhook.js` |
| Target Sheet | `[SHEET_URL_MASKED]` |
| Verify Token | `[VERIFY_TOKEN_MASKED]` |
| Facebook Page | `[PAGE_NAME_MASKED]` |
| Page ID | `[PAGE_ID_MASKED]` |
| App ID | `[APP_ID_MASKED]` |

---

## 2. Architecture / Kiến Trúc

### System Flow / Luồng Hệ Thống

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Facebook      │     │  Google Apps    │     │  Google Sheet   │
│   Lead Form     │────▶│    Script      │────▶│    (Leads)      │
│                 │     │  (Webhook)      │     │                 │
└─────────────────┘     └─────────────────┘     └─────────────────┘
        │                       │                       │
        │   POST webhook        │   appendRow()         │
        │   (JSON payload)      │   (Data saved)        │
        ▼                       ▼                       ▼
   User submits          Script processes         Lead appears
   lead form             & validates data         in Sheet
```

### Component Diagram / Sơ Đồ Thành Phần

```
meta_webhook.js
├── CONFIG                    # Configuration object / Đối tượng cấu hình
├── SCRIPT_STATE              # State management keys / Khóa quản lý trạng thái
├── Webhook Handlers          # HTTP request handlers / Xử lý HTTP request
│   ├── doGet()              # GET requests (verification) / Xử lý GET
│   └── doPost()             # POST requests (leads) / Xử lý POST
├── Data Processing           # Data handling / Xử lý dữ liệu
│   ├── processLeadData_()   # Main processor / Xử lý chính
│   ├── extractFieldValue_() # Field extraction / Trích xuất field
│   └── normalizePhoneNumber_() # Phone normalization / Chuẩn hóa SĐT
├── Control Functions         # Script control / Điều khiển script
│   ├── pauseScript()        # Pause webhook / Tạm dừng
│   ├── resumeScript()       # Resume webhook / Tiếp tục
│   └── emergencyStop()      # Emergency stop / Dừng khẩn cấp
├── Logging & Monitoring      # Logging system / Hệ thống log
│   ├── logEvent_()          # Log events / Ghi log
│   └── viewRecentLogs()     # View logs / Xem logs
└── Utility Functions         # Helper functions / Hàm hỗ trợ
    ├── testConnection()     # Test connectivity / Test kết nối
    ├── testAddSampleLead()  # Add sample lead / Thêm lead mẫu
    └── healthCheck()        # System health / Kiểm tra sức khỏe
```

---

## 3. Features / Tính Năng

### Core Features / Tính Năng Chính

| Feature | Description (EN) | Mô tả (VI) | Status |
|---------|------------------|------------|--------|
| Webhook Receiver | Receives POST from Meta | Nhận POST từ Meta | ✅ |
| Auto-save to Sheet | Saves leads automatically | Tự động lưu leads | ✅ |
| Duplicate Check | Prevents duplicate leads | Kiểm tra trùng lặp | ✅ |
| Rate Limiting | Limits requests/minute | Giới hạn request/phút | ✅ |
| Logging | Detailed event logging | Ghi log chi tiết | ✅ |
| Pause/Resume | Control script state | Điều khiển trạng thái | ✅ |
| Email Alerts | Notify on new leads | Thông báo email | ⚙️ Optional |

### Feature Flags / Cờ Tính Năng

```javascript
CONFIG.FEATURES = {
  ENABLE_LOGGING: true,           // Log to sheet / Ghi log vào sheet
  ENABLE_DUPLICATE_CHECK: true,   // Check duplicates / Kiểm tra trùng
  ENABLE_EMAIL_ALERT: false,      // Email notifications / Thông báo email
  ENABLE_RATE_LIMIT: true         // Rate limiting / Giới hạn request
};
```

---

## 4. Installation / Cài Đặt

### Step 1: Create Apps Script / Bước 1: Tạo Apps Script

1. Open Google Sheet / Mở Google Sheet
2. Extensions → Apps Script
3. Delete default code / Xóa code mặc định
4. Paste `meta_webhook.js` content / Paste nội dung file
5. Save with name `meta_webhook` / Lưu với tên `meta_webhook`

### Step 2: Deploy Web App / Bước 2: Deploy Web App

1. Click **Deploy** → **New deployment**
2. Select type: **Web app**
3. Configure / Cấu hình:
   - Execute as: **Me**
   - Who has access: **Anyone**
4. Click **Deploy**
5. Copy webhook URL / Copy URL webhook

### Step 3: Configure Meta / Bước 3: Cấu hình Meta

1. Go to [developers.facebook.com](https://developers.facebook.com)
2. Select your App / Chọn App của bạn
3. Add **Webhooks** product / Thêm sản phẩm Webhooks
4. Configure / Cấu hình:
   - Callback URL: `[Your webhook URL]`
   - Verify Token: `[Your VERIFY_TOKEN from CONFIG]`
5. Subscribe to **leadgen** field / Subscribe field leadgen

---

## 5. Configuration / Cấu Hình

### CONFIG Object / Đối Tượng CONFIG

```javascript
const CONFIG = {
  // Sheet Settings / Cài đặt Sheet
  // 🔐 MASKED - Check script for actual values
  SPREADSHEET_ID: '[SPREADSHEET_ID_MASKED]',
  SHEET_NAME: 'Leads',
  LOG_SHEET_NAME: 'Logs',
  
  // Security / Bảo mật
  // 🔐 MASKED - Do not share publicly
  VERIFY_TOKEN: '[VERIFY_TOKEN_MASKED]',
  
  // Rate Limiting / Giới hạn tốc độ
  RATE_LIMIT: {
    MAX_REQUESTS_PER_MINUTE: 60
  },
  
  // Email Alerts / Thông báo Email
  ALERT_EMAIL: '[EMAIL_MASKED]'
};
```

### Sensitive Data Locations / Vị Trí Dữ Liệu Nhạy Cảm

| Data Type | Location | Description |
|-----------|----------|-------------|
| `SPREADSHEET_ID` | CONFIG | Google Sheet ID |
| `VERIFY_TOKEN` | CONFIG | Meta verification token |
| `ALERT_EMAIL` | CONFIG | Notification email |
| `PAGE_ID` | Meta Dashboard | Facebook Page ID |
| `APP_ID` | Meta Dashboard | Meta App ID |
| `APP_SECRET` | Meta Dashboard | Meta App Secret |
| `ACCESS_TOKEN` | Graph API | Page Access Token |

### Sheet Columns / Các Cột Sheet

| Column | Description (EN) | Mô tả (VI) |
|--------|------------------|------------|
| Timestamp | When lead received | Thời gian nhận lead |
| Lead ID | Unique Meta lead ID | ID lead duy nhất |
| Form ID | Lead form ID | ID form |
| Form Name | Lead form name | Tên form |
| Page ID | Facebook Page ID | ID Page |
| Page Name | Facebook Page name | Tên Page |
| Ad ID | Advertisement ID | ID quảng cáo |
| Họ và tên | Full name | Full name |
| Email | Email address | Địa chỉ email |
| Số điện thoại | Phone number | Phone number |
| Thành phố | City | City |
| Status | Lead status | Trạng thái lead |
| Raw Data | Original JSON | JSON gốc |

---

## 6. API Reference / Tham Khảo API

### Control Functions / Hàm Điều Khiển

| Function | Purpose (EN) | Mục đích (VI) |
|----------|--------------|---------------|
| `pauseScript()` | Stop receiving webhooks | Dừng nhận webhook |
| `resumeScript()` | Resume receiving webhooks | Tiếp tục nhận webhook |
| `emergencyStop()` | Emergency shutdown | Dừng khẩn cấp |
| `getScriptStatus()` | Get current status | Lấy trạng thái hiện tại |

### Testing Functions / Hàm Test

| Function | Purpose (EN) | Mục đích (VI) |
|----------|--------------|---------------|
| `testConnection()` | Test Sheet connection | Test kết nối Sheet |
| `testAddSampleLead()` | Add one sample lead | Thêm 1 lead mẫu |
| `testAddMultipleLeads(n)` | Add n sample leads | Thêm n leads mẫu |
| `clearTestLeads()` | Remove test leads | Xóa leads test |

### Utility Functions / Hàm Tiện Ích

| Function | Purpose (EN) | Mục đích (VI) |
|----------|--------------|---------------|
| `initialSetup()` | First-time setup | Cài đặt lần đầu |
| `healthCheck()` | System health check | Kiểm tra sức khỏe |
| `getLeadStats()` | Get lead statistics | Lấy thống kê leads |
| `exportLeadsToJson()` | Export leads as JSON | Xuất leads ra JSON |
| `viewRecentLogs(n)` | View last n logs | Xem n logs gần nhất |
| `clearLogs()` | Clear all logs | Xóa tất cả logs |

---

## 7. Workflow Diagrams / Sơ Đồ Luồng

### Main Webhook Flow / Luồng Webhook Chính

```
┌──────────────────────────────────────────────────────────────────┐
│                    WEBHOOK PROCESSING FLOW                        │
│                    LUỒNG XỬ LÝ WEBHOOK                            │
└──────────────────────────────────────────────────────────────────┘

     ┌─────────┐
     │  START  │
     └────┬────┘
          │
          ▼
    ┌───────────┐     Yes    ┌─────────────┐
    │ Is Paused?│──────────▶│ Return 503  │
    └─────┬─────┘            └─────────────┘
          │ No
          ▼
    ┌───────────┐     Yes    ┌─────────────┐
    │Rate Limit?│──────────▶│ Return 429  │
    └─────┬─────┘            └─────────────┘
          │ No
          ▼
    ┌───────────┐     No     ┌─────────────┐
    │Valid JSON?│──────────▶│ Return 400  │
    └─────┬─────┘            └─────────────┘
          │ Yes
          ▼
    ┌───────────┐     No     ┌─────────────┐
    │ Leadgen?  │──────────▶│   Ignore    │
    └─────┬─────┘            └─────────────┘
          │ Yes
          ▼
    ┌───────────┐     Yes    ┌─────────────┐
    │ Duplicate?│──────────▶│    Skip     │
    └─────┬─────┘            └─────────────┘
          │ No
          ▼
    ┌───────────┐
    │ Save Lead │
    └─────┬─────┘
          │
          ▼
    ┌───────────┐
    │  Log &    │
    │  Return   │
    └─────┬─────┘
          │
          ▼
     ┌─────────┐
     │   END   │
     └─────────┘
```

### Verification Flow / Luồng Xác Thực

```
Meta sends GET request          Script receives
Meta gửi GET request            Script nhận
        │                             │
        ▼                             ▼
┌─────────────────┐          ┌─────────────────┐
│ hub.mode =      │          │ Check verify    │
│ subscribe       │ ───────▶│ token matches   │
│ hub.verify_token│          │ CONFIG token    │
│ hub.challenge   │          └────────┬────────┘
└─────────────────┘                   │
                                      ▼
                              ┌───────────────┐
                      Yes ◀───│   Match?      │───▶ No
                              └───────────────┘
                                      │
                    ┌─────────────────┴─────────────────┐
                    ▼                                   ▼
            ┌───────────────┐                  ┌───────────────┐
            │ Return        │                  │ Return        │
            │ hub.challenge │                  │ Error 401     │
            └───────────────┘                  └───────────────┘
                    │                                   │
                    ▼                                   ▼
            ✅ Verified!                        ❌ Failed!
```

---

## 8. Troubleshooting / Xử Lý Lỗi

### Common Errors / Lỗi Thường Gặp

| Error | Cause (EN) | Nguyên nhân (VI) | Solution / Giải pháp |
|-------|------------|------------------|----------------------|
| 503 Service Unavailable | Script paused | Script bị tạm dừng | Run `resumeScript()` |
| 429 Too Many Requests | Rate limit exceeded | Vượt giới hạn request | Wait 1 minute / Đợi 1 phút |
| 400 Invalid JSON | Bad request format | Format request sai | Check webhook config |
| Invalid verification | Token mismatch | Token không khớp | Check VERIFY_TOKEN |

### Debug Steps / Các Bước Debug

1. **Check status / Kiểm tra trạng thái**
   ```javascript
   getScriptStatus();  // View current state
   ```

2. **View logs / Xem logs**
   ```javascript
   viewRecentLogs(50);  // View last 50 logs
   ```

3. **Health check / Kiểm tra sức khỏe**
   ```javascript
   healthCheck();  // Full system check
   ```

4. **Test connection / Test kết nối**
   ```javascript
   testConnection();  // Test Sheet access
   ```

---

## 9. Development Guide / Hướng Dẫn Phát Triển

### Adding New Features / Thêm Tính Năng Mới

#### Example: Filter by Form ID / Ví dụ: Lọc theo Form ID

```javascript
// In processLeadData_() function
// Trong hàm processLeadData_()

const ALLOWED_FORMS = ['FORM_001', 'FORM_002'];

if (!ALLOWED_FORMS.includes(leadData.form_id)) {
  logEvent_('FILTERED', 'Form not in whitelist: ' + leadData.form_id);
  return false;
}
```

#### Example: Custom Field Mapping / Ví dụ: Mapping Field Tùy Chỉnh

```javascript
// Add custom field extraction
// Thêm trích xuất field tùy chỉnh

function extractCustomField_(leadData, customFieldName) {
  if (leadData.field_data) {
    const field = leadData.field_data.find(f => 
      f.name.toLowerCase() === customFieldName.toLowerCase()
    );
    return field ? field.values.join(', ') : '';
  }
  return '';
}
```

### Environment Variables / Biến Môi Trường

For production, consider using Script Properties instead of hardcoded values:

```javascript
// Set property (run once)
PropertiesService.getScriptProperties().setProperty('VERIFY_TOKEN', 'your_token');

// Get property (in CONFIG)
const VERIFY_TOKEN = PropertiesService.getScriptProperties().getProperty('VERIFY_TOKEN');
```

### Deployment Checklist / Checklist Triển Khai

- [ ] Update CONFIG.SPREADSHEET_ID
- [ ] Update CONFIG.VERIFY_TOKEN
- [ ] Run `initialSetup()`
- [ ] Deploy as Web App
- [ ] Configure Meta webhook
- [ ] Test with `testAddSampleLead()`
- [ ] Verify in Google Sheet
- [ ] Switch to Live mode (if needed)
- [ ] **Remove sensitive data from documentation**

### Version History / Lịch Sử Phiên Bản

| Version | Date | Changes (EN) | Thay đổi (VI) |
|---------|------|--------------|---------------|
| 1.0 | 2025-12-29 | Initial release | Phiên bản đầu tiên |
| 2.0 | 2025-12-29 | Added security, logging, controls | Thêm bảo mật, logging, điều khiển |
| 2.1 | 2025-12-29 | Masked sensitive data in docs | Mã hóa dữ liệu nhạy cảm |

---

## 🔐 Security Best Practices / Thực Hành Bảo Mật

1. **Never commit sensitive data to version control**  
   Không bao giờ commit dữ liệu nhạy cảm vào version control

2. **Use Script Properties for secrets**  
   Sử dụng Script Properties cho các secret

3. **Rotate VERIFY_TOKEN periodically**  
   Thay đổi VERIFY_TOKEN định kỳ

4. **Monitor logs for suspicious activity**  
   Theo dõi logs để phát hiện hoạt động đáng ngờ

5. **Limit access to Google Sheet**  
   Giới hạn quyền truy cập Google Sheet

---

## 📞 Support / Hỗ Trợ

For issues or questions / Nếu có vấn đề hoặc câu hỏi:
1. Check logs: `viewRecentLogs(100)`
2. Run health check: `healthCheck()`
3. Review this documentation / Xem lại tài liệu này

---

*Document generated on 2025-12-29 | Sensitive data masked for security*

