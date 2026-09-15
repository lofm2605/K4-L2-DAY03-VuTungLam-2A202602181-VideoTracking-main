# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: Vũ Tùng Lâm — 2A202602181 (làm cá nhân)
Ngày: 2026-09-15

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT 2.75.0 self-hosted (Docker), Rectangle Track, export MOT 1.1 |
| Thời gian gán `clip_02` (warm-up) | không đo chính xác, làm ngắt quãng (task tạo 14:44, export 23:21) |
| Thời gian gán `clip_01` | ~18 phút (task tạo 23:25, export 23:43, theo log CVAT) |
| Số track đã vẽ trong `clip_01` | 8 (631 bbox, phủ frame 1–190) |
| Số keyframe trung bình mỗi track | ~9.4 (75 keyframe / 8 track; ít nhất 2, nhiều nhất 16) |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Xe buýt vào khung từ mép phải (MOT frame 61–79, ID 5).** Frame 61 chỉ thấy một dải mỏng đầu xe ở rìa ảnh. Tôi bắt đầu track ngay khi nhận ra đó là xe buýt và cho bbox chạm đúng rìa; reference chờ tới frame 79 mới mở track. Đây là chỗ hai bên lệch nhiều nhất (18 frame).
2. **Xe con tối màu bị xe buýt che (MOT frame 79–101, ID 6).** Xe này xuất hiện ở mép phải phía sau xe buýt, chỉ thấy một phần nóc/đuôi. Tôi mở track từ frame 79 với bbox ôm phần nhìn thấy; reference mở ở frame 101 khi xe đã lộ rõ. Tôi không dùng cờ occluded ở đây, đây là thiếu sót.
3. **Khoảng giữa hai keyframe khi xe buýt và xe con cùng di chuyển nhanh (frame 54 và 84, ID 4 và 5).** Bbox nội suy tụt IoU xuống ~0.55 vì xe đổi tốc độ/hướng giữa hai keyframe cách nhau 10–15 frame. Tôi đặt keyframe thưa ở đoạn xe đi thẳng và không dày thêm quanh đoạn tăng tốc.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1 (ID): 8 xe, 8 ID, không ID nào bị dùng lại cho xe khác; validator xác nhận 0 IDSW so với reference. Cảnh báo "track 2 đứng im frame 1–15" là xe con đỗ trước vạch, không phải quên outside.
- Lượt 2 (frame đầu/cuối): tất cả track kết thúc bằng keyframe outside (trừ ID 2 và ID 7 còn trong khung ở frame 190). Tuy vậy so với reference tôi kết thúc ID 4 muộn 3 frame (149–151) và ID 8 muộn 4 frame (169–172): xe đã gần như ra khỏi ảnh nhưng tôi vẫn giữ bbox mỏng ở rìa.
- Lượt 3 (frame giữa): phát hiện hai chỗ nội suy trôi (frame 54 ID 4, frame 84 ID 5, IoU ~0.55) do keyframe thưa khi xe tăng tốc.

Kiểm chéo với: **không có partner** (làm cá nhân, sát giờ nộp). Tôi tự review theo checklist ở `reports/review_partner.md`, lấy validator và bảng lỗi khi chấm với reference làm finding.
Số lỗi bạn tìm được trong bản của bạn ấy: không áp dụng. Số lỗi tự tìm được trong bản của mình: 5 (4 bbox thừa/treo ở đầu–cuối track, 2 bbox trôi gộp thành 1 finding geometry).

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

Khác nhau giữa tôi và reference nằm ở **thời điểm mở/đóng track**: tôi mở track ngay khi nhận ra xe ở rìa (frame 61, 79) còn reference chờ xe lộ đủ; tôi đóng track khi xe khuất hẳn còn reference đóng sớm hơn 3–4 frame. Guideline ban đầu chưa có ngưỡng "xe phải lộ bao nhiêu phần mới mở track" và "còn bao nhiêu phần thì đóng". Đã bổ sung ở mục 3 và 5 của `GUIDELINE_MINI.md`.

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
| Bbox thừa trước khi xe vào khung (theo reference) | 61–78 | 5 | Chưa sửa. Xem lại frame 61–78: xe buýt thật sự đã có một phần trong ảnh, tôi giữ quyết định mở sớm (needs-review, xem mục 5 câu 5). |
| Bbox thừa trước khi xe vào khung (theo reference) | 79–100 | 6 | Chưa sửa. Xe con bị xe buýt che phần lớn, chỉ thấy nóc; đúng luật của tôi (ôm phần nhìn thấy) nhưng đáng lẽ phải bật occluded. Ghi vào guideline. |
| Bbox treo sau khi xe rời khung | 149–151 | 4 | Chưa sửa trong CVAT vì hết giờ. Nếu rework: bấm outside ở frame 149 khi xe chỉ còn dải mỏng ở rìa. |
| Bbox treo sau khi xe rời khung | 169–172 | 8 | Chưa sửa trong CVAT vì hết giờ. Nếu rework: bấm outside ở frame 169. |
| Bbox trôi giữa hai keyframe (IoU 0.55 / 0.54) | 54 / 84 | 4 / 5 | Chưa sửa. Nếu rework: thêm keyframe ở frame 54 và 84, kéo bbox khít lại. |

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

