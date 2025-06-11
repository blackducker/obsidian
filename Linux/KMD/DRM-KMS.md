在各类SOC上，CRTC+Endode+Connector一般是集成在一个外设模块挂在系统总线上
我们按照connector-->encoder-->crtc-->framebuffer的顺序倒过来介绍吧
## **Connector**
Connector其实就是和显示器连接的物理接口，常见的有VGA/HDMI/DVI/DP等。
## **Encoder**
Encoder比较好理解，在此处其实就是将一定格式的图像信号（如RGB、YUV等）编码成connector需要输出的信号。
## **CRTC**
CRTC的任务是从Framebuffer中读出待显示的图像，并按照相应的格式输出给Encoder

## **Planes**
Plane其实就是图层，实际输出的图像往往由多个图层叠加而成（想象一下photoshop的过程）

## **Framebuffer**
Framebuffer对应着存储空间中的图像数据，此处对应硬件为DDR。

我们知道Framebuffer是存储待显示图像信息的空间，因此，Framebuffer相关驱动中也就是对内存的操作，也就涉及到下面两个部分：  
① 对内存的管理（如GEM，for Graphics Execution Manager）  
② 内存中数据的更显方式（如DMA等）

对于第一点，GEM主要完成的事情是：  
① 对图像内存（显存）的空间开辟、释放；  
② 不同硬件对同一显存资源访问下的管理；
因为对于ARM而言，framebuffer是system memory，通过分配一块连续的内存CMA来实现gem的底层，也不需要增加iommu的支持，在mmap接口中直接映射`remap_pfn_range()`所有的gem到用户空间。
在Linux Kernel下的默认实现方式是CMA（Contiguous Memory Allocator）实现的，内核中对应代码是：drivers/gpu/drm/drm_gem_dma_helper.c

Linux原生系统中提供由DRM+KMS构成的DRI（Direct Rendering Infrastructure）中：
- DRM主要负责负责数据流，即通过软件或硬件，生成目标图像，存储在framebuffer中；
- KMS主要负责控制流，即针对外置LCD以及指定的显示模式设置，将生成好了的frame数据信息送到响应display port上（VGA、HDMI等）；
Kernel将这两大块的基本API抽出来封装成libdrm供X使用，整个应用层+kernel相关的GUI结构如下图：