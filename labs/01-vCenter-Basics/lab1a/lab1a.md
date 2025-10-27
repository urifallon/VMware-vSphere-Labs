# Lab 1A: Cài đặt VMware vSphere

## I. Khởi tạo tài nguyên (VMware)

### 1. Phần cứng
![Phần cứng](/img/lab1a-phancung.png)

### 2. Mạng
![Mạng](/img/lab1a-mang.png)

## II. Thiết lập

### 1. Cài đặt máy ảo ESXi 

![Bước 1](/img/lab1a-buoc1.png)

### 2. Thiết lập trong VM

![Bước 2](/img/lab1a-buoc2.png)

**Lưu ý:** Bấm `F2` → Nhập mật khẩu bạn đã thiết lập khi cài ISO

![Bước 3](/img/lab1a-buoc3.png)

**Hướng dẫn:** Bấm mũi tên xuống → `Enter` chọn `Configure Management Network`

![Bước 4](/img/lab1a-buoc4.png)

**Hướng dẫn:** Bấm mũi tên xuống → `Enter` chọn `IPv4 Configuration`

![Bước 5](/img/lab1a-buoc5.png)

**Các tùy chọn IPv4:**
- **Disable IPv4 configuration for management network:** Tắt không sử dụng IPv4
- **Use dynamic IPv4 address and network configuration:** Cho phép sử dụng IP được cung cấp từ DHCP của VMware
- **Set static IPv4 address and network configuration:** Thiết lập IP tĩnh (trong dự án này sẽ sử dụng IP tĩnh)

![Bước 6](/img/lab1a-buoc6.png)

**Hướng dẫn:** Nhập các trường thông tin mạng bạn muốn thiết lập

![Bước 7](/img/lab1a-buoc7.png)

**Lưu ý:** Tương tự IPv4 nhưng vì không sử dụng nên sẽ Disable

![Bước 8](/img/lab1a-buoc8.png)

**Thiết lập DNS:**
- Bạn có thể sử dụng một máy chủ DNS riêng
- Sử dụng `10.10.10.1` nếu đang làm trong lab mà không muốn có thêm một VM
- Đặt hostname để dễ nhận diện máy chủ

**Kết thúc:** Sau khi thiết lập xong các bước → bấm `Esc` → bấm phím `Y`

### 3. Thiết lập vCenter

![Bước 9](/img/lab1a-buoc9.png)

**Truy cập ESXi:** Sử dụng link đã được cấu hình, ví dụ: `https://h1/` hoặc `https://10.10.10.101`

![Bước 10](/img/lab1a-buoc10.png)

**Thông tin đăng nhập:**
- Tên tài khoản mặc định: `root`
- Mật khẩu: Mật khẩu khi bạn thiết lập cài đặt hệ điều hành ESXi

![Bước 11](/img/lab1a-buoc11.png)

**Hướng dẫn:** Chọn `Virtual Machines` → `Create / Register VM`

![Bước 12](/img/lab1a-buoc12.png)

**Lưu ý:** Ở đây tôi chọn tùy chọn cài đặt vCenter từ file OVA

![Bước 13](/img/lab1a-buoc13.png)

**Tải file và tiến hành cài đặt từng mục như sau:**

#### Select the OVF and VMDK files:
| Trường | Giá trị |
|--------|---------|
| Enter a name for the virtual machine | Điền tên máy vCenter |
| Click to select files | Click tải lên file OVA |

→ `Next`

#### Select storage
→ `Next`

#### License agreements
→ `I AGREE` → `I AGREE`

#### Deployment options
| Trường | Giá trị |
|--------|---------|
| Network mappings | VM Network |
| Deployment type | Tiny vCenter Server with Embedded PSC |
| Disk provisioning | Thin |
| Power on automatically | ✓ Check |

→ `Next`

#### Additional settings

