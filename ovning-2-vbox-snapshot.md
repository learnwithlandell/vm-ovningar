# 🧩 Labb: Hantera Snapshots i VirtualBox

## 🎯 Mål

I denna övning lär du dig att skapa, återställa och ta bort **Snapshots** i VirtualBox.
Du använder dem för att säkra din VM innan förändringar görs.

---

## 🔧 Förberedelser

* VirtualBox är installerat.
* Du har minst en virtuell maskin, t.ex. **serverA**.
* Maskinen är avstängd eller i viloläge innan du börjar.

---

## 🪄 Steg 1️⃣ – Skapa ett Snapshot

1. Starta **VirtualBox Manager**.
2. Markera din VM → Klicka på fliken **Snapshots**.
3. Klicka på **Take** (kamera-ikonen).
4. Döp snapshotet till t.ex. `Före konfigändring`.
5. Bekräfta med **OK**.

💡 *Tips:* Ett snapshot sparar VM:ens tillstånd, hårddisk och konfiguration.

---

## ⚙️ Steg 2️⃣ – Gör en konfigurationsändring

1. Starta din VM.
2. Gör en enkel förändring, t.ex. byt hostname:
   ```
   sudo hostnamectl set-hostname test-server
   ```
3. Kontrollera att ändringen gått igenom:
   ```
   hostname
   ```

---

## 🔁 Steg 3️⃣ – Återställ till Snapshot

1. Stäng av den virtuella maskinen.
2. Gå till fliken **Snapshots** i VirtualBox.
3. Markera snapshotet `Före konfigändring`.
4. Klicka på **Restore** → bekräfta.
5. Starta maskinen igen.

---

## 🧪 Steg 4️⃣ – Verifiera

Kör:
```
hostname
```
➡️ Den ska nu visa det **gamla** värdet (innan ändringen).

---

## 💡 Extra utmaning

* Skapa flera snapshots och växla mellan dem.
* Ta bort ett snapshot och observera diskstorlekens förändring.
* Kombinera snapshot-återställning med dina nätverksövningar.

---

**🎓 Klart!**
Du har nu lärt dig att arbeta med snapshots i VirtualBox – en viktig del för säker testning och felsökning.
