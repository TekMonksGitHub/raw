# Installing NGINX, ModSecurity on FreeBSD 12.0
## NGINX 1.15.9, ModSecurity v3

---

(1) `pkg install gmake wget`, needed to build ModSecurity below, and for wget below. Feel free to use `fetch` instead of `wget`.

(2) ModSecurity build from source and install, in build root directory the commands
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

(3) Download NGINX code, in build root directory the commands, pick the latest release
~~~~
wget https://nginx.org/download/nginx-1.15.9.tar.gz
tar -xvzf nginx-1.15.9.tar.gz & rm nginx-1.15.9.tar.gz
~~~~

(4) Build ModSecurity NGINX connector, in build root, and then make Nginx and install it
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

(4) Load the ModSecurity module, add the line in Main Context i.e. outside any {..} 
sections, maybe 3rd line from the top

~~~~
vi /usr/local/nginx/conf/nginx.conf
load_module modules/ngx_http_modsecurity_module.so;
~~~~

(5) Need to add the ModSecurity libs to the system library path, the following will do

~~~~
ldconfig -m /usr/local/modsecurity/lib
~~~~

(6) Add to /usr/sbin the following script as nginx, it takes care of non-default paths above
~~~~
#!/bin/sh

ldconfig -m /usr/local/modsecurity/lib
/usr/local/nginx/sbin/nginx $*
~~~~

(7) Run nginx and should see the following in `/usr/local/nginx/logs/error.log`
~~~~
[notice] 1076#0: ModSecurity-nginx v1.0.0 (rules loaded inline/local/remote: 0/0/0)
~~~~
