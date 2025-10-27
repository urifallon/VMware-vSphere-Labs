# 📘 vSphere Scalability Lab Guide

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
│     └─ Storage Policies: xác định replication, caching, performance        │
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

## 🧠 PHẦN 1 – Course Introduction & Environment Setup

### 🧩 Lab 1A – Hiểu bản chất môi trường vSphere

**Mục tiêu:** Nắm vững các thành phần cốt lõi của vSphere, bao gồm ESXi hosts, vCenter Server, Datacenter, và Cluster. Lab này giúp hiểu cách vSphere quản lý tài nguyên tập trung, từ việc thêm host đến tổ chức inventory.

**Thiết lập:**
- 1 máy chủ ESXi (phiên bản 7.0 hoặc mới hơn)
- 1 máy chủ vCenter Server Appliance (VCSA)
- 1 máy client (Windows/Linux) với trình duyệt để truy cập vSphere Client (HTML5)

**Yêu cầu chi tiết:**
1. Cài đặt vCenter Server Appliance qua file OVA hoặc installer, cấu hình IP tĩnh và DNS
2. Truy cập vSphere Client qua `https://<vCenter-IP>/ui` và đăng nhập với tài khoản admin
3. Thêm máy chủ ESXi vào vCenter: Chọn "Hosts and Clusters" > Right-click vCenter > Add Host > Nhập IP/hostname của ESXi, username (root), password
4. Tạo Datacenter: Right-click vCenter > New Datacenter > Đặt tên (e.g., "MainDC")
5. Tạo Cluster: Right-click Datacenter > New Cluster > Đặt tên (e.g., "BasicCluster"), không bật HA/DRS lúc này
6. Di chuyển ESXi vào Cluster: Drag-and-drop hoặc right-click ESXi > Move To > Chọn Cluster

**Kết quả mong đợi:** Inventory hiển thị cấu trúc: vCenter > Datacenter > Cluster > ESXi Host. Bạn có thể kiểm tra bằng cách xem resource pools và storage/network liên kết. Nếu gặp lỗi kết nối, kiểm tra firewall trên ESXi (`esxcli network firewall ruleset set --ruleset-id=syslog --enabled=true`) hoặc DNS resolution. Lab chứng minh vCenter là trung tâm quản lý, cho phép giám sát và cấu hình tập trung.

---

### 🧩 Lab 1B – Mô phỏng test: Cluster cơ bản

**Mục tiêu:** Kiểm thử High Availability (HA) và Distributed Resource Scheduler (DRS) để hiểu fault tolerance và load balancing.

**Thiết lập:**
- 3 máy chủ ESXi với shared storage (e.g., NFS hoặc vSAN)
- 1 vCenter Server quản lý các ESXi

**Yêu cầu chi tiết:**
1. Thêm tất cả 3 ESXi vào vCenter và tạo Cluster
2. Bật HA: Right-click Cluster > Settings > vSphere HA > Enable > Cấu hình Admission Control (e.g., Host failures cluster tolerates: 1)
3. Bật DRS: Trong cùng menu > vSphere DRS > Enable > Automation Level: Fully Automated
4. Tạo 2 VM (e.g., Ubuntu VMs) trên ESXi1, với vCPU và memory đủ để test
5. Simulate failure: Tắt ESXi1 qua console hoặc power off (nếu virtual)
6. Quan sát trong vSphere Client: Events tab để thấy HA failover

**Kết quả mong đợi:** VM tự động restart trên ESXi2 hoặc ESXi3 trong vòng 1-5 phút. DRS có thể migrate VM trước failure nếu tải cao. Kiểm tra bằng `esxtop` trên host còn lại. Nếu HA không trigger, kiểm tra heartbeat datastores hoặc network isolation. Lab minh họa tính sẵn sàng cao trong môi trường test.

---

### 🧩 Lab 1C – Mô phỏng doanh nghiệp: Multi-site

**Mục tiêu:** Hiểu triển khai multi-site với vCenter quản lý các Cluster phân tán địa lý, nhấn mạnh network segmentation.

