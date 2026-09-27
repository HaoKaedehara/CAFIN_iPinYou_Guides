# Kế hoạch thực nghiệm v2 — khai báo 19/09/2026

Người dùng yêu cầu tự triển khai hướng nghiên cứu CTR–RTB để chuẩn bị bản thảo;
không cần xác nhận lại các bước trong phạm vi đã thống nhất. Không có cam kết
tạp chí chấp nhận. Tên làm việc: **Campaign Shift, Probability Quality, and
Budgeted Bidding: An Evaluation of Cross-Attentive Feature Interaction on iPinYou**.

## Protocol và vai trò

- P0: `x1_local_full`, packaged-train tail. 15 model/variant seed11 đã hoàn tất.
  Giữ làm thí nghiệm campaign shift thăm dò; không gọi là benchmark chính đã tune.
- P1: `x1_grouped_v1`. SHA-256 của `[split_seed, 16 feature values]`, lấy 64 bit đầu
  và ngưỡng 0,1. Split seed cố định 20260919; label không nằm trong hash. Advertiser
  nằm trong feature tuple. Cùng tuple luôn ở cùng split, kể cả nhãn khác nhau.
  Đây là holdout ngẫu nhiên xác định theo nhóm, xấp xỉ 90/10, **không phải exact
  stratification theo campaign/label**, không chronological/official. Kiểm tra cả
  hai nhãn và campaign coverage trước train; không dò nhiều seed để chọn split đẹp.
  Packaged test giữ nguyên. Chưa chứng minh độc lập các impression cùng user/IP.
- P2: raw season2, train 06–10/06/2013, validation 11–12/06, leaderboard test
  13–15/06. Boundary `20130611000000000`, kiểm tra timestamp thực tế và không overlap.
  Nhãn train/valid từ click join, test click_count>0. Bid ID phải duy nhất trong
  mỗi ngày, kèm kiểm tra cross-day trước tuyên bố toàn tập duy nhất. Giữ giá gốc.
  **Bổ sung sau kiểm tra integrity, trước train P2:** phát hiện cùng bid ID ở dòng
  200/250 của impression 06/06 nhưng timestamp và ipinyouid khác nhau. Không thể
  dùng bid ID đó để gán click duy nhất. P2 loại *mọi* impression có bid ID xuất hiện
  hơn một lần trong tổng train+leaderboard, độc lập nhãn; giữ nguyên file gốc và
  ghi số dòng/click bị loại. Click singleton được ghép về ngày impression tương ứng.
  Đây là **conservative singleton subset**, không phải exact full-raw benchmark.
  Không giữ một bản trùng tùy ý hoặc gán click cho mọi bản. Đánh giá tác động
  selection/exclusion trong giới hạn; dữ liệu gốc vẫn giữ để phân tích độ nhạy sau.
- Season3 chưa có local: là giai đoạn mở rộng còn thiếu, không giả tạo kết quả
  season2→3. Không coi P1/P2 là hai dataset độc lập.

Dataset card x1 xác nhận nguồn và checksum nhưng không nêu recipe validation cụ
thể đã xác minh trong lần đọc này:
https://huggingface.co/datasets/reczoo/iPinYou_x1/blob/main/README.md
Do đó P1 là protocol do dự án khai báo, không tái lập exact BARS configuration.

## Model và ngân sách ban đầu

P1: LR, FM, DNN, IPNN, DeepFM, DCN, DCNv2, xDeepFM, AutoInt, CAFIN_DNN,
CAFIN_CrossOnly, CAFIN_AttentionOnly, CAFIN_Concat, CAFIN_CrossAttention, CAFIN.
Seeds: 11,23,42,71,101. Một cấu hình cố định/model, tối đa 30 epochs, patience3,
batch512, Adam lr0,001, weight_decay1e-6, embedding16, hidden128/64, dropout0,1.
Thông số cấu trúc theo config đang triển khai. 75 run validation, tuần tự, đủ
kiểm tra độ ổn định của **cấu hình cố định**, chưa phải full fair hyperparameter
search. Phân tích giới hạn này trong bài. Không tuyên bố vượt tuned published SOTA.

P2 trước mắt so sánh LR, FM, DeepFM, DCNv2, AutoInt, CAFIN với cùng 5 seeds và
ngân sách cấu hình cố định. Ablation mở rộng raw và baseline Tier2 là bước sau,
không được báo là đã chạy. Full ablation nằm ở P1.

Chọn checkpoint bằng validation LogLoss, AUC chỉ tie-break chính xác như code.
Không đổi tiêu chí sau khi xem test. Không chọn seed đẹp nhất. Báo mean/sample std,
per-campaign, AP và trapezoidal PR-AUC tách biệt, Brier; so với constant train-CTR.
Phân tích calibration dùng bin khai báo trước, không dùng ECE một mình ở CTR hiếm.
Không tự áp dụng class weighting hoặc negative downsampling cho main comparison.

## RTB và test

Bidder linear `beta*pCTR/train_CTR`; cùng grid và tie-break cho mọi model; chọn
beta theo validation/campaign/budget. Main budget1/8; grid1/2,1/4,1/8,1/16,1/32.
Freeze model/config/bidder trước test. Test chỉ mở sau khi các run của giai đoạn
đủ và kiểm tra integrity; không dùng test để sửa split/tune/model-selection.
Report clicks/spend/wins/eCPC/utilization; observed-impression replay, không online lift.

## Tài nguyên và tự động hóa

Mỗi GPU chỉ một tiến trình train. Runner có khóa OS, hash checks, resume,
evaluate-only và disk guard (còn dưới 3GiB thì dừng, không xóa data/checkpoint).
75 P1 + 30 P2 là nhiều giờ/ngày tính toán và có thể vượt dung lượng hiện còn;
state files ghi chính xác completed/remaining/disk_guard/failure. Không coi
khởi động queue là hoàn thành thí nghiệm. Không đổi source model trong khi queue chạy.

Đầu ra mỗi giai đoạn: artifacts + bảng từ kết quả thật + manuscript ghi rõ phạm
vi đang có. Abstract/kết luận cuối chỉ cập nhật khi đủ bằng chứng. Literature
matrix, statistical CI, Season3 và chọn venue cụ thể vẫn cần hoàn thiện.
