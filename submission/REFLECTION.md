# Reflection

Anti-pattern mình quan tâm nhất là small-file problem: pipeline streaming ghi quá
nhiều file nhỏ vào lakehouse sau mỗi micro-batch. Dữ liệu vẫn đúng nhưng truy vấn
chậm vì engine phải mở và lập kế hoạch trên rất nhiều file; chi phí object storage
cũng tăng do nhiều request và metadata. Khi bảng lớn dần, vấn đề trở thành chi phí
vận hành và độ trễ, không chỉ là chuyện sắp xếp file.

Cách phòng tránh là đặt ngưỡng kích thước file và lịch compaction rõ ràng, theo dõi
số file trên mỗi partition, thời gian lập kế hoạch và tỷ lệ file bị skip. Job
clustering/Z-order có thể cải thiện min/max statistics cho các truy vấn thường dùng.
Compaction cần chạy cùng chính sách snapshot expiry và orphan cleanup; nếu chỉ
rewrite file mà không dọn dữ liệu không còn tham chiếu, chi phí lưu trữ vẫn tăng.
Các job này phải giữ retention đủ dài cho reader và time travel, đồng thời đo
trước/sau để phát hiện khi maintenance gây thêm chi phí hoặc làm mất khả năng
khôi phục cần thiết.
