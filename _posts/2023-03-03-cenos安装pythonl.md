
1. 确认系统版本：
  lsb_release -a 查看系统版本
  本次踩坑安装的为centos 6.3
2. 安装依赖包
```
yum install -y zlib-devel bzip2 bzip2-devel readline readline-devel readline-static sqlite sqlite-devel openssl-devel xz xz-devel ncurses-devel tk-devel gdbm-devel db4-devel libpcap-devel lapack lapack-devel blas blas-devel libffi-devel python-devel python34-devel
```
3. 安装ssl
这里特别要注意，因为python pip依赖ssl，但是不确认ssl是否正确安装或者版本是否ok
ps: 个人在这里遇到的问题比较多，主要是wget报ssl错误后，安装ssl依然报错，最好按照以下流程严格执行下ssl安装
- 换个机子下载 https://www.openssl.org/source/openssl-1.1.1q.tar.gz 传到 centos6机子上
   scp 操作即可
- 编译安装，记住这里的安装路径**usr/local/openssl**
``` shell
tar -xzf openssl-1.1.1q.tar.gz

cd openssl-1.1.1q

./config --prefix=/usr/local/openssl

./config -t

make

make install
```

- 替换老版本
``` shell
which openssl
openssl version
#查看所需要的动态链接
cd /usr/local
ldd /usr/local/openssl/bin/openssl
#
mv /usr/bin/openssl /usr/bin/openssl.old

ln -s /usr/local/openssl/bin/openssl /usr/bin

mv /usr/include/openssl /usr/include/openssl.old

ln -s /usr/local/openssl/include/openssl /usr/include

cd /usr/lib64

mv libssl.so.6 libssl.so.6.old

ln -s /usr/local/openssl/lib/libssl.so libssl.so.6

mv libcrypto.so.6 libcrypto.so.6.old

ln -s /usr/local/openssl/lib/libcrypto.so libcrypto.so.6
```
- 检查是否安装ready 
```
ldconfig -v | grep ssl
```
- 如果异常，可以添加配置
/etc/ld.so.conf.d/openssl.conf
添加
/usr/local/openssl/lib
ps：这里踩的第二个坑，没有检查是否ok，直接去安装python了，导致安装完的python缺少ssl
4. 下载python包
```
wget https://www.python.org/ftp/python/3.8.16/Python-3.8.16.tgz --no-check-certificate
```
如果依然异常，比如ssl，可以check下ssl是否正常，获取机器上安装了其它ssl
.bashrc等均可排查一下

5. 编译安装python
```bash
tar -zxvf Python-3.8.16.tgz
cd Python-3.8.16

./configure --prefix=/usr/local/python3.8 --with-openssl=/usr/local/openssl
make && make install
#更新 pip
pip install --upgrade pip

```

6. 调整下python3软链
 如果不能直接python3
```bash
ln -s /usr/local/python3.8/bin/python3.8 /usr/bin/python3 
ln -s /usr/local/python3.8/bin/pip3 /usr/bin/pip 
ln -s /usr/local/python3.8/bin/pyinstaller /usr/bin/pyinstaller
```
