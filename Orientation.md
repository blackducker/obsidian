# Orientation总结


## 开发职责
### 角色
- Linux Graphics Dev (NPI work)
- Display Dev(维护最新的内核版本)

### 职责
- 项目经理
- mentor
- 个人
- 测试反馈
  
### 工作范围`仅包含kernel部分，涉及opengl需要确认其中kernel的部分`
#### install
- driver
#### kernel
- modprobe
- amdgpu_test
- kfdtest
- amdgpu_blacklist
- mcbp
#### basic
- startx
#### multimedia
- vdpauinfo
- playback_vdpau
- playback_ogl
#### opengl
- unigine_heaven
- glmark2
- specviewperf11
- disaster
#### opencl
- clinfo
- kfft
- conformance_reallyquick
- luxmark
#### power
- s3
- sclk
#### ras
- gpu_recovery
  
## 开发流程
### cherry-pick
- drm-next
- dkms-5.18, 时刻保持与next对齐，以及向后兼容
- amdkcl & autoconf 


### bugfix
- reproduce
- update
- regression
- open a JIRA ticket

## 工程框架
### T.B.D
### usrspace
- wayland
- vulkan
- mesa
- Rocm
#### libdrm
- 底层 ioctl
- 上层 `用户层接口有哪些(范例)`

#### libdrm_amdgpu

### kernel
#### DRM core
##### GEM
- UMB
- PRIME
- fence
##### KMS
- CRTC
- ENCODER
- CONNECTOR
- PLANE
- FB
- VBLANK
- property
#### amdgpu


## 项目代码
###  brahma`仅关注三部分主体代码` 

#### libdrm
##### libdrm_amdgpu
- basic test: 执行基本的功能验证
- bo test: 内存分配的功能验证
- ip test(jpeg, ras, uvd, vcn) 
- disaster test(deadlock, hotunplug, security)
- other test(syncobj, cp_dam, cs) 
#### firmware
- make 流程，生成fw_gen后根据不同的显卡输出对应IP组合的binary
- raw 存储firmware的头文件，以供调用程序使用
- src 包含通过fw_gen做为入口调用不同的c文件进行分流处理
#### gpu driver(linux dkms)
- amdkcl & autoconf `拿掉相应的patch，观察错误现象`
- driver init 顺序
- ip init 顺序
- ip 功能
- ip selftest

## 测试

### 测试环境
#### kernel
- 编译 bbh脚本, 对于config文件的使用
- 安装 reboot CI到指定内核，-k <version> -l
#### firmware
- 编译 make工程组织，Makefile
- 安装以及加载 由于firmware加载的差异，在安装FW之后，应进行系统重启以生效
#### dkms
- 编译 ln链接conf文件，然后打了一个tar包
- ~~安装 1.amdpro(会装其他什么？) 2.dkms install~~

### 测试方法
- 测试 测试dkms时，与linux kernel的冲突或者其他环境冲突吗？
- 主要测试dkms，怎样明确问题在哪
- 如何解读测试报告


### 测试
#### Firmware integration test
##### 测试意义
- 联合firmware，进行相关功能性测试
##### 测试内容
- ...
#### RAS Integration Test
##### 测试意义
- 针对于服务器级别，安全记录等功能
##### 测试内容
- ...
#### DTP test on Tier 1 OS
##### 测试意义
- 与FW integration test差异? 侧重点不同，算是Integration的集合测试，在Integration后执行。
##### 测试内容
- modeprobe 
- amdgpu unit test 
- kfd unit test
- startx
- glxgears
- clinfo
- rocminfo
- vulkaninfo
#### Mode1 reset
##### 测试意义
- 针对与显卡，自动回复测试
##### 测试内容
- ...

## 学习经验
- 确认事项: 工作流程是怎样的? Task,bug分配好，第一时间就应该明确的事项是什么?诸如有些bug已经修复过
- 主动推进: 如何去主动推进Task，每日汇报，每周汇报？ 对应到给你这个task的人，还是自己的mentor(developer 互相交流)
- 深入思考: 现阶段主要是学习，后面的大多数时间应该都是边做边学，那么这之间的时间是怎么平衡的. 日常积累知识的话，程度是怎样的，或者说以什么样的角色去处理问题.(学习日，学习会议啥的)
- 获取答案: 按照现在的工作风格都是主动去问，以后也是这样吗? 问题应该自己先消化，然后提出来. 那些提问题的标准是？(自己卡住一段时间，问的时候有些不好意思。而且对于背景了解不深很难快速沟通对齐，感觉自己学的还是不够)还是应该学会从多方向获取答案，大家都是怎么做的

