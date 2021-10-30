# Jarkom-Modul-2-A11-2021

## Anggota

1. Frederick William Edlim 05111940000016
2. Allam
3. Tom

## 1. Membuat nodes yang dapat mengakses internet

* Membuat topologi sebagai berikut :
  ![gambar topologi](images/nomor%201%20gambar%201.jpg)

* Lalu setting pada setiap IP node sebagai berikut
    * Foosha
      ![gambar setting_foosha](images/nomor%201%20gambar%202.jpg)
    * Loguetown
      ![gambar setting_loguetown](images/nomor%201%20gambar%203.jpg)
    * Alabasta
      ![gambar setting_alabasta](images/nomor%201%20gambar%204.jpg)
    * EniesLobby
      ![gambar setting_enieslobby](images/nomor%201%20gambar%205.jpg)
    * Water7
      ![gambar setting_water7](images/nomor%201%20gambar%206.jpg)
    * Skypie
      ![gambar setting_skypie](images/nomor%201%20gambar%207.jpg)
* Lalu pada `.bashrc` di Foosha ditambahkan line
  berikut `iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.174.0.0/16`
* Lalu pada masing masing node client dan server ditambahkan line berikut pada file `.bashrc` :

    ```
    echo nameserver 192.174.2.2 > /etc/resolv.conf
    echo nameserver 192.174.2.3 >> /etc/resolv.conf 
    echo nameserver 192.168.122.1 >> /etc/resolv.conf

    ```
* Sehingga tiap node dapat melakukan ping ke`google.com`
  ![gambar ping_google](images/nomor%201%20gambar%208.jpg)

## 2. Membuat website utama `franky.a11.com` dengan alias `www.franky.a11.com`

* Membuat sebuah file `restart-dns.sh` pada EniesLobby yang berisi berikut :
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
* Lalu pada folder root dibuat file `named.conf.local` yang berisi sebagai berikut
  ```
  zone "franky.a11.com" {
        type master;
        notify yes;
        also-notify { 192.174.2.3; };
        allow-transfer { 192.174.2.3; };
        file "/etc/bind/kaizoku/franky.a11.com";
  };
  ```
* Lalu pada folder root dibuat file `named.conf.options` yang berisi sebagai berikut
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
* Lalu pada folder root dibuat file `franky.a11.com` yang berisi sebagai berikut
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
* Lalu jalankan file `restart-dns.sh` dengan perintah `bash restart-dns.sh`
* Lalu lakukan ping menuju `franky.a11.com`
  ![gambar ping_franky.a11.com](images/nomor%202%20gambar%201.jpg)

# 3. Buat subdomain `super.franky.a11.com` dengan alias `www.super.franky.a11.com`

* Ubah file `franky.a11.com` sehingga berisi sebagai berikut
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
* Lalu jalankan file `restart-dns.sh` dengan perintah `bash restart-dns.sh`
* Lalu lakukan ping menuju `super.franky.a11.com`
  ![gambar ping_super.franky.a11.com](images/nomor%203%20gambar%201.jpg)

# 4. Buat reverse domain untuk domain utama

* Ubah file `named.conf.local` sehingga berisi sebagai berikut
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
* Pada folder root dibuat file `2.174.192.in-addr.arpa` yang berisi sebagai berikut
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
* Lalu jalankan file `restart-dns.sh` dengan perintah `bash restart-dns.sh`
* Lalu lakukan test dengan perintah `host -t PTR 192.174.2.4`
  ![gambar_test_reverse_dns](images/nomor%204%20gambar%201.jpg)

# 5. Buat Water7 sebagai DNS Slave

* Membuat sebuah file `restart-dns.sh` pada EniesLobby yang berisi berikut :
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
* Lalu pada folder root dibuat file `named.conf.local` yang berisi sebagai berikut
  ```
  zone "franky.a11.com" {
    type slave;
    masters { 192.174.2.2; }; // Masukan IP EniesLobby tanpa tanda petik
    file "/var/lib/bind/franky.a11.com";
  };
  ```
* Lalu pada folder root dibuat file `named.conf.options` yang berisi sebagai berikut
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
* Lalu jalankan file `restart-dns.sh` dengan perintah `bash restart-dns.sh`
* Lalu test dengan cara menghentikan service bind9 di EniesLobby dan lakukan ping kepada franky.a11.com melalui
  Loguetown atau Alabasta
  ![gambar_test_dns_slave](images/nomor%205%20gambar%201.jpg)

## 6. Buat subdomain `mehcha.franky.a11.com` yang didelegasikan ke Water7

* Ubah file `franky.a11.com` pada EniesLobby sehingga berisi sebagai berikut
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
* Lalu jalankan file `restart-dns.sh` dengan perintah `bash restart-dns.sh`
* Lalu pada Water7 file `named.conf.local` diubah sehingga berisi sebagai berikut
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
* Lalu buat file `mecha.franky.a11.com` di root dengan isi sebagai berikut 
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
* Lalu jalankan file `restart-dns.sh` dengan perintah `bash restart-dns.sh`
* Lalu test dengan lakukan ping mecha.franky.a11.com
  ![gambar_test_mecha.franky.a11.com](images/nomor%206%20gambar%201.jpg)

# 7. Buat subdomain general.mecha.franky.a11.com
* Ubah file `mecha.franky.a11.com` sehingga isi sebagai berikut
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
* Lalu jalankan file `restart-dns.sh` dengan perintah `bash restart-dns.sh`
* Lalu test dengan lakukan ping general.mecha.franky.a11.com
  ![gambar_test_general.mecha.franky.a11.com](images/nomor%207%20gambar%201.jpg)
