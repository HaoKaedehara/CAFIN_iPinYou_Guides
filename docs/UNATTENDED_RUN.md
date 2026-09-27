# Chạy tự động khi người dùng AFK

Ngày22/09/2026, người dùng yêu cầu tiếp tục kế hoạch tự động, không cần xác nhận.
Giữ protocol/model/source/batch/seed và các ngưỡng kiểm tra hiện tại.

## Cách vận hành

`scripts/supervise_research_pipeline.py` chạy pipeline đã khai báo, một instance.
Khóa supervisor/pipeline/runner chống chạy trùng. Trong khi làm việc, supervisor
dùng Windows SetThreadExecutionState để giữ hệ thống thức; không ép màn hình sáng
và không đổi power scheme toàn máy. Khi supervisor kết thúc, yêu cầu giữ thức
được gỡ. Không đảm bảo tiếp tục qua mất điện/reboot hoặc hành động Sleep thủ công.

Trình tự không đổi: hoàn tất75validation P1 → hoàn tất30validation P2 → chọn
policy RTB trên validation → freeze/test/replay. Các hash/data/test guards vẫn
do pipeline gốc thực thi. Sửa source model sẽ làm mất khả năng tiếp tục queue.

Supervisor chỉ tự thử lại lỗi checkpoint-save WinError5 ở os.replace, tối đa
3lần khởi động tổng cộng. Lỗi identity, loss, dữ liệu, lỗi chưa phân loại hoặc
disk guard khiến dừng có lý do, không lặp vô hạn hoặc hạ ngưỡng bảo vệ.

## Dung lượng: nén không mất dữ liệu

`scripts/compact_research_storage.py` dùng NTFS transparent compression.
Chỉ chạy maintenance lúc pipeline đã dừng và lấy được khóa.

- Nén prediction CSV đã có metrics và run complete; SHA-256 trước phải khớp
  metrics, SHA-256 sau phải khớp trước.
- Nén các canonical/raw CSV lớn trong hai thư mục được chỉ định; SHA-256
  trước/sau phải khớp. Không sửa nội dung hoặc đường dẫn.
- Đặt thuộc tính nén cho thư mục runs và từng run để file tương lai kế thừa.
- Không xóa file, không bỏ optimizer/checkpoint, không đổi config/source/data.
- Giữ nguyên disk guard3GiB trước lượt train; finalizer vẫn có guard2GiB.

Tỷ lệ nén và overhead I/O phụ thuộc dữ liệu. Kiểm tra thực trên file tạm xác
nhận nén giữ byte, file mới kế thừa và atomic rename vẫn hoạt động. Đây là kiểm
tra vận hành, không phải kết quả nghiên cứu. File dự đoán thật thử đầu tiên giảm
từ66.852.187 xuống33.431.552byte vật lý và SHA-256 khớp artifact.

## Xem tiến độ

- `experiments/supervisor_state.json`: heartbeat khoảng30giây trong khi pipeline
  đang chạy, PID, lần chạy, dung lượng, số file metrics đã xuất hiện. Counts này
  là tiện ích giám sát; pipeline vẫn kiểm tra integrity để chấp nhận kết quả.
- `experiments/pipeline_state.json`: stage và trạng thái pipeline.
- `experiments/{x1_grouped,season2}/plan_state.json`: run hiện tại/completed.
- `experiments/supervisor_launch_receipt.json`: PID và log của supervisor.
- `experiments/pipeline_launch_receipt.json`: PID và log pipeline mới nhất.
- `results/diagnostics/storage_compression_latest.json`: danh sách file/hash
  kiểm tra và dung lượng trước/sau maintenance.

Không mở/torch.load `last.pt` khi trainer còn chạy trên Windows; chỉ stat mtime,
đọc log và history. Việc đọc checkpoint trực tiếp đã gây nguy cơ khóa file ghi.

Tiếp tục sau reboot: dùng Python dự án, `PYTHONPATH=src`, chạy
`python -u scripts/supervise_research_pipeline.py` từ workspace. Khóa OS bảo vệ
tránh instance thứ hai. Chỉ chạy sau kiểm tra dữ liệu và dung lượng còn đủ.

Chỉ gọi hoàn tất thực nghiệm đã khai báo khi có
`experiments/finalization_receipt.json` và trạng thái tương ứng. Hoàn tất pipeline
không đồng nghĩa đã hoàn tất novelty review, code tác giả external, Season3,
thống kê mở rộng hay bài báo sẵn sàng nộp.
