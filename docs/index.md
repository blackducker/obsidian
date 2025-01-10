# Welcome to MkDocs

For full documentation visit [mkdocs.org](https://www.mkdocs.org).

## Commands

* `mkdocs new [dir-name]` - Create a new project.
* `mkdocs serve` - Start the live-reloading docs server.
* `mkdocs build` - Build the documentation site.
* `mkdocs -h` - Print help message and exit.

## Project layout

    mkdocs.yml    # The configuration file.
    docs/
        index.md  # The documentation homepage.
        ...       # Other markdown pages, images and other files.


# blog 发布链条
obsidian - mkdocs - cloudflare
mkdocs继承了python的极简，非常符合我的需求，只需要两个文件就可以部署在cloudflare上。

## Note Meaning

## Note Type

1. 闪念笔记 (Fleeting Note)
2. 永久笔记 (Permanent Note)
3. 项目笔记 (Project Note)


先看内存管理，再进程调度，最后iommu， nbio一个一个的模块看


``` C
	/* nbio */
    struct amdgpu_nbio      nbio;
    /* hdp */
    struct amdgpu_hdp       hdp;
    /* smuio */
    struct amdgpu_smuio     smuio;
    /* mmhub */
    struct amdgpu_mmhub     mmhub;
    /* gfxhub */
    struct amdgpu_gfxhub        gfxhub;
    /* gfx */
    struct amdgpu_gfx       gfx;
    /* sdma */
    struct amdgpu_sdma      sdma;
    /* lsdma */
    struct amdgpu_lsdma     lsdma;
    /* uvd */
    struct amdgpu_uvd       uvd;
    /* vce */
    struct amdgpu_vce       vce;
    /* vcn */
    struct amdgpu_vcn       vcn;
    /* jpeg */
    struct amdgpu_jpeg      jpeg;
    /* vpe */
    struct amdgpu_vpe       vpe;
    /* umsch */
    struct amdgpu_umsch_mm      umsch_mm;
    bool                enable_umsch_mm;
    /* firmwares */
    struct amdgpu_firmware      firmware;
    /* PSP */
    struct psp_context      psp;
    /* GDS */
    struct amdgpu_gds       gds;
    /* for userq and VM fences */
    struct amdgpu_seq64     seq64;
    /* KFD */
    struct amdgpu_kfd_dev       kfd;
    /* UMC */
    struct amdgpu_umc       umc;
```

