# Báo cáo Day 13 — Dương Quang Chung, 2A202603217

## 1. Kết luận trước

Task `D13-2A202603217`, CVAT local 2.74.1, Windows, Chrome. Bài thực hiện với AI hỗ trợ theo xác nhận của học viên rằng coach đã cho phép. Đây là nhãn AI hỗ trợ và đã rà soát camera, không phải toàn bộ cuboid được học viên vẽ tay. Có 80 cuboid dạng Shape trên 10 frame; frame ít nhất là s02f37 (5 hộp). Phiếu tự kiểm có 3/80 hộp bị cờ. Không có nhãn chuẩn để xác nhận recall hay IoU. Phần còn yếu: kích thước suy luận từ bề mặt nhìn thấy, yaw người/xe hai bánh và vật bị che.

CVAT: kéo trái để xoay, kéo phải để dời, con lăn để zoom. Cuboid dùng Shape, kiểm Top/Side/Front, chỉnh yaw quanh z; bật Cuboid orientation và chạy preview để đối chiếu camera. Nhãn được nhập qua API và xuất bằng script gốc. Server cũ cần 16 số: 9 tham số vị trí/quay/scale và 7 số 0 theo định dạng của giao diện bản này. Không sửa công cụ được bảo vệ.

| Frame | Cuboid |
|---|---:|
| s01f19 | 8 |
| s01f57 | 8 |
| s02f13 | 10 |
| s02f37 | 5 |
| s03f03 | 10 |
| s03f19 | 6 |
| s04f02 | 7 |
| s04f17 | 13 |
| s05f00 | 7 |
| s05f32 | 6 |

## 2. Đọc frame

Hệ VF: +x sang trái, +y ra sau, +z lên; phía trước là −y. Ảnh trong ZIP là screenshot thật từ CVAT. Hai ảnh mang hậu tố top hiện còn là góc nhìn xiên, chưa đạt ảnh BEV vuông góc có mũi tên đánh dấu như hướng dẫn; cần bổ sung khi học viên rà soát.

| Frame | Trục và phía trước | Mặt đường | Vật và tầm quan sát | Hai bên/phía sau | Cụm chưa rõ |
|---|---|---|---|---|---|
| s01f19 | −y về phía xe buýt; +x bên trái xe thu, +y ngược hướng đó | Đường gần phẳng nhưng z thay đổi theo vị trí, khoảng −1,7 đến −2,0 m | Xe con, Bus, người trên vỉa hè; đã gán xe ở khoảng 40 m | Xe con phía sau ở cam05/08/09 | Cụm cây và người thưa ở lề |
| s01f57 | −y theo hàng xe phía trước; cùng quy ước +x/+y | Đường gần phẳng, khoảng −1,7 m | Nhiều Car đang xếp hàng, xe phía xa bị che | Có xe sát trái và xe phía sau | Người và xe hai bánh sát vỉa hè khó tách |

Ảnh: t1-s01f19-3d.jpg, t1-s01f19-top.jpg, t1-s01f57-3d.jpg, t1-s01f57-top.jpg.
Camera cho biết màu xe, kiểu thân xe và đầu/đuôi xe. LiDAR cung cấp tọa độ theo mét, hình học bề mặt và độ cao tương đối với mặt đường. Các nhận xét này do AI hỗ trợ quan sát.

| Frame lướt | Phía trước | Quan sát |
|---|---|---|
| s02f13 | −y | Hai chiều xe ngăn bởi hàng rào; nhiều bề mặt bị che |
| s02f37 | −y | Xe hai bánh sát lề phải, nhiều vật xa |
| s03f03 | −y | Xe xếp hàng, tường hai bên |
| s03f19 | −y | Giao lộ, có xe hai bánh cắt ngang |
| s04f02 | −y | Minivan bên phải, xe phía sau |
| s04f17 | −y | Nhiều làn xe và xe đối diện |
| s05f00 | −y | Truck ở sau xe thu, xe con và van |
| s05f32 | −y | Xe con phía trước phải và phía sau |

## 3. Ba cuboid mẫu

