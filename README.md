# ansible-role-nginx
Nginx role for ansible

This was extracted from a more complex ansible environment I built for a client, and is slowly being migrated to something stand-alone.
It's targetting debian (9/10), and I'm adding support for CentOS/RedHat 8 at this time.

Compatibility with debian should be good, centos will take a few more patches over the coming days.

On CentOS/RedHat this role might use some non-default paths and options, as it's mostly a
direct conversion from the Debian version I built before (and I like the debian setup
better).

# Usage

Make sure to define 'manage_nginx: true' and configure your websites with 'nginx_sites'.
An example 'nginx_sites':

nginx_sites:
  - name: "packages.example.com"
    hostname: "packages.example.com"
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

