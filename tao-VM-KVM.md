# Hướng dẫn cài máy ảo trên KVM

## Mục lục

###  [1. Cài máy ảo bằng virt-install](virt-install)

### [2. Cài máy ảo bằng virt-manager](virt-manager)

### [3. Cài máy ảo bằng webvirt](webvirt)

---



### <a name ="virt-install"> 1. Cài máy ảo bằng virt-install </a>

**Tạo VM từ image có sẵn**

- Tải file image (giống file ghost) về để khởi động, ví dụ này sẽ images linux được thu gọn. File được đặt trong thư mục chứa images của KVM (thư mục `/var/lib/libvirt/images`) bằng câu lệnh:

  `cd /var/lib/libvirt/images wget wget https://ncu.dl.sourceforge.net/project/gns-3/Qemu%20Appliances/linux-microcore-3.8.2.img`

- Tạo VM từ images

  ```sh
  sudo virt-install \
        -n VM01 \
        -r 128 \
         --vcpus 1 \
        --os-variant=generic \
        --disk path=/var/lib/libvirt/images/linux-microcore 3.8.2.img,format=qcow2,bus=virtio,cache=none \
        --network network=br0 \
        --hvm --virt-type kvm \
        --vnc --noautoconsole \
        --import
  ```

  ​

**Tạo VM từ file ISO**

- ```sh
  virt-install \
   -n myRHELVM1 \
   --description "Test VM with RHEL 6" \
   --os-type=Linux \
   --os-variant=rhel6 \
   --ram=2048 \
   --vcpus=2 \
   --disk path=/var/lib/libvirt/images/myRHELVM1.img,bus=virtio,size=10 \
   --graphics none \
   --cdrom /var/rhel-server-6.5-x86_64-dvd.iso \
   --network bridge:br0
  ```

  ​

**Tạo VM bằng cách tải từ trên internet**

-  ```sh
  virt-install \
  --name template \
  --ram 2048 \
  --disk path=/var/kvm/images/template.img,size=30 \
  --vcpus 2 \
  --os-type linux \
  --os-variant ubuntutrusty \
  --network bridge=br0 \
  --graphics none \
  --console pty,target_type=serial \
  --location 'http://jp.archive.ubuntu.com/ubuntu/dists/trusty/main/installer-amd64/' \
  --extra-args 'console=ttyS0,115200n8 serial'
  ```

  ​

**Những thông số đáng chú ý**

- --name: Tên máy ảo

- --ram: RAM của máy ảo

- --disk path=xxx, size=xxx

  'path=' => nơi lưu file ổ cứng của máy ảo

  'size=' => số lượng file ổ cứng của máy ảo

- --vcpus : số lượng CPU của máy ảo

- --os-type : loại OS

- --os-variant : tên OS

- --network : loại mạng mà VM sử dụng

- --graphics : loại graphics

- --console : loại console

- --location : nơi lưu file cài đặt

- --cdrom nơi chứa file ISO để cài đặt

### <a name ="virt-manager"> 2. Tạo máy ảo bằng virt-manager </a>

- Tiến hành cài đặt công cụ virt-manager để quản lí máy ảo bằng câu lệnh

  `apt-install virt-manager`

- Sau khi đã cài đặt virt-manager, tiến hành tải file ISO từ trên mạng internet và để vào thư mục mặc định đó là:

  `/var/lib/libvirt/images/`

- Hoặc bạn cũng có thể tải các file image từ internet về và để trong thư mục trên bằng câu lệnh sau:

  ```sh
  cd /var/lib/libvirt/images/

  wget https://ncu.dl.sourceforge.net/project/gns-3/Qemu%20Appliances/linux-microcore-3.8.2.img

  wget http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img
  ```

  Hai file images trên sẽ được dùng để tạo máy ảo mà không cần phải cài từ đầu(giống các file ghost)

- Tiến hành kích hoạt virt-manager bằng câu lệnh:

  `virt-manager`

- Lưu ý: Nếu bạn sử dụng MobaXterm sẽ hỗ trợ Forward X11 mặc định. Trong trường hợp bạn sử dụng Putty, cần tiến hành các bước sau:

  - Tải Xming tại địa chỉ https://sourceforge.net/projects/xming/

  - Cài đặt Xming, trong quá trình cài đặt, để mặc định các tùy chọn

  - Khởi động Xming

  - Khởi động Putty, cấu hình để kích hoạt X11 phía client theo các thao tác `Connection` => `SSH` => `X11` 

    <img src="https://github.com/hocchudong/KVM-QEMU/raw/master/hinhanh/img1.png">

  - Thực hiện nhập IP của máy chủ vào bằng tài khoản root

  - Khởi động virt-manager bằng câu lệnh `virt-manager`

- Sau khi thực hiện lệnh phía trên, một cửa sổ quản lí KVM sẽ hiện ra cho người dùng

  <img src="http://i.imgur.com/B3v0wMa.png">

- Tiến hành tạo máy ảo tại đây bằng cách:

  - Tại cửa sổ `Virtual Machine Manager` chọn `new`

    <img src="http://i.imgur.com/zpoxyRh.png">

  - Nhập tên máy ảo và lựa chọn cách cài máy ảo
    - Nếu bạn cài từ file ISO, hãy chọn `Local install media`

    - Nếu bạn cài từ file images, hãy chọn `Import existing disk image`

    - Hai lựa chọn còn lại dành cho việc cài đặt bằng cách tải từ trên mạng internet

      <img src="http://i.imgur.com/hf2Xebc.png">

  - Ở đây tôi chọn kiểu cài từ file ISO, lựa chọn `Use ISO image` chọn `Browse` để tìm tới file ISO đã có sẵn

    <img src="http://i.imgur.com/dpCkQkI.png">

  - Chọn RAM và CPU cho máy ảo

    <img src="http://i.imgur.com/H6NVdyJ.png">

  - Chọn dung lượng ổ cứng cho máy ảo

    <img src="http://i.imgur.com/9QfbdDn.png">

  - Kết thúc quá trình tạo máy ảo, chọn `Customize configuration before install`để có thể thay đổi những cài đặt trước khi bắt đầu cài OS.

    <img src="http://i.imgur.com/MYY73Dc.png">

  - Tại đây, bạn có thể thay đổi những thông số mà mình đã chọn vd như: RAM, CPU, Card mạng...

    <img src="http://i.imgur.com/ouHbPLX.png">

  - Nhấn `Begin Installation` để bắt đầu