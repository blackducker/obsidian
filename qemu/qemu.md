

 搭环境 qemu 起一个虚拟机，显卡直通，环境 vmid 
mdev vfio模块 pcie bar vfio -> host kernel

##  **确认硬件支持**‌

- ‌**CPU和主板**‌：需要支持‌**VT-d**‌（Intel）或‌**AMD-Vi**‌（IOMMU）。
- ‌**GPU**‌：建议使用独立显卡（NVIDIA/AMD），部分集成显卡可能不支持。
- ‌**BIOS设置**‌：确保启用VT-d/AMD-Vi、Above 4G Decoding、SR-IOV等选项。

## ‌**启用IOMMU**
/etc/default/grub 启动参数GRUB_CMDLINE_LINUX添加
amd_iommu=on iommu=pt

验证则是，dmesg | grep -e "DMAR" -e "IOMMU"

3.查看PCI设备信息 lspci -nnv
记录GPU的‌**PCI ID**‌（格式如`10de:13c2`）和所在IOMMU组
```
0b:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Navi 31 [Radeon RX 7900 XT/7900 XTX/7900 GRE/7900M] [1002:744c] (rev ce) (prog-if 00 [VGA controller])
        Subsystem: Advanced Micro Devices, Inc. [AMD/ATI] RX 7900 XTX / RX 7900 GRE [XFX] [1002:0e3b]
        Flags: bus master, fast devsel, latency 0, IRQ 95, IOMMU group 25
        Memory at d0000000 (64-bit, prefetchable) [size=256M]
        Memory at e0000000 (64-bit, prefetchable) [size=2M]
        I/O ports at e000 [size=256]
        Memory at fca00000 (32-bit, non-prefetchable) [size=1M]
        Expansion ROM at 000c0000 [disabled] [size=128K]
        Capabilities: <access denied>
        Kernel driver in use: amdgpu
        Kernel modules: amdgpu
```

```
0b:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Navi 31 [Radeon RX 7900 XT/7900 XTX/7900 GRE/7900M] [1002:744c] (rev ce)
        Subsystem: Advanced Micro Devices, Inc. [AMD/ATI] RX 7900 XTX / RX 7900 GRE [XFX] [1002:0e3b]
        Kernel driver in use: vfio-pci
        Kernel modules: amdgpu
0b:00.1 Audio device [0403]: Advanced Micro Devices, Inc. [AMD/ATI] Navi 31 HDMI/DP Audio [1002:ab30]
        Subsystem: Advanced Micro Devices, Inc. [AMD/ATI] Navi 31 HDMI/DP Audio [1002:ab30]
        Kernel driver in use: vfio-pci
        Kernel modules: snd_hda_intel
0b:00.2 USB controller [0c03]: Advanced Micro Devices, Inc. [AMD/ATI] Navi 31 USB [1002:7446]
        Subsystem: Advanced Micro Devices, Inc. [AMD/ATI] Navi 31 USB [1002:7446]
        Kernel driver in use: vfio-pci
        Kernel modules: xhci_pci
0b:00.3 Serial bus controller [0c80]: Advanced Micro Devices, Inc. [AMD/ATI] Device [1002:7444]
        Subsystem: Advanced Micro Devices, Inc. [AMD/ATI] Device [1002:0408]
        Kernel driver in use: vfio-pci
        Kernel modules: i2c_designware_pci
```

4.配置VFIO PCI驱动
加载VFIO模块
```
echo "vfio" | sudo tee /etc/modules-load.d/vfio.conf 
echo "vfio_pci" | sudo tee -a /etc/modules-load.d/vfio.conf
```
修改/etc/modprobe.d/vfio.conf来绑定iommu组
```
options vfio-pci ids=10de:13c2
```
验证vfio启动成功
```
sudo update-initramfs -u
sudo reboot
lspci -k | grep -i -e 7900 -A 3
```

输出显示 Kernel driver in use: vfio-pci

启动虚拟机
~~apparmor.d/libvirt/libvirt-92f6bbed-23fe-4513-92b7-01e233e527f4~~
~~➜  ~ sudo virt-install --name vm-test --os-variant ubuntu22.04 --ram 8192 --vcpus 8 --disk path=./win11.qcow2,size=100 --graphics spice --cdrom ./ubuntu-22.04.5-desktop-amd64.iso~~

