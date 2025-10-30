https://www.howtoforge.com/how-to-install-openemr-on-ubuntu-24-04-server/


## 🧩 **I. Sơ đồ mạng tổng thể – bản cập nhật (đúng với vDS hiện tại)**

| Zone / Network | Chức năng                      | Subnet        | pfSense Interface IP       | Gợi ý VM IP | Mô tả                                      |
| -------------- | ------------------------------ | ------------- | -------------------------- | ----------- | ------------------------------------------ |
| **WAN-DHCP**   | Internet access                | DHCP / VMNet8 | DHCP (ví dụ 192.168.198.x) | —           | pfSense ra Internet, cập nhật package      |
| **GW**         | pfSense LAN core               | 10.10.1.0/24  | 10.10.1.1                  | —           | pfSense nội bộ – route giữa các zone       |
| **WebProxy**   | Reverse Proxy (Nginx + SSL)    | 10.10.20.0/24 | 10.10.20.1                 | 10.10.20.10 | Nhận HTTPS từ client, chuyển tiếp sang App |
| **App**        | PHP-FPM + OpenEMR logic        | 10.10.12.0/24 | 10.10.12.1                 | 10.10.12.10 | Xử lý request, kết nối DB                  |
| **DB**         | MariaDB                        | 10.10.80.0/24 | 10.10.80.1                 | 10.10.80.10 | Lưu dữ liệu EMR                            |
| **MGMT**       | Quản trị ESXi, backup, monitor | 10.10.10.0/24 | 10.10.10.1                 | 10.10.10.10 | Quản lý, SSH, cron                         |

---

## ⚙️ **II. Mapping network trong ESXi / vDS**

| vDS PortGroup                        | Gán cho VM                              | Subnet        | Dạng kết nối           | Ghi chú               |
| ------------------------------------ | --------------------------------------- | ------------- | ---------------------- | --------------------- |
| **DPG-GW_10.10.1.0_prefix24**        | pfSense (NIC1)                          | 10.10.1.0/24  | Internal route         | LAN gateway           |
| **DPG-WAN-DHCP**                     | pfSense (NIC2)                          | DHCP / VMNet8 | External (ra Internet) | WAN                   |
| **DPG-WebProxy_10.10.20.0_prefix24** | pfSense (NIC3), VM1                     | 10.10.20.0/24 | Internal               | Reverse Proxy network |
| **DPG-App_10.10.12.0_prefix24**      | pfSense (NIC4), VM2                     | 10.10.12.0/24 | Internal               | App layer             |
| **DPG-DB_10.10.80.0_prefix24**       | pfSense (NIC5), VM3                     | 10.10.80.0/24 | Internal               | Database layer        |
| **DPG-MGMT_10.10.10.0_prefix24**     | pfSense (NIC6), VM4, vCenter, ESXi host | 10.10.10.0/24 | Internal               | Quản lý               |

> 👉 pfSense có 6 NIC tương ứng 6 DPG ở trên.
> Các VM Web/App/DB/MGMT chỉ gắn vào đúng DPG tương ứng với zone của chúng.

---

## 🔧 **III. Cấu hình pfSense – từng bước chi tiết**

### 1️⃣ Trong trình cài đặt (Installer)

Chọn cài bình thường → “Auto (UFS)” → hostname `pfsense.lab.local`
Sau khi reboot:

### 2️⃣ Gán Interface:

Khi pfSense hỏi:

```
Enter WAN interface: em0  → DPG-WAN-DHCP
Enter LAN interface: em1  → DPG-GW_10.10.1.0_prefix24
Add another interface? [y/n]: y
```

Thêm:

* `em2` → DPG-WebProxy_10.10.20.0_prefix24
* `em3` → DPG-App_10.10.12.0_prefix24
* `em4` → DPG-DB_10.10.80.0_prefix24
* `em5` → DPG-MGMT_10.10.10.0_prefix24

Hoàn tất:
`WAN = em0`, `GW = em1`, `WEB = em2`, `APP = em3`, `DB = em4`, `MGMT = em5`.

### 3️⃣ Cấu hình IP cho từng interface

Vào Console (option 2 – Assign IP):

| Interface | IP/Subnet     | Gateway | DHCP  |
| --------- | ------------- | ------- | ----- |
| WAN       | DHCP          | Tự động | Có    |
| GW        | 10.10.1.1/24  | —       | Không |
| WebProxy  | 10.10.20.1/24 | —       | Không |
| App       | 10.10.12.1/24 | —       | Không |
| DB        | 10.10.80.1/24 | —       | Không |
| MGMT      | 10.10.10.1/24 | —       | Không |

Tắt DHCP trên tất cả trừ WAN (chỉ pfSense quản lý gateway).

---

## 🔐 **IV. Cấu hình trong WebGUI pfSense**

Truy cập:
`https://10.10.1.1` (qua network GW)

### 1️⃣ NAT Outbound

