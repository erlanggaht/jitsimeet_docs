
# Jitsi Meet Ubuntu 22

## Install Jitsi Server


### 1. Update apt
```
sudo apt update && sudo apt upgrade
```

```
sudo apt install wget
```

### 2. Set FQDN untuk Jitsi

```
sudo hostnamectl set-hostname jisti.erlanggaht.com
```

setting hosts ubuntu :

```
sudo nano /etc/hosts
```

tambahkan ini didalamnya

x.x.x.x jisti.erlanggaht.com

x.x.x.x : ganti dengan ip local ubuntu.

setelah itu restart ubuntu atau virtualbox
```
sudo reboot
```

### 3. Tambahkan kunci dan repository Jitsi GPG

Kita perlu menambahkan repositori paket Jisti Meet secara manual di Ubuntu 22.04 kita karena tidak tersedia melalui repositori standar Ubuntu. 

```
curl https://download.jitsi.org/jitsi-key.gpg.key | sudo sh -c 'gpg --dearmor > /usr/share/keyrings/jitsi-keyring.gpg'
```

```
echo 'deb [signed-by=/usr/share/keyrings/jitsi-keyring.gpg] https://download.jitsi.org stable/' | sudo tee /etc/apt/sources.list.d/jitsi-stable.list > /dev/null
```

```
sudo apt update
```

### 4. Install Nginx Web Server
Untuk melayani pertemuan Jisti melalui browser web, kami memerlukan server web – Anda dapat menggunakan Apache atau Nginx. Di sini kami memilih Nginx, file konfigurasi host yang diperlukan untuk Jisti akan secara otomatis dibuat saat menginstal Jisti pada langkah berikutnya. 
```
sudo apt install nginx-full
```

start enable nginx 

```
sudo systemctl enable --now nginx.service
```

### 5. Install Jitsi Server

```
sudo apt install jitsi-meet
```


saat muncul ini masukan host ubuntu yg sudah di set tadi. jitsi.erlanggaht.com
![alt text](https://linux.how2shout.com/wp-content/uploads/2022/05/Set-Hostname-for-Jitsi.png)

pilih yg generate blabla
![alt text](https://linux.how2shout.com/wp-content/uploads/2022/05/configure-Jisti-meet-web-config.png)

tunggu beberapa menit, penginstallan agak lumayan lama.

seteleh selesai

cek layanan jitsi, sudah jalan atau tidak: 

```
sudo systemctl status jitsi-videobridge2
```

untuk mencoba restart jitsi : 

```
sudo systemctl restart jitsi-videobridge2
```

### 6. Buka Izinkan Port

```
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 10000/udp
sudo ufw allow 22/tcp
sudo ufw allow 3478/udp
sudo ufw allow 5349/tcp
```
### 7.  Buat sertifikat Let's Encrypt (opsional) 

```
sudo apt install certbot
```

```
sudo sed -i 's/\.\/certbot-auto/certbot/g' /usr/share/jitsi-meet/scripts/install-letsencrypt-cert.sh
```

```
sudo ln -s /usr/bin/certbot /usr/sbin/certbot
```

```
sudo /usr/share/jitsi-meet/scripts/install-letsencrypt-cert.sh
```

### 8. Kunjungi Web Jitsi Meet 

jitsi.erlanggaht.com       

## Custom Jitsi Client

Clone repository
```
git clone https://github.com/jitsi/jitsi-meet
```

```
npm install
```

Untuk membangun aplikasi Jitsi Meet, ketik saja
```
make
```

noted : saran nodejs dan npm paling terbaru agar menghindari error 
saya pakai nodejs : v21
npm : v10

### Menghubungkan jitsi client ke server jitsi 

```
export WEBPACK_DEV_SERVER_PROXY_TARGET=https://jitsi.erlanggaht.com
make dev
```

## Authentication Jitsi Server

https://jitsi.github.io/handbook/docs/devops-guide/secure-domain

# Install Etherpad

```
sudo su
```

buka folder **/usr/share**

```
cd /usr/share
```

clone etherpad 

```
git clone --branch master https://github.com/ether/etherpad-lite.git etherpad
chown -R etherpad:etherpad etherpad
chmod -R 744 etherpad
cd etherpad
```

## Install mysql & konfigurasi ubuntu

```
sudo apt install mysql-server
```

jalankan mysql server: 

```
sudo systemctl start mysql.service
```

### konfigurasi authentication mysql 
```
sudo mysql

mysql > ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 
'buat_password_disini';
```

### buat database untuk etherpad nanti
```
CREATE DATABASE erlanggaht;
exit
```


## Konfigurasi etherpad

install pm2 

```
npm install pm2 -g
```

temukan dan konfigurasikan pengaturan ini setting.json: 

```
"dbType" : "mysql",
  "dbSettings" : {
    "user":     "root",
    "host":     "localhost",
    "port":     3306,
    "password": "masukan_password_disini",
    "database": "erlanggaht",
    "charset":  "utf8mb4"
  },
```

buat env dengan membuka **/etc/environment**, tambahkan 

```
NODE_ENV=production
```

edit run.sh di **bin/run.sh** 

```
sudo nano bin/run.sh
```

ganti node ke pm2 
```
exec **node** "$SCRIPTPATH/node_modules/ep_etherpad-lite/node/server.js" "$@"
````
menjadi 
```
exec **pm2** "$SCRIPTPATH/node_modules/ep_etherpad-lite/node/server.js" "$@"
```

### Nginx Konfigurasi

Konfigurasi host virtual Openjitsi nginx di **/etc/nginx/site-available/<your jitsi domain>** dan tambahkan pernyataan ini: 
```
# Etherpad Integration
    location ^~ /etherpad/ {
        proxy_pass http://localhost:9001/;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_buffering off;
        proxy_set_header ost $host;
    }
```
### konfigurasi jitsi
buka config.js aktif **/etc/jitsi/meet/<your domain>-config.js** dan tambahkan pernyataan ini atau cari dan uncomment: 

```
etherpad_base: 'https://<your domain>/etherpad/p/'
```


setelah selesai semua, untuk mejalankan nya ketikan perintah: 


```
bin/run.sh
```