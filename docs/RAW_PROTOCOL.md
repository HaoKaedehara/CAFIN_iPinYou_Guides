# Nhập raw và leaderboard iPinYou

CAFIN new_design_v1 dùng nhãn CTR nhị phân. Thiết bị thí nghiệm hiện tại là máy
Windows theo quyết định mới của người dùng. [Nguồn và mức kiểm chứng](DATA_SOURCES.md) được
tách rõ: metadata, mẫu schema nhỏ và kiểm tra toàn bộ là các bước khác nhau.

## Hai định dạng có bộ đọc riêng

| Input | Lệnh | Nhãn |
|---|---|---|
| Impression và click logs headerless 24 cột | `import-raw` | Có bid ID trong click logs tương ứng |
| Leaderboard season 2/3 headerless 26 cột | `import-leaderboard` | Số click ở cột 25 lớn hơn 0 |
| Log đã có header và nhãn nhị phân | `normalize-logs` | Cột click sẵn có |

Nguồn schema: [tài liệu iPinYou, mục 2.4 và bảng 3](https://contest.ipinyou.com/ipinyou-dataset.pdf),
[schema 24 cột](https://github.com/wnzhang/make-ipinyou-data/blob/master/schema.txt)
và [mktest.py của Weinan Zhang](https://github.com/wnzhang/make-ipinyou-data/blob/master/python/mktest.py).
Mẫu nhỏ season 2 trên Figshare đã khớp số cột và logtype; chưa kiểm tra toàn file.

Leaderboard gồm 24 cột impression, sau đó là số click nguyên không âm và cờ
conversion 0/1. Adapter yêu cầu logtype=1 và bid ID không rỗng. Cột 21 là payprice;
cột 18 slotprice không được dùng thay thế. Cờ conversion không tạo nhãn click.

Số click 2 hoặc lớn hơn vẫn cho nhãn CTR 1. Vì vậy, chỉ tiêu clicks trong pipeline
là số impression có click, không phải tổng số sự kiện click khi một impression
có nhiều click. Nhãn và cách tính này thống nhất với training join nhị phân.
Conversion/click count không được đưa vào feature set.

Ví dụ sau khi giải nén season 2 và chọn đủ các file tương ứng:

```bash
python -m cafin import-raw --impressions /content/raw/training2nd/imp.*.txt.bz2 --clicks /content/raw/training2nd/clk.*.txt.bz2 --season 2 --output /content/canonical/season2_train.csv
python -m cafin import-leaderboard --inputs /content/raw/testing2nd/leaderboard.test.data.20130613_15.txt.bz2 --season 2 --output /content/canonical/season2_test.csv
```

Đường dẫn trên là ví dụ Bash; đổi `/content/raw` thành thư mục giải nén thực tế.
Trên notebook Python, dùng `Path.glob()` và truyền danh sách đường dẫn vào
`run_cli`; wildcard không được subprocess tự mở rộng.

Hai lệnh sắp theo timestamp/bid ID bằng SQLite, từ chối bid ID trùng trong một
lần nhập, ghi manifest nguồn và hash output. Output tồn tại sẽ không bị ghi đè.
`input_format=leaderboard_26` và `label_semantics` được lưu để truy nguồn.

## Chốt split theo timeline thật

- Season 2: tên file chỉ ra train 06–12/06/2013 và test 13–15/06/2013. Có thể
  khai báo validation_start bên trong khoảng train, sau khi kiểm tra timestamp
  toàn file và trước khi dùng test để đánh giá mô hình.
- Season 3: tài liệu gốc nói test được tạo từ training; metadata liệt kê train
  imp/clk 19–27/10 và leaderboard 21–28/10. Không gọi cặp file phát hành này là
  chronological split chỉ vì đã sắp xếp lại các dòng.
- Pipeline `prepare` hiện yêu cầu train < validation < test trên toàn timeline.
  Adapter season 3 không thay đổi điều kiện này. Dữ liệu chồng lấn vẫn bị từ chối.
- Không gộp train cả season 2 và 3 rồi test cả season 2 và 3 để chạy protocol
  chronological toàn cục: test tháng 6 sẽ nằm trước phần train tháng 10.

Với season 3, cần chọn lại các tập không chồng thời gian từ nguồn đã xác minh,
kiểm tra bid ID trùng trước khi bỏ chúng ở canonical output, rồi ghi rõ đây là
split mới do nghiên cứu khai báo. Chưa có bước tự động tái chia/deduplicate giữa
training và leaderboard; không ghép chúng rồi coi như độc lập.

Sao chép `configs/dataset_chronological.example.json`, điền input, output mới và
validation_start thật. Sau `prepare`, kiểm tra manifest (số dòng, positives,
timestamp và hashes) trước huấn luyện. Không suy class balance từ 50 dòng mẫu.
