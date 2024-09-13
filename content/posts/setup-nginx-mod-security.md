+++
title = 'ติดตั้ง nginx mod security บน Ubuntu 22.04'
date = 2024-06-24T00:00:00+07:00
+++

ทดสอบบน Ubuntu 22.04 แบบ Clean Install นะครับ

```sh
linux test-nginx 5.15.0-116-generic #126-Ubuntu SMP Mon Jul 1 10:14:24 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
```

#### 1. ติดตั้ง Dependencies ที่สำคัญ

```sh
sudo apt update && sudo apt upgrade -y

sudo apt-get install -y bison build-essential \
ca-certificates curl dh-autoreconf doxygen \
flex gawk git iputils-ping libcurl4-gnutls-dev \
libexpat1-dev libgeoip-dev liblmdb-dev libpcre3-dev \
libpcre++-dev libssl-dev libtool libxml2 libxml2-dev \
libyajl-dev locales liblua5.3-dev pkg-config wget \
zlib1g-dev libxslt1-dev libgd-dev
```

#### 2. ติดตั้ง Nginx

```sh
sudo apt install nginx -y
```

#### 3. Compile ModSecurity

```sh
# Clone
cd /tmp && git clone https://github.com/SpiderLabs/ModSecurity
cd ModSecurity

# Update sub modules
git submodule init
git submodule update

# build
./build.sh

# user@test-nginx:/tmp/ModSecurity$ ./build.sh
# libtoolize: putting auxiliary files in '.'.
# libtoolize: copying file './ltmain.sh'
# libtoolize: putting macros in AC_CONFIG_MACRO_DIRS, 'build'.
# libtoolize: copying file 'build/libtool.m4'
# libtoolize: copying file 'build/ltoptions.m4'
# libtoolize: copying file 'build/ltsugar.m4'
# libtoolize: copying file 'build/ltversion.m4'
# libtoolize: copying file 'build/lt~obsolete.m4'
# configure.ac:50: installing './ar-lib'
# configure.ac:50: installing './compile'
# configure.ac:168: installing './config.guess'
# configure.ac:168: installing './config.sub'
# configure.ac:45: installing './install-sh'
# configure.ac:45: installing './missing'
# parallel-tests: installing './test-driver'
# examples/multiprocess_c/Makefile.am: installing './depcomp'
# configure.ac: installing './ylwrap'
```

```sh
./configure
# ...
#ModSecurity - v3.0.12-92-g3dda900e for Linux
#
# Mandatory dependencies
#   + libInjection                                  ....v3.9.2-92-gb9fcaaf
#   + Mbed TLS                                      ....v3.6.0
#   + SecLang tests                                 ....a3d4405
#
# Optional dependencies
#   + GeoIP/MaxMind                                 ....found
#      * (GeoIP) v1.6.12
#         -lGeoIP, -I/usr/include/
#   + LibCURL                                       ....found v7.81.0
#      -lcurl,  -DWITH_CURL_SSLVERSION_TLSv1_2 -DWITH_CURL
#   + YAJL                                          ....found v2.1.0
#      -lyajl, -DWITH_YAJL -I/usr/include/yajl
#   + LMDB                                          ....disabled
#   + LibXML2                                       ....found v2.9.13
#      -lxml2, -I/usr/include/libxml2 -DWITH_LIBXML2
#   + SSDEEP                                        ....not found
#   + LUA                                           ....found v503
#      -llua5.3 -L/usr/lib/x86_64-linux-gnu/, -DWITH_LUA -DWITH_LUA_5_3 -I/usr/include/lua5.3
#   + PCRE2                                          ....not found
#
# Other Options
#   + Test Utilities                                ....enabled
#   + SecDebugLog                                   ....enabled
#   + afl fuzzer                                    ....disabled
#   + library examples                              ....enabled
#   + Building parser                               ....disabled
#   + Treating pm operations as critical section    ....disabled
```

```sh
make
# ...
# make[2]: Leaving directory '/tmp/ModSecurity/test'
# make[1]: Leaving directory '/tmp/ModSecurity/test'
# make[1]: Entering directory '/tmp/ModSecurity'
# make[1]: Nothing to be done for 'all-am'.
# make[1]: Leaving directory '/tmp/ModSecurity'
```

