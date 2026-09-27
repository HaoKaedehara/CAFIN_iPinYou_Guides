# Nguồn dữ liệu đã kiểm tra ngày 18/09/2026

Đã đọc metadata và phần cuối ZIP bằng HTTP Range để kiểm tra danh sách file.
Lần tiếp theo đọc thêm tối đa 256 KiB đầu của hai member nén trên Figshare,
kiểm tra 50 dòng impression và 50 dòng leaderboard; không giữ các bản ghi thật.
Đã lấy danh sách file raw trên Kaggle. Bằng chứng cấu trúc và hash mẫu nằm ở
[evidence/raw_source_probe.json](evidence/raw_source_probe.json).
Phần kiểm tra mạng này chưa tải đầy đủ ZIP. Sau đó người dùng đã cung cấp hai ZIP
tại máy Windows; kết quả xác minh local ở phần tiếp theo.

## Dữ liệu đã có tại máy Windows

Người dùng chuyển sang chạy local ngày 18/09/2026. Báo cáo thực đo:
`results/diagnostics/local_data_audit.json`.

- `DataSet Downloaded/iPinYou_x1.zip`: SHA-256 toàn file khớp LFS công bố.
  Hai CSV đã giải nén và khớp MD5 công bố; quét toàn bộ xác nhận schema 17 cột,
  nhãn 0/1, train 15.395.258 dòng / 11.557 positive và test 4.100.716 dòng /
  3.008 positive. Đây là kiểm tra dataset, không phải điểm đánh giá mô hình.
- `DataSet Downloaded/ipinyou.contest.datasetseason2.zip`: MD5 toàn archive khớp
  Figshare. Chưa giải nén/import toàn bộ raw logs hoặc xác minh riêng files.md5.
- Chưa có raw season 3 tại máy này.
- Đã chạy CAFIN một epoch trên mẫu x1 thật bằng GPU; run được đánh dấu sample_only,
  không dùng làm full benchmark. Xem [LOCAL_RUNBOOK.md](LOCAL_RUNBOOK.md).

## iPinYou_x1 — nguồn dùng cho notebook CTR hiện tại

