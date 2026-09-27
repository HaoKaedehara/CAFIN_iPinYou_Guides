# Chạy trên máy Windows hiện tại

Người dùng đã chuyển từ Colab sang thiết bị hiện tại ngày 18/09/2026. Dữ liệu gốc
ở `DataSet Downloaded`; giữ nguyên ZIP để đối chiếu checksum. Cấu hình máy đã
kiểm tra: khoảng 16 GB RAM, GPU GTX 1650 Max-Q 4 GB, PyTorch 2.6.0+cu124 và CUDA
forward/backward hoạt động. Không cần cài thêm PyTorch hoặc upload lên Drive.

## Đường dẫn và lệnh PowerShell

Mở PowerShell trong workspace rồi thiết lập:

```powershell
$cafinPython = 'C:\Users\Chi Hao Le Nguyen\AppData\Local\Programs\Python\Python313\python.exe'
$env:PYTHONPATH = (Join-Path (Get-Location) 'src')
```

File cấu hình `*_local*.json` dùng đường dẫn tuyệt đối của máy này. Hai ZIP được
`scripts/setup_local_data.py` kiểm tra: SHA-256 iPinYou_x1 và MD5 raw season 2.
Script giải nén x1, kiểm tra MD5 từng CSV, quét schema/nhãn/số dòng toàn CSV rồi
tạo mẫu chẩn đoán và config. Script không tải mạng hoặc huấn luyện. Không cần
chạy lại setup sau khi đã có báo cáo thành công.

| Artifact | Đường dẫn |
|---|---|
| CSV x1 đầy đủ | `data/raw/ipinyou_x1/` |
| Kiểm tra nguồn và số dòng | `results/diagnostics/local_data_audit.json` |
| Mẫu đầu train 100.000 / test 5.000 dòng | `data/samples/x1_prefix100k/` |
| Preprocessing mẫu | `data/processed/x1_local_smoke/` |
| Run thử CAFIN, seed 11 | `runs/cafin_local_smoke_seed11/` |
| Preprocessing toàn bộ | `data/processed/x1_local_full/` |
| Cấu hình train toàn bộ | `configs/train_cafin_local.json` |

Trạng thái đã xác nhận trong phiên chuyển máy:

- Hai archive khớp checksum công bố; toàn bộ CSV x1 có schema/nhãn hợp lệ.
- Preprocessing full đã hoàn tất trong ~12,12 phút, output ~1,69 GB, hash tất cả
  artifacts đã kiểm tra lại. Train/valid/test lần lượt 13.855.732 / 1.539.526 /
  4.100.716 dòng. Không chạy lại lệnh prepare vào output này.
- Smoke CAFIN đã hoàn thành một epoch GPU (~10,83 giây) và xuất validation.
- Capacity full-vocabulary, batch 512 đã đạt; peak allocated ~273 MiB trong
  phép đo ngắn. Full training và final test chưa chạy.

## Kiểm tra bằng dữ liệu thật trước full training

```powershell
& $cafinPython -m cafin prepare --config configs/dataset_x1_local_smoke.json
& $cafinPython -m cafin train --config configs/train_cafin_local_smoke.json
& $cafinPython -m cafin evaluate --run runs/cafin_local_smoke_seed11 --split valid
```

Mẫu dùng 90.000 dòng train và 10.000 dòng validation từ 100.000 dòng đầu packaged
train. Chỉ chạy một epoch, batch 256, GPU. `sample_only=true` trong manifest/run;
mục đích là kiểm tra hoạt động và đo tài nguyên, không phải kết quả benchmark.
Phần mẫu packaged test được encode nhưng không đánh giá mô hình trên test.

Các lệnh `prepare` yêu cầu output trống, `train` yêu cầu run mới. Nếu artifacts
đã tồn tại, xem manifest/status trước khi chạy lại. Run có checkpoint nhưng chưa
hoàn thành dùng cùng config với `--resume`; không sửa code/config của run đó.

## Tiền xử lý và huấn luyện đầy đủ

```powershell
& $cafinPython -m cafin prepare --config configs/dataset_x1_local.json
& $cafinPython -m cafin train --config configs/train_cafin_local.json
```

Config full dùng 16 feature như thiết kế hiện tại, validation là 10% cuối packaged
train, batch 512, seed 11, tối đa 30 epoch và early stopping patience 3. Đây là
cấu hình khởi đầu để chạy một model/seed, chưa phải sweep hay cấu hình đã tune.
Chỉ gọi train full sau khi kiểm tra đủ RAM/VRAM/đĩa và manifest full đã hoàn chỉnh.
Nếu cần đổi batch size sau profiling, phải dùng run mới.

Baseline DeepFM cùng seed/split dùng cấu hình [train_deepfm_local.json](../configs/train_deepfm_local.json). Run này độc lập với CAFIN:

```powershell
& $cafinPython -m cafin train --config configs/train_deepfm_local.json
& $cafinPython -m cafin evaluate --run runs/deepfm_local_full_seed11 --split valid
```

Sau khi prepare full hoàn thành, có thể kiểm tra dung lượng GPU với đúng
vocabulary đầy đủ và batch 512:

```powershell
& $cafinPython scripts/profile_local_model.py
```

Script thực hiện 2 bước warmup và 10 bước Adam trên một batch train lặp lại,
không tạo checkpoint hoặc điểm đánh giá mô hình. Báo cáo nằm ở
`results/diagnostics/local_gpu_profile.json`. Thời gian này không gồm đọc toàn
dataset, chuyển batch CPU–GPU, validation hoặc ghi checkpoint, nên không suy ra
thời gian epoch chính xác từ nó.

Bước tiếp theo trên workspace hiện tại chỉ cần lệnh train full ở trên. Các lệnh
setup/prepare/smoke được giữ trong hướng dẫn để tái dựng, không cần lặp lại.

Raw season 2 mới được xác minh archive; nhập impression/click và leaderboard là
công việc riêng theo [RAW_PROTOCOL.md](RAW_PROTOCOL.md). Tránh giải nén toàn bộ
bid logs khi chỉ cần impression/click/leaderboard. Raw season 3 chưa có trên máy.

Trạng thái thực thi chính xác và các lần đo nằm trong SESSION_HANDOFF.md và các
artifact tương ứng; việc có config không có nghĩa lượt huấn luyện đã chạy.
