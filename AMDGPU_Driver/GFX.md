# GFX
- [GFX, Graphics and Compute Engine]()
- SMU, System Management Unit
#### gfx_v10_0_early_init
##### spm
##### kiq_pm4
- [KIQ, Kernel Interface Queue](https://confluence.amd.com/pages/viewpage.action?pageId=109835219)
  TBD 通过pdf文件理解KIQ
##### ring
###### gfx_v10_0_ring_funcs_kiq
###### (multi) gfx_v10_0_ring_funcs_gfx
###### (multi) gfx_v10_0_ring_funcs_compute
##### irq
##### gds
##### rlc
- run list 
##### mqd

#### gfx_v10_0_sw_init
##### gfx_v10_0_gfx_ring_init
###### amdgpu_ring_init
##### gfx_v10_0_compute_ring_init

#### resume --> gfx_v10_0_hw_init
#### gfx_v10_0_hw_init
##### gfx_v10_0_constants_init
##### gfx_v10_0_rlc_resume
- gfx_v10_0_rlc_load_microcode ->  WREG32_SOC15(fw_data++) `TBD`
##### gfx_v10_0_cp_resume `TBD`