# Báo cáo Day 5 — điền trực tiếp trong fork của bạn

**Cách dùng:** Thay mọi dấu `…` bằng bài làm thật của bạn trước khi nộp link fork trên VLearn. Giữ nguyên bốn mục và bảng để coach đọc nhanh. Viết ngắn, cụ thể theo ảnh/vùng; không cần thuật ngữ chuyên sâu. Ví dụ trong [hướng dẫn mẫu](reports/REPORT_TEMPLATE.md) chỉ giúp hiểu cách điền, không phải câu trả lời để chép lại.

- Mã học viên theo lớp: 2A202602098
- Ngày / CVAT local: 17/09/2026 · CVAT v2.74.1 chạy local (`localhost:8080`), sau đó CVAT v2.74.1 trên máy GPU thuê truy cập qua SSH tunnel
- Công cụ đã dùng: Polygon, Brush (Polygon +/−, Eraser), gợi ý tự động AI Tools → Interactor **Segment Anything** (SAM ViT-H, tự triển khai qua Nuclio trên GPU); mọi mask gợi ý đều tự kiểm class, số object và biên

Mã học viên là mã lớp cấp; không cần ghi họ tên trong report nếu kênh VLearn đã nhận diện bạn. Chỉ ghi công cụ thật sự đã dùng; không có SAM vẫn làm bài bình thường.

## 1. Bài đã nộp

Ghi tên ZIP đúng như file trong `submissions/` và số ảnh đã vẽ, Save. Chưa làm hoặc export lỗi thì ghi `chưa có`, không tạo ZIP rỗng. Cột điểm là điểm tối đa của task, **không phải điểm tự chấm**.

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | `easy_semantic.zip` | 3 / 3 | 20 |
| medium_instance | `medium_instance.zip` | 3 / 3 | 32 |
| hard_panoptic | `hard_panoptic.zip` | 2 / 2 | 30 |
| cp1_holes | `cp1_holes.zip` | 1 / 1 | 3 |
| cp2_slice | `cp2_slice.zip` | 1 / 1 | 3 |
| cp5_occlusion | `cp5_occlusion.zip` | 1 / 1 | 3 |
| cp3_thin | `cp3_thin.zip` | 1 / 1 | 3 |
| cp4_curb | `cp4_curb.zip` | 1 / 1 | 3 |
| cp6_coverage | `cp6_coverage.zip` | 1 / 1 | 3 |
| **Tổng tối đa** | | | **100** |

Nếu export lỗi, ghi task, dữ liệu đã Save đến đâu và lỗi đã báo coach.

Không có export lỗi. Cả 9 ZIP chạy `python3 scripts/inspect_submissions.py --dir submissions` đều `OK` (Semantic: Segmentation mask 1.1; Instance/Panoptic: COCO 1.0).

## 2. Một quyết định trước khi dùng gợi ý

Chọn object đầu tiên bạn tự vẽ ở `medium_instance`, trước khi xem bất kỳ đề xuất tự động nào cho object đó. Ghi ảnh/vị trí đủ để tìm lại; “quy tắc biên” là lý do bạn chọn hoặc dừng mask ở ranh đó.

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: `000000181542.jpg`, người lái xe máy đội mũ bảo hiểm ở giữa ảnh, bị người phụ nữ mặc áo dài đứng phía trước che một phần chân và tay; vẽ tay bằng Polygon.
- Class và quy tắc tôi dùng để chọn biên: `person`. Chỉ vẽ phần nhìn thấy: mép phải phần chân/tay dừng ở mép áo dài của người đứng trước, không đoán phần bị che. Người lái và người áo dài là hai instance riêng.
- Nếu dùng gợi ý sau đó: vùng gợi ý sai/đúng, hành động sửa/giữ và lý do: có dùng SAM cho các object khác. Điểm dễ sai là mask ăn sang người đứng trước hoặc chiếc xe máy chồng phía sau; tôi chỉ giữ mask khi không tràn sang vật khác, và chiếc xe máy bị người áo dài che giữ là **một** `motorcycle` gồm hai mảng nhìn thấy, không tách thành hai object.
- Nếu không dùng gợi ý: không áp dụng (có dùng gợi ý như trên).

