# Văn phòng BA AI

Dashboard 3D theo dõi một đội AI agent hỗ trợ công việc Business Analyst. Mỗi phòng là một agent (dự kiến chạy bằng workflow n8n). Mỗi yêu cầu đi lần lượt qua từng agent và **dừng ở cổng duyệt sau mỗi bước** cho đến khi bạn xác nhận.

> **Trạng thái:** bản thử. Văn phòng bắt đầu trống, chưa có hồ sơ nào. Agent trả kết quả mẫu, hồ sơ lưu trong `localStorage` của trình duyệt. Chưa nối n8n.

## Mở văn phòng

- **Trên máy:** mở `index.html` bằng trình duyệt (cần internet để tải Three.js và font).
- **Trên web:** bật GitHub Pages cho repo (Settings → Pages → Deploy from branch → `main` / root). Repo private cần gói GitHub trả phí mới dùng được Pages.

## Đội agent và flow

Yêu cầu mới: **Quân → Mai → Bảo → Vy → Tú → Đạt → Linh → Lưu trữ**
Change request: **Quân → Khoa →** quay lại bước bị ảnh hưởng sớm nhất trong flow trên.

| Bước | Agent | Tác vụ | Tại cổng duyệt, bạn kiểm tra |
|---|---|---|---|
| 1 | Quân | Tiếp nhận & phân loại | Đúng loại (mới/CR), đúng phạm vi, các bước cần chạy hợp lý |
| 2 | Mai | Bóc tách yêu cầu | Không sót ý, mỗi yêu cầu có mã REQ, giả định và câu hỏi mở ghi riêng |
| 3 | Bảo | Soạn BRD / FRS | Đủ mọi REQ, quy tắc có nguồn, không tự thêm quy định |
| 4 | Vy | Vẽ luồng BPMN | Actor, điểm quyết định, luồng ngoại lệ khớp FRS |
| 5 | Tú | Viết user story & AC | Mỗi REQ có story, AC phủ cả luồng ngoại lệ |
| 6 | Đạt | Mapping dữ liệu | Đủ field, quy tắc chuyển đổi rõ, field thiếu được đánh dấu |
| 7 | Linh | Rà soát tổng thể | Ma trận truy vết REQ → FRS → BPMN → story → mapping không còn ô trống |
| CR | Khoa | Phân tích tác động | Đủ thành phần bị ảnh hưởng, chọn đúng bước quay lại |

Tại mỗi cổng: **Duyệt** (sang bước sau) · **Yêu cầu sửa** (agent làm lại) · **Trả về bước trước**.

## Cách dùng

**Gửi việc** (khu đầu trang): tiêu đề, agent bắt đầu, nguồn, nội dung hoặc tài liệu cần xử lý: dán trực tiếp, hoặc tải / kéo thả file Word (`.docx`), Excel (`.xlsx`, `.xls`), `.csv`, `.txt`, `.md` (tối đa 10 MB/tệp, chọn nhiều tệp được). Nội dung file được chuyển thành văn bản, giữ heading, danh sách và bảng; mỗi sheet Excel thành một bảng.

| Giao cho | Phạm vi | Kết quả |
|---|---|---|
| Quân | Yêu cầu mới | Chạy đủ quy trình; bỏ tick được bước không cần |
| Quân | Change request | Quân → Khoa → quay lại bước bị ảnh hưởng |
| Khoa | Change request | Bỏ qua Quân, Khoa phân tích rồi quay lại bước bị ảnh hưởng |
| Bất kỳ agent khác | Chỉ agent này | Agent đó xử lý một mình, chờ duyệt rồi vào Lưu trữ |
| Bất kỳ agent khác | Chạy tiếp các bước sau | Bắt đầu từ agent đó, chạy tiếp các bước được tick |

Mọi trường hợp đều dừng ở cổng duyệt sau mỗi bước.

Change Request bắt buộc gắn với một Requirement đã có: khi chọn Change Request (hoặc giao cho Khoa), trang liệt kê các Requirement để chọn Requirement gốc. Chưa có Requirement nào thì chưa tạo được Change Request.

**Giao thêm việc cho hồ sơ đang chạy** (trong khung của từng agent):

| Hồ sơ đang ở đâu | Kết quả |
|---|---|
| Chưa tới bước của agent | Chỉ thị được giữ, áp dụng khi tới bước đó |
| Đang ở đúng bước đó | Agent làm lại ngay với chỉ thị, rồi chờ duyệt |
| Đã qua bước đó / đã chốt | Mở lại bước đó; các bước sau chạy lại, mỗi bước chờ duyệt |
| Bước đang bị bỏ qua | Thêm lại bước vào flow |
| Việc lẻ | Agent làm riêng, chờ duyệt rồi vào Lưu trữ |

## Kế hoạch nối n8n

- Mỗi agent là một workflow gắn tag `ba-office`, đặt tên theo bước: `BA-01 · Tiếp nhận & phân loại` … `BA-07 · Rà soát tổng thể`, `BA-CR · Phân tích tác động`.
- Mỗi workflow kết thúc bằng node **Wait** (cổng duyệt). Output chỉ chuyển sang workflow kế tiếp khi bạn duyệt.
- Trạng thái execution → hiển thị:

| n8n | Văn phòng |
|---|---|
| `running` | Running |
| `waiting` | Awaiting Approval |
| `success` | Approved (hồ sơ xong cả flow: Closed) |
| `error`, `crashed` | Error |

- Việc còn lại: nối connector n8n (MCP) để trang đọc execution thật và để nút Gửi / Duyệt gọi workflow thật.

## Lưu ý dữ liệu

Không dán thông tin khách hàng hoặc tài liệu nội bộ của tổ chức vào bản thử này, hay vào n8n/LLM bên ngoài, trừ khi chính sách nơi làm việc cho phép.
