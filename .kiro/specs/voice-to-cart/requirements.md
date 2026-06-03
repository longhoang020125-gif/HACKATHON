# Requirements: Voice-to-Cart  End-User Food Ordering Assistant

## 1. Phạm vi hệ thống

Backend C# .NET Core Web API phục vụ duy nhất luồng **đặt món của người dùng cuối (Grab Customer)** thông qua giọng nói. Hệ thống KHÔNG bao gồm bất kỳ tính năng quản lý merchant hoặc admin nào.

Luồng tổng quát:
```
Người dùng nói  LLM trích xuất entities  POST /api/cart/voice-command
 VoiceCartService khớp menu  CartSession (draft cart)
 Frontend hiển thị cart, cho phép sửa  PUT /api/cart/item
 GET /api/cart lấy trạng thái hiện tại
```

---

## 2. Yêu cầu chức năng

### REQ-001: Dữ liệu thực đơn in-memory
- Hệ thống tải `Data/merchants.json` vào RAM một lần khi khởi động (singleton).
- Dữ liệu được giữ nguyên trong suốt vòng đời ứng dụng; không ghi lại file.

### REQ-002: Quản lý phiên giỏ hàng (CartSession)
- Mỗi phiên làm việc có một `CartSession` lưu danh sách `CartItem`.
- Prototype này dùng in-memory store; session được định danh bằng `sessionId` (string).
- `CartItem` lưu: `itemId`, `merchantId`, `itemName`, `quantity`, `unitPrice`, `specialNotes`, và danh sách `selectedOptions`.
- `CartSession` tính `totalPrice` động từ tổng `(unitPrice + optionPrice)  quantity` của tất cả items.

### REQ-003: Xử lý lệnh giọng nói  VoiceCartService
Service nhận object đã được LLM parse sẵn với các field: `dishName`, `quantity`, `specialNotes`.

**Luồng xử lý:**

**a. Accurate Match (1 kết quả duy nhất)**
- Tìm thấy đúng 1 `MenuItem` khớp keyword  thêm/cập nhật item trong `CartSession`.
- Nếu item đã có trong giỏ: cộng dồn `quantity`, merge `specialNotes`.
- Trả về `VoiceCommandResult { IsAmbiguous = false, Cart = <updated CartSession> }`.

**b. Ambiguous Match (nhiều kết quả)**
- Tìm thấy  2 `MenuItem` khớp keyword  KHÔNG thêm vào giỏ.
- Trả về `VoiceCommandResult { IsAmbiguous = true, Suggestions = [danh sách MenuItem gợi ý], Cart = <cart hiện tại chưa đổi> }`.

**c. No Match**
- Không tìm thấy kết quả nào.
- Trả về `VoiceCommandResult { IsAmbiguous = false, Suggestions = [], Cart = <cart hiện tại>, Message = "Không tìm thấy món phù hợp" }`.

**Quy tắc tìm kiếm keyword:**
- Case-insensitive substring match trên `item_name`.
- Case-insensitive substring match trên bất kỳ phần tử nào trong `aliases`.
- Normalized match: loại bỏ dấu tiếng Việt trước khi so sánh.

### REQ-004: API  POST /api/cart/voice-command
- Nhận body JSON: `{ "sessionId": "string", "dishName": "string", "quantity": 1, "specialNotes": "string" }`.
- `sessionId` bắt buộc; nếu session chưa tồn tại thì tạo mới.
- `dishName` bắt buộc, không rỗng.
- `quantity` mặc định = 1 nếu không truyền.
- Trả về `VoiceCommandResult` với HTTP 200.
- HTTP 400 nếu thiếu field bắt buộc.

### REQ-005: API  PUT /api/cart/item (Correction Path)
- Nhận body: `{ "sessionId": "string", "cartItemId": "string", "quantity": 0, "specialNotes": "string" }`.
- `quantity = 0`  xóa item khỏi giỏ.
- `quantity > 0`  cập nhật số lượng.
- `specialNotes`  ghi đè ghi chú cũ.
- Trả về `CartSession` đã cập nhật với HTTP 200.
- HTTP 404 nếu `sessionId` hoặc `cartItemId` không tồn tại.

### REQ-006: API  GET /api/cart?sessionId={id}
- Trả về `CartSession` hiện tại với HTTP 200.
- HTTP 404 nếu session không tồn tại.

---

## 3. Yêu cầu phi chức năng

### REQ-NF-001: Interface-based design
- `IMerchantRepository` và `ICartSessionStore` phải được định nghĩa qua interface để dễ swap sang DB sau này.

### REQ-NF-002: CORS
- Bật CORS allow-all trong môi trường Development.

### REQ-NF-003: Cấu hình
- Đường dẫn file JSON cấu hình qua `appsettings.json` key `DataFilePaths:Merchants`.

### REQ-NF-004: Không có authentication
- Prototype không yêu cầu auth; `sessionId` là định danh duy nhất.
