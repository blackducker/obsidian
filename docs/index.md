# About
- 🔭 Currently, I'm a Senior Machine Learning Engineer at [NetEase Fuxi](https://fuxi.163.com/), focusing on Data-Centric AI.

- 🔭 Previously, I worked at [ByteDance](https://github.com/bytedance) as an Machine Learning Engineer in the AIOps field. My primary focus was on developing and implementing algorithms for time series forecasting, anomaly detection, and root cause analysis. The main objective was to proactively detect system failures and swiftly identify root causes, thereby reducing Mean Time To Repair (MTTR) through the analysis of Metrics, Logs, and Traces.

- 🔭 Prior to ByteDance, I contributed to algorithm research in network security (risk management) at [Tencent](https://github.com/Tencent). My work centered on developing anomaly detection algorithms leveraging association analysis and subspace clustering techniques, with a primary application in anti-robot systems. I take pride in the effectiveness of our algorithms, which successfully intercepted numerous attacks. I'm deeply grateful to my mentor, Mr. Liao, whose extensive experience in the security field significantly enhanced my professional growth.

- 🔭 My academic background is in statistics, having earned both my undergraduate degree from Hainan University (HNU) and my graduate degree from Sun Yat-sen University ([SYSU](https://github.com/sysu)) in this field.


# Presentation
- 2024/07: [Build Modern Python Application——MPPT Practice](https://datahonor.com/mppt/#news)
- 2023/12: [MPPT: A Modern Python Package Template](https://github.com/shenxiangzhuang/career-public/blob/master/presentation/mppt.pdf)
- 2023/08: [Data Mining Odyssey](https://github.com/shenxiangzhuang/career-public/blob/master/presentation/review/2023/career_review_2023_public.pdf)
- 2020/09: [Technical debt in machine learning systems](https://github.com/shenxiangzhuang/career-public/blob/master/presentation/mlsys/ML-Debt.pdf)

# Current Projects

- 🌱 I’m currently working on my [projects](https://datahonor.com/project/)

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