**Thiết lập:**
- 2 subnet: 10.10.10.x/24 (HN) và 10.10.20.x/24 (SG), với routing giữa chúng
- 1 vCenter ở subnet HN quản lý cả hai

**Yêu cầu chi tiết:**
1. Thêm host: Add Host cho ESXi ở 10.10.10.x (HN) và 10.10.20.x (SG)
2. Tạo Cluster HN: Right-click Datacenter > New Cluster > "HN-Cluster"
3. Tạo Cluster SG: Tương tự, "SG-Cluster"
4. Cấu hình DNS: Trên DNS server, thêm A records như hn-esxi.local (10.10.10.10) và sg-esxi.local (10.10.20.10). Cập nhật `/etc/hosts` trên vCenter nếu cần
5. Kiểm tra latency: Sử dụng ping từ vCenter đến từng host, đảm bảo <50ms

**Kết quả mong đợi:** vCenter hiển thị hai Cluster riêng, với host phân loại theo site. DNS giúp resolve tên dễ dàng. Nếu latency cao, có thể ảnh hưởng vMotion cross-site. Lab chứng minh scalability cho doanh nghiệp đa địa điểm, với potential cho Site Recovery Manager (SRM) sau.

---

## 🌐 PHẦN 2 – Network Scalability

### 🧩 Lab 2A – Hiểu bản chất: vDS vs vSS

**Mục tiêu:** Phân biệt vSwitch Standard (per-host) và Distributed (centralized), hiểu lợi ích scalability.

**Thiết lập:**
- 2 ESXi, mỗi có 1 NIC (vmnic0)

**Yêu cầu chi tiết:**
1. Tạo vSS: Trên ESXi1 console hoặc vSphere Client > Host > Networking > Add Standard Switch > Add uplink (vmnic0) > Tạo port group "VM Network"
2. Tạo vDS: Trong vCenter > Networking > Right-click Datacenter > New Distributed Switch > Add hosts (cả 2 ESXi) > Assign uplinks
3. Tạo port group trên vDS: Right-click vDS > New Distributed Port Group
4. So sánh: Migrate VM từ vSS sang vDS để kiểm tra seamless

**Kết quả mong đợi:** vDS cho phép cấu hình một lần áp dụng cho tất cả host, giảm lỗi. Kiểm tra bằng cách thay đổi VLAN trên port group – thay đổi lan tỏa ngay. Nếu migration fail, kiểm tra MTU mismatch. Lý tưởng cho doanh nghiệp lớn.

---

### 🧩 Lab 2B – Mô phỏng test: LACP & NIOC

**Mục tiêu:** Cấu hình load balancing và traffic shaping để tối ưu throughput.

**Thiết lập:**
- 2 uplink (vmnic0, vmnic1) trên mỗi host

**Yêu cầu chi tiết:**
1. Tạo LACP trên vDS: New LAG (Link Aggregation Group) > Add uplinks > Mode: Active
2. Bật NIOC: vDS Settings > Resource Allocation > Enable Network I/O Control > Set shares (e.g., vMotion: Low, VM Traffic: High)
3. Test: Cài iperf trên 2 VM > `iperf -s` on server, `iperf -c <IP> -t 60` on client > So sánh trước/sau NIOC

**Kết quả mong đợi:** Throughput tăng 2x với LACP; NIOC giới hạn vMotion không ảnh hưởng VM. Nếu low throughput, kiểm tra cable/switch config. Lab test hiệu năng mạng.

---

### 🧩 Lab 2C – Mô phỏng doanh nghiệp: Port Mirroring & QoS

**Mục tiêu:** Giám sát và ưu tiên traffic cho security và performance.

**Thiết lập:**
- 3 VM: AppServer (web app), DBServer (database), Monitor (Wireshark)

**Yêu cầu chi tiết:**
1. Port Mirroring: vDS > Edit > Port Mirroring > New Session > Source: AppServer port > Destination: Monitor port > Enable
2. QoS: Port group DB > Edit > Traffic Shaping > Set Egress/Ingress policies; Tag DSCP cho DB traffic (e.g., value 46 for high priority)
3. Test: Generate traffic (e.g., SQL queries từ App to DB) > Capture trên Monitor với Wireshark