**Networking Configuration:**
| Trường | Giá trị |
|--------|---------|
| Host Network IP Address Family | IPv4 (hoặc IPv6 nếu bạn sử dụng) |
| Host Network Mode | Static (hoặc DHCP nếu bạn muốn cấp IP tự động) |
| Host Network IP Address | 10.x.x.x (IP bạn sẽ thiết lập cho trang máy chủ vSphere) |
| Host Network Prefix | 24 (subnet mask) |
| Host Network Default Gateway | Gateway của bạn (mặc định thường là octet 1 trong dải mạng, ví dụ: 10.10.10.1) |
| Host Network DNS Servers | Địa chỉ máy chủ DNS của bạn (chưa có có thể điền IP của chính máy đó) |
| Host Network Identity | Domain của vSphere (ví dụ: vcenter.lab.local) |

→ `Next`

**SSO Configuration:**
| Trường | Giá trị |
|--------|---------|
| Directory Password | Mật khẩu của trang quản trị vSphere |
| Directory Password confirm | Nhập lại mật khẩu |

→ `Next`

**System Configuration:**
| Trường | Giá trị |
|--------|---------|
| Root Password | Mật khẩu của máy chủ vSphere |
| Root Password confirm | Nhập lại mật khẩu |

→ `Next`

**Các cài đặt khác:** Có thể tạm thời bỏ qua
→ `Next` → `Finish`

![Bước 14](/img/lab1a-buoc14.png)

**Lưu ý:** Đợi vài phút cho máy cài đặt và khởi chạy

![Bước 15](/img/lab1a-buoc15.png)

**Truy cập:** Sử dụng đường link đã cấu hình (hoặc với IP của nó đi kèm port, ví dụ: `https://10.10.10.10:5480` hoặc `https://vcenter.lab.local:5480`)

![Bước 16](/img/lab1a-buoc16.png)

**Hướng dẫn:** Nhập mật khẩu của máy chủ vSphere

![Bước 17](/img/lab1a-buoc17.png)

**Hướng dẫn:** Chọn `Setup`

#### Setup Wizard

**vCenter Server Configuration:**
| Trường | Giá trị |
|--------|---------|
| Network configuration | Mặc định |
| IP version | Mặc định |
| System name | Mặc định |
| IP address | Mặc định |
| Subnet mask or prefix length | Mặc định |
| Default gateway | Mặc định |
| DNS servers | Mặc định |
| Time synchronization mode | Synchronize time with the ESXi host (hoặc Synchronize time with the NTP servers) |
| SSH access | Active (Nên để để có thể truy cập tìm lỗi và sửa) |

→ `Next`

**SSO Configuration (Create a new SSO domain):**
| Trường | Giá trị |
|--------|---------|
| Single Sign-On domain name | Đặt domain cho tài khoản truy cập (ví dụ: Nếu bạn đặt là vsphere.local → tên tài khoản của bạn sẽ là Administrator@vsphere.local) |
| Single Sign-On password | Mật khẩu của trang quản trị vSphere |
| Confirm password | Nhập lại mật khẩu |

→ `Next`

**Configure CEIP:**
→ `Next` → `Finish`

### 4. Hoàn tất cài đặt và cấu hình vCenter

![Bước 18](/img/lab1a-buoc18.png)

**Lưu ý:** Đợi đến khi cài đặt hoàn tất

![Bước 19](/img/lab1a-buoc19.png)

**Truy cập vSphere Client:**
- Sau khi cài đặt xong, vào trang quản trị vSphere bằng domain (IP address bạn đã đặt cho vSphere, ví dụ: `https://10.10.10.10/`)
- Đợi đến khi cài đặt hoàn tất
- Click `Launch Vsphere Client`

![Bước 20](/img/lab1a-buoc20.png)

**Tạo datacenter mới:** Right Click `vcenter.lab.local`

![Bước 21](/img/lab1a-buoc21.png)

**Đăng nhập:** Sử dụng tài khoản SSO của bạn

![Bước 22](/img/lab1a-buoc22.png)

**Tạo Datacenter:** New datacenter → Đặt tên cho DC

![Bước 23](/img/lab1a-buoc23.png)

**Đặt tên cho Cluster**

![Bước 24](/img/lab1a-buoc24.png)

**Cấu hình Cluster:**

| Tính năng | Trạng thái |
|-----------|------------|
| vSphere DRS | Disable |
| vSphere HA | Disable |
| vSAN | Disable |
| Enable vSAN ESA | Uncheck |
| Manage all hosts in the cluster with a single image | Uncheck |
| Manage configuration at a cluster level | Uncheck |

→ `Next` → `Finish`

### 5. Thêm ESXi Host vào Cluster

![Bước 25](/img/lab1a-buoc25.png)

**Hướng dẫn:** Cluster → `Configure` → `Quick Start` → `Add Host` → `ADD`

**⚠️ Lưu ý quan trọng:** Hãy chắc chắn bước này bạn có 3 thiết bị ESXi trở lên (nếu bạn chỉ có 1 cái, hãy thiết lập thêm 2 cái ESXi với địa chỉ IP khác - tham khảo phần <a href="#1-cài-đặt-máy-ảo-esxi">### 1. Cài đặt máy ảo ESXi</a>)

![Bước 26](/img/lab1a-buoc26.png)

**Thông tin kết nối:**

| Trường | Giá trị |
|--------|---------|
| IP address or FQDN | Nhập IP hoặc tên miền máy chủ (nếu bạn có DNS) |
| Username | root (mặc định của máy chủ ESXi) |
| Password | Mật khẩu bạn đặt cho máy chủ ESXi |

→ `Next`

**Xác nhận:** Tích chọn các máy chủ → `OK`

**Hoàn tất:** Host summary → `Next` → `Finish`

**⚠️ Xử lý lỗi License:** Nếu gặp lỗi license khi thêm ESXi host, hãy sử dụng các license keys có sẵn tại: https://gist.github.com/urifallon/ecefb8e4a8473174aca3d6adc67fdfe5

### 6. Xác nhận và kiểm tra kết quả

![Bước 27](/img/lab1a-buoc27.png)

**Kiểm tra trạng thái Host:** 
- Add hosts: Not configured hosts: 3 → Đã thêm thành công

![Bước 28](/img/lab1a-buoc28.png)

**Thoát Maintenance Mode:** Host → Right-click → Maintenance Mode → Exit Maintenance Mode

## III. Kiểm tra và xác nhận

### Kết quả mong đợi

Sau khi hoàn thành các bước trên, bạn có thể:

1. **Truy cập vSphere Client:** Sử dụng địa chỉ `https://[IP-vCenter]/ui`
2. **Đăng nhập:** Sử dụng tài khoản `Administrator@[domain]` và mật khẩu đã thiết lập
3. **Kiểm tra kết nối:** Xác nhận ESXi host đã được quản lý bởi vCenter

### Cấu trúc Inventory

**Kết quả mong đợi:** Inventory hiển thị cấu trúc: **vCenter > Datacenter > Cluster > ESXi Host**

Bạn có thể kiểm tra bằng cách:
- Xem resource pools và storage/network liên kết
- Kiểm tra trạng thái của các ESXi host
- Xác nhận tất cả host đã thoát khỏi Maintenance Mode

### Xử lý sự cố

Nếu gặp lỗi kết nối, hãy kiểm tra:

1. **Firewall trên ESXi:**
   ```bash
   esxcli network firewall ruleset set --ruleset-id=syslog --enabled=true
   ```

2. **DNS Resolution:** Đảm bảo các host có thể resolve được tên miền của vCenter

3. **Network Connectivity:** Kiểm tra kết nối mạng giữa vCenter và ESXi hosts

### Mục tiêu Lab

Lab này chứng minh vCenter là trung tâm quản lý, cho phép:
- Giám sát tập trung tất cả ESXi hosts
- Cấu hình và quản lý cluster
- Tối ưu hóa tài nguyên và hiệu suất hệ thống

## IV. Lưu ý quan trọng

- **Bảo mật:** Luôn sử dụng mật khẩu mạnh cho tất cả tài khoản
- **Backup:** Thực hiện backup cấu hình sau khi thiết lập thành công
- **Monitoring:** Thiết lập monitoring để theo dõi trạng thái hệ thống
- **Documentation:** Ghi lại tất cả thông tin cấu hình để tham khảo sau này
