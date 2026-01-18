
# VMware vSphere Labs — Scalability, Performance & Hardening (VI)

Bộ tài liệu **hands-on labs** giúp bạn **xây dựng, vận hành và mở rộng** môi trường **VMware vSphere** theo cách có hệ thống: từ nền tảng (ESXi/vCenter/Cluster) đến **networking (vSS/vDS/LACP/NIOC)**, **storage (VMFS/NFS/iSCSI/vSAN/SPBM)**, **HA/DRS/vMotion**, tối ưu hiệu năng và **hardening bảo mật**.

> Mục tiêu của repo: biến kiến thức vSphere từ “biết dùng UI” thành “thiết kế + vận hành + troubleshoot theo tư duy production”.

---

## Nội dung chính

- **Thiết kế lab vSphere chuẩn thực tế** (nested hoặc vật lý) và baseline vận hành
- **Khả năng mở rộng & tính sẵn sàng cao**: Cluster/HA/DRS/vMotion/Storage vMotion
- **Networking**: vSS vs vDS, VLAN, uplink design, LACP, NIOC, Port Mirroring/QoS
- **Storage**: VMFS/NFS/iSCSI, Storage Policy (SPBM), Storage DRS, vSAN (HCI)
- **Performance & Troubleshooting**: đọc chỉ số, nhận diện bottleneck (CPU/RAM/NET/DISK), `esxtop`
- **Security**: RBAC, chứng chỉ, encryption, lockdown mode, hardening & audit mindset

---

## Đối tượng phù hợp

- SysAdmin/Infra Engineer muốn đi sâu vSphere theo hướng vận hành bài bản
- DevOps/SRE cần nền tảng ảo hóa vững để triển khai workload ổn định
- Sinh viên/người tự học muốn có “lab path” rõ ràng để luyện kỹ năng

---

## Cấu trúc repo (khuyến nghị cách đọc)

- `docs/` — tài liệu nền tảng (khái niệm, yêu cầu lab, best practices)
- `labs/` — hướng dẫn thực hành theo từng lab (step-by-step)
- `img/` — sơ đồ, hình minh hoạ

> Cách học tối ưu: **đọc `docs/` trước** → làm labs theo thứ tự → ghi lại “notes + kết quả + lỗi gặp” để hình thành kỹ năng vận hành.

---

## Kiến trúc tổng quan (high level)

<details>
<summary>Nhấn để xem sơ đồ kiến trúc vSphere (Layers)</summary>

