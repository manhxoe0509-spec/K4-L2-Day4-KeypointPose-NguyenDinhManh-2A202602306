# Báo cáo Ngày 4 - Keypoint & Pose

Họ tên: Nguyễn Đình Manh   Nhóm: ______   Ngày: 2026-09-16

## 1. Nhãn của tôi

| Chỉ số | Giá trị |
| --- | ---: |
| Số ảnh đã gán | 20 |
| Số skeleton | 29 |
| v=2 / v=1 / v=0 | 358 / 110 / 25 |
| Thời gian trung bình mỗi ảnh | Chưa ghi nhận |

Ba khớp có `%v=1` cao nhất:

1. `left_ear`: 41%
2. `right_ear`: 38%
3. `left_eye` và `right_eye`: 28%

Tai có tỷ lệ bị che cao nhất, phù hợp với việc tai thường bị tóc hoặc góc mặt che. Mắt trái và cổ tay phải cùng ở mức 26%, cho thấy đây là các khớp thường khó quan sát trong một số tư thế. Tỷ lệ v=1 phản ánh mức độ bị che, không tự nó chứng minh vị trí đó khó xác định về mặt giải phẫu.

## 2. Chấm với gold

| Chỉ số | Trước rework | Sau rework |
| --- | ---: | ---: |
| OKS trung bình | 0.9335 | 0.9397 |
| OKS@0.50 | 0.9310 | 1.0000 |
| OKS@0.75 | 0.8966 | 1.0000 |
| Lỗi `dao_trai_phai` | 1 | 0 |
| Lỗi `nham_nguoi` | 0 | 0 |
| Lỗi `xoa_khop_bi_che` | 0 | 0 |

Kết quả trước rework: gold có 29 người, nhãn của tôi ghép được 27 người, thiếu 2 người, không có người thừa. Evaluator ghi nhận 11 lỗi lệch nhẹ, 47 cờ khác gold nhưng vị trí vẫn đúng, và 67 trường hợp gold để v=0 ở khớp đã gán. Sau rework, ghép được đủ 29/29 người, không thiếu/thừa, còn 10 lỗi lệch nhẹ, 53 cờ khác gold nhưng vị trí vẫn đúng, và 76 trường hợp gold để v=0 ở khớp đã gán.

**Tôi đã sửa gì giữa hai lần chạy**

Tôi đã export lại toàn bộ 20 ảnh sau khi rework. Bản export mới có 29 skeleton thay vì 27:

- `train_13.jpg`, người #1 và #2: bổ sung hai skeleton người bị thiếu.
- `train_15.jpg`, người #2: sửa các keypoint trái/phải bị đảo.
- Kết quả sau sửa: không còn skeleton nào cần rework, OKS@0.75 đạt 1.0000.

**Lỗi đảo trái/phải của tôi xảy ra ở ảnh nào?**

Lỗi được evaluator ghi nhận ở lần chạy trước tại `train_15.jpg`, người #2. Đây là lỗi cần ưu tiên vì đảo trái/phải có thể làm model học sai khi dùng augmentation lật ảnh. Sau rework, lỗi này đã được sửa và lần chạy mới không còn finding đảo trái/phải.

## 3. Kiểm chéo

Bạn cùng nhóm: Chưa ghi nhận

Chưa có bảng visibility của bạn cùng nhóm, nên chưa thể tính khớp lệch `%v=1`.

| Khớp | Bạn | Họ | Lệch | Nguyên nhân (guideline hay gán sai?) |
| --- | ---: | ---: | ---: | --- |
| Chưa có dữ liệu | - | - | - | Chưa chạy kiểm chéo |

Luật mới đã bổ sung vào `GUIDELINE_MINI.md` sau khi thống nhất:

- Nếu khớp bị che nhưng phần cơ thể liền kề vẫn cho phép ước lượng vị trí và khớp còn trong khung, giữ chấm tại vị trí ước lượng và chọn `v=1`; chỉ chọn `v=0` khi khớp đã ra ngoài mép ảnh.

## 4. Model

| Chỉ số | yolo26n-pose gốc | Sau fine-tune | Chênh |
| --- | ---: | ---: | ---: |
| pose_mAP50 | 0.8450 | 0.8450 | +0.0000 |
| pose_mAP50-95 | 0.6853 | 0.6853 | +0.0000 |
| pose_precision | 0.9734 | 0.9745 | +0.0011 |
| pose_recall | 0.8462 | 0.8462 | +0.0000 |
| box_mAP50-95 | 0.8119 | 0.8054 | -0.0065 |

### Trả lời năm câu hỏi ở cuối notebook

1. `pose_mAP50-95` không thay đổi: chênh lệch là `0.0000`. Với chỉ 20 ảnh và 27 skeleton ở lần fine-tune trước, tập dữ liệu nhỏ chưa tạo ra cải thiện đo được trên 10 ảnh test. Kết quả này không chứng minh fine-tune vô ích; nó cho thấy dữ liệu hiện tại chưa đủ để thay đổi điểm số ngoài dao động đo.

2. Sau fine-tune, `box_mAP50-95` là `0.8054`, còn `pose_mAP50-95` là `0.6853`, chênh `0.1201`. Model tìm người dễ hơn tìm chính xác các khớp, vì box chỉ cần bao quanh người còn pose phải đặt đúng nhiều keypoint và chịu ảnh hưởng của che khuất, đảo trái/phải.

3. Ở lần chạy model trước, một lỗi rõ trong tập đối chiếu là `train_13`: model dự đoán `3` người trong khi nhãn có `1` người, tương ứng lỗi nhầm người/thừa dự đoán. Đây là lỗi về số người, gần với nhóm “nhầm người” trong bốn loại lỗi; cần xem overlay để phân biệt dự đoán thừa thuộc cơ thể nào.

4. OKS thấp nhất giữa model và nhãn của tôi là `train_15`, người có OKS `0.536`. Gold evaluator cũng đánh dấu `train_15.jpg`, người #2 là lỗi đảo trái/phải, nên bằng chứng hiện có nghiêng về việc nhãn/model bất đồng ở trái-phải; cần xác nhận bằng ảnh overlay và gold trước khi kết luận ai đúng.

5. Có dấu hiệu trùng nhau: `train_13` là ảnh nhãn tệ nhất vì thiếu hai người theo gold, đồng thời model cũng lệch số người (`model 3 / bạn 1`). Điều này cho thấy ảnh có cấu trúc nhiều người hoặc khó tách người, nên cả annotation và dự đoán đều dễ sai ở bước ghép người.

## 5. Một rule evidence bạn đã dùng

Ở `train_13`, người #1, hai mắt cá chân nằm ngoài mép ảnh theo annotation COCO; sau khi chuyển đổi, `left_ankle` và `right_ankle` được ghi `v=0` với tọa độ bằng 0. Đây là bằng chứng hình học để phân biệt Outside với Occluded: khớp không còn nằm trong khung nên không thể đặt chấm ước lượng đáng tin cậy trong ảnh. Vì vậy chọn `v=0`, không chọn `v=1`.
