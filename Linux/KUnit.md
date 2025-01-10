https://zhuanlan.zhihu.com/p/661288357

要使用不同的`.kunitconfig`文件（例如提供用于测试特定子系统的文件），请将其作为选项传递：

./tools/testing/kunit/kunit.py run --kunitconfig=drivers/gpu/drm/tests/.kunitconfig