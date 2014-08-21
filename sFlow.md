sFlow
=============

Sử dụng công cụ sFlow monitor Open vSwitch

# 1. Giới thiệu sFlow  

SFlow là một công nghệ cho phép chúng ta có thể monitor switch tốc độ cao. Nó cung cấp một góc nhìn đầy đủ về mạng giúp tối ưu hiệu suất, kiểm toán và giúp ngăn chặn các nguy cơ tấn công.
Trong bài lab này tôi sẽ sử dụng sFlow để monitor một switch ảo là Open vSwitch.

# 2. Bài Lab sử dụng sFlow để monitor Open vSwitch

### 2.1. Mục tiêu bài lab
- Moitor lưu lượng mạng gửi tới và đi của VM1, VM2 sử dụng công nghệ sFlow
- Tìm hiểu được cách sử dụng công cụ sFlow để monitor Open vSwitch
- Hiểu được các kết quả trả về của công cụ sFlow

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
- Sử dụng công cụ sFlow để monitor OVS.

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
    kvm -m 512 -net nic,macaddr=12:42:52:CC:CC:12 -net tap cirros2 –nographic

Sau khi tạo mỗi VM thì sẽ bị mất phiên ssh vào máy chủ, thay vào đó sẽ là phiên của mỗi máy ảo, ta cần mở một kết nối khác tới máy chủ để cấu hình.

Ta tạo một file mới có tên là sflow.sh với nội dung như sau:

    #Set parameters switch to connect to COLLECTOR
    COLLECTOR_IP=10.0.0.1
    COLLECTOR_PORT=6343
    AGENT_IP=eth1
    HEADER_BYTES=128
    SAMPLING_N=64
    POLLING_SECS=1
    #Virtual Switch is monitored
    BRIDGE=br0
    
    #Configure Sflow in virtual Switch
    ovs-vsctl -- --id=@sflow create sflow agent=${AGENT_IP} target=\"${COLLECTOR_IP}:${COLLECTOR_PORT}\" header=${HEADER_BYTES} sampling=${SAMPLING_N} polling=${POLLING_SECS} -- set  bridge ${BRIDGE} sflow=@sflow

Với COLLECTOR_IP là IP của monitoring host, eth1 là card mạng nối với monitoring host.

    chmod +x sflow.sh
    ./sflow.sh
    
#### 2.4.2. Trên host Monitoring

Cài JAVA cho Windows theo [hướng dẫn] (http://www.java.com/en/download/help/windows_manual_download.xml) nếu chưa có.

Vào [link] (http://www.inmon.com/products/sFlowTrend.php) download file sFlowTrend.jnlp về và chạy trực tiếp file đó.
<img src=http://i.imgur.com/igp1f9P.png>

Tắt filewall nếu chưa tắt.

Trên hai máy ảo VM1 và VM2 ta có thể ping lẫn nhau hoặc ra ngoài, hoặc dùng lệnh wget download một file và quan sát sự thay đổi trên Monitoring Host.

Giao diện của sflow:
<img src=http://i.imgur.com/Nsz1GXH.png>

sFlow đang monitor trên một card mạng
<img src=http://i.imgur.com/CviElL6.png>
