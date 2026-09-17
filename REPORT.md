# Báo cáo Day 5 — điền trực tiếp trong fork của bạn

- Mã học viên theo lớp: _(cần bổ sung)_
- Ngày / CVAT local: 17/09/2026 · CVAT v2.74.1 (local trên laptop, sau đó CVAT cùng phiên bản chạy trên máy GPU thuê, truy cập qua SSH tunnel)
- Công cụ đã dùng: Polygon, Brush (Polygon +/−, Eraser); AI Tools → Interactor **Segment Anything (SAM ViT-H)** chạy GPU cho một phần object, mọi mask gợi ý đều được kiểm và sửa tay.

## 1. Bài đã nộp

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | `submissions/easy_semantic.zip` (Segmentation mask 1.1) | 3 / 3 | 20 |
| medium_instance | `submissions/medium_instance.zip` (COCO 1.0) | 3 / 3 | 32 |
| hard_panoptic | `submissions/hard_panoptic.zip` (COCO 1.0) | 2 / 2 | 30 |
| cp1_holes | `submissions/cp1_holes.zip` (COCO 1.0) | 1 / 1 | 3 |
| cp2_slice | `submissions/cp2_slice.zip` (COCO 1.0) | 1 / 1 | 3 |
| cp5_occlusion | `submissions/cp5_occlusion.zip` (COCO 1.0) | 1 / 1 | 3 |
| cp3_thin | `submissions/cp3_thin.zip` (Segmentation mask 1.1) | 1 / 1 | 3 |
| cp4_curb | `submissions/cp4_curb.zip` (Segmentation mask 1.1) | 1 / 1 | 3 |
| cp6_coverage | `submissions/cp6_coverage.zip` (Segmentation mask 1.1) | 1 / 1 | 3 |
| **Tổng tối đa** | | | **100** |

Cả 9 ZIP qua `python3 scripts/inspect_submissions.py --dir submissions` với trạng thái `OK`. Máy GPU thuê lần đầu đã bị xóa trước khi export nên phần vẽ trên đó phải làm lại trên máy mới; các ZIP trên là bản export cuối.

## 2. Một quyết định trước khi dùng gợi ý

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: `000000181542.jpg`, người lái xe máy đội mũ bảo hiểm ở giữa ảnh, bị người phụ nữ mặc áo dài đứng phía trước che một phần chân và tay.
- Class và quy tắc tôi dùng để chọn biên: `person`, vẽ bằng Polygon tay. Chỉ vẽ phần nhìn thấy: mép phải phần chân/tay dừng ở mép áo dài của người đứng trước, không vẽ xuyên qua người che. Người lái và người áo dài là hai instance riêng.
- Nếu dùng gợi ý sau đó: dùng SAM cho các object khác trong ảnh. Rủi ro tôi kiểm khi giữ gợi ý: mask ăn sang người đứng trước hoặc chiếc xe máy chồng phía sau; chiếc xe máy của người lái phải là **một** `motorcycle` gồm hai mảng nhìn thấy hai bên người áo dài, không tách thành hai object.

## 3. Một lỗi tôi tìm thấy và sửa

- Task/ảnh/vùng: `medium_instance`, `000000181542.jpg`, polygon `person` của người lái xe máy.
- Lỗi thuộc loại: biên (tràn sang instance khác).
- Bằng chứng tôi nhìn thấy: mép phải polygon phần chân và tay phủ lên cánh tay, tay áo và vạt áo dài của người đứng phía trước.
- Quy tắc và hành động sửa: vẽ theo phần nhìn thấy, người đứng trước giữ pixel của mình; kéo các điểm mép phải về bám mép áo dài.
- Sau sửa đã Save và export lại chưa? Đã Save và export. Kiểm lại ZIP cuối: trong ảnh này không còn cặp mask nào chồng lấn quá 100 pixel.

Kết quả tự chạy `scoring/score.py` với reference ba tier được phát (ZIP Medium và Hard đã export trước giờ phát đáp án; chưa sửa bài sau khi xem kết quả):

- easy_semantic: mIoU 0.814 (sidewalk 0.670, vegetation 0.601 thấp nhất).
- medium_instance: mean matched IoU 0.820, R@0.5 0.69, TP 49 / FP 9 / FN 22, nộp 58 so với 71 object. Lỗi còn lại chủ yếu là **bỏ sót người nhỏ ở xa**, không phải biên.
- hard_panoptic: PQ 0.352 (SQ 0.586, RQ 0.435). Bỏ sót `traffic light`, nhiều `person` nhỏ; `sidewalk` chưa khớp.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| `easy_semantic` · `7ee6d192-89e2408b.jpg` · đồi cỏ khô và bụi cây hai bên đường cao tốc | (a) `vegetation`; (b) địa hình đất/cỏ khô không thuộc 5 class của task | Task chỉ có road/sidewalk/building/vegetation/sky; phần lớn đồi là cỏ khô màu nâu, chỉ các mảng cây sẫm là thảm thực vật rõ | Để trống vùng đồi (khoảng 22% ảnh chưa gán). Hỏi coach: đồi cỏ khô có tính là `vegetation` không? |
| `medium_instance` · `000000181542.jpg` · xe bán tải màu bạc bên phải phía sau | (a) `car`; (b) `truck` | Có thùng hàng hở phía sau như truck, nhưng kích thước và cabin giống xe con | Gán `car`. Hỏi coach: pickup nên theo quy ước COCO là `truck`? |
| `medium_instance` · `000000181542.jpg` · xe máy của người lái bị người áo dài che ở giữa | (a) hai object (đuôi xe và bánh trước); (b) một object có hai mảng rời | Phiếu quy tắc: vật bị che thành vùng nhìn thấy rời nhau vẫn là một instance | Một `motorcycle`, vẽ bằng Brush/SAM gồm cả hai mảng, bỏ phần bị người che |
