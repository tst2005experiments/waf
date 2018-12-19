(google translated)

# waf
- Customize WAF (Web application firewall) with Nginx+Lua
- Watched Lua for two days, practiced hands, refer to https://github.com/loveshell/ngx_lua_waf

### Requires generation
    Because some of Nginx's security protection features are limited, study whether you can write a WAF yourself, refer to (copy) Kindle's ngx_lua_waf, try to write one yourself, use two days, learn Lua, and write. But not a security professional, only a few simple features have been implemented:

#### function list:
1. Support IP whitelist and blacklist functions, and directly reject blacklisted IP access.
2. Support URL whitelisting, which defines URLs that do not need to be filtered.
3. Support User-Agent filtering, match the entries in the custom rules, and then process them (return 403).
4. Supports CC attack protection. The number of accesses of a single URL at a specified time exceeds the set value and returns directly to 403.
5. Support cookie filtering, match the entries in the custom rules, and then process them (return 403).
6. Support URL filtering to match entries in custom rules. If the URL requested by the user contains these, return 403.
7. Support URL parameter filtering, the principle is the same as above.
8. Support logging, log all rejected operations to the log.
9. Logging is in JSON format for easy log analysis, such as using ELKStack for attack log collection, storage, search and display.

#### WAF implementation
   WAF describes the HTTP request (protocol parsing module), rule detection (rule module), performs different defense actions (action module), and records the defense process (log module). Therefore, the implementation of WAF in this paper consists of five modules (configuration module, protocol parsing module, rule module, action module, error handling module).

#### Nginx + Lua Deployment

Environmental preparation
    [root@nginx-lua ~]# cd /usr/local/src
First, Nginx now installs the necessary Nginx and PCRE packages.
<pre>
[root@nginx-lua src]# wget 'http://nginx.org/download/nginx-1.12.1.tar.gz'
[root@nginx-lua src]# wget https://nchc.dl.sourceforge.net/project/pcre/pcre/8.41/pcre-8.41.tar.gz
</pre>
Second, download the latest luajit and ngx_devel_kit (NDK), and lua-nginx-module written by Chun Ge (chapter)
<pre>
  [root@nginx-lua src]# wget http://luajit.org/download/LuaJIT-2.0.5.tar.gz
  [root@nginx-lua src]# wget https://github.com/simpl/ngx_devel_kit/archive/v0.3.0.tar.gz
  [root@nginx-lua src]# wget wget https://github.com/chaoslawful/lua-nginx-module/archive/v0.10.10.zip
</pre>

Finally, create a regular user running Nginx
   [root@nginx-lua src]# useradd -s /sbin/nologin -M www

Unzip NDK and lua-nginx-module
<pre>
    [root@openstack-compute-node5 src]# tar zxvf v0.3.0.tar.gz After decompression is ngx_devel_kit-0.3.0
    [root@openstack-compute-node5 src]# unzip -q v0.10.10.zip decompressed as lua-nginx-module-0.10.10
</pre>

Install LuaJIT
Luajit is a Lua just-in-time compiler.
<pre>
[root@webs-ebt src]# tar zxvf LuaJIT-2.0.5.tar.gz 
[root@webs-ebt src]# cd LuaJIT-2.0.5
[root@webs-ebt LuaJIT-2.0.5]# make && make install
</pre>

Install Nginx and load the module
<pre>
[root@webs-ebt src]# tar zxf nginx-1.12.1.tar.gz
[root@webs-ebt src]# tar zxvf pcre-8.41.tar.gz 
[root@webs-ebt src]# cd nginx-1.12.1
[root@webs-ebt nginx-1.12.1]# export LUAJIT_LIB=/usr/local/lib
[root@webs-ebt nginx-1.12.1]# export LUAJIT_INC=/usr/local/include/luajit-2.0
[root@webs-ebt nginx-1.12.1]#./configure --user=www --group=www --prefix=/usr/local/nginx-1.12.1/ --with-pcre=/usr/local/src/pcre-8.41 --with-http_stub_status_module --with-http_sub_module --with-http_gzip_static_module --without-mail_pop3_module --without-mail_imap_module --without-mail_smtp_module  --add-module=../ngx_devel_kit-0.3.0/ --add-module=../lua-nginx-module-0.10.10/
[root@webs-ebt nginx-1.12.1]# make -j2 && make install
[root@webs-ebt nginx-1.12.1]# ln -s /usr/local/nginx-1.12.1 /usr/local/nginx
[root@webs-ebt nginx-1.12.1]# ln -s /usr/local/lib/libluajit-5.1.so.2 /lib64/libluajit-5.1.so.2
</pre>
If you do not create a symbolic link, the following exception may occur:
error while loading shared libraries: libluajit-5.1.so.2: cannot open shared object file: No such file or directory

##### Test Installation
After the installation is complete, the following can be tested and installed, modify nginx.conf to add the first configuration.
<pre>
         Location /hello {
                 Default_type 'text/plain';
                 Content_by_lua 'ngx.say("hello,lua")';
         }
    
[root@webs-ebt src]# /usr/local/nginx/sbin/nginx -t
[root@webs-ebt src]# /usr/local/nginx/sbin/nginx -t
</pre>

Then visit http://xxx.xxx.xxx.xxx/hello if hello, lua appears. Indicates that the installation is complete and then you can.

Note: You can also deploy Chunge's open source project directly: https://github.com/openresty

#### OpenResty Deployment
<pre>
Installation dependency package
# yum install -y readline-devel pcre-devel openssl-devel
# cd /usr/local/src
Download and compile and install openresty
# wget "https://openresty.org/download/openresty-1.11.2.5.tar.gz"
# tar zxf openresty-1.11.2.5.tar.gz
# cd openresty-1.11.2.5
# ./configure --prefix=/usr/local/openresty-1.11.2.5 \
--with-luajit --with-http_stub_status_module \
--with-pcre=/usr/local/src/pcre-8.41 --with-pcre-jit
# gmake && gmake install
# ln -s /usr/local/openresty-1.11.2.5 /usr/local/openresty

Test openresty installation
# vim /usr/local/openresty/nginx/conf/nginx.conf
server {
    location /hello {
            default_type text/html;
            content_by_lua_block {
                ngx.say("HelloWorld")
            }
        }
}
[root@webs-ebt src]# /usr/local/openresty-1.11.2.5/nginx/sbin/nginx -t
nginx: the configuration file /usr/local/openresty-1.11.2.5/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/openresty-1.11.2.5/nginx/conf/nginx.conf test is successful
# /usr/local/openresty/nginx/sbin/nginx
Hello World
# curl http://192.168.199.33/hello
HelloWorld
</pre>

#### WAFDeployment

<pre>
#git clone https://github.com/unixhot/waf.git
#cp -a ./waf/waf /usr/local/openresty/nginx/conf/

Modify the Nginx configuration file and add the following configuration. Pay attention to the path, and the WAF log is stored by default in /tmp/date_waf.log
#WAF
     Lua_shared_dict limit 50m;
     Lua_package_path "/usr/local/openresty/nginx/conf/waf/?.lua";
     Init_by_lua_file "/usr/local/openresty/nginx/conf/waf/init.lua";
     Access_by_lua_file "/usr/local/openresty/nginx/conf/waf/access.lua";

[root@openstack-compute-node5 ~]# /usr/local/openresty/nginx/sbin/nginx –t
[root@openstack-compute-node5 ~]# /usr/local/openresty/nginx/sbin/nginx
</pre>

