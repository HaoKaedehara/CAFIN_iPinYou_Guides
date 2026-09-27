# Chuyển máy và tiếp tục trên Colab

CAFIN hiện là `new_design_v1`, không phải bản tái tạo code Criteo. Máy Windows
chỉ sửa code, kiểm tra CPU nhỏ và đóng gói. Các bước tải dữ liệu thật, tiền xử lý
full và huấn luyện dưới đây dành cho thiết bị chạy thí nghiệm.

## Bàn giao source

1. Tạo lại notebooks bằng `python scripts/make_notebooks.py`, rồi tạo ZIP bằng
   `python scripts/package_project.py` trên máy phát triển.
2. Upload `dist/cafin_ipinyou_code.zip` vào `Google Drive/MyDrive/CAFIN/` và mở
   `notebooks/01_prepare_and_train_colab.ipynb` trên Colab.
3. Chọn GPU, chạy cell thiết lập và cell kiểm tra nhỏ trước khi tải dataset.
   Báo cáo môi trường nằm ở `MyDrive/CAFIN/diagnostics/preflight_*.json`.
   Tests dùng dữ liệu giả và CPU; chúng không xác minh GPU hay benchmark iPinYou.
4. Kiểm tra RAM/VRAM được cấp và quota Drive thực tế. Dung lượng filesystem của
   Drive trong báo cáo không nhất thiết bằng quota còn lại của tài khoản.
5. Chạy các cell khôi phục/tải/prepare, rồi một model/seed để đánh giá tài nguyên
   trên dữ liệu thật trước khi chạy sweep. Chốt cấu hình sau bước khảo sát này.

Cell thiết lập giải nén source vào thư mục tạm rồi mới chuyển thành PROJECT.
Receipt `.cafin_archive.json` lưu SHA-256 ZIP và từng file đã giải nén. Khi chạy
lại cell, ZIP và các file phân phối phải khớp. Các config mới do notebook tạo
không nằm trong danh sách file phân phối nên vẫn dùng được.

Nếu ZIP đổi, source bị sửa hoặc PROJECT cũ thiếu receipt, cell dừng với thông
báo cụ thể. Khởi tạo runtime mới và chạy setup với đúng ZIP. Giữ riêng ZIP tương
ứng với những run đang cần resume; không ghi đè source giữa lúc huấn luyện.

## Tiếp tục phiên bị ngắt

1. Mở lại notebook 01 với cùng ZIP; mount cùng Drive và chạy setup/kiểm tra nhỏ.
2. Khôi phục cache processed vào `/content/cafin_data/standardized`.
3. Giữ đúng `MODEL`, `EXPERIMENT`, `SEED` để trỏ tới RUN_DIR cũ. Notebook đọc config
   đã lưu và dùng `last.pt` nếu có. Hash source/config/data phải khớp.
4. Nếu run đã hoàn thành, notebook bỏ qua training và xuất lại validation.
   Chỉ thay đường dẫn khi chuyển máy; đổi tham số tính toán thì dùng run mới.

Checkpoint được lưu định kỳ, nên phần sau checkpoint cuối có thể phải chạy lại.
Nếu run đã tạo file nhưng chưa có `last.pt`, chưa có checkpoint để resume; giữ
thư mục đó để kiểm tra lỗi và chọn tên run mới. Không coi thư mục có file là một
run đã hoàn tất; xem `status.json`.

Nếu prepare bị ngắt trước khi có `manifest.json`, dùng output trống mới cho lần
prepare sau. Nếu cache giải nén lỗi, giữ cache để kiểm tra và dùng thư mục đích
mới; không trộn với output dở. Giữ tên cache/output khác nhau cho standardized,
chronological và dữ liệu mẫu.

## Sweep và final test

- Notebook 03 tạo plan và chỉ chạy một lượt mỗi lệnh khi `RUN_SWEEP=True`.
  Giữ cùng plan, source và `EXPERIMENT_ROOT` khi tiếp tục.
- Run sweep có hash trong tên. Trong notebook 02, thay RUN_DIR bằng đúng thư mục
  `experiments/.../runs/<run_id>`. Cell summarize tự lấy thư mục cha của RUN_DIR,
  lưu bảng vào thư mục results cạnh thư mục runs tương ứng.
- Chốt model/config/seed trước test. Với raw chronological, chọn bidder trên
  validation trước khi mở test. Notebook 02 mặc định `RUN_FINAL_TEST=False`.
- Sau khi freeze/test, không quay lại tune dựa trên kết quả test. Raw RTB cần
  payingprice thật; không lấy slotprice standardized thay thế.

## Artifact cần giữ để phân tích

Giữ ZIP source, config dữ liệu, manifest/vocab/cache, plan và toàn bộ thư mục run
(config, môi trường, checkpoints, history, status, predictions, metrics, policy).
Báo cáo CPU nhỏ nằm ở diagnostics, không đưa vào bảng kết quả nghiên cứu.

Chưa có xác nhận notebook đã chạy thành công trên Colab hoặc dữ liệu iPinYou
thật. Sau lần chạy đầu, cập nhật SESSION_HANDOFF.md bằng đường dẫn artifact,
phiên bản môi trường, số dòng/class counts, thời gian và bộ nhớ thực đo.