- [Trang dataset RecZoo](https://huggingface.co/datasets/reczoo/iPinYou_x1).
- [Tải iPinYou_x1.zip](https://huggingface.co/datasets/reczoo/iPinYou_x1/resolve/main/iPinYou_x1.zip?download=true).
- [Metadata file từ Hugging Face API](https://huggingface.co/api/datasets/reczoo/iPinYou_x1/tree/main).
- [Checksum CSV do nhà phát hành công bố](https://huggingface.co/datasets/reczoo/iPinYou_x1/raw/main/README.md).

| File | Dung lượng chính xác | Checksum công bố |
|---|---:|---|
| iPinYou_x1.zip | 251.577.945 byte | SHA-256 `25fb5e518cc1a3cc64c6ec57b0f5dcd6e0ce93d202d88a8b4b89c1fbdd4a77e1` |
| train.csv trong ZIP | 1.613.212.352 byte | MD5 `9dd8979d265ab1ed7662ffd49fd73247` |
| test.csv trong ZIP | 429.489.089 byte | MD5 `a94374868687794ff8c0c4d0b124a400` |

SHA-256 ZIP là LFS object digest trong API. Kích thước CSV lấy từ ZIP central
directory. ZIP khoảng 252 MB, hai CSV khoảng 2,04 GB (đơn vị thập phân). Giữ cả
ZIP và CSV cần khoảng 2,30 GB trước khi tính output preprocessing và checkpoints.
Không dùng dung lượng ZIP để ước tính tổng RAM/đĩa cần cho thí nghiệm.

Hugging Face viewer hiển thị 19.495.974 dòng dưới một split `train`; đó không phải
xác nhận số dòng của riêng packaged train.csv. Pipeline tải hai CSV gốc trong ZIP
và giữ packaged test, không lấy split tự động của viewer thay thế. Row count và
class counts từng CSV đã được xác minh trong lần quét local nêu ở trên.

Sau cell setup của notebook 01, dùng cell download có sẵn. Lệnh tương đương từ
thư mục project trên máy thí nghiệm:

```bash
python -m cafin download-x1 --output data/raw/ipinyou_x1
python -m cafin prepare --config configs/dataset_x1.json
```

Từ package 0.2.1, downloader lấy ZIP, kiểm tra SHA-256, giải nén từng CSV theo
luồng, kiểm tra MD5 rồi ghi SHA-256 từng CSV vào download_manifest.json. Có thể
chạy lại để dùng file hợp lệ sẵn có; nếu download bị ngắt giữa ZIP, lần sau tải
lại ZIP từ đầu. ZIP hợp lệ đã tải xong có thể dùng lại để khôi phục CSV còn thiếu.

Dataset này phù hợp pipeline CTR standardized hiện tại. Không tự suy ra timestamp,
season hoặc historical payingprice từ các category đã mã hóa để chạy raw RTB.

## Raw season 2 — bản lưu trên Figshare

- [Trang bản lưu của Zilong Jiang](https://figshare.com/articles/dataset/ipinyou_contest_dataset_season2/5732328).
- DOI: [10.6084/m9.figshare.5732328.v1](https://doi.org/10.6084/m9.figshare.5732328.v1).
- [Metadata API](https://api.figshare.com/v2/articles/5732328).
- [Tải ipinyou.contest.dataset-season2.zip](https://ndownloader.figshare.com/files/10082688).
- Kích thước: **3.610.249.162 byte**, khoảng **3,61 GB** (3,36 GiB).
- MD5 do Figshare báo cáo: `9ee18bf8c4dd19d1c41e6e77088367f9`.

Danh sách ZIP xác nhận thư mục gốc `ipinyou.contest.dataset-season2/`, gồm:

- README, README.old, files.md5 và known.data.bugs.txt.
- training2nd: bid/imp/clk/conv cho các ngày 20130606 đến 20130612.
- testing2nd: leaderboard.test.data.20130613_15.txt.bz2.
- Bảng region/city và user profile tags.

Đây là bản lưu bên thứ ba, chưa đối chiếu toàn bộ với archive gốc. Đã xác nhận
endpoint trả HTTP 206, đọc README/known.data.bugs và kiểm tra mẫu đầu: 50 dòng
impression có 24 cột; 50 dòng leaderboard có 26 cột, logtype=1. Mẫu nhỏ chưa đủ
kết luận toàn file hợp lệ; cần kiểm tra files.md5 sau khi tải.

Lệnh tải trên Colab/máy thí nghiệm (chạy trong terminal hoặc cell `%%bash`):

```bash
mkdir -p /content/cafin_data/raw/season2_archive
cd /content/cafin_data/raw/season2_archive
curl --fail --location --retry 3 --output ipinyou.contest.dataset-season2.zip.part https://ndownloader.figshare.com/files/10082688
echo '9ee18bf8c4dd19d1c41e6e77088367f9  ipinyou.contest.dataset-season2.zip.part' | md5sum --check --status && mv ipinyou.contest.dataset-season2.zip.part ipinyou.contest.dataset-season2.zip
```

Chỉ import impression/click logs tương ứng sau khi kiểm tra schema. `import-raw`
nhận 24 cột; từ 0.2.2, `import-leaderboard` nhận 26 cột theo tài liệu gốc và tạo
nhãn nhị phân `RelateClicks > 0`. Xem [RAW_PROTOCOL.md](RAW_PROTOCOL.md).

## Raw season 3 — tìm thấy trong bộ Kaggle lastsummer/ipinyou

- [Trang dataset](https://www.kaggle.com/datasets/lastsummer/ipinyou).
- [API danh sách file](https://www.kaggle.com/api/v1/datasets/list/lastsummer/ipinyou?pageSize=100).
- Đã đọc hết phân trang: 109 file trong dataset, gồm 34 file thuộc training3rd/
  testing3rd/ với tổng 985.629.556 byte theo metadata.
- Chỉ chọn imp/clk và leaderboard season 3: 19 file, 332.768.955 byte (khoảng
  333 MB, chưa tính overhead ZIP nếu máy chủ bọc file khi tải).
- training3rd: imp/clk ngày 20131019–20131027; bid ngày 20131019–20131028.
- testing3rd: `leaderboard.test.data.20131021_28.txt.bz2`, 112.633.775 byte.
- Có README, files.md5 và known.data.bugs.txt ở thư mục gốc dataset.

Đây là xác nhận metadata, chưa xác nhận tải đầy đủ các file Kaggle hoặc checksum.
Không tự coi file không có trong danh sách là ngày không phát sinh click. Mirror
khác `pleaseholdme/ipinyou` có 60 file đã xử lý theo campaign, không thay thế trực
tiếp bộ raw. Danh sách cả hai mirror được lưu trong evidence JSON.

Có thể tải từng file trên Colab bằng Kaggle CLI sau khi thiết lập quyền truy cập
Kaggle tại runtime. Ví dụ chỉ lấy một impression file, không phải toàn season:

```bash
kaggle datasets download -d lastsummer/ipinyou -f ipinyou.contest.dataset/training3rd/imp.20131019.txt.bz2 -p /content/cafin_data/raw/kaggle
```

Kaggle có thể trả ZIP bọc file được chọn; kiểm tra nội dung trước khi giải nén.
Lấy đủ imp/clk của các ngày đã chọn và README/files.md5 để xác minh trước import.
Hiện ưu tiên dùng ZIP người dùng đã tải; chỉ tải bổ sung file còn thiếu khi cần.

## Nguồn raw còn cần xác minh

- [Repo xử lý của Weinan Zhang](https://github.com/wnzhang/make-ipinyou-data)
  liên kết archive UCL; lần truy cập qua web tool này bị timeout. Không kết luận
  archive đã biến mất, cũng không coi là đã xác nhận tải được.
- [Mô tả gốc từ iPinYou](https://contest.ipinyou.com/ipinyou-dataset.pdf) là nguồn
  tham khảo schema và protocol; chưa thay thế kiểm tra log thực tế.
- Raw season 3 đã xuất hiện trong metadata Kaggle; tải file thực và đối chiếu
  checksum đầy đủ là bước tiếp theo trên máy thí nghiệm.

Ưu tiên chạy CTR bằng iPinYou_x1 đã khớp định dạng tải. Với raw, phải kiểm tra
timeline: tài liệu gốc mục 2.4 cho biết test season 3 lấy từ giai đoạn training,
không bảo đảm split chronological. Không bỏ kiểm tra overlap để ép dữ liệu chạy.
