#修改wsl内torch引用目录

pip show torch

#显示了torch引用目录

-=>  Location: /home/longlyao/.local/lib/python3.10/site-packages

cd /home/longlyao/.local/lib/python3.10/site-packages/torch/lib

#设置binary指向路径

patchelf --set-rpath /opt/rocm/lib libamdhip64.so

#或者可以将我们已经编译过的直接替换过来，修改名字也行 mv xxx/

#查看是否已经修改

readelf -d libamdhip64.so

>> 0x000000000000001d (RUNPATH)            Library runpath: [/opt/rocm/lib]

#验证

python

>>import torch

>>torch.cuda.is_available()