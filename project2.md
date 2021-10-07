# Project 2: : Dựng NFS server

Yêu cầu:

* Cài đặt NFS server và cấu hình export thư mục /DATA trên server sao cho chỉ 2 IP Client kết nối tới.

* Sau đó để Client-1 ghi dữ liệu và kiểm tra dữ liệu đã đồng bộ trên Client-2 chưa, và ngược lại

### Mô hình LAB ###

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/xg5bstojgj_Untitled-1.jpg)

>
Mô hình gồm có 1 NFS Server và 2 Client

Cả Server và Client đều sử dụng hệ điều hành CentOS 7

NFS Server: 192.168.1.184

Client 1: 192.168.1.233

Client 2: 192.168.1.177

### Cài đặt và cấu hình NFS Server ###

Cài đặt cài package *nfs-utils*

` # yum install nfs-utils -y `

Tạo thư mục chia sẻ tài nguyên trên server

` # mkdir /DATA `

Sửa file /etc/exports để tạo mount point export

` # vi /etc/exports `

Thêm vào nội dung sau, lưu lại và thoát
```
/DATA 192.168.1.233(rw,sync,no_root_squash,no_subtree_check)
/DATA 192.168.1.177(rw,sync,no_root_squash,no_subtree_check) 
```
Trong đó

` /DATA ` là đường dẫn chia sẻ trên Server

` 192.168.1.233 ` và ` 192.168.1.177 ` là địa chỉ IP cho phép kết nối truy xuất tài nguyên trên Server

Nếu IP thay bằng * thì mọi client có quyền truy cập

Khởi động NFS Server
```
# systemctl enable rpcbind nfs-server
# systemctl start rpcbind nfs-server
```
Cấu hình firewall cho phép dịch vụ nfs
```
# firewall-cmd --permanent --add-service=nfs
# firewall-cmd --permanent --add-service=mountd
# firewall-cmd --permanent --add-service=rpc-bind
# firewall-cmd --reload
```
### Cài đặt và cấu hình NFS Client ###

**Trên Client 1:**

Cài đặt cài package *nfs-utils*

` # yum install nfs-utils -y `

Tạo thư mục để mount tài nguyên được chia sẻ từ NFS Server

` # mkdir /DATA1 `

Sử dụng lệnh sau để mount

` # mount -t nfs 192.168.1.184:/DATA /DATA1 `

Trong đó:

10.10.10.11:/DATA là ip và đường dẫn đến thư mục tài nguyên chia sẻ trên NFS Server

/DATA1 là đường dẫn của thư mục trên client hay còn gọi là mount point

Kiểm tra xem mount point đã hoạt động hay chưa ta sử dụng lệnh ` # df -h `

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/9f6xv128z1_Screenshot%202021-10-07%20142921.png)

**Trên Client 2**

Thực hiện tương tự như Client 1

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/timzfhcki3_Screenshot%202021-10-07%20143010.png)

###Kiểm tra kết quả###

Trên Client 1, ta truy cập /DATA1 và tạo file mới là huanpham.txt

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/ys03am2j3d_Screenshot%202021-10-07%20144445.png)

Trên Client 2, ta truy cập /DATA2 và thấy file huanpham.txt đã được đồng bộ

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/orau8n9le4_Screenshot%202021-10-07%20144526.png)

Và trên NFS Server ta truy cập /DATA cũng thấy file huanpham.txt

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/atcs4ikh0d_Screenshot%202021-10-07%20144639.png)

Như vậy Client đã nhận dung lượng và mount thành công tài nguyên từ Server lên
