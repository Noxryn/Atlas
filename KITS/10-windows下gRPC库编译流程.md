# 前言


# 准备源码

## 1.拉取 grpc 源码

使用 git 拉取 grpc, 速度较慢，请挂代理或使用 gitee 镜像
```
git clone https://github.com/grpc/grpc
```
注：可以使用 `-b` 选项指定版本

## 2.拉取 submodule

命令：
```
cd grpc
git submodule init
git submodule update
```

注1：下载速度慢可以修改 `.git/config` 文件中的 url 为 gitee 镜像路径后再次 `update`
注2：确认路径是否正确，某模块失败后请手动清除以下载的文件
注3: 更新失败可以手动克隆后注册

# 编译 gRPC

```
mkdir -p cmake/build
mkdir -p release
cd camke/build

cmake -DgRPC_INSTALL=ON \
      -DgRPC_BUILD_TESTS=OFF \
      -DCMAKE_INSTALL_PREFIX=../../release \
      ../..

make -j
make install
```
以上为 make 编译命令，windows 环境下当执行 cmake 命令后生成 `.sln`，使用 VS 打开重新生成解决方案即可

# 依赖 OpenSSL

因为不少库依赖于 OpenSSL库，但 gRPC 使用的是 BoringSSL，同时使用会导致冲突，解决方法为编译 gRPC 时依赖 OpenSSL
```
cmake -DgRPC_INSTALL=ON \
      -DCMAKE_BUILD_TYPE=Release \
      -DgRPC_BUILD_TESTS=OFF \
      -DCMAKE_INSTALL_PREFIX=../../release \
      -DgRPC_SSL_PROVIDER=package \                /* 当cmake找不到路径时使用下述参数*/
      -DOPENSSL_ROOT_DIR="" \
      -DOPENSSL_INCLUDE_DIR="" \
      -DOPENSSL_CRYPTO_LIBRARY="" \
      -DOPENSSL_SSL_LIBRARY="" \
      -DCMAKE_POLICY_VERSION_MINIMUM=3.5 \
      ../..
```