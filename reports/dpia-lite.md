# DPIA-lite — checkpoint 5

## 1. Dữ liệu nào được xử lý?

Run A đọc ticket hỗ trợ trong `corpus/`. Ticket là nội dung không đáng tin cậy và có thể chứa tên, email, CCCD, số điện thoại, số tài khoản hoặc prompt injection. Run B có thể đọc hồ sơ khách hàng tổng hợp trong `data/customers.json`; các trường định danh và tài chính được coi là `restricted`. Bài lab không dùng dữ liệu cá nhân thật.

## 2. Ai xử lý và cho mục đích gì?

`support_l1` được tìm ticket để tổng hợp hoặc hỗ trợ trả lời, nhưng không được đọc hồ sơ khách. `dpo` được đọc hồ sơ liên quan cho mục đích `support-reply` hoặc các mục đích đã khai báo trong fixture. Quyền hiệu lực còn phụ thuộc scope của agent, scope cha khi có delegation và TTL. Tham số `--as` chỉ chọn principal giả lập; nó không xác thực người đang sử dụng máy.

## 3. Dữ liệu đi qua đâu?

Run A gọi `search_docs`, phát hiện lure và tạo bản nội dung đã redact để tóm tắt. Ranh giới sang Run B chỉ mang danh sách số ticket lấy từ tên file; customer ID và URL trong free text không được truyền qua. Run B dùng `related_tickets` ở nguồn dữ liệu tin cậy để tìm khách liên quan, kiểm policy rồi mới gọi `read_customer`. Mọi quyết định được ghi vào `reports/ledger.jsonl` bằng `args_hash`, không ghi tham số hay hồ sơ thô. Sink thử nghiệm chỉ nghe tại `http://127.0.0.1:10024`; policy từ chối `restricted + egress` trước khi `http_post` chạy.

## 4. Control đã chứng minh và giới hạn còn lại

Sau containment, 4 lure × 2 principal đều pass; `support_l1` không đọc private store, DPO vẫn đọc hồ sơ hợp lệ, `reports/attack-after.log` có kích thước 0 byte và ledger có dòng deny `http_post` kèm reason. PII regex đạt precision/recall 1.000 trên testset tổng hợp, còn hash-chain phát hiện một dòng cũ bị sửa.

Đây vẫn chỉ là mô phỏng học tập: mock LLM có hành vi cố định, principal không được xác thực, regex không bao phủ mọi cách viết PII, allowlist localhost không đại diện cho kiểm soát mạng production, và ledger không có mốc tin cậy bên ngoài nên không phát hiện chắc chắn việc cắt dòng cuối hoặc viết lại cả chuỗi. Tài liệu này không phải DPIA pháp lý chính thức.
