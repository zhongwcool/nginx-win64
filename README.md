# 编译64位nginx for windows

>为建立中文知识库加块砖
　　　　　　　　——中科大胡不归

### 0、编译环境
#####安装 Visual Studio 2019 community
官网下载，安装是勾选C++编译功能模块。

##### 安装 msys2
[官网下载](https://www.msys2.org/)，选择64位版本。

##### 安装 Perl(ActivePerl)：
[云盘下载 ](https://pan.baidu.com/s/1cdcPlt2tK9nuGFdcTQ9luQ) (提取码：q68n)，提供的为64位版本。

### 1、源码准备
##### 下载 nginx源码
[官网](http://nginx.org/en/download.html)下载nginx源码：[点我下载](http://hg.nginx.org/nginx/archive/5e8d52bca714.zip)，本文档编写将采用版本是1.17.9。

##### 下载 zlib源码
[官网](http://www.zlib.net/)下载zlib源码：[点我下载](http://www.zlib.net/zlib-1.2.11.tar.gz)，本文档编写将基于最新稳定版本是1.2.11。

##### 下载 openssl源码
[官网](https://www.openssl.org/source/)下载openssl源码：[点我下载](https://www.openssl.org/source/old/1.0.2/openssl-1.0.2u.tar.gz)，本文档编写时采用版本是1.0.2u。

##### 下载 pcre源码
[官网](http://www.pcre.org/)下载pcre源码：[点我下载](ftp://ftp.pcre.org/pub/pcre/pcre-8.44.zip)，本文档编写时采用版本是8.44。

##### 下载 nginx-rtmp模块[可选]
```shell
git clone https://github.com/arut/nginx-rtmp-module.git
```
如果速度慢，也可以直接: [点我下载](https://codeload.github.com/arut/nginx-rtmp-module/zip/v1.2.1) 。

下载完成，将源码解压到同级目录，参考：
.
├── nginx-1.17.9
├── zlib-1.2.11
├── openssl-1.0.2u
├── pcre-8.44
└── nginx-rtmp-module-1.2.1

### 2、编译配置

##### 修改1：
编辑nginx\auto\lib\openssl\makefile.msvc文件：找到“VC-WIN32”替换为“VC-WIN64A”，“if exist ms\do_ms.bat”替换为“if exist ms\do_win64a.bat”，“ms\do_ms”替换为“ms\do_win64a”。
```text
	perl Configure VC-WIN32 no-shared				\
		--prefix="%cd%/openssl" 				\
		--openssldir="%cd%/openssl/ssl" 			\
		$(OPENSSL_OPT)

	if exist ms\do_ms.bat (						\
		ms\do_ms					\
 修改为：
	perl Configure VC-WIN64A no-shared				\
		--prefix="%cd%/openssl" 				\
		--openssldir="%cd%/openssl/ssl" 			\
		$(OPENSSL_OPT)

	if exist ms\do_win64a.bat (						\
		ms\do_win64a						\
```

##### 修改2：
由于Nignx没有提供相关配置项改变缺省banner，所以我们需要改变源码，然后重编译和重新安装一下， 具体操作：
找到/nginx/src/http/ngx_http_header_filter_module.c文件，修改以下变量的声明：
```c
static u_char ngx_http_server_string[] = "Server: nginx" CRLF;
static u_char ngx_http_server_full_string[] = "Server: " NGINX_VER CRLF;
static u_char ngx_http_server_build_string[] = "Server: " NGINX_VER_BUILD CRLF;      
 修改为：
static u_char ngx_http_server_string[] = "Server: " CRLF;
static u_char ngx_http_server_full_string[] = "Server:  " CRLF;
static u_char ngx_http_server_build_string[] = "Server: " CRLF;
```

### 3、 执行编译
1、启动 msys2 并cd到nginx源码目录下，执行：
```shell
auto/configure --with-cc=cl --builddir=objs \
--with-debug --prefix= --conf-path=conf/nginx.conf \
--pid-path=logs/nginx.pid --http-log-path=logs/access.log \
--error-log-path=logs/error.log --sbin-path=nginx.exe \
--http-client-body-temp-path=temp/client_body_temp \
--http-proxy-temp-path=temp/proxy_temp \
--http-fastcgi-temp-path=temp/fastcgi_temp \
--http-scgi-temp-path=temp/scgi_temp \
--http-uwsgi-temp-path=temp/uwsgi_temp \
--with-cc-opt=-DFD_SETSIZE=1024 \
--with-pcre=./../pcre-8.44 \
--with-zlib=./../zlib-1.2.11 \
--with-select_module \
--with-http_v2_module \
--with-http_realip_module \
--with-http_addition_module \
--with-http_sub_module \
--with-http_dav_module \
--with-http_stub_status_module \
--with-http_flv_module \
--with-http_mp4_module \
--with-http_gunzip_module \
--with-http_gzip_static_module \
--with-http_auth_request_module \
--with-http_random_index_module \
--with-http_secure_link_module \
--with-http_slice_module \
--with-mail \
--with-stream \
--with-openssl=./../openssl-1.0.2u \
--with-openssl-opt=no-asm \
--with-http_ssl_module \
--with-mail_ssl_module \
--with-stream_ssl_module \
--add-module=./../nginx-rtmp-module-1.2.1/ (此句可选)
```

2、如果选择支持nginx-rtmp-module，需要修改objs\Makefile文件：

将“-WX”删除，否则nmake时会报错“nginx error:c2220（警告被视为错误）。
```shell
CFLAGS =  -O2  -W3  -WX -nologo -MT -Zi -Fdobjs/nginx.pdb -DFD_SETSIZE=1024 -DNO_SYS_TYPES_H
 修改为：
CFLAGS =  -O2  -W3 -nologo -MT -Zi -Fdobjs/nginx.pdb -DFD_SETSIZE=1024 -DNO_SYS_TYPES_H
```

3、打开MS本地工具，使用nmake生成可执行文件
![](https://upload-images.jianshu.io/upload_images/9258027-df10be27dfa305b8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 在此工具中cd 到 nginx源码目录下，执行：
```shell
nmake -f objs/Makefile
```
运行成功，就可以见到nginx.exe生成：
![](https://upload-images.jianshu.io/upload_images/9258027-f3c9069a0fb68afd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 4、 打包发布
nginx.exe运行依赖同级目录下存在conf、html、logs、temp目录及配置文件，否则无法启动。
1、[Nginx配置包](https://pan.baidu.com/s/12K3PTZTDTi75fRAoY6ggPA) 提取码：qjrb
2、[Nginx with rtmp配置包](https://pan.baidu.com/s/1dEHrIaCmCbryz6noS6HP5w) 提取码：yxup

### 参考文章：
1、 [windows编译64位nginx](https://blog.csdn.net/u010505059/article/details/92661913)
2、 [nginx flv/rtmp/hls for Windows x64](https://www.jianshu.com/p/a429c87c1b04)
3、 [Windows编译nginx](https://www.cnblogs.com/iamyuxing/p/10883626.html)

