1. Cài đặt công cụ cần thiết
sudo yum update 
sudo yum install -y createrepo genisoimage syslinux
2. Tạo cấu trúc thư mục làm việc
mkdir -p /root/centos-custom
cd /root/centos-custom
3. Sao chép toàn bộ hệ thống CentOS đã tối ưu
rsync -av / /root/centos-custom \
  --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/media/*","/lost+found"}
4. Thêm file Kickstart (Tùy chỉnh cài đặt tự động)
Tạo hoặc chỉnh sửa file:
vim /root/centos-custom/ks.cfg
Ví dụ nội dung đơn giản:

#version=CentOS6
install
cdrom
lang en_US.UTF-8
keyboard us
network --bootproto=dhcp
rootpw yourpassword
firewall --disabled
authconfig --enableshadow --passalgo=sha512
selinux --disabled
timezone Asia/Ho_Chi_Minh
bootloader --location=mbr
clearpart --all --initlabel
autopart
reboot
%packages
@core
%end
5. Tạo repository các gói cài đặt

mkdir -p /root/centos-custom/packages
cp /path/to/centos-rpms/*.rpm /root/centos-custom/packages
cd /root/centos-custom/packages
createrepo .
 6. Chuẩn bị cấu hình bootloader
Sao chép thư mục isolinux từ ISO cài đặt gốc CentOS 6:

mkdir -p /root/centos-custom/isolinux
cp -r /path/to/centos-iso/isolinux/* /root/centos-custom/isolinux/
7. Chỉnh sửa file isolinux.cfg
vim /root/centos-custom/isolinux/isolinux.cfg
Thêm hoặc sửa mục label:

cfg
Sao chép
Chỉnh sửa
label linux
  menu label ^Install Custom CentOS
  kernel vmlinuz
  append initrd=initrd.img ks=cdrom:/ks.cfg
Lưu ý: Đảm bảo vmlinuz và initrd.img tồn tại trong isolinux/ (hoặc copy từ ISO gốc nếu thiếu).

 8. Tạo file ISO khởi động được
cd /root/centos-custom
genisoimage -o /root/custom-centos.iso \
  -b isolinux/isolinux.bin \
  -c isolinux/boot.cat \
  -no-emul-boot \
  -boot-load-size 4 \
  -boot-info-table \
  -J -R -V "Custom CentOS" \

(root)
  .
 9. (Tùy chọn) Kiểm tra lại ISO
Dùng máy ảo (VirtualBox hoặc QEMU) để test file ISO:

qemu-kvm -cdrom /root/custom-centos.iso -m 2048
