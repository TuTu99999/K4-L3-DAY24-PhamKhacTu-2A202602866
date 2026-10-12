# Baseline — checkpoint 1

1. Baseline chưa có `agent_id`, `run_id` hay danh tính người dùng để kiểm quyền. Hàm `_naive_loop()` chỉ nhận `message` và `llm`, sau đó gọi thẳng `search_docs()` (`agent/loop.py:27-31`).

2. Trước `http_post` không có policy nào kiểm mục đích, quyền hoặc loại dữ liệu. Nội dung ticket vốn chỉ là dữ liệu lại được `find_injection()` hiểu thành lệnh; chính kết quả này dẫn luồng chạy tới `read_customer()` rồi `http_post()` (`agent/loop.py:34-44`). Allowlist trong `agent/tools.py:68-84` chỉ giữ URL ở localhost, không quyết định hồ sơ nào được phép gửi.

3. Body POST được tạo ngay tại `agent/loop.py:44` dưới dạng `{"records": collected}`. Sink nhận body trong `do_POST()` và ghi từng request vào `reports/sink.log` (`sink/sink.py:37,42-47`), nên đây là nơi kiểm tra bằng chứng rò rỉ dữ liệu giả.
