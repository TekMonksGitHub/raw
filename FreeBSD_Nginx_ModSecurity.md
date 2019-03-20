# Installing NGINX, ModSecurity on FreeBSD 12.0 in 8 Easy Steps
## NGINX 1.15.9, ModSecurity v3

---

(1) `pkg update ; pkg install gmake wget git-lite autoheader libtool autoconf automake`, needed to build ModSecurity below, and for wget below. Feel free to use `fetch` instead of `wget`.

(2) To build ModSecurity from source and install. We will call whichever directory we are downloading all this as the _build root directory_. The commands are as follows
~~~~
git clone https://github.com/SpiderLabs/ModSecurity
cd ModSecurity
git submodule init
git submodule update
./build.sh
./configure
gmake
gmake install
~~~~

(3) Download NGINX code, again in build root directory. The following commands will do this. Feel free to pick the latest NGINX release instead of 1.15.9, as you wish.
~~~~
wget https://nginx.org/download/nginx-1.15.9.tar.gz
tar -xvzf nginx-1.15.9.tar.gz ; rm nginx-1.15.9.tar.gz
~~~~

(4) Now let's build ModSecurity NGINX connector. The following commands executed in build root directory will accomplish this.
~~~~
git clone https://github.com/SpiderLabs/ModSecurity-nginx
cd nginx-1.15.9
./configure --with-compat --add-dynamic-module=../ModSecurity-nginx
~~~~~

Note the NGINX paths, I actually like the defaults, as everything is under /usr/local/nginx
* nginx path prefix: "/usr/local/nginx"
* nginx binary file: "/usr/local/nginx/sbin/nginx"
* nginx modules path: "/usr/local/nginx/modules"
* nginx configuration prefix: "/usr/local/nginx/conf"
* nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
* nginx pid file: "/usr/local/nginx/logs/nginx.pid"
* nginx error log file: "/usr/local/nginx/logs/error.log"
* nginx http access log file: "/usr/local/nginx/logs/access.log"
* nginx http client request body temporary files: "client_body_temp"
* nginx http proxy temporary files: "proxy_temp"
* nginx http fastcgi temporary files: "fastcgi_temp"
* nginx http uwsgi temporary files: "uwsgi_temp"
* nginx http scgi temporary files: "scgi_temp"

~~~~
make modules
make 
make install
~~~~

(5) Load the ModSecurity module, in the `nginx.conf` file. Add the following line in Main NGINX Context i.e. outside any {..} 
sections, maybe 3rd line from the top

~~~~
vi /usr/local/nginx/conf/nginx.conf
load_module modules/ngx_http_modsecurity_module.so;
~~~~

(6) We also need to add the ModSecurity libs to the system library path, the following will accomplish this.

~~~~
ldconfig -m /usr/local/modsecurity/lib
~~~~

(7) Add to `/usr/sbin` directory the following script. Name it as `nginx`, it takes care of non-default paths above. The permissions should be 755 for this file i.e. `chmod 755 /usr/sbin/nginx` once you have created it.
~~~~
vi /usr/sbin/nginx
~~~~
and then paste the following
~~~~
#!/bin/sh

ldconfig -m /usr/local/modsecurity/lib
/usr/local/nginx/sbin/nginx $*
~~~~
and finally 
~~~~
chmod 755 /usr/sbin/nginx
~~~~

(8) Run nginx and should see the following in `/usr/local/nginx/logs/error.log`
~~~~
[notice] 1076#0: ModSecurity-nginx v1.0.0 (rules loaded inline/local/remote: 0/0/0)
~~~~
