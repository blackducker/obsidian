
DRM是linux内核子系统，负责与显卡交互。DRM提供了一组API，用户空间程序可以使用该API将命令呵护数据发送到GPU并执行诸如配置显示器的模式设置之类的操作。

Linux 内核已经有fbdev的API，但是不能满足3D加速需求
当两个或多个程序试图同时控制相同的硬件，会导致问题。
DRM的创建时为了允许多个程序同时使用视频硬件资源。DRM获得对GPU的独占访问权，并负责初始话和维护命令队列，内存和任何其他资源。希望使用GPU的程序将请求发送刀片DRM，DRM充当仲裁程序，并规避可能的冲突

多年来，DRM的范围已得到扩展，以涵盖以前由用户空间程序处理的多重功能，例如==framebuffer管理==和模式设置，==内存共享对象==和==内存同步==。这些扩展中的某些被赋予特定的名称，例如 图形执行管理器(GEM) 或 内核模式设置(KMS)

# Software architecture

DRM驻留在内核空间中，因此用户空间程序必须使用内核系统调用来请求其服务。但是DRM没有定义自己的自定义调用。相反，它遵循Unix原则“一切皆文件”，使用/dev层次结构下的设备文件通过文件系统名称空间公开GPU。DRM检测到的每个GPU都称为DRM设备，并创建了一个设备文件/dev/dri/cardX（X是一个序列号）与之连接，并使用ioctl调用与DRM进行通信。不同的ioctl对应于DRM API的不同功能。

 创建了一个名**为==libdrm==的库**，以方便用户空间程序与DRM子系统的接口。该库只是一个包装器，它为DRM API的每个ioctl提供用C编写的函数，以及常量、结构和其他辅助元素。使用libdrm不仅避免将内核接口直接暴露给应用程序，而且还提供了在程序之间重用和共享代码的优点。

**DRM由两部分组成：通用“DRM core”和每种受支持的特定部分（“DRM Driver”）**。DRM core提供了可以注册不同DRM驱动程序的基本框架，还为用户空间提供了具有通用的，独立于硬件的，功能的最少ioctl集。另一方面，DRM Driver实现API的硬件相关部分，具体取决于它所支持的GPU类型，它应提供DRM核心未涵盖的其余ioctl的实现。

 DRM驱动也可以扩展API，提供特定GPU上可用的具有附加功能的附加ioctl。当特定的DRM驱动程序提供增强的API时，用户空间libdrm也将通过一个额外的库libdrm-driver扩展，这个扩展库可以被用户空间用来调用其他ioctl接口。

##  API
DRM Core将几个接口导出到用户空间应用程序，让相应的libdrm包装成函数后来使用。
DRM Driver通过ioctl和sysfs文件导出设备专用的接口，供用户空间驱动程序和支持设备的应用程序使用。**外部接口包括：==内存映射==，==上下文管理==，==DMA操作==，==AGP管理==，==vblank控制==，==fence管理==，==内存管理和输出管理==**。

## GEM
Graphics Execution Manager（GEM）是一种内存管理方法。由于视频存储器的大小增加以及诸如OpenGL之类的图形API的日益复杂性，从性能角度看，在每个上下文切换处重新初始化图形卡状态的策略过于低效。另外，现代linux桌面还需要一种最佳方式与合成管理器（compositing manager）共享屏幕外缓冲区。这些要求诞生开发了用于管理内核内部图形缓冲区的新方法，图形执行管理方法（GEM）是其中一种。

GEM为API提供了显式的内存管理原语。通过GEM，用户空间程序可以创建，处理和销毁GPU视频内存中的内存对象（Bo）。从用户空间程序的角度看，这些被称为GEM的对象是持久性的，不需要在程序每次重新获得对GPU的控制时，重新加载他们。当用户空间需要大量的视频内存(以存储frame buffer，纹理或者GPU所需的任何其他数据)时，它将使用GEM API请求分配。DRM驱动程序会跟踪已使用的视频内存，如果有可用的空闲内存，==将句柄返回给用户空间，在以后的操作中进一步引用分配的内存==. GEM API 还提供了一些操作，以填充BO并在不需要时释放。当用户空间进程有意或者由于终止而关闭DRM设备文件描述符，未释放的GEM内存将恢复.

