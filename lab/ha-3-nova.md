# Cài đặt dịch vụ compute (Nova) trên Controller Node
## Tạo database cho Nova
**Thực hiện các bước sau trên 1 trong 3 node, các node còn lại sẽ tự đồng bộ.**  
Đăng nhập vào MySQL với password root đã tạo
```
mysql -u root -p
```
Tạo database nova_api, nova và nova_cell0 và gán quyền truy cập
```
CREATE DATABASE nova_api;
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY 'NOVA_DBPASS';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY 'NOVA_DBPASS';

CREATE DATABASE nova;
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY 'NOVA_DBPASS';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY 'NOVA_DBPASS';

CREATE DATABASE nova_cell0;
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' IDENTIFIED BY 'NOVA_DBPASS';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' IDENTIFIED BY 'NOVA_DBPASS';
```
## Tạo user, service và endpoint
**Thực hiện các bước sau trên 1 trong 3 node, các node còn lại sẽ tự đồng bộ.**  
Chạy script biến môi trường 
```
. admin-openrc
```
Tạo user `nova` với password *123456*
```
openstack user create --domain default --password-prompt nova
User Password:
Repeat User Password:
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 0a18f0472b0d467293eacd96ad0c24f8 |
| name                | nova                             |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```
Add user nova vào project service với role admin
```
openstack role add --project service --user nova admin
```
Tạo service nova
```
openstack service create --name nova --description "OpenStack Compute" compute
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Compute                |
| enabled     | True                             |
| id          | ad3f5f017b2c4c3c895497b3b2b2f10c |
| name        | nova                             |
| type        | compute                          |
+-------------+----------------------------------+
```
Tạo Endpoint cho Nova
```
openstack endpoint create --region RegionOne compute public http://controller:8774/v2.1
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 1f3b816c2ed14e848132a97bf6fcb717 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | ad3f5f017b2c4c3c895497b3b2b2f10c |
| service_name | nova                             |
| service_type | compute                          |
| url          | http://controller:8774/v2.1      |
+--------------+----------------------------------+

openstack endpoint create --region RegionOne compute internal http://controller:8774/v2.1
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 84122cdad7d849d99d65b9e373bc49b2 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | ad3f5f017b2c4c3c895497b3b2b2f10c |
| service_name | nova                             |
| service_type | compute                          |
| url          | http://controller:8774/v2.1      |
+--------------+----------------------------------+

openstack endpoint create --region RegionOne compute admin http://controller:8774/v2.1
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 288486920ddc4036ab16b4899409ed81 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | ad3f5f017b2c4c3c895497b3b2b2f10c |
| service_name | nova                             |
| service_type | compute                          |
| url          | http://controller:8774/v2.1      |
+--------------+----------------------------------+
```
Tạo user `placement` với password 123456
```
openstack user create --domain default --password-prompt placement
User Password:
Repeat User Password:
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 5d2763bedad64a8aae29ea1782c675c2 |
| name                | placement                        |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```
Add user `placement` vào project `service` với role `admin`  
```
openstack role add --project service --user placement admin
```
Tạo Placement API entry trong service catalog
```
openstack service create --name placement --description "Placement API" placement
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Placement API                    |
| enabled     | True                             |
| id          | fa5cd8d899594db696e6106bbdb0f702 |
| name        | placement                        |
| type        | placement                        |
+-------------+----------------------------------+
```
Tạo Endpoint cho dịch vụ Placement API
```
openstack endpoint create --region RegionOne placement public http://controller:8778
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | beee1ad0f1984a0d89620673dae992c2 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | fa5cd8d899594db696e6106bbdb0f702 |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://controller:8778           |
+--------------+----------------------------------+

openstack endpoint create --region RegionOne placement internal http://controller:8778
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 1787f4c825704e7bba74c8f2084908e5 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | fa5cd8d899594db696e6106bbdb0f702 |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://controller:8778           |
+--------------+----------------------------------+

openstack endpoint create --region RegionOne placement admin http://controller:8778
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | e77283879ddc4afb83aecd710e7d53e3 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | fa5cd8d899594db696e6106bbdb0f702 |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://controller:8778           |
+--------------+----------------------------------+
```
## Cài đặt và cấu hình nova
**Thực hiện các bước sau trên cả 3 node controller.**  
Cài đặt các gói Nova
```
apt install -y nova-api nova-conductor nova-consoleauth \
nova-novncproxy nova-scheduler nova-placement-api
```
Trong file `/etc/nova/nova.conf` cấu hình như sau:  
**Lưu ý: Các option có [IP Controller] thì phải bind vào IP của Controller đang cấu hình. Vì nếu không bind vào IP cụ thể thì các service sẽ bind 0.0.0.0, như vậy nova sẽ không thể mở port được do haproxy đang bind VIP:8774, VIP:8775, VIP:8778, VIP:6080.** Ví dụ: Cấu hình trên controller1 điền IP 172.16.69.90, controller2 172.16.69.83, controller3 172.16.69.57.
```
[DEFAULT]
lock_path = /var/lock/nova
state_path = /var/lib/nova
transport_url = rabbit://openstack:RABBIT_PASS@controller1,openstack:RABBIT_PASS@controller2,openstack:RABBIT_PASS@controller3,
my_ip = 0.0.0.0
osapi_compute_listen = [IP Controller]
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver
metadata_listen = [IP Controller]
[api]
auth_strategy = keystone
[api_database]
connection = mysql+pymysql://nova:NOVA_DBPASS@controller/nova_api
[cells]
enable = False
[database]
connection = mysql+pymysql://nova:NOVA_DBPASS@controller/nova
[glance]
api_servers = http://controller:9292
[keystone]
[keystone_authtoken]
auth_url = http://controller:5000/v3
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = 123456
[oslo_concurrency]
lock_path = /var/lib/nova/tmp
[placement]
os_region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://controller:5000/v3
username = placement
password = 123456
[vnc]
enabled = true
vncserver_listen = $my_ip
vncserver_proxyclient_address = $my_ip
novncproxy_host = [IP Controller]
```