推荐直接使用virt-manage 图形工具创建虚拟机，这样方便配置参数。
1. 使用qxl虚拟显卡，成功安装ubuntu后，开启ssh服务。
2. 手动add pci device，可以自动使用vfio-pci来占用pci device。显示器会黑屏
3. ssh链接进虚拟机后，lspci可以发现显卡设备已经被直通过来了。

目前的问题是，无法在虚拟机中，运行显卡驱动，FW加载失败，不确定amdgpu_device_ip_early_init是否正确识别到vfio偏移的相关内存地址。
note: ubuntu不自带FW，需要手动安装。

## Qemu 编译
ubuntu通常virsh自带qemu版本
检查  virsh version
```
Compiled against library: libvirt 8.0.0
Using library: libvirt 8.0.0
Using API: QEMU 8.0.0
Running hypervisor: QEMU 6.2.0
```
升级QEMU ~~6.2.0~~ -> 7.2.0
1. 下载qemu源码, 默认编译命令
```
mkdir build
cd build
../configure
make
```
2. 进阶编译命令，增加spice协议支持，并仅编译x86的版本 qemu-system-x86_64
```
mkdir build
cd build
../configure --target-list=x86_64-softmmu --enable-spice
make -j8
```



-sandbox support is not enabled in this QEMU binary

```
../configure --target-list=x86_64-softmmu --enable-spice --enable-seccomp
```

virsh的xml直接用来qemu参数启动的话，需要将-object 修改为 顺序严格参数格式

### 替换virsh后端qemu版本
第一种，指定安装路径到/usr并make install替换。

第二种，修改apparmor加入新路径的权限。以支持virsh调用新版本的qemu工具。
AppArmor 是一款与 SeLinux 类似的安全框架 / 工具，其主要作用是控制应用程序的各种权限，例如对某个目录 / 文件的读 / 写，对网络端口的打开 / 读 / 写等

virsh 编辑配置文件的 `emulator` 部分，修改后使其生效时，会出现权限错误。
解决方法：
在 `/etc/apparmor.d/usr.sbin.libvirtd` 文件中，添加一行:
使能生效：`sudo systemctl reload apparmor` 或 `sudo systemctl restart apparmor.service`

## qemu参数解析


1.首先通过qemu-img创建磁盘文件
2.加载系统安装iso并启动到磁盘
```
qemu-system-x86_64 -enable-kvm -m 8G -smp 4 -boot once=d -cdrom ../ubuntu-22.04.5-desktop-amd64.iso test-qemu.qcow2
```

3.qemu配置增加sdl以支持基础显示，后可通过-nographic关闭显示或切换到vnc
`sudo apt install sdl2`

4.基础参数优化



参照virsh优化的参数
```
qemu-system-x86_64 \
-machine pc-q35-6.2,usb=off,vmport=off,dump-guest-core=off,memory-backend=pc.ram \
-object qom-type=memory-backend-ram,id=pc.ram,size=4294967296 \
-enable-kvm -m 4G -smp 4 \
-device pcie-rootort,port=16,chassis=1,id=pci.1,bus=pcie.0,multifunction=on,addr=0x3 \
-device pcie-root-port,port=19,chassis=4,id=pci.4,bus=pcie.0,addr=0x3.0x1 \
-blockdev driver=file,filename=./test-qemu.qcow2,node-name=libvirt-2-storage,auto-read-only=true,discard=unmap \
-blockdev node-name=libvirt-2-format,read-only=false,discard=unmap,driver=raw,file=libvirt-2-storage \
-device virtio-blk-pci,bus=pci.4,addr=0x0,drive=libvirt-2-format,id=virtio-disk0,bootindex=1
```
### 基础参数优化

machine启动默认配置，并设置磁盘启动
-machine 参数用来 指定使用pc-q35-6.2芯片组，启动相关默认配置
```
qemu-system-x86_64 -machine pc-q35-6.2,usb=off,vmport=off,dump-guest-core=off,memory-backend=pc.ram \
-object qom-type=memory-backend-ram,id=pc.ram,size=4294967296 -accel kvm -cpu host,migratable=on -m 4096 \
-overcommit mem-lock=off -smp 4,sockets=4,cores=1,threads=1 -no-user-config -nodefaults
```





