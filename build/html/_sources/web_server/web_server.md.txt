# web server

1. **board install and start httpd**

   ```
   dnf install httpd
   systemctl start httpd
   firewall-cmd --add-service=http
   ```

   

2. **PC use browser to visit**

   Input board IP, if you can read *Fedora Webserver Test Page*, it means that the web server installed at this site is working properly.

   ![testpage](https://github.com/jianlonghuang/docs/blob/master/source/web_server/pic/testpage.png)

3. **disable Test Page**

   To disable the test page, comment out all the lines in the file `/etc/httpd/conf.d/welcome.conf` using `#` as follows:

   ```
   # <LocationMatch "^/+$">
   #    Options -Indexes
   #    ErrorDocument 403 /.noindex.html
   # </LocationMatch>
   
   # <Directory /usr/share/httpd/noindex>
   #    AllowOverride None
   #    Require all granted
   # </Directory>
   
   # Alias /.noindex.html /usr/share/httpd/noindex/index.html
   ```

4. **copy your files to /var/www/html/**

   Input board IP, if you can read your files

​	![testhtml](https://github.com/jianlonghuang/docs/blob/master/source/web_server/pic/testhtml.png)



