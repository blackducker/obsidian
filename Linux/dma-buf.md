
## DMA API
### coherent DMA mapping
gmc_v12_0_sw_init
	r = dma_set_mask_and_coherent(adev->dev, DMA_BIT_MASK(44));
设定coherent DMA 操作的地址掩码

### streaming DMA mapping
1、map/umap单个的dma buffer


2、map/umap多个形成scatterlist的dma buffer
kfd_mem_dmamap_userptr
	ret = dma_map_sgtable(adev->dev, ttm->sg, direction, 0);

amdgpu_amdkfd_gpuvm_get_sg_table
	ret = sg_alloc_table(sg, chunks, GFP_KERNEL);
	for_each_sg(sg->sgl, s, sg->orig_nents, idx)

## 共享DMA缓冲区
导出者
- 实现并管理`strcut dma_bud_ops`中缓冲区的操作
- 允许其他用户通过使用`dma_buf`共享API来共享缓冲区
- 管理缓冲区分配的详细信息，包装在`dma_buf`结构体中
- 决定发生分配的实际后备存储
- 负责改缓冲区的所有(共享)用户的分散列表的任何迁移
缓冲用户
- 是缓冲区的(许多)共享用户之一
- 不需要担心缓冲区是如何分配的，或者在哪里分配的
- 并且需要一种机制来访问构成内存中缓冲区的分散列表，映射到自己的地址空间，以便它可以访问同一内存区域。该接口由`struct dma_buf_attachment` 提供
### 用户空间接口
大多数情况下，DMA缓冲区文件描述符只是用户空间的不透明对象，因此公开的通用接口非常少。但有一些事情需要考虑:
- 支持DMA缓冲区内容的内存映射
- DMA缓冲区FD是可查询的
- DMA缓冲区FD是支持一些dma-buf特定的ioctl
### 基本操作合设备DMA访问
设备DMA访问共享DMA缓冲区，通常需要采用下列步骤
- 导出者使用`DEFINE_DMA_BUF_EXPORT_INFO()`定义`dma-buf`实例，并调用`dma_buf_export()`,将私有缓冲区对象封装到dma_buf中. 然后，调用`dma_buf_fd()`将`dma_buf`作为文件描述符导出到用户空间。
- 用户空间将此文件描述符传递给它希望共享此缓冲区的所有驱动程序: 首先使用dma_buf_get()将文件描述符转换为dma_buf；然后使用dma_buf_attach()将缓冲区附加到设备。
- 一旦缓冲区连接到设备，用户空间就可以启动对共享缓冲区的DMA访问。在内核中，这是通过调用`dma_buf_map_attachnment()`和`dma_buf_unmap_attachnment()`来完成的。
- 一旦驱动程序使用完共享缓冲区，调用`dam_buf_detach()`,然后再调用`dma_buf_put()`释放使用`dma_buf_get()`的引用。

### 隐式围栏(fence)查询支持
为了支持缓冲区访问的跨设备和跨驱动程序同步，可以将隐式围栏(在内核内部用struct dam_fence表示)附加到dma_buf。dma_resv结构中提供了它的粘合剂和一些相关的东西。
用户空间可以使用poll()和相关系统调用，查询这些隐式跟踪的栅栏状态:
- 检查EPOLLIN (即读访问) 可用于查询最近的写，或独占栅栏的状态。
- 检查EPOLLOUT (即读访问) 可用于查询所有附加栅栏(共享栅栏和独占栅栏) 的状态。
请注意，这仅表示相应栅栏已完成，即DMA传输已完成。在CPU访问开始之前，缓存刷新和任何其他必要的准备工作仍然需要继续进行。
作为poll()的替代方案，可以使用dma_buf_sync_file_export将DMA缓冲区上的栅栏集导出为sync_file.


