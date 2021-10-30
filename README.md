# Jarkom-Modul-2-A11-2021

## Anggota

1. Frederick William Edlim 05111940000016
2. Thomas Dwi Awaka 05111940000021
3. Allam Taju Sarof 05111940000053

## 1. Membuat nodes yang dapat mengakses internet

- Membuat topologi sebagai berikut :
  ![gambar topologi](images/nomor%201%20gambar%201.jpg)

- Lalu setting pada setiap IP node sebagai berikut
  - Foosha
    ![gambar setting_foosha](images/nomor%201%20gambar%202.jpg)
  - Loguetown
    ![gambar setting_loguetown](images/nomor%201%20gambar%203.jpg)
  - Alabasta
    ![gambar setting_alabasta](images/nomor%201%20gambar%204.jpg)
  - EniesLobby
    ![gambar setting_enieslobby](images/nomor%201%20gambar%205.jpg)
  - Water7
    ![gambar setting_water7](images/nomor%201%20gambar%206.jpg)
  - Skypie
    ![gambar setting_skypie](images/nomor%201%20gambar%207.jpg)
- Lalu pada `.bashrc` di Foosha ditambahkan line
  berikut `iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.174.0.0/16`
- Lalu pada masing masing node client dan server ditambahkan line berikut pada file `.bashrc` :

  ```
  echo nameserver 192.174.2.2 > /etc/resolv.conf
  echo nameserver 192.174.2.3 >> /etc/resolv.conf
  echo nameserver 192.168.122.1 >> /etc/resolv.conf

  ```

- Sehingga tiap node dapat melakukan ping ke`google.com`
  ![gambar ping_google](images/nomor%201%20gambar%208.jpg)

## 2. Membuat website utama `franky.a11.com` dengan alias `www.franky.a11.com`

- Membuat sebuah file `restart-dns.sh` pada EniesLobby yang berisi berikut :

  ```
  cd /etc/bind
  cp ~/named.conf.local named.conf.local
  cp ~/named.conf.options named.conf.options
  rm -rf kaizoku
  mkdir kaizoku
  cd kaizoku/
  cp ~/franky.a11.com franky.a11.com
  cp ~/2.174.182.in-addr.arpa 2.174.182.in-addr.arpa

  service bind9 restart

  ```

  tujuan dari file ini adalah dengan mudah untuk melakukan restart dns jika diperlukan

- Lalu pada folder root dibuat file `named.conf.local` yang berisi sebagai berikut
  ```
  zone "franky.a11.com" {
        type master;
        notify yes;
        also-notify { 192.174.2.3; };
        allow-transfer { 192.174.2.3; };
        file "/etc/bind/kaizoku/franky.a11.com";
  };
  ```
- Lalu pada folder root dibuat file `named.conf.options` yang berisi sebagai berikut

  ```
  options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

        // forwarders {
        //      0.0.0.0;
        // };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        //dnssec-validation auto;
        allow-query{any;};

        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
  };
  ```

- Lalu pada folder root dibuat file `franky.a11.com` yang berisi sebagai berikut
  ```
    $TTL    604800
    @       IN      SOA     franky.a11.com. root.franky.a11.com. (
                    2021102401              ; Serial
                    604800         ; Refresh
                    86400         ; Retry
                    2419200         ; Expire
                    604800 )       ; Negative Cache TTL
    ;
    @       IN      NS      franky.a11.com.
    @       IN      A       192.174.2.4
    www     IN      CNAME   franky.a11.com.
  ```
- Lalu jalankan file `restart-dns.sh` dengan perintah `bash restart-dns.sh`
- Lalu lakukan ping menuju `franky.a11.com`
  ![gambar ping_franky.a11.com](images/nomor%202%20gambar%201.jpg)

## 3. Buat subdomain `super.franky.a11.com` dengan alias `www.super.franky.a11.com`

- Ubah file `franky.a11.com` sehingga berisi sebagai berikut
  ```
    $TTL    604800
    @       IN      SOA     franky.a11.com. root.franky.a11.com. (
                    2021102401              ; Serial
                    604800         ; Refresh
                    86400         ; Retry
                    2419200         ; Expire
                    604800 )       ; Negative Cache TTL
    ;
    @       IN      NS      franky.a11.com.
    @       IN      A       192.174.2.4
    www     IN      CNAME   franky.a11.com.
    super   IN      A       192.174.2.4
    www.super   IN      A       192.174.2.4
  ```