```sh
sudo make install

# ...
# make[2]: Entering directory '/tmp/ModSecurity'
# make[2]: Nothing to be done for 'install-exec-am'.
#  /usr/bin/mkdir -p '/usr/local/modsecurity/lib/pkgconfig'
#  /usr/bin/install -c -m 644 modsecurity.pc '/usr/local/modsecurity/lib/pkgconfig'
# make[2]: Leaving directory '/tmp/ModSecurity'
# make[1]: Leaving directory '/tmp/ModSecurity
```

#### 4. install ModSecurity NGINX Connector

```sh
# cd /tmp && git clone --depth 1 https://github.com/SpiderLabs/ModSecurity-nginx.git
cd /tmp && git clone --depth 1 https://github.com/owasp-modsecurity/ModSecurity-nginx.git
```

ตรวจสอบ version ของ nginx ที่ติดตั้งอยู่

```sh
nginx -v
# nginx version: nginx/1.18.0 (Ubuntu)

# download version ให้ตรงกับที่ใช้อยู่นะครับ
wget http://nginx.org/download/nginx-1.18.0.tar.gz

# extract
tar -xzvf nginx-1.18.0.tar.gz
```

ติดตั้ง ngx_http_geoip2_module

```sh
cd /tmp && wget https://github.com/maxmind/libmaxminddb/releases/download/1.10.0/libmaxminddb-1.10.0.tar.gz

# Extract
tar -xzvf libmaxminddb-1.10.0.tar.gz
# ...
# libmaxminddb-1.10.0/t/bad_pointers_t.c
# libmaxminddb-1.10.0/t/basic_lookup_t.c
# libmaxminddb-1.10.0/t/mmdblookup_t.pl
# libmaxminddb-1.10.0/t/get_value_pointer_bug_t.c
# libmaxminddb-1.10.0/t/maxminddb_test_helper.c
# libmaxminddb-1.10.0/t/threads_t.c
# libmaxminddb-1.10.0/ltmain.sh
cd libmaxminddb-1.10.0/

./configure
# ...
# config.status: creating config.h
# config.status: creating include/maxminddb_config.h
# config.status: executing libtool commands
# config.status: executing depfiles commands

make && make check
# ...
# ===================
# All 20 tests passed
# ===================
# make[2]: Leaving directory '/tmp/libmaxminddb-1.10.0/t'
# make[1]: Leaving directory '/tmp/libmaxminddb-1.10.0/t'
# make[1]: Entering directory '/tmp/libmaxminddb-1.10.0'
# if [ ! -f man/man1/mmdblookup.1 ]; then mkdir -p man/man1 && touch man/man1/mmdblookup.1; fi
# if [ ! -f man/man3/libmaxminddb.3 ]; then mkdir -p man/man3 && touch man/man3/libmaxminddb.3; fi
# if [ ! -f man/man3/libmaxminddb.3 ]; then mkdir -p man/man3 && touch man/man3/libmaxminddb.3; fi
# if [ ! -f man/man3/libmaxminddb.3 ]; then mkdir -p man/man3 && touch man/man3/libmaxminddb.3; fi
# if [ ! -f man/man3/libmaxminddb.3 ]; then mkdir -p man/man3 && touch man/man3/libmaxminddb.3; fi
# if [ ! -f man/man3/libmaxminddb.3 ]; then mkdir -p man/man3 && touch man/man3/libmaxminddb.3; fi
# if [ ! -f man/man3/libmaxminddb.3 ]; then mkdir -p man/man3 && touch man/man3/libmaxminddb.3; fi
# if [ ! -f man/man3/libmaxminddb.3 ]; then mkdir -p man/man3 && touch man/man3/libmaxminddb.3; fi
# if [ ! -f man/man3/libmaxminddb.3 ]; then mkdir -p man/man3 && touch man/man3/libmaxminddb.3; fi
# if [ ! -f man/man3/libmaxminddb.3 ]; then mkdir -p man/man3 && touch man/man3/libmaxminddb.3; fi
# if [ ! -f man/man3/libmaxminddb.3 ]; then mkdir -p man/man3 && touch man/man3/libmaxminddb.3; fi
# if [ ! -f man/man3/libmaxminddb.3 ]; then mkdir -p man/man3 && touch man/man3/libmaxminddb.3; fi
# if [ ! -f man/man3/libmaxminddb.3 ]; then mkdir -p man/man3 && touch man/man3/libmaxminddb.3; fi
# if [ ! -f man/man3/libmaxminddb.3 ]; then mkdir -p man/man3 && touch man/man3/libmaxminddb.3; fi
# if [ ! -f man/man3/libmaxminddb.3 ]; then mkdir -p man/man3 && touch man/man3/libmaxminddb.3; fi
# if [ ! -f man/man3/libmaxminddb.3 ]; then mkdir -p man/man3 && touch man/man3/libmaxminddb.3; fi
# make[1]: Leaving directory '/tmp/libmaxminddb-1.10.0'

sudo make install
sudo ldconfig
```