其中amdgpu通过drm_gem_object -> funcs -> export(amdgpu_gem_prime_export)来提供支持
``` C
struct dma_buf *amdgpu_gem_prime_export(struct drm_gem_object *gobj,
					int flags)
{
	struct amdgpu_bo *bo = gem_to_amdgpu_bo(gobj);
	struct dma_buf *buf;

	if (amdgpu_ttm_tt_get_usermm(bo->tbo.ttm) ||
	    bo->flags & AMDGPU_GEM_CREATE_VM_ALWAYS_VALID)
		return ERR_PTR(-EPERM);

	buf = drm_gem_prime_export(gobj, flags);
	if (!IS_ERR(buf))
		buf->ops = &amdgpu_dmabuf_ops;

	return buf;
}
```
``` C
const struct dma_buf_ops amdgpu_dmabuf_ops = {
    .attach = amdgpu_dma_buf_attach,
    .pin = amdgpu_dma_buf_pin,
    .unpin = amdgpu_dma_buf_unpin,
    .map_dma_buf = amdgpu_dma_buf_map,
    .unmap_dma_buf = amdgpu_dma_buf_unmap,
    .release = drm_gem_dmabuf_release,
    .begin_cpu_access = amdgpu_dma_buf_begin_cpu_access,
    .mmap = drm_gem_dmabuf_mmap,
    .vmap = drm_gem_dmabuf_vmap,
    .vunmap = drm_gem_dmabuf_vunmap,
};
```

``` C
amdgpu_dma_buf_attach()
    if (pci_p2pdma_distance(adev->pdev, attach->dev, false) < 0)
        attach->peer2peer = false;
```
amdgpu在dma-buf的attachment加入了一个函数指针dma_buf_attach_ops，在调用dma_buf_move_notify时回调该ops函数，以调用amdgpu_dma_buf_move_notify()
``` C
struct dma_buf_attachment *
dma_buf_dynamic_attach(struct dma_buf *dmabuf, struct device *dev,
               const struct dma_buf_attach_ops *importer_ops, void *importer_priv)
```

``` C
void dma_buf_move_notify(struct dma_buf *dmabuf)
{
	struct dma_buf_attachment *attach;

	dma_resv_assert_held(dmabuf->resv);

	list_for_each_entry(attach, &dmabuf->attachments, node)
		if (attach->importer_ops)
			attach->importer_ops->move_notify(attach);
}
EXPORT_SYMBOL_NS_GPL(dma_buf_move_notify, DMA_BUF);
```

dma_buf_move_notify 通知所有attachments，dmabuf已经被移动，需要销毁或者重新创建映射
amdgpu_bo_move_notify -> dma_buf_move_notify

### DMA Fence
DMA fence, 由struct dma_fence 表示，是用于DMA操作的内核内部同步原语，例如GPU渲染， 视频编解码或在屏幕上显示缓冲区。
Fence通过dma_fence_init()进行初始化，并通过dam_fence_signal()进行完成。Fence与上下文相关联，通过`dma_fence_context_alloc()`分配,同一上下文中的所有Fence都是完全有序的。
由于Fence的目的是促进跨设备和跨应用程序的同步，有多种使用方式:
- 可以将单个Fence公开为sync_file，从用户空间作为文件描述符访问，通过调用sync_file_create()创建。这称为显示Fence, 用户空间传递显示同步点。
- 一些子系统还具有自己的显示Fence 原语，如drm_syncobj。 与sync_file相比，drm_syncobj允许更新底层栅栏。
- 还有隐式栅栏，其中同步点作为共享的`dma_buf`实例的一部分隐式传递。这种隐式栅栏存储在通过`dma_buf.resv`指针的`struct dma_resv`中.

### DMA RESV (Reservation Objects)
amdgpu_gem_object_create
	amdgpu_bo_create() -> drm_gem_private_object_init() -> dma_resv_init(&obj->_resv);
	(*obj)->funcs = &amdgpu_gem_object_funcs | .export = amdgpu_gem_prime_export

