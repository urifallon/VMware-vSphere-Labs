Phân chia mạng hoàn chỉnh
| Zone                       | Chức năng                   | Subnet                | Gợi ý IP cho VM | Mô tả                          |
| -------------------------- | --------------------------- | --------------------- | --------------- | ------------------------------ |
| **WEB zone**               | Reverse Proxy (Nginx) + SSL | **10.0.10.0/24**      | 10.0.10.10      | Public entry point, nhận HTTPS |
| **APP zone**               | PHP-FPM + OpenEMR app logic | **10.0.12.0/24**      | 10.0.12.10      | Xử lý request PHP              |
| **DB zone**                | MariaDB                     | **10.0.80.0/24**      | 10.0.80.10      | Lưu dữ liệu EMR                |
| **MGMT zone**              | Quản lý, cronjob, backup    | **10.0.40.0/24**      | 10.0.40.10      | Dành cho quản trị viên         |
| **GATEWAY zone (pfSense)** | NAT & route                 | **10.0.1.0/24 (LAN)** | 10.0.1.1        | Định tuyến giữa các subnet     |
| **WAN (Internet)**         | pfSense WAN interface       | DHCP / VMNet8         | Tự động IP      | Kết nối ra ngoài mạng thực     |

Cấu hình từng máy ảo
| VM      | Zone    | Subnet            | IP         | OS            | Chức năng               | Gợi ý cấu hình               |
| ------- | ------- | ----------------- | ---------- | ------------- | ----------------------- | ---------------------------- |
| **VM1** | WEB     | 10.0.10.0/24      | 10.0.10.10 | Ubuntu 24.04  | Reverse Proxy, SSL, UFW | 2 vCPU / 2GB RAM / 20GB disk |
| **VM2** | APP     | 10.0.12.0/24      | 10.0.12.10 | Ubuntu 24.04  | PHP-FPM + OpenEMR code  | 2 vCPU / 4GB RAM / 30GB disk |
| **VM3** | DB      | 10.0.80.0/24      | 10.0.80.10 | Debian 12     | MariaDB server          | 2 vCPU / 4GB RAM / 40GB disk |
| **VM4** | MGMT    | 10.0.40.0/24      | 10.0.40.10 | Ubuntu 24.04  | Cron, backup, monitor   | 2 vCPU / 2GB RAM / 30GB disk |
| **VM5** | Gateway | 10.0.1.0/24 (LAN) | 10.0.1.1   | pfSense 2.7.x | Router, NAT, Firewall   | 1 vCPU / 1GB RAM             |

Interface layout:
| Interface | IP/Subnet     | Zone     | Description                   |
| --------- | ------------- | -------- | ----------------------------- |
| **WAN**   | DHCP (VMNet8) | Internet | Truy cập web, SSL, apt update |
| **LAN1**  | 10.0.10.1/24  | WEB      | Giao tiếp với Nginx           |
| **LAN2**  | 10.0.12.1/24  | APP      | Giao tiếp với PHP-FPM         |
| **LAN3**  | 10.0.80.1/24  | DB       | Giao tiếp MariaDB             |
| **LAN4**  | 10.0.40.1/24  | MGMT     | Quản lý toàn hệ thống         |

Firewall rules mẫu (pfSense)
| From                | To                 | Port   | Action | Description             |
| ------------------- | ------------------ | ------ | ------ | ----------------------- |
| WEB (10.0.10.0/24)  | APP (10.0.12.0/24) | 9000   | ALLOW  | Gửi request đến PHP-FPM |
| APP (10.0.12.0/24)  | DB (10.0.80.0/24)  | 3306   | ALLOW  | Kết nối MariaDB         |
| MGMT (10.0.40.0/24) | all                | 22     | ALLOW  | SSH quản lý             |
| WEB (10.0.10.0/24)  | Internet           | 443    | ALLOW  | SSL, apt update         |
| DB (10.0.80.0/24)   | any                | *      | DENY   | Tăng bảo mật dữ liệu    |
| Internet            | WEB (10.0.10.0/24) | 80,443 | ALLOW  | Client truy cập         |



