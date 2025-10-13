# 🧱 Övning: Skydda SSH med Fail2ban

## 🎯 Mål

Efter övningen ska du kunna:

* Installera och aktivera **Fail2ban**
* Anpassa inställningar för att blockera IP-adresser efter upprepade felinloggningar
* Visa loggar som bekräftar att blockering sker
* Testa att blockeringen hävs efter 2 minuter

---

Importera system1 och system2 i VirtualBox, du hittar dem [här](https://github.com/learnwithlandell/vm-ovningar/blob/main/ovning-5-vbox-import-appliances.md)

## 🧩 Miljö

| Roll   | System    | IP-adress  | OS      |
| ------ | --------- | ---------- | ---------
| Server | system1   | `10.0.2.7` | Debian 13
| Klient | system2   | `10.0.2.8` | Debian 13

SSH är redan installerat och aktiverat.

---

## 🔧 Del 1: Installera Fail2ban på servern

**På Debian 13 (10.0.2.7):**

```
su root
sudo apt update
sudo apt install fail2ban -y
```

Bekräfta att tjänsten körs:
```
sudo systemctl status fail2ban
```

Du ska se något i stil med:
```
Active: active (running)
```

---

## 📁 Del 2: Skapa en egen konfiguration

Gör en kopia av standardfilen:
```
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

Öppna din lokala konfiguration:
```
sudo nano /etc/fail2ban/jail.local
```

Leta upp sektionen `[DEFAULT]` och redigera så här:
```
#banaction = iptables-multiport
banaction = nftables-multiport
```

Leta upp sektionen `[sshd]` och redigera så här:

```
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
findtime = 300
bantime = 120
ignoreip = 127.0.0.1/8
backend = %(sshd_backend)s
```

**Förklaring:**

* **enabled** – aktiverar skyddet
* **maxretry 3** – tre försök tillåts
* **findtime 300** – tiden (i sekunder) inom vilken försöken räknas
* **bantime 120** – blockeringstid 2 minuter
* **ignoreip** – IP-adresser som aldrig blockeras

Spara och stäng (`Ctrl+O`, `Enter`, `Ctrl+X`).

---

## 🔄 Del 3: Starta om och verifiera

```
sudo systemctl restart fail2ban
sudo fail2ban-client status sshd
```

Exempel på utdata:
```
Status for the jail: sshd
|- Filter
|  |- Currently failed: 0
|  |- Total failed: 0
|  `- File list: /var/log/auth.log
`- Actions
|- Currently banned: 0
|- Total banned: 0
`- Banned IP list:
```

---

## 💻 Del 4: Testa från klienten (10.0.2.8)

Från klienten, försök logga in med fel lösenord 3 gånger:

```
ssh student@10.0.2.7
```

Skriv fel lösenord tre gånger i rad.

---

## 🧠 Del 5: Kontrollera blockeringen på servern

På Debian 13 (server):

Visa loggen:
```
sudo tail -n 20 /var/log/fail2ban.log
```

Exempel på utdata:
```
INFO    [sshd] Found 10.0.2.8 - 2025-10-12 20:18:45
INFO    [sshd] Found 10.0.2.8 - 2025-10-12 20:18:50
INFO    [sshd] Found 10.0.2.8 - 2025-10-12 20:18:55
INFO    [sshd] Ban 10.0.2.8
```

Visa status:
```
sudo fail2ban-client status sshd
```

Utdata ska visa:
```
Banned IP list: 10.0.2.8
```

---

## 🧱 Del 6: Testa blockeringen

Från klienten (10.0.2.8), försök igen:

```
ssh student@10.0.2.7
```

Du bör få:
```
ssh: connect to host 10.0.2.7 port 22: Connection refused
```

---

## ⏱️ Del 7: Vänta 2 minuter och testa igen

Efter 2 minuter:
```
ssh student@10.0.2.7
```

Nu ska du kunna logga in normalt igen.

---

## 📜 Del 8: Visa loggar som bevis

På servern:
```
sudo grep "Ban|Unban" /var/log/fail2ban.log
```

Exempelutdata:
```
INFO    [sshd] Ban 10.0.2.8
INFO    [sshd] Unban 10.0.2.8
```

Det visar att IP:et först bannats, sedan släppts efter 2 minuter.

---

## 🧩 Bonus: Se realtidsloggning

Vill du se allt live:

```
sudo tail -f /var/log/fail2ban.log
```

Låt fönstret vara öppet medan du gör nya inloggningsförsök från klienten.

---

## ✅ Slutresultat

När du är klar ska du kunna visa:

1. Din `/etc/fail2ban/jail.local` innehåller rätt inställningar
2. Utdrag ur `fail2ban.log` som visar `Ban` och `Unban`
3. Att du blir utelåst efter 3 försök, och kan logga in igen efter 2 minuter
   ```
