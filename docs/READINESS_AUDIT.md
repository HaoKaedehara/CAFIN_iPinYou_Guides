# Rà soát mức độ sẵn sàng cho báo cáo CAFIN–iPinYou

Thời điểm kiểm tra: 18/09/2026, khoảng 23:40 giờ Việt Nam. Đây là ảnh chụp trạng
thái khi suite đang chạy; kết quả mới hơn phải lấy từ `runs/*/status.json` và
`valid_metrics.json`.

## Kết luận

**Code lõi đủ để chạy thực nghiệm ban đầu; chưa hoàn thiện toàn bộ mục tiêu nghiên
cứu. Có thể viết báo cáo tiến độ/bản thảo phương pháp và kết quả sơ bộ, chưa đủ
cho bài báo kết quả cuối theo định hướng CTR → RTB ban đầu.**

Đối chiếu với `01_DE_TAI_CHI_TIET.md` (Definition of Done),
`04_KE_HOACH_THUC_NGHIEM.md` và `07_OUTLINE_BAI_BAO.md`. Yêu cầu chờ source Criteo
đã được thay bằng quyết định triển khai CAFIN `new_design_v1` trong
`docs/DECISIONS.md`; không coi thiếu source gốc là trở ngại cần giải quyết.

## 1. Code: đã kiểm tra và phần chưa hoàn thiện

- Chạy lại toàn bộ **30 tests: đạt**, 10,543 giây; log:
  `results/diagnostics/code_audit_tests.log`. Tests dùng dữ liệu giả nhỏ, không
  phải kết quả thực nghiệm. Các tests bootstrap ở đây là bootstrap môi trường
  notebook, **không phải statistical bootstrap confidence interval**.
- Đọc code models/data/train/metrics/sweep/report, đối chiếu CAFIN với
  `docs/CAFIN_DESIGN.md`: có cross stream, attention stream, cross-attention hai
  chiều song song, gate, 5 ablation; vocabulary chỉ fit train; khóa test;
  checkpoint và kiểm tra source/config/data identity.
- Kiểm tra lại hash checkpoint/predictions và identity của cả 4 run full đã
  evaluate: đều khớp. Evidence: `results/diagnostics/readiness_audit.json`.
- Chưa phải audit toàn bộ code theo từng dòng hoặc chứng minh không có lỗi.
  Chưa xác minh độc lập tất cả biến thể baseline với bài gốc trong lần rà này.

Các vấn đề còn cần xử lý:

1. **Script suite local chưa tự khôi phục sau gián đoạn.**
   `scripts/run_local_model_suite.ps1:29` luôn gọi `train` không có `--resume`.
   Khi run đã có `last.pt` nhưng chưa complete, khởi động lại script sẽ bị từ
   chối vì thư mục đã tồn tại (`src/cafin/train.py`). Nếu train đã complete
   nhưng thiếu validation metrics, script cũng gọi train lại thay vì chỉ
   evaluate. Đây là lỗi nhánh restart, chưa phải lỗi của tiến trình đang chạy.
   Core `src/cafin/sweep.py` đã có nhánh resume/evaluate-only đúng; wrapper local
   chưa dùng nó.
2. **Nhánh skip của wrapper chỉ kiểm tra trạng thái và sự tồn tại metrics**
   (`scripts/run_local_model_suite.ps1:20`), chưa kiểm tra hash như core sweep.
   Wrapper cũng chưa có khóa chống hai instance cùng chạy. Những khẳng định
   trước đây về khả năng tự phục hồi/chống trùng của wrapper cần giới hạn lại.
3. **Tên `inference_seconds` chưa phản ánh thời gian forward riêng.**
   `src/cafin/train.py:253` đo toàn bộ `predict`, bao gồm đọc batch, truyền dữ liệu,
   forward và tính/sort metrics. Có thể báo là thời gian đánh giá, không được
   gọi là latency inference thuần hay latency phục vụ một bid request.
4. Chưa có code statistical bootstrap CI hoặc post-hoc calibrator/ECE trong
   `src/cafin`. Brier đã có. CI/calibration bổ sung là phân tích cần thiết nếu
   muốn đưa ra các kết luận tương ứng; không phải mọi bổ sung đều bắt buộc
   như 5-seed, ablation, dual protocol và RTB trong mục tiêu chính.