==GEM还允许使用同一个DRM设备的两个或者多个用户空间进程共享GEM对象==。 GEM句柄是某个进程唯一的局部32位整数，但在其他进程中可重复，因此不适合共享。所以需要一个全局命名空间，GEM通过使用称为GEM names的全局句柄来提供一个命名空间。GEM名称是指使用相同的DRM驱动程序在同一个DRM设备中通过使用唯一的32位整数创建一个GEM对象，并且仅是一个GEM对象. GEM提供了一个操作flink来从GEM句柄获取GEM名称。然后该进程可以使用任何可用IPC机制将GEM名称传递给一个进程。

~~不幸的是，使用GEM名称共享BO并不安全。一个恶意的第三方进程访问同一DRM设备可以通过探查32整数来猜测两个进程共享的缓冲区GEM名称~~。==后来通过在DRM中引入了DMA_BUF支持来克服此缺点，因此DMA_BUF将用户空间中的BO表示为文件描述符，可以安全的共享它们。==

除了管理视频内存空间外，任何视频内存管理系统的另一个重要任务是处理GPU和CPU之间的内存同步。当前的存储器架构非常复杂，并且通常涉及用于SRAM以及有时也用于VRAM的各种级别的高速缓存。因此，视频内存管理器还应处理缓存一致性，以确保CPU和GPU之间共享的数据一致，这意味着VRAM通常高度依赖于GPU和存储体系结构的硬件细节，因此取决于驱动程序。

GEM最初是由英特尔工程师开发的，旨在为其i915驱动程序提供VRAM管理。英特尔GMA家族具有统一内存架构(UMA)的集成GPU，其中GPU和CPU共享物理内存，并且没有专用的VRAM。GEM定义了用于内存同步的==内存域==，尽管这些内存域与GPU无关, 但他们在设计时特别考虑了UMA内存架构，这使其不适用于其他内存架构，例如具有单独VRAM的内存架构。因此，其他的DRM驱动程序已决定向用户空间程序公开GEM API，但在内部他们实现了更适合其特定硬件和内存体系结构的其他内存管理器。

GEM API还提供了用于控制执行流(ring buffer)的ioctl,除了特定于内存管理的ioctl，没有DRM驱动程序试图去实现GEM API的其他任何部分。

## TTM (Translation Table Maps)

 Translation Table Maps（TTM）是在GEM之前开发的用于GPU的通用内存管理器名称。它专门用于==管理GPU可能访问的不同类型的内存==，包括专用的Video RAM（通常安装在显卡中）与可通过称为Graphics Address Remapping Table（GART）的I/O内存管理单元访问的系统内存。考虑到用户空间图形应用程序通常会处理大量视频数据，TTM还应处理CPU无法直接寻址的Video RAM部分，并以最佳性能来实现。另一个重要的事情是保持所涉及的不同内存和缓存之间的一致性。

TTM的主要概念是BO，即在某些时刻GPU是可寻址的视频内存。当用户空间图形应用程序想要访问某个BO时，TTM可能需要将其重新定位为CPU可寻址的一种RAM。当GPU需要访问的BO还不在GPU的地址空间中时，可能还会发生进一步的重定位或者GART映射操作。这些重定位操作中的每一个都必须处理任何相关的数据和缓存一致性问题。

 TTM的另一个重要概念是围栏（==fences==）。围栏本质上是一种管理CPU和GPU之间并发性的机制。围栏跟踪GPU何时不再使用缓冲区对象，通常用于通知任何具有访问权限的用户空间进程.

 TTM试图以合适的方式管理所有类型的内存体系结构，包括有和没有专用VRAM的体系结构，并在内存管理器中提供所有可想到的功能，以便与任何类型的硬件一起使用，这导致了过于复杂的情况。API远远超出所需的解决方案。一些DRM开发人员认为它不适用于任何特定的驱动程序，尤其是API。==当GEM成为一种更简单的内存管理器时，它的API优于TTM==。但是一些驱动程序开发人员认为TTM所采用的方法更适合于具有专用Video RAM和IOMMU的分立显卡，因此他们决定在内部使用TTM，同时将其缓冲区对象公开为GEM对象，从而支持GEM API。==当前使用TTM作为内部内存管理器但提供GEM API的驱动程序实例包括AMD显卡的radeon驱动程序和NVIDIA显卡的nouveau驱动程序。==