- Lalu jalankan file `restart-dns.sh` dengan perintah `bash restart-dns.sh`
- Lalu lakukan ping menuju `super.franky.a11.com`
  ![gambar ping_super.franky.a11.com](images/nomor%203%20gambar%201.jpg)

## 4. Buat reverse domain untuk domain utama

- Ubah file `named.conf.local` sehingga berisi sebagai berikut

  ```
  zone "franky.a11.com" {
        type master;
        notify yes;
        also-notify { 192.174.2.3; };
        allow-transfer { 192.174.2.3; };
        file "/etc/bind/kaizoku/franky.a11.com";
  };

  zone "2.174.192.in-addr.arpa" {
        type master;
        file "/etc/bind/kaizoku/2.174.192.in-addr.arpa";
  };
  ```

- Pada folder root dibuat file `2.174.192.in-addr.arpa` yang berisi sebagai berikut
  ```
    $TTL    604800
    @       IN      SOA     franky.a11.com. root.franky.a11.com. (
                    2021102401              ; Serial
                    604800         ; Refresh
                    86400         ; Retry
                    2419200         ; Expire
                    604800 )       ; Negative Cache TTL
    ;
    2.174.192.in-addr.arpa.         IN      NS      franky.a11.com.
    4       IN      PTR     franky.a11.com.
  ```
- Lalu jalankan file `restart-dns.sh` dengan perintah `bash restart-dns.sh`
- Lalu lakukan test dengan perintah `host -t PTR 192.174.2.4`
  ![gambar_test_reverse_dns](images/nomor%204%20gambar%201.jpg)

## 5. Buat Water7 sebagai DNS Slave

- Membuat sebuah file `restart-dns.sh` pada EniesLobby yang berisi berikut :

  ```
  cd /etc/bind
  cp ~/named.conf.local named.conf.local
  cp ~/named.conf.options named.conf.options
  rm -rf sunnygo
  mkdir sunnygo
  cd sunnygo/
  cp ~/mecha.franky.a11.com mecha.franky.a11.com

  service bind9 restart
  ```

- Lalu pada folder root dibuat file `named.conf.local` yang berisi sebagai berikut
  ```
  zone "franky.a11.com" {
    type slave;
    masters { 192.174.2.2; }; // Masukan IP EniesLobby tanpa tanda petik
    file "/var/lib/bind/franky.a11.com";
  };
  ```
- Lalu pada folder root dibuat file `named.conf.options` yang berisi sebagai berikut

  ```
  options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

        // forwarders {
        //      0.0.0.0;
        // };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        //dnssec-validation auto;
        allow-query{any;};

        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
  };
  ```

- Lalu jalankan file `restart-dns.sh` dengan perintah `bash restart-dns.sh`
- Lalu test dengan cara menghentikan service bind9 di EniesLobby dan lakukan ping kepada franky.a11.com melalui
  Loguetown atau Alabasta
  ![gambar_test_dns_slave](images/nomor%205%20gambar%201.jpg)

## 6. Buat subdomain `mehcha.franky.a11.com` yang didelegasikan ke Water7

- Ubah file `franky.a11.com` pada EniesLobby sehingga berisi sebagai berikut
  ```
    $TTL    604800
    @       IN      SOA     franky.a11.com. root.franky.a11.com. (
                    2021102401              ; Serial
                    604800         ; Refresh
                    86400         ; Retry
                    2419200         ; Expire
                    604800 )       ; Negative Cache TTL
    ;
    @       IN      NS      franky.a11.com.
    @       IN      A       192.174.2.4
    www     IN      CNAME   franky.a11.com.
    super   IN      A       192.174.2.4
    www.super   IN      A       192.174.2.4
    ns1     IN      A       192.174.2.3
    mecha   IN      NS      ns1
  ```
- Lalu jalankan file `restart-dns.sh` dengan perintah `bash restart-dns.sh`
- Lalu pada Water7 file `named.conf.local` diubah sehingga berisi sebagai berikut

  ```
  zone "franky.a11.com" {
    type slave;
    masters { 192.174.2.2; }; // Masukan IP EniesLobby tanpa tanda petik
    file "/var/lib/bind/franky.a11.com";
  };

  zone "mecha.franky.a11.com" {
    type master;
    file "/etc/bind/sunnygo/mecha.franky.a11.com";
  };
  ```