5. README/handoff có các đoạn trạng thái lịch sử đã lỗi thời. Đã sửa tóm tắt
   README và thêm liên kết tới audit này; các ghi chép cũ trong handoff vẫn là
   lịch sử, không được dùng thay trạng thái run thực.

Không sửa `src/cafin` hoặc các config/run đang dùng trong lần audit, để giữ
identity và khả năng resume. Suite đang hoạt động tiếp tục theo cấu hình cũ.

## 2. Dữ liệu: đã kiểm chứng nhưng có độ lệch split đáng kể

Các kiểm tra checksum/schema/label/count và hash preprocessing đã có trong
`results/diagnostics/local_data_audit.json` và `local_full_verification.json`.

| Split x1 | Dòng | Click | CTR |
|---|---:|---:|---:|
| Train | 13.855.732 | 9.685 | 0,069899% |
| Validation | 1.539.526 | 1.872 | 0,121596% |
| Packaged test | 4.100.716 | 3.008 | 0,073353% |

Thống kê test lấy từ manifest đã có; lần audit này không inference hoặc mở thêm
test để chọn mô hình. Validation là **10% dòng cuối packaged train**, chưa có
bằng chứng để gọi là random, official validation hay chronological.

Đã bổ sung kiểm tra toàn bộ validation metadata (hash khớp), train/validation
feature drift và đánh giá theo advertiser cho CAFIN:

- **1.000.054/1.539.526 dòng validation (64,9586%)** thuộc hai advertiser x1
  không nằm trong train vocabulary: `937616` (687.617 dòng) và `937618`
  (312.437 dòng). Train advertiser UNK rate = 0; đây không chỉ là suy đoán từ
  UNK gộp rare/missing. Các mã này là mã trong x1, chưa map sang raw advertiser ID.
- Creative UNK rate validation cũng là 64,9586%.
- JSD train→validation trên encoded advertiser = 0,830725; creative = 0,831039;
  domain = 0,691396 (log cơ số 2). UNK vẫn gộp nhiều giá trị, nên không đo được
  mọi thay đổi bên trong nhóm UNK.
- CAFIN pooled AUC = 0,802964, trong khi AUC theo 4 advertiser chỉ khoảng
  0,596084–0,675966. Advertiser `937618` có 1.386 positive và LogLoss 0,042743;
  advertiser `937615` chỉ có 26 positive. Cần báo cả pooled và per-campaign,
  không suy hiệu quả trong từng campaign từ pooled AUC.

Evidence: `readiness_audit.json`, `x1_validation_drift.json`,
`cafin_validation_campaigns.json` trong `results/diagnostics/`.

Độ lệch này không chứng minh implementation sai hay dữ liệu bị hỏng, nhưng
protocol hiện tại đang chứa lượng lớn campaign chưa thấy ở train. Trước benchmark
chính thức cần xác định rõ phạm vi: giữ nó như một protocol có campaign shift,
hoặc khai báo split mới phù hợp câu hỏi nghiên cứu và chạy lại trong thư mục mới.
Không đổi split rồi gộp chung với các kết quả hiện có.

Raw season 2 mới xác minh archive; chưa có processed chronological dataset hay
RTB artifacts. Season 3 chưa có local. Hai ZIP x1/raw là hai biểu diễn của họ
dữ liệu iPinYou, không phải hai benchmark độc lập đã được đánh giá đầy đủ.

## 3. Kết quả thực có đến thời điểm audit

4/15 model/variant đã train xong và evaluate validation; DNN đang training.
Mỗi model mới có seed 11, protocol standardized. Không tính smoke run.

| Model | Validation LogLoss ↓ | ROC-AUC ↑ | PR-AUC ↑ |
|---|---:|---:|---:|
| LR | 0,01485618 | 0,794225 | 0,00519916 |
| FM | 0,01168891 | 0,796146 | 0,00468162 |
| DeepFM | 0,01102164 | 0,788544 | 0,00494228 |
| CAFIN | 0,01163046 | 0,802964 | 0,00524406 |