## DMA Buffer Sharing and PRIME

 ==DMA缓冲区共享API（通常缩写为DMA-BUF）是一种Linux内核内部API，旨在提供一种通用机制来在多个设备之间共享DMA缓冲区，并可能由不同类型的设备驱动进行管理==。例如，Vdieo4Linux设备和图形适配器设备可以通过DMA-BUF共享缓冲区，以实现前者产生并由后者消耗的视频流数据的零复制。任何Linux设备驱动程序都可以将此API实现为导出器、用户（消费者）或者二者兼而有之。

在DRM中首次利用此功能来实现PRIME(显卡交火), 这是一种用于GPU卸载的解决方案，它使用DMA-BUF在独立显卡和集成显卡之间共享生成的framebuffer。DMA-BUF的一项重要功能是为了将共享BO作为文件描述符提供给用户空间。==为了开发PRIME，在DRM API中添加了两个新的ioctl, 一个将本地GEM句柄转换为DMA-BUF文件描述符，另一个用于相反的操作。==

后来这两个新的ioctl被用作解决GEM缓冲区共享固有的不安全性的一种方法。与GEM名称不同，无法猜测到文件描述符(它们不是全局命名空间) ， 并且Unix操作系统提供了一种安全的方法来传递Unix域套接字，通过使用SCM_RIGHTS语义。==一个想要与另一个进程共享GEM对象的进程可以将本地GEM句柄转换为DMA-BUF文件符，然后将其传递给接收者，后者可以从接收的文件描述符中获得自己的GEM句柄。==DRI3使用这种方法在客户端和X Server或者Wayland之间共享缓冲区。

==fd是独立于每个进程的，所以发送方的fd可能和接收方的fd值不一样。创建UNIX socket，设置cmptr->cmsg_type为SCM_RIGHTS.==


# Kernel Mode Setting
 **为了正常工作，显卡或者图形适配器必须设置一种模式（屏幕分辨率、颜色深度和刷新率的组合），该模式应在其自身和所连接的显示屏所支持的值的范围内。此操作称为mode-setting。通常需要对图形硬件进行原始访问，写入视频卡某些寄存器的能力。在开始使用framebuffer之前，以及在应用程序或者用户要求更改模式时，都必须执行模式设置操作。**
 
在早期，希望使用图形framebuffer的用户空间程序还负责提供模式设置操作，因此它们需要在对视频硬件进行特权访问的情况下运行。在Unix类型的操作系统中，X Server是最突出的示例，其模式设置实现存在于每种特定类型的显卡的DDX驱动程序中。这种方法（以后称为用户空间模式设置或UMS）带来了几个问题。这不仅打破了操作系统应在程序和硬件之间提供的隔离性，从而提高稳定性和安全性，而如果两个或多个用户空间程序尝试在操作系统上进行模式设置，可能会使图形硬件处于不一致的状态。同时，为了避免这种冲突，实际上X Server成为了唯一执行模式设置操作的用户空间程序。其余的用户空间程序依靠X Server来设置适当的模式并处理涉及模式设置的任何其他操作。最初，模式设置仅在X Server启动过程中执行，但是后来X Server可以在运行时进行设置。XFree86 3.1.2中引入了XFree-VidModeExtension扩展，以允许X-Client请求对X Server的modeline（分辨率）更改。VidModeExtension后来被更加通用的XRandR扩展取代。

