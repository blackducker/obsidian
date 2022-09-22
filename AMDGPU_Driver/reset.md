# mode1 rest

## 流程(amdgpu_device_gpu_recover)

### amdgpu_device_lock_reset_domain 
-  increase gpu_reset counter and lock reset
   atomic_inc(&tmp_adev->gpu_reset_counter)

### cancel_delayed_work_sync
#### cancel a delayed work and wait for it to finish
#### cancel_delayed_work - cancel a delayed work

### drm_sched_stop 
### dma_fence_is_signaled(&job->hw_fence)
#### job struct have only one dma_fence(hw_fence)

### amdgpu_device_pre_asic_reset
#### drm_sched_increase_karma
- Update sched_entity guilty flag
  因为drm_sched_stop,未执行job应该被置成guilty
  TBD, 调查drm_sched_increase_karma的用法，关联drm_sched
#### amdgpu_device_ip_suspend
- suspend all IP blocks reversively
##### amdgpu_device_ip_suspend_phase1
- handle DCE(AMD_IP_BLOCK_TYPE_DCE), Display and Compositing Engine
  the ip block is updated to DCN, Display Core Next
  TBD, Why it first? 
  refer to [https://confluence.amd.com/display/SWDAL/Introduction+To+The+Display+Pipe](https://confluence.amd.com/display/SWDAL/Introduction+To+The+Display+Pipe)

##### amdgpu_device_ip_suspend_phase2
- handle other



### amdgpu_do_asic_reset
#### amdgpu_asic_reset
- soc包括所有的ip, 但是soc reset不代表ip reset
##### soc15_asic_reset_method --> amdgpu_device_mode1_reset
######  "GPU smu mode1 reset\n" --> amdgpu_dpm_mode1_reset
- SMU system manegement unit 系统管理单元其实就是电量管理
  调用到smu的ppt_function
  `atombios TBD`
  
#### amdgpu_device_asic_init

#### amdgpu_device_ip_resume_phase1 
##### COMMON resume
- pcie_gen3, aspm, nbio, hdp, doorbell
##### GMC resume
- gart_enable, -->  amdgpu_gtt_mgr_recover(MMHUB, GFXHUB)
   rebind GTT pages
- vram_checking --> bo_create(AMDGPU_GEM_DOMAIN_VRAM) 对比内存start, mid, end的内存数据

##### IH resume
- ih_v6_0_irq_init
- nbio.funcs->ih_control(adev)
- ih_v6_0_enable_ring -> register setting

#### amdgpu_device_check_vram_lost  (not correct)
- Checks the reset magic value written to the gart pointer in VRAM.
- adev->gart.ptr指向 gmc init的申请的gart table内存块
 amdgpu_gart_table_vram_alloc -> amdgpu_bo_create_kernel(adev,  adev->gart.table_size,
 PAGE_SIZE, AMDGPU_GEM_DOMAIN_VRAM, &adev->gart.bo, NULL, (void *)&adev->gart.ptr);
#### amdgpu_device_fw_loading
##### psp_resume
######  psp_hw_start
- bootloader_load_kdb， spl等 -> psp bootloader 加载自己的运行组件(从sos.bin中解析) 因为没有RB同步load_fw
- psp_ring_create -> 初始化时已经初始化ring buffer内存，create将相关内存信息写入寄存器进行RB的配置
- psp_tmr_init -> Set up Trusted Memory Region, psp->tmr_mc_addr
- psp_tmr_load
###### psp_load_non_psp_fw -> 初始化在gfx_v10_0_me_init -> gfx_v10_0_init_microcode
- 遍历ucode list加载fw -> amdgpu_firmware_info *ucode
  AMDGPU_UCODE_ID 下有gfx-cp-ce,pfp,me,mec gfx-rlc
- AMDGPU_UCODE_ID_RLC_G -> psp_rlc_autoload_start
  RLC作为gfx的一个block包含一个微控制器F32支持三个线程，每个线程都是单独的控制器有自己的Ucode
  因此分为RLC_G,RLC_V,RLC_P三个Ucode

###### psp_asd_initialize
- load ta里的asd， ta_load_type=GFX_CMD_ID_LOAD_ASD
- psp_ta_load -> psp_copy_fw 拷贝到内核内存地址
- psp_prep_ta_load_cmd_buf
- psp_cmd_submit_buf
###### psp_rl_load
- 最后load  GFX_FW_TYPE_REG_LIST, 单独拎出来最后处理

###### ta_fw
- psp_ras_initialize
- psp_hdcp_initialize
- psp_dtm_initialize
- psp_rap_initialize
- psp_securedisplay_initialize

#### amdgpu_device_ip_resume_phase2
- gfx resume 加载gfx的ucode
##### ring test 
- `TBD`

####  amdgpu_irq_gpu_reset_resume_helper
#### amdgpu_ib_ring_tests
##### gfx_v10_0_ring_test_ib
#### amdgpu_device_recover_vram

### drm_sched_resubmit_jobs
### drm_sched_start
### amdgpu_device_unlock_adev


<!-- --------------------------------------------------------- -->
## IP

### amdgpu_device_ip_init
#### amdgpu_ras_init
#### num_ip_blocks
##### adev->ip_blocks[i].version->funcs->sw_init
##### amdgpu_device_vram_scratch_init
##### AMD_IP_BLOCK_TYPE_GMC --> adev->ip_blocks[i].version->funcs->hw_init
##### amdgpu_device_wb_init

#### amdgpu_ucode_create_bo
##### amdgpu_bo_create_kernel --> fw_buf
#### amdgpu_device_ip_hw_init_phase1
#####  COMMON PSP IH
#### amdgpu_device_fw_loading
#### amdgpu_device_ip_hw_init_phase2
#### amdgpu_ras_recovery_init
#### amdgpu_device_init_schedulers
##### drm_sched_init 调度器内核线程初始化
#### amdgpu_fru_get_product_info

### [psp](../GPU_IP/PSP.md)

### [gfx](../GPU_IP/GFX.md)

<!-- --------------------------------------------------------- -->
## sched
### 数据结构
#### drm_sched_entity
#### drm_sched_rq
#### drm_sched_fence
#### drm_sched_job
#### drm_sched_backend_ops
#### drm_gpu_scheduler

### function
#### amdgpu_cs_submit
##### drm_sched_job_arm
##### dma_fence_get


### 范例流程
#### amdgpu_cs_ioctl
##### amdgpu_cs_submit
#### drm_sched_init
##### drm_sched_main --> sched->thread = kthread_run(drm_sched_main, sched, sched->name);
##### amdgpu_sched_ops --> sched->ops = ops; 
###### amdgpu_job_run
###### amdgpu_job_timedout

<!-- --------------------------------------------------------- -->
### dma-fence


<!-- --------------------------------------------------------- -->
## BO
### 数据结构
 ttm_buffer_object


### function
#### amdgpu_bo_create_reserved
#### amdgpu_bo_create_kernel


### 范例流程

<!-- --------------------------------------------------------- -->
## AMDGPU Virtual Memory
### 数据结构
#### struct amdgpu_prt_cb
#### struct amdgpu_vm_tlb_seq_cb
### function
### 范例流程

<!-- --------------------------------------------------------- -->
## Interrupt Handling

<!-- --------------------------------------------------------- -->
## RAS `TBD`
### 数据结构
### function
### 范例流程

## Q&A
### amdgpu_discovery_set_ip_blocks / add ip block number
#### hdp psp 