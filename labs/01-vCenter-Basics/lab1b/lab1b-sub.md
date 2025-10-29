# Lab 1B Sub: Chuẩn bị cho kiểm thử HA/DRS

## I. Yêu cầu và cách thức hoạt động

### 1. Mục tiêu triển khai Cluster (HA + DRS)

| Tính năng                            | Mục đích                                                       |
| ------------------------------------ | -------------------------------------------------------------- |
| **HA (High Availability)**           | Tự động khởi động lại VM trên host khác khi một host bị lỗi.   |
| **DRS (Dynamic Resource Scheduler)** | Cân bằng tải CPU/RAM giữa các host thông qua vMotion.          |
| **vMotion / Storage vMotion**        | Di chuyển VM giữa các host (hoặc datastore) mà không downtime. |

### 2. Thiết bị và hạ tầng cần có

| Thành phần                              | Số lượng / yêu cầu tối thiểu     | Mô tả                                                                         |
| --------------------------------------- | -------------------------------- | ----------------------------------------------------------------------------- |
| **1. vCenter Server (Appliance)**       | 1 máy ảo                         | Trung tâm quản lý Cluster, DRS, HA, vMotion, Datastore.                       |
| **2. ESXi Host**                        | ≥ 2 host vật lý                  | Cần tối thiểu 2 host để DRS và HA hoạt động.                                  |
| **3. NIC vật lý (Physical NIC)**        | ≥ 4 NIC mỗi host (khuyến nghị)   | Phân tách traffic: Management, vMotion, vSAN, VM Network.                     |
| **4. Shared Storage (Datastore chung)** | Bắt buộc                         | Tất cả host phải truy cập được **cùng một Datastore** (iSCSI, NFS hoặc vSAN). |
| **5. VLAN & Switch Layer 2/3**          | VLAN riêng cho từng loại traffic | VLAN Management, vMotion, vSAN, VM Network.                                   |
| **6. DNS Server / NTP Server**          | Có                               | vCenter và các host cần DNS phân giải 2 chiều (FQDN) và đồng bộ giờ NTP.      |
| **7. License VMware**                   | vSphere Standard trở lên         | DRS cần **Enterprise Plus**, HA cần **Standard** trở lên.                     |

### 3. Mạng và VLAN

| Loại traffic           | VLAN riêng | Giao thức / yêu cầu                      |
| ---------------------- | ---------- | ---------------------------------------- |
| **Management Network** | VLAN10     | vCenter ↔ ESXi (SSH, API, HA heartbeat)  |
| **vMotion Network**    | VLAN20     | VMkernel riêng, MTU ≥ 9000 (Jumbo Frame) |
| **vSAN Network**       | VLAN30     | Multicast/Unicast, MTU ≥ 9000            |
| **VM Network**         | VLAN40+    | Các VLAN của VM (App, DB, Frontend,...)  |

### 4. Cách HA hoạt động

**Quy trình hoạt động:**
1. vCenter bật HA cho cluster → tạo FDM Agent (Fault Domain Manager) trên mỗi host
2. Các host gửi heartbeat qua management network (hoặc datastore)
3. Khi 1 host không phản hồi trong X giây (default: 15s), cluster đánh giá:
   - Nếu host chết thực sự, các VM của nó sẽ được khởi động lại trên host còn sống
   - Nếu chỉ mất kết nối tạm thời, HA sẽ không di chuyển VM

## II. Triển khai vSS/vDS và Shared Storage (vSAN) sử dụng cho HA/DRS

### 1. Triển khai dịch vụ mạng và VLAN 

**Lưu ý:** Tham khảo phần [3. Mạng và VLAN](#3-mạng-và-vlan) để biết các cấu hình VLAN cần thiết.



## III. Migrate vSS to vDS (Bonus: vmk0 vSS to DPG-MGMT vDS)

> Di chuyển chỉ vmk0 (Management IP) từ vSwitch0 (vSS) sang vDS-Cluster → DPG_10.10.10.0_prefix24, dùng cùng subnet 10.10.10.0/24.
> (giữ nguyên IP/Netmask/Gateway, không đổi VLAN nếu lab không có trunk).

1. Chuẩn bị mạng của VM như sau 
![buoc1](./img/lab1b-sub-buoc1.png)

ESXI cần 2 nic (Adapter) khác nhau nhưng cùng 1 dải mạng để có thể dự phòng khi chuyển đổi từ vSs sang vDs

- Khi chuyển đổi không có mạng dự phòng:
hình ảnh
    - Khi chuyển đưa vcenter từ vSs sang dùng mạng vDs sẽ trực tiếp bị ngắt
    - Từ đó cũng không không thể migrate Esxi sang sử dụng vDs được 

- Khi chuyển đổi có mạng dự phòng:
hình ảnh
    - Khi này sử dụng vDs ta sẽ có nic1 được gắn vào uplink1, uplink1 được gắn vào PG-MGMT => ta có một đường y hệt đường từ nic0 đến VM Network
    - 