Dòng kết nối ứng dụng OpenEMR
| Luồng              | Giao thức      | Mục đích                    |
| ------------------ | -------------- | --------------------------- |
| Client → VM1       | HTTPS (443)    | Truy cập OpenEMR            |
| VM1 → VM2          | FastCGI (9000) | Xử lý PHP                   |
| VM2 → VM3          | MySQL (3306)   | Lưu dữ liệu                 |
| VM4 → VM2/VM3      | SSH / cron     | Quản lý, backup             |
| pfSense → Internet | HTTP/HTTPS     | Cập nhật hệ thống, cert SSL |

+---------------------+
|     Internet        |
|  (Client / Browser) |
+----------+----------+
           | HTTPS (443)
           v
+---------------------+      (WAN: DHCP)
|   pfSense Gateway   |<-------------------+
|   (10.0.1.1)        |                    |
+----------+----------+                    |
           |                                |
           | NAT / Port Forward 80,443      | (VMNet8 – Internet access)
           v                                |
+---------------------+                    |
|   VM1: WEB Zone     |                    |
|   Nginx + SSL       |                    |
|   IP: 10.0.10.10    |                    |
+----------+----------+                    |
           | FastCGI over TCP (port 9000)   |
           v                                |
+---------------------+                    |
|   VM2: APP Zone     |                    |
|   PHP-FPM + OpenEMR |                    |
|   IP: 10.0.12.10    |                    |
+----------+----------+                    |
           | MySQL (3306)                   |
           v                                |
+---------------------+                    |
|   VM3: DB Zone      |                    |
|   MariaDB           |                    |
|   IP: 10.0.80.10    |                    |
+----------+----------+                    |
           ^                                |
           | SSH / Backup / Cron            |
           |                                |
+---------------------+                    |
|   VM4: MGMT Zone    |--------------------+
|   Admin, Monitoring |
|   IP: 10.0.40.10    |
+---------------------+

+--------------------------------------------------+
|                pfSense Firewall Rules            |
| • Internet → WEB: allow 80,443                   |
| • WEB → APP: allow 9000                          |
| • APP → DB: allow 3306                           |
| • MGMT → all: allow 22 (SSH)                     |
| • DB → any: DENY (default-deny principle)        |
+--------------------------------------------------+

+--------------------------------------------------+
|                Network Segmentation              |
| • PG-WEB:    10.0.10.0/24                        |
| • PG-APP:    10.0.12.0/24                        |
| • PG-DB:     10.0.80.0/24                        |
| • PG-MGMT:   10.0.40.0/24                        |
| • PG-GW:     10.0.1.0/24 (pfSense LAN interface) |
+--------------------------------------------------+

MGMT
10.0.40.0/24
10.0.40.1
ESXi, vCenter, VM4
WEB
10.0.10.0/24
10.0.10.1
VM1 (Nginx)
APP
10.0.12.0/24
10.0.12.1
VM2 (PHP-FPM + OpenEMR)
DB
10.0.80.0/24
10.0.80.1
VM3 (MariaDB)
GW
10.0.1.0/24
—
pfSense LAN interface (định tuyến)
WAN
Internet (DHCP)
—
pfSense WAN → ra ngoài


---------------------
vmnet6 - HostOnly - Host connect - DHCD Enable - 10.0.40.0 

Esxi 8 
Cấu hình mạng của Esxi
- gắn adapter đầu tiên -> nối với vmnet6 -> vnic0 -> vSs0
- ipv4 10.0.40.5 - 255.255.255.0(subnet) - 10.0.40.1 (GW)
- DNS không cấu hình (not set)
- ipv6 disable

vCenter trên esxi 8
Cấu hình mạng của vcenter
- gắn với vSs0
- ipv4 10.0.40.20 - 255.255.255.0 - 10.0.40.1(GW) - 8.8.8.8(DNS)
