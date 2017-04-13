# Hướng dẫn cài đặt KVM

- Bước 1:  Để cài đặt KVM, CPU của máy phải hỗ trợ ảo hóa phần cứng Intel VT-x hoặc AMD-V. 
  Để xác định CPU có những tính năng này không, thực hiện lệnh sau:

`egrep -c '(svm|vmx)' /proc/cpuinfo`

Giá trị 0 chỉ thị rằng CPU không hỗ trợ ảo hóa phần cứng trong khi giá trị khác 0 chỉ thị có hỗ trợ. 
Người dùng có thể vẫn phải kích hoạt chức năng hỗ trợ ảo hóa phần cứng trong BIOS của máy kể cả khi câu lệnh này trả về giá trị khác 0.

<img src="http://i.imgur.com/Cs2zowA.png">

Ở đây, giá trị trả về của mình là 1.

Nếu bạn sử dụng VMWare, hãy nhớ bật chức năng ảo hóa Intel VT-x/EPT hoặc AMD-V/RVI trong phần `Processors`

<img src="http://i.imgur.com/PIngLij.png">

- Bước 2: Sử dụng câu lệnh sau để cài đặt KVM và các gói phụ trợ liên quan

`sudo apt-get install qemu-kvm libvirt-bin bridge-utils`

- Trong đó:
  <ul>
  <li>libvirt-bin: cung cấp libvirt mà bạn cần quản lý qemu và kvm bằng libvirt</li>
  <li>qemu-kvm: Phần phụ trợ cho KVM</li>
  <li>bridge-utils: chứa một tiện ích cần thiết để tạo và quản lý các thiết bị bridge.</li>
  </ul>

- Bước 3: Chỉ quản trị viên (root user) và những người dùng thuộc libvirtd group có quyền sử dụng máy ảo KVM. 
  Chạy lệnh sau để thêm tài khoản người dùng vào libvirtd group:

`sudo adduser Username libvirtd`

- Bước 4: Đăng xuất rồi đăng nhập trở lại. Nhập câu lệnh sau sau khi đăng nhập:

`virsh --connect qemu:///system list`

<img src ="http://i.imgur.com/NUNueTd.png">

Một danh sách máy ảo còn trống xuất hiện. Điều này thể hiện mọi thứ đang hoạt động đúng.

- Bước 4: Cấu hình Linux Bridge

Đầu tiên, hãy chạy câu lệnh sau để tạo một Bridge mới:

`sudo brctl addbr br0`

Sau đó, tiến hành gán card mạng cho bridge đó

`sudo brctl addif br0 Ten_card_mang`

Để lấy tên card mạng, tiến hành chạy lệnh:

`sudo ip addr show`

Ở đây tôi gán card trong máy là ens33

Cấu hình Linux Bridge trong `/etc/network/interfaces` như sau:

<img src="http://i.imgur.com/j1C9XYR.png">

- Một vài câu lệnh cần lưu ý:
  <ul>
  <li>bridge_ports none: nếu bạn không muốn gán bất cứ port nào vào bridge</li>
  <li>bridge_stp off: tắt chế độ STP (Spanning Tree Protocol)</li>
  <li>bridge_fd 0: không delay forwarding</li>
  </ul>

- Khởi động lại dịch vụ mạng bằng câu lệnh: 
   `sudo /etc/init.d/networking restart`

- Kiểm tra lại bridge bằng câu lệnh:

`sudo brctl show`

<img src="http://i.imgur.com/sKm4Yxg.png">