ตรวจสอบ arguments และ dependencies ที่ใช้สำหรับการ build

```sh
nginx -V

# --with-cc-opt='-g -O2 -ffile-prefix-map=/build/nginx-zctdR4/nginx-1.18.0=. -flto=auto -ffat-lto-objects -flto=auto -ffat-lto-objects -fstack-protector-strong -Wformat -Werror=format-security -fPIC -Wdate-time -D_FORTIFY_SOURCE=2' --with-ld-opt='-Wl,-Bsymbolic-functions -flto=auto -ffat-lto-objects -flto=auto -Wl,-z,relro -Wl,-z,now -fPIC' --prefix=/usr/share/nginx --conf-path=/etc/nginx/nginx.conf --http-log-path=/var/log/nginx/access.log --error-log-path=/var/log/nginx/error.log --lock-path=/var/lock/nginx.lock --pid-path=/run/nginx.pid --modules-path=/usr/lib/nginx/modules --http-client-body-temp-path=/var/lib/nginx/body --http-fastcgi-temp-path=/var/lib/nginx/fastcgi --http-proxy-temp-path=/var/lib/nginx/proxy --http-scgi-temp-path=/var/lib/nginx/scgi --http-uwsgi-temp-path=/var/lib/nginx/uwsgi --with-compat --with-debug --with-pcre-jit --with-http_ssl_module --with-http_stub_status_module --with-http_realip_module --with-http_auth_request_module --with-http_v2_module --with-http_dav_module --with-http_slice_module --with-threads --add-dynamic-module=/build/nginx-zctdR4/nginx-1.18.0/debian/modules/http-geoip2 --with-http_addition_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_sub_module

cd /tmp && git clone https://github.com/leev/ngx_http_geoip2_module.git

cd /tmp/nginx-1.18.0/

# sudo ./configure --add-dynamic-module=../ModSecurity-nginx <args>
# Note: Changed --add-dynamic-module=/build/nginx-zctdR4/nginx-1.18.0/debian/modules/http-geoip2 to --add-dynamic-module=/tmp/ngx_http_geoip2_module
sudo ./configure --add-dynamic-module=../ModSecurity-nginx --with-cc-opt='-g -O2 -ffile-prefix-map=/build/nginx-zctdR4/nginx-1.18.0=. -flto=auto -ffat-lto-objects -flto=auto -ffat-lto-objects -fstack-protector-strong -Wformat -Werror=format-security -fPIC -Wdate-time -D_FORTIFY_SOURCE=2' --with-ld-opt='-Wl,-Bsymbolic-functions -flto=auto -ffat-lto-objects -flto=auto -Wl,-z,relro -Wl,-z,now -fPIC' --prefix=/usr/share/nginx --conf-path=/etc/nginx/nginx.conf --http-log-path=/var/log/nginx/access.log --error-log-path=/var/log/nginx/error.log --lock-path=/var/lock/nginx.lock --pid-path=/run/nginx.pid --modules-path=/usr/lib/nginx/modules --http-client-body-temp-path=/var/lib/nginx/body --http-fastcgi-temp-path=/var/lib/nginx/fastcgi --http-proxy-temp-path=/var/lib/nginx/proxy --http-scgi-temp-path=/var/lib/nginx/scgi --http-uwsgi-temp-path=/var/lib/nginx/uwsgi --with-compat --with-debug --with-pcre-jit --with-http_ssl_module --with-http_stub_status_module --with-http_realip_module --with-http_auth_request_module --with-http_v2_module --with-http_dav_module --with-http_slice_module --with-threads --add-dynamic-module=/tmp/ngx_http_geoip2_module --with-http_addition_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_sub_module

# ...
# Configuration summary
#   + using threads
#   + using system PCRE library
#   + using system OpenSSL library
#   + using system zlib library

#   nginx path prefix: "/usr/share/nginx"
#   nginx binary file: "/usr/share/nginx/sbin/nginx"
#   nginx modules path: "/usr/lib/nginx/modules"
#   nginx configuration prefix: "/etc/nginx"
#   nginx configuration file: "/etc/nginx/nginx.conf"
#   nginx pid file: "/run/nginx.pid"
#   nginx error log file: "/var/log/nginx/error.log"
#   nginx http access log file: "/var/log/nginx/access.log"
#   nginx http client request body temporary files: "/var/lib/nginx/body"
#   nginx http proxy temporary files: "/var/lib/nginx/proxy"
#   nginx http fastcgi temporary files: "/var/lib/nginx/fastcgi"
#   nginx http uwsgi temporary files: "/var/lib/nginx/uwsgi"
#   nginx http scgi temporary files: "/var/lib/nginx/scgi"

sudo make modules
# ...
# objs/ngx_http_geoip2_module_modules.o \
# -Wl,-Bsymbolic-functions -flto=auto -ffat-lto-objects -flto=auto -Wl,-z,relro -Wl,-z,now -fPIC -lmaxminddb \
# -shared
# make[1]: Leaving directory '/tmp/nginx-1.18.0

# Check
ls -la objs/
# 264176 Jul 24 13:51 ngx_http_modsecurity_module.so

# create modsec
sudo mkdir -p /etc/nginx/modules
sudo mkdir -p /etc/nginx/modsec
sudo cp /tmp/nginx-1.18.0/objs/ngx_http_modsecurity_module.so /etc/nginx/modules/ngx_http_modsecurity_module.so
sudo cp /tmp/ModSecurity/unicode.mapping /etc/nginx/modsec/
```

