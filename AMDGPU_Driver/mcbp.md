# MCBP

## 流程图 

### amdgpu_debugfs_ib_preempt

``` c 
debugfd->amdgpu_debugfs_ib_preempt
    kthread_park
        amdgpu_fence_process
```
