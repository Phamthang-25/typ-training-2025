## Báo cáo CI/CD với GitLab

### 1. Mở đầu về CI/CD
- **Khái quát về CI/CD**
    - Trong thực tế, CI/CD là việc thay thế các bước thủ công, lặp đi lặp lại bằng một hệ thống tự động hóa hoàn toàn mà các lập trình viên chỉ cần tương tác qua Git.
    - Các lợi ích của CI/CD:
        - Tốc độ phát hành nhanh
        - Độ tin cậy và chất lượng được đảm bảo
        - Khả năng hồi phụ nhanh, giẩm thiểu downtime
        - Tối ưu hóa nguồn lực và giảm chi phí vận hành 
- **Tạo repo Gitlab và đẩy source code lên**
    - Tạo repo ở local và thao tác để push lên Gitlab repo tương tự như làm việc với Github

    <img src="./images_4/a1.png" alt="" width="1000" height="500">

- **Tìm hiểu cơ bản về file `.gitlab-ci.yml`**
    - `.gitlab-ci.yml` là một file cấu hình được đặt tại thư mục gốc của dự án, và hướng dẫn cho GitLab Runner những công việc cần làm khi có thay đổi trong mã nguồn
    - File tổ chức công việc thành 2 phần chính: **Stages** và **Jobs**
    - **Stages**: Là các giai đoạn logic của pipeline (build, test, ...)
        - Các Stages được thực thi tuần tự (theo thứ tự từ trên xuống dưới). Một Stage sẽ không bắt đầu cho đến khi tất cả các Jobs trong Stage trước đó hoàn thành thành công
        - Định nghĩa các stages:
        ```yml
        stages:
            - build
            - test
            - deploy
            - ...
        ```
    - **Jobs**: Là các tác vụ cụ thể sẽ được thực hiện
        - Các Jobs trong cùng một Stage được thực thi song song để tăng tốc độ pipeline
        - Khai báo cái Jobs:
        ```yml
        stages:
            - build
            - ...

        Job_name: 
            stage: build  
            script:        
                - echo "Bắt đầu biên dịch..."
                - mvn clean package
        ```
    - Một vài từ khóa quan trọng:
        - **stage**: Chỉ định Job thuộc về Stage nào đã được định nghĩa
        - **script**: Là danh sách các lệnh shell sẽ được Runner thực thi
        - **image**: Chỉ định môi trường Docker container mà Job sẽ chạy bên trong
        - **only / except**: Điều kiện để Job được chạy. Xác định Job chỉ chạy trên branch/tag/event nào
        - **tags**: Yêu cầu Runner có tags cụ thể để thực thi Job này
        - **artifacts**: Các file hoặc thư mục được tạo ra bởi Job, được lưu lại để tải xuống hoặc chuyển sang các Jobs/Stages tiếp theo
- **Cài đặt Gitlab Runner**
    - Sử dụng 1 con EC2 instance làm runner
    - Đã cài đặt và đăng kí thành công:

    <img src="./images_4/a2.png" alt="" width="900" height="300">

    - Thử với vài stage đơn giản:
    ```yml
    stages:
        - build
        - test

    build:
        stage: build
        script:
            - whoami
            - pwd
            - echo "hello CI/CD"
        tags:
            - lab-runner

    test:
        stage: test
        script:
            - echo "start test ...."
            - echo "final test ..."
        tags:
            - lab-runner
    ```
    - Đã chạy lần đầu thành công:

    <img src="./images_4/a3.png" alt="" width="900" height="300">

    - Check thử log khi chạy:

    <img src="./images_4/a4.png" alt="" width="900" height="350">

    <img src="./images_4/a5.png" alt="" width="900" height="350">

- **Sử dụng variable để quản lý thông tin (username, password, ...)**

### 2. Thực hành triên khai với Container
- Tạo luông tự động khi push code lên theo tag
- Thực hiện tự động build Docker image
- Thêm stage scan images (Trivy), để kiểm soát chất lượng image
- Push image lên registry sau khi pass stage scan
- Thiếp lập môi trường Staging để deploy

### 3. Hay ho hơn một chút!!!
- Thiết lập deploy thủ công lên Production
- Thiết lập branch rule
    - develop -> deploy lên Staging
    - main -> deploy lên Production
- Tìm hiểu một chút về Scan bảo mật trong CI/CD pipeline
- Thiết lập thông báo và xuát báo cáo về cho chủ nhân