- Lalu buat file `mecha.franky.a11.com` di root dengan isi sebagai berikut
  ```
    $TTL    604800
    @       IN      SOA     mecha.franky.a11.com. root.mecha.franky.a11.com. (
                    2021102401              ; Serial
                    604800         ; Refresh
                    86400         ; Retry
                    2419200         ; Expire
                    604800 )       ; Negative Cache TTL
    ;
    @       IN      NS      mecha.franky.a11.com.
    @       IN      A       192.174.2.4
    www     IN      CNAME   mecha.franky.a11.com.
  ```
- Lalu jalankan file `restart-dns.sh` dengan perintah `bash restart-dns.sh`
- Lalu test dengan lakukan ping mecha.franky.a11.com
  ![gambar_test_mecha.franky.a11.com](images/nomor%206%20gambar%201.jpg)

## 7. Buat subdomain general.mecha.franky.a11.com

- Ubah file `mecha.franky.a11.com` sehingga isi sebagai berikut
  ```
    $TTL    604800
    @       IN      SOA     mecha.franky.a11.com. root.mecha.franky.a11.com. (
                    2021102401              ; Serial
                    604800         ; Refresh
                    86400         ; Retry
                    2419200         ; Expire
                    604800 )       ; Negative Cache TTL
    ;
    @       IN      NS      mecha.franky.a11.com.
    @       IN      A       192.174.2.4
    www     IN      CNAME   mecha.franky.a11.com.
    general IN      A       192.174.2.4
    www.general IN  A       192.174.2.4
  ```
- Lalu jalankan file `restart-dns.sh` dengan perintah `bash restart-dns.sh`
- Lalu test dengan lakukan ping general.mecha.franky.a11.com
  ![gambar_test_general.mecha.franky.a11.com](images/nomor%207%20gambar%201.jpg)

## 8. Buat webserver www.franky.a11.com dengan DocumentRoot berada di /var/www/franky.a11.com
* Pada Skypie buat file `restart-apache.sh` yang isinya sebagai berikut : 
  ```
  cp -a  ~/apache-config/. /etc/apache2/sites-available/
  cp ~/ports.conf /etc/apache2/

  a2enmod rewrite

  cp ~/apache-default/.htaccess /var/www/html

  a2ensite franky.a11.com
  a2ensite super.franky.a11.com
  a2ensite general.mecha.franky.a11.com

  rm -rf /var/www/franky.a11.com
  mkdir /var/www/franky.a11.com
  cp -a ~/Praktikum-Modul-2-Jarkom/franky/. /var/www/franky.a11.com

  rm -rf /var/www/super.franky.a11.com
  mkdir /var/www/super.franky.a11.com
  cp -a ~/Praktikum-Modul-2-Jarkom/super.franky/. /var/www/super.franky.a11.com

  rm -rf /var/www/general.mecha.franky.a11
  mkdir /var/www/general.mecha.franky.a11
  cp -a ~/Praktikum-Modul-2-Jarkom/general.mecha.franky/. /var/www/general.mecha.franky.a11

  service apache2 restart
  ```
* Lalu buat file config di folder apache-config dengan nama `franky.a11.com.conf`
  ```
  <VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/franky.a11.com
        ServerName franky.a11.com
        ServerAlias www.franky.a11.com

        <Directory /var/www/franky.a11.com/> 
                Options +Indexes
                AllowOverride All
        </Directory>
        
        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf
  </VirtualHost>

  # vim: syntax=apache ts=4 sw=4 sts=4 sr noet
  ```
* Lalu jalankan file `restart-apache.sh` dengan perintah `bash restart-apache.sh`
* Test dengan melakukan `lynx www.franky.a11.com`
  ![gambar_test_webserver_franky](images/nomor%208%20gambar%201.jpg)

## 9. Mengganti url www.franky.a11.com/index.php/home menjadi www.franky.a11.com/home

* Pada folder ~/Praktikum-Modul-2-Jarkom/franky buat sebuah file `.htaccess` dengan isi sebagai berikut
  ```
  RewriteEngine On
  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteRule ^([^\.]+)$ $1index.php [NC,L]
  ```
