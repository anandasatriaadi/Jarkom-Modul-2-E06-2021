# Jarkom-Modul-2-E06-2021

## Nomor 1
###	EniesLobby akan dijadikan sebagai DNS Master, Water7 akan dijadikan DNS Slave, dan Skypie akan digunakan sebagai Web Server. Terdapat 2 Client yaitu Loguetown, dan Alabasta. Semua node terhubung pada router Foosha, sehingga dapat mengakses internet

### _Solusi_
   ![image7](https://user-images.githubusercontent.com/36522826/139077922-f8d83fef-6bee-4de0-9908-5dd9f761779a.png)
   Terbuatlah topologi seperti gambar diatas, dengan IP node Skypie. Kemudian setting IP node Skypie menggunakan konfigurasi seperti ini

   ```
   auto eth0
   iface eth0 inet static
    address 10.32.2.4
    netmask 255.255.255.0
    gateway 10.32.2.1
   ```
   
## Nomor 2
###	Luffy ingin menghubungi Franky yang berada di EniesLobby dengan denden mushi. Kalian diminta Luffy untuk membuat website utama dengan mengakses franky.yyy.com dengan alias www.franky.yyy.com pada folder kaizoku

### _Solusi_
   * Mengedit /etc/bind/named.conf.local dengan konfigurasi seperti ini
   ```
   zone "franky.e06.com" {
       type master;
       file "/etc/bind/kaizoku/franky.e06.com";
   };
   ```
   * Kemudian membuat folder baru di /etc/bind dengan nama `kaizoku` dan copy file `etc/bind/db.local` dan beri nama `franky.e06.com` di `/etc/bind/kaizoku` dan konfigurasinya seperti ini
   ```
   ;
   ; BIND data file for local loopback interface
   ;
   $TTL    604800
   @       IN      SOA     franky.e06.com. root.franky.e06.com. (
                        2021100401         ; Serial
                            604800        	; Refresh
                             86400         ; Retry
                           2419200         ; Expire
                            604800 )       ; Negative Cache TTL
   ;
   @   IN NS    franky.e06.com.
   @   IN	A     10.32.2.2
   www IN	CNAME	franky.e06.com.
   ```
   * Ditambah juga CNAME record agar juga bisa mengakses www.franky.e06.com

   * Restart bind9 dan coba ping dari LogueTown (nameserver di LogueTown dirubah dulu ke 10.32.2.2)

   * Ping ke franky.e06.com
   ![image21](https://user-images.githubusercontent.com/36522826/139078723-a8b3e602-4a9b-46d8-ace0-7486c255c25f.png)
   
   * Ping ke www.franky.e06.com
   ![image17](https://user-images.githubusercontent.com/36522826/139078787-d0efe812-e436-43d0-b7df-4817d0f4cac4.png)

## Nomor 3
###	Setelah itu buat subdomain super.franky.yyy.com dengan alias www.super.franky.yyy.com yang diatur DNS nya di EniesLobby dan mengarah ke Skypie

### _Solusi_
   * Menambahkan subdomain dengan menambah A record dan CNAME record file `/etc/bind/kaizoku/franky.e06.com`, untuk IP nya diarahkan ke node Skypie
   ```
   ;
   ; BIND data file for local loopback interface
   ;
   $TTL    604800
   @       IN      SOA     franky.e06.com. root.franky.e06.com. (
                        2021100401         ; Serial
                            604800         ; Refresh
                             86400         ; Retry
                           2419200         ; Expire
                            604800 )       ; Negative Cache TTL
   ;
   @         IN      NS      franky.e06.com.
   @         IN      A       10.32.2.2 ;IP EniesLobby
   www       IN      CNAME   franky.e06.com.
   www.super IN      CNAME   super.franky.e06.com.
   super     IN      A       10.32.2.4
   ```

   * Kemudian restart bind dan coba ping ke super.franky.e06.com dan www.super.franky.e06.com
   ![image23](https://user-images.githubusercontent.com/36522826/139079307-b650eaf7-876b-4b54-b956-48f034112d0b.png)

## Nomor 4
###	Buat juga reverse domain untuk domain utama

### _Solusi_
   * Mengedit file `/etc/bind/named.conf.local` dan tambahkan konfigurasi ini
   ```
   zone "2.32.10.in-addr.arpa" {
       type master;
       file "/etc/bind/kaizoku/2.32.10.in-addr.arpa";
   };
   ```
   * Kemudian copy file `etc/bind/db.local` dan beri nama `2.32.10.in-addr.arpa` di `/etc/bind/kaizoku` dan konfigurasinya seperti ini
   ```
   ;
   ; BIND data file for local loopback interface
   ;
   $TTL    604800
   @       IN      SOA     franky.e06.com. root.franky.e06.com. (
                        2021100401         ; Serial
                            604800         ; Refresh
                             86400         ; Retry
                           2419200         ; Expire
                            604800 )       ; Negative Cache TTL
   ;
   2.32.10.in-addr.arpa.	IN      NS      franky.e06.com.
   2                     IN      PTR     franky.e06.com.
   ```
   * Kemudian jalankan command `host -t PTR "10.32.2.2"` di LogueTown
   ![image8](https://user-images.githubusercontent.com/36522826/139080223-5e2ca796-e1fe-43b8-ade7-546818922c68.png)

## Nomor 5
###	Supaya tetap bisa menghubungi Franky jika server EniesLobby rusak, maka buat Water7 sebagai DNS Slave untuk domain utama

### _Solusi_
   * Edit file /etc/bind/named.conf.local di EniesLobby dan tambahkan konfigurasi seperti ini
   ```
   zone "franky.e06.com" {
       type master;
       notify yes;
       also-notify { 10.32.2.3; };
       allow-transfer { 10.32.2.3; };
       file "/etc/bind/kaizoku/franky.e06.com";
   };
   ```
   * Restart bind. Kemudian ke Water7 dan edit file /etc/bind/named.conf.local 
   ```
   zone "franky.e06.com" {
       type slave;
       masters { 10.32.2.2; };
       file "/var/lib/bind/franky.e06.com";
   };
   ```
   * Kemudian untuk ceknya di stop terlebih dahulu service bind9 di node EniesLobby
   ![image3](https://user-images.githubusercontent.com/36522826/139080634-6bbee150-6239-4689-a243-2048cdb1c132.png)
   
   * Kemudian ganti nameserver di LogueTown dan coba ping franky.e06.com
   ![image21](https://user-images.githubusercontent.com/36522826/139080735-dffec02e-ef15-4a76-a19c-4a2b1e1180c7.png)

## Nomor 6
###	Setelah itu terdapat subdomain mecha.franky.yyy.com dengan alias www.mecha.franky.yyy.com yang didelegasikan dari EniesLobby ke Water7 dengan IP menuju ke Skypie dalam folder sunnygo

### _Solusi_
   * Edit terlebih dahulu file `/etc/bind/kaizoku/franky.e06.com` dan tambahkan NS record
   ```
   ;
   ; BIND data file for local loopback interface
   ;
   $TTL    604800
   @       IN      SOA     franky.e06.com. root.franky.e06.com. (
                        2021100401         ; Serial
                            604800         ; Refresh
                             86400         ; Retry
                           2419200         ; Expire
                            604800 )       ; Negative Cache TTL
   ;
   @         IN      NS      franky.e06.com.
   @         IN      A       10.32.2.2 ;IP EniesLobby
   www       IN      CNAME   franky.e06.com.
   www.super IN      CNAME   super.franky.e06.com.
   super     IN      A       10.32.2.4
   ns1       IN      A       10.32.2.4
   mecha     IN      NS      ns1
   ```
   * Kemudian edit file `/etc/bind/named.conf.options` pada EniesLobby. Tambahkan `allow-query{any;};` dan comment `nssec-validation auto;`

   * Edit file `/etc/bind/named.conf.local`
   ```
   zone "franky.e06.com" {
       type master;
       notify yes;
       also-notify { 10.32.2.3; };
       allow-transfer { 10.32.2.3; };
       file "/etc/bind/kaizoku/franky.e06.com";
   };
   ```
   * Kemudian restart service bind9. Pada node Water7 edit file `/etc/bind/named.conf.options` pada Water7. Tambahkan `allow-query{any;};` dan comment `dnssec-validation auto;`

   * Kemudian edit `/etc/bind/named.conf.local` di Water7
   ```
   zone "mecha.franky.e06.com" {
       type master;
       file "/etc/bind/sunnygo/mecha.franky.e06.com";
   };
   ```
   * Kemudian membuat folder baru di /etc/bind dengan nama “sunnygo” dan copy file etc/bind/db.local dan beri nama mecha.franky.e06.com di /etc/bind/sunnygo dan konfigurasinya seperti ini
   ```
   ;
   ; BIND data file for local loopback interface
   ;
   $TTL    604800
   @       IN      SOA     mecha.franky.e06.com. root.mecha.franky.e06.com. (
                        2021100401         ; Serial
                            604800         ; Refresh
                             86400         ; Retry
                           2419200         ; Expire
                            604800 )       ; Negative Cache TTL
   ;
   @	  IN      NS      mecha.franky.e06.com.
   @	  IN      A       10.32.2.4
   www IN      CNAME   mecha.franky.e06.com.
   ```
   * Restart bind dan coba ping dari LogueTown  
   ![image26](https://user-images.githubusercontent.com/36522826/139081435-e7f42edb-e625-4ef8-9a1b-e995d953022e.png)  

## Nomor 7
###	Untuk memperlancar komunikasi Luffy dan rekannya, dibuatkan subdomain melalui Water7 dengan nama general.mecha.franky.yyy.com dengan alias www.general.mecha.franky.yyy.com yang mengarah ke Skypie.

### _Solusi_
Karena kita perlu menambahkan subdomain untuk mecha.franky.yyy.com, maka pada Water7 edit */etc/bind/sunnygo/franky.e06.com* tambahkan A record dan CNAME record untuk www.general.franky.e06.com  
  
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     mecha.franky.e06.com. root.mecha.franky.e06.com. (
      2021102401		; Serial
      604800		; Refresh
      86400			; Retry
      2419200		; Expire
      604800 )		; Negative Cache TTL
;
@		IN	NS		mecha.franky.e06.com.
@		IN	A		10.32.2.4
www		IN	CNAME	mecha.franky.e06.com.
www.general	IN	CNAME	mecha.franky.e06.com.
general	IN	A		10.32.2.4
```  
Setelah diubah, lakukan `service bind9 restart` dan coba lakukan ping dari client.  
![image](https://user-images.githubusercontent.com/57354564/139165888-810e4e1f-25b3-4916-958c-79aa87735c29.png)  

## Nomor 8
###	Setelah itu terdapat subdomain mecha.franky.yyy.com dengan alias www.mecha.franky.yyy.com yang didelegasikan dari EniesLobby ke Water7 dengan IP menuju ke Skypie dalam folder sunnygo

### _Solusi_
Pertama, ubah config `/etc/bind/kaizoku/franky.e06.com` pada EniesLobby kemudian ubah IP A record pertama menjadi IP Skypie.
![image](https://user-images.githubusercontent.com/57354564/139166710-ae8d12f2-033b-4c4d-bd02-2920c0b81358.png)  
  
Copy file `/etc/apache2/sites-available/000-default.conf` ke `/etc/apache2/sites-available/franky.e06.com.conf` dengan isi file sebagai berikut:  
```
<VirtualHost *:80>

        ServerName franky.e06.com
        ServerAlias www.franky.e06.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/franky.e06.com
        <Directory /var/www/franky.e06.com>
                Options +FollowSymLinks -Multiviews
                AllowOverride All
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```
![image](https://user-images.githubusercontent.com/57354564/139166806-bb1c1bf4-49e0-4c6a-a9ea-10b570182012.png)  
Kemudian tambahkan folder franky.e06.com pada /var/www/ dalam folder tersebut download file zip yang sudah disediakan dan unzip pada folder yang baru dibuat.  
  
Coba akses `lynx franky.206.com`
![image](https://user-images.githubusercontent.com/57354564/139167417-785dd8c1-5828-44c4-86b9-416fccf4345a.png)


## Nomor 9
###	Setelah itu, Luffy juga membutuhkan agar url **www.franky.yyy.com/index.php/home** dapat menjadi menjadi **www.franky.yyy.com/home**.

### _Solusi_
Untuk dapat mengubah url awal menjadi **www.franky.yyy.com/home**. Maka perlu enable plugin rewrite dengan **a2enmod rewrite** kemudian pada folder **/var/www/franky.e06.com/** tambah file **.htaccess** dengan isi file:
```
RewriteEngine On
RewriteRule ^home$ index.php/home
```
Test dengan melakukan lynx franky.e06.com/home

![image](https://user-images.githubusercontent.com/57354564/139166930-71bb0b0e-483f-4e9d-a1a6-ea2d45c9fe20.png)

## Nomor 10
###	Setelah itu, pada subdomain **www.super.franky.yyy.com**, Luffy membutuhkan penyimpanan aset yang memiliki DocumentRoot pada **/var/www/super.franky.yyy.com**.   

### _Solusi_
Copy file `/etc/apache2/sites-available/000-default.conf` ke `/etc/apache2/sites-available/super.franky.e06.com.conf` dengan isi file sebagai berikut:

```
<VirtualHost *:80>

        ServerName super.franky.e06.com
        ServerAlias www.super.franky.e06.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/super.franky.e06.com

</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

Kemudian tambahkan folder super.franky.e06.com pada /var/www/ dalam folder tersebut download file zip yang sudah disediakan dan unzip pada folder yang baru dibuat.  

## Nomor 11
###	Akan tetapi, pada folder /public, Luffy ingin hanya dapat melakukan directory listing saja. 

### _Solusi_
Pada `/etc/apache2/sites-available/super.franky.e06.com.conf` tambahkan konfigurasi dengan tag Directory sebagai berikut:

```
     <Directory /var/www/super.franky.e06.com>
                Options +Indexes
     </Directory>

     <Directory /var/www/super.franky.e06.com/public>
                Options +Indexes
     </Directory>
```
![image](https://user-images.githubusercontent.com/57354564/139167116-b7fee4d1-a5a1-42ed-835a-d03d29803adf.png)

Test dengan mengakses folder `lynx super.franky.e06.com/public`  

![image](https://user-images.githubusercontent.com/57354564/139167162-a9ca4633-9c0e-44dd-a378-5fcb4ab177ab.png)

## Nomor 12
###	Tidak hanya itu, Luffy juga menyiapkan error file 404.html pada folder /error untuk mengganti error kode pada apache. 

### _Solusi_
Pada `/etc/apache2/sites-available/super.franky.e06.com.conf` tambahkan konfigurasi ErrorDocument sebagai berikut:
```
ErrorDocument 404 /error/404.html
```
![image](https://user-images.githubusercontent.com/57354564/139167274-9c393faf-5873-46ae-a4f4-0ce25c4fc6f5.png)

Mencoba mengakses invalid url misal lynx super.franky.e06.com/loguetown  

![image](https://user-images.githubusercontent.com/57354564/139167305-38c072c0-f63d-42ae-8cd6-3f1a16b4d302.png)

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

