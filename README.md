###Mô hình hệ thống: 

![Alt text](http://i.imgur.com/KC6anmb.png)

------------------

####Quá trình chuẩn bị môi trường:

- Trên `Salt-master`:

 ```sh
 apt-get update && apt-get upgrade -y
 apt-get install python-software-properties -y
 apt-get install git -y
 cd ~
 git clone https://github.com/thanh142/salt-openstack.git
 cp -R salt-openstack/salt /srv
 cp -R salt-openstack/pillar /srv
 add-apt-repository ppa:saltstack/salt -y
 apt-get install salt-master –y
 ```

  Sửa file cấu hình cho `salt-master` tại `/etc/salt/master` :

 ```sh
 # saltmaster sẽ listen trên tất cả các IP 
 interface: 0.0.0.0
 # Saltmaster sẽ đọc các file cấu hình ở /srv/salt
 file_roots:
   base:
     - /srv/salt
 # Saltmaster sẽ đọc các file pillar ở /srv/pillar
 pillar_roots:
   base:
     - /srv/pillar
 ```

- Trên `salt-minion`:

 Tiến hành cập nhật hệ thống và cài gói salt-minion:

 ```sh
 apt-get update && apt-get upgrade -y
 apt-get install salt-minion -y
 ```

 Chỉnh sửa cấu hình cho `salt-minion` tại `/etc/salt/minion` :

 ```sh
 # trỏ tới ip hoặc hostname của salt master:
 master: 192.168.146.70
 ```

Restart minion: `/etc/init.d/salt-minion restart`

-------------------------------

Sau khi tiến hành cài đặt các phần mềm để tạo môi trường cho salt-master và các salt-minion, các minion sẽ gửi các yêu cầu kết nối đến salt master. Các yêu cầu kết nối được thể hiện bằng các key, cũng nhằm để xác thực giữa minion và master trong suốt phiên làm việc. Trên salt master, ta liệt kê các minion đã gửi yêu cầu kết nối: 

 `salt-key -L`

![](http://img.prntscr.com/img?url=http://i.imgur.com/E3aVofN.png)

Các minion chưa được add key sẽ nằm tại: Unaccepted Keys: Để add key làm như sau:

    • Add 1 minion: salt-key -a $ten_minion Ví dụ: salt-key -a controller

    • Add tất cả minion: salt-key –A

![](http://img.prntscr.com/img?url=http://i.imgur.com/Sgfy8Xz.png)

Sau khi add xong salt-key -L sẽ thấy các minion được add nằm tại Accepted Keys:

![](http://img.prntscr.com/img?url=http://i.imgur.com/VgHhizT.png)


Lúc này các minion đã được salt master chấp nhận kết nối và nằm dưới sự kiểm soát cấu hình của salt master. Kiểm tra kết nối giữa salt master và các minion:

`salt '*' test.ping`

![](http://img.prntscr.com/img?url=http://i.imgur.com/ssjwTgq.png)


Kết quả trả về “True” trên tất cả các node, quá trình cài đặt và thiết lập môi trường đã hoàn tất và sẵn sàng cho việc sử dụng và cấu hình.

------------------------

####Tiến hành cài đặt:
Tải các script cài đặt từ salt-master tới các salt-minion để bắt đầu việc cài đặt môi trường Openstack bằng câu lệnh trên salt-master:

`salt '*' state.highstate`

Sau khi cài đặt thành công, mở trình duyệt tại địa chỉ http://192.168.146.71 và đăng nhập bằng tài khoản admin password: f0290c8447ff2c7e5962 để sử dụng OpenStack.

-----------------

####Cấu hình image mẫu Cirros cho hệ thống:

```sh
mkdir images
cd images/
wget https://launchpad.net/cirros/trunk/0.3.0/+download/cirros-0.3.0-x86_64-disk.img
glance image-create --name "cirros-0.3.3-x86_64" --disk-format qcow2 --container-format bare --is-public True --progress < cirros-0.3.0-x86_64-disk.img
cd  
rm -r images
```
