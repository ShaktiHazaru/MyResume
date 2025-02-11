# MyResume #
This resume is built only using html and css

I have deployed this html on both HTTPD and NGINX.
login to the VM and become root. 

1. First check if nginx is installed already
    rpm -qa | grep nginx
   if there are no results then go to step 2

2. To download nginx on centos

   Run dnf install nginx

 3. start the nginx service
     systemctl start nginx
    
5.  enable the service - > this would start the nginx service whenever the service is started or rebooted 
    systemctl enable nginx

6. Instead of editing the original conf file /etc/nginx/nginx.conf, create your own conf file under ex: /etc/nginx/conf.d/sdas.conf
   This would avoid issues for any misconfigurations and changes would apply only to your file.
   your file would be picked automatically as in the original config file below is already included. 

   # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;
   
server {
  listen 80;
  server_name 192.168.0.6;

  root /var/www/html;
  index index.html;

  location / {
    try_files $uri $uri/ =404;
    add_header Cache-Control no-cache;

 }
}

   6. give proper permissions to the ID nginx for the /var/www/html and index.html run below
       chmod -R nginx:nginx /var/www/html
       chmod 644 /var/www/hello/html/index.html
      
   7. disable for the firewalld for enabling access, this is only in test.
      systemctl stop firewalld
      systemctl disable firewalld

8. Validate Nginx Configuration
    Run the following command to ensure there are no syntax errors in your Nginx configuration:
     nginx -t
   it should return something like below if everything is correct.
     nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
     nginx: configuration file /etc/nginx/nginx.conf test is successful
10. For debuggin check the logs
    tail -f /var/log/nginx/error.log
   
11. To check if the page is loading you can do curl localhost
     curl -i 127.0.0.1 or curl hostname

12. Run SEstaus, if SELinux is enforcing, ensure that /var/www/html/ has the correct SELinux context:

     sudo chcon -R -t httpd_sys_content_t /var/www/hello/html/
    
14. Finally launch the html page on your fav browser using the IP of your VM. 

Issues faced during and how I resolved. 
tail -f /var/log/nginx/error.log

rewrite or internal redirection cycle while internally redirecting to "//=404/=404/=404/=404/=404/=404/=404/=404/=404/=404/=404", client: 127.0.0.1, server: 192.168.0.6, request: "GET / HTTP/1.1", host: "127.0.0.1"


the error message indicates that there is an internal redirection cycle caused by the try_files directive in your Nginx configuration. Specifically, the try_files directive is trying to resolve the URI /=404, but it keeps redirecting to itself, creating an infinite loop.

The issue arises because $uri/=404 is treated as a file or directory path relative to your root directive. Since no such file or directory exists, Nginx keeps redirecting, causing the infinite loop.

Solution
Update the try_files directive in /etc/nginx/conf.d/sdas.conf to correctly handle the 404 error:

location / {
    try_files $uri $uri/ =404;
}



Here:

$uri checks if the exact file exists.
$uri/ checks if the directory exists.
=404 returns a 404 Not Found response if neither exists, avoiding the loop.      
