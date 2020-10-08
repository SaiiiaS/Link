GIT – OpenVPN là một phần mềm mạng riêng ảo mã nguồn mở dành cho việc tạo các đường ống (tunnel) điểm-tới-điểm được mã hóa giữa các máy chủ. Phần mềm này do James Yonan viết và được phổ biến dưới giấy phép GNU GPL.


Về cơ bản, VPN là một mạng riêng sử dụng hệ thống mạng công cộng (thường là Internet) để kết nối các địa điểm hoặc người sử dụng từ xa với một mạng LAN ở trụ sở trung tâm. Thay vì dùng kết nối thật khá phức tạp như đường dây thuê bao số, VPN tạo ra các liên kết ảo được truyền qua Internet giữa mạng riêng của một tổ chức với địa điểm hoặc người sử dụng ở xa.

Mạng riêng ảo (Virtual Private Network – VPN) là một kết nối rất an toàn, đáng tin cậy giữa mạng cục bộ (LAN) và một hệ thống khác. Bạn có thể hình dung router của mình là chiếc cầu nối để các mạng kết nối vào. Máy tính của bạn và máy chủ OpenVPN (trong trường hợp này chính là router) sẽ “bắt tay” với nhau bằng cách sử dụng chứng chỉ xác nhận lẫn nhau. Sau khi xác nhận, cả máy khách và máy chủ sẽ đồng ý “tin tưởng” nhau và cho phép truy cập vào mạng của server.

OpenVPN sử dụng thiết bị tun/tap (hầu như có sẵn trên các bản Linux) và Openssl để xác nhận (authenticate), mã hóa (khi gởi) và giải mã (khi nhận) đường truyền giữa hai bên thành chung một network. Có nghĩa là khi người dùng nối vào  máy chủ OpenVPN từ xa, họ có thể sử dụng các dịch vụ như chia sẻ tập tin sử dụng Samba/NFS/FTP/SCP … đọc thư (bằng  cách khai báo địa chỉ nội bộ trên máy họ, ví dụ, 192.168.1.1), duyệt intranet, sử dụng các phần mềm khác..v..v..như là họ đang ngồi trong văn phòng.

Thông thường, triển khai phần mềm VPN và phần cứng tốn nhiều thời gian và chi phí, do đó OpenVPN là một giải pháp mã nguồn mở VPN hoàn toàn miễn phí. Sau đây là các bước cần thực hiện:

#1. Cài đặt OpenVPN

a. Cài đặt các gói liên quan :

`yum install openssl lzo pam openssl-devel lzo-devel pam-devel`

`wget http://swupdate.openvpn.org/community/releases/openvpn-2.2.2.tar.gz`

– Lưu ý : gói lzo có thể thay bằng lzo2 đối với môt số bản linux ( CentOS 6, RedHat 6 …)

b. Thư viện cài đặt :

`yum install make gcc-c++`

Chú ý :

+ Chay dòng trên nếu gcc chưa được setup , hay có thể dùng A C compiler tương tự như GCC
+ Nếu bạn setup từ source thì cần dùng thư việc gcc-c++

c. Cài OpenVPN:

`tar -xvfz  openvpn-2.2.2.tar.gz`

`cd openvpn-2.2.2`

`./configure`

`make`

`make install`

#2.Cấu hình OPENVPN

A. Tạo Certificate Authority ( CA ) certificate & key

– Bạn vào easy-rsa có trong /usr/share/doc/openvpn-2.2.2 ( tùy vào phiên bản mà bạn download về setup , ở đây là bản openvpn-2.2.2) hay /usr/share/doc/packages/openvpn và chỉnh sửa lại file vars những thông số sau cho phù hợp với bạn

```sh
KEY_COUNTRY=VN
KEY_PROVINCE=Q3
KEY_CITY=HCM
KEY_ORG=”OpenVPN-GocIT”
KEY_EMAIL=”admin@gocit.vn”
```

– Thực hiện tiếp dòng lệnh sau

```sh
. ./vars
./clean-all
./build-ca
./build-ca
Generating a 1024 bit RSA private key
............++++++
...........++++++
writing new private key to 'ca.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [VN]:
State or Province Name (full name) [Q3]:
Locality Name (eg, city) [HCM]:
Organization Name (eg, company) [OpenVPN-GocIT]:
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server's hostname) []:OpenVPN-CA
Email Address [admin@gocit.vn]:
```

-Tạo certificate & key cho server :
`./build-key-server server`
-Tạo certificate & key cho client
 `./build-key hautp`
 
Note : nếu muốn đặt passwd cho client thì có thể dùng build-key-pass để đặt passwd cho client , phần này chúng ta không cần quan tâm vì chúng ta sẽ dùng webmin để tạo accout cho user

`./build-key-pass hautp 123456`
– Tạo Diffie Hellman
```sh
./build-dh
./build-dh
Generating DH parameters, 1024 bit long safe prime, generator 2
This is going to take a long time
.................+...........................................
...................+.............+.................+.........
......................................
```
– Key file

– Tiến hành cấu hình cho openvpn server .

`mkdir config`
`cp /etc/openvpn/sample-config-files/server.conf /etc/openvpn/config`
`cd /etc/openvpn/easy-rsa`
`p dh1024.pem server.key server.crt ca.crt /etc/openvpn/config`

Cấu hình chức năng Forwarding (dùng để thực hiện Lan Routing)

`vi /etc/sysctl.conf`
`net.ipv4.ip_forward = 1`
`sysctl –p (để cho các thông số có hiệu lực)`
`echo 1 > /proc/sys/net/ipv4/ip_forward`

