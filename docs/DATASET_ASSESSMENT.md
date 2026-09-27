# Đánh giá dữ liệu iPinYou cho CAFIN–RTB

Ngày đánh giá: 19/09/2026. Phạm vi mới: quét **toàn bộ 15.395.258 dòng packaged
train**, tách thống kê theo split hiện hữu; không lấy mẫu để suy rộng. Đã đối chiếu
SHA-256 file train/vocabulary và số dòng, nhãn, UNK với manifest preprocessing.
Không đổi dữ liệu, split hoặc config huấn luyện. Thống kê test dưới đây lấy từ
manifest đã có; không thực hiện inference hoặc phân tích phân bố test mới.

## Kết luận sử dụng

**Dữ liệu x1 hợp lệ để chạy CTR, nhưng split lấy 10% cuối làm validation có độ
lệch campaign rất lớn. Chưa nên dùng nó như bằng chứng duy nhất cho benchmark
CTR tổng quát hoặc hiệu quả RTB.** Vấn đề đã xác nhận nằm ở thành phần campaign
của split, không có bằng chứng cho thấy file tải bị hỏng.

Raw season 2 có các nhóm file cần cho RTB nhưng mới xác minh archive; chưa đủ
để gọi là dataset chronological/RTB đã chuẩn bị xong. Hai archive là hai biểu
diễn của họ dữ liệu iPinYou, không phải hai bộ benchmark độc lập đã hoàn tất.

## 1. Tính toàn vẹn và nhãn

Các checksum toàn archive và CSV đã xác minh trong
[local_data_audit.json](../results/diagnostics/local_data_audit.json).
Lần này quét lại 17 cột và nhãn nhị phân của toàn bộ packaged train: khớp
manifest; 16 input fields và target click.

| Split | Dòng | Click | CTR | Không click / 1 click |
|---|---:|---:|---:|---:|
| Train hiện tại | 13.855.732 | 9.685 | 0,069899% | 1.429,64 |
| Validation | 1.539.526 | 1.872 | 0,121596% | 821,40 |
| Packaged test — thống kê cũ | 4.100.716 | 3.008 | 0,073353% | — |

CTR validation cao **1,740 lần** CTR train. Dữ liệu rất mất cân bằng; số lượng
triệu dòng không đồng nghĩa có nhiều quan sát click trong từng campaign.
Ví dụ validation advertiser 937615 chỉ có 26 click.

Không thấy chuỗi rỗng ở bất kỳ input field nào trong 15.395.258 dòng vừa quét.
Điều này chỉ mô tả CSV đã xử lý: không xác minh được missing gốc đã bị thay bằng
mã category trước khi đóng gói x1 hay chưa. Các số như slotwidth/slotprice ở đây
là giá trị đã mã hóa; không suy chúng là pixel hoặc giá đấu thầu gốc.

## 2. Campaign shift do điểm cắt validation

Train có **7 advertiser**, validation có **4**, chỉ **2 advertiser giao nhau**.
Tổng cộng packaged train có 9 advertiser.

| Advertiser x1 | Train: dòng / click | Validation: dòng / click | Có trong train? |
|---|---:|---:|---|
| 937614 | 3.083.056 / 2.454 | 0 / 0 | Có |
| 937615 | 766.761 / 254 | 68.795 / 26 | Có |
| 937616 | 0 / 0 | 687.617 / 207 | **Không** |
| 937617 | 851.884 / 590 | 470.677 / 253 | Có |
| 937618 | 0 / 0 | 312.437 / 1.386 | **Không** |
| 937619 | 1.742.104 / 1.358 | 0 / 0 | Có |
| 937620 | 2.847.802 / 2.076 | 0 / 0 | Có |
| 937621 | 2.593.765 / 1.926 | 0 / 0 | Có |
| 937622 | 1.970.360 / 1.027 | 0 / 0 | Có |

Hai advertiser chưa thấy chiếm **1.000.054 dòng (64,9586%) và 1.593 click
(85,0962%) của validation**. Riêng advertiser 937618 chiếm 74,0385% click
validation và có CTR 0,4436096%.

