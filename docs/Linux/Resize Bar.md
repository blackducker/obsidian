pci_resize_resource()

1. intel
``` C
static void

_resize_bar(struct drm_i915_private *i915, int resno, resource_size_t size)

{

    struct pci_dev *pdev = to_pci_dev(i915->drm.dev);

    int bar_size = pci_rebar_bytes_to_size(size);

    int ret;

  

    _release_bars(pdev);

  

    ret = pci_resize_resource(pdev, resno, bar_size);

    if (ret) {

        drm_info(&i915->drm, "Failed to resize BAR%d to %dM (%pe)\n",

             resno, 1 << bar_size, ERR_PTR(ret));

        return;

    }

  

    drm_info(&i915->drm, "BAR%d resized to %dM\n", resno, 1 << bar_size);

}
```

2. amd
AMDKCL_ENABLE_RESIZE_FB_BAR