# Nhận xét phản biện nội bộ — bản thảo LNCS/LNNS

## Kết luận đề xuất

**Major revision trước khi nộp.** Bài đã đủ nền tảng thực nghiệm để gửi hội nghị thuộc LNNS sau khi hoàn thiện hình thức và literature audit, nhưng chưa nên tuyên bố một kiến trúc mới vượt trội hoặc một SOTA mới.

## Điểm mạnh

1. Pipeline đã hoàn tất 75 run P1 và 30 run P2/replay; mọi run cuối dùng năm seed và có checkpoint đóng băng.
2. Dữ liệu, source, configuration, test-access receipt và replay policy đều có hash; khả năng tái lập tốt hơn nhiều bài chỉ báo cáo một con số.
3. Thiết kế P1/P2 tách được campaign shift và chronology; bài báo không trộn validation với test.
4. Kết quả trung thực: CAFIN cạnh tranh nhưng không thắng mọi baseline. Đây là cách diễn giải phù hợp với kết quả hiện tại.
5. Có cả CTR metrics và budgeted replay, đúng với bản chất RTB của iPinYou.

## Điểm yếu cần sửa

1. **Novelty còn vừa phải.** Cross network, self-attention, cross-attention và gating đều là thành phần đã biết. Đóng góp nên định vị là protocol-controlled empirical study và thiết kế tích hợp có kiểm chứng, không phải phát minh attention mới.
2. **So sánh literature chưa cùng protocol.** Bảng mới được thêm là bảng comparability; cần giữ nguyên cảnh báo này và không đặt số liệu khác split cạnh số liệu P1/P2 như một bảng xếp hạng.
3. **Chưa có Season 3.** Cần ghi đây là giới hạn dữ liệu/phạm vi, không gọi là đánh giá xuyên mùa.
4. **Chưa có khoảng tin cậy hoặc kiểm định cặp seed.** Có thể bổ sung bootstrap trên dự đoán test hoặc permutation/paired seed analysis; ít nhất phải giải thích vì sao seed standard deviation chỉ là độ biến thiên thực nghiệm.
5. **RTB replay mới là historical replay.** Không được gọi là online lift, causal gain hoặc live auction result.
6. **Baseline là implementation độc lập.** Nên thêm phụ lục nêu rõ từng model, số tham số, embedding, optimizer, epoch selection và những khác biệt với code gốc.
7. **Bản nộp cần hoàn thiện metadata.** Điền tác giả, affiliation, funding, data/code availability và biên dịch bằng bộ `llncs` chính thức để kiểm tra overflow bảng, tài liệu tham khảo và layout.

## Research gap có thể bảo vệ

Khoảng trống có thể bảo vệ là thiếu một đánh giá có kiểm soát protocol trên iPinYou nối ba khía cạnh thường bị tách rời: campaign/distribution shift, chất lượng xác suất CTR và utility của bidder dưới ngân sách. Các bài trước thường tập trung vào CTR ranking hoặc bid optimisation; các số liệu khó đối chiếu do khác campaign, split, preprocessing và price semantics. Bài này lấp khoảng trống bằng provenance, năm seed, frozen test, chronological singleton protocol và replay artifacts có hash.

## Có gì nổi trội hiện tại

Điểm nổi trội không phải CAFIN thắng tuyệt đối. Điểm nổi trội là:

- cùng một codebase thực hiện baseline, ablation, frozen test và replay;
- báo cáo đồng thời LogLoss, AUC, AP, Brier và chỉ số ngân sách;
- cho thấy thứ hạng model thay đổi giữa P1 và P2;
- công khai giới hạn và không biến số liệu khác protocol thành SOTA.

## Đánh giá khả năng LNNS/Q4

**Có khả năng nộp sau major revision**, đặc biệt nếu venue nhận bài về machine learning ứng dụng, advertising systems hoặc data-driven networks. Không thể bảo đảm acceptance hoặc quartile vì quyết định phụ thuộc call for papers, reviewer và phiên bản chỉ mục tại thời điểm xuất bản. LNNS là book series/proceedings; Q4 là phân loại Scopus/SJR theo ngành và có thể thay đổi theo năm/ngành, không phải nhãn cố định cho mọi volume.

Mức sẵn sàng hiện tại: **khoảng 70--75% cho submission kỹ thuật**, **chưa đủ để gọi là camera-ready**. Ba việc quan trọng nhất trước khi nộp là:

1. hoàn thiện literature audit và phụ lục baseline;
2. bổ sung uncertainty analysis hoặc giải thích thống kê chặt chẽ;
3. compile PDF LNCS và rà lỗi hình thức, tài liệu tham khảo, metadata.

## Quyết định phản biện

**Major revision, then resubmit.** Không khuyến nghị reject: artifacts và kết quả thật đã đủ giá trị thực nghiệm. Không khuyến nghị accept hiện tại: novelty framing, external literature comparison và statistical uncertainty vẫn cần chỉnh.
