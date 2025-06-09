

在用户模式 下配置linux内核，清理源码树，以进一步执行kunit test
make ARCH=um mrproper
执行python脚本，并指定drm和ttm对应的kunitconfig启动测试
./tools/testing/kunit/kunit.py run --kunitconfig=drivers/gpu/drm/tests/.kunitconfig
./tools/testing/kunit/kunit.py run --kunitconfig=drivers/gpu/drm/ttm/tests/.kunitconfig