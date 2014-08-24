NetFlow-sFlow
=============

Sử dụng công cụ NetFlow monitor Open vSwitch

# 1. Giới thiệu NetFlow

Cisco NetFlow là một công nghệ do Cisco phát triển với nhiều tính năng giúp chúng ta có thể giám sát băng thông của mạng thông qua công cụ NetFlow Analyzer.

# 2. Bài Lab sử dụng NetFlow để monitor Open vSwitch

### 2.1. Mục tiêu bài lab
- Moitor lưu lượng mạng gửi tới và đi của VM1, VM2 sử dụng công nghệ NetFlow
- Tìm hiểu được cách sử dụng công cụ NetFlow để monitor Open vSwitch
- Hiểu được các kết quả trả về của công cụ NetFlow

### 2.2. Mô hình thực hiện

<img src=http://i.imgur.com/EW5DHrj.png>

Trong mô hình này:

- Host 1: máy ảo Ubuntu được cài Open vSwitch trên công nghệ ảo hoá VMWare Workstation với hai card mạng như hình sau:
<img src=http://i.imgur.com/Uq02EEs.png>
Card đầu tiên (eth0) là card bridge được nối ra ngoài.
Card thứ hai (eth1) được nối với máy vật lý dùng làm máy monitor qua card mạng ảo vmnet1.

- Monitoring host là máy vật lý, hệ điều hành Windows 7, dùng card vmnet1 nối với eth1 của máy ảo.

### 2.3. Kịch bản bài Lab

- Cài đặt Open vSwitch trên máy ảo VMWare Workstation.
- Cài KVM và cài đặt hai máy ảo trên đó.
- Sử dụng công cụ NetFlow để monitor OVS.

### 2.4. Quá trình thực hiện

#### 2.4.1. Trên host 1

Cài đặt các gói sau:
    apt-get install -y openvswitch-switch openvswitch-datapath-dkms

Tạo brigde br0
    sudo ovs-vsctl add-br br0
    
Thêm port eth0 vào br0
    ovs-vsctl add-port br0 eth0
    
Sau khi thực hiện câu lệnh trên thì card eth0 sẽ mất kết nối, ta cần xóa IP của card eth0 và lấy IP đó đặt cho card br0

    ifconfig eth0 0.0.0.0
    ifconfig br0 192.168.1.116/24
    
Tạo hai máy ảo:

    wget http://cloudhyd.com/openstack/images/cirros-0.3.0-x86_64-disk.img
    mv cirros-0.3.0-x86_64-disk.img cirros1.img
    cp cirros1 cirros2
    kvm -m 512 -net nic,macaddr=12:42:52:CC:CC:12 -net tap cirros1 –nographic
    kvm -m 512 -net nic,macaddr=12:42:52:CC:CC:13 -net tap cirros2 –nographic

Sau khi tạo mỗi VM thì sẽ bị mất phiên ssh vào máy chủ, thay vào đó sẽ là phiên của mỗi máy ảo, ta cần mở một kết nối khác tới máy chủ để cấu hình.

Ta tạo một file mới có tên là netflow.sh với nội dung như sau:

    ovs-vsctl -- -- set Bridge br0 netflow=@nf -- --id=@nf create NetFlow targets=\"10.0.0.1:9996\" active-timeout=60

Với 10.0.0.1 là IP của monitoring host, eth1 là card mạng nối với monitoring host.

    chmod +x netflow.sh
    ./netflow.sh

Cấu hình xóa NetFlow trên OpenvSwitch

    ovs-vsctl -- clear Bridge br-ex netflow

Liệt kê các NetFlow

    ovs-vsctl list netflow


#### 2.4.2. Trên host Monitoring

Download bộ cài đặt của NetFlow Analyzer tại [địa chỉ] (http://www.manageengine.com/products/netflow/download.html)

Cài đặt bình thường với các tham số mặc định như cổng nghe là 9996, cổng quản lý là 8080

Truy cập vào điạ chỉ **http://localhost:8080 ** với username và password admin

Giao diện của chương trình:

<img src=http://i.imgur.com/H3WtoOX.png>

