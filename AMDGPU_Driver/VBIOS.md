# VBIOS

VBIOS 集成在IFWI(integrated firmware image)中，需要通过显卡硬件上的JTAG口使用软件进行烧写，才能更新。
[VBIOS Flashing with DediProg Tool/amdvbflash](https://confluence.amd.com/pages/viewpage.action?pageId=479188268)

整个软件架构组成分为三个部分: IFWI, FW, Driver 