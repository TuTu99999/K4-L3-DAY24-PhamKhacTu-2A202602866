# Verification — điền trước khi nộp

- OS / Python / Gitleaks version:
- `python -m pytest tests/ -q`:
- `pre-commit run --all-files`:
- Ledger verify:
- Các lỗi còn lại:

## CP3a — PII gate

- Lệnh: `python -m pytest tests/test_pii.py -v -s --basetemp=.pytest_tmp`
- Kết quả: `2 passed`; `gold=118`, `pred=118`, `tp=118`, precision `1.000`, recall `1.000`.
- Giới hạn: bộ nhận diện dùng regex theo định dạng và nhãn tiếng Việt trong dữ liệu lab. Nó chưa hiểu ngữ cảnh tự do, số bị viết bằng chữ, Unicode lookalike hoặc các định dạng PII ngoài bốn loại đang kiểm tra; vì vậy kết quả này không đại diện cho dữ liệu production.
