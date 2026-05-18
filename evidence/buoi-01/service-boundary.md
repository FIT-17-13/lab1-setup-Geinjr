Service Boundary của nhóm AI Vision

1. Thông tin nhóm
   Tên nhóm: Nhóm 11
   Lớp: CNTT17-13
   Thành viên: Đỗ Trung Kiên , Lưu Thế Hưng
   Service nhóm phụ trách: AI Vision Analysis Service
   Sản phẩm tổng thể của lớp: Hệ thống giám sát và phân tích thông minh (Product B)

2. Actor
   Hệ thống Camera/IoT: Tự động gửi luồng dữ liệu hình ảnh về để phân tích.
   Người dùng cuối (User): Tải ảnh trực tiếp lên để kiểm tra đối tượng.
   Hệ thống quản trị (Admin): Theo dõi hiệu suất của model AI và cấu hình các tham số nhận diện.

3. System Boundary
   Phần nhóm kiểm soát:
   AI Model (YOLO/CNN): Bộ não chịu trách nhiệm nhận diện vật thể.
   Vision API: Cung cấp các cổng giao tiếp để bên ngoài gửi ảnh vào.
   Processing Logic: Tiền xử lý ảnh (resize, chuẩn hóa) trước khi đưa vào AI.
   Phần nhóm chỉ tích hợp:
   Image Storage: Nơi lưu trữ file ảnh vật lý (S3 hoặc MinIO).
   Auth Service: Hệ thống xác thực người dùng chung của cả lớp.
4. Service Boundary
   Trách nhiệm: Nhận dạng vật thể, phân loại nhãn (Labeling), xác định vị trí (Bounding Box) và trả về kết quả dưới dạng JSON.
   KHÔNG làm gì: Không chịu trách nhiệm hiển thị giao diện người dùng (UI), không xử lý việc lưu trữ lâu dài các file ảnh gốc.

5. Input / Output
   Input
   {
   "camera_id": "cam-gate-01",
   "image_url": "http://example.com/frame.jpg",
   "timestamp": "2026-05-02T09:10:00"
   }
   Output
   {
   "detected": true,
   "object": "person",
   "confidence": 0.91,
   "risk_level": "medium"
   }
6. API dự kiến
   | Method | Endpoint | Mục đích |
   | :--- | :--- | :--- |
   | GET | `/health` | Kiểm tra trạng thái hoạt động của AI Service |
   | POST | `/v1/vision/analyze` | Gửi ảnh và nhận kết quả phân tích AI |
   | GET | `/v1/vision/models` | Lấy danh sách các phiên bản model AI đang hỗ trợ |

7. Phụ thuộc service khác
   Gọi đến: Image Storage Service để lưu lại các ảnh đã được vẽ khung nhận diện (annotated images).
   Bị gọi bởi: API Gateway hoặc Core Business Service khi cần thông tin phân tích để ra quyết định.

8. Sơ đồ minh họa

```mermaid
flowchart TD
    %% Định nghĩa các Actor (Vòng tròn)
    CAM((Ref: Camera / IoT<br>Actor))
    USR((Ref: User<br>Actor))
    ADM((Ref: Admin<br>Actor))

    %% Định nghĩa các bên tích hợp ngoài
    GW[API Gateway<br>Caller Service]
    AUTH[Auth Service<br>Token validation]
    STORE[(Image Storage<br>S3 / MinIO)]

    %% Ranh giới hệ thống của nhóm
    subgraph BOUNDARY ["System Boundary · AI Vision Service"]
        API["Vision API<br>• /v1/vision/analyze<br>• /health<br>• /v1/vision/models"]
        PRE[Pre-processing<br>Resize · Normalize]
        MODEL[AI Model<br>YOLO / CNN Detection]
        POST[Post-processing<br>JSON Result Builder]

        API -->|Input: image| PRE
        PRE --> MODEL
        MODEL --> POST
    end

    %% Luồng dữ liệu kích hoạt hệ thống
    CAM -->|Input: image stream| API
    USR -->|Input: upload image| API
    ADM -->|Input: config / monitor| API

    %% Luồng xác thực hỗ trợ
    AUTH -.->|validate token| API

    %% Luồng kết quả đầu ra
    POST -->|Output: JSON detections| GW
    POST -.->|Output: annotated image| STORE

```