然而，这不是在linux系统中进行模式设置的唯一代码。==在系统引导过程中，Linux内核必须为虚拟控制台设置最小文本模式（基于VESA BIOS扩展定义的标准模式）。==Linux内核帧缓冲驱动程序还包含用于配置帧缓冲设备的模式代码。为了避免模式设置冲突，XFree86 Server（以及后来的X.Org Server）在用户从图形环境切换到文本虚拟控制台时保存模式设置状态，并在切回到X图形环境时还原它。此过程在过渡中引发了闪烁现象，并且还可能失败，从而导致输出显示损坏或无法使用。

==为了解决这些问题，将模式设置代码移至内核中的单个位置，放到了现有的DRM模块中。==然后每个进程（包括X Server）都应该能够命令内核执行模式设置操作，并且内核将确保并发操作不会导致不一致的状态。添加到DRM模块以执行这些模式设置操作的新内核API和代码称为Kernel Mode-Setting(KMS)。


Kernel Mode-Setting(KMS)有几个好处。最直接是从内核（Linux控制台，fbdev）和用户空间（X Server DDX驱动程序）中删除重复的模式设置代码。KMS还使编写替代图形系统变得更加容易，而这些图形系统现在不需要实现自己的模式设置代码。通过提供集中的模式管理，KMS解决了在控制台和X之间以及X的不同实例之间切换引起的闪屏问题。由于它在内核中可用，因此也可以在引导过程开始时使用它，从而避免了由于这些早期阶段的模式更改而导致的闪烁。

为避免破坏DRM API的向后兼容性，内核模式设置作为某些DRM驱动程序的附件驱动程序功能提供。**任何DRM驱动程序在向DRM内核注册时都可以选择提供DRIVER_MODESET标志，以指示它支持KMS API**。那些实现内核模式设置的驱动程序通常被称为KMS驱动程序，以将它们与传统的（无KMS）DRM驱动。


### 2.9 Render nodes

         在原始DRM API中，DRM设备“/dev/dri/cardX”用于特权（模式设置，其他显示控件）和非特权（渲染，GPGPU计算）操作。 出于安全原因，打开关联的DRM设备文件需要“等同于root特权”的特殊特权。这导致了一种体系结构，其中只有一些可靠的用户空间程序（X服务器，图形合成器等）可以完全访问DRM API，包括特权部分（如模式集API）。 想要渲染或进行GPGPU计算的其余用户空间应用程序应由DRM设备的所有者（“DRM主设备”）通过使用特殊的身份验证界面来授予。然后，经过身份验证的应用程序可以使用DRM API的受限版本来呈现或进行计算，而无需特权操作。这种设计施加了严格的约束：必须始终有运行中的图形服务器（X服务器，Wayland合成器，...）充当DRM设备的DRM主设备，以便可以授予其他用户空间程序使用DRM设备的权限，即使在不涉及任何图形显示（如GPGPU计算）的情况下。

         “渲染节点（Render nodes）”概念试图通过将DRM用户空间API分成两个接口（一个特权和一个非特权）并为每个接口使用单独的设备文件（或“节点”）来解决这些情况。对于找到的每个GPU，其对应的DRM驱动程序（如果它支持渲染节点功能）除了主节点/dev /dri/cardX之外，还会创建一个设备文件/dev/dri/renderDX，称为渲染节点。使用直接渲染模型的客户端和想要利用GPU的计算功能的应用程序可以通过简单地打开任何现有的渲染节点并使用DRM API支持的有限子集来分派GPU操作，而无需其他特权即可做到这一点--前提是它们具有打开设备文件的文件系统权限。 显示服务器，合成器和任何其他请求模式API或任何其他特权操作的程序，都必须打开授予对完整DRM API的访问权限的标准主节点，并像往常一样使用它。渲染节点显式禁止GEM flink操作，以防止使用不安全的GEM全局名称共享缓冲区。 只能通过使用PRIME（DMA-BUF）文件描述符的方式，与包括图形服务器在内的另一个客户端共享缓冲区。

