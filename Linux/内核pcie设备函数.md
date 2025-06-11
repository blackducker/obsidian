
pci_resource_start是一个在linux内核中获取PCI设备资源起始地址的函数。这个函数属于pci_dev结构体的resource数组的成员，该数组保存了PCI设备的各种资源的信息。
在bios启动时，会读取pci配置空间从而将设备内存映射到系统物理空间中的 memory space 或者 io space，
``` C
resource_size_t pci_resource_start(struct pci_dev *dev, int bar);
```
其中，dev是指向PCI设备结构体的指针， bar是BAR的编号(通常为0到5).
值得注意的是，`pci_resource_start` 返回的是PCI设备在物理内存或I/O 端口空间中的地址，而不是CPU可以直接访问的虚拟地址。如果驱动程序需要访问，还需要使用 `ioremap` 函数.
此外，`pci_resource_start` 函数返回的地址是PCI总线域的地址，而不是在CPU的存储器域地址。因此在将这个地址提供给用户空间进行访问的时候，还需要进行地址空间的转换。`通常使用pci_resource_to_user` 来完成，

``` C
int pci_resource_to_user(struct pci_dev *dev, int bar, struct resource *rsrc, resource_size_t *user_base, resource_size_t *user_length);
```
使用`pci_resource_to_user`函数时，需要注意以下几点：
1. `pci_resource_to_user`函数用于将PCI设备的资源映射到用户空间，以便用户空间程序可以直接访问设备的寄存器或缓冲区。
2. 在调用`pci_resource_to_user`函数之前，需要先调用`pci_request_regions`函数来请求PCI设备的资源。
3. `bar`参数指定要映射的基址寄存器（BAR）索引。BAR是PCI设备中用于指定设备在系统内存中的地址范围的寄存器。
4. `rsrc`参数用于存储映射后的资源信息，包括物理地址、长度等。
5. `user_base`和`user_length`参数用于存储映射后的用户空间基址和长度。
6. 在使用完映射的资源后，应该调用`pci_release_regions`函数释放PCI设备的资源。