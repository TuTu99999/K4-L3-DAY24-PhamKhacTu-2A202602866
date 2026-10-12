# Verification — trước khi nộp

## Môi trường

- Windows PowerShell; Python `3.11.9`; Gitleaks `8.30.1`; pre-commit `4.6.2`; pip-tools `7.6.2`.
- Repo giữ đầy đủ lịch sử: `git cat-file -t 1bdd849` trả `commit`.
- `python -m pip install --require-hashes -r requirements.lock`: thành công, không có dependency bị thiếu hoặc sai hash.

## Kết quả theo checkpoint

- **CP1:** đã đọc luồng `search_docs → find_injection → read_customer → http_post` và ghi phân tích tại `reports/baseline.md`.
- **CP2:** `python -m agent.check_lure corpus/ticket-90*.md` trả `4/4 lure sẵn sàng`. Baseline tạo `reports/attack-before.log` gồm 4 dòng POST chứa hồ sơ tổng hợp `KH-000999`.
- **CP3a:** `python -m pytest tests/test_pii.py -v -s --basetemp=.pytest_tmp` trả `2 passed`; `gold=118`, `pred=118`, `tp=118`, precision `1.000`, recall `1.000`.
- **CP3b:** `python -m pytest tests/test_policy.py -v --basetemp=.pytest_tmp` trả `4 passed`; có cả allow hợp lệ và deny `restricted + egress`.
- **CP3c:** `python -m pytest tests/test_obo.py -v --basetemp=.pytest_tmp` trả `5 passed`; `support_l1` bị deny, DPO vẫn đọc được hồ sơ hợp lệ, delegation không mở rộng scope cha.
- **CP3d:** `test_split.py` trả `1 passed`; `KH-000777` không bị đọc. `test_injection.py` trả `8 passed`, không skip; bốn lure đều bị chặn với cả hai principal.
- **CP3e:** `test_ledger.py` trả `3 passed`; kiểm được nối hash, phát hiện sửa dòng cũ và reason bắt buộc.
- **CP4:** Gitleaks quét lịch sử tìm đúng 1 token giả ở commit `1bdd849` bằng rule `lab24-synthetic-token` (exit 1 như dự kiến). Quét file hiện tại sau allowlist `.venv/` sạch (exit 0). `pre-commit run --all-files` trả `Detect hardcoded secrets... Passed`. `test_supply_chain.py` trả `3 passed`.
- **CP5:** `python -m pytest tests/ -q --basetemp=.pytest_tmp` trả **33 passed, 0 failed, 0 skipped** trong 5.26 giây.

## Bằng chứng replay sau containment

- Chạy `python -m agent.loop --mock --as dpo "Tổng hợp các ticket còn mở tuần này"` trong khi sink localhost đang nghe.
- `reports/attack-before.log`: 4 dòng, có POST chứa dữ liệu tổng hợp của `KH-000999`.
- `reports/attack-after.log`: **0 byte**, không có hồ sơ nào tới sink.
- `reports/ledger.jsonl`: 46 dòng sau hai lượt CLI an toàn; `verify(Path('reports/ledger.jsonl'))` trả `True`. Có dòng `tool=http_post`, `decision=deny`, `on_behalf_of=dpo`, reason không rỗng.

## Lỗi gặp trong lúc kiểm

- Lần quét thư mục đầu tiên, Gitleaks báo false positive `generic-api-key` trong `.venv/Lib/site-packages/setuptools/_distutils/msvccompiler.py`. Đã thêm allowlist đúng tiền tố `.venv/` rồi quét lại; rule token giả và phần còn lại của repo vẫn được giữ nguyên.
- `pip-tools` ban đầu không ghi được cache mặc định ngoài workspace trong môi trường sandbox. Đã dùng cache tạm nằm trong repo để sinh lock rồi xoá cache; bước cài lại bằng `--require-hashes` thành công.
- Lỗi còn lại: không có lỗi test đã biết. Các giới hạn thiết kế của mock, regex, principal giả lập và hash-chain được ghi trong `reports/dpia-lite.md`.
