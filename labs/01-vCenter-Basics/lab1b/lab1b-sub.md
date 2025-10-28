# Lab 1B Sub: Chuẩn bị cho kiểm thử HA/DRS
## Phần này sẽ hỗ trợ cho phần triển khai lab1b phía sau.

I. Yêu cầu & cách thức hoạt động
1. MỤC TIÊU TRIỂN KHAI CLUSTER (HA + DRS)

| Tính năng                            | Mục đích                                                       |
| ------------------------------------ | -------------------------------------------------------------- |
| **HA (High Availability)**           | Tự động khởi động lại VM trên host khác khi một host bị lỗi.   |
| **DRS (Dynamic Resource Scheduler)** | Cân bằng tải CPU/RAM giữa các host thông qua vMotion.          |
| **vMotion / Storage vMotion**        | Di chuyển VM giữa các host (hoặc datastore) mà không downtime. |

2. THIẾT BỊ & HẠ TẦNG CẦN CÓ

| Thành phần                              | Số lượng / yêu cầu tối thiểu     | Mô tả                                                                         |
| --------------------------------------- | -------------------------------- | ----------------------------------------------------------------------------- |
| **1. vCenter Server (Appliance)**       | 1 máy ảo                         | Trung tâm quản lý Cluster, DRS, HA, vMotion, Datastore.                       |
| **2. ESXi Host**                        | ≥ 2 host vật lý                  | Cần tối thiểu 2 host để DRS và HA hoạt động.                                  |
| **3. NIC vật lý (Physical NIC)**        | ≥ 4 NIC mỗi host (khuyến nghị)   | Phân tách traffic: Management, vMotion, vSAN, VM Network.                     |
| **4. Shared Storage (Datastore chung)** | Bắt buộc                         | Tất cả host phải truy cập được **cùng một Datastore** (iSCSI, NFS hoặc vSAN). |
| **5. VLAN & Switch Layer 2/3**          | VLAN riêng cho từng loại traffic | VLAN Management, vMotion, vSAN, VM Network.                                   |
| **6. DNS Server / NTP Server**          | Có                               | vCenter và các host cần DNS phân giải 2 chiều (FQDN) và đồng bộ giờ NTP.      |
| **7. License VMware**                   | vSphere Standard trở lên         | DRS cần **Enterprise Plus**, HA cần **Standard** trở lên.                     |

3. MẠNG & VLAN

| Loại traffic           | VLAN riêng | Giao thức / yêu cầu                      |
| ---------------------- | ---------- | ---------------------------------------- |
| **Management Network** | VLAN10     | vCenter ↔ ESXi (SSH, API, HA heartbeat)  |
| **vMotion Network**    | VLAN20     | VMkernel riêng, MTU ≥ 9000 (Jumbo Frame) |
| **vSAN Network**       | VLAN30     | Multicast/Unicast, MTU ≥ 9000            |
| **VM Network**         | VLAN40+    | Các VLAN của VM (App, DB, Frontend,...)  |

4. Cách HA hoạt động

vCenter bật HA cho cluster → tạo FDM Agent (Fault Domain Manager) trên mỗi host.
Các host gửi heartbeat qua management network (hoặc datastore).
Khi 1 host không phản hồi trong X giây (default: 15s), cluster đánh giá:
Nếu host chết thực sự, các VM của nó sẽ được khởi động lại trên host còn sống.
Nếu chỉ mất kết nối tạm thời, HA sẽ không di chuyển VM.

II. Triển khai vSs/vDs và Shared Storage (vSan) sử dụng cho HA/DRS
1. Triển khai dịch vụ mạng và vlan (link đến: 3. MẠNG & VLAN)


III. Migrate vSs to vDs (bonus vmk0 vSs to DPG-MGMT vDs)

> Di chuyển chỉ vmk0 (Management IP) từ vSwitch0 (vSS) sang vDS-Cluster → DPG_10.10.10.0_prefix24, dùng cùng subnet 10.10.10.0/24.
> (giữ nguyên IP/Netmask/Gateway, không đổi VLAN nếu lab không có trunk).

1. Snapshot VM ESXi (nếu chạy nested) và snapshot vCenter (VCSA).
2. Kiểm tra IP hiện tại của vmk0 trên host:
- Truy cập vào Host (Có cách khác là dùng Console hoặc SSH).

![iiibuoc1](/img/lab1b-iii-buoc1.png)

> Enter vào Troubleshooting Option

![iiibuoc2](/img/lab1b-iii-buoc2.png)

> Enter vào `ESXI Shell`

=> khi đó hiển thị như này:
![iiibuoc3](/img/lab1b-iii-buoc3.png)

> Sau đó ESC để thoát ra ngoài màn hình chính.

> Ở ngoài màn hình chính bấm tổ hợp nút `Alt` + `F1` -> Sau đó nhập user root và mật khẩu của bạn

- Lệnh kiểm tra:
```shell
esxcli network ip interface ipv4 get
```
=> khi đó hiển thị như này:
![iiibuoc4](/img/lab1b-iii-buoc4.png)

**Lưu ý:** Ghi lại vmk0 IP / Netmask / Gateway.
3. Kiểm tra vmnic0 hiện đang gắn vào vSwitch0 và là uplink:
 Host Client → Configure → Networking → Virtual switches → vSwitch0 → Physical adapters.

![iiibuoc5](/img/lab1b-iii-buoc5.png)


4. Kiểm tra vDS + DPG:
Trong vCenter: Networking → vDS-Cluster → Port groups → Có DPG_10.10.10.0_prefix24 chưa?
Nếu chưa thực hiện như bên dưới
Nếu có rồi chuyển sang bước (5)

**⚠️ Lưu ý quan trọng:** Nếu bạn lab trên Workstation (VMnet8 host-only), set VLAN = 0 (none/untagged) cho Port Group này. Nếu port-group đang có VLAN ≠ 0 trong môi trường không có trunk vật lý, sẽ bị drop.