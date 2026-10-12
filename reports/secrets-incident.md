# Secret giả trong lịch sử Git — checkpoint 4

- **Scanner:** Gitleaks `8.30.1`, chạy với `.gitleaks.toml` và tùy chọn `--redact` để không in lại giá trị token.
- **Commit chứa token giả:** `1bdd849922156c868a5ee9293d68f5b72e242d2c` (`1bdd849`), file `lab-fake-secret.txt`, dòng 1.
- **Rule và kết quả quét lịch sử:** rule `lab24-synthetic-token` tìm thấy đúng 1 finding trong 9 commit; lệnh `gitleaks git --config .gitleaks.toml --redact --verbose .` trả exit code 1 như dự kiến. Đây là token tổng hợp, vô hiệu, được cài sẵn cho bài tập chứ không phải credential thật.
- **Lần quét file hiện tại đầu tiên:** `gitleaks dir --config .gitleaks.toml --redact --verbose .` báo một false positive `generic-api-key` trong `.venv/Lib/site-packages/setuptools/_distutils/msvccompiler.py`. Đây là mã thư viện nằm trong môi trường ảo cục bộ, không phải mã nguồn nộp bài. Mình thêm allowlist chỉ cho đường dẫn bắt đầu bằng `.venv/`, không tắt rule và không bỏ qua phần còn lại của repo.
- **Kết quả sau khi chỉnh allowlist:** quét lịch sử vẫn tìm thấy đúng token giả ở commit `1bdd849` và trả exit code 1; quét file hiện tại không còn finding và trả exit code 0.
- **Nếu đây là credential thật:** cần thu hồi hoặc rotate ngay tại dịch vụ đã cấp token, kiểm log để xác định token có bị sử dụng sai hay không, thay credential ở nơi triển khai và xử lý các bản sao còn tồn tại. Sau đó mới cân nhắc làm sạch lịch sử Git và thông báo cho những người đã clone repo.
- **Vì sao xoá file hoặc pin dependency chưa đủ:** xoá file ở commit mới không làm credential trong lịch sử biến mất hay mất hiệu lực. Pin dependency và hook giúp giảm rủi ro về sau, nhưng không thể thay thế việc thu hồi credential đã lộ.