**Sanity check bổ sung:** dự đoán cùng một xác suất bằng CTR train
(`p = 9685 / 13855732`) cho mọi dòng validation đạt LogLoss **0,00953339**,
Brier **0,00121475**, ROC-AUC 0,5. Đây là đối chứng tính trực tiếp từ số nhãn thật,
không phải model được train và không phải baseline được tune trên validation.
Cả bốn model hoàn tất đều kém đối chứng này về LogLoss và Brier. Ranking và
ước lượng độ lớn xác suất đang cho tín hiệu khác nhau; cần phân tích calibration,
campaign shift và tuning trên validation trước các kết luận về bidding utility.
Chưa xác định nguyên nhân từ những con số này.

DNN có metric theo epoch trong `history.json` nhưng chưa có final validation
artifact tại thời điểm audit, nên chưa đưa vào bảng final-run ở trên.

## 4. Đối chiếu mục tiêu ban đầu

| Hạng mục | Code / chuẩn bị | Thực nghiệm đủ cho kết luận cuối? |
|---|---|---|
| CAFIN mới + 5 ablation | Có đặc tả và implementation | CAFIN 1 seed; ablation chưa hoàn tất |
| 9 baseline CTR | Có implementation và config | LR/FM/DeepFM xong; DNN đang chạy; 5 model còn lại chưa bắt đầu |
| 5 seeds | Config kế hoạch có 11/23/42/71/101 | Chỉ seed 11 hiện có kết quả |
| Hai data protocol | Có adapter/prepare | Chỉ x1 đã preprocessing và train |
| Fair tuning | Có grid/sweep tooling | Chưa search với ngân sách đã chốt |
| Final test | Có freeze/evaluate gate | Chưa có test metrics |
| Campaign robustness | Có group-metrics | Mới bổ sung CAFIN trên validation, chưa đủ so sánh |
| Season 2/3 drift | Có công cụ cho metadata season | Chưa có dữ liệu season 3/local runs tương ứng |
| CTR → RTB budget replay | Có code và tests nhỏ | Chưa chạy trên raw thật |
| Bảng, hình kết quả chính | Có công cụ tổng hợp/vẽ | Chưa đủ multi-seed/test/RTB artifacts |
| Literature/novelty audit | Có source registry | Chưa có rà soát đầy đủ và matrix cập nhật |

Suite PowerShell hiện chỉ chạy phần còn lại của **seed 11, x1**. Nó không tự triển
khai 4 seed còn lại, raw protocol, RTB, literature hay viết bài. Ngay cả khi suite
đó hoàn tất, toàn bộ đề tài vẫn chưa hoàn tất.

## 5. Viết được gì ngay và thứ tự tiếp theo

Viết được báo cáo tiến độ: mục tiêu, phạm vi CAFIN mới, phương pháp, pipeline,
đặc tả kiến trúc, thống kê dataset, protocol, kết quả validation sơ bộ và các
hạn chế được kiểm chứng ở trên. Phần kết quả cuối, abstract có số tổng kết và
kết luận RQ1–RQ6 chưa thể hoàn chỉnh.

Ưu tiên tiếp theo:

1. Giải thích/chốt vai trò campaign shift và protocol validation; bổ sung
   calibration diagnostics và so sánh theo campaign cho các baseline.
2. Hoàn tất run đang chạy; củng cố wrapper resume/evaluate-only/hash/lock trước
   khi dựa vào khả năng khôi phục tự động hoặc mở rộng nhiều seed.
3. Hoàn tất baseline/ablation, ngân sách tuning công bằng và 5 seeds trên protocol
   đã chốt. Giữ riêng các run thăm dò/split cũ.
4. Import raw, xác minh chronology/giá/nhãn, chuẩn bị season analysis; train,
   tune bidder trên validation rồi chốt protocol trước final test và RTB.
5. Tổng hợp mean/std, per-campaign, drift, RTB và trade-off tài nguyên; viết kết
   luận từ artifacts thật theo outline ban đầu. CI và baseline Tier 2 bổ sung
   theo phạm vi claim/compute, không thay thế các phần bắt buộc còn thiếu.
