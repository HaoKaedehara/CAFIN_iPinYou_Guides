## Khoảng trống nghiên cứu, bằng chứng thực nghiệm và giải thích CAFIN

Cập nhật 27/09/2026. Tổng hợp từ 105 test_metrics thật (P1: 75, P2: 30), 30 replay và audit dữ liệu. Các phép đối chiếu theo seed trong tài liệu này là phân tích hậu nghiệm trên kết quả đã đóng băng; không phải giả thuyết thống kê đăng ký trước. Không huấn luyện, suy luận test hoặc thay đổi cấu hình trong lần tổng hợp này.

## 1. Luận điểm nghiên cứu và khoảng trống có thể bảo vệ

Câu hỏi trung tâm: trong các protocol iPinYou được mô tả minh bạch, việc phối hợp tương tác nhân tường minh với biểu diễn attention có cải thiện chất lượng xác suất, thứ hạng và số click dưới ngân sách một cách nhất quán hay không? CAFIN new_design_v1 là thiết kế mới của dự án; chưa có căn cứ gọi là tái tạo CAFIN Criteo, phương pháp đầu tiên hoặc SOTA.

Khoảng trống được định vị là nhu cầu kiểm chứng chung các yếu tố trên đối với thiết kế cụ thể này, không phải tuyên bố toàn bộ literature chưa nghiên cứu CTR–RTB. Zhang et al. đã đánh giá cả CTR và bidding; DeepFM/xDeepFM đã kết hợp các dạng tương tác. Do đó chỉ ghép hai nhánh hoặc nối CTR với RTB chưa đủ làm đóng góp mới. Giá trị hiện có nằm ở thực nghiệm có truy vết và các kết quả xác định giới hạn của thiết kế.

| Câu hỏi / khoảng trống cần kiểm chứng | Thí nghiệm trả lời | Kết luận từ bằng chứng hiện có |
| --- | --- | --- |
| G1. Kết luận có phụ thuộc split, campaign và prior click? | P0 audit; P1 grouped holdout; P2 chronological; kiểm tra vocabulary và calibration. | P0 có campaign shift lớn; không gộp điểm P0/P1/P2 để xếp hạng. Chưa có kết luận cross-season. |
| G2. Thành phần nào thực sự đóng góp trong CAFIN? | P1: 9 baseline, 5 ablation, full CAFIN; ghép cặp 5 seed. | Gate cải thiện AUC 5/5 seed so với fusion tuyến tính cùng số tham số; full CAFIN chưa hơn CrossOnly. |
| G3. Chất lượng CTR có chuyển thành utility dưới ngân sách? | P2: 6 model, 5 seed; cùng họ bidder/grid, β chọn bằng validation; 5 mức budget. | Có đánh đổi clicks/eCPC và phụ thuộc budget. Không suy AUC tốt hơn luôn thắng replay. |
| G4. Lợi ích có tương xứng chi phí và đủ để vượt prior work? | Số tham số/thời gian từ status; đối chiếu nguồn và protocol paper. | Chưa chứng minh hiệu quả tính toán hoặc hơn code tác giả; số liệu literature khác protocol chỉ làm bối cảnh. |

## 2. Các nghiên cứu liên quan và vị trí của bài