范例
``` C
#include <errno.h>
#include <fcntl.h>
#include <stdbool.h>
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/mman.h>
#include <time.h>
#include <unistd.h>
#include "xf86drm.h"
#include "xf86drmMode.h"
 
#define uint32_t unsigned int 
 
struct framebuffer{
	uint32_t size;
	uint32_t handle;	
	uint32_t fb_id;
	uint32_t *vaddr;	
};
 
static void create_fb(int fd,uint32_t width, uint32_t height, uint32_t color ,struct framebuffer *buf)
{
	struct drm_mode_create_dumb create = {};
 	struct drm_mode_map_dumb map = {};
	uint32_t i;
	uint32_t fb_id;
 
	create.width = width;
	create.height = height;
	create.bpp = 32;
	drmIoctl(fd, DRM_IOCTL_MODE_CREATE_DUMB, &create);	//创建显存,返回一个handle
 
	drmModeAddFB(fd, create.width, create.height, 24, 32, create.pitch,create.handle, &fb_id); 
	
	map.handle = create.handle;
	drmIoctl(fd, DRM_IOCTL_MODE_MAP_DUMB, &map);	//显存绑定fd，并根据handle返回offset
 
	//通过offset找到对应的显存(framebuffer)并映射到用户空间
	uint32_t *vaddr = mmap(0, create.size, PROT_READ | PROT_WRITE,MAP_SHARED, fd, map.offset);	
 
	for (i = 0; i < (create.size / 4); i++)
		vaddr[i] = color;
 
	buf->vaddr=vaddr;
	buf->handle=create.handle;
	buf->size=create.size;
	buf->fb_id=fb_id;
 
	return;
}
 
static void release_fb(int fd, struct framebuffer *buf)
{
	struct drm_mode_destroy_dumb destroy = {};
	destroy.handle = buf->handle;
 
	drmModeRmFB(fd, buf->fb_id);
	munmap(buf->vaddr, buf->size);
	drmIoctl(fd, DRM_IOCTL_MODE_DESTROY_DUMB, &destroy);
}
 
int main(int argc, char **argv)
{
	int fd;
	struct framebuffer buf[3];
	drmModeConnector *connector;
	drmModeRes *resources;
	uint32_t conn_id;
	uint32_t crtc_id;
 
 
	fd = open("/dev/dri/card0", O_RDWR | O_CLOEXEC);	//打开card0，card0一般绑定HDMI和LVDS
 
	resources = drmModeGetResources(fd);	//获取drmModeRes资源,包含fb、crtc、encoder、connector等
	
	crtc_id = resources->crtcs[0];			//获取crtc id
	conn_id = resources->connectors[0];		//获取connector id
 
	connector = drmModeGetConnector(fd, conn_id);	//根据connector_id获取connector资源
 
	printf("hdisplay:%d vdisplay:%d\n",connector->modes[0].hdisplay,connector->modes[0].vdisplay);
 
	create_fb(fd,connector->modes[0].hdisplay,connector->modes[0].vdisplay, 0xff0000, &buf[0]);	//创建显存和上色
	create_fb(fd,connector->modes[0].hdisplay,connector->modes[0].vdisplay, 0x00ff00, &buf[1]);	
	create_fb(fd,connector->modes[0].hdisplay,connector->modes[0].vdisplay, 0x0000ff, &buf[2]);	
 
	drmModeSetCrtc(fd, crtc_id, buf[0].fb_id,	
			0, 0, &conn_id, 1, &connector->modes[0]);	//初始化和设置crtc，对应显存立即刷新
	sleep(5);
 
	drmModeSetCrtc(fd, crtc_id, buf[1].fb_id,
		0, 0, &conn_id, 1, &connector->modes[0]);
	sleep(5);
 
	drmModeSetCrtc(fd, crtc_id, buf[2].fb_id,
		0, 0, &conn_id, 1, &connector->modes[0]);
	sleep(5);
 
	release_fb(fd, &buf[0]);
	release_fb(fd, &buf[1]);
	release_fb(fd, &buf[2]);
 
	drmModeFreeConnector(connector);
	drmModeFreeResources(resources);
 
	close(fd);
 
	return 0;
```

# 引用
https://blog.csdn.net/yangguoyu8023/article/details/129241987?spm=1001.2014.3001.5501

