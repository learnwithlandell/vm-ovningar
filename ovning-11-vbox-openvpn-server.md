# 🔒 Övning: Installera och testa en OpenVPN-server på Debian 13

I denna övning lär du dig att installera, konfigurera och testa en **OpenVPN-server** på Debian.
Du kommer att skapa certifikat, sätta upp nätverket med `netplan`, och sedan ansluta från din dator (Windows/Mac) till VPN-servern i VirtualBox – för att nå andra system i samma interna nätverk.

---

## 🎯 Mål

Efter övningen ska du kunna:

* Installera och konfigurera en OpenVPN-server på Debian
* Förstå hur VirtualBox-näten **NAT Network** och **Internal Network** samverkar
* Skapa certifikat och nycklar med Easy-RSA
* Ansluta från din host till VPN-servern
* Nå interna system via VPN-tunneln

---

## 🧰 Förutsättningar

Du har importerat tre Debian-maskiner i VirtualBox, du hittar dem
[här](https://github.com/learnwithlandell/vm-ovningar/blob/main/ovning-5-vbox-import-appliances.md).
Vi använder även din host-dator (Windows/Mac/Linux) i denna övning.

**Nätverksupplägg (efter Steg 10 i guiden nedan):**

| System         | Adapter 1                     | Adapter 2                    | IP-adress(er)                                | Kommentar           |
| -------------- | ----------------------------- | ---------------------------- | -------------------------------------------- | ------------------- |
| **system1**    | **Disabled**                  | **Internal Network: vpnlab** | 10.0.3.10                                    | Endast internt      |
| **system2**    | **Disabled**                  | **Internal Network: vpnlab** | 10.0.3.20                                    | Endast internt      |
| **system3**    | **NAT Network: MinPlattform** | **Internal Network: vpnlab** | 10.0.2.15 (NAT), 10.0.3.1 (vpnlab)           | VPN-server & router |
| **Host-dator** | –                             | –                            | 127.0.0.1:1194 → (port forward till system3) | OpenVPN-klient      |

> Varför så?
>
> * **system3** behöver NAT för att hosten ska kunna nå port **1194/UDP** via **Port Forwarding**.
> * **system1 & system2** är rena interna noder (ingen internetåtkomst), därför **Adapter 1 = Disabled** på dem.
> * Alla tre har **Adapter 2 = Internal Network: vpnlab** för det interna 10.0.3.0/24-nätet.

---

## 🪜 Steg 1 – Installera nödvändiga paket på system3

Logga in på **system3 (10.0.2.15)** och kör:

```
su -
apt update && apt upgrade -y
apt install openvpn easy-rsa -y
systemctl disable ufw --now
```

---

## 🧩 Steg 2 – Förbered Easy-RSA

```
mkdir ~/openvpn-ca
cp -r /usr/share/easy-rsa/* ~/openvpn-ca
cd ~/openvpn-ca
cp vars.example vars
nano vars
```

Redigera fälten (ta bort # om de finns):

```
set_var EASYRSA_DN "org"
set_var EASYRSA_REQ_COUNTRY "SE"
set_var EASYRSA_REQ_PROVINCE "ST"
set_var EASYRSA_REQ_CITY "Stockholm"
set_var EASYRSA_REQ_ORG "Minskola"
set_var EASYRSA_REQ_EMAIL "hej@minskola.se"
set_var EASYRSA_REQ_OU "Kontoret"
```

Spara (**Ctrl+O**) och avsluta (**Ctrl+X**).

---

## 🧱 Steg 3 – Skapa certifikat och nycklar

```
./easyrsa init-pki
./easyrsa build-ca
./easyrsa gen-req server nopass
./easyrsa sign-req server server
./easyrsa gen-req client1 nopass
./easyrsa sign-req client client1
```

---

## 🔑 Steg 4 – Generera Diffie-Hellman och TLS-nyckel

```
./easyrsa gen-dh
openvpn --genkey tls-auth ta.key
```

---

## 📂 Steg 5 – Kopiera filer till OpenVPN-mappen

```
cp pki/ca.crt /etc/openvpn/
cp pki/issued/server.crt /etc/openvpn/
cp pki/private/server.key /etc/openvpn/
cp pki/dh.pem /etc/openvpn/
cp ta.key /etc/openvpn/
```

Kopiera även klientfiler till `/home/vboxuser/`:

```
cp pki/issued/client1.crt /home/vboxuser/
cp pki/private/client1.key /home/vboxuser/
cp pki/ca.crt /home/vboxuser/
chown vboxuser:vboxuser /home/vboxuser/*
```

---

## ⚙️ Steg 6 – Skapa och redigera serverkonfiguration

```
# skippa nästa rad om du vill klistra in hela kodblocket
# cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf /etc/openvpn/

# öppna eller skapa openvpn server konfigurationsfil
nano /etc/openvpn/server.conf
```

Kontrollera/ändra:

```
port 1194
proto udp
dev tun
ca /etc/openvpn/ca.crt
cert /etc/openvpn/server.crt
key /etc/openvpn/server.key
dh /etc/openvpn/dh.pem

server 192.168.3.0 255.255.255.0
topology subnet
push "route 10.0.3.0 255.255.255.0"

keepalive 10 120
cipher AES-256-GCM
persist-key
persist-tun
status /var/log/openvpn/openvpn-status.log
verb 6
auth SHA256
explicit-exit-notify 1
#tls-auth ta.key 0
```

---

## 💻 Steg 7 – Skapa klientkonfiguration (host-dator)

1. Skapa `C:\vpn` (Windows) eller `~/vpn` (Mac).
2. Kopiera från **system3**: `ca.crt`, `client1.crt`, `client1.key`
```
mkdir c:\vpn
scp -P 2325 vboxuser@localhost:/home/vboxuser/*.* c:\vpn
```
3. Skapa `client.ovpn`:

```
client
dev tun
proto udp
remote 127.0.0.1 1194
resolv-retry infinite
nobind
persist-key
persist-tun
ca C:\\vpn\\ca.crt
cert C:\\vpn\\client1.crt
key C:\\vpn\\client1.key
remote-cert-tls server
cipher AES-256-GCM
auth SHA256
verb 3
```

---

## 🧩 Steg 8 – Installera OpenVPN GUI (host-dator)

1. Ladda ner [OpenVPN Client](https://openvpn.net/client/)
2. Installera och starta som administratör
3. Högerklicka på ikonen → **Import file** → välj `client.ovpn`
4. Högerklicka igen → **Connect**

✅ “Connected to server” betyder att tunneln är uppe.

---

## 🚀 Steg 9 – Starta och aktivera OpenVPN-servern

```
# på system3
systemctl start openvpn@server
systemctl enable openvpn@server
systemctl status openvpn@server
```

---

## 🌐 Steg 10 – Netplan-konfigurationer

### 🛡 system3 (NAT + internt)

```
nano /etc/netplan/01-netcfg.yaml
```

```
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    enp0s8:
      dhcp4: no
      addresses:
        - 10.0.3.1/24
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

### 🖥 system1 (endast internt)

```
nano /etc/netplan/01-netcfg.yaml
```

```
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    enp0s8:
      dhcp4: no
      addresses:
        - 10.0.3.10/24
      routes:
        - to: default
          via: 10.0.3.1
      nameservers:
        addresses:
          - 8.8.8.8
```

```
netplan apply
ping 10.0.3.1
```

### 🖥 system2 (endast internt)

```
nano /etc/netplan/01-netcfg.yaml
```

```
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    enp0s8:
      dhcp4: no
      addresses:
        - 10.0.3.20/24
      routes:
        - to: default
          via: 10.0.3.1
      nameservers:
        addresses:
          - 8.8.8.8
```

```
netplan apply
ping 10.0.3.1
```

---

## 🧩 Steg 11 – IP-forwarding och rp_filter (Debian 13-sättet)

Skapa fil på **system3**:

```
nano /etc/sysctl.d/99-forwarding.conf
```
Innehåll:

```
net.ipv4.ip_forward=1
net.ipv4.conf.all.rp_filter=0
net.ipv4.conf.default.rp_filter=0
net.ipv4.conf.tun0.rp_filter=0
net.ipv4.conf.enp0s8.rp_filter=0
```
Ladda om:

```
sysctl --system
```

---

## 🔄 Steg 12 – Ställ in VirtualBox-nät (adapterordning)

Gör så här i **VirtualBox**:

* **system1**

  * **Adapter 1:** Disabled
  * **Adapter 2:** Internal Network → **Name:** `vpnlab`

* **system2**

  * **Adapter 1:** Disabled
  * **Adapter 2:** Internal Network → **Name:** `vpnlab`

* **system3**

  * **Adapter 1:** **NAT Network** → **Name:** `MinPlattform`

    * **Advanced → Port Forwarding**:

      | Name    | Protocol | Host IP   | Host Port | Guest IP  | Guest Port |
      | ------- | -------- | --------- | --------- | --------- | ---------- |
      | openvpn | UDP      | 127.0.0.1 | 1194      | 10.0.2.15 | 1194       |
  * **Adapter 2:** Internal Network → **Name:** `vpnlab`

Starta alla tre maskiner.

---

## 🌍 Steg 13 – Testa åtkomst till interna system

Säkerställ att servern redan har raden i **/etc/openvpn/server.conf** (du lade in den i Steg 6):

```
push "route 10.0.3.0 255.255.255.0"
```
Starta om OpenVPN på **system3** om du ändrat något:

```
systemctl restart openvpn@server
```

Testa från **hosten** (när VPN är anslutet):

```
ping 10.0.3.1
ping 10.0.3.10
ping 10.0.3.20
```

Om du får svar från alla → VPN & routing OK 🎉

---

## 🧠 Felsökning (kort)

* `sysctl net.ipv4.ip_forward` ska vara `= 1` på system3.
* `ip route show` på system3 ska visa:
  `10.0.3.0/24 dev enp0s8` och `192.168.3.0/24 dev tun0`.
* `route print` (Windows) ska innehålla route till `10.0.3.0/24` via `192.168.3.1`.
* Testa med `tcpdump` på system3:
  `tcpdump -ni tun0 icmp` och `tcpdump -ni enp0s8 icmp` medan du pingar 10.0.3.10 från hosten.

---

## ✅ Resultat

* **system1 & system2** har **Adapter 1 avstängd**, kör endast **Internal Network (vpnlab)** på Adapter 2.
* **system3** har **NAT Network (MinPlattform)** på Adapter 1 + **Internal Network (vpnlab)** på Adapter 2.
* Host ansluter via **127.0.0.1:1194** (port forward) och når **10.0.3.10** och **10.0.3.20** via VPN-tunneln.

---

✍️ *Senast uppdaterad: Oktober 2025*
