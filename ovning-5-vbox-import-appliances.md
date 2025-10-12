# 🧪 Övning: Importera och testa nätverk mellan tre Debian-system i VirtualBox

I denna övning importerar du tre färdiga Debian-maskiner (system1, system2 och system3) i VirtualBox och testar nätverkskommunikationen mellan dem
Detta är samma som skapades i [ovning-4-vbox-lamp-wp-plattform.md](https://github.com/learnwithlandell/vm-ovningar/blob/main/ovning-4-vbox-lamp-wp-plattform.md).

---

## 🎯 Mål

Efter övningen ska du kunna:

* Importera `.ova`-filer i VirtualBox
* Kontrollera att varje system har rätt IP-adress
* Testa anslutning mellan systemen med `ping`

---

## 📦 1. Importera virtuella maskiner

Importera de tre appliances som du fått tillgång till:

```
system1.ova
system2.ova
system3.ova
```

Du kan ladda hem dem [från denna länk](https://contactusse-my.sharepoint.com/:f:/g/personal/emanuel_contactus_se/EtnLRmGB--1KkFj2bRAaF74B3QRBJFj54rYr0Y2uwigW1A?e=t5xkjA)

### Steg-för-steg

1. Öppna **VirtualBox**
2. Gå till menyn **File → Import Appliance**
3. Välj t.ex. `system1.ova`
4. Klicka **Next → Import**
5. Upprepa samma steg för `system2.ova` och `system3.ova`

---

## 🌐 2. Kontrollera nätverksinställningar

Alla tre maskiner ska vara kopplade till samma **Eget NAT-nätverk** (t.ex. `NATNetwork1`).

Kontrollera detta:

1. Markera varje maskin → **Settings → Network**
2. Under **Attached to**, välj **NAT Network**
3. Kontrollera att samma nätverk används för alla tre

---

## 💻 3. Starta upp maskinerna

Starta alla tre maskiner (system1, system2, system3).
Logga in på varje system med användaren **vboxuser** (eller den som gäller för miljön).

---

## 🔍 4. Verifiera IP-adresser

Kör följande kommando i varje maskin:

```
ip a
```

Kontrollera att adresserna är:

| System  | IP-adress |
| ------- | --------- |
| system1 | 10.0.2.7  |
| system2 | 10.0.2.8  |
| system3 | 10.0.2.15 |

---

## 🔄 5. Testa anslutning mellan systemen

Från **system1**, testa att pinga system2 och system3:

```
ping 10.0.2.8
ping 10.0.2.15
```

Från **system2**, testa att pinga system1:

```
ping 10.0.2.7
```

Från **system3**, testa att pinga system1 och system2:

```
ping 10.0.2.7
ping 10.0.2.8
```

Om du ser svar liknande detta fungerar nätverket:

```
64 bytes from 10.0.2.8: icmp_seq=1 ttl=64 time=0.523 ms
```

---

## ✅ 6. Klart!

Nu har du importerat tre Debian-system och verifierat att de kan kommunicera via ditt VirtualBox-nätverk.

---

### 💡 Tips

* Om `ping` inte fungerar, kontrollera att **firewall (ufw)** är avstängd:
  ```
  sudo ufw status
  sudo ufw disable
  ```
* Du kan använda `hostname -I` för att snabbt se aktuell IP-adress.

---

✍️ *Senast uppdaterad: Oktober 2025*
