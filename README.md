Chào bạn, đây là phiên bản README đã được tối ưu hóa, tập trung vào sự rõ ràng, mạch lạc và giúp người học dễ dàng điều hướng kho lưu trữ.

Phiên bản này được viết dựa trên giả định rằng các chi tiết của mỗi bài lab (như "Yêu cầu chi tiết", "Kết quả mong đợi") sẽ được đặt trong các file `.md` riêng biệt (ví dụ: `labs/part-1/lab-1a.md`), và README này đóng vai trò là **trang chủ và mục lục** chính.

-----

# 📘 Hướng dẫn Lab về Khả năng Mở rộng của vSphere

Chào mừng bạn đến với Hướng dẫn Lab về Khả năng Mở rộng của vSphere\! Đây là một bộ sưu tập các bài thực hành (hands-on labs) được thiết kế để giúp bạn xây dựng, quản lý và tối ưu hóa một môi trường VMware vSphere từ cơ bản đến nâng cao.

Kho lưu trữ này tập trung vào các kịch bản thực tế, giúp bạn hiểu rõ về kiến trúc, hiệu suất, tính sẵn sàng cao và khả năng mở rộng của nền tảng ảo hóa hàng đầu thế giới.

## 🎯 Đối tượng hướng tới

Kho lưu trữ này lý tưởng cho:

  * **Quản trị viên Hệ thống (SysAdmins)** muốn nâng cao kỹ năng vSphere.
  * **Kỹ sư Giải pháp (Solution Architects)** cần thiết kế các môi trường có khả năng mở rộng.
  * **Sinh viên và Người mới bắt đầu** muốn có kinh nghiệm thực tế về ảo hóa.

-----

## 🏛️ Sơ đồ Kiến trúc vSphere

Để hiểu rõ các thành phần cốt lõi và cách chúng tương tác với nhau, bạn có thể xem sơ đồ kiến trúc tổng quan dưới đây.

\<details\>
\<summary\>Nhấn vào đây để xem Sơ đồ Kiến trúc các Lớp (Layers)\</summary\>

```
┌────────────────────────────────────────────────────────────────────────────┐
│                           [ MANAGEMENT LAYER ]                             │
│────────────────────────────────────────────────────────────────────────────│
│  vCenter Server                                                            │
│     ├─ Quản lý tập trung tất cả các ESXi Hosts                             │
│     ├─ Tạo / Giám sát Cluster                                              │
│     ├─ Cấu hình tài nguyên (DRS, HA, vSAN, vDS, vMotion, ...)              │
│     └─ Cung cấp API, PowerCLI, SSO, Role-Based Access Control              │
└────────────────────────────────────────────────────────────────────────────┘
                     │
                     ▼
┌────────────────────────────────────────────────────────────────────────────┐
│                              [ CLUSTER LEVEL ]                             │
│────────────────────────────────────────────────────────────────────────────│
│  Cluster (ESXi1, ESXi2, ..., ESXin)                                        │
│     ├─ Tổng hợp tài nguyên vật lý (CPU, RAM, Disk, NIC)                    │
│     ├─ Cho phép tính năng DRS (Dynamic Resource Scheduler)                 │
│     ├─ Cho phép HA (High Availability)                                     │
│     ├─ Cho phép vMotion / Storage vMotion                                  │
│     └─ Kết nối đến Datastore, vSAN, và Virtual Networks                    │
└────────────────────────────────────────────────────────────────────────────┘
                     │
                     ▼
┌────────────────────────────────────────────────────────────────────────────┐
│                             [ COMPUTE LAYER ]                              │
│────────────────────────────────────────────────────────────────────────────│
│  Virtual Machines (VMs)                                                    │
│     ├─ Sử dụng CPU & RAM được phân bổ từ ESXi                              │
│     ├─ Hỗ trợ snapshot, clone, template                                    │
│     ├─ Chạy OS (Linux, Windows, Appliance, vCSA, vROps, vCenter ...)       │
│     └─ DRS tự động cân bằng tải CPU/RAM giữa các host                      │
└────────────────────────────────────────────────────────────────────────────┘
                     │
                     ▼
┌────────────────────────────────────────────────────────────────────────────┐
│                             [ NETWORK LAYER ]                              │
│────────────────────────────────────────────────────────────────────────────│
│  vSphere Distributed Switch (vDS)                                          │
│     ├─ Distributed Port Groups (App, DB, Mgmt, vMotion...)                 │
│     ├─ VLAN Tagging (VLAN 10, 20, 30...)                                   │
│     ├─ Uplink Groups (NIC0..NICn) kết nối vật lý tới Switch thật           │
│     ├─ Load Balancing / Failover                                           │
│     └─ Hỗ trợ Network I/O Control, vMotion traffic, Management traffic     │
└────────────────────────────────────────────────────────────────────────────┘
                     │
                     ▼
┌────────────────────────────────────────────────────────────────────────────┐
│                             [ STORAGE LAYER ]                              │
│────────────────────────────────────────────────────────────────────────────│
│  vSAN / Datastore / iSCSI / NFS                                            │
│     ├─ vSAN: chia sẻ dung lượng local disk của các host                    │
│     ├─ Datastore: nơi lưu file VM (.vmdk, .vmx, snapshot, ISO)             │
│     ├─ iSCSI / NFS: lưu trữ mạng ngoài (NAS / SAN)                         │
Setting, performance │
└────────────────────────────────────────────────────────────────────────────┘
                     │
                     ▼
┌────────────────────────────────────────────────────────────────────────────┐
│                    [ SECURITY & AUTOMATION LAYER ]                         │
│────────────────────────────────────────────────────────────────────────────│
│  - SSO (Single Sign-On)                                                    │
│  - Role-Based Access Control (RBAC)                                        │
│  - Certificates, Trust, Audit Logs                                         │
│  - PowerCLI / API / vRealize Automation                                    │
│  - Backup & Restore (vSphere Replication, Veeam, ...)                      │
└────────────────────────────────────────────────────────────────────────────┘
```

