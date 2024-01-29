## Enable snow @ HomePage

```
https://ams-gemini01.dedicateware.com:2083/cpsess8954897532/frontend/jupiter/filemanager/index.html
 ```

public_html -> app -> design -> frontend -> local -> default -> layout -> local.xml

## Set the “ea-php74” package as the default “PHP” programming language.
``` php 
# php -- BEGIN cPanel-generated handler, do not edit
# Set the “ea-php74” package as the default “PHP” programming language.
<IfModule mime_module>
  AddHandler application/x-httpd-ea-php74 .php .php7 .phtml
</IfModule>
# php -- END cPanel-generated handler, do not edit

```
