# 🧩 Labb: SSH-åtkomst utan Lösenord till Kali (VirtualBox)

## 🎯 Mål

I denna övning lär du dig att skapa ett ssh keypair för att logga in utan lösenord över ssh.
Innan vi sätter igång behöver vi förbereda Nätverket genom att sätta upp Port Forwarding

## Förberedelser
Innan du startar Kali, konfigurera VirtualBox så att trafik från Windows kommer fram.

### A. VirtualBox Inställningar

1.  Stäng av din Kali VM. Gå till **Inställningar** $\rightarrow$ **Nätverk** (Adapter 1, ska vara **NAT**).
2.  Klicka på **Port Forwarding**. Lägg till denna enda regel:

| Fält | Värde |
| :--- | :--- |
| **Namn** | `SSH_Calle` |
| **Värdport** (Host Port) | **`2325`** |
| **Gästport** (Guest Port) | **`22`** |



### B. Kali Nätverksfil (Valfritt)

Om du vill ha statisk IP istället för DHCP (rekommenderas ofta i labbmiljö):

```bash
sudo nano /etc/network/interfaces
```

Se till att filen ser ut så här (förutsätter att du använder `eth0` i VirtualBox NAT):

```text
# The loopback network interface
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
        address 10.0.2.15
        netmask 255.255.255.0
        gateway 10.0.2.1
        dns-nameservers 10.0.2.1 8.8.8.8
```

---

## 2. Förbered Kali VM

Starta Kali och logga in.

### A. Skapa Användaren `calle`

```bash
sudo adduser calle
# Ange lösenord.
```

### B. Starta SSH-servern

Kontrollera att servern är igång:

```bash
# Installera om den saknas
sudo apt update
sudo apt install openssh-server

# Se till att tjänsten är aktiv
sudo systemctl enable ssh --now
```

---

## 3. SSH Key Pair: Windows $\rightarrow$ Kali

### A. Skapa Nycklarna på Windows

Öppna **Windows Terminal** eller **PowerShell**.

```bash
# Skapa det privata (id_ed25519) och publika (id_ed25519.pub) nyckelparet
ssh-keygen -t ed25519 -C "calle_kali_key"
```

> **Obs:** Spara filerna på standardplatsen (`C:\Users\DittAnvändarnamn\.ssh\`) och ange en **lösenfras**.

### B. Flytta Publika Nyckeln till Kali

Testa först anslutningen och flytta sedan filen.

1.  **Testa anslutning (med lösenord)**:
    ```bash
    ssh calle@localhost -p 2325
    # Logga ut direkt om det fungerar: exit
    ```

2.  **Kopiera den publika nyckeln** (Ange `calle`s lösenord en sista gång!):
    ```bash
    # OBS: Byt ut id_ed25519.pub om ditt filnamn är annorlunda.
    scp -P 2325 C:\Users\DittAnvändarnamn\.ssh\id_ed25519.pub calle@localhost:/home/calle/id_ed25519.pub
    ```

### C. Slutkonfiguration på Kali

Logga in **med lösenord** igen för att städa upp och konfigurera.

```bash
ssh calle@localhost -p 2325
```

**Inne på Kali:**

```bash
# 1. Skapa SSH-mappen för 'calle'
mkdir -p /home/calle/.ssh

# 2. Flytta filen och byt namn till authorized_keys
mv /home/calle/id_ed25519.pub /home/calle/.ssh/authorized_keys

# 3. Sätt korrekta behörigheter (KRITISKT!)
chmod 700 /home/calle/.ssh
chmod 600 /home/calle/.ssh/authorized_keys

# Säkerställ att 'calle' äger filerna
chown -R calle:calle /home/calle/.ssh

# Logga ut
exit
```

---

## 4. Testa Lösenordsfri Inloggning

Från **Windows Terminal**:

```bash
ssh calle@localhost -p 2325
```

➡️ Du ska nu logga in direkt (du anger bara lösenfrasen för nyckeln om du skapade en sådan). Klart!

---

# 🔒 Extra Steg: Stäng Av Lösenordsinloggning
> (för bättre säkerhet)

När du vet att nyckel-inloggningen fungerar perfekt, bör du stänga av möjligheten att logga in med lösenord via SSH. Detta ökar säkerheten dramatiskt.

### A. Redigera SSHD-konfigurationen på Kali

Logga in med din **nyckel**-inloggning först (Steg 4).

```bash
sudo nano /etc/ssh/sshd_config
```

1.  Hitta raden som säger `PasswordAuthentication yes` eller som är kommenterad (börjar med `#`).
2.  Ändra den till:

    ```text
    PasswordAuthentication no
    ```

3.  Spara och stäng filen (`Ctrl+O`, `Enter`, `Ctrl+X`).

### B. Starta om SSH-tjänsten

```bash
sudo systemctl restart ssh
```

> **Test:** Försök nu logga in med lösenord. Du ska få ett felmeddelande, men din nyckel-inloggning ska fortfarande fungera!

---
**🎓 Klart!** Du har nu en säker SSH-förbindelse till din Kali VM.