\</details\>

-----

## 🗺️ Lộ trình Học tập (Mục lục Labs)

Các bài lab được chia thành 6 phần chính, đi từ thiết lập cơ bản đến tối ưu hóa và bảo mật nâng cao.

### 🧠 PHẦN 1 – Giới thiệu & Thiết lập Môi trường

| Lab | Chủ đề | Mục tiêu |
| :--- | :--- | :--- |
| **Lab 1A** | Môi trường vSphere Cốt lõi | Nắm vững các thành phần ESXi, vCenter, Datacenter, và Cluster. |
| **Lab 1B** | Kiểm thử Cluster (HA/DRS) | Mô phỏng lỗi (fault tolerance) và cân bằng tải (load balancing) cơ bản. |
| **Lab 1C** | Mô phỏng Đa địa điểm (Multi-site) | Hiểu cách vCenter quản lý các cluster phân tán địa lý và phân đoạn mạng. |

### 🌐 PHẦN 2 – Khả năng Mở rộng Mạng (Network)

| Lab | Chủ đề | Mục tiêu |
| :--- | :--- | :--- |
| **Lab 2A** | vDS vs. vSS | Phân biệt và hiểu lợi ích của Distributed Switch (vDS) so với Standard Switch (vSS). |
| **Lab 2B** | LACP & NIOC | Cấu hình gộp link (LACP) và kiểm soát I/O mạng (NIOC) để tối ưu băng thông. |
| **Lab 2C** | Port Mirroring & QoS | Giám sát (security) và ưu tiên (performance) lưu lượng mạng. |

### 💾 PHẦN 3 – Khả năng Mở rộng Lưu trữ (Storage)

| Lab | Chủ đề | Mục tiêu |
| :--- | :--- | :--- |
| **Lab 3A** | VMFS & NFS | So sánh và triển khai các loại datastore phổ biến (block-level vs. file-level). |
| **Lab 3B** | Storage Policy & DRS | Tự động hóa cân bằng tải I/O và dung lượng lưu trữ dựa trên chính sách. |
| **Lab 3C** | vSAN Cluster | Triển khai lưu trữ siêu hội tụ (HCI) với vSAN để đạt được khả năng mở rộng và dự phòng. |

### 🧑‍💻 PHẦN 4 – Quản lý & Triển khai Host

| Lab | Chủ đề | Mục tiêu |
| :--- | :--- | :--- |
| **Lab 4A** | Content Library | Quản lý tập trung các ISO, template và script để triển khai VM nhất quán. |
| **Lab 4B** | Host Profile | Đảm bảo tính tuân thủ (compliance) và đồng nhất cấu hình trên nhiều ESXi host. |
| **Lab 4C** | Auto Deploy | Triển khai hàng loạt ESXi host tự động qua mạng (PXE) cho môi trường stateless. |

### ⚡ PHẦN 5 – Tối ưu hóa Hiệu suất

| Lab | Chủ đề | Mục tiêu |
| :--- | :--- | :--- |
| **Lab 5A** | CPU/Memory Scheduler | Hiểu các chỉ số `esxtop` quan trọng như %RDY và Ballooning khi có tranh chấp tài nguyên. |
| **Lab 5B** | Giám sát với `esxtop` | Sử dụng `esxtop` để xác định các điểm nghẽn (bottlenecks) về CPU, RAM, Mạng, và Đĩa. |
| **Lab 5C** | Tinh chỉnh DRS | Tối ưu hóa các cài đặt của DRS (ví dụ: Aggression Level) để cân bằng tải hiệu quả. |

### 🔒 PHẦN 6 – Bảo mật vSphere

| Lab | Chủ đề | Mục tiêu |
| :--- | :--- | :--- |
| **Lab 6A** | Users & Roles (RBAC) | Triển khai Phân quyền Dựa trên Vai trò (RBAC) để giới hạn quyền truy cập. |
| **Lab 6B** | Certificate & Encryption | Thay thế chứng chỉ (VMCA) và mã hóa máy ảo (VM Encryption) để bảo vệ dữ liệu. |
| **Lab 6C** | Tuân thủ Bảo mật | Áp dụng các kỹ thuật "hardening" (làm cứng) và Lockdown Mode để tăng cường an ninh cho host. |

-----

## 🚀 Bắt đầu như thế nào?

1.  **Xem Yêu cầu:** Đảm bảo bạn có môi trường lab tối thiểu (ví dụ: 2-3 ESXi host, 1 vCenter, và storage) như mô tả trong `docs/YEU-CAU.md` 
2.  **Học theo thứ tự:** Đi qua các bài lab từ Phần 1 đến Phần 6 để xây dựng kiến thức một cách có hệ thống.
3.  **Đọc lý thuyết trước:** Tham khảo các file tài liệu (`vSan.md`, `vDs-vSs.md`) trong thư mục `docs/` để nắm vững lý thuyết trước khi thực hành.
4.  **Thực hành:** Mở thư mục `labs/` và làm theo hướng dẫn chi tiết cho từng bài lab.

## 🤝 Đóng góp

Nếu bạn phát hiện lỗi, có ý tưởng cải tiến hoặc muốn bổ sung thêm các bài lab, vui lòng tạo một **Issue** hoặc gửi **Pull Request**\!