MOTA của tôi (0.881) **thấp hơn** IDF1 (0.944). Nghĩa là lỗi của tôi nằm ở phần phát hiện/coverage (63 FP, chủ yếu là bbox ở đầu–cuối track lệch so với reference) chứ không phải ở identity: 0 ID switch, AssA 0.779. Nếu ngược lại, MOTA cao mà IDF1 thấp, thì xe được phủ đủ nhưng ID bị đổi giữa chừng. MOTA = 1 − (FP + FN + IDSW)/GT chỉ đếm **số lần** đổi ID, mỗi lần đổi tính bằng một lỗi; một track bị cắt đôi ở giữa chỉ mất 1 điểm trong 573. IDF1 thì tính theo **số frame** được gán đúng ID xuyên suốt, nên một lần đổi ID ở giữa track dài kéo cả nửa track thành IDFP/IDFN. Vì vậy IDF1 (và AssA) mới là chỉ số phản ánh lỗi identity.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

Treatment cao hơn control ở IDF1 (0.900 so với 0.875) và AssA (0.820 so với 0.776); IDSW bằng nhau (2 và 2). Cả hai đều sinh 16 track cho 8 xe. Chuỗi frame đáng chú ý là **xe buýt (gold ID 5) đoạn frame 85–94**: ByteTrack mở track 23 ở frame 85 rồi đổi sang track 32 ở frame 94 (ID switch), còn treatment mở track 17 ở frame 85 rồi đổi sang 18 ngay frame 87 và giữ 18 đến hết (52 frame). Cả hai cùng bị một lần đổi ID ở đầu xe buýt khi xe mới lộ một phần ở rìa, nhưng treatment ổn định sớm hơn 7 frame nên phần track dài hơn được tính đúng ID, kéo AssA và IDF1 lên. Chỗ thứ hai là xe con sau xe buýt (gold ID 6): ByteTrack không đổi ID nhưng chỉ phủ 42/56 frame; treatment đổi ID ở frame 113 (24 → 31) khi xe vừa ló ra khỏi xe buýt, đúng đoạn occlusion nặng, cho thấy appearance embedding lấy từ bbox bị che không đủ phân biệt. Chênh lệch này **không** cô lập được tác dụng của ReID: BoT-SORT và ByteTrack khác nhau ở Kalman model, ngưỡng match và cách fuse score, nên một phần khác biệt đến từ chính implementation tracker.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

DetA tăng từ 0.649 lên 0.711, FN giảm mạnh từ 54 xuống 26, FP gần như không đổi (88 → 91). Cùng một detector input nên FN giảm không phải vì detector tốt hơn mà vì treatment giữ track qua các frame detection yếu (track_buffer và match theo appearance), tức là lỗi FN của control là **association**. Phần FP còn lại (~90) ở cả hai run là **detector**: cả hai cùng có track ma tại vùng kiosk giữa ảnh (track 10/7, frame 16–116, 42–43 frame) và track 41/27 (frame 106–121), là những vật thể không phải xe bốn bánh trong reference nhưng YOLO26n vẫn ra bbox class car/truck với conf ≥ 0.25. Lỗi này tracker nào cũng không sửa được, phải lọc ở detector (conf, class) hoặc vùng ROI.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Frame 16–116, vùng kiosk bán hàng giữa ảnh (không có ID trong nhãn của tôi; ReID track 7, ByteTrack track 10). Cả hai model gán một bbox ổn định suốt 43 frame lên quầy kiosk có mái vuông và người đứng xung quanh. Tôi không gán vì đó không phải xe; reference cũng không có track nào ở đó. Đây là false positive của detector mà ReID còn làm "bền" hơn vì appearance của kiosk không đổi qua các frame.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

Frame 61–78, xe buýt ở mép phải (ID 5 của tôi). ReID (và reference) chỉ bắt đầu xe buýt ở frame 85/79, còn tôi mở track từ frame 61 khi chỉ thấy một dải mỏng ở rìa ảnh. Xem lại frame 61: phần xe trong ảnh chưa đủ để một người mới nhìn nhận ra là xe buýt nếu không xem các frame sau. Tôi chưa sửa annotation (chỉ sửa khi có luật rõ), nhưng đã thêm luật ngưỡng "lộ tối thiểu" vào guideline để lần sau quyết định nhất quán. Ngược lại, ở frame 79–100 (ID 6, xe con sau xe buýt) tôi giữ quyết định của mình: xe đã thấy được nóc và một phần thân, đúng luật "bbox ôm phần nhìn thấy".

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

- Guideline: (1) ngưỡng mở track — chỉ mở khi thấy được ít nhất khoảng một nửa chiều dài xe hoặc nhận ra được loại xe trong chính frame đó, không dựa vào frame sau; (2) ngưỡng đóng track — bấm outside ngay frame xe chỉ còn dải mỏng dưới ~10% chiều rộng ở rìa; (3) bắt buộc bật occluded khi phần nhìn thấy dưới một nửa; (4) keyframe cách nhau tối đa 8 frame ở đoạn xe tăng tốc, rẽ hoặc bị che, tối đa 15 frame ở đoạn đi thẳng.
- Quy trình: chạy `check_mot_labels.py` ngay sau lần export đầu chứ không đợi cuối; chạy `visualize_tracks.py` để soát lượt 3 (geometry) bằng mắt trước khi lock; có partner review thật thay vì tự review; ghi guideline ngay khi phát sinh ca mơ hồ chứ không viết lại sau.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `reports/review_partner.md` (tự review, không có partner)
- [x] `reports/REPORT.md` (file này)
