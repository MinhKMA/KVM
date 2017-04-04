# Tìm hiểu KVM

## Mục lục

### [1. Khái niệm và vai trò](#vai-tro)

### [2. Cấu trúc](#cau-truc)

### [3. Mô hình triển khai](#mo-hinh)

### [4. Cơ chế hoạt động](#co-che)

### [5. Lab](#lab)

### [6. Tính năng](#tinh-nang)

-------

### <a name = "vai-tro"></a> 1. Khái niệm và vai trò.

- KVM (Kernel-based virtual machine) là giải pháp ảo hóa cho hệ thống linux trên nền tảng phần cứng x86 có các module mở rộng hỗ trợ ảo hóa (Intel VT-x hoặc AMD-V). 
Về bản chất, KVM không thực sự là một hypervisor có chức năng giải lập phần cứng để chạy các máy ảo. 
Chính xác KVM chỉ là một module của kernel linux hỗ trợ cơ chế mapping các chỉ dẫn trên CPU ảo (của guest VM) 
sang chỉ dẫn trên CPU vật lý (của máy chủ chứa VM). 
Hoặc có thể hình dung KVM giống như một driver cho hypervisor để sử dụng được tính năng ảo hóa của các vi xử lý như Intel VT-x 
hay AMD-V, mục tiêu là tăng hiệu suất cho guest VM.

- KVM ban đầu được phát triển bởi Qumranet – một công ty nhỏ, sau đó được Redhat mua lại vào tháng 9 năm 2008. 
Ta có thể thấy KVM là thế hệ tiếp theo của công nghệ ảo hóa. KVM được sử dụng mặc định từ bản RHEL (Redhat Enterprise Linux) 
từ phiên bản 5.4 và phiên bản Redhat Enterprise Virtualization dành cho Server.

- Qumranet phát hành mã của KVM cho cộng đồng mã nguồn mở. Hiện nay, các coog ty nổi tiếng như IBM, 
Intel và ADM cũng đã cộng tác với dự án. Từ phiên bản 2.6.20, KVM trở thành một phần của hạt nhân Linux.


- Bảng so sánh KVM với VMWare:

<img src="http://i.imgur.com/SRDtP40.png">

### <a name = "cau-truc"></a> 2. Cấu trúc của KVM

- Trong kiến trúc KVM, máy ảo là một tiến trình Linux, được lập lịch bởi chuẩn Linux scheduler. 
Trong thực tế mỗi CPU ảo xuất hiện như là một tiến trình Linux. Điều này cho phép KVM sử dụng tất cả các tính năng của Linux kernel.
- Cấu trúc tổng quan:

<img src ="http://i.imgur.com/rnvsnoJ.png">

- Linux có tất cả các cơ chế của một VMM cần thiết để vận hành (chạy) các máy ảo. 
Chính vì vậy các nhà phát triển không xây dựng lại mà chỉ thêm vào đó một vài thành phần để hỗ trợ ảo hóa. 
KVM được triển khai như một module hạt nhân có thể được nạp vào để mở rộng Linux bởi những khả năng này.
- Trong một môi trường linux thông thường mỗi process chạy hoặc sử dụng user-mode hoặc kernel-mode. 
KVM đưa ra một chế độ thứ 3, đó là guest-mode. Nó dựa trên CPU có khả năng ảo hóa với kiến trúc Intel VT hoặc AMD SVM, 
một process trong guest-mode bao gồm cả kernel-mode và user-mode.

**Trong cấu trúc tổng quan của KVM bao gồm 3 thành phần chính:**
- KVM kernel module: 
  <ul>
  <li>là một phần trong dòng chính của Linux kernel.</li>
  <li>cung cấp giao diện chung cho Intel VMX và AMD SVM (thành phần hỗ trợ ảo hóa phần cứng)</li>
  <li>chứa những mô phỏng cho các instructions và CPU modes không được support bởi Intel VMX và AMD SVM</li>
  </ul>
- quemu - kvm: là chương trình dòng lệnh để tạo các máy ảo, thường được vận chuyển dưới dạng các package "kvm" hoặc "qemu-kvm".
  Có 3 chức năng chính:
  <ul>
  <li>Thiết lập VM và các thiết bị ra vào (input/output)</li>
  <li>Thực thi mã khách thông qua KVM kernel module</li>
  <li>Mô phỏng các thiết bị ra vào (I/O) và di chuyển các guest từ host này sang host khác</li>
  </ul>
- libvirt management stack:
  <ul>
  <li>cung cấp API để các tool như virsh có thể giao tiếp và quản lí các VM</li>
  <li>cung cấp chế độ quản lí từ xa an toàn</li>
  </ul>
### <a name ="mo-hinh"></a> 3. Mô hình triển khai
- Hình dưới đây mô tả mô hình thực hiện của KVM. Đây là một vòng lặp của các hành động diễn ra để vận hành các máy ảo. 
Những hành động này được phân cách bằng 3 phương thức chúng ta đã đề cập trước đó: user-mode, kernel-mode, guest-mode.

<img src="http://i.imgur.com/wG8E4s8.png">

Như ta thấy:

- User-mode: Các modul KVM gọi đến sử dụng ioclt() để thực thi mã khách cho đến khi hoạt động I/O khởi xướng bởi guest hoặc một 
sự kiện nào đó bên ngoài xảy ra. Sự kiện này có thể là sự xuất hiện của một gói tin mạng, cũng có thể là trả lời của một gói tin 
mạng được gửi bởi các may chủ trước đó. Những sự kiện như vậy được biểu diễn như là tín hiệu dẫn đến sự gián đoạn của thực thi mã khách.
- Kernel-mode: Kernel làm phần cứng thực thi các mã khách tự nhiên. Nếu bộ xử lý thoát khỏi guest do cấp phát bộ nhớ hay I/O hoạt động, 
kernel thực hiện các nhiệm vụ cần thiết và tiếp tục luồng thực hiện, Nếu các sự kiện bên ngoài như tín hiệu hoặc I/O hoạt động khởi 
xướng bởi các guest tồn tại, nó thoát tới user-mode.
- Guest-mode: Đây là cấp độ phần cứng, nơi mà các lệnh mở rộng thiết lập của một CPU có khả năng ảo hóa được sử dụng để thực thi mã 
nguồn gốc, cho đến khi một lệnh được gọi như vậy cần sự hỗ trợ của KVM, một lỗi hoặc một gián đoạn từ bên ngoài.

- Khi một máy ảo chạy, có rất nhiều chuyển đổi giữa các chế độ. Từ kernel-mode tới guest-mode và ngược lại rất nhanh, 
bởi vì chỉ có mã nguồn gốc được thực hiện trên phần cứng cơ bản. Khi I/O hoạt động diễn ra các luồng thực thi tới user-mode, 
rất nhiều thiết bị ảo I/O được tạo ra, do vậy rất nhiều I/O thoát ra và chuyển sang chế độ user-mode chờ. 
Hãy tưởng tượng mô phỏng một đĩa cứng và 1 guest đang đọc các block từ nó. Sau đó QEMU mô phỏng các hoạt động bằng cách giả 
lập các hoạt động bằng các mô phỏng hành vi của các ổ đĩa cứng và bộ điều khiển nó được kết nối. Để thực hiện các hoạt động đọc, 
nó đọc các khối tương ứng từ một tập tin lớp và trả về dữ liệu cho guest. Vì vậy, user-mode giả lập I/O có xu hướng xuất hiện một 
nút cổ chai làm chậm việc thực hiện máy ảo.

### <a name ="co-che"></a> 4. Cơ chế hoạt động

- Để các máy ảo giao tiếp được với nhau, KVM sử dụng Linux Bridge và OpenVSwitch, đây là 2 phần mềm cung cấp các giải pháp ảo hóa network.
Trong bài này, tôi sẽ sử dụng Linux Bridge.
- Linux bridge là một phần mềm đươc tích hợp vào trong nhân Linux để giải quyết vấn đề ảo hóa phần network trong các máy vật lý. 
Về mặt logic Linux bridge sẽ tạo ra một con switch ảo để cho các VM kết nối được vào và có thể nói chuyện được với nhau cũng như sử 
dụng để ra mạng ngoài.
- Cấu trúc của Linux Bridge khi kết hợp với KVM - QEMU.

<img src = "http://i.imgur.com/Krk1JRm.png">

- Ở đây:
  <ul>
  <li>Bridge: tương đương với switch layer 2</li>
  <li>Port: tương đương với port của switch thật</li>
  <li>Tap (tap interface): có thể hiểu là giao diện mạng để các VM kết nối với bridge cho linux bridge tạo ra</li>
  <li>fd (forward data): chuyển tiếp dữ liệu từ máy ảo tới bridge</li>
  </ul>

- Các tính năng chính:
  <ul>
  <li>STP: Spanning Tree Protocol - giao thức chống lặp gói tin trong mạng</li>
  <li>VLAN: chia switch (do linux bridge tạo ra) thành các mạng LAN ảo, cô lập traffic giữa các VM trên các VLAN khác nhau của cùng một switch.</li>
  <li>FDB (forwarding database): chuyển tiếp các gói tin theo database để nâng cao hiệu năng switch. 
  Database lưu các địa chỉ MAC mà nó học được. Khi gói tin Ethernet đến, bridge sẽ tìm kiếm trong database có chứa MAC address không. 
  Nếu không, nó sẽ gửi gói tin đến tất cả các cổng.</li
  </ul>

### <a name ="lab"></a> 5. Lab

**Hướng dẫn cài đặt KVM**

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

Trong đó:
  <ul>
  <li>libvirt-bin: cung cấp libvirt mà bạn cần quản lý qemu và kvm bằng libvirt</li>
  <li>qemu-kvm: Phần phụ trợ cho KVM</li>
  <li>bridge-utils: chứa một tiện ích cần thiết để tạo và quản lý các thiết bị bridge. 
  Điều này rất hữu ích trong việc thiết lập hệ thống mạng cho một máy ảo được lưu trữ.</li>
  </ul>

- Bước 3: Chỉ quản trị viên (root user) và những người dùng thuộc libvirtd group có quyền sử dụng máy ảo KVM. 
Chạy lệnh sau để thêm tài khoản người dùng vào libvirtd group:

`sudo adduser Username libvirtd`

- Bước 4: Đăng xuất rồi đăng nhập trở lại. Nhập câu lệnh sau sau khi đăng nhập:

`virsh --connect qemu:///system list`

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
