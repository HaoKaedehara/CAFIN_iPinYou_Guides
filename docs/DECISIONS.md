# Quyết định triển khai hiện tại

Người dùng xác nhận chưa có code CAFIN gốc và yêu cầu:

> Triển khai thiết kế mới, ghi rõ khác biệt.

Vì vậy, các yêu cầu “phải đợi source Criteo” trong tài liệu định hướng ban đầu
được thay bằng việc đặc tả một kiến trúc mới minh bạch, có version và ablation.
Không cần chờ source gốc để viết CAFIN mới.

## Đã chốt cho phiên bản code này

- Version: `new_design_v1`.
- Nhánh cross: full-matrix cross kiểu DCNv2 trên embedding flatten, reshape về
  từng field và LayerNorm.
- Nhánh context: scaled multi-head self-attention, residual, dropout, LayerNorm.
- Cross-attention: hai chiều song song, projection riêng, Q từ nhánh đang cập
  nhật và K/V từ nhánh kia.
- Mean pooling, gate sigmoid theo từng chiều embedding, MLP và linear logit.
- Năm ablation và full model. Head giữ cùng kích thước đầu vào d.
- CAFIN_CrossAttention và full CAFIN có fusion cùng số tham số, khác phép toán.

## Có thể và chưa thể nói về khác biệt

Có thể đối chiếu cơ chế với AutoInt, DCNv2 và các ablation vì code đã rõ.
Không thể khẳng định khác gì chính xác so với “CAFIN Criteo gốc” khi chưa có
source hoặc phương trình gốc. Những điểm trên là lựa chọn thiết kế mới, không
phải thông tin đã khôi phục từ bản gốc.

Đặc tả kỹ thuật: [CAFIN_DESIGN.md](CAFIN_DESIGN.md).
Không dùng “novel”, “first”, “SOTA” hoặc “tăng hiệu quả kinh doanh thực tế” khi
chưa có rà literature và thực nghiệm tương ứng.

## Phân công thiết bị

**Cập nhật 18/09/2026:** người dùng yêu cầu không làm trên Colab nữa, chuyển sang
thiết bị Windows hiện tại. Dữ liệu đã tải ở `DataSet Downloaded` trong workspace.
Quyết định này thay hạn chế cũ chỉ code/tests CPU nhỏ: được tiền xử lý và chạy
mô hình tại máy này. Kiểm tra RAM/VRAM/đĩa và chạy smoke dữ liệu thật trước,
chọn batch size và số lượt phù hợp tài nguyên; không tự khởi chạy toàn bộ sweep.
Các tests dùng dữ liệu giả trong thư mục tạm; không đưa vào bảng kết quả nghiên cứu.

## Triển khai tự động nghiên cứu — 19/09/2026

Người dùng yêu cầu triển khai hướng bài CTR–RTB và tự làm tiếp không cần xác nhận.
Cho phép các queue đã khai báo trong `EXPERIMENT_PROTOCOL_V2.md`, chạy tuần tự
theo tài nguyên, có disk guard và checkpoint. Điều này mở rộng giới hạn cũ
không tự chạy sweep: không hỏi lại cho P1/P2, nhưng vẫn ghi rõ tiến độ thực.

- P0 giữ nguyên 15 run seed11, thăm dò campaign shift.
- P1 dùng split feature-group hash mới (không dùng label, không official/temporal),
  15 model/variant × 5 seeds ở cấu hình cố định.
- P2 chuẩn bị conservative singleton subset của raw season2 do đã phát hiện
  bid ID trùng với nội dung khác nhau; 6 model × 5 seeds và RTB.
- Chỉ finalize test sau đủ cohort validation và policy RTB; không đổi cấu hình
  theo test. Full tuning search, Season3 và literature audit đầy đủ vẫn là
  phần còn thiếu, không được lấp bằng số giả hoặc gọi là đã hoàn thành.
- Giữ nguyên source `src/cafin` để tiếp tục kiểm chứng run cũ; tooling mới ở
  `scripts/`. Source hash vẫn `8036373c86bc6efe5c67b9ef27cf499c7bb416bb41b2c69b85d74efcfa4e6f2c`.

## Tiếp tục tự động khi AFK — 22/09/2026

Người dùng yêu cầu bám kế hoạch, tự động tiếp tục và không cần xác nhận khi AFK
đến sáng. Giữ lựa chọn trước: chỉ tối ưu vận hành, không đổi protocol/batch.
Triển khai supervisor cho queue đã có, heartbeat, giữ Windows thức trong lúc
tính toán, thử lại có giới hạn lỗi khóa checkpoint đã biết và giữ disk guards.
Thu gọn lưu trữ bằng nén NTFS không mất dữ liệu, kiểm tra SHA-256, giữ mọi file
và đường dẫn. Không suy rộng thành quyền xóa dữ liệu hoặc thay thí nghiệm.
Hướng dẫn: `docs/UNATTENDED_RUN.md`.
