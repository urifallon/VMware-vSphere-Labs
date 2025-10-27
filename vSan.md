⚙️ YÊU CẦU TRƯỚC KHI TRIỂN KHAI
1️⃣ Môi trường cơ bản

03 ESXi host (cấu hình tối thiểu mỗi host):

RAM ≥ 16GB

≥ 2 NIC (1 NIC cho Management, 1 NIC cho vSAN/vMotion)

≥ 2 Disk:

1 SSD (cache tier)

1 HDD hoặc SSD (capacity tier)

01 máy chạy vCenter Server Appliance (VCSA).

🚀 PHẦN 1 — CẤU HÌNH MẠNG
1. Tạo mạng Management (nếu chưa có)

Mỗi host có:

vSwitch0 → Port Group: Management Network

vmk0 → IP: 10.10.10.x

Gán “Management traffic”

2. Tạo mạng riêng cho vSAN

Mục tiêu: vSAN dùng dải riêng (VD: 10.10.20.x)

Trên từng host:

Vào Networking → Virtual Switches → Add Standard Switch

Tên: vSwitch1

NIC: chọn physical NIC mới cắm (vd: vmnic1)

Vào Port groups → Add port group

Name: vSAN Network

Switch: vSwitch1

VLAN ID: 0 (hoặc VLAN riêng nếu có)

Vào VMkernel adapters → Add VMkernel

Port Group: vSAN Network

Enable: ✅ vSAN traffic

IP: 10.10.20.x (ví dụ)

Subnet: 255.255.255.0

Kiểm tra:

esxcli vsan network list


→ thấy vmk1 có Traffic Type: vsan là OK.

🟩 Làm tương tự cho cả 3 host (10.10.20.11 / 12 / 13 chẳng hạn).

🧱 PHẦN 2 — TẠO CLUSTER & BẬT vSAN

Mở vSphere Client → Hosts and Clusters

Chuột phải Datacenter → New Cluster

Nhập tên: vSAN-Cluster

Tick:

✅ vSAN

✅ DRS (nếu muốn)

✅ HA (nếu muốn)

Thêm host:

Add từng ESXi host (10.10.10.11, .12, .13)

Enable vSAN:

Vào Cluster → Configure → vSAN → Services → Configure

Chọn:

vSAN type: Standard

Deduplication/Compression: Off (lab)

Encryption: Off (lab)

Bấm Next

Disk Claiming:

Chọn từng host → SSD cho Cache tier → HDD cho Capacity tier

Bấm Finish

🟢 Chờ vài phút → vSAN sẽ tự tạo datastore vsanDatastore.

🧩 PHẦN 3 — KIỂM TRA MẠNG & HEALTH

SSH vào từng host:

esxcli vsan network list


→ đảm bảo mỗi host có 1 vmkX với Traffic Type: vsan.

Trong vCenter:

Cluster → Monitor → vSAN → Skyline Health

Bấm Retest

Tất cả phải “Green”.

Nếu thấy cảnh báo như:

“vSAN HCL DB up-to-date” → không có Internet, bỏ qua.

“Online health connectivity” → lab không cần Internet, bỏ qua.

📊 PHẦN 4 — KIỂM TRA HIỆU NĂNG (TÙY CHỌN)

Bật Performance Service:

Cluster → Configure → vSAN → Services

Bật ✅ “Performance Service”

Chọn Storage Policy mặc định.

Xem biểu đồ ở:

Cluster → Monitor → vSAN → Performance

💾 PHẦN 5 — KIỂM TRA KHA NĂNG LƯU TRỮ

Vào Datastores → vsanDatastore

Tạo thử 1 VM → đặt trên vSAN Datastore

Vào Cluster → vSAN → Monitor → Resyncing Objects
→ đảm bảo không có lỗi “Degraded”.

🧹 PHẦN 6 — KHẮC PHỤC LỖI PHỔ BIẾN
Lỗi	Nguyên nhân	Cách khắc phục
vSAN Cluster Configuration Consistency	1 host chưa join đúng network vSAN	Kiểm tra esxcli vsan network list
vSAN HCL DB up-to-date	Không có Internet	Bỏ qua hoặc import HCL offline
vSAN build recommendation	Không tải được bản release catalog	Bỏ qua (lab)
vSAN network alarm	VMkernel chưa bật “vSAN traffic”	Bật lại trên đúng vmk
✅ KẾT QUẢ CUỐI

vsanDatastore xuất hiện.

Tất cả vSAN Health = Green.

Mạng riêng 10.10.20.x chạy ổn định.

Có thể tạo VM và lưu trực tiếp vào vSAN.

Nếu bạn muốn, mình có thể viết luôn phiên bản “tự động bằng CLI (esxcli)” — để bạn chỉ cần SSH vào 3 host, chạy script là xong (không cần click giao diện).
👉 Bạn có muốn mình viết bản đó không?