| Bài báo | Đã giải quyết | Liên hệ với khoảng trống |
| --- | --- | --- |
| Zhang et al. (2014; cập nhật 2015) | Benchmark iPinYou cho CTR và bidding. | G1/G3: cần công bố split, quần thể và quy tắc replay; không nhận việc nối CTR–RTB là mới. |
| Qu et al. (ICDM 2016), PNN | Embedding, product layer và mạng fully connected. | IPNN local là đối chứng mạnh nhất về mean AUC/LogLoss/AP ở P1; cần giữ trong kết luận. |
| Guo et al. (2017), DeepFM | Kết hợp FM và deep network với đầu vào dùng chung. | G2: CAFIN có lợi so với DeepFM ở một số metric, nhưng kết hợp tương tác tự nó không mới. |
| Wang et al. (WWW 2021; preprint 2020), DCNv2 | Cross network tăng năng lực biểu diễn tương tác. | G2: nguồn nhánh matrix-cross; local dùng full matrix, không mixture-of-experts. |
| Lian et al. (KDD 2018), xDeepFM | CIN tường minh phối hợp DNN ngầm định. | G2: đối chiếu một cách kết hợp tương tác khác; local CIN full non-split, identity activation. |
| Song et al. (CIKM 2019), AutoInt | Multi-head self-attention học tương tác giữa các field. | G2/G3: local AutoInt dẫn AUC/LogLoss P2; không được lược bỏ vì bất lợi cho CAFIN. |
| Huang, Chen, Deng (2020), FO-FTRL-DCN | DCN, FTRL và xử lý đặc trưng/SMOTE trên iPinYou. | Chỉ đối chiếu tài liệu/số công bố; chưa chạy code tác giả, khác advertiser/split/preprocessing. |
| Cai et al. (WSDM 2017), RLB | Bidding tuần tự bằng reinforcement learning có trạng thái ngân sách. | G3: một hướng baseline bidder còn thiếu. Không lấy AUC để so CAFIN với một thuật toán bidding. |