Đã xác minh trực tiếp thứ tự dòng nguồn:

- Validation bắt đầu tại dòng dữ liệu **13.855.733** (không tính header).
- Advertiser 937618 chỉ bắt đầu xuất hiện tại dòng **14.224.034**.
- Advertiser 937616 chỉ bắt đầu xuất hiện tại dòng **14.450.761**.

Vì vậy, việc thiếu hai advertiser trong train là hệ quả xác định của điểm cắt
hiện tại trên thứ tự file. Chưa xác minh vì sao nguồn đóng gói có thứ tự này;
không gán nó cho season hoặc chronology khi x1 không có timestamp đầy đủ.
Đây cũng chưa phải protocol holdout advertiser được thiết kế cân bằng: validation
trộn cả advertiser đã thấy và chưa thấy, đồng thời bỏ 5 advertiser của train.

![Campaign theo vị trí dòng nguồn](../results/figures/dataset_assessment/row_order.png)

## 3. Category chưa thấy và UNK

Vocab chỉ fit train, min_frequency=2. Phân rã chính xác từ chuỗi category trong
CSV trước bước encoding local, không suy mọi UNK đều là category mới:

| Field | Validation chưa thấy trong train | Đã thấy nhưng bị lọc hiếm | Tổng UNK |
|---|---:|---:|---:|
| advertiser | 64,9586% | 0% | 64,9586% |
| creative | 64,9586% | 0% | 64,9586% |
| slotwidth | 26,1944% | 0% | 26,1944% |
| slotheight | 20,5622% | 0% | 20,5622% |
| adexchange | 20,2944% | 0% | 20,2944% |
| slotid | 15,3947% | 0,2459% | 15,6406% |
| IP | 5,2623% | 1,8072% | 7,0694% |
| domain | 6,2992% | 0,4321% | 6,7314% |

Không có chuỗi rỗng nên thành phần missing trong UNK ở lần quét này bằng 0.
Giảm min_frequency xuống 1 có thể giảm phần lọc hiếm; **không thể khắc phục
advertiser/creative chưa từng xuất hiện trong train**.

Train có 669.671 category IP khác nhau, giữ 561.959 sau lọc; slotid có 165.489,
giữ 47.682. Cardinality cao làm tăng bảng embedding và độ thưa; các con số này là
số mã trong x1, không khẳng định tương ứng người dùng hay slot gốc duy nhất.

JSD train→validation trên encoded features (log cơ số 2, miền [0,1]): advertiser
0,830725; creative 0,831039; slotvisibility 0,733752; slotformat 0,730056;
domain 0,691396. Có field UNK bằng 0 nhưng JSD lớn: tất cả category đã thấy
không có nghĩa tần suất xuất hiện giữa hai split giống nhau.

![Tổng quan dữ liệu](../results/figures/dataset_assessment/overview.png)

## 4. Hệ quả đối với đánh giá mô hình

- Pooled metric trộn các campaign có base CTR khác nhau. Cần báo riêng nhóm
  advertiser đã thấy/chưa thấy và từng campaign; không diễn giải pooled AUC
  thành chất lượng trong mọi campaign.
- Đối chứng xác suất hằng bằng CTR train đạt validation LogLoss **0,00953339**
  (tính từ nhãn thật, không fit validation). Nên giữ đối chứng này cùng AUC,
  AP/PR-AUC, Brier và kiểm tra calibration. Không dùng accuracy làm kết quả chính.
- Không sửa bằng cách fit vocab trên validation/test hoặc loại campaign khó sau
  khi xem kết quả. Điều đó làm thay đổi câu hỏi nghiên cứu hoặc rò rỉ dữ liệu.
- Các kết quả hiện có vẫn dùng được nếu ghi chính xác `packaged_train_tail` và
  campaign shift. Không có bằng chứng để gọi split này là official validation,
  random hay chronological.

## 5. Bộ raw season 2 và phần chưa kiểm chứng

