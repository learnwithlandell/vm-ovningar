# 🧩 Labb: Skapa ett internt nätverk i VirtualBox

## 🎯 Mål

I denna övning ska du skapa ett **internt nätverk** i VirtualBox och koppla samman två virtuella maskiner — `serverA` och `serverB`.
Du kommer också att konfigurera **statiska IP-adresser** och testa kommunikationen med `ping`.

---

## 🔧 Förberedelser

* Oracle VirtualBox är installerat.
* Två virtuella maskiner finns färdiga: **serverA** och **serverB**.
* Båda kör **Ubuntu Server 22.04 LTS** (redan installerat men ej nätverkskonfigurerat).

---

## 🪄 Steg 1 – Skapa ett internt nätverk

1. Öppna **VirtualBox Manager**.
2. Välj din första VM (t.ex. `serverA`) → **Settings** → **Network**.
3. Under **Attached to**, välj **Internal Network**.
4. Döp nätverket till t.ex. `privnet`.
5. Upprepa samma inställning för `serverB`.
   (Båda ska ha exakt samma nätverksnamn: `privnet`.)

---

## ⚙️ Steg 2 – Starta båda maskinerna

Starta `serverA` och `serverB` i VirtualBox.

---

## 🌐 Steg 3 – Konfigurera statisk IP i Ubuntu

> Utför dessa steg på båda servrarna – med olika IP-adresser.

### 1️⃣ Öppna nätverkskonfiguration

```
sudo nano /etc/netplan/01-netcfg.yaml
```

### 2️⃣ Redigera filen (exempel för `serverA`)

```
network:
version: 2
renderer: networkd
ethernets:
enp0s3:
dhcp4: no
addresses:
- 192.168.1.2/24
```

För `serverB`:
```
network:
version: 2
renderer: networkd
ethernets:
enp0s3:
dhcp4: no
addresses:
- 192.168.1.3/24
```

### 3️⃣ Spara och tillämpa inställningarna

```
sudo netplan apply
```

---

## 🧪 Steg 4 – Testa kommunikationen

Från `serverA`:
```
ping 192.168.1.3
```

Från `serverB`:
```
ping 192.168.1.2
```

✅ Du ska få svar (Reply from…).

---

## 💡 Extra utmaning

* Lägg till en tredje maskin i samma interna nätverk.
* Testa `hostname -I` för att se IP-adressen.
* Kontrollera anslutningen med `ip a`.

---

**🎓 Klart!**
Du har nu skapat ett privat nätverk i VirtualBox och fått två servrar att kommunicera via statiska IP-adresser.