* Chuyển sang **Manual Outbound NAT**
* Thêm rule cho từng network (Web/App/DB/MGMT) → Interface: WAN → Translation: Interface address
  → Mục đích: Cho phép các subnet nội bộ ra Internet (apt, update, certbot).

### 2️⃣ Firewall Rules

| From           | To     | Port  | Action            | Description |
| -------------- | ------ | ----- | ----------------- | ----------- |
| WAN → WebProxy | 80,443 | ALLOW | Public access     |             |
| WebProxy → App | 9000   | ALLOW | Nginx → PHP-FPM   |             |
| App → DB       | 3306   | ALLOW | OpenEMR ↔ MariaDB |             |
| MGMT → all     | 22     | ALLOW | SSH quản lý       |             |
| DB → any       | *      | DENY  | Tăng bảo mật      |             |

---

## 🧱 **V. Cấu hình IP static trên các VM**

| VM                 | Zone     | IP          | Gateway    | DNS     |
| ------------------ | -------- | ----------- | ---------- | ------- |
| **VM1 – WebProxy** | WebProxy | 10.10.20.10 | 10.10.20.1 | 8.8.8.8 |
| **VM2 – App**      | App      | 10.10.12.10 | 10.10.12.1 | 8.8.8.8 |
| **VM3 – DB**       | DB       | 10.10.80.10 | 10.10.80.1 | 8.8.8.8 |
| **VM4 – MGMT**     | MGMT     | 10.10.10.10 | 10.10.10.1 | 8.8.8.8 |

---

## 🌐 **VI. Dòng kết nối OpenEMR**

```
Client (Browser)
     ↓ HTTPS 443
pfSense (WAN → NAT → WEB)
     ↓
WebProxy (Nginx + SSL) 10.10.20.10
     ↓ FastCGI 9000
App (PHP-FPM + OpenEMR) 10.10.12.10
     ↓ MySQL 3306
DB (MariaDB) 10.10.80.10
```

---

## 📈 **VII. Tiến trình triển khai gợi ý**

| Giai đoạn | Hành động                                         |
| --------- | ------------------------------------------------- |
| 1️⃣       | Cài pfSense, gán IP & kiểm tra ping giữa các zone |
| 2️⃣       | Cấu hình NAT Outbound, Firewall Rules             |
| 3️⃣       | Cài đặt DB Server (MariaDB)                       |
| 4️⃣       | Cài đặt App Server (PHP + OpenEMR Source)         |
| 5️⃣       | Cài đặt WebProxy (Nginx + SSL, reverse proxy)     |
| 6️⃣       | Test truy cập qua HTTPS → Web → App → DB          |
| 7️⃣       | Cấu hình MGMT server (monitoring, backup)         |

===============================================================================

-> Chuẩn bị
detail vDs with uplink,vmnic mapping
vmnic && portgroup && uplink usaged (vDs-Cluster)
vmnic0| vDS PortGroup                        | Uplink Usaged                           | 
------| ------------------------------------ | --------------------------------------- | 
vmnic0| none                                 | none                                    |
vmnic1| **DPG-MGMT_10.10.10.0_prefix24**     | uplink 1                                |
vmnic2| **DPG-GW_10.10.1.0_prefix24**        | uplink 2                                |
vmnic3| **DPG-WAN-DHCP**                     | uplink 3                                |
vmnic4| **DPG-WebProxy_10.10.20.0_prefix24** | uplink 4                                |
vmnic5| **DPG-App_10.10.12.0_prefix24**      | uplink 5                                |
vmnic6| **DPG-DB_10.10.80.0_prefix24**       | uplink 6                                |

-> bước 1: pfsense
ESXI runner (đã được gắn với vDs-Cluster): 
- Tải file iso pfsense https://www.pfsense.org/download/ 
- Tạo VM pfsense trên esxi runner, gắn các card mạng với thứ tự sau:
Adapter 1| **DPG-GW_10.10.1.0_prefix24**        | 
Adapter 2| **DPG-WAN-DHCP**                     | 
Adapter 3| **DPG-WebProxy_10.10.20.0_prefix24** | 
Adapter 4| **DPG-App_10.10.12.0_prefix24**      | 
Adapter 5| **DPG-DB_10.10.80.0_prefix24**       | 
![pfsense](./img/openemr-pfsense-1.png)

Cấu hình pfsense:
-  ... tý làm lại ghi bước sau
![pfsense](./img/openemr-pfsense-2.png)

-> bước 2: cài đặt máy cấu hình cho pfsense (máy ảo utest - ubuntu)
Tại sao phải cài một máy cấu hình? -> hiện tại ta đang làm trong lab sử dụng vmw -> esxi nằm trong vmw -> vm nằm trong esxi -> host không thể với tới để cấu hình bằng gui cho pfsense được -> cài đặt một máy ubuntu cùng dải mạng GW để có thể cấu hình cho pfsense
- truy cập domain 10.10.1.1 từ trình duyệt trong vm utest 
- tài khoản: admin
- mật khẩu : 
