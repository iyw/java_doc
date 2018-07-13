# fastDFS搭建图片服务器(完整流程)

## 安装fastdfs

#### 1) 先安装依赖
安装依赖 yum install libevent

安装基础库 
```sh
git clone https://github.com/happyfish100/libfastcommon.git
cd libfastcommon
./make.sh
./make install
```

#### 2) 下载fastdfs
```sh
git clone https://github.com/happyfish100/fastdfs.git 
cd fastdfs

./make.sh
./make.sh install
```

安装成功后 将conf下的文件copy 到/etc/fdfs

```sh
    conf/
    ├── anti-steal.jpg
    ├── client.conf
    ├── http.conf
    ├── mime.types
    ├── storage.conf
    ├── storage_ids.conf
    └── tracker.conf
```

#### 3)配置
```sh
修改tracker.conf
base_path=/home/fastDFS (目录可以自行定义)

修改storage.conf
store_path0=/home/fastDFS/fdfs_storage
...可以定义多个目录 只需依次配置store_path1 store_path2 ...

tracker_server=118.190.19.45:22122
```

### 4)启动

```sh
启动tracker
sudo /usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf start

启动storage
 sudo /usr/bin/fdfs_storaged /etc/fdfs/storage.conf start
```

### 5)测试

```sh
/usr/bin/fdfs_test /etc/fdfs/client.conf  upload /www/test/image/suo/shasha.jpeg 

提示如下说明安装成功

[2018-03-16 14:08:08] DEBUG - base_path=/home/fastDFS, connect_timeout=30, network_timeout=60, tracker_server_count=1, anti_steal_token=0, anti_steal_secret_key length=0, use_connection_pool=0, g_connection_pool_max_idle_time=3600s, use_storage_id=0, storage server id count: 0

tracker_query_storage_store_list_without_group: 
    server 1. group_name=, ip_addr=118.190.19.45, port=23000

    group_name=group1, ip_addr=118.190.19.45, port=23000
    storage_upload_by_filename
    group_name=group1, remote_filename=M00/00/00/dr4TLVqrX0iATB-XAAJIl05hkF071.jpeg
    source ip address: 118.190.19.45
    file timestamp=2018-03-16 14:08:08
    file size=149655
    file crc32=1315016797
    example file url: http://118.190.19.45/group1/M00/00/00/dr4TLVqrX0iATB-XAAJIl05hkF071.jpeg
    storage_upload_slave_by_filename
    group_name=group1, remote_filename=M00/00/00/dr4TLVqrX0iATB-XAAJIl05hkF071_big.jpeg
    source ip address: 118.190.19.45
    file timestamp=2018-03-16 14:08:08
    file size=149655
    file crc32=1315016797
    example file url: http://118.190.19.45/group1/M00/00/00/dr4TLVqrX0iATB-XAAJIl05hkF071_big.jpeg
     ~/build/fastdfs packet_write_wait: Connection to 118.190.19.45 port 22: Broken pipe
```

## 二、安装fastdfs-nginx-module

```sh
 git clone https://github.com/happyfish100/fastdfs-nginx-module.git /usr/local/fastdfs-nginx-module

 并将src下的 mod_FastDFS.conf 拷贝至/etc/fdfs下
 并修改mod_FastDFS.conf
 vim /etc/fdfs/mod_fastdfs.conf

 base_path=/home/fastDFS
 tracker_server=118.190.19.45:22122
 store_path0=/home/fastDFS/fdfs_storage
 url_have_group_name = true


下载nginx 并安装 

 ./configure  --prefix=/usr/local/nginx --with-http_gzip_static_module --http-client-body-temp-path=/var/temp/nginx/client --http-proxy-temp-path=/var/temp/nginx/proxy --http-fastcgi-temp-path=/var/temp/nginx/fastcgi --http-uwsgi-temp-path=/var/temp/nginx/uwsgi --http-scgi-temp-path=/var/temp/nginx/scgi --add-module=/usr/local/fastdfs-nginx-module/src/
 make && make install 

```

#### 添加nginx配置

```sh
server {
    listen 8080; 
    server_name www.httpds.com;
    access_log /home/wwwlogs/www.httpds.com.access.log;
    error_log /home/wwwlogs/www.httpds.com.error.log info;
    index index.html index.js;
    root /home/fastDFS/fdfs_storage/data;
    #root /www/test/imag;
    location / { 
        try_files $uri $uri/ /index.html;
    }   

    location /group1/M00/ {
         ngx_fastdfs_module;
    }   

}
```
成功配置 上传图片访问地址 为  http://www.httpds.com:8080/group1/M00/00/00/dr4TLVqrX0iATB-XAAJIl05hkF071.jpeg


