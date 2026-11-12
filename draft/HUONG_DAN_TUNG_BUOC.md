# Hướng dẫn làm Lab Day03 — Image Segmentation (từng bước)

> File tổng hợp từ toàn bộ tài liệu trong repo ([README](../README.md), [GUIDE](../GUIDE.md), [CHECKPOINTS](../CHECKPOINTS.md), [CVAT_SETUP](../CVAT_SETUP.md), [RUBRIC](../RUBRIC.md), [RULES](../RULES.md), [SUBMISSION](../SUBMISSION.md), [docs/](../docs/), [phiếu quy tắc](../guideline-mini-sheet.md), [lab-guide.html](../lab-guide.html)). Làm lần lượt từ Bước 0 đến Bước 12, tick `[x]` khi xong.
>
> **Deadline:** 12/11/2026 lúc 12:00 trưa (UTC+7). Nộp **link fork** trên [LMS](https://aithucchien.vinuni.edu.vn).

---

## Bước 0 — Hiểu bài trong 2 phút

Bài cá nhân, 240 phút trên lớp, tối đa 100 điểm. Bạn **tự vẽ mask trong CVAT** cho 14 ảnh (9 task), xuất ZIP, rồi viết 1 report. Không bắt buộc code, Python, Jupyter hay SAM.

**Ba loại segmentation:**

| Loại     | Câu hỏi                                     | Ví dụ                                       |
| -------- | ------------------------------------------- | ------------------------------------------- |
| Semantic | Pixel thuộc **loại vùng** nào?              | road và sidewalk là 2 lớp dù màu giống nhau |
| Instance | Pixel thuộc **vật nào**?                    | 2 xe sát nhau = 2 object riêng              |
| Panoptic | Cả hai: vùng nền (stuff) + từng vật (thing) | road, sky + car #1, car #2                  |

- **Mask** = vùng pixel gán cho một lớp/vật. Bounding box **không** thay được mask.
- **Stuff** = vùng không đếm được (road, sky…). **Thing** = vật đếm được (car, person…).
- Chỉ vẽ **phần nhìn thấy**, không đoán phần bị che.

**Bảng 9 task:**

|   # | Task              | Loại     |    Ảnh | Thư mục ảnh                                                                        | Class (đúng từng chữ)                                                                                                         | Export CVAT           |    Điểm |
| --: | ----------------- | -------- | -----: | ---------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- | --------------------- | ------: |
|   1 | `easy_semantic`   | Semantic |      3 | [data/tiers/easy_semantic/images](../data/tiers/easy_semantic/images/)             | road, sidewalk, building, vegetation, sky                                                                                     | Segmentation mask 1.1 |      20 |
|   2 | `medium_instance` | Instance |      3 | [data/tiers/medium_instance/images](../data/tiers/medium_instance/images/)         | person, bicycle, car, motorcycle, bus, truck                                                                                  | COCO 1.0              |      32 |
|   3 | `hard_panoptic`   | Panoptic |      2 | [data/tiers/hard_panoptic/images](../data/tiers/hard_panoptic/images/)             | **Stuff:** road, sidewalk, building, vegetation, sky · **Thing:** person, car, bus, truck, motorcycle, bicycle, traffic light | COCO 1.0              |      30 |
|   4 | `cp1_holes`       | Instance |      1 | [data/checkpoints/cp1_holes/images](../data/checkpoints/cp1_holes/images/)         | person, bicycle, car, motorcycle, bus, truck                                                                                  | COCO 1.0              |       3 |
|   5 | `cp2_slice`       | Instance |      1 | [data/checkpoints/cp2_slice/images](../data/checkpoints/cp2_slice/images/)         | person, bicycle, car, motorcycle, bus, truck                                                                                  | COCO 1.0              |       3 |
|   6 | `cp5_occlusion`   | Instance |      1 | [data/checkpoints/cp5_occlusion/images](../data/checkpoints/cp5_occlusion/images/) | person, bicycle, car, motorcycle, bus, truck                                                                                  | COCO 1.0              |       3 |
|   7 | `cp3_thin`        | Semantic |      1 | [data/checkpoints/cp3_thin/images](../data/checkpoints/cp3_thin/images/)           | pole, traffic sign, sky, road                                                                                                 | Segmentation mask 1.1 |       3 |
|   8 | `cp4_curb`        | Semantic |      1 | [data/checkpoints/cp4_curb/images](../data/checkpoints/cp4_curb/images/)           | road, sidewalk                                                                                                                | Segmentation mask 1.1 |       3 |
|   9 | `cp6_coverage`    | Semantic |      1 | [data/checkpoints/cp6_coverage/images](../data/checkpoints/cp6_coverage/images/)   | road, sidewalk, building, vegetation, sky, car, person                                                                        | Segmentation mask 1.1 |       3 |
|     | **Tổng**          |          | **14** |                                                                                    |                                                                                                                               |                       | **100** |

> Nguồn chuẩn: [data/manifest.json](../data/manifest.json) và `classes.json` của từng task. **Mỗi task có class list riêng** — đừng dùng class của Easy cho mọi task.

**Lộ trình 240 phút:**

|    Phút | Việc                                            | Kết quả                                           |
| ------: | ----------------------------------------------- | ------------------------------------------------- |
|    0–15 | Mở CVAT, đọc quy tắc, tạo task Easy             | Task đầu mở được, biết chỗ Save/Export            |
|   15–45 | Easy semantic                                   | `easy_semantic.zip`                               |
|  45–110 | Medium instance                                 | `medium_instance.zip` + ghi object đầu vào report |
| 110–120 | Nghỉ                                            | Giữ ZIP an toàn                                   |
| 120–175 | Hard panoptic                                   | `hard_panoptic.zip`                               |
| 175–185 | Nghỉ; ground truth 3 tier được phát từ phút 180 | Không đưa đáp án lên fork                         |
| 185–215 | 6 checkpoint; xong sớm thì chạy tự chấm         | ZIP các trạm                                      |
| 215–235 | Kiểm file, sửa/export lại, điền report          | Report hoàn chỉnh                                 |
| 235–240 | Save lần cuối, kiểm ZIP + report                | Sẵn sàng push                                     |

---

## Bước 1 — Chuẩn bị repo và tài liệu

- [x] Fork đã có: `https://github.com/MHuy21/K5-L2B-DAY03-DuongManhHuy-2A202603301-Image-Segmentation`. Quy ước tên trong [SUBMISSION.md](../SUBMISSION.md) là `K5-L2-DAY03-HoVaTen-MSSV-Image-Segmentation` (không dấu, không khoảng trắng) — nếu không chắc tên hiện tại (`L2B`) có được chấp nhận, hỏi coach.
- [x] Repo để **public** hoặc coach có quyền đọc.
- [x] Trên GitHub fork → tab **Actions** → bật workflows (fork mới không tự bật). Không cần tạo secret/token.
- [x] Đọc [phiếu quy tắc gán nhãn](../guideline-mini-sheet.md) (1 trang).
- [ ] Mở [lab-guide.html](../lab-guide.html) bằng Chrome/Edge để xem ảnh chụp giao diện CVAT (ảnh trong [docs/images/](../docs/images/) chỉ để tìm nút, **không phải ảnh chấm**).

---

## Bước 2 — Chạy CVAT local

- [ ] Chạy CVAT local theo hướng dẫn của lớp.

> Repo không ghi lệnh cài. Nếu máy chưa có CVAT, cách cài chuẩn (tham khảo tài liệu chính thức của CVAT, cần Docker Desktop):
>
> ```bash
> git clone https://github.com/cvat-ai/cvat
> cd cvat
> docker compose up -d
> docker exec -it cvat_server bash -ic 'python3 ~/manage.py createsuperuser'
> ```
>
> Mở `http://localhost:8080` và đăng nhập bằng tài khoản vừa tạo. **Clone CVAT ra ngoài thư mục repo bài lab.** Nếu lớp có cấu hình riêng, làm theo lớp.

- [ ] Không vào được CVAT → chụp màn hình lỗi, ghi thời điểm + địa chỉ đã thử, báo coach.
- [ ] **Không thấy SAM là bình thường** — dùng Brush/Polygon. Intelligent Scissors/SAM chỉ là tùy chọn.

---

## Bước 3 — Quy trình chung cho MỌI task trong CVAT

Lặp lại các bước con này cho cả 9 task.

**3.1 Tạo task** ([ảnh](../docs/images/03-task-form.png))

- [ ] **Tasks → Create new task**.
- [ ] **Name** = đúng mã task, ví dụ `easy_semantic`.
- [ ] Upload **chỉ** các JPG trong `images/` của task đó. Không gộp ảnh của nhiều task vào một task.

**3.2 Thêm labels** — chọn 1 trong 2 cách ([ảnh](../docs/images/04-label.png))

- [ ] **Cách A (nhanh):** mở `cvat-labels.json` của **đúng task**, copy toàn bộ từ `[` đến `]` → **Labels → Raw** → dán đè → **Done**.
- [ ] **Cách B:** **Constructor → Add label**, thêm từng tên trong `classes.json`.
- [ ] ❌ **Không dán `classes.json` vào Raw** (file có metadata, CVAT không nhận).
- [ ] Kiểm số lượng và tên label trước khi **Submit**. `traffic sign` có dấu cách, không đổi thành `traffic_sign`.
- [ ] Task đã có annotation thì **không dán đè Raw** (có thể mất liên kết annotation) — báo coach nếu cần sửa label.

**3.3 Vẽ mask** ([ảnh brush](../docs/images/08-brush.jpg), [ảnh polygon](../docs/images/07-polygon.png))

- [ ] Mở **Job** của task.
- [ ] Chọn **Brush** hoặc **Polygon** trong công cụ mask, chọn đúng label rồi vẽ theo phần nhìn thấy.
- [ ] Phóng to ở ranh giới; thu nhỏ brush cho nét mảnh.
- [ ] Dùng danh sách **Objects** để kiểm class và số object.

**3.4 Tự QC rồi Save** — đi qua từng ảnh theo thứ tự:

1. Đúng ảnh?
2. Đúng loại segmentation và đúng tên class?
3. Đủ vùng/vật? Có thiếu, thừa, gộp 2 vật, tách nhầm 1 vật không?
4. Biên có tràn sang nền/bóng, hay bỏ sót vùng rõ ràng?
5. Bấm **Save** → chuyển sang ảnh khác → quay lại xem dữ liệu còn không.

**3.5 Export** ([menu](../docs/images/10-export-menu.jpg), [chọn format](../docs/images/11-export-format.jpg))

- [ ] Menu Job → **Export job dataset**.
- [ ] Semantic → `Segmentation mask 1.1`; Instance/Panoptic → `COCO 1.0`.
- [ ] Tải ZIP, **đổi tên file ZIP bên ngoài** thành `<mã_task>.zip` (giữ nguyên nội dung bên trong).
- [ ] Chép vào thư mục [submissions/](../submissions/) của repo.
- [ ] Export lỗi hoặc không thấy format → giữ dữ liệu đã Save, chụp lỗi, báo coach. **Không đổi sang format khác, không sửa JSON/PNG trong ZIP bằng tay.**

---

### Phương án B — pre-label bằng SAM rồi sửa trong CVAT

Thay vì vẽ từ đầu ở 3.3: chạy [sam_prelabel/prelabel.py](sam_prelabel/prelabel.py) để tạo mask nháp (YOLO11-seg + SAM 2.1 cho vật, SegFormer cho vùng), import vào task CVAT qua **Actions → Upload annotations → COCO 1.0**, sửa lại rồi tiếp tục từ 3.4. Hướng dẫn đầy đủ: [sam_prelabel/README.md](sam_prelabel/README.md).

---

## Bước 4 — Easy semantic (20 điểm · phút 15–45)

Ảnh: `7ee6d192-89e2408b.jpg`, `817bca71-00000000.jpg`, `81ae7cbb-6bc63a4a.jpg`

- [ ] Tạo task `easy_semantic`, 3 ảnh, labels từ [cvat-labels.json](../data/tiers/easy_semantic/cvat-labels.json) → đúng **5 class**: `road`, `sidewalk`, `building`, `vegetation`, `sky`.
- [ ] Tô phủ vùng nhìn thấy của từng lớp.
- [ ] Ranh `road`–`sidewalk` theo **chức năng và bó vỉa**, không theo màu nhựa đường.
- [ ] Kiểm các mảng rõ ràng còn trống.
- [ ] Save → Export **Segmentation mask 1.1** → `submissions/easy_semantic.zip`.
- [ ] Trong lúc vẽ, ghi nháp các ca phân vân (dùng cho report mục 4).

Chấm: mIoU theo class (floor 0.40, đủ điểm ở 0.85).

---

## Bước 5 — Medium instance (32 điểm · phút 45–110)

Ảnh: `000000181542.jpg`, `000000373353.jpg`, `000000458325.jpg`

- [ ] Tạo task `medium_instance`, 3 ảnh, labels từ [cvat-labels.json](../data/tiers/medium_instance/cvat-labels.json) → **6 class**: `person`, `bicycle`, `car`, `motorcycle`, `bus`, `truck`.
- [ ] ⚠️ **Bắt buộc:** tự vẽ **object đầu tiên bằng tay, trước khi xem bất kỳ gợi ý tự động nào**. Ghi ngay vào report mục 2: ảnh, vị trí, class, quy tắc chọn biên.
- [ ] Vẽ tiếp: **mỗi vật đếm được = 1 object riêng**. 2 xe cùng class sát nhau vẫn là 2 object.
- [ ] Nếu dùng gợi ý (SAM/Scissors): tự kiểm class, số object, biên. Gợi ý hay ăn bóng/nền, gộp 2 xe, bỏ sót chi tiết. Ghi 1 chỗ đã sửa hoặc lý do giữ đề xuất.
- [ ] Kiểm thiếu/thừa vật, gộp/tách nhầm, biên ăn nền bằng danh sách Objects.
- [ ] Save → Export **COCO 1.0** → `submissions/medium_instance.zip`.

Chấm: mean matched IoU × recall@0.5 (floor 0.40, cap 0.85). Bỏ sót vật làm giảm recall — mask khít không bù được vật bị bỏ sót.

---

## Bước 6 — Hard panoptic (30 điểm · phút 120–175)

Ảnh: `000000350023.jpg`, `000000460147.jpg`

- [ ] Tạo task `hard_panoptic`, 2 ảnh, labels từ [cvat-labels.json](../data/tiers/hard_panoptic/cvat-labels.json) → đủ **12 class**.
- [ ] Vẽ **stuff**: `road`, `sidewalk`, `building`, `vegetation`, `sky`.
- [ ] Vẽ **từng thing** riêng: `person`, `car`, `bus`, `truck`, `motorcycle`, `bicycle`, `traffic light`. Một mask `car` **không** thay cho mọi xe.
- [ ] Vẽ **từ xa tới gần** (nền trước, vật gần sau).
- [ ] Kiểm bằng mắt: chồng lấn giữa các mask, khoảng trống bất hợp lý.
- [ ] Pixel không quyết định được → ghi vào report là ca mơ hồ, **không bịa class**.
- [ ] Save → Export **COCO 1.0** → `submissions/hard_panoptic.zip`.

Chấm: PQ trên stuff và từng thing (floor 0.20, cap 0.65). ZIP COCO hợp lệ **không** chứng minh panoptic đúng.

---

## Bước 7 — Sáu checkpoint (6 × 3 = 18 điểm · phút 185–215)

Mỗi trạm 1 ảnh, **tạo task riêng**, dùng `cvat-labels.json` riêng của trạm.

| Trạm            | Ảnh                     | Class                                                  | Cần làm                                                           | Lỗi dễ gặp                                         | Export                |
| --------------- | ----------------------- | ------------------------------------------------------ | ----------------------------------------------------------------- | -------------------------------------------------- | --------------------- |
| `cp1_holes`     | `000000144300.jpg`      | 6 class instance                                       | Kính/khe **nằm trong** mask vật, **không khoét lỗ**               | Khoét tùy tiện làm vật bị rỗng                     | COCO 1.0              |
| `cp2_slice`     | `000000017627.jpg`      | 6 class instance                                       | Hai xe sát nhau = **hai instance**                                | Một mask gộp hai xe                                | COCO 1.0              |
| `cp5_occlusion` | `000000336232.jpg`      | 6 class instance                                       | Vật bị che thành nhiều mảng rời vẫn là **một instance**; đếm đúng | Tách thành 2 object, hoặc vẽ xuyên qua vùng bị che | COCO 1.0              |
| `cp3_thin`      | `839f7736-abe28069.jpg` | pole, traffic sign, sky, road                          | Cột/biển mảnh: **zoom lớn, brush 2–3 px**                         | Bỏ sót cột/biển hoặc brush quá dày                 | Segmentation mask 1.1 |
| `cp4_curb`      | `7d83710e-4697c3b2.jpg` | road, sidewalk                                         | Ranh road–sidewalk theo **bó vỉa/chức năng**                      | Chọn theo màu nhựa đường                           | Segmentation mask 1.1 |
| `cp6_coverage`  | `7daa6479-67988f3f.jpg` | road, sidewalk, building, vegetation, sky, car, person | Phủ hết vùng nhìn thấy thuộc 7 class                              | Khe trống lớn, hoặc tô ép vùng không chắc          | Segmentation mask 1.1 |

Với mỗi trạm:

- [ ] `cp1_holes` → `submissions/cp1_holes.zip`
- [ ] `cp2_slice` → `submissions/cp2_slice.zip`
- [ ] `cp5_occlusion` → `submissions/cp5_occlusion.zip`
- [ ] `cp3_thin` → `submissions/cp3_thin.zip`
- [ ] `cp4_curb` → `submissions/cp4_curb.zip`
- [ ] `cp6_coverage` → `submissions/cp6_coverage.zip`

Ca mơ hồ → ghi 2 cách hiểu và bằng chứng vào report, không bịa nhãn.

---

## Bước 8 — Tự kiểm cấu trúc ZIP (tùy chọn, không cần thư viện ngoài)

Cần Python 3.10+. Từ thư mục gốc repo (Windows dùng `py -3`, macOS/Linux dùng `python3`):

```bash
py -3 scripts/inspect_submissions.py --dir submissions
# hoặc kiểm một task:
py -3 scripts/inspect_submissions.py --dir submissions --task easy_semantic
```

> Nếu Windows báo `UnicodeEncodeError` (không in được chữ "THIẾU"), chạy `$env:PYTHONUTF8=1` trong PowerShell trước rồi chạy lại lệnh. Đây là lỗi hiển thị, không phải lỗi ZIP.

| Kết quả | Ý nghĩa                                    | Làm gì                                       |
| ------- | ------------------------------------------ | -------------------------------------------- |
| `OK`    | Cấu trúc ZIP đúng (tên ảnh, class, format) | Vẫn phải tự kiểm mask bằng mắt               |
| `THIẾU` | Chưa có ZIP của task                       | Không phải lỗi; làm task hoặc ghi vào report |
| `LỖI`   | ZIP sai cấu trúc                           | Sửa trong CVAT → Save → export lại           |

Lưu ý: số `annotations` của COCO chỉ là **số mask bạn đã nộp**, không phải số object đúng. Script không đọc đáp án, không tính điểm.

Thay thế: [notebook Colab](../notebooks/day3-segmentation-tu-kiem.ipynb) theo [notebooks/README.md](../notebooks/README.md) — cũng chỉ kiểm cấu trúc.

---

## Bước 9 — Tự chấm sau khi phát ground truth (tùy chọn, sau phút 180)

Chỉ cho 3 tier Easy + Medium + Hard, tối đa **82 điểm** (không phải điểm chính thức /100). Checkpoint không tự chấm được.

**Cách 1 — GitHub Actions (dễ nhất):**

- [ ] Push ZIP vào `submissions/` trên fork (xem Bước 11).
- [ ] **Actions → Day 3 self-check → lần chạy mới nhất → Summary**.
- [ ] Trước khi có đáp án: Summary chỉ kiểm cấu trúc, ghi "chưa có ground truth" (không phải điểm 0).
- [ ] Sau khi đáp án được công bố: bấm **Run workflow** hoặc push ZIP mới → Summary hiện scorecard /82.

**Cách 2 — chạy trên máy:**

1. Giải nén gói đáp án đúng chỗ (thư mục `groundtruth/` đã nằm trong `.gitignore`):
   ```text
   data/tiers/easy_semantic/groundtruth/<tên_ảnh>.png
   data/tiers/medium_instance/groundtruth/instances.json
   data/tiers/hard_panoptic/groundtruth/panoptic.json
   data/tiers/hard_panoptic/groundtruth/png/<tên_ảnh>.png
   ```
2. Cài thư viện và chạy:
   ```bash
   py -3 -m pip install -r requirements.txt
   py -3 scoring/score.py --list
   py -3 scoring/score.py easy_semantic submissions/easy_semantic.zip --group tiers
   py -3 scoring/score.py medium_instance submissions/medium_instance.zip --group tiers
   py -3 scoring/score.py hard_panoptic submissions/hard_panoptic.zip --group tiers
   py -3 scoring/scorecard.py --group tiers --dir submissions --out reports/tiers
   ```
3. Đọc kết quả:
   - Easy: `per-class IoU` — class nào thấp thì xem lại ranh/vùng đó.
   - Medium: `mean matched IoU`, `R@0.5`, `FP/FN` — mask ăn nền, vật thừa, bỏ sót.
   - Hard: `PQ`, `SQ` (chất lượng biên), `RQ` (vật/vùng thiếu-thừa).
   - Công thức điểm: `clamp((metric − floor)/(cap − floor), 0, 1) × trọng số`.
4. Sửa trong **CVAT → Save → export lại → chạy lại**. Ghi 1 lỗi thật và cách sửa vào report mục 3.

⚠️ **Không commit/push ground truth** (`groundtruth/`, `instances.json`, `panoptic.json`, PNG đáp án), không upload lên Colab công khai. `REVIEW SIGNALS`/`SUSPECT` chỉ là cờ cần xem lại, không phải kết luận gian lận.

---

## Bước 10 — Điền report (phút 215–235)

- [ ] Đổi tên [report/individual/MSSV_HoTen.md](../report/individual/MSSV_HoTen.md) thành `report/individual/2A202603301_DuongManhHuy.md` (MSSV + họ tên thật, không dấu).
- [ ] Thay mọi dấu `…` bằng nội dung thật. Giữ nguyên 4 mục và bảng.
- [ ] Đầu file: mã học viên, ngày / CVAT local, công cụ **thật sự** đã dùng.

| Mục                           | Cần ghi                                                                                                                                                                                                                  |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **1. Bài đã nộp**             | Tên ZIP + số ảnh đã làm cho từng task. Task chưa làm ghi `chưa có`. **Không tự điền điểm.**                                                                                                                              |
| **2. Quyết định trước gợi ý** | Object Medium đầu tiên tự vẽ: ảnh, vị trí, class, quy tắc chọn biên. Nếu dùng gợi ý sau đó: vùng sai/đúng, sửa/giữ, lý do. Không dùng thì ghi "không dùng" + giải thích 1 quyết định.                                    |
| **3. Một lỗi đã sửa**         | Task/ảnh/vùng; loại lỗi (sai lớp / thiếu-thừa / gộp-tách / biên / phủ vùng); bằng chứng nhìn thấy; quy tắc + hành động sửa; đã Save + export lại chưa. Có thể ghi metric trước/sau nếu đã tự chấm. **Lỗi phải có thật.** |
| **4. Ba ca cân nhắc**         | Mỗi ca: ảnh/vị trí · 2 cách hiểu · quy tắc/chứng cứ · quyết định hoặc câu hỏi cho coach.                                                                                                                                 |

> Report có link tới `reports/REPORT_TEMPLATE.md` nhưng file này **không có trong repo** — bỏ qua link đó, dựa vào bảng trên.

---

## Bước 11 — Đưa bài lên fork và nộp LMS

Cấu trúc cần có trên fork:

```text
report/individual/2A202603301_DuongManhHuy.md
submissions/easy_semantic.zip
submissions/medium_instance.zip
submissions/hard_panoptic.zip
submissions/cp1_holes.zip
submissions/cp2_slice.zip
submissions/cp3_thin.zip
submissions/cp4_curb.zip
submissions/cp5_occlusion.zip
submissions/cp6_coverage.zip
```

- [ ] Commit và push report + các ZIP lên fork (bằng Git, hoặc trên GitHub: **Add file → Upload files** vào `submissions/`, sửa report bằng nút **Edit** → **Commit changes**).
- [ ] Mở tab **Actions** xem Day 3 self-check chạy xong, không có dòng `LỖI`.
- [ ] Mở lại fork trên trình duyệt, kiểm file đã hiện đủ.
- [ ] Copy URL **fork cá nhân** (không phải repo đề bài) → dán vào ô nộp Day03 trên [LMS](https://aithucchien.vinuni.edu.vn) → xác nhận.

**Checklist cuối trước khi nộp:**

- [ ] Tên repo đúng quy ước K5-L2-DAY03 + họ tên + MSSV.
- [ ] Report đã đổi tên, điền đủ 4 mục.
- [ ] Mỗi ZIP đúng tên, đúng format, có dữ liệu. Task chưa làm thì **không có ZIP rỗng** và đã ghi trong report.
- [ ] Không có ground truth, secret/token, `.env`, bản sao ảnh gốc, file tạm notebook.
- [ ] Coach đọc được repo; link nộp trước 12:00 ngày 12/11/2026.

Tùy chọn — đóng thêm 1 gói lưu trữ (không thay ZIP riêng trên fork):

```bash
py -3 scripts/package_submission.py --learner-id <mã_học_viên> --report report/individual/2A202603301_DuongManhHuy.md
```

Muốn nộp bản mới trước hạn: sửa trong CVAT → Save → export lại → push lại.

---

## Bước 12 — Khi bị kẹt

| Tình huống                       | Làm ngay                                                                                    |
| -------------------------------- | ------------------------------------------------------------------------------------------- |
| Không vào được CVAT              | Chụp lỗi, ghi thời điểm + địa chỉ đã thử, báo coach                                         |
| Không thấy SAM                   | Dùng Brush/Polygon, không chờ cài tool                                                      |
| Không biết chọn class/ranh       | Mở `classes.json` + [phiếu quy tắc](../guideline-mini-sheet.md); ghi 2 cách hiểu vào report |
| Hai vật sát nhau / vật bị che    | Xem quy tắc instance/checkpoint; đếm object trong CVAT trước khi Save                       |
| Export sai format hoặc thiếu ảnh | Save, export lại đúng task; không có format thì báo coach                                   |
| Không có Python/Jupyter          | Làm toàn bộ trên CVAT, nộp ZIP + report như bình thường                                     |
| Action đỏ                        | Mở log bước lỗi, báo coach; vẫn nộp bài cốt lõi                                             |

**Những điều không được làm:**

- ❌ Sửa JSON/PNG bên trong ZIP hoặc sửa workflow để "xanh"/"đạt điểm".
- ❌ Dán `classes.json` vào Labels → Raw.
- ❌ Dùng class list của task khác.
- ❌ Đưa ground truth lên fork/Colab/LMS hoặc chia sẻ đáp án.
- ❌ Tạo ZIP rỗng cho task chưa làm.
- ❌ Sao chép ZIP/report của người khác; dùng AI viết kết quả hoặc lỗi chưa quan sát.
- ❌ Xem gợi ý tự động trước khi tự vẽ object Medium đầu tiên.
