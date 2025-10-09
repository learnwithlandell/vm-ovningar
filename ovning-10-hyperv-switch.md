# 🧩 Labb: Skapa ett privat nätverk i Hyper-V

## 🎯 Mål

I denna övning ska du skapa ett **privat virtuellt nätverk** i Hyper-V och koppla samman två virtuella maskiner — `serverA` och `serverB`.
Du kommer också att konfigurera **statisk IP-adress** på båda servrarna och testa kommunikationen med `ping`.

---

## 🔧 Förberedelser

* Hyper-V är redan installerat.
* Två virtuella maskiner finns färdiga: **serverA** och **serverB**.
* Båda kör **Ubuntu Server 22.04 LTS** (redan installerat men ej nätverkskonfigurerat).

---

## 🪄 Steg 1 – Skapa en ny virtuell switch

1. Öppna **Hyper-V Manager**.
2. Klicka på **Virtual Switch Manager** i högermenyn.
3. Välj **New virtual network switch → Private**.
4. Namnge switchen `Ny Virtuell Switch`.
5. Skriv en kort anteckning, t.ex. `Ny privat switch`.
6. Klicka **Apply** och sedan **OK**.

📸 Exempel:

![Skapa virtuell switch](./94a41a04-6b21-4bb7-8e13-7d6336198c84.png)

---

## ⚙️ Steg 2 – Anslut båda servrar till switchen

1. Högerklicka på `serverA` → **Settings**.
2. Under **Network Adapter**, välj **Ny Virtuell Switch** i listan.
3. Upprepa samma steg för `serverB`.
4. Klicka **OK** för att spara.

📸 Exempel:

![Koppla VM till switch](./f1cfb61a-2c3c-4af7-8004-f1d6dac93b61.png)

---

## 🌐 Steg 3 – Konfigurera statisk IP i Ubuntu

> Utför dessa steg på **båda servrarna**, men med olika IP-adresser.

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
eth0:
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
eth0:
dhcp4: no
addresses:
- 192.168.1.3/24
```

### 3️⃣ Spara och applicera

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

✅ Om allt är rätt konfigurerat ska du få svar (Reply from…).

---

## 💡 Extra utmaning

* Lägg till en tredje maskin i nätverket.
* Testa att använda `hostname -I` för att verifiera IP-adressen.
* Använd `ip link` eller `ip a` för att se nätverkskortens status.

---

**🎓 Klart!**
Du har nu skapat ett privat nätverk i Hyper-V, anslutit flera servrar, och konfigurerat statiska IP-adresser i Ubuntu.