* Lalu jalankan file `restart-apache.sh` dengan perintah `bash restart-apache.sh`
* Test dengan melakukan `lynx www.franky.a11.com/index.php/home`
  ![gambar_test_webserver_franky](images/nomor%209%20gambar%201.jpg)

## 10. subdomain www.super.franky.a11.com, buat penyimpanan aset yang memiliki DocumentRoot pada /var/www/super.franky.a11.com
* Lalu buat file config di folder apache-config dengan nama `super.franky.a11.com.conf`
  ```
  <VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/super.franky.a11.com
        ServerName super.franky.a11.com
        ServerAlias www.super.franky.a11.com

        <Directory /var/www/super.franky.a11.com/>
            AllowOverride All
        </Directory>

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf
  </VirtualHost>

  # vim: syntax=apache ts=4 sw=4 sts=4 sr noet
  ```
* Lalu jalankan file `restart-apache.sh` dengan perintah `bash restart-apache.sh`
* Test dengan melakukan `lynx www.super.franky.a11.com`
  ![gambar_test_webserver_super.franky](images/nomor%2010%20gambar%201.jpg)

## 11. Buat folder public menjadi directory listing saja
* Ubah config di folder apache-config dengan nama `super.franky.a11.com.conf` sehingga menjadi seperti ini
  ```
  <VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/super.franky.a11.com
        ServerName super.franky.a11.com
        ServerAlias www.super.franky.a11.com

        <Directory /var/www/super.franky.a11.com/public>
            Options +Indexes
        </Directory>

        <Directory /var/www/super.franky.a11.com/>
            AllowOverride All
        </Directory>

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf
  </VirtualHost>

  # vim: syntax=apache ts=4 sw=4 sts=4 sr noet
  ```
* Lalu jalankan file `restart-apache.sh` dengan perintah `bash restart-apache.sh`
* Test dengan melakukan `lynx www.super.franky.a11.com/public`
  ![gambar_test_webserver_super.franky](images/nomor%2011%20gambar%201.jpg)

## 12. Error file 404 khusus
* Buat file `.htacces` pada folder ~/Praktikum-Modul-2-Jarkom/super.franky dengan isi sebagai berikut 
  ```
  ErrorDocument 404 /error/404.html
  ```
* Lalu jalankan file `restart-apache.sh` dengan perintah `bash restart-apache.sh`
* Test dengan melakukan `lynx www.super.franky.a11.com/jsss`
  ![gambar_test_webserver_super.franky](images/nomor%2012%20gambar%201.jpg)

## 13. Virtual host untuk dapat mengakses file asset www.super.franky.a11.com/public/js menjadi www.super.franky.a11.com/js

- Ubah config di folder apache-config dengan nama `super.franky.a11.com.conf` sehingga menjadi seperti ini

  ```
  <VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/super.franky.a11.com
        ServerName super.franky.a11.com
        ServerAlias www.super.franky.a11.com

        <Directory /var/www/super.franky.a11.com/public>
            Options +Indexes
        </Directory>

        <Directory /var/www/super.franky.a11.com/>
            AllowOverride All
        </Directory>

        <Directory /var/www/super.franky.a11.com/public/js>
            Options +Indexes
        </Directory>
        Alias "/js" "/var/www/super.franky.a11.com/public/js"

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf
  </VirtualHost>

  # vim: syntax=apache ts=4 sw=4 sts=4 sr noet
  ```

- Lalu jalankan file `restart-apache.sh` dengan perintah `bash restart-apache.sh`
- Test dengan melakukan `lynx www.super.franky.a11.com/js`
  ![gambar_test_webserver_super.franky](images/nomor%2013%20gambar%201.jpg)

## 14. Web general.mecha.franky.a11.com hanya bisa diakses melalui port 15000 dan 15500

- Buat file `ports.conf` dengan isi :

  ```
  # If you just change the port or add more ports here, you will likely also
  # have to change the VirtualHost statement in
  # /etc/apache2/sites-enabled/000-default.conf

  Listen 80
  Listen 15000
  Listen 15500

  <IfModule ssl_module>
          Listen 443
  </IfModule>

  <IfModule mod_gnutls.c>
          Listen 443
  </IfModule>

  # vim: syntax=apache ts=4 sw=4 sts=4 sr noet
  ```

