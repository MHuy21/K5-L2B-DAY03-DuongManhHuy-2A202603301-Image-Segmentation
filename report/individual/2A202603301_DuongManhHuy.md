# Báo cáo thực hành Day 3 — điền trực tiếp trong fork của bạn

**Cách dùng:** Thay mọi dấu `…` bằng bài làm thật của bạn trước khi nộp link fork trên [LMS](https://aithucchien.vinuni.edu.vn). Giữ nguyên bốn mục và bảng để coach đọc nhanh. Viết ngắn, cụ thể theo ảnh/vùng; không cần thuật ngữ chuyên sâu. Ví dụ trong [hướng dẫn mẫu](../../reports/REPORT_TEMPLATE.md) chỉ giúp hiểu cách điền, không phải câu trả lời để chép lại.

- Mã học viên theo lớp: 2A202603301
- Ngày / CVAT local: 11–12/11/2026 / CVAT local 2.74.1
- Công cụ đã dùng:
  1. **CVAT local:** tạo task, import annotation (COCO 1.0), sửa mask bằng Brush / Eraser / Polygon, Save và export.
  2. **YOLO11x-seg** tìm vật và **SAM 2.1-B** cắt mask theo khung, cho person, bicycle, car, motorcycle, bus, truck, traffic light.
  3. **SegFormer-B5 (Cityscapes)** cho road, sidewalk, building, vegetation, sky, pole, traffic sign.

Mã học viên là mã lớp cấp; không cần ghi họ tên trong report nếu kênh [LMS](https://aithucchien.vinuni.edu.vn) đã nhận diện bạn. Chỉ ghi công cụ thật sự đã dùng; không có SAM vẫn làm bài bình thường.

## 1. Bài đã nộp

Ghi tên ZIP đúng như file trong `submissions/` và số ảnh đã vẽ, Save. Chưa làm hoặc export lỗi thì ghi `chưa có`, không tạo ZIP rỗng. Cột điểm là điểm tối đa của task, **không phải điểm tự chấm**.

| Task            | File ZIP đúng tên   | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --------------- | ------------------- | -----------------: | ---------------------------: |
| easy_semantic   | easy_semantic.zip   |              3 / 3 |                           20 |
| medium_instance | medium_instance.zip |              3 / 3 |                           32 |
| hard_panoptic   | hard_panoptic.zip   |              2 / 2 |                           30 |
| cp1_holes       | cp1_holes.zip       |              1 / 1 |                            3 |
| cp2_slice       | cp2_slice.zip       |              1 / 1 |                            3 |
| cp5_occlusion   | cp5_occlusion.zip   |              1 / 1 |                            3 |
| cp3_thin        | cp3_thin.zip        |              1 / 1 |                            3 |
| cp4_curb        | cp4_curb.zip        |              1 / 1 |                            3 |
| cp6_coverage    | cp6_coverage.zip    |              1 / 1 |                            3 |
| **Tổng tối đa** |                     |                    |                      **100** |

Không có export lỗi; cả 9 ZIP đều `OK` khi chạy `scripts/inspect_submissions.py`.

## 2. Một quyết định trước khi dùng gợi ý

Chọn object đầu tiên bạn tự vẽ ở `medium_instance`, trước khi xem bất kỳ đề xuất tự động nào cho object đó. Ghi ảnh/vị trí đủ để tìm lại; “quy tắc biên” là lý do bạn chọn hoặc dừng mask ở ranh đó.

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: **Không có.** Ở `medium_instance`, tôi import mask gợi ý ngay sau khi tạo task.
- Class và quy tắc tôi dùng để chọn biên: Khi kiểm từng object, tôi áp dụng: chỉ giữ **phần nhìn thấy**; **mỗi vật một object** (người lái và xe máy là hai object riêng); vật bị che vẫn là **một** object; vật không có trong 6 class (ví dụ xe đẩy em bé, ván trượt) thì **để trống**.
- Nếu dùng gợi ý sau đó: vùng gợi ý sai/đúng, hành động sửa/giữ và lý do: Ảnh `000000181542.jpg`, góc trên trái, trong cửa kính xe buýt: gợi ý tạo một mask `person` rất nhỏ. Nhìn kỹ, tôi không xác định được đó là một người nhìn thấy rõ, nên tôi **xóa** để tránh object thừa.

## 3. Một lỗi tôi tìm thấy và sửa

Chọn một lỗi **có thật** trong bài. Nếu công cụ lỗi khiến bạn chưa sửa được, ghi rõ đã thử gì và cần coach hỗ trợ gì; không ghi “đã sửa” khi chưa sửa.

- Task/ảnh/vùng: `cp1_holes`, ảnh `000000144300.jpg`, mask chiếc mô tô Honda ở giữa ảnh.
- Lỗi thuộc loại: sai lớp / thiếu-thừa vật / gộp-tách / biên / phủ vùng / khác: **biên (mask bị khoét lỗ)**, kèm **thừa vật** (mask trùng).
- Bằng chứng tôi nhìn thấy: Mask gợi ý của chiếc mô tô bị lốm đốm, vỡ thành nhiều mảnh rời ở khung, bình xăng, yên và gắp sau. Trên xe cắm trại bên trái có 2 mask (`truck` và `bus`) trùng nhau 100%.
- Quy tắc và hành động sửa: Theo quy tắc của trạm, kính, khe và chi tiết của xe nằm **trong** mask, không khoét. Tôi dùng Brush tô kín thân xe, dùng Eraser bỏ chân dựng màu đỏ (không thuộc xe), và xóa mask trùng trên xe cắm trại, chỉ giữ một object.
- Sau sửa đã Save và export lại chưa? Đã Save và export lại (COCO 1.0) thành `submissions/cp1_holes.zip`, kiểm lại được `OK`.

Các lỗi khác tôi cũng đã sửa: `cp5_occlusion`, hai taxi vàng góc phải dưới có mask chồng nhau 51% → bỏ phần chồng khỏi taxi phía sau; `easy_semantic` ảnh `7ee6d192-89e2408b.jpg`, rào chắn bê tông bên trái bị gán `road` → xóa (khoảng 28 nghìn px); `cp4_curb`, dải bê tông sáng dưới dải cỏ bị gán `road` → đổi thành `sidewalk` (khoảng 44 nghìn px).

Nếu bạn **đã xem Summary tự đánh giá trên GitHub Actions hoặc tự chạy script**, ghi ngắn một kết quả liên quan lỗi vừa sửa (ví dụ task, metric trước/sau nếu có): chưa có điểm (checkpoint không tự chấm được khi chưa có reference). Scorecard ba tier tối đa **82**, không phải điểm cuối trên 100. Không tự ghi PASS/top 3/bonus; người phụ trách xác nhận theo tiêu chí lớp. Không đưa file ground truth vào fork.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

Mỗi ca là một **vùng cụ thể** khiến bạn phải cân nhắc hai cách hiểu. Ghi dấu hiệu nhìn thấy hoặc quy tắc đã dùng, rồi nêu quyết định hoặc câu hỏi cho coach. Không cần ba lỗi; ca đã quyết định được cũng hợp lệ.

| Ảnh/vị trí                                                                          | Hai cách hiểu có thể                                                          | Quy tắc/chứng cứ                                                                                | Quyết định hoặc câu hỏi cho coach                                                                                                                                                          |
| ----------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1. `cp1_holes` `000000144300.jpg`, xe cắm trại bên trái                             | (a) rơ-moóc kéo theo, không thuộc 6 class → để trống; (b) phương tiện → `car` | Không thấy động cơ hay cabin; 6 class không có rơ-moóc                                          | Giữ `car` vì vật nhìn thấy rõ, có thân kín, cửa sổ, cửa ra vào và bánh xe giống xe con; trong 6 class, `car` là class gần nhất, và tôi ưu tiên không bỏ sót một phương tiện nhìn thấy rõ.  |
| 2. `medium_instance` `000000181542.jpg`, xe bán tải xanh giữa ảnh                   | (a) `car`; (b) `truck`                                                        | Có thùng hàng hở phía sau; theo quy ước COCO, bán tải thường là `truck`                         | Giữ `car` vì phần đầu và cabin giống xe con, thùng sau ngắn.                                                                                                                               |
| 3. `cp3_thin` `839f7736-abe28069.jpg`, hộp đèn giao thông trên cột bên trái và giữa | (a) tính cả hộp đèn vào `pole`; (b) chỉ thân cột là `pole`, hộp đèn để trống  | Trạm chỉ có pole, traffic sign, sky, road; theo chuẩn Cityscapes, đèn giao thông là class riêng | Chỉ tô thân cột và tay ngang là `pole`, hộp đèn để trống                                                                                                                                   |
