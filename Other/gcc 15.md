
下载源码库 https://github.com/gcc-mirror/gcc.git
利用命令加速
git clone https://github.com/gcc-mirror/gcc.git --branch releases/gcc-15 --depth 1

安装编译依赖
./contrib/download_prerequisites

./configure \ --prefix= \ --enable-languages=c,c++ \ --enable-static \ --disable-shared \ --disable-bootstrap \ --disable-multilib \ CFLAGS="-O2" \ CXXFLAGS="-O2" \ LDFLAGS="-s"
