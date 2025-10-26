# 🔒 Övning: Installera och testa en OpenVPN-server på Debian

I denna övning lär du dig att installera, konfigurera och testa en **OpenVPN-server** på Debian.
Du kommer att skapa certifikat, konfigurera brandväggen och sedan ansluta från din dator (Windows/Mac) till VPN-servern i VirtualBox – för att nå andra system i samma interna nätverk.

---

## 🎯 Mål

Efter övningen ska du kunna:

* Installera och konfigurera en OpenVPN-server på Debian
* Förstå hur VirtualBox-nätverken NAT Network och Internal Network fungerar
* Skapa certifikat och nycklar med Easy-RSA
* Ansluta från din host till VPN-servern
* Nå interna system via VPN-tunneln

---

## 🧰 Förutsättningar

Du har importerat tre Debian-maskiner i VirtualBox, du hittar dem [här](https://github.com/learnwithlandell/vm-ovningar/blob/main/ovning-5-vbox-import-appliances.md). 
Vi kommer även använda din host-dator (Windows/Mac/Linux) i den här övningen. 

| System         | Roll                     | IP-adress | Nätverksinställning (initialt)            | Kommentar                            |
| -------------- | ------------------------ | --------- | ----------------------------------------- | ------------------------------------ |
| **system1**    | Intern server/klient     | 10.0.2.7  | **Adapter 1: NAT Network (MinPlattform)** | Internetåtkomst för apt-installation |
| **system2**    | Intern server/klient     | 10.0.2.8  | **Adapter 1: NAT Network (MinPlattform)** | Internetåtkomst för apt-installation |
| **system3**    | **VPN-server (OpenVPN)** | 10.0.2.15 | **Adapter 1: NAT Network (MinPlattform)** | Internetåtkomst för apt-installation |
| **Host-dator** | VPN-klient (Windows/Mac) | –         | Internet + OpenVPN GUI                    | Ansluter till system3 via VPN tunnel |

---

## 🪜 Steg 1 – Installera nödvändiga paket på system3

Logga in på **system3 (10.0.2.15)** och öppna terminalen:

```
sudo -i
apt update && apt upgrade -y
apt install ssh -y
apt install openvpn easy-rsa -y
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

Redigera fält (ta bort # om de finns):

```
set_var EASYRSA_DN "org"
set_var EASYRSA_REQ_COUNTRY "SE"
set_var EASYRSA_REQ_PROVINCE "ST"
set_var EASYRSA_REQ_CITY "Stockholm"
set_var EASYRSA_REQ_ORG "Minskola"
set_var EASYRSA_REQ_EMAIL "[hej@minskola.se](mailto:hej@minskola.se)"
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
sudo cp pki/ca.crt /etc/openvpn/
sudo cp pki/issued/server.crt /etc/openvpn/
sudo cp pki/private/server.key /etc/openvpn/
sudo cp pki/dh.pem /etc/openvpn/
```

Kopiera även klientfiler till hemkatalogen:

```
sudo cp pki/issued/client1.crt ~/
sudo cp pki/private/client1.key ~/
sudo cp pki/ca.crt ~/
```

---

## ⚙️ Steg 6 – Skapa och redigera serverkonfiguration

```
sudo cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf /etc/openvpn/
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
ifconfig-pool-persist /var/log/openvpn/ipp.txt
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

## 🔓 Steg 7 – Öppna brandväggen (UFW)

```
sudo apt install ufw -y
sudo ufw allow 22/tcp
sudo ufw allow 1194/udp
sudo ufw enable
sudo ufw status verbose
```

---

## 🚀 Steg 8 – Starta och aktivera OpenVPN-servern

```
sudo systemctl start openvpn@server
sudo systemctl enable openvpn@server
sudo systemctl status openvpn@server
```

---

## 🔄 **Steg 9 – Byt nätverksläge till Internal Network**

När installationen och uppdateringar är klara:

1. Stäng av **system1**, **system2** och **system3**.
2. I VirtualBox, gå till **Settings → Network → Adapter 1**
3. Ändra från **NAT Network (MinPlattform)** till **Internal Network**
4. Ange **Name:** `vpnlab`
5. Starta om alla tre maskiner.

### ✅ Resultat:

* Maskinerna ligger nu i samma isolerade nätverk.
* IP-adresserna (10.0.2.7, .8, .15) finns kvar tack vare netplan.
* Ingen internetåtkomst – all trafik stannar i labbnätet.

---

## 💻 Steg 10 – Skapa klientkonfiguration (på host-dator)

1. Skapa `C:\vpn` (Windows) eller `~/vpn` (Mac).
2. Kopiera över från **system3**:

   * `ca.crt`
   * `client1.crt`
   * `client1.key`
3. Skapa `client.ovpn`:

```
client
dev tun
proto udp
remote 10.0.2.15 1194
resolv-retry infinite
nobind
persist-key
persist-tun
ca C:\vpn\ca.crt
cert C:\vpn\client1.crt
key C:\vpn\client1.key
remote-cert-tls server
cipher AES-256-GCM
auth SHA256
verb 3
```

---

## 🧩 Steg 11 – Installera OpenVPN GUI (host-dator)

1. Ladda ner OpenVPN GUI: [https://openvpn.net/community-downloads/](https://openvpn.net/community-downloads/)
2. Installera och starta som administratör.
3. Högerklicka på ikonen → **Import file** → välj `client.ovpn`
4. Högerklicka igen → **Connect**

✅ “Connected to server” betyder att tunneln fungerar.

---

## 🌍 Steg 12 – Testa åtkomst till interna system

På **system3** (VPN-server):

```
sudo nano /etc/openvpn/server.conf
```

Lägg till:
```
push "route 10.0.2.0 255.255.255.0"
```

Starta om:
```
sudo systemctl restart openvpn@server
```

På **host-datorn** (ansluten via VPN):
```
ping 10.0.2.7
ping 10.0.2.8
```

Om du får svar → VPN-routing OK 🎉

---

## 🧠 Felsökning

* **TLS Error:** kontrollera certifikat och portar.
* **Ingen ping:** verifiera att alla VMs ligger i `vpnlab`.
* **UFW blockerar:** `sudo ufw status verbose`.

---

## ✅ Resultat

* Du kan ansluta till VPN-servern på 10.0.2.15
* Du når system1 och system2 via VPN-tunneln
* Alla IP-adresser och netplan-konfigurationer återanvänds
* Du förstår hur man går från NAT Network → Internal Network på ett säkert sätt

---

✍️ *Senast uppdaterad: Oktober 2025*