Cấu hình VPN Server

– Copy file cấu hình server.conf mẫu từ source cài đặt vào /etc/openvpn/
`cp /root/openvpn-2.2.2/sample-config-files/server.conf /etc/openvpn/`
– Chỉnh sửa file cấu hình:

`cd /etc/openvpn/`
`vi server.conf local 192.168.1.200 (chọn card mạng user quay VPN đến, có thể không cần option này)`
```sh
port 199 (default là 1194)
proto udp (protocol udp)
dev tun (dùng tunnel, nếu dùng theo bridge chọn dev tap0 và những config khác sẽ khác với tunnel)
ca /etc/openvpn/easy-rsa/keys/ca.crt (khai báo đuờng dẫn cho file ca.crt)
cert /etc/openvpn/easy-rsa/keys/openvpnserver.crt
key /etc/openvpn/easy-rsa/keys/openvpnserver.key
dh /etc/openvpn/easy-rsa/keys/dh1024.pem
server 10.8.0.0 255.255.255.0 (khai báo dãy IP cần cấp cho VPN Client, mặc định VPN Server sẽ lấy IP đầu tiên – 10.8.0.1)
;ifconfig-pool-persist ipp.txt (dùng để cho VPN Client lấy lại IP trước đó nếu bị đứt kết nối với VPN server, do chúng ta dùng IP tĩnh nên không sử dụng thông số này)
push “route 172.16.0.0 255.255.255.0” (lệnh này sẽ đẩy route mạng 172.16.0.0 đến Client, hay còn gọi là Lan Routing trong Windows Server, giúp cho VPN Client thấy được mạng bên trong của công ty)
;push “route 192.168.1.200 255.255.255.0” do bài Lab của chúng ta VPN Client đã connect đến được network 192.168.1.0 nên không cần add route dòng này (nếu có sẽ không chạy được)
,chỉ cần add route các lớp mạng bên trong công ty mà Client bên ngoài không connect được)
client-config-dir ccd (dùng để khai báo cấp IP tĩnh cho VPN Client)
client-to-client (cho phép các VPN client nhìn thấy nhau, mặc định client chỉ thấy server)Cũng khá đơn giản nhỉ, ngoài ra còn cónhững thông số khác không dùng đến như:
;push “redirect-gateway” (mọi traffic của VPN Client – http, dns, ftp, … đều thông qua đuờng Tunnel. Khác với lệnh push route, chỉ những traffic đi vào mạng nội bộ mới thông qua Tunnel, khi dùng lệnh này yêu cầu bên trong mạng nội bộ cần có NAT Server, DNS Server)
push “dhcp-option DNS (WINS) 10.8.0.1” đẩy DNS or WINS config vào VPN Client
```

Cấu hình file IP tĩnh tương ứng với từng User:
Sau khi đã cấu hình server, tiếp đó ta sẽ cấu hình các file đặt trong thư mục cdd/ tương ứng với từng User VPN.+ Tạo thư mục ccd (/etc/openvpn/ccd)
`mkdir /etc/openvpn/ccd`
+ Tạo profile cho user hautp
`vi /etc/openvpn/ccd/hautp`
`ifconfig-push 10.8.0.2 10.8.0.1 theo file cấu hình trên user hautp sẽ nhận IP là 10.8.0.2. Cặp IP khai báo trong lệnh trên phải thuộc bảng bên dưới, ứng với mỗi user sẽ có 1 cặp ip tương ứng.`
Start VPN Server
`p /root/openvpn-2.2.2/sample-scripts/openvpn.init /etc/init.d/openvpn`
`/etc/init.d/openvpn start`
Các bạn kiểm tra lại log để giải quyết lỗi nhé.
Phần 2 : Thiết lập Open VPN Client .

Bước 1 : Download bản open VPN dành cho Windows tại đây http://swupdate.openvpn.org/community/releases/openvpn-2.2.2-install.exe  .

Bước 2: Tiến hành các thủ tục cài đặt mặc định . Rồi copy các files ca.crt , client.crt , client.key. Trên server linux Vào thư mục C:\Program Files\OpenVPN\config trên máy Windows XP

Bước 3 : Dùng notepad tiến hành edit files C:\Program Files\OpenVPN\sample-config\client.opvn
```sh
client
dev tun (tunnel)
proto udp (upd protocol)
remote 192.168.1.200 199 (khai báo IP:Port server OpenVPN)
nobind
persist-key
persist-tun
ca ca.crt (khai báo CA server)
cert hautp.crt (certificate user hautp)
key hautp.key (private key hautp)
comp-lzo
verb 3
```
Save lại . Rồi copy files client.opvn vào thư mục C:\Program Files\OpenVPN\config trên máy Windows .

Bước 4: Khởi động OpenvpnGui . Sẽ thấy biểu tượng ở góc phải taskbar phải màn hình . Nhấp chuột phải biểu tượng và Click vào mục connect .

Nguồn tham khảo :

Wikipedia : http://en.wikipedia.org/wiki/Openvpn

OpenVPN Community : http://openvpn.net/index.php/open-source/documentation/howto.html

Cùng thảo luận bài viết Hướng dẫn cài đặt OpenVPN tại forum : http://forum.gocit.vn/threads/huong-dan-cai-dat-openvpn.111/

Bài viết này nằm trong dự án Linux Toàn Tập website : www.gocit.vn, forum hổ trợ forum.gocit.vn , xem thêm tại link http://www.gocit.vn/du-an-linux-toan-tap/