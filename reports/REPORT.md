# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: Vũ Tùng Lâm — 2A202602181 (làm cá nhân)
Ngày: 2026-09-15

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT 2.75.0 self-hosted (Docker), Rectangle Track, export MOT 1.1 |
| Thời gian gán `clip_02` (warm-up) | `...` phút |
| Thời gian gán `clip_01` | `...` phút |
| Số track đã vẽ trong `clip_01` | 8 (631 bbox, phủ frame 1–190) |
| Số keyframe trung bình mỗi track | `...` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. `...`
2. `...`
3. `...`

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `...`
- Lượt 2: `...`
- Lượt 3: `...`

Kiểm chéo với: `...`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của bạn: `...`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`...`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `12b0dd8dc0d2c51b3154e1c412d46f622342d24940e551f3f0298fa1a4a96f0c` |
| Thời điểm khóa | 2026-09-15T16:48:28Z (23:48 Asia/Ho_Chi_Minh) |
| Số row / frame / track trước khi mở reference | 631 row / 190 frame / 8 track |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.750 | 0.727 | 0.779 | 0.833 | 0.944 | 0.881 | 0.812 | 63 | 5 | 0 |
| Sau rework | 0.750 | 0.727 | 0.779 | 0.833 | 0.944 | 0.881 | 0.812 | 63 | 5 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có** (bản pre-gold đã qua cổng; chưa rework, nên hai dòng giống nhau — `outputs/eval_pre_gold.json` và `outputs/eval_vs_gold.json`).

Danh sách lỗi validator đưa ra khi chấm với reference (chưa sửa):

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox thừa trước khi xe vào khung (theo reference) | 61–78 | 5 | `...` (chưa sửa / not-a-defect vì ...) |
| Bbox thừa trước khi xe vào khung (theo reference) | 79–100 | 6 | `...` |
| Bbox treo sau khi xe rời khung | 149–151 | 4 | `...` |
| Bbox treo sau khi xe rời khung | 169–172 | 8 | `...` |
| Bbox trôi giữa hai keyframe (IoU 0.55 / 0.54) | 54 / 84 | 4 / 5 | `...` |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | 3.12.13 / 8.4.153 / 2.14.0+cpu / 0.5.13 |
| weights / hai tracker | `yolo26n.pt` / `bytetrack.yaml` (control) và `configs/trackers/botsort-reid.yaml` (treatment, `with_reid: true`, `gmc_method: none`) |
| conf / IoU / imgsz / classes | 0.25 / 0.70 / 960 / [2, 5, 7] = car, bus, truck |
| device | cpu (chạy bằng `tools/run_tracker.py`, `persist=True`) |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.750 | 0.727 | 0.779 | 0.833 | 0.944 | 0.881 | 0.812 | 63 | 5 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.684 | 0.619 | 0.761 | 0.839 | 0.859 | 0.720 | 0.816 | 91 | 84 | 2 |

Cả hai model đều sinh 16 track cho 8 xe của reference (ByteTrack 607 bbox, ReID 638 bbox).

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

`...`

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

`...`

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

`...`

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

`...`

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

`...`

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

`...`

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [ ] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [ ] `reports/REPORT.md` (file này)
