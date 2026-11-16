# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 3
- Lượt gán đủ **80/80**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 205 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 40 | 205 | 0.815 | 93.2% | 42.4% | 88.3% | 43/251 | 0 | CHƯA ĐẠT |
| 161|165 | 13 | 54 | 0.760 | 90.7% | 46.3% | 77.8% | 9/77 | 0 | CHƯA ĐẠT |
| 161|168 | 13 | 85 | 0.799 | 94.1% | 44.7% | 90.6% | 15/70 | 0 | CHƯA ĐẠT |
| 165|168 | 14 | 66 | 0.881 | 93.9% | 36.4% | 93.9% | 19/104 | 0 | CHƯA ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 596 ({'bo_sot_thua': 294, 'co_can_review': 47, 'iou_thap': 109, 'sai_thuoc_tinh': 126, 'sai_lop': 14, 'bat_dong_ignore': 6}), trên 39 ảnh cần calibration.