### 网络参数
1.  用户模式网络
首先是简单版的用户模式网络后端
`sudo apt install libslirp-dev`
--enable-slirp， SLiRP 在用户模式下完全模拟 TCP/IP 协议栈，相比桥接或 TAP 模式，SLiRP 无需宿主机安装 `bridge-utils`、`dnsmasq` 等工具，也无需手动配置网桥或 IP 转发规则。用户模式下运行，不要求管理员（root）权限，适合快速启动轻量级虚拟机

qemu 增加下面参数使用，用户模式网络
`-netdev user,id=net0 -device virtio-net-pci,netdev=net0`

2. 桥接网络virbr0
KVM启动后，默认创建virbr0网络设备
`-netdev bridge,id=net0,br=virbr0 -device virtio-net-pci,netdev=net0`
容易遇到，**访问/dev/net/tun的权限问题**
**设置 `qemu-bridge-helper` 权限**
`sudo chmod u+s /usr/libexec/qemu-bridge-helper`

### 显示参数

1.增加qxl虚拟显卡支持
`-device qxl-vga,id=video0,ram_size=67108864,vram_size=67108864,vram64_size_mb=0,vgamem_mb=16,max_outputs=1,bus=pcie.0,addr=0x0`


```
qemu-system-x86_64 -machine pc-q35-6.2,usb=off,vmport=off,dump-guest-core=off,memory-backend=pc.ram \
-object qom-type=memory-backend-ram,id=pc.ram,size=4294967296 -accel kvm -cpu host,migratable=on -m 4G -smp 4,sockets=4,cores=1,threads=1 \
-device pcie-root-port,port=16,chassis=1,id=pci.1,bus=pcie.0,multifunction=on,addr=0x3 \
-device pcie-root-port,port=17,chassis=2,id=pci.2,bus=pcie.0,addr=0x3.0x1 \
-device pcie-root-port,port=18,chassis=3,id=pci.3,bus=pcie.0,addr=0x3.0x2 \
-device pcie-root-port,port=19,chassis=4,id=pci.4,bus=pcie.0,addr=0x3.0x3 \
-blockdev driver=file,filename=./test-qemu.qcow2,node-name=libvirt-2-storage,auto-read-only=true,discard=unmap \
-blockdev node-name=libvirt-2-format,read-only=false,discard=unmap,driver=raw,file=libvirt-2-storage \
-device virtio-blk-pci,bus=pci.2,addr=0x0,drive=libvirt-2-format,id=virtio-disk0,bootindex=1 \
-device qxl-vga,id=video0,ram_size=67108864,vram_size=67108864,vram64_size_mb=0,vgamem_mb=16,max_outputs=1,bus=pcie.0,addr=0x2
```


2.vfio透传pcie 显卡
`-device vfio-pci,host=0000:0b:00.0,id=hostdev0,bus=pci.3,addr=0x0`
需要解绑设备驱动，并重新绑定到vfio
`lspci -nnk`
`echo 0000:0b:00.0 > /sys/bus/pci/drivers/amdgpu/unbind`
`echo -n "1002 744c"  >  /sys/bus/pci/drivers/vfio-pci/new_id`
`lspci -s 0b:00.0 -k`

修改iommu组权限
`sudo chmod 660 /dev/vfio/25`

**错误**
qemu-system-x86_64: -device vfio-pci,host=0000:0b:00.0,id=hostdev0,bus=pci.4,addr=0x0: VFIO_MAP_DMA failed: Cannot allocate memory
qemu-system-x86_64: -device vfio-pci,host=0000:0b:00.0,id=hostdev0,bus=pci.4,addr=0x0: vfio 0000:0b:00.0: failed to setup container for group 25: memory listener initialization failed: Region pc.ram: vfio_dma_map(0x5b4411be9ab0, 0x100000000, 0x80000000, 0x79022fe00000) = -12 (Cannot allocate memory)