#### 5. แก้ไขการตั้งค่า

```sh
sudo nano /etc/nginx/nginx.conf

# user www-data;
# worker_processes auto;
# pid /run/nginx.pid;
# include /etc/nginx/modules-enabled/*.conf;
# load_module /etc/nginx/modules/ngx_http_modsecurity_module.so; <-- Add this line
```

```sh
cd /tmp && git clone https://github.com/coreruleset/coreruleset

cd coreruleset

# change file
mv crs-setup.conf.example crs-setup.conf

# change file
mv rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf.example rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf

# copy file
cd /tmp && sudo cp -r coreruleset /usr/local/modsecurity-crs

# copy
cd /tmp && sudo cp ModSecurity/modsecurity.conf-recommended /etc/nginx/modsec/modsecurity.conf

# edit config
sudo nano /etc/nginx/modsec/modsecurity.conf

# SecRuleEngine DetectionOnly -> SecRuleEngine On

# Create New File
sudo nano /etc/nginx/modsec/main.conf
# Add this content
# Include /etc/nginx/modsec/modsecurity.conf
# Include /usr/local/modsecurity-crs/crs-setup.conf
# Include /usr/local/modsecurity-crs/rules/*.conf
```

#### 6. นำ config ไปใช้งาน

```sh
sudo nano /etc/nginx/sites-available/default

# server {
#         listen 80 default_server;
#         listen [::]:80 default_server;

#         modsecurity on; <-- Add
#         modsecurity_rules_file /etc/nginx/modsec/main.conf; <-- Add

sudo nginx -t

sudo systemctl restart nginx
```

#### 7. Test

```sh
http://ip/?file=/etc/passwd

ถ้าเจอหน้า 403 แสดงว่าใช้งานได้
```

#### 8. Bonus หากต้องการ Allow บาง IP

```sh

sudo nano /etc/nginx/modsec/main.conf

# เพิ่ม config โดยแก้ไข 1.2.3.4, 5.6.7.8 เป็น IP ที่ต้องการ allow WAF
SecRule REMOTE_ADDR "@ipMatch 1.2.3.4, 5.6.7.8" "id:10000,phase:1,nolog,allow,ctl:ruleEngine=Off"
# sudo nginx -t && sudo systemctl restart nginx
```

#### Ref

1. [https://github.com/maxmind/libmaxminddb](https://github.com/maxmind/libmaxminddb)
2. [https://github.com/coreruleset/coreruleset](https://github.com/coreruleset/coreruleset)
3. [https://github.com/leev/ngx_http_geoip2_module](https://github.com/leev/ngx_http_geoip2_module)
4. [https://medium.com/codelogicx/securing-nginx-server-using-modsecurity-oswaf-7ba79906d84c](https://medium.com/codelogicx/securing-nginx-server-using-modsecurity-oswaf-7ba79906d84c)