- Lalu buat file config di folder apache-config dengan nama `general.mecha.franky.a11.com.conf`

  ```
  <VirtualHost *:15000 *:15500>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/general.mecha.franky.a11
        ServerName general.mecha.franky.a11.com
        ServerAlias www.general.mechafranky.a11.com

        <Directory /var/www/general.mecha.franky.a11/>
                Options +Indexes
        </Directory>

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf
  </VirtualHost>

  # vim: syntax=apache ts=4 sw=4 sts=4 sr noet
  ```

- Lalu jalankan file `restart-apache.sh` dengan perintah `bash restart-apache.sh`
- Test dengan melakukan `lynx www.general.mecha.franky.a11.com:15000`
  ![gambar_test_webserver_general.mecha.franky](images/nomor%2014%20gambar%202.jpg)

## 15. Web general.mecha.franky.a11.com memiliki autentikasi dengan username luffy dan password onepiece

- ubah file `general.mecha.franky.a11.com.conf` menjadi

  ```
  <VirtualHost *:15000 *:15500>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/general.mecha.franky.a11
        ServerName general.mecha.franky.a11.com
        ServerAlias www.general.mechafranky.a11.com

        <Directory /var/www/general.mecha.franky.a11/>
                Options +Indexes
                AuthType Basic
                AuthName "Restricted Content"
                AuthUserFile /var/www/general.mecha.franky.a11/.htpasswd
                Require valid-user
        </Directory>

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf
  </VirtualHost>

  # vim: syntax=apache ts=4 sw=4 sts=4 sr noet
  ```

- Buat file `.htpasswd` pada folder ~/Praktikum-Modul-2-Jarkom/general.mecha.franky dengan isi sebagai berikut
  ```
  luffy:$apr1$xPSiTGNW$RNZqtqAknyvtISrVP0coY/
  ```
  dimana ini didapatkan dengan mengeksekusi perintah `htpasswd ~/Praktikum-Modul-2-Jarkom/general.mecha.franky/.htpasswd luffy`
- Lalu jalankan file `restart-apache.sh` dengan perintah `bash restart-apache.sh`
- Test dengan melakukan `lynx www.general.mecha.franky.a11.com:15000`
  ![gambar_test_webserver_general.mecha.franky](images/nomor%2014%20gambar%201.jpg)
  ![gambar_test_webserver_general.mecha.franky](images/nomor%2014%20gambar%202.jpg)

## 16. Setiap kali mengakses IP dari Skypie akan di redirect menuju www.franky.a11.com

- Buat file `000-default.conf` di folder apache-config yang nantinya akan digunakan untuk mereplace faile default yang ada

  ```
  <VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html

        <Directory /var/www/html>
                AllowOverride All
        </Directory>

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf
  </VirtualHost>

  # vim: syntax=apache ts=4 sw=4 sts=4 sr noet
  ```

- Buat `.htaccess` di folder apache-default
  ```
  RewriteEngine On
  RewriteBase /
  RewriteCond %{HTTP_HOST} ^192\.174\.2\.4$
  RewriteRule ^(.*)$ http://www.franky.a11.com/$1 [L,R=301]
  ```
- Lalu jalankan file `restart-apache.sh` dengan perintah `bash restart-apache.sh`
- Test dengan melakukan `lynx 192.174.2.4`
  ![gambar_test_webserver_ip_skypie](images/nomor%2016%20gambar%201.jpg)

## 17. Redirect semua gambar yang ada di super.franky.a11.com yang memiliki kata kata franky menuju franky.png

- Pada folder ~/Praktikum-Modul-2-Jarkom/super.franky/public/images buat sebuah file `.htaccess` dengan isi sebagai berikut
  ```
  RewriteEngine On
  RewriteBase /
  RewriteCond %{REQUEST_FILENAME} !\bfranky.png\b
  RewriteRule franky http://www.super.franky.a11.com/public/images/franky.png$1 [L,R=301]
  ```
- Lalu jalankan file `restart-apache.sh` dengan perintah `bash restart-apache.sh`
- Test dengan melakukan `lynx www.super.franky.a11.com/public/images/eyeoffranky.jpg`, maka akan diredirect untuk mendownload gambar franky.png
  ![gambar_test_webserver_ip_skypie](images/nomor%2017%20gambar%201.jpg)
  
# Kendala
  Saat pengerjaan, terdapat kendala koneksi terganggu saat melakukan `apt install` sehingga harus menyediakan kuota tambahan untuk tethering demi kelancaran saat mengerjakan
