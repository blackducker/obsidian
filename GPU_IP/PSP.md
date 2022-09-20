
## psp
- [TMZ, Trusted Memory Zone](https://confluence.amd.com/display/AMDGPU/TMZ+Support)
### early_init
#### psp_v11_0_set_psp_funcs
#### ring
- `init, create, stop, destroy(TBD)`
#### bootloader_load
- `kdb, spl, sysdrv, sos(TBD)`
#### mem_training

### psp_sw_init
#### psp_init_microcode 
##### psp_init_ta_microcode
#####  psp_init_sos_microcode
######  psp_init_sos_base_fw --> parse_sos_bin_descriptor
#### psp_memory_training_init --> psp_mem_training
- `TBD`

### psp_hw_init 
#### amdgpu_ucode_init_bo
- `TBD`
#### psp_load_fw
##### psp_ring_init(psp, PSP_RING_TYPE__KM);
##### psp_hw_start --> psp_ring_create --> psp_tmr_load --> psp_cmd_submit_buf
##### psp_load_non_psp_fw
##### psp_initialize

### psp_late_init
- `TBD`

### psp_resume
#### psp_mem_training(psp, PSP_MEM_TRAIN_RESUME);
#### psp_hw_start
#### psp_load_non_psp_fw
#### psp_initialize

### psp_suspend
- `TBD`