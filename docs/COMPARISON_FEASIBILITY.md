# Khả năng so sánh CAFIN với công trình dùng iPinYou

Đánh giá ngày20/09/2026 từ source, artifacts validation và nguồn primary.
Chưa chạy repo tác giả của các bài bổ sung; không đổi queue hiện tại.

| Yêu cầu | Có thể làm ngay | Phần còn thiếu |
|---|---|---|
| Mô tả phương pháp liên quan | Viết từ paper/repo đã xác minh | Audit đầy đủ thuật toán, code/config và chạy thử repo |
| Mô tả thuật toán CAFIN | Đặc tả và pseudocode khớp source | Chứng minh tính mới và lợi ích thành phần |
| So sánh thực nghiệm | Bảng sơ bộ giữa implementation local cùng protocol/seed | Code tác giả chạy lại, đủ cohort, final test, tuning và uncertainty |

Queue hiện tại dùng baseline tự triển khai. Hoàn tất P1/P2 không tự tạo kết quả
tái lập repo tác giả. Ước lượng54giờ train còn lại không gồm các repo bổ sung.

## Công trình và tình trạng code

- **RLCP, Liu et al., Knowledge-Based Systems, volume283, 2024:** ensemble CTR,
  quyết định số model và trọng số cho từng impression bằng RL. Bài dùng
  iPinYou/Criteo/Avazu; repo có `src/ipinyou_base_model` và hướng dẫn TD3 biến thể.
  [Paper](https://www.sciencedirect.com/science/article/pii/S0950705123009024),
  [code tác giả](https://github.com/mjliu-advertising/RLCP).
  Là ứng viên đối chiếu CTR nhưng chi phí gồm cả base models. Publisher ghi
  volume11/01/2024, DOI có2023; Word đang ghi2023 cần chuẩn hóa citation.
- **RLB, Cai et al., WSDM2017:** MDP tối ưu giá thầu/ngân sách qua chuỗi auction.
  Repo có thí nghiệm iPinYou; demo chỉ là prefix350.000dòng/campaign.
  [Paper](https://arxiv.org/abs/1701.02490),
  [code tác giả](https://github.com/han-cai/rlb-dp).
  Phù hợp RTB, không phải đối thủ CTR để so AUC trực tiếp với CAFIN.
- **IIBidder, Luo et al., 2024:** imitation và imagination cho bidding khi thông
  tin giá không đầy đủ. Repo có code, môi trường replay và demo iPinYou;
  hướng dẫn Python3.9, chưa thử tương thích môi trường local Python3.13.
  [Code và citation tác giả](https://github.com/JNU-Tangyin/imagineRTB).
- **FO-FTRL-DCN, Huang et al., Algorithms2020:** DCN, FTRL và xử lý đặc trưng/SMOTE;
  bài dùng bốn advertiser, split7ngày/3ngày.
  [Paper](https://www.mdpi.com/1999-4893/13/12/342).
  Chưa xác minh repo tác giả trong lần tra cứu này; không gọi là đã có code.

DeepFM/DCNv2/AutoInt/xDeepFM hữu ích cho đối chiếu cơ chế nhưng implementation
local là variant độc lập trong `SOURCES.md`. Tên model không chứng minh đã dùng
code tác giả. [Repo nhóm AutoInt](https://github.com/DeepGraphLearning/RecommenderSystems)
có implementation; dataset/protocol của paper gốc cần audit riêng.
[BARS](https://openbenchmark.github.io/BARS/CTR/) là benchmark có code/dataset IDs,
không mặc nhiên là code tác giả gốc của tất cả model.

## Thuật toán CAFIN new_design_v1

Nguồn: `src/cafin/models.py`, `src/cafin/train.py`, `docs/CAFIN_DESIGN.md`.
LN: LayerNorm; MHA: multi-head attention; Pool: trung bình theo fields.

```text
Đầu vào: train, validation; vocabulary chỉ fit train; config và seed đã chốt.
Đầu ra: checkpoint tốt nhất theo validation LogLoss, AUC phá hòa.
Khởi tạo model và Adam theo seed.
Lặp epoch đến max_epochs hoặc early stopping:
    Với mỗi minibatch (x,y) theo shuffle đã khai báo:
        E ← Embedding(x)
        c0 ← Flatten(E); c ← c0
        Lặp các lớp cross: c ← c0 ⊙ (W_l c + b_l) + c
        C ← LN(Reshape(c)); A ← E
        Lặp các lớp attention: A ← LN(A + Dropout(MHA_l(A,A,A)))
        Zc ← LN(C + Dropout(MHA_CA(C,A,A)))
        Za ← LN(A + Dropout(MHA_AC(A,C,C)))
        u ← Pool(Zc); v ← Pool(Za)
        g ← sigmoid(W_g [u;v] + b_g)
        h ← g ⊙ u + (1-g) ⊙ v
        z ← LinearTerm(x) + MLP(h)
        loss ← BCEWithLogits(z,y)
        Kiểm tra loss hữu hạn; backprop; clip gradient norm ≤10; Adam.step
        Lưu checkpoint theo chu kỳ.
    Đánh giá validation, cập nhật best checkpoint và early stopping.
Inference: pCTR ← sigmoid(z), dropout tắt.
```

Hai cross-attention có tham số riêng, cùng đọc C/A ban đầu và cập nhật song song.
Gate theo từng chiều embedding. Đây là thiết kế mới của dự án, không phải bản
khôi phục CAFIN Criteo. Kết hợp cross/attention/gate là giả thuyết cần kiểm chứng;
không tự chứng minh novelty. Có5ablation để kiểm tra vai trò từng thành phần.

## Bảng có thể báo cáo ngay

Chỉ **P1 validation, seed11**, từ từng
`experiments/x1_grouped/runs/<model>_seed11/valid_metrics.json`.
Không so mean hai seed baseline với một seed CAFIN như đối chứng hoàn chỉnh.

| Implementation local | LogLoss ↓ | ROC-AUC ↑ |
|---|---:|---:|
| CAFIN | 0.00558522 | 0.774860 |
| DeepFM | 0.00560265 | 0.773563 |
| DCNv2 | 0.00559482 | 0.777118 |
| xDeepFM | 0.00558007 | 0.778037 |
| AutoInt | 0.00559394 | 0.772451 |

CAFIN tốt hơn DeepFM ở hai chỉ số trong snapshot này, nhưng xDeepFM tốt hơn
CAFIN ở cả hai. Chưa có kết luận ý nghĩa thống kê hoặc ưu thế chung.
P2 seed11: CAFIN0.00616050/0.751338, DCNv2 0.00612070/0.752833(LogLoss/AUC),
cũng chưa cho thấy CAFIN đứng đầu. Không gọi các hàng này là published results.

## Điều kiện đối chiếu cuối và bước tiếp theo

1. Phân biệt số trích paper, code tác giả chạy lại và implementation local.
   Chỉ tính chênh lệch như đối chứng trực tiếp khi protocol tương thích.
2. Cùng split thực tế/campaign/features/nhãn/sampling, không chỉ cùng tên iPinYou.
   P1 là grouped split do dự án đặt ra; P2 singleton subset khác full raw.
3. Cùng tiêu chí selection và ngân sách tuning tương xứng. Cùng learning rate
   không tự chứng minh tuning công bằng. Chạy cohort seeds đã khai báo, báo
   mean/std và uncertainty phù hợp cấu trúc campaign/time/user.
4. Pin commit/config/dependencies cho repo tham chiếu, audit adapter và chạy
   sanity check. Với RLCP, audit prediction cho meta-training để tránh leakage,
   cân nhắc internal split/out-of-fold phù hợp và báo tổng chi phí ensemble.
5. RTB: đo đóng góp CTR bằng cùng bidder family/selection procedure, thay pCTR;
   so bidder bằng cùng pCTR và môi trường. Nếu thay cả hai, gọi so sánh hệ thống,
   không quy toàn bộ lợi ích cho CAFIN. Đồng nhất ngân sách/giá/quy tắc replay.
6. Viết Related Work và Algorithm ngay. Giữ queue hiện tại; chuẩn bị cohort
   external riêng trước khi xem test cho đối chiếu đó. Không chạy GPU thứ hai.
7. Hoàn tất validation/policy rồi test. Nếu không có ưu thế, báo trade-off hoặc
   đóng góp empirical về protocol/campaign/calibration; không tuyên bố SOTA.

Đã xác minh paper/README và sự tồn tại repo, chưa audit toàn bộ code/giấy phép,
chưa benchmark môi trường của repo tác giả. Không cam kết thời hạn cho phần này.