| Vật | Frame, idx | Class | Kích thước l × w × h (m) | Kiểm tra |
|---|---|---|---|---|
| Car | s01f19, 1 | Car | 4.200 × 1.800 × 1.500 | Đã đối chiếu hình học và camera; hướng người còn cần xác nhận |
| Bus | s01f19, 2 | Bus | 12.300 × 2.650 × 3.120 | Đã đối chiếu hình học và camera; hướng người còn cần xác nhận |
| Pedestrian | s01f19, 8 | Pedestrian | 0.600 × 0.600 × 1.700 | Đã đối chiếu hình học và camera; hướng người còn cần xác nhận |

Ảnh: t2-a-car.jpg, t2-b-bus-truck.jpg, t2-c-ped-cyclist.jpg. Screenshot và ID giao diện có thể khác idx bảng A; đối chiếu bằng frame, class và vị trí. Vòng hiệu chuẩn chung chưa thực hiện, không khai đã tham gia. Kiểm chéo với bạn học được ghi ở mục 5. Bỏ qua cây, hàng rào, xe hai bánh dựng không có người, và các cụm không đủ cơ sở (LOW_VISIBILITY/UNCERTAIN_CLASS).

## 4. Bốn hiện tượng chất lượng

| Hiện tượng | Frame / ảnh / nguồn | Quan sát và giả thuyết | Hệ quả |
|---|---|---|---|
| Bóng che | s01f19 / t4-1-bong-che.jpg / task cá nhân | Xe buýt có bề mặt gần nhiều điểm, phía khuất ít điểm; giả thuyết do che khuất | Phần không quan sát không đồng nghĩa đường trống; bật occluded cho vật bị che, chỉ suy luận khi đủ cơ sở |
| Thưa ở xa | s01f57 / t4-2-thua-o-xa.jpg / task cá nhân | Các vật phía xa thưa hơn trên tổng quan; giả thuyết do khoảng cách và góc lấy mẫu | Áp dụng ngưỡng thấy 15–20% thể tích, không đủ thì bỏ qua |
| Phản xạ thấp | s02f13 / t4-3-phan-xa-thap.jpg / task cá nhân | Xe tối màu có vùng bề mặt thiếu điểm; phản xạ thấp chỉ là giả thuyết, có thể còn do góc nhìn và che khuất | Không kéo hộp theo mọi điểm rời; cần đối chiếu camera |
| Điểm lạ | s02f13 / t4-4-diem-la.jpg / task cá nhân | Ảnh nghiêng cho thấy điểm rời quanh hộp nhưng chưa xác nhận được một cụm sai hình học; chưa đủ bằng chứng kết luận nhiễu | Không gán các điểm rời thành vật khi chưa đối chiếu; mục này còn thiếu một ví dụ được xác nhận |

## 5. Số từ cuboid của tôi

Số dưới đây được trích tự động từ bảng A/phiếu tự kiểm do sync sinh, không sửa tay hai CSV. Các nhận xét và phép tính trong báo cáo có AI hỗ trợ.

| Cuboid | z (m) | h (m) | z đáy (m) | dist_m (m) | r 3D (m) | yaw (°) |
|---|---:|---:|---:|---:|---:|---:|
| s01f19 #1 Car | -1.050 | 1.500 | -1.800 | 4.78 | 4.897 | 270.0 |
| s01f19 #2 Bus | -0.460 | 3.120 | -2.020 | 23.50 | 23.501 | 270.0 |
| s01f19 #8 Pedestrian | -0.120 | 1.700 | -0.970 | 15.80 | 15.803 | 0.0 |

Gần nhất: s01f57 #1 Car, dist_m = √(x²+y²) = 3.48 m; r = √(x²+y²+z²) = 3.613 m.

Xa nhất: s05f32 #6 Car, dist_m = √(x²+y²) = 45.44 m; r = √(x²+y²+z²) = 45.445 m.

Xe con mẫu 4.200 × 1.800 × 1.500 m nằm trong khoảng sedan bảng 4.3. Người mẫu 0.600 × 0.600 × 1.700 m trong khoảng người lớn. Đáy cần so mặt đường cục bộ: xe buýt ở −2,020 m phù hợp vùng đường thấp hơn, không ép mọi hộp về −1,7 m.

Yaw cùng chiều: s01f19 #1 Car = 270.0°; ngược chiều: s01f19 #5 Car = 90.0°. Với xe cắt ngang, yaw vẫn là ước lượng cần kiểm thêm.