**Sync db trên 1 node bất kỳ.**  
Sync db nova-api
```
su -s /bin/sh -c "nova-manage api_db sync" nova
```
Đăng ký cell0 database
```
su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
```
Tạo cell `cell1`
```
su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
```
Sync db nova-manage
```
su -s /bin/sh -c "nova-manage db sync" nova
```
Verify nova cell0 và nova cell1 được đăng ký thành công
```
nova-manage cell_v2 list_cells
+-------+--------------------------------------+------------------------------------+-------------------------------------------------+
|  Name |                 UUID                 |           Transport URL            |               Database Connection               |
+-------+--------------------------------------+------------------------------------+-------------------------------------------------+
| cell0 | 00000000-0000-0000-0000-000000000000 |               none:/               | mysql+pymysql://nova:****@controller/nova_cell0 |
| cell1 | b41b3e20-64b0-4f9d-af45-7c01064ec8ef | rabbit://openstack:****@controller |    mysql+pymysql://nova:****@controller/nova    |
+-------+--------------------------------------+------------------------------------+-------------------------------------------------+
```
**Khởi động lại các dịch vụ của Nova trên cả 3 node controller.**
```
service nova-api restart
service nova-consoleauth restart
service nova-scheduler restart
service nova-conductor restart
service nova-novncproxy restart
service nova-api status
service nova-consoleauth status
service nova-scheduler status
service nova-conductor status
service nova-novncproxy status
```

# Cài đặt dịch vụ Compute (Nova) trên các Compute Node
Cài đặt package
```
apt update -y
apt install -y nova-compute
```
Trong file `/etc/nova/nova.conf` cấu hình như sau:  
**Lưu ý: Option my_ip phải điền IP Management của Compute đang cấu hình.** Ví dụ: Cấu hình trên compute1 điền IP 172.16.69.80, compute2 172.16.69.45.
```
[DEFAULT]
lock_path = /var/lock/nova
state_path = /var/lib/nova
transport_url = rabbit://openstack:RABBIT_PASS@controller1,openstack:RABBIT_PASS@controller2,openstack:RABBIT_PASS@controller3,
my_ip = [IP Management Compute]
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver
[api]
auth_strategy = keystone
[api_database]
connection = sqlite:////var/lib/nova/nova_api.sqlite
[cells]
enable = False
[database]
connection = sqlite:////var/lib/nova/nova.sqlite
[glance]
api_servers = http://controller:9292
[keystone_authtoken]
auth_url = http://controller:5000/v3
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = 123456
[oslo_concurrency]
lock_path = /var/lib/nova/tmp
[placement]
os_region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://controller:5000/v3
username = placement
password = 123456
[vnc]
enabled = True
vncserver_listen = 0.0.0.0
vncserver_proxyclient_address = $my_ip
novncproxy_base_url = http://controller:6080/vnc_auto.html
```

## Hoàn tất cài đặt
Sử dụng lệnh sau để xem Compute Node có hỗ trợ ảo hóa không
```
egrep -c '(vmx|svm)' /proc/cpuinfo
```
Nếu giá trị trả về >= 1 thì Compute Node hỗ trợ tăng tốc phần cứng. Nếu giá trị trả về = 0 thì Compute Node không hỗ trợ tăng tốc phần cứng, bạn phải cấu hình libvirt sử dụng QEMU thay KVM trong file `/etc/nova/nova-compute.conf`
```
[libvirt]
virt_type = qemu
```
Restart Compute Service
```
service nova-compute restart
```

