# 🧩 Övning: Aktivera Multi-Factor Authentication (MFA) i OpenVPN

I denna övning lär du dig att aktivera **tvåfaktorsautentisering (MFA)** i din befintliga OpenVPN-lösning med hjälp av **Google Authenticator**.
Varje användare får en personlig **TOTP-kod** (Time-based One-Time Password) som genereras i en app som Google Authenticator, Microsoft Authenticator eller Authy.

---

## 🎯 Mål

Efter övningen ska du kunna:

* Installera och konfigurera Google Authenticator för MFA
* Förstå hur PAM-autentisering används i OpenVPN
* Starta OpenVPN manuellt i förgrundsläge
* Testa att MFA krävs vid VPN-anslutning

---

## 🧰 Förutsättningar

Du har redan genomfört övningen:
➡️ *“Installera och testa en OpenVPN-server på Debian 13”*

Använd samma miljö:

| System         | Roll           | IP                   | Kommentar                  |
| -------------- | -------------- | -------------------- | -------------------------- |
| **system3**    | OpenVPN-server | 10.0.2.15 / 10.0.3.1 | Här aktiveras MFA          |
| **Host-dator** | VPN-klient     | 127.0.0.1            | Ansluter via GUI eller CLI |

---

## 🪜 Steg 1 – Installera Google Authenticator på system3

Logga in som root på **system3**:

```
su -
apt update
apt install libpam-google-authenticator -y
```

---

## 🔑 Steg 2 – Skapa hemlig TOTP-nyckel för användaren `vboxuser`

Kör kommandot **som `vboxuser`**:

```
sudo -u vboxuser google-authenticator
```

Svara **y** på alla frågor:

1. *Time-based tokens (y/n)?* → `y`
2. *Update “~/.google_authenticator” file?* → `y`
3. *Disallow multiple uses of the same token?* → `y`
4. *Enable rate-limiting?* → `y`

Du får nu en **QR-kod** och en **hemlig nyckel**.
📱 Skanna koden med Google Authenticator eller Microsoft Authenticator-appen.

---

## ⚙️ Steg 3 – Aktivera PAM-inloggning i OpenVPN

Öppna serverkonfigurationen:

```
nano /etc/openvpn/server.conf
```

Lägg till eller uppdatera raderna nedanför `auth SHA256`:

```
plugin /usr/lib/x86_64-linux-gnu/openvpn/plugins/openvpn-plugin-auth-pam.so openvpn
verify-client-cert none
username-as-common-name
```

Spara (**Ctrl+O**) och avsluta (**Ctrl+X**).

---

## 🔐 Steg 4 – Skapa PAM-konfiguration för OpenVPN

Skapa filen `/etc/pam.d/openvpn`:

```
nano /etc/pam.d/openvpn
```

Innehåll:

```

# PAM-konfiguration för OpenVPN med Google Authenticator

auth required pam_google_authenticator.so nullok
account required pam_permit.so
```

Spara (**Ctrl+O**) och avsluta (**Ctrl+X**).

---

## 🚫 Steg 5 – Stäng av systemd-tjänsten

Eftersom vi vill köra OpenVPN manuellt och se vad som händer i terminalen, stoppa daemonen helt:

```
systemctl stop openvpn@server
systemctl disable openvpn@server
```

---

## 🚀 Steg 6 – Starta OpenVPN manuellt i förgrund

Kör följande kommando på **system3**:

```
sudo openvpn --config /etc/openvpn/server.conf
```

OpenVPN startar nu direkt i terminalen och du ser loggutdata i realtid.
Lämna detta fönster öppet medan du ansluter från din klient.

---

## 💻 Steg 7 – Klientkonfiguration

På din hostdator, öppna `client.ovpn` och se till att följande rad finns:

```
auth-user-pass
```

Exempel (Windows-format):

```
client
dev tun
proto udp
remote 127.0.0.1 1194
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
auth-user-pass
verb 3
```

---

## 🔐 Steg 8 – Testa MFA-inloggning

1. Starta **OpenVPN GUI** på hosten
2. Högerklicka → **Connect**
3. Ange:

   ```
   Username: vboxuser
   Password: <sexsiffrig kod från appen>
   ```
4. Titta på terminalen i **system3** – du ska se:

   ```
   AUTH-PAM: BACKGROUND: user 'vboxuser' authentication succeeded
   ```
5. Klienten visar “Connected” → tunneln är uppe 🎉

---

## 🧠 Felsökning

| Problem                                      | Lösning                                                                                         |
| -------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| **Ingen kodruta visas i klienten**           | Kontrollera att `auth-user-pass` finns i `.ovpn`                                                |
| **`Failed to read ~/.google_authenticator`** | Kör `sudo -u vboxuser google-authenticator` på nytt                                             |
| **`AUTH-FAILED` i logg**                     | Fel TOTP-kod – vänta till nästa 30-sekunderscykel                                               |
| **Plugin hittas inte**                       | Kontrollera sökvägen:<br>`/usr/lib/x86_64-linux-gnu/openvpn/plugins/openvpn-plugin-auth-pam.so` |
| **Servern körs inte**                        | Kör `sudo openvpn --config /etc/openvpn/server.conf` manuellt igen                              |

Visa systemloggar (om du testat med systemd tidigare):

```
sudo journalctl -u openvpn@server -e
```

---

## ✅ Resultat

* OpenVPN startas manuellt i terminalen
* MFA via Google Authenticator fungerar korrekt
* Inloggning kräver både certifikat **och** TOTP-kod
* Du förstår hur PAM, pluginet och OpenVPN samverkar utan systemd-daemon

---

✍️ *Senast uppdaterad: Oktober 2025*