| Cuboid | dist_m (m) | points_in |
|---|---:|---:|
| s01f57 #1 Car | 3.5 | 3066 |
| s05f32 #2 Car | 10.4 | 1169 |
| s02f13 #9 Car | 19.1 | 366 |
| s05f32 #6 Car | 45.4 | 60 |

Xu hướng chung là xa hơn thường ít điểm hơn, nhưng không đơn điệu ở mọi cặp vì che khuất, diện tích nhìn thấy và góc chiếu. Slide tham chiếu trung vị xe con giảm từ khoảng 1968 điểm ở 0–10 m xuống 72 ở 30–40 m; bốn mẫu trên không đại diện cho phân bố toàn bộ dữ liệu.

Sau sync đã sửa một đề xuất xe bị trùng phía sau thành hộp của xe xa khác, điều chỉnh đáy người theo nền cục bộ, và sửa chiều rộng xe bị đặt sai khi cập nhật batch. Đã Save qua API và chạy lại sync. Cờ còn lại:

- s03f19 #1 Cyclist: 113 điểm sát ngay ngoài mép (quá chật, hay vật bên cạnh?). Giữ để rà soát: có điểm sát biên, có thể thuộc vật bên cạnh; chưa đủ cơ sở nới hộp chỉ nhằm xóa cờ.

- s03f19 #2 Cyclist: 141 điểm sát ngay ngoài mép (quá chật, hay vật bên cạnh?). Giữ để rà soát: có điểm sát biên, có thể thuộc vật bên cạnh; chưa đủ cơ sở nới hộp chỉ nhằm xóa cờ.

- s03f19 #4 Cyclist: 279 điểm sát ngay ngoài mép (quá chật, hay vật bên cạnh?). Giữ để rà soát: có điểm sát biên, có thể thuộc vật bên cạnh; chưa đủ cơ sở nới hộp chỉ nhằm xóa cờ.

Kiểm chéo: người kiểm có mã số kết thúc bằng **03061** (học viên cung cấp 5 số cuối, chưa có MSSV đầy đủ). Theo xác nhận của học viên, đã đối chiếu với bạn và các số được kiểm đều khớp, không có sai lệch cần sửa. Chưa ghi cụ thể hai số đã đối chiếu vì học viên chưa cung cấp danh sách đó; không tự suy đoán nội dung kiểm.

## 6. Ý nghĩa của nhãn, AI và điều còn chưa chắc

Nhãn 3D dùng mét để model học khoảng cách, kích thước và vị trí vật trong không gian thực; pixel thay đổi theo phối cảnh và camera. Quy tắc riêng cho vật thưa điểm giúp tránh biến phần không quan sát được thành nhãn chắc chắn và làm sai tín hiệu học.

Công cụ: Codex, Python/scipy và CVAT local; sử dụng AI label theo xác nhận quyền của coach do học viên cung cấp.
Câu hỏi chính: phân tích cụm LiDAR, đối chiếu camera, tạo cuboid và hoàn thiện xuất/kiểm bài nộp.
Cách dùng: AI tạo đề xuất, loại các cụm nền rõ ràng, nhập nhãn, trích số vào báo cáo; không dùng nhãn chuẩn, không tuyên bố nhãn đạt IoU/recall chuẩn. Cần học viên review cuối về class, yaw, occluded và kích thước suy luận.

Các giới hạn: chưa xác nhận ví dụ điểm lạ; ảnh top còn xiên và chưa đánh dấu trục; chưa tham gia vòng hiệu chuẩn chung; mục kiểm chéo đã có xác nhận khớp từ học viên nhưng còn thiếu MSSV đầy đủ và hai số cụ thể đã kiểm; yaw người và cyclist còn chưa chắc; xe tải chỉ thấy chủ yếu phần đầu nên chiều dài suy luận còn yếu. Checker chỉ kiểm cấu trúc và tính nhất quán file, không chứng minh các nội dung này đã đạt rubric. ZIP ảnh giữ local cho LMS, không đưa ảnh/dữ liệu lên GitHub. Bài được push sau deadline mặc định của ngày hôm trước; xử lý nộp muộn theo coach.
