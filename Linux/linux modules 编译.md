
只编译单个模块
make modules_prepare  
这将准备模块编译环境并生成`Module.symvers`文件
make -j32 M=drivers/gpu/drm/amd/amdgpu modules

保留汇编文件
make -j32 M=drivers/gpu/drm/amd/amdgpu KCFLAGS="-S" modules

git log -L '/^int drm_prime_fd_to_handle_ioctl(/',/^}/:drivers/gpu/drm/drm_prime.c