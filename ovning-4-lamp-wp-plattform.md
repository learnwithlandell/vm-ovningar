# 🐧 Installera Debian och WordPress i VirtualBox

Denna guide hjälper dig att:

* Installera senaste **Debian** i VirtualBox
* Konfigurera nätverk med **Netplan**
* Sätta upp **Apache, PHP och MariaDB**
* Installera **WordPress** lokalt

---

## 📥 1. Ladda ner nödvändiga filer

**Debian (netinst ISO)**
👉 [https://www.debian.org/CD/netinst/](https://www.debian.org/CD/netinst/)

**VirtualBox**
👉 [https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads)

---

## 💻 2. Grundinstallation i Debian

Efter att du installerat Debian (standardinställningar räcker):

```
su -
apt update && apt upgrade -y
apt install netplan.io -y
netplan status
shutdown now
```

---

## 🌐 3. Konfigurera nätverk

### 🖥️ System 1 (10.0.2.7)

```
nano /etc/netplan/01-netcfg.yaml
```

```

# This is the network config written by debian-installer.

# For more information, see netplan(5).

network:
version: 2
renderer: NetworkManager
ethernets:
enp0s3:
dhcp4: no
addresses:
- 10.0.2.7/24
routes:
- to: default
via: 10.0.2.1
nameservers:
addresses:
- 8.8.8.8
```

```
netplan apply
```

### 🖥️ System 2 (10.0.2.8)

```
nano /etc/netplan/01-netcfg.yaml
```

```
network:
version: 2
renderer: NetworkManager
ethernets:
enp0s3:
dhcp4: no
addresses:
- 10.0.2.8/24
routes:
- to: default
via: 10.0.2.1
nameservers:
addresses:
- 8.8.8.8
```

```
netplan apply
```

### 🖥️ System 3 (10.0.2.15)

```
nano /etc/netplan/01-netcfg.yaml
```

```
network:
version: 2
renderer: NetworkManager
ethernets:
enp0s3:
dhcp4: no
addresses:
- 10.0.2.15/24
routes:
- to: default
via: 10.0.2.1
nameservers:
addresses:
- 8.8.8.8
```

```
netplan apply
```

---

## 🧩 4. (Frivilligt) SSH från host

Om du har port forwarding konfigurerat:

```
ssh vboxuser@localhost -p 2323
```

---

## 🛠️ 5. Installera WordPress och databasen (System 1 – 10.0.2.17)

```
su -
apt install php apache2 libapache2-mod-php php-mysql mariadb-server -y
mariadb-secure-installation
```

Starta MySQL och skapa databas + användare:

```
mysql
```

```
CREATE DATABASE wordpress;
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

Ladda ner och installera WordPress:

```
cd /var/www/html
wget [https://wordpress.org/latest.zip](https://wordpress.org/latest.zip)
unzip latest.zip
rm latest.zip index.html
```

---

## 🌍 6. Konfigurera WordPress

Öppna Firefox på samma system:

[http://localhost/wordpress](http://localhost/wordpress)

👉 Följ installationsguiden.

---

## ⚙️ 7. Uppdatera `wp-config.php`

```
nano /var/www/html/wordpress/wp-config.php
```

Lägg till (justera IP efter ditt nätverk):

```
define( 'WP_HOME', '[http://10.0.2.7/wordpress](http://10.0.2.7/wordpress)' );
define( 'WP_SITEURL', '[http://10.0.2.7/wordpress](http://10.0.2.7/wordpress)' );
```

---

## ✅ Klart!

Du har nu en fungerande **WordPress-installation på Debian** i VirtualBox.
Testa att öppna `http://10.0.2.7/wordpress` i din webbläsare.

---

### 💡 Tips

* Ta en **snapshot** i VirtualBox innan du börjar experimentera.
* Om du inte får DNS att fungera, testa byta `nameservers` till `1.1.1.1`.
* Vill du testa flera maskiner? Använd **“Eget NAT-nätverk”** i VirtualBox.

---

✍️ *Senast uppdaterad: Oktober 2025*