## 3. Một lỗi tôi tìm thấy và sửa

Chọn một lỗi **có thật** trong bài. Nếu công cụ lỗi khiến bạn chưa sửa được, ghi rõ đã thử gì và cần coach hỗ trợ gì; không ghi “đã sửa” khi chưa sửa.

- Task/ảnh/vùng: `medium_instance`, `000000181542.jpg`, nhóm người đi bộ nhỏ ở xa trên vỉa hè, dưới biển hiệu CHANEL (khoảng giữa phía trên ảnh).
- Lỗi thuộc loại: sai lớp / thiếu-thừa vật / gộp-tách / biên / phủ vùng / khác: gộp-tách (nhiều người gộp thành một object).
- Bằng chứng tôi nhìn thấy: trong ZIP đã nộp, cả nhóm chỉ là **một** mask `person` rộng khoảng 156 × 37 px, gồm 4 mảng rời nhau, mỗi mảng là một người khác nhau; danh sách Objects đếm ít người hơn số người nhìn thấy.
- Quy tắc và hành động sửa: mỗi người là một instance riêng, kể cả khi nhỏ và đứng sát nhau; cần xóa mask gộp và vẽ từng người thành một `person`, người quá mờ không phân biệt được thì ghi lại thay vì gộp.
- Sau sửa đã Save và export lại chưa? **Chưa sửa.** Phát hiện khi rà lại ZIP sau lúc nộp; CVAT trên máy GPU dùng để vẽ Medium đã bị xóa, muốn sửa phải import lại ZIP vào CVAT, tách từng người, Save và export lại.

Nếu bạn **đã xem Summary tự đánh giá trên GitHub Actions hoặc tự chạy script**, ghi ngắn một kết quả liên quan lỗi vừa sửa (ví dụ task, metric trước/sau nếu có): chạy `scoring/scorecard.py --group tiers` với gói reference ba tier được phát, **40.4 / 82**. `medium_instance`: mean matched IoU 0.820, R@0.5 0.69 (TP 49 / FP 9 / FN 22) , nộp 58 object so với 71 — biên các mask ghép được khá khớp, mất điểm chủ yếu do bỏ sót hoặc gộp người nhỏ ở xa như lỗi trên. Lỗi chưa sửa nên chưa có số trước/sau; các ZIP chưa sửa lại sau khi xem kết quả. Scorecard ba tier tối đa **82**, không phải điểm cuối trên 100. Không tự ghi PASS/top 3/bonus; người phụ trách xác nhận theo tiêu chí lớp. Không đưa file ground truth vào fork.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

Mỗi ca là một **vùng cụ thể** khiến bạn phải cân nhắc hai cách hiểu. Ghi dấu hiệu nhìn thấy hoặc quy tắc đã dùng, rồi nêu quyết định hoặc câu hỏi cho coach. Không cần ba lỗi; ca đã quyết định được cũng hợp lệ.

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| `easy_semantic` · `7ee6d192-89e2408b.jpg` · đồi cỏ khô và bụi cây hai bên đường cao tốc | (a) `vegetation`; (b) địa hình cỏ khô/đất không thuộc 5 class của task | Task chỉ có road/sidewalk/building/vegetation/sky; phần lớn đồi là cỏ khô màu nâu, chỉ các mảng cây sẫm là thảm thực vật rõ | Để trống vùng đồi. Hỏi coach: đồi cỏ khô có tính là `vegetation` không? |
| `medium_instance` · `000000181542.jpg` · xe bán tải màu bạc bên phải phía sau | (a) `car`; (b) `truck` | Có thùng hàng hở phía sau như truck, nhưng cabin và kích thước gần xe con | Gán `car`. Hỏi coach: xe bán tải nên theo quy ước COCO là `truck`? |
| `medium_instance` · `000000181542.jpg` · xe máy của người lái bị người áo dài che ở giữa | (a) hai object (đuôi xe và bánh trước); (b) một object có hai mảng rời | Phiếu quy tắc: vật bị che thành các vùng nhìn thấy rời nhau vẫn là một instance | Một `motorcycle` gồm cả hai mảng, bỏ phần bị người che |