# Add các Compute Node vào Cell Database
**Thực hiện trên 1 Controller Node bất kỳ**  
Chạy script `. admin-openrc` và xác nhận có Compute Host trong database
```
. admin-openrc

openstack compute service list --service nova-compute
+----+--------------+----------+------+---------+-------+----------------------------+
| ID | Binary       | Host     | Zone | Status  | State | Updated At                 |
+----+--------------+----------+------+---------+-------+----------------------------+
| 57 | nova-compute | compute1 | nova | enabled | up    | 2018-09-20T11:54:32.000000 |
| 60 | nova-compute | compute2 | nova | enabled | up    | 2018-09-20T11:54:32.000000 |
| 63 | nova-compute | compute3 | nova | enabled | up    | 2018-09-20T11:54:33.000000 |
+----+--------------+----------+------+---------+-------+----------------------------+
```
Discover Compute Host
```
su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova
Found 2 cell mappings.
Skipping cell0 since it does not contain hosts.
Getting computes from cell 'cell1': 81c8bfa2-c720-4011-a84e-512cc7f69163
Checking host mapping for compute host 'compute1': ba9da252-a4f3-4fba-be60-42d6f7211d3a
Creating host mapping for compute host 'compute1': ba9da252-a4f3-4fba-be60-42d6f7211d3a
Checking host mapping for compute host 'compute2': be69e42e-8325-442d-b3c6-45ced3e265e7
Creating host mapping for compute host 'compute2': be69e42e-8325-442d-b3c6-45ced3e265e7
Checking host mapping for compute host 'compute3': a54cd6e7-0535-48ed-be8f-ae8d4241a656
Creating host mapping for compute host 'compute3': a54cd6e7-0535-48ed-be8f-ae8d4241a656
Found 3 unmapped computes in cell: 81c8bfa2-c720-4011-a84e-512cc7f69163
```
**Khi thêm compute node mới phải chạy lệnh `nova-manage cell_v2 discover_hosts` trên Controller để đăng ký Compute Node**

# Verify Operation
**Thực hiện các lệnh trên Controller Node**  
Chạy Script biến môi trường `. admin-openrc`  
List các dịch vụ thành phần  
```
openstack compute service list
+----+------------------+------------+----------+---------+-------+----------------------------+
| ID | Binary           | Host       | Zone     | Status  | State | Updated At                 |
+----+------------------+------------+----------+---------+-------+----------------------------+
|  1 | nova-scheduler   | controller | internal | enabled | up    | 2018-01-03T03:46:43.000000 |
|  2 | nova-consoleauth | controller | internal | enabled | up    | 2018-01-03T03:46:43.000000 |
|  3 | nova-conductor   | controller | internal | enabled | up    | 2018-01-03T03:46:43.000000 |
|  6 | nova-compute     | compute    | nova     | enabled | up    | 2018-01-03T03:46:44.000000 |
+----+------------------+------------+----------+---------+-------+----------------------------+
```
List các API Endpoint trong KeyStone để xác nhận các kết nối với KeyStone
```
openstack catalog list
+-----------+-----------+-----------------------------------------+
| Name      | Type      | Endpoints                               |
+-----------+-----------+-----------------------------------------+
| keystone  | identity  | RegionOne                               |
|           |           |   admin: http://controller:35357/v3/    |
|           |           | RegionOne                               |
|           |           |   public: http://controller:5000/v3/    |
|           |           | RegionOne                               |
|           |           |   internal: http://controller:5000/v3/  |
|           |           |                                         |
| glance    | image     | RegionOne                               |
|           |           |   admin: http://controller:9292         |
|           |           | RegionOne                               |
|           |           |   internal: http://controller:9292      |
|           |           | RegionOne                               |
|           |           |   public: http://controller:9292        |
|           |           |                                         |
| nova      | compute   | RegionOne                               |
|           |           |   public: http://controller:8774/v2.1   |
|           |           | RegionOne                               |
|           |           |   admin: http://controller:8774/v2.1    |
|           |           | RegionOne                               |
|           |           |   internal: http://controller:8774/v2.1 |
|           |           |                                         |
| placement | placement | RegionOne                               |
|           |           |   internal: http://controller:8778      |
|           |           | RegionOne                               |
|           |           |   public: http://controller:8778        |
|           |           | RegionOne                               |
|           |           |   admin: http://controller:8778         |
|           |           |                                         |
+-----------+-----------+-----------------------------------------+
```
List các Image trong Glance
```
openstack image list
+--------------------------------------+--------+--------+
| ID                                   | Name   | Status |
+--------------------------------------+--------+--------+
| 6f7ad56c-3d46-4fe2-9e1b-b49ec75d0791 | cirros | active |
+--------------------------------------+--------+--------+
```
Kiểm tra cell và placement API chạy thành công
```
nova-status upgrade check
+---------------------------+
| Upgrade Check Results     |
+---------------------------+
| Check: Cells v2           |
| Result: Success           |
| Details: None             |
+---------------------------+
| Check: Placement API      |
| Result: Success           |
| Details: None             |
+---------------------------+
| Check: Resource Providers |
| Result: Success           |
| Details: None             |
+---------------------------+
```