[Zhang et al. (2014; cập nhật 2015)](https://arxiv.org/abs/1407.7073)

[Qu et al. (ICDM 2016), PNN](https://arxiv.org/abs/1611.00144)

[Guo et al. (2017), DeepFM](https://arxiv.org/abs/1703.04247)

[Wang et al. (WWW 2021; preprint 2020), DCNv2](https://arxiv.org/abs/2008.13535)

[Lian et al. (KDD 2018), xDeepFM](https://arxiv.org/abs/1803.05170)

[Song et al. (CIKM 2019), AutoInt](https://arxiv.org/abs/1810.11921)

[Huang, Chen, Deng (2020), FO-FTRL-DCN](https://www.mdpi.com/1999-4893/13/12/342)

[Cai et al. (WSDM 2017), RLB](https://arxiv.org/abs/1701.02490)

Đã mở nguồn arXiv cho bảy bài tương ứng trong phiên này. Trang MDPI không truy cập được khi kiểm tra lại; thông tin và số FO-FTRL-DCN kế thừa bản trích Table 2 cùng provenance đã lưu ngày 26/09, không tuyên bố vừa xác minh lại bảng nguồn. Đây là rà soát có mục tiêu, chưa phải systematic review đến năm 2026. Mọi hàng baseline P1/P2 là implementation local mô tả tại SOURCES.md, không phải kết quả tái chạy repository tác giả.

## 3. Dữ liệu thực nghiệm và câu trả lời G1

| Protocol / split | Số impression | Positive | CTR (%) |
| --- | --- | --- | --- |
| P0 / train | 13,855,732 | 9,685 | 0.06990 |
| P0 / valid | 1,539,526 | 1,872 | 0.12160 |
| P0 / test | 4,100,716 | 3,008 | 0.07335 |
| P1 / train | 13,855,384 | 10,411 | 0.07514 |
| P1 / valid | 1,539,874 | 1,146 | 0.07442 |
| P1 / test | 4,100,716 | 3,008 | 0.07335 |
| P2 / train | 8,770,137 | 5,959 | 0.06795 |
| P2 / valid | 3,375,166 | 2,722 | 0.08065 |
| P2 / test | 2,511,211 | 1,854 | 0.07383 |

P0 lấy 10% cuối packaged train làm validation, chỉ dùng thăm dò với seed 11. P1 hash toàn bộ 16 giá trị feature, độc lập nhãn, để chia train/validation xấp xỉ 90/10; giữ nguyên packaged test. P1 không phải official split, chronological split hoặc bảo đảm độc lập user/IP. P0 và P1 dùng cùng packaged test; bảng số đếm P0 test không có nghĩa đã thực hiện một cohort P0 test. P2 dùng raw Season 2: train 06–10/06/2013, valid 11–12/06, test 13–15/06; loại mọi impression mang bid ID không duy nhất trong tập nhập. P2 là conservative singleton subset, không phải full-raw benchmark. P1/P2 là hai protocol cùng nguồn iPinYou, không phải hai dataset độc lập.

Audit P0 phát hiện 1,000,054 impression validation ở campaign chưa xuất hiện trong train, chứa 1,593/1,872 click (85.10%). CAFIN seed 11 có tỷ lệ tổng click dự đoán/thực tế 5.054 ở P0 validation, so với 1.058 ở P1 validation. Đây là dấu hiệu lệch mức xác suất trong protocol cũ, không phải ước lượng hiệu ứng nhân quả của đổi split: quần thể validation và dữ liệu train đều đổi.

Một giới hạn cụ thể khác: manifest P2 ghi UNK của weekday bằng 100% trên validation và 0% trên test. Train 06–10/06 chỉ chứa năm thứ; hai thứ 11–12/06 chưa có trong vocabulary train. Vì vậy validation không hoàn toàn đại diện test về weekday. Đây là đặc tính split/vocabulary cần báo cáo, chưa đo được tác động riêng lên mô hình. Không sửa encoding sau khi đã xem test; thí nghiệm encoding tiếp theo phải khai báo riêng. UNK của các trường nói chung còn gộp rare/missing/unseen.

Vocabulary chỉ fit train; checkpoint chọn bằng validation LogLoss, AUC phá hòa; test đã mở sau freeze. Không dùng số liệu hậu nghiệm này để tune lại. Các hàng P1/P2 vẫn cùng cấu hình cố định và seed set 11/23/42/71/101: Adam lr 0.001, batch 512, embedding 16, hidden 128/64, dropout 0.1, tối đa 30 epoch, patience 3. Chưa có ngân sách tìm hyperparameter tương xứng cho từng model; cùng learning rate không đồng nghĩa tuning công bằng.

## 4. Kết quả CTR và ablation: câu trả lời G2

Mỗi ô dưới là mean ± sample SD của năm seed; SD không phải confidence interval. AP là average precision, khác PR-AUC tích phân hình thang. Hai protocol được trình bày riêng.

Bảng P1: kết quả test đầy đủ, sắp theo mean ROC-AUC giảm dần.

| Model | ROC-AUC ↑ | LogLoss ↓ | AP ↑ |
| --- | --- | --- | --- |
| IPNN | 0.759549 ± 0.004032 | 0.005618 ± 0.000015 | 0.007011 ± 0.000708 |
| DCN | 0.759395 ± 0.005697 | 0.005621 ± 0.000017 | 0.006610 ± 0.000426 |
| CAFIN_CrossOnly | 0.758432 ± 0.002853 | 0.005623 ± 0.000016 | 0.006495 ± 0.000289 |
| DNN | 0.758390 ± 0.004123 | 0.005624 ± 0.000017 | 0.006504 ± 0.000466 |
| AutoInt | 0.758358 ± 0.005166 | 0.005620 ± 0.000024 | 0.006718 ± 0.000480 |
| CAFIN | 0.758005 ± 0.003940 | 0.005622 ± 0.000015 | 0.006458 ± 0.000261 |
| DCNv2 | 0.757510 ± 0.003821 | 0.005630 ± 0.000019 | 0.006836 ± 0.000877 |
| CAFIN_DNN | 0.757366 ± 0.001822 | 0.005627 ± 0.000008 | 0.006348 ± 0.000120 |
| LR | 0.757188 ± 0.001848 | 0.005627 ± 0.000007 | 0.006356 ± 0.000118 |
| xDeepFM | 0.756741 ± 0.004926 | 0.005625 ± 0.000021 | 0.006633 ± 0.000241 |
| CAFIN_CrossAttention | 0.754684 ± 0.003694 | 0.005630 ± 0.000014 | 0.006399 ± 0.000185 |
| CAFIN_Concat | 0.754644 ± 0.001208 | 0.005635 ± 0.000006 | 0.006250 ± 0.000233 |
| CAFIN_AttentionOnly | 0.754432 ± 0.002099 | 0.005634 ± 0.000007 | 0.006393 ± 0.000180 |
| DeepFM | 0.753052 ± 0.007877 | 0.005632 ± 0.000024 | 0.006399 ± 0.000423 |
| FM | 0.746889 ± 0.007758 | 0.005660 ± 0.000031 | 0.005952 ± 0.000733 |

Bảng P2: kết quả test đầy đủ, sắp theo mean ROC-AUC giảm dần.

| Model | ROC-AUC ↑ | LogLoss ↓ | AP ↑ |
| --- | --- | --- | --- |
| AutoInt | 0.744038 ± 0.003315 | 0.005686 ± 0.000012 | 0.007019 ± 0.001398 |
| DCNv2 | 0.741108 ± 0.003616 | 0.005687 ± 0.000010 | 0.007227 ± 0.000724 |
| CAFIN | 0.740676 ± 0.004028 | 0.005709 ± 0.000013 | 0.005280 ± 0.000470 |
| LR | 0.738403 ± 0.004030 | 0.005723 ± 0.000018 | 0.004807 ± 0.000309 |
| DeepFM | 0.733599 ± 0.001751 | 0.005725 ± 0.000013 | 0.005458 ± 0.000420 |
| FM | 0.730305 ± 0.003310 | 0.005735 ± 0.000012 | 0.005545 ± 0.000538 |

P1: IPNN dẫn mean AUC, LogLoss và AP trong các model đầy đủ; CAFIN kém IPNN về AUC ở cả 5/5 seed. P2: AutoInt dẫn AUC/LogLoss, DCNv2 dẫn AP. CAFIN tốt hơn DeepFM về mean AUC và LogLoss trong cả hai protocol, nhưng AP P2 thấp hơn DeepFM. Vì vậy bằng chứng cho một số cải thiện cục bộ không đủ để kết luận CAFIN tốt nhất hoặc mọi metric cùng cải thiện.

| Đối chiếu A − B trên P1 | Δ AUC mean ± SD | Số seed A tốt hơn B | Δ LogLoss mean ± SD |
| --- | --- | --- | --- |
| CAFIN_CrossOnly − CAFIN_DNN | 0.0010660 ± 0.0025889 | 2/5 | -0.00000491 ± 0.00000953 |
| CAFIN_AttentionOnly − CAFIN_DNN | -0.0029343 ± 0.0037875 | 0/5 | 0.00000647 ± 0.00000996 |
| CAFIN_Concat − CAFIN_CrossOnly | -0.0037879 ± 0.0036348 | 0/5 | 0.00001264 ± 0.00001315 |
| CAFIN_CrossAttention − CAFIN_Concat | 0.0000402 ± 0.0037730 | 3/5 | -0.00000498 ± 0.00001373 |
| CAFIN − CAFIN_CrossAttention | 0.0033206 ± 0.0033908 | 5/5 | -0.00000827 ± 0.00001168 |
| CAFIN − CAFIN_CrossOnly | -0.0004271 ± 0.0059251 | 3/5 | -0.00000062 ± 0.00002417 |
| CAFIN − DeepFM | 0.0049525 ± 0.0064854 | 3/5 | -0.00001042 ± 0.00000920 |
| CAFIN − IPNN | -0.0015435 ± 0.0013499 | 0/5 | 0.00000360 ± 0.00000536 |

![Chênh lệch theo từng seed giúp nhìn độ ổn định của ablation; Δ LogLoss âm mới là tốt hơn. Các seed dùng chung test set, không phải năm mẫu dữ liệu độc lập.](../results/research_gap_evidence/paired_auc.png)

Gate: full CAFIN so với CAFIN_CrossAttention có cùng tổng số tham số và chỉ khác toán tử fusion. Δ AUC trung bình +0.0033206, tăng ở 5/5 seed; LogLoss thấp hơn ở 4/5 seed. Gate g=σ(W[u;v]+b) giới hạn phép trộn từng chiều h=g⊙u+(1−g)⊙v, trong khi projection tuyến tính không có ràng buộc này. Suy luận phù hợp với kết quả là phép trộn có ràng buộc và phụ thuộc đầu vào có thể giúp fusion ở cấu hình hiện tại. Chưa đo phân bố gate, perturbation hay khả năng chống nhiễu nên chưa chứng minh gate tự phát hiện đặc trưng nhiễu hoặc tạo giải thích nhân quả.

Cross-attention: CAFIN_CrossAttention so với CAFIN_Concat chỉ tăng mean AUC +0.0000402; SD chênh lệch 0.0037730. Dữ liệu chưa ủng hộ lợi ích AUC ổn định của bước trao đổi hai chiều. Full CAFIN so với CrossOnly có Δ AUC −0.0004271 và Δ LogLoss −0.00000062: thêm toàn bộ context/cross-attention/gate không cho ưu thế nhất quán. CrossOnly dùng mean pooling và head riêng; không đồng nhất với baseline DCNv2. CAFIN_DNN cũng là control pooling, khác DNN flatten.

Về cơ chế, phép cross nhân biểu diễn gốc với biến đổi tuyến tính của trạng thái hiện tại, cho phép tạo các tương tác nhân tường minh. Self-attention điều chỉnh tổng hợp field theo biểu diễn đầu vào; cross-attention cho phép hai nhánh trao đổi thông tin trước pooling. Đây là lý do thiết kế hợp lý để thử nghiệm, không tự chứng minh hiệu quả. Thêm nhánh có thể dư thừa, pooling có thể làm mất thông tin và cấu hình có thể chưa tối ưu; các khả năng này mới là giả thuyết giải thích kết quả âm, chưa được đo tách biệt.

Phép ghép cặp ở đây dùng cùng seed ID giữa các kiến trúc; không bảo đảm cùng quỹ đạo số ngẫu nhiên vì số tham số và phép toán khác nhau. Không báo p-value hoặc tuyên bố ý nghĩa thống kê từ 5/5 lần tăng; còn nhiều đối chiếu hậu nghiệm, ít seed và chưa có CI theo campaign/thời gian. Chưa có ablation P2 nên không chuyển kết luận gate của P1 sang raw RTB.

## 5. Xác suất, click dưới ngân sách và câu trả lời G3

| Protocol | LogLoss constant train-CTR | LogLoss CAFIN | Brier constant train-CTR | Brier CAFIN |
| --- | --- | --- | --- | --- |
| P1 | 0.00602784 | 0.00562195 ± 0.00001475 | 0.00073299 | 0.00073088 ± 0.00000023 |
| P2 | 0.00606443 | 0.00570920 ± 0.00001327 | 0.00073775 | 0.00073621 ± 0.00000020 |

Constant train-CTR lấy xác suất từ train và tính loss trên nhãn test đã có bằng công thức giải tích; không dùng test CTR làm xác suất dự báo. Việc CAFIN vượt hằng số cho thấy đã học tín hiệu, nhưng không chứng minh vượt baseline mạnh. Brier/LogLoss phản ánh chất lượng xác suất tổng thể, không cô lập calibration; ở CTR rất hiếm, Brier thấp và accuracy cao vẫn có thể đạt bằng dự đoán gần 0. Phải đọc cùng AP, ROC-AUC và calibration theo bin.

| Model P2, budget 1/8 | Clicks ↑ | eCPC ↓ | Spend | Dùng ngân sách (%) |
| --- | --- | --- | --- | --- |
| AutoInt | 589.6 ± 14.9 | 44079.8 ± 1659.9 | 25973322 ± 584769 | 97.23 ± 2.19 |
| CAFIN | 575.8 ± 23.7 | 42608.0 ± 1323.3 | 24540215 ± 1385453 | 91.86 ± 5.19 |
| DCNv2 | 568.4 ± 21.7 | 46713.8 ± 2134.6 | 26515115 ± 185094 | 99.25 ± 0.69 |
| DeepFM | 468.8 ± 54.7 | 42766.6 ± 3010.4 | 20011951 ± 2324702 | 74.91 ± 8.70 |
| FM | 412.0 ± 81.9 | 37338.1 ± 4476.9 | 15663008 ± 4804693 | 58.63 ± 17.99 |
| LR | 338.4 ± 60.2 | 36589.0 ± 2387.5 | 12425082 ± 2695864 | 46.51 ± 10.09 |

| Budget | CAFIN clicks | AutoInt clicks | DCNv2 clicks | CAFIN / AutoInt eCPC mean |
| --- | --- | --- | --- | --- |
| 1/32 | 198.6 ± 12.1 | 259.8 ± 11.3 | 259.6 ± 10.5 | 18718.8 / 22023.6 |
| 1/16 | 246.6 ± 26.7 | 297.2 ± 9.1 | 306.4 ± 17.8 | 26971.7 / 29152.1 |
| 1/8 (chính) | 575.8 ± 23.7 | 589.6 ± 14.9 | 568.4 ± 21.7 | 42608.0 / 44079.8 |
| 1/4 | 723.2 ± 42.6 | 832.8 ± 52.0 | 852.8 ± 30.8 | 45737.1 / 48231.8 |
| 1/2 | 1287.0 ± 42.6 | 1343.4 ± 24.8 | 1365.6 ± 30.2 | 64862.1 / 69674.1 |

Ở budget chính 1/8, CAFIN thu ít click hơn AutoInt 2.34%, đồng thời eCPC thấp hơn 3.34%. CAFIN mua ít click hơn với chi phí trung bình mỗi click thấp hơn; không thể gọi là thắng khi mục tiêu chọn policy là tối đa click. Spend và mức sử dụng ngân sách trong bảng giúp giải thích sự đánh đổi. eCPC là trung bình của spend/click tính riêng từng seed, không phải tỷ số của hai trung bình; giá giữ đơn vị gốc dataset.

| Campaign P2, budget 1/8 | CAFIN clicks | AutoInt clicks | Δ clicks CAFIN − AutoInt |
| --- | --- | --- | --- |
| 1458 | 128.0 ± 3.7 | 128.6 ± 5.8 | -0.6 |
| 3358 | 93.2 ± 8.9 | 109.0 ± 4.2 | -15.8 |
| 3386 | 168.0 ± 6.3 | 151.4 ± 14.8 | +16.6 |
| 3427 | 111.4 ± 6.1 | 115.6 ± 4.4 | -4.2 |
| 3476 | 75.2 ± 13.8 | 85.0 ± 10.1 | -9.8 |

Phân rã theo campaign giải thích số tổng: CAFIN hơn AutoInt 16.6 click trung bình ở campaign 3386, nhưng kém 15.8 click ở 3358 và 9.8 click ở 3476; cộng đủ năm campaign cho chênh lệch −13.8 click. Đây là phân rã quan sát, không phải chứng minh nguyên nhân. Kết quả cho thấy lợi ích cục bộ có thể bị bù trừ khi tổng hợp; không được chỉ chọn campaign thuận lợi để kết luận. Tại 1/8, DCNv2 có AUC/AP tốt hơn CAFIN nhưng ít click hơn, là phản ví dụ cụ thể cho việc dùng riêng metric CTR để suy thứ hạng replay.

Công thức bid=β·pCTR/train_CTR sử dụng mức xác suất, trong khi AUC chỉ phụ thuộc thứ hạng; giá đấu, thời điểm và ngân sách còn quyết định impression được mua. Vì thế AUC/AP không xác định duy nhất clicks/eCPC. β được tune riêng theo model/campaign/budget bằng cùng grid và quy tắc validation: so sánh đo hệ thống pCTR cộng thủ tục chọn bidder chung, không phải giữ nguyên một β. Một phép nhân đều pCTR có thể được β bù lại nếu grid phù hợp; không quy mọi khác biệt replay cho calibration.

Replay chỉ quan sát impression logs lịch sử, chịu selection/censoring; không phải online lift hay hiệu ứng nhân quả. CAFIN chưa được so trực tiếp với RLB/IIBidder trong môi trường đồng nhất. Kết quả ở một budget thuận lợi không thay thế kết quả tại budget chính 1/8 đã khai báo.

## 6. Chi phí, so sánh literature và câu trả lời G4

| P1 model | Tổng tham số | Tham số ngoài embedding | Train phút mean ± SD |
| --- | --- | --- | --- |
| LR | 650,762 | 1 | 13.6 ± 3.8 |
| IPNN | 10,468,753 | 56,577 | 52.7 ± 23.1 |
| DeepFM | 11,104,155 | 41,218 | 39.3 ± 9.7 |
| AutoInt | 10,414,481 | 2,305 | 61.1 ± 28.8 |
| CAFIN_CrossOnly | 11,205,051 | 142,114 | 50.6 ± 18.5 |
| CAFIN_CrossAttention | 11,210,059 | 147,122 | 88.9 ± 34.1 |
| CAFIN | 11,210,059 | 147,122 | 80.1 ± 27.5 |

Thời gian lấy từ status của các run thực; gồm số epoch khác nhau do early stopping và có thể chịu trạng thái máy/resume. Đây không phải benchmark latency có kiểm soát. Số tham số full CAFIN và CrossAttention phải bằng nhau trong audit; các ablation khác không được cân bằng toàn bộ dung lượng. Chưa có căn cứ gọi full CAFIN hiệu quả tính toán hơn nhánh đơn hoặc baseline.

| FO-FTRL-DCN, Huang et al. Table 2 | AUC công bố | LogLoss thang thường |
| --- | --- | --- |
| 1458 | 0.8338 | 0.004480 |
| 3358 | 0.8969 | 0.004697 |
| 3386 | 0.8040 | 0.005891 |
| 3427 | 0.8574 | 0.004564 |

Bảng literature trên chỉ cung cấp bối cảnh: bốn advertiser riêng, bảy ngày train/ba ngày test và xử lý dữ liệu khác; LogLoss đã đổi từ thang 10⁻² của paper. Không tính phần trăm CAFIN hơn/kém các số này, không ghép vào bảng xếp hạng P1/P2. Đối chiếu trực tiếp hợp lệ hiện tại là giữa implementation local cùng protocol. Để so prior work ở mức tái lập cần pin code/config tác giả, đồng nhất population/split/features/sampling, ngân sách tuning và bidder.

## 7. Đoạn kết luận có thể dùng trong bài và phần còn thiếu

Thực nghiệm trên iPinYou cho thấy lợi ích của CAFIN new_design_v1 phụ thuộc thành phần và tiêu chí đánh giá. Trong P1, gate cải thiện AUC nhất quán qua năm seed so với fusion tuyến tính có cùng số tham số, nhưng cross-attention hai chiều chưa tạo cải thiện AUC ổn định và mô hình đầy đủ chưa vượt CrossOnly. CAFIN cải thiện một số metric so với DeepFM, trong khi IPNN và AutoInt vẫn là đối chứng mạnh ở P1 và P2. Trong replay P2, CAFIN có eCPC thấp hơn AutoInt ở ngân sách 1/8 nhưng cũng thu được ít click hơn. Những kết quả này ủng hộ đánh giá đồng thời protocol, ablation và utility; không ủng hộ tuyên bố ưu thế phổ quát của kiến trúc đề xuất.

| Muốn khẳng định | Bằng chứng bổ sung cần có |
| --- | --- |
| Gate hoạt động nhờ chọn tín hiệu tốt/loại nhiễu | Đo phân bố/saturation gate, perturbation và control gate hằng số trên protocol mới khai báo; không dùng attention weight làm bằng chứng nhân quả. |
| Cross-attention cần thiết cho RTB | Ablation đầy đủ ở P2, cùng bidder procedure và budget; hiện chưa có. |
| Hiệu quả bền vững qua miền/thời gian | Season 3 hoặc dataset độc lập, campaign/time uncertainty và sensitivity của singleton exclusion. |
| Vượt prior work hoặc có ý nghĩa thống kê | Tuning tương xứng, chạy code tác giả, thêm seed/CI phù hợp cấu trúc dữ liệu và kiểm soát nhiều đối chiếu. |
| Calibration gây cải thiện bidding | Control hiệu chỉnh xác suất học trên validation và fixed-pCTR/fixed-bidder phù hợp; chỉ dùng kết quả mới theo protocol khai báo. |

Nguồn số liệu chính: results/x1_grouped_test_summary.csv; results/season2_test_summary.csv; experiments/{x1_grouped,season2}/runs/*/{test_metrics,status}.json; experiments/season2/runs/*/rtb_results.json; data/processed/*/manifest.json; results/diagnostics/dataset_assessment.json; results/calibration. JSON provenance kèm báo cáo lưu SHA-256 từng đầu vào đã đọc. Các giới hạn trên là phần chưa có bằng chứng, không được viết như thí nghiệm đã hoàn tất.
