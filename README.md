# ansible-role-nginx
Nginx role for ansible

This was extracted from a more complex ansible environment I built for a client, and is slowly being migrated to something stand-alone.
It's targetting debian (9/10), and I'm adding support for CentOS/RedHat 8 at this time.

Compatibility with debian should be good, centos will take a few more patches over the coming days.

On CentOS/RedHat this role might use some non-default paths and options, as it's mostly a
direct conversion from the Debian version I built before (and I like the debian setup
better).

# Usage

The entire role can always be included, as it won't do anything unless 'manage_nginx' is enabled (default false)

A default 'dummy' configuration will be created, listening on port 80, which will basically only redirect everything to https, and have a configuration for /.well-known/acme-challenge, so we can do ACME validations.

Besides this default 'dummy' configuration, actual useful websites should be configured in the 'nginx_sites' structure

An example 'nginx_sites':

nginx_sites:
  - name: "packages.example.com"
    hostname: "packages.example.com"
    bind: "1.2.3.4"
    port: "8000"
    nossl: true
    config:
      - "# debpkgs"
      - "root /var/www/packages;"
      - "index index.html;"
      - "autoindex on;"
  - name: "monitoring.example.com"
    config:
      - "# icinga"
      - "root /usr/share/icinga/htdocs/;"
      - "index index.html;"
      - "auth_pam 'Restricted Access';"
      - "auth_pam_service_name \"nginx-allusers\";"
      - "location /icinga/ {"
      - "    alias /usr/share/icinga/htdocs/;"
      - "}" 
      - "location ~ \\.cgi$ {"
      - "    root /usr/lib/cgi-bin/icinga;"
      - "    rewrite ^/cgi-bin/icinga/(.*)$ /$1;"
      - "    include /etc/nginx/fastcgi_params;"
      - "    fastcgi_param AUTH_USER $remote_user;"
      - "    fastcgi_param REMOTE_USER $remote_user;"
      - "    fastcgi_param SCRIPT_FILENAME /usr/lib/cgi-bin/icinga$fastcgi_script_name;"
      - "    fastcgi_pass fcgiwrap;"
      - "}" 

# Options

## Base defaults

nginx_v4_default_bind
: Address to bind to using ipv4, default ''

nginx_v6_default_bind
: Address to bind to using ipv6, default '[::]'

nginx_v4_default_port
: Port-number to listen on for the default ipv4 site, defaults to '80'

nginx_v6_default_port
: Port-number to listen on for the default ipv6 site, defaults to '80'

nginx_global_extra
: Extra configuration to include into the default website configuration


## Per-site settings
nossl
: The default is to use ssl, and bind on port 443, setting nossl to false will change this back to port 80 and no-ssl.

port
: Set the portnumber to listen on, default will be either 80 or 443, depending on the value of **nossl**

bindproto
: The protocols to bind on, either 'ipv4', 'ipv6' or undefined, which means both.

bind
: The IPv4 to bind on, default is '0.0.0.0'

bind6
: The IPv6 to bind on, default is '[::]'

sslchain
: The location of the certificate-chain to be used for TLS, will default to **nginx_default_chain** unless specified

sslkey
: The location of the key to be used for TLS, will default to **nginx_default_key** unless specified

hostname (mandatory)
: The site names to listen on, space separated

accesslog
: Where to store the request logs, default will be in /var/log/nginx/**sitename**.access.log

errorlog
: Where to store the error logs, default will be in /var/log/nginx/**sitename**.error.log

config
: Additional configuration for this website, will be copied verbatim into the resulting site-configuration. Should usually include things like 'root' and 'index' entries, or 'location' and 'proxy' statements.