```

┌───────────────────────────────────────────────────────────────────────────┐
│                           [ MANAGEMENT LAYER ]                              │
│  vCenter Server                                                             │
│   ├─ Quản lý tập trung ESXi Hosts / Datacenter / Cluster                    │
│   ├─ DRS, HA, vMotion, vSAN, vDS, Templates/Content Library                 │
│   └─ API/Automation: PowerCLI / REST API / RBAC / SSO                       │
└───────────────────────────────────────────────────────────────────────────┘
│
▼
┌───────────────────────────────────────────────────────────────────────────┐
│                            [ CLUSTER SERVICES ]                             │
│  Cluster                                                                     │
│   ├─ Tổng hợp tài nguyên (CPU/RAM/Storage/Network)                          │
│   ├─ HA (High Availability) / DRS (Load balancing)                          │
│   ├─ vMotion / Storage vMotion                                              │
│   └─ Policy & Compliance (Host Profiles, SPBM, etc.)                        │
└───────────────────────────────────────────────────────────────────────────┘
│
▼
┌───────────────────────────────────────────────────────────────────────────┐
│                             [ ESXi / COMPUTE ]                              │
│  ESXi Hosts                                                                  │
│   ├─ VMkernel networks (Mgmt / vMotion / vSAN / iSCSI/NFS)                  │
│   └─ Workloads: Virtual Machines / Appliances                               │
└───────────────────────────────────────────────────────────────────────────┘
│
┌────────┴─────────┐
▼                  ▼
┌───────────────────┐  ┌───────────────────────────────────────────────────┐
│ [ NETWORK LAYER ] │  │                  [ STORAGE LAYER ]                 │
│ vSS / vDS         │  │ VMFS / NFS / iSCSI / vSAN                          │
│ VLAN/LACP/NIOC    │  │ SPBM / Storage DRS / Datastore design              │
└───────────────────┘  └───────────────────────────────────────────────────┘

┌───────────────────────────────────────────────────────────────────────────┐
│                  [ SECURITY & AUTOMATION (CROSS-CUTTING) ]                 │
│  RBAC/SSO/Certificates/Audit • Encryption • Lockdown/Hardening • API/CLI   │
└───────────────────────────────────────────────────────────────────────────┘

````
</details>

---

## Yêu cầu lab (gợi ý tối thiểu)

Bạn có thể chạy theo 2 hướng:

### A. Nested Lab (phổ biến cho self-study)
- 1 máy workstation/PC mạnh (khuyến nghị **>= 32GB RAM**, càng nhiều càng tốt)
- 2–3 ESXi (nested) + 1 vCenter (VCSA)
- Network plan: ít nhất các mạng **Mgmt / vMotion / Storage(vSAN hoặc iSCSI/NFS)**

### B. Lab vật lý (gần production hơn)
- 2–3 ESXi host vật lý + 1 vCenter
- Storage: vSAN hoặc NAS/SAN (NFS/iSCSI)

> Lưu ý license: vSphere/vCenter thường cần license hoặc trial. Repo này chỉ phục vụ học tập.

---

## Lộ trình Labs (Study Path)

Các labs được chia theo “nhóm năng lực” để bạn đi từ nền tảng → mở rộng → tối ưu → bảo mật.

### Phần 1 — Nền tảng & Thiết lập môi trường
| Lab | Chủ đề | Mục tiêu |
|---:|---|---|
| 1A | vSphere Core (ESXi/vCenter/Datacenter/Cluster) | Nắm cấu trúc và luồng quản trị |
| 1B | Cluster Baseline (HA/DRS) | Hiểu cơ chế HA/DRS và kiểm thử cơ bản |
| 1C | Multi-site (mô phỏng) | Làm quen tư duy phân đoạn & vận hành đa cụm |

### Phần 2 — Networking (Scalability)
| Lab | Chủ đề | Mục tiêu |
|---:|---|---|
| 2A | vSS vs vDS | Nắm khác biệt, use-cases, ưu/nhược |
| 2B | LACP & NIOC | Thiết kế uplink + kiểm soát băng thông |
| 2C | Port Mirroring & QoS | Giám sát và ưu tiên lưu lượng |

### Phần 3 — Storage (Scalability)
| Lab | Chủ đề | Mục tiêu |
|---:|---|---|
| 3A | VMFS & NFS | Triển khai datastore và so sánh block vs file |
| 3B | Storage Policy & Storage DRS | Tự động hóa theo policy và cân bằng tải |
| 3C | vSAN Cluster | HCI, fault domain, policy, operational checks |

### Phần 4 — Host Lifecycle & Deployment
| Lab | Chủ đề | Mục tiêu |
|---:|---|---|
| 4A | Content Library | Chuẩn hóa ISO/template/OVF để triển khai nhất quán |
| 4B | Host Profiles | Compliance, drift detection, đồng nhất cấu hình |
| 4C | Auto Deploy (overview) | Khái niệm stateless/provisioning ở quy mô lớn |

### Phần 5 — Performance & Troubleshooting
| Lab | Chủ đề | Mục tiêu |
|---:|---|---|
| 5A | CPU/Memory scheduling | Hiểu %RDY, contention, ballooning, swapping |
| 5B | `esxtop` thực chiến | Khoanh vùng bottleneck CPU/RAM/NET/DISK |
| 5C | Tuning DRS | Điều chỉnh để cân bằng tải “đúng mục tiêu” |

### Phần 6 — Security & Hardening
| Lab | Chủ đề | Mục tiêu |
|---:|---|---|
| 6A | Users & Roles (RBAC) | Phân quyền theo vai trò, tối thiểu đặc quyền |
| 6B | Certificates & Encryption | Quản trị chứng chỉ và mã hóa workload |
| 6C | Hardening & Lockdown | Checklist hardening + vận hành an toàn |

---

## Bắt đầu nhanh

1. **Clone repo**
   ```bash
   git clone https://github.com/urifallon/VMware-vSphere-Labs.git
   cd VMware-vSphere-Labs
````

2. **Đọc tài liệu nền tảng** trong `docs/` (yêu cầu lab, network/storage plan).
3. **Làm labs theo thứ tự** từ Phần 1 → Phần 6 trong `labs/`.
4. **Ghi chép kết quả**: khuyến nghị tạo `notes/` (local) để lưu topology, IP plan, lỗi gặp và cách xử lý.

---

## Quy ước (để repo dễ maintain)

* Mỗi lab nên có:

  * **Mục tiêu** (Objective)
  * **Prerequisites**
  * **Steps**
  * **Validation** (tiêu chí kiểm chứng thành công)
  * **Rollback/Cleanup**
  * **Troubleshooting** (lỗi thường gặp)

---

## Đóng góp

Nếu bạn phát hiện lỗi hoặc muốn bổ sung labs:

* Tạo **Issue** mô tả rõ bối cảnh + log/ảnh (nếu có)
* Hoặc gửi **Pull Request** (nêu mục tiêu thay đổi + cách test)

---

## Disclaimer

Repo này phục vụ **mục đích học tập và thử nghiệm**. Không liên kết chính thức với VMware/Broadcom.
Không khuyến nghị áp dụng trực tiếp vào production nếu chưa review theo tiêu chuẩn nội bộ và tài liệu vendor.

---