解决方案，
内核信息显示，[56881.551620] vfio_pin_pages_remote: RLIMIT_MEMLOCK (4193574912) exceeded
需要ulimit -l , 增加用户进程可以锁定的内存。可能被配置文件强制锁定，那样的话，还需要修改/etc/security/limits.conf，解除锁定。
` bob             hard    memlock         unlimited `
 `

```
qemu-system-x86_64 -machine pc-q35-6.2,usb=off,vmport=off,dump-guest-core=off,memory-backend=pc.ram \
-object qom-type=memory-backend-ram,id=pc.ram,size=4294967296 -accel kvm -cpu host,migratable=on -m 4096 \
-overcommit mem-lock=off -smp 4,sockets=4,cores=1,threads=1 -no-user-config -nodefaults \
-device pcie-root-port,port=16,chassis=1,id=pci.1,bus=pcie.0,multifunction=on,addr=0x3 \
-device pcie-root-port,port=17,chassis=2,id=pci.2,bus=pcie.0,addr=0x3.0x1 \
-device pcie-root-port,port=18,chassis=3,id=pci.3,bus=pcie.0,addr=0x3.0x2 \
-device pcie-root-port,port=19,chassis=4,id=pci.4,bus=pcie.0,addr=0x3.0x3 \
-blockdev driver=file,filename=/home/bob/qemu-sh/test-qemu.qcow2,node-name=libvirt-2-storage,auto-read-only=true,discard=unmap \
-blockdev node-name=libvirt-2-format,read-only=false,discard=unmap,driver=raw,file=libvirt-2-storage \
-netdev bridge,id=net0,br=virbr0 -device virtio-net-pci,netdev=net0,mac=52:54:00:af:83:d5,bus=pci.1,addr=0x0 \
-device virtio-blk-pci,bus=pci.2,addr=0x0,drive=libvirt-2-format,id=virtio-disk0,bootindex=1 \
-device qxl-vga,id=video0,ram_size=67108864,vram_size=67108864,vram64_size_mb=0,vgamem_mb=16,max_outputs=1,bus=pcie.0,addr=0x2 \
-device virtio-balloon-pci,id=balloon0,bus=pci.3,addr=0x0 \
-device vfio-pci,host=0000:0b:00.0,id=hostdev0,bus=pci.4,addr=0x0
```




默认dmesg不需要权限
```
echo "kernel.dmesg_restrict = 0" | sudo tee /etc/sysctl.d/dmesg.conf  
sudo sysctl -p /etc/sysctl.d/dmesg.conf  # 加载配置  
```


更新blacklist.conf
sudo update-initramfs -u  # Debian/Ubuntu  
sudo dracut -fv           # RHEL/CentOS/Fedora  
\

lspci -d 1002: -vvvnnnn | grep -i region
```
		Region 0: Memory at d0000000 (64-bit, prefetchable) [size=256M]
        Region 2: Memory at e0000000 (64-bit, prefetchable) [size=2M]
        Region 4: I/O ports at e000 [size=256]
        Region 5: Memory at fca00000 (32-bit, non-prefetchable) [size=1M]
```

../configure --target-list=x86_64-softmmu --enable-spice --enable-seccomp --enable-kvm --prefix=/usr --enable-usb-redir --enable-sdl -enable-slirp

#### 鼠标偏移
`-usbdevice tablet`
因为存在鼠标偏移的问题，所以在使用VNC方式启动客户机时，强烈建议将“-usb”和“-usbdevice tablet”这两个USB选项一起使用，从而解决上面提到的鼠标偏移问题。​“-usb”参数开启为客户机USB驱动的支持（默认已经生效，可以省略此参数）​，而“**-usbdevice tablet**”参数表示添加一个“tablet”类型的USB设备。​“tablet”类型的设备是一个使用绝对坐标定位的指针设备，就像在触摸屏中那样定位，这样可以让QEMU能够在客户机不抢占鼠标的情况下获得鼠标的定位信息。

#### 非图形模式

在qemu命令行中，添加“-nographic”参数可以完全关闭QEMU的图形界面输出，从而让QEMU在该模式下成为简单的命令行工具。而在QEMU中模拟产生的串口被重定向到了当前的控制台（console）中，所以在客户机中对其内核进行配置使内核的控制台输出重定向到串口后，依然可以在非图形模式下管理客户机系统或调试客户机的内核。





## qemu resize 分区大小
qemu-img resize xxx +2G

apt install -y cloud-guest-utils
### 修改分区信息
growpart /dev/vdb 1
df -h

### 刷新文件系统
resize2fs /dev/vdb1

参照AppArmor权限问题
https://winddoing.github.io/post/de22fd34.html




