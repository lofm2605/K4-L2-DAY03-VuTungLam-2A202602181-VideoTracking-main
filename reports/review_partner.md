# Peer review — Day 3

Chép file này thành `reports/review_partner.md`. Reviewer chỉ ghi finding; tác giả
tự sửa bài của mình và điền closure.

| Trường | Giá trị |
| --- | --- |
| Author | Vũ Tùng Lâm — 2A202602181 |
| Reviewer | không có partner — tác giả tự review theo checklist (làm cá nhân, sát giờ nộp) |
| Pair ID | N/A (self-review) |
| CVAT version | 2.75.0 (self-hosted) |
| Thời điểm review | 2026-09-15 23:45–23:55 (trước lock) và 2026-09-16 00:00 (sau khi chấm với reference) |

## Danh sách finding

Mỗi finding bắt buộc có `frame + ID + lỗi + sửa thế nào`. Nếu chưa thống nhất,
dùng `needs-review`; không ép tác giả sửa theo cảm tính.

| # | CVAT frame | MOT frame | ID | Loại lỗi | Quan sát + rule áp dụng | Cách sửa đề xuất | Closure: fixed / not-a-defect / needs-review |
| ---: | ---: | ---: | ---: | --- | --- | --- | --- |
| 1 | 60–77 | 61–78 | 5 | entry sớm | Xe buýt chỉ là dải mỏng ở rìa phải; reference mở track ở frame 79. Rule: "bắt đầu từ frame đầu tiên xác định được là xe". | Dời keyframe đầu tới frame ~79 hoặc giữ nếu chấp nhận rule mở sớm | needs-review — tác giả giữ nhãn, bổ sung ngưỡng "lộ tối thiểu" vào guideline |
| 2 | 78–100 | 79–101 | 6 | occlusion / thiếu cờ | Xe con sau xe buýt chỉ thấy nóc, chưa bật occluded; reference mở track ở 101. Rule: bbox ôm phần nhìn thấy. | Bật occluded frame 79–113; giữ ID | not-a-defect về ID/bbox; thiếu cờ occluded ghi vào guideline |
| 3 | 148–150 / 168–171 | 149–151 / 169–172 | 4 / 8 | exit muộn | Xe đã gần ra khỏi ảnh, bbox chỉ còn dải mỏng; reference đóng sớm hơn 3–4 frame. Rule: outside đúng frame rời khung. | Bấm outside ở frame 149 (ID 4) và 169 (ID 8) | needs-review — chưa sửa trong CVAT vì hết giờ, đã ghi ở REPORT mục 3 |
| 4 | 53 / 83 | 54 / 84 | 4 / 5 | geometry / interpolation | IoU với reference tụt xuống 0.55 và 0.54 giữa hai keyframe cách 10–15 frame khi xe tăng tốc. | Thêm keyframe tại frame 54 và 84 | needs-review — chưa sửa, luật keyframe ≤ 8 frame đã thêm vào guideline |

## Reviewer checklist

| Hạng mục | PASS / FINDING / N/A | Frame–ID–evidence |
| --- | --- | --- |
| Có tối thiểu 6 track hợp lệ; chỉ gồm xe bốn bánh | PASS | 8 track, 631 bbox; không có track ở kiosk/người đi bộ (model bắt nhầm kiosk frame 16–116, nhãn tay không) |
| Một xe giữ một ID; không reuse ID cho xe khác | PASS | 8 ID cho 8 xe; eval vs reference IDSW = 0 |
| Occlusion ngắn giữ ID; crossing không đổi ID | PASS | ID 6 bị xe buýt (ID 5) che frame 79–113, giữ ID; AssA 0.779 |
| Entry/exit đúng; không box treo sau khi xe rời khung | FINDING | Finding #1, #3: entry sớm ID 5 (61–78), exit muộn ID 4 (149–151), ID 8 (169–172) |
| Bbox ôm phần nhìn thấy, không đoán phần bị che/ngoài khung | PASS (thiếu cờ) | ID 6 frame 79–100 bbox ôm phần lộ; chưa bật occluded (finding #2) |
| Frame giữa hai keyframe không bị interpolation drift | FINDING | Finding #4: frame 54 ID 4 (IoU 0.55), frame 84 ID 5 (IoU 0.54) |
| Export đúng MOT 1.1; frame bắt đầu từ 1; cột 2 là track ID | PASS | `check_mot_labels.py` 0 lỗi cho cả clip_01 và clip_02; frame 1..190 |
| Mọi finding có cách sửa và closure do tác giả điền | PASS | 4 finding đều có cách sửa; closure needs-review/not-a-defect do tác giả ghi |

## Self-QC attestation của reviewer

| Lượt | PASS / ĐÃ SỬA / NEEDS-REVIEW | Frame–ID–evidence |
| --- | --- | --- |
| 1 — identity/timeline | PASS | 8 track, mỗi xe một ID, timeline liên tục, 0 IDSW |
| 2 — endpoint/scope | NEEDS-REVIEW | ID 5 vào sớm (61), ID 4/8 ra muộn (151/172) so với reference |
| 3 — geometry/interpolation | NEEDS-REVIEW | frame 54 ID 4 và frame 84 ID 5 IoU ~0.55 |

## Exit ticket

1. Finding quan trọng nhất và rule dùng để kết luận: #1 entry sớm ID 5 (18 frame, chiếm phần lớn 63 FP). Rule dùng: "bắt đầu track từ frame đầu tiên xác định được là xe" — rule này mơ hồ vì "xác định được" phụ thuộc người gán có xem frame sau hay không.
2. Một finding tác giả đóng là `not-a-defect`, kèm lý do (nếu có): #2 ID 6 frame 79–100: xe con thật sự có mặt sau xe buýt, bbox ôm phần nhìn thấy đúng luật; chỉ thiếu cờ occluded chứ không sai ID hay bbox.
3. Một rule cần Lab Coach làm rõ (nếu có): reference mở/đóng track theo ngưỡng nào (bao nhiêu phần xe phải trong ảnh)? Guideline lab chỉ nói "frame đầu tiên xác định được là xe bốn bánh".
