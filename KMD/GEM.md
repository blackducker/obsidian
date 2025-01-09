## mesa

``` C
ret = drmIoctl(fd, DRM_IOCTL_AMDGPU_GEM_MMAP, &map);
sim_bo->mmap_offset = map.out.addr_ptr;

sim_bo->gem_vaddr = mmap(NULL, sim_bo->size,PROT_READ | PROT_WRITE, MAP_SHARED,
fd, sim_bo->mmap_offset);
```
## gem ioctl

``` C
DRM_IOCTL_DEF_DRV(AMDGPU_GEM_CREATE, amdgpu_gem_create_ioctl, DRM_AUTH|DRM_RENDER_ALLOW),

DRM_IOCTL_DEF_DRV(AMDGPU_GEM_MMAP, amdgpu_gem_mmap_ioctl, DRM_AUTH|DRM_RENDER_ALLOW),
```


1. 
	amdgpu_gem_create_ioctl 创建一个内存对象，并实例化成一个dma-buf，返回handle
``` C
	amdgpu_bo_create_user(adev, &bp, &ubo);
		(*obj)->funcs = &amdgpu_gem_object_funcs;
```

2. 
	amdgpu_gem_mmap_ioctl 拿到内存对象的虚拟地址偏移量
``` C
*offset_p = amdgpu_bo_mmap_offset(robj);
		drm_vma_node_offset_addr(&bo->tbo.base.vma_node);
```

3. addr = mmap(NULL, size, PROT_READ | PROT_WRITE, MAP_SHARED, fd, mmap_offset);
``` C
sim_bo->gem_vaddr = mmap(NULL, sim_bo->size, PROT_READ | PROT_WRITE, MAP_SHARED,
						 fd, sim_bo->mmap_offset);
```
	drm_gem_dmabuf_mmap
		drm_gem_prime_mmap
这个时候，如果在初始化gem_obj的时候设置了mmap函数以及vm_ops，则会调用。其中需要关注的是vm_ops.fault函数`amdgpu_gem_fault`以及mmap函数`amdgpu_gem_object_mmap`.
``` C
int drm_gem_prime_mmap(struct drm_gem_object *obj, struct vm_area_struct *vma)
{
	/* Add the fake offset */
	vma->vm_pgoff += drm_vma_node_start(&obj->vma_node);
	
	if (obj->funcs && obj->funcs->mmap) {
		vma->vm_ops = obj->funcs->vm_ops;

		drm_gem_object_get(obj);
		ret = obj->funcs->mmap(obj, vma);
		if (ret) {
			drm_gem_object_put(obj);
			return ret;
		}
		vma->vm_private_data = obj;
		return 0;
	}

}

```

## prime import/export
DRM提供了两个ioctl命令，分别对应导入和导出：
``` C
DRM_IOCTL_DEF(DRM_IOCTL_PRIME_HANDLE_TO_FD, drm_prime_handle_to_fd_ioctl, DRM_RENDER_ALLOW)

DRM_IOCTL_DEF(DRM_IOCTL_PRIME_FD_TO_HANDLE, drm_prime_fd_to_handle_ioctl, DRM_RENDER_ALLOW)
```

DMA Fence
``` C
DRM_IOCTL_DEF_DRV(AMDGPU_WAIT_CS, amdgpu_cs_wait_ioctl, DRM_AUTH|DRM_RENDER_ALLOW),

DRM_IOCTL_DEF_DRV(AMDGPU_WAIT_FENCES, amdgpu_cs_wait_fences_ioctl, DRM_AUTH|DRM_RENDER_ALLOW),
```

i915_gem_object_wait_reservation