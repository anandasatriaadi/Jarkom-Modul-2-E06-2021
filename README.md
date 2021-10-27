# Jarkom-Modul-2-E06-2021

## Nomor 13
###	Luffy juga meminta Nami untuk dibuatkan konfigurasi virtual host. Virtual host ini bertujuan untuk dapat mengakses file asset www.super.franky.yyy.com/public/js menjadi www.super.franky.yyy.com/js. 

### _Solusi_
  * Pada Skypie, edit `apache2/sites-available/super.franky.e06.com.conf` tambahkan
    <pre>
    <code>
    Alias "/js" "/var/www/super.franky.e06.com/public/js"
    </code>
    </pre>
  * Test dengan `lynx www.super.franky.e06.com/js`

    ![image](https://user-images.githubusercontent.com/43901559/139071545-4a74da6d-7665-46fc-b438-244ea8c5ae4f.png)

## Nomor 14
### Dan Luffy meminta untuk web www.general.mecha.franky.yyy.com hanya bisa diakses dengan port 15000 dan port 15500

### _Solusi_
  * Pada Skypie `apache2/sites-available`, buat `general.mecha.franky.e06-15000.com.conf` untuk port 15000 dengan menggunakan `000-default.conf` sebagai template.
  * Edit file yang baru dibuat
    ```
    <VirtualHost *:15000> 
      ServerName		general.mecha.franky.e06.com 
      ServerAlias		www.general.mecha.franky.e06.com 

      ServerAdmin		webmaster@localhost 
      DocumentRoot	/var/www/general.mecha.franky.e06.com 

      ErrorLog 		${APACHE_LOG_DIR}/error.log 
      CustomLog 		${APACHE_LOG_DIR}/access.log combined 
    </VirtualHost>
    ```
  * Buat file `general.mecha.franky.e06-15500.com.conf` untuk port 15500
  * Edit `/etc/apache2/ports.conf`
    <pre>
    <code>
    Listen 15000 
    Listen 15500 
    </code>
    </pre>
  * Jalankan
    <pre>
    <code>
    a2ensite general.mecha.franky.e06-15000.com 
    a2ensite general.mecha.franky.e06-15500.com
    </code>
    </pre>
  * Pindah ke `/var/www/`
  * Buat folder baru, `general.mecha.franky.e06.com`
  * Download file yang disediakan `wget -O file.zip https://github.com/FeinardSlim/Praktikum-Modul-2-Jarkom/blob/main/general.mecha.franky.zip?raw=true`
  * Extract dengan `unzip`
  * Pindah ke LogueTown, check dengan
    <pre>
    <code>
    lynx www.general.mecha.franky.e06.com:15000
    lynx www.general.mecha.franky.e06.com:15500
    </code>
    </pre>
    ![image](https://user-images.githubusercontent.com/43901559/139073460-f164c36f-9540-49aa-9911-51a7c708895f.png)
    
    ![image](https://user-images.githubusercontent.com/43901559/139073480-1af3dde8-19e7-447b-82ad-2cec9cb5c74b.png)

## Nomor 15
### dengan authentikasi username luffy dan password onepiece dan file di /var/www/general.mecha.franky.yyy

### _Solusi_
  * Di Skypie, jalankan `htpasswd -c /etc/apache2/.htpasswd luffy`
  * Masukkan `onepiece` sebagai password ketika di prompt
  * Cek
    
    ![image](https://user-images.githubusercontent.com/43901559/139073754-307504c2-840a-4740-993d-726dcb3a3277.png)
  * Pada `/etc/apache2/sites-available/`, edit file `general.mecha.franky.e06-15000.com.conf` dan `general.mecha.franky.e06-15500.com.conf`
  * Tambahkan
    ```
    <Directory "var/www/general.mecha.franky.e01.com"> 
      AuthType Basic 
      AuthName "Restricted Content" 
      AuthUserFile /etc/apache2/.htpasswd 
      Require valid-user 
    </Directory>
    ```
  * Test
  
    ![image](https://user-images.githubusercontent.com/43901559/139074042-a236cd83-9432-461f-839a-11d7d6974048.png)
    
    ![image](https://user-images.githubusercontent.com/43901559/139074077-fc2ffa5f-ab99-4131-9015-40c507e302c2.png)

    ![image](https://user-images.githubusercontent.com/43901559/139074095-f49189dc-19d3-440e-ae71-3e28dcbb7900.png)

    ![image](https://user-images.githubusercontent.com/43901559/139074112-0439e98e-cc93-4991-b7ba-b7b57bd2b8d0.png)

    ![image](https://user-images.githubusercontent.com/43901559/139074129-3adaac6d-3342-4750-86b2-87cbed38e33f.png)

## Nomor 16
### Dan setiap kali mengakses IP Skypie akan diahlikan secara otomatis ke www.franky.yyy.com

### _Solusi_
  * Di Skypie, `/var/www/franky.e06.com`, tambahkan di `.htaccesss`
    ```
    RewriteCond %{HTTP_HOST} ^10\.32\.2\.4$
    RewriteRule ^(.*)$ http://www.franky.e06.com/$1 [L,R=301]
    ```
  * Check dengan `lynx 10.32.2.4` di LogueTown
  
    ![image](https://user-images.githubusercontent.com/43901559/139074409-5f25f12f-9379-4b79-a9ae-69bf98fb5784.png)

## Nomor 17
### Dikarenakan Franky juga ingin mengajak temannya untuk dapat menghubunginya melalui website www.super.franky.yyy.com, dan dikarenakan pengunjung web server pasti akan bingung dengan randomnya images yang ada, maka Franky juga meminta untuk mengganti request gambar yang memiliki substring “franky” akan diarahkan menuju franky.png. Maka bantulah Luffy untuk membuat konfigurasi dns dan web server ini!

### _Solusi_
  * Di Skypie, folder `/var/www/super.franky.e06.com`, edit `.htaccess`
    ```
    RewriteEngine On
    RewriteCond %{REQUEST_URI} franky
    RewriteRule .* /var/www/super.franky.e06.com/public/images/franky.png
    ```
  * Sekarang di `/etc/apache2/sites-available/super.franky.e06.com.conf`, tambahkan
    ```
    AllowOverride All
    ```
  * Testing di LogueTown, `lynx http://www.super.franky.e06.com/frankyasd`
    
    ![image](https://user-images.githubusercontent.com/43901559/139074713-c18903c4-4651-42eb-aaf7-146489ca47bb.png)