Archive local có **7 file imp, 7 file clk và 1 leaderboard**. Tổng dung lượng
các member `.bz2` tương ứng: 794.108.628; 733.477; 171.842.537 byte. Đây là dung
lượng file nén, không phải số impression hoặc dung lượng sau giải nén.
Các nhóm file này là đầu vào dự kiến cho import CTR/RTB; không cần toàn bộ bid
logs cho observed-impression replay đang thiết kế.

Chưa hoàn tất: checksum từng member, schema/nhãn toàn log, join click theo bid ID,
kiểm tra duplicate bid ID, payingprice, timestamp, campaign, chronology và độ
chồng lấn split. Known-bugs file trong archive có ghi chú lỗi logtype conversion
season 1; ghi chú này không chứng minh season 2 không có lỗi.

x1 không có timestamp/payingprice đủ dùng cho historical auction replay.
Không thay payingprice bằng slotprice. Season 3 chưa có local, nên chưa đánh giá
season 2→3. Lần này cũng **chưa kiểm tra đầy đủ bản ghi trùng hoặc overlap giữa
train/test**; feature rows giống nhau trong x1 không đủ để kết luận là cùng một
impression vì thiếu bid ID. Nhãn khớp schema/checksum không chứng minh nhãn
đúng với sự kiện ngoài hệ thống logs.

## 6. Đề xuất protocol trước benchmark cuối

Đây là đề xuất dựa trên audit, chưa được áp dụng vào dataset hiện tại:

1. Giữ split hiện tại cùng kết quả như một thí nghiệm thăm dò có campaign shift.
2. Cho benchmark standardized: kiểm tra và ưu tiên exact split recipe của nguồn
   benchmark nếu có. Nếu không có, khai báo một protocol mới tạo validation
   deterministic trong packaged train với phân tầng campaign và nhãn, kiểm tra
   trùng/nhóm để tránh chia cùng impression vào hai phần. Nó không phải temporal
   validation và không được gọi là official. Giữ packaged test riêng; fit vocab
   lại chỉ trên train mới, dùng output/run mới cho mọi model.
3. Cho câu hỏi chronology/RTB: dùng raw season 2, chọn boundary theo timestamp
   thật và kiểm tra campaign/label/price. Báo advertiser mới như nhóm riêng nếu
   chúng xuất hiện sau boundary; không buộc chia ngẫu nhiên để loại drift thật.
4. Chốt protocol và ngân sách tuning trước mở rộng 5 seeds/final test. Chỉ so
   sánh các run có cùng split và định nghĩa feature. Không gộp số liệu cũ/mới.

Đánh giá về tính phù hợp: **đủ dữ liệu cho thí nghiệm CTR; split hiện tại chưa
đại diện tốt cho đánh giá cùng campaign; dữ liệu raw chưa sẵn sàng cho RTB; chưa
đủ season 2/3**. Báo cáo dữ liệu này có thể đưa vào bản thảo phương pháp/giới hạn,
nhưng không thay cho các kết quả thực nghiệm còn thiếu.

## Artifact và tái lập

- [Thống kê JSON](../results/diagnostics/dataset_assessment.json)
- [Bảng feature CSV](../results/diagnostics/dataset_features.csv)
- [Bảng campaign CSV](../results/diagnostics/dataset_campaigns.csv)
- [Tổng quan PDF](../results/figures/dataset_assessment/overview.pdf)
- [Vị trí dòng PDF](../results/figures/dataset_assessment/row_order.pdf)
- Scripts: `scripts/assess_dataset.py`, `scripts/plot_dataset_assessment.py`.

Chạy scripts với Python dự án và `PYTHONPATH=src`. JSON lưu dataset/source hash,
thời điểm quét và phạm vi; figures có provenance SHA-256 của JSON đầu vào.
Hàm phân rã empty/rare/unseen đã kiểm tra trường hợp nhỏ độc lập; toàn bộ kết quả
UNK thực tế khớp manifest. Không lưu các mã IP riêng lẻ trong báo cáo aggregate.