**Kết quả mong đợi:** Monitor capture mirrored packets; DB traffic prioritized in congestion. Nếu no capture, kiểm tra session status. Phù hợp doanh nghiệp với monitoring tools như IDS.

---

## 💾 PHẦN 3 – Storage Scalability

### 🧩 Lab 3A – Hiểu bản chất: VMFS & NFS

**Mục tiêu:** So sánh local vs shared storage.

**Thiết lập:**
- 1 ESXi với local disk và NFS server (Ubuntu)

**Yêu cầu chi tiết:**
1. Tạo VMFS: Host > Storage > New Datastore > VMFS > Chọn disk > Format VMFS6
2. Mount NFS: Cấu hình NFS trên Ubuntu (`exports /nfs *(rw,sync,no_subtree_check)`) > Trong ESXi: New Datastore > NFS > Version 4.1 > Server IP, folder /nfs
3. Test: Copy large file (`dd if=/dev/zero of=test.img bs=1G count=1`) > Đo thời gian

**Kết quả mong đợi:** VMFS faster for local IOPS; NFS better for sharing. Nếu mount fail, kiểm tra permissions. Hiểu format (VMFS clustered) vs mount (NFS network).

---

### 🧩 Lab 3B – Mô phỏng test: Storage Policy & DRS

**Mục tiêu:** Policy-based automation cho I/O balancing.

**Thiết lập:**
- 3 ESXi với 2 shared datastores

**Yêu cầu chi tiết:**
1. Bật Storage DRS: Cluster Settings > Storage DRS > Enable > Automation: Fully Automated
2. Tạo policies: vSphere Tags > New Tag Category "StorageTier" > Tags "Gold" (high IOPS), "Silver"
3. Assign: Tạo VM1 gán Gold, VM2 Silver > Storage Policies > Edit VM Storage Policy
4. Test: Chạy fio trên VM để tạo load > Quan sát migration

**Kết quả mong đợi:** VM migrate đến datastore phù hợp. Nếu no migration, kiểm tra threshold. Tự động hóa storage.

---

### 🧩 Lab 3C – Mô phỏng doanh nghiệp: vSAN Cluster

**Mục tiêu:** Hyper-converged storage với redundancy.

**Thiết lập:**
- 3 ESXi, mỗi có disk groups (1 SSD cache, 1 HDD capacity)

**Yêu cầu chi tiết:**
1. Bật vSAN: Cluster > Configure > vSAN > Services > Enable
2. Policy: New Storage Policy > vSAN > Failures to Tolerate: 1 (RAID-1)
3. Tạo VM gán policy > Monitor: vSAN > Virtual Objects

**Kết quả mong đợi:** Objects distributed/replicated. Test tắt host – data accessible. Nếu fault, kiểm tra disk health. Scalable cho enterprise.

---

## 🧑‍💻 PHẦN 4 – Host & Management Scalability

### 🧩 Lab 4A – Hiểu bản chất: Content Library

**Mục tiêu:** Centralized content cho deployment.

**Thiết lập:**
- vCenter với 100GB datastore

**Yêu cầu chi tiết:**
1. Tạo Library: Inventory > Content Libraries > New > Local > Mount datastore
2. Add items: Upload ISO, convert VM to template > Publish
3. Deploy: Right-click template > New VM from This Template

**Kết quả mong đợi:** VM deployed nhanh, consistent. Nếu sync issue, kiểm tra subscription. Tiết kiệm thời gian.

---

### 🧩 Lab 4B – Mô phỏng test: Host Profile

**Mục tiêu:** Compliance qua profiles.

**Thiết lập:**
- 3 ESXi

**Yêu cầu chi tiết:**
1. Tạo Profile: Right-click esxi01 > Extract Host Profile
2. Apply: Attach to esxi02/esxi03 > Check Compliance > Remediate
3. Compare: View differences in profile editor

**Kết quả mong đợi:** Hosts aligned. Nếu non-compliant, fix manually. Nhanh cho nhiều host.

