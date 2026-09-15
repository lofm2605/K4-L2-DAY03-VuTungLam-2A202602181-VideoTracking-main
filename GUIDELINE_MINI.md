# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: Vũ Tùng Lâm — 2A202602181 (làm cá nhân)
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có): Xe buýt khớp nối (2 khoang) tính là **một** xe, một ID, một bbox ôm cả hai khoang. Kiosk/quầy bán hàng giữa ảnh không phải xe dù model hay bắt nhầm.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | Trong 2 giây xe không kịp đổi làn/hướng đáng kể ở góc camera cố định này, nên có thể nối identity chắc chắn. |
| Xe bị che lâu hơn ngưỡng trên | vẫn giữ ID nếu xác định được bằng màu, loại xe và vị trí quỹ đạo hợp lý; nếu không chắc thì mở track mới và ghi vào mục 4 | Không có ca nào trong clip_01 vượt 25 frame; luật này dự phòng. |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | Ngoài khung không có bằng chứng hình ảnh; camera cố định nên xe quay lại thường là chuyến khác. |
| Hai xe cắt nhau / chồng lên nhau | mỗi xe giữ ID riêng; xe bị che ôm phần nhìn thấy và bật occluded; đặt keyframe ở frame bắt đầu che, frame che nhiều nhất và frame lộ lại | Ca xe buýt (ID 5) che xe con (ID 6) frame 79–113: nếu chỉ có keyframe hai đầu thì bbox nội suy chạy xuyên xe buýt. |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: **thấy được ít nhất một nửa chiều dài xe hoặc nhận ra loại xe trong chính frame đó** (rút ra sau khi chấm với reference: lần đầu tôi mở ID 5 ở frame 61 khi chỉ thấy dải mỏng ở rìa, reference mở ở 79) |
| Xe đang đỗ, không di chuyển | vẫn gán và giữ một ID suốt thời gian trong khung (ID 2 clip_01 đứng yên frame 1–15 rồi mới đi); chỉ cần keyframe ở đầu và ở frame bắt đầu chuyển động |
| Keyframe đặt dày ở đâu | frame vào/ra khung, lúc bắt đầu và kết thúc bị che, lúc xe rẽ hoặc tăng/giảm tốc; tối đa 8 frame giữa hai keyframe ở đoạn này, tối đa 15 frame ở đoạn đi thẳng đều (rút ra từ hai chỗ trôi IoU ~0.55 ở frame 54 và 84) |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: clip_01 / MOT frame 61–79 (CVAT 60–78) / ID 5
- Tình huống: xe buýt vào khung từ mép phải, frame 61 chỉ thấy một dải mỏng đầu xe.
- Quyết định: mở track từ frame 61, bbox chạm đúng rìa ảnh.
- Lý do: tôi đã biết đó là xe buýt vì xem các frame sau; nhưng reference mở ở frame 79 nên luật "lộ tối thiểu" ở mục 3 được thêm để lần sau quyết định mà không cần nhìn frame sau.

### Ca 2
- Clip / frame / ID: clip_01 / MOT frame 79–113 (CVAT 78–112) / ID 6
- Tình huống: xe con tối màu xuất hiện phía sau xe buýt (ID 5), bị che phần lớn, chỉ thấy nóc và một phần thân, tới frame ~113 mới lộ hẳn.
- Quyết định: mở track từ frame 79, bbox ôm phần nhìn thấy, giữ nguyên ID khi lộ ra.
- Lý do: xe chưa rời khung, chỉ bị che dưới 25 frame ở mức nặng; theo luật giữ ID. Thiếu sót: chưa bật occluded, đã ghi thành luật ở mục 3.

### Ca 3
- Clip / frame / ID: clip_01 / MOT frame 149–151 và 169–172 (CVAT 148–150, 168–171) / ID 4 và ID 8
- Tình huống: xe rời khung ở mép trái (ID 4) và mép phải (ID 8), vài frame cuối chỉ còn một dải mỏng ở rìa.
- Quyết định: giữ bbox mỏng tới khi xe khuất hẳn rồi mới bấm outside.
- Lý do: lúc gán tôi hiểu "outside khi xe rời khung" là khi không còn pixel nào. Reference đóng sớm hơn 3–4 frame. Bổ sung luật đóng track ở mục 5.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Ngưỡng mở track: chỉ mở khi thấy được ít nhất một nửa chiều dài xe hoặc nhận ra loại xe ngay trong frame đó (không dựa vào frame sau). Trước đây guideline chỉ nói "frame đầu tiên xác định được là xe" nên mở quá sớm (ID 5, ID 6).
- Ngưỡng đóng track: bấm outside ngay frame mà phần xe còn trong ảnh dưới ~10% chiều rộng xe; không giữ dải mỏng ở rìa (ID 4, ID 8).
- Occluded: bắt buộc bật khi phần nhìn thấy dưới một nửa xe (ID 6 frame 79–113). Cờ này không đổi bbox nhưng giúp người review và eval hiểu vì sao bbox nhỏ.
- Keyframe: dày hơn (≤ 8 frame) ở đoạn xe tăng tốc/rẽ/bị che; hai chỗ trôi IoU ~0.55 (frame 54 ID 4, frame 84 ID 5) đều nằm ở khoảng keyframe 10–15 frame.
