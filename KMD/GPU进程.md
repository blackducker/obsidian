## 进程gtt映射表基地址写入
用户OpenGL会创建一个context，在intel用户态驱动mesa中，用户context会对应创建一个gem context。内核驱动根据硬件有多少个引擎(video, render, blitter等等) 创建多少个intel context(驱动内部使用)。每个引擎对应一个intel context。每个intel context会创建出一个logical context, logical context保存了在硬件上切换任务时候的上下文，保存了一些硬件的寄存器状态。函数intel_engine_context_size用来获取intel不同genX logical context的大小，最新的gen12 logical context 14个page size。Logical Context包含三个部分:
- Per-process HW Status Page(4K)
- Ring Context (RingBuffer Control Registers, Page Directory Pointers)
- Engine Context (PipelineStatu, Non-pipelineState, Statistics, MMIO)
Ring Context 中保存了Page Directory Pointers，也就是用来保存转换表的基地址。ppgtt转换分配在内存中。在驱动i5_gem_create_context -> i915_ppgtt_create -> gen8_ppgtt_create。 也就是一个用户态的gem context有一个ppgtt的映射转换表。 驱动中存在gen6和gen8两种ppgtt接口实现。

ppgtt指向的内存是一个在system上分配的buffer。

ppgtt_init函数，初始化了vm同时绑定了vma的操作函数。并且初始化了i915中的vm(gpu进程地址空间)