---

### 🧩 Lab 4C – Mô phỏng doanh nghiệp: Auto Deploy

**Mục tiêu:** Stateless provisioning.

**Thiết lập:**
- vCenter + Ubuntu (DHCP/TFTP)

**Yêu cầu chi tiết:**
1. Cài Auto Deploy: vCenter > Auto Deploy > Enable
2. Tạo Image Profile: PowerCLI > `New-EsxImageProfile`
3. Boot host PXE: Cấu hình DHCP option 66/67 > Boot từ network

**Kết quả mong đợi:** ESXi installed automatically. Scale cho hàng trăm host.

---

## ⚡ PHẦN 5 – Performance Optimization

### 🧩 Lab 5A – Hiểu bản chất: CPU/Memory Scheduler

**Mục tiêu:** Overcommit metrics.

**Thiết lập:**
- 1 ESXi, 2 VM overcommit (e.g., 8 vCPU on 4-core host)

**Yêu cầu chi tiết:**
1. Chạy stress (`stress-ng --cpu 8`) trên VM
2. esxtop: SSH to ESXi > `esxtop` > c (CPU) xem %RDY > m (Memory) xem Ballooning

**Kết quả mong đợi:** High %RDY indicate contention. Reclaim via ballooning.

---

### 🧩 Lab 5B – Mô phỏng test: esxtop Performance Monitoring

**Mục tiêu:** Resource tracking.

**Thiết lập:**
- 1 ESXi, 2 VM

**Yêu cầu chi tiết:**
1. Chạy stress tests
2. esxtop views: c/m/n/u > Capture khi load cao

**Kết quả mong đợi:** Identify bottlenecks (e.g., high %CSTP for CPU).

---

### 🧩 Lab 5C – Mô phỏng doanh nghiệp: Performance Tuning

**Mục tiêu:** DRS optimization.

**Thiết lập:**
- 3 ESXi Cluster

**Yêu cầu chi tiết:**
1. Bật DRS Fully Automated
2. Load VM > Monitor migrations
3. Tune: Set aggression to Aggressive

**Kết quả mong đợi:** Balanced resources. Tune for large apps.

---

## 🔒 PHẦN 6 – vSphere Security

### 🧩 Lab 6A – Hiểu bản chất: User & Role

**Mục tiêu:** RBAC basics.

**Thiết lập:**
- 1 vCenter, 2 local users

**Yêu cầu chi tiết:**
1. Tạo roles: Administration > Roles > New
2. Assign: Access Control > Global Permissions

**Kết quả mong đợi:** Viewer can't edit. Secure access.

---

### 🧩 Lab 6B – Mô phỏng test: Certificate & Encryption

**Mục tiêu:** Secure comms and data.

**Thiết lập:**
- vCenter, 1 VM

**Yêu cầu chi tiết:**
1. VMCA: Certificate Manager > Replace
2. Encryption: New Policy > Enable > Assign to VM

**Kết quả mong đợi:** Encrypted disks. Verify with vSphere Client.

---

### 🧩 Lab 6C – Mô phỏng doanh nghiệp: Security Compliance

**Mục tiêu:** Hardening.

**Thiết lập:**
- Cluster

**Yêu cầu chi tiết:**
1. Lockdown: Host > Configure > Security Profile
2. Scan: Use Hardening Guide PDF > Apply fixes

**Kết quả mong đợi:** Compliant environment. Reduce risks.

---

## 🏁 Tổng quan Lab (18 labs)

| Chủ đề | Số Lab | Mục tiêu chính |
|--------|--------|----------------|
| 1. Course Intro | 3 | Nắm cấu trúc môi trường, cluster, site |
| 2. Network Scalability | 3 | vDS, NIOC, LACP, Port Mirroring |
| 3. Storage Scalability | 3 | VMFS, DRS, vSAN |
| 4. Host & Management | 3 | Content Library, Host Profile, Auto Deploy |
| 5. Performance | 3 | esxtop, tuning, DRS |
| 6. Security | 3 | Role, Certificate, Hardening |
