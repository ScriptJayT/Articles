## Install Xampp

### Launch 

If the program does not appear under programs on you desktop:
In c:/xampp; find the xampp-control.exe file and execute it to open the manager.

### Edit root

In xampp cpanel: apache config - Apache httpd.config
Find 
```
DocumentRoot "/xampp/htdocs"
<Directory "/xampp/htdocs">
```
and change the path to your liking; start with /; but don't close it

## Install Mamp

## Rerouting yourself

Add .htaccess to the root
```
RewriteEngine on
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule . index.php [L]
```