考虑如下场景
一个dmabuf，先由GPU完成渲染，然后再交给DRM进行显示输出。那么GPU渲染完成后，如何通知DRM进行显示输出呢？也就是GPU和DRM之间如何进行同步?这就需要引入fence用于设备间的同步，fence用于表示一个操作的完成状态，故fence有两个状态，not done和done。
首先GPU在开始渲染操作前，创建一个fence，注册回调函数，将fence添加到dmabuf中，随后DRM等待该fence done。当GPU渲染完成中断上来后，会通知fence done。随后DRM线程被唤醒，进行显示操作。
另外，dma fence还需要考虑多设备访问的情况，即有可能有多个设备在等待fence完成，那么fence就必须支持多个设备的等待。

举例说明，GPU线程会在操作dmabuf前，创建fence，并等待fence完成，同时DRM也会等待该fence完成。当GPU渲染完成中断产生后，会调用fence done，依次唤醒GPU DRM线程，GPU线程可以继续下一帧图像的渲染，而DRM就可以将已经完成渲染的图像显示到屏幕。
这个过程中调用的接口有:
1. dma_fence_init(): 初始化一个dma_fence对象
2. dma_resv_reserve_shared(): 从dma resv 中保留一个share fence指针
3. dma_resv_add_shared_fence(): 将dma fence添加到resv对象
4. dma_fence_default_wait(): 向dma fence注册回调函数dma_fence_default_wait_cb, 并睡眠等待dma fence完成
5. dma_fence_signal(): 标志dma fence完成，并回调dma fence中的所有回调函数
其中有一个dma_resv对象，简单来说dma_resv是一个存放dma_fence的地方，一个dmabuf可能同时有若干个dma fence，且dma fence还有共享和独占两种。dma_resv可以理解为一块内存区域，专门存放dma fence，故要将dma fence添加到dmabuf时，要先调用dma_resv_reserve_shared()预留出dma fence的位置，然后再调用dma_resv_add_shared_fence()添加到dma resv。


### DMA Fence作用


fence 同步机制可以确保 buffer 在共享过程中，驱动程序或用户空间不会对一个正在被写入的 buffer 进行读操作，或对一个仍在被其他模块使用的 buffer 进行写操作。fence 确保了这些操作能够有序进行，即只会在 buffer 不被使用时才进行读写操作。例如，当一个GPU的job被塞进队列时，该job中的buffer会被关联上一个fence，其他驱动程序可以借助该fence来进行同步。在收到该fence的信号之前，这些驱动程序不会对该buffer进行任何操作，fence signal表明该buffer可以被正常使用了。同样地，为了让GPU驱动能等待buffer从显示屏中切换出来，我们可以给显示驱动采用相同的fence机制，以便GPU能再次使用这些buffer进行渲染。
fence时该机制的核心元素，每当向kernel发送buffer相关的请求时，该buffer都会附带一个fence。用户空间或者其他驱动程序可以使用fence来等待硬件工作完成。因此一旦工作完成，该fence就会被signal，等待该fence的模块也就可以继续对该buffer进行他们想要的操作了。



虽然implicit Fence在buffer同步方面起了很大作用，但在某些情况下，整个桌面的合成可能会被卡住。想象一下下面的合成过程：有ABC三个buffer需要处理，A和B被GPU拿去同时做渲染，而C将作为A与B合成的结果拿去显示。但只有在AB这两个buffer都渲染完成时才会通知compositor做合成，因此如果B的渲染时间太长，将导致整个桌面的合成因为需要等待B而被block住，于是C就无法及时的显示出来。

### sync file

DRM实现Explicit Fence需要建立在两个kernel基础框架之上: `struct dma_fence` -- 该结构体表示fence，和`struct sync_file` -- 用来提供与用户空间共享的文件描述符. 在使用fence时，dispaly底层驱动需要等待该fence发出信号后，才能在屏幕上显示该buffer。在Explicit Fence的实现中，fence从用户空间发送给内核空间，display底层驱动同样会给用户空间返回一个新的fence，该fence被封装在sync_file结构体中，当该buffer被显示到屏幕上时，它所对应的fence就会被signal。同样的，在渲染测也采用相同的等待流程。
必须使用atomic modesetting。我们不打算支持legacy API。 DRM要等待fence需要通过每个DRM plane