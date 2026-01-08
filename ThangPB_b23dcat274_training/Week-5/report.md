## Báo cáo Ansible

### Phần 1: Lý thuyết
- **Ansible** là một công cụ tự động hóa mã nguồn mở, dùng để quản lý cấu hình, triển khai phần mềm và tự động hóa các tác vụ
    - Cơ chế hoạt động: Ansible hoạt động theo mô hình Push. Viết code ở máy điều khiển (control node), sau đó Ansible đẩy các thiêt lập đó tới các máy đích (managed nodes) qua SSH
- Mục đích sử dụng của Ansible:
    - **Configuration Management**: thiết lập đồng nhất cấu hình cho rất nhiều server cùng lúc
    - **Application Deployment**: tự động hóa việc đưa code từ Git lên server, cài môi trường và chạy ứng dụng
    - **Provisioning**: khởi tạo tài nguyên trên các nền tảng đám mây (AWS, Azure, GCP, ...)
    - **Orchestration**: điều phối các tác vụ phức tạp theo trình tự giữa nhiều server khác nhau
- Cách cài đặt Ansible (Ubuntu):
    ```bash
    sudo apt update
    sudo apt install -y software-properties-common
    sudo add-apt-repository --yes --update ppa:ansible/ansible
    sudo apt install -y ansible
    ```
- Cách thành phần cốt lõi của Ansible:
    - **Ansible Configuration** (`ansible.cfg`): là tệp cấu hình trung tâm của Ansible. Nó quy định các thông số mặc định để Ansible biết phải hoạt động như thế nào
        - Vị trí: Thường nằm trong thư mục dự án hoặc `/etc/ansible/ansible.cfg`
        - Chức năng: định nghĩa đường dẫn file Inventory, user thực thi lệnh, các thiết lập bảo mật, và tối ưu hóa tốc độ kết nối
    - **Inventory**: là tệp chứa danh sách máy chủ muốn qunr lý
        - có thể chia nhóm các máy chủ (`[web]`, `[db]`)
        - cho phép đặt các biến riêng cho từng máy hoặc từng nhóm máy
    - **Ad-Hoc commands**: Đây là các lệnh chạy trực tiếp từ terminal mà không cần viết Playbook
        - **Đặc điểm**: Nhanh, dùng cho các tác vụ tạm thời, thực hiện một lần
        - Ví dụ: `ansible all -m ping`
    - **Playbook**: là file (YAML) chứa các kịch bản tự động hóa được sắp xếp theo trình tự
        - Playbook giúp lưu trữ quy trình công việc để có thể tái sử dụng
    - **Task**: là một đơn vị hành động nhỏ nhất nằm bên trong Playbook
        - Mỗi Task thường chỉ thực hiện một việc duy nhất
        - Các Task sẽ được thực thi tuần tự từ trên xuống dưới
    - **Module**: là các chương trình thực hiện công việc thực tế trên máy đích. Ansible có hàng ngàn module có sẵn
    - **Role**: là cách tổ chức và đóng gói các Playbook theo một cấu trúc thư mục tiêu chuẩn
        - Khi dự án lớn, việc viết tất cả vào 1 file Playbook sẽ rất rối
        - Role giúp tách riêng: Biến (vars), Tác vụ (tasks), File cấu hình (files), và Template (templates)
- Cách viết các file `*.yml` trong ansible:
    - Ansible hoạt động theo nguyên tắc **Declarative**: mô tả trạng thái mong muốn
    - VD: so sánh việc cài đặt Docker
        - Với shell: `apt install -y docker.io`
        - Với ansible:
            ```yml
            - name: Docker is installed
              apt:
                name: docker.io
                state: present
            ```
    - Định dạng tổng quát:
    ```yml
    - name: Đảm bảo điều gì đó
      module:
        name: đối tượng
        state: trạng thái mong muốn
    ```
    - Các module cốt lõi:
        - Quản lý package: `apt`
        - Quản lý service: `service`
        - Copy file từ control node sang server: `copy`
        - Quản lý file / thư mục: `file`
        - Quản lý user: `user
        - Sửa 1 dòng trong file config: `lineinfile`
        - Chạy container: `docker_container`
        - Download file: `get_url`
        - In biến / debug: `debug`

### Phần 2: Thực hành
- **Đề bài**: Deploy 1 project bất ky sử dụng Ansible. Images được đẩy trươc lên registry và viết ansible task để pull về và chạy
- Dùng máy Ubuntu cài Ansible làm Control node 
- Sử dụng 3 con instance tương ứng: FE - BE - DB
- Cấu trúc thư mục:
```
ansible_lab/
├── ansible.cfg                 # File cài đặt mặc định của ansible
├── inventory/
│   └── hosts.ini               # Danh sách các server
├── playbooks/
│   └── site.yml                # Kịch bản chính ansible thực hiện
└── roles/
    ├── common/
    │   └── tasks/main.yml      # Task: setup môi trường cho server
    ├── fe/
    │   └── tasks/main.yml      # Task: triển khai Frontend
    ├── be/
    │   └── tasks/main.yml      # Task: triển khai Backend
    └── db/
        └── tasks/main.yml      # Task: triển khai Database
```
- Luồng chạy:
    - Ansible đọc `ansible.cfg` và `hosts.ini`
    - SSH vào 3 instance
    - Chạy `common/` cho cả 3 máy
    - Chạy lần lượt `DB -> BE -> FE` theo thứ tự trong `site.yml`