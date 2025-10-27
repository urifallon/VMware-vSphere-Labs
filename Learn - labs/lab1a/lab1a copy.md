I. Khởi tạo tài nguyên (VMware)
1. phan cứng
![phancung](/img/lab1a-phancung.png)

2. mạng
![mang](/img/lab1a-mang.png)

II. Thiết lập
1. Cài đặt máy ảo 

![buoc1](/img/lab1a-buoc1.png)

2. Thiết lập mạng trong vm

![buoc2](/img/lab1a-buoc2.png)

> Bấm `F2` -> Nhập mật khẩu bạn đã thiết lập khi cài iso

![buoc3](/img/lab1a-buoc3.png)

> Bấm mũi tên xuống -> `Enter` chọn `Configure Management Network` 

![buoc4](/img/lab1a-buoc4.png)

> Bấm mũi tên xuống -> `Enter` chọn `IPv4 Configuration` 

![buoc5](/img/lab1a-buoc5.png)

> Bấm nút xuống để di chuyển, space để chọn
> Disable IPv4 configuration for management network: Bạn sẽ tắt không sử dụng IPv4
> Use dynamic IPv4 address and network configuration: cho phép bạn sử dụng ip được cung cấp từ DHCP của VMware.
> Set static IPv4 address and network configuration: Lựa chọn này cho phép bạn thiết lập IP tĩnh (trong dự án này tôi sẽ sử dụng ip tĩnh và được thiết lập như sau).

![buoc6](/img/lab1a-buoc6.png)

> Nhập các trường thông tin mạng bạn muốn set

![buoc7](/img/lab1a-buoc7.png)

> Tương tự IPv4 nhưng vì không sử dụng nên thôi sẽ Disable

![buoc8](/img/lab1a-buoc8.png)

> Thiết lập DNS cho máy
> Bạn có thể dụng một máy chủ DNS riêng, sử dụng 10.10.10.1 nếu đang làm trong lab mà không muốn có thêm một vm
> Đặt hostname để dễ nhận diện máy chủ

> Sau khi thiết lập xong các bước -> bấm `esc` -> bấm phím `Y`

3. Thiết lập vCenter

![buoc9](/img/lab1a-buoc9.png)

> Truy cập esxi bằng link được do mình đã cấu hình, ở đây của tôi là `https://h1/` hoặc `https://10.10.10.101`


![buoc10](/img/lab1a-buoc10.png)

> Tên tài khoản mặc định là root
> Mật khẩu khi bạn thiết lập cài đặt hệ điều hành esxi

![buoc11](/img/lab1a-buoc11.png)

> Chọn `Virtual Machines` -> `Create / Register VM`

![buoc12](/img/lab1a-buoc12.png)

> Ở đây tôi chọn tùy chọn cài đặt vCenter từ file OVA, bạn có thể xem thêm thông tin tải về tại đây ....

![buoc13](/img/lab1a-buoc13.png)
Tải file và tiến hành cài đặt từng mục như sau:
- Select the OVF and VMDK files: 
| Enter a name for the virtual machine.  | Điền tên máy vcenter|
| Click to select files | Click tải lên file OVA |
> `Next`

- Select storage
> `Next`

- License agreements
> `I AGREE` -> `I AGREE`

- Deployment options
| Network mappings | VM Network |
| Deployment type | Tiny vCenter Server with Embedded PSC |
| Disk provisioning | Thin |
| Power on automatically | Check |
> `Next`

- Additional settings
	+ Networking Configuration 
		| Host Network IP Address Family | IPv4 (hoặc ipv6 nếu bạn sử dụng)| 
		| Host Network Mode | static (hoặc dhcp nếu bạn muốn cấp ip tự động) |
		| Host Network IP Address | 10.x.x.x (đây là ip bạn sẽ thiết lập cho trang máy chủ vsphere)|
		| Host Network Prefix | 24 (subnet mask)|
		| Host Network Default Gateway | Điền gateway của bạn (mặc định thường là otec 1 trong dải mạng vd: 10.10.10.1)|
		| Host Network DNS Servers | Điền địa chỉ máy chủ DNS của bạn (chưa có có thể điền ip của chính máy đó) |
		| Host Network Identity | Domain của vsphere (vd: vcenter.lab.local)|  
	> `Next`
	+ SSO Configuration
		| Directory Password | Mật khẩu của trang quản trị vsphere|
		|  Directory Password confirm | Nhập lại mật khẩu |
	> `Next`
	+ System Configuration
		| Root Password | Mật khẩu của máy chủ vsphere |
		|  Root Password confirm | Nhập lại mật khẩu |
	> `Next`
	+ Những cái còn lại có thể tạm thời bỏ qua
	> `Next` -> `Finish`

![buoc14](/img/lab1a-buoc14.png)
> Đợi vài phút cho máy cài đặt và khởi chạy

![buoc15](/img/lab1a-buoc15.png)
> Truy cập theo đường link đã cấu hình (hoặc với ip của nó đi kèm port, vd: https://10.10.10.10:5480 hoặc https://vcenter.lab.local:5480)

![buoc16](/img/lab1a-buoc16.png)
> Nhập mật khaair của máy chủ vsphere

![buoc17](/img/lab1a-buoc17.png)
> Chọn `Setup`

- Setup Wizard
	+ vCenter Server Configuration
		| Network configuration | Mặc định |
		| IP version | Mặc định |
		| System name | Mặc định |
		| IP address | Mặc định |
		| Subnet mask or prefix length | Mặc định |
		| Default gateway | Mặc định |
		| DNS servers | Mặc định |
		| Time synchronization mode | Synchronize time with the ESXi host (hoặc Synchronize time with the NTP servers )
		| SSH access | Active (Nên để để có thể truy cập tìm lỗi và sửa)|
	> `Next`
	+ SSO Configuration (Create a new SSO domain)
		| Single Sign-On domain name | đặt đồ main cho tài khoản truy cập (vd: Nếu bạn đặt là vsphere.local -> tên tài khoản của bạn sẽ là Administrator@vsphere.local) |
		| Single Sign-On password |  Mật khẩu của trang quản trị vsphere |
		| Confirm password | Nhập lại mật khẩu |
	> `Next`
	+  Configure CEIP
	> `Next` -> `Finish`






