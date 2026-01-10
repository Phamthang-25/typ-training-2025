## Docker Security: Hardening Lifecycle

### 1. Image Security: Bảo mật từ "Gốc"
- Bảo mật bắt đầu ngay từ khi viết **Dockerfile**, image càng nhẹ, càng ít thành phần thừa thãi thì thì cơ hội khai thác của Hacker càng thấp
#### 1.1. Phân tích rủi ro của tag `latest` và các base image lớn:
- Các rủi ro của tag `latest`: trong Docker tag `latest` mặc định trỏ vào image cuối cùng được đẩy lên mà không được gán tag cụ thể
    - **Tính không xác đinh**: Khi base iamge `latest` được cập nhật bởi nhà phát hành, dẫn đến xung đột về thư viện hoặc tahy đổi về cấu hình môi trường. Gây khó khăn trong việc tái hiện lỗi và debug trong môi trường Production
    - **Đứt gãy chuỗi cung ứng**: Khi dùng base image `latest`, coi như đã đặt niềm tin tuyệt đối vào bên thứ ba. Nếu base image của họ bị lỗi hoặc bị chèn mã độc vào bản cập nhập mới nhất, thì hệ thống CI/CD sẽ tự động kéo images đó về để triển khai ===> Toang cả hệ thống!!!
    - **Mất khả năng truy vết**: Khi hệ thống xảy ra sự cố, việc biết conatiner đang chạy `image:latest` là vô dụng. Không thể biết được mã nguồn nào, tại thời điểm nào đã tạo ra image đó. Việc rollback về phiên bản đã chạy ngon trước đó là bất khả thi
- Các rủi ro tù base image lớn: 
    - **Bề mặt tấn công rộng**: Các image lớn chứa rất nhiều tiện ích mà có thể ứng dụng của mình sẽ không cần tới. Khi đó nếu Hacker thực hiện Command Injection vào ứng dụng thì nó sẽ có cả một kho đồ chơi để khai thác
    - **Mật độ lỗ hổng cao**: Mỗi package trong OS là một nguồn tiềm năng gây ra CVE (Common Vulnerabilities and Exposures)
    - **Nhiễu thông tin bảo mật**: Khi quét những image lớn, các công cụ bảo mật trả về rất nhiều các các lỗ hổng, cảnh báo. Nó khiến cho đội ngủ bảo mật có thể bị bỏ qua các lỗ hổng thực sự nguy hiểm
- Một số các giải pháp:
    - Sử dụng `image:v1.2.3` thay vì `image:latest`
    - Chọn các base image nhẹ như: `alpine`, `slim`, `distroless`, ... thay vì các base image lớn như: `ubuntu`, `node:18`, ...
    - Không sử dụng các tiện ích OS: No Shell, No Package Manager
#### 1.2. Kỹ thuật Multi-stage builds để tối ưu dung lượng và giảm bề mặt tấn công
- ...