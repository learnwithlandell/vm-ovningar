# 🧩 Labb: Arbeta med Checkpoints (Snapshots) i Hyper-V

## 🎯 Mål

I denna övning ska du lära dig att använda **Checkpoints** (även kallat snapshots) i Hyper-V.
Du kommer att skapa en checkpoint innan en förändring, göra en ändring i din virtuella maskin, och sedan återställa den till ursprungsläget.

---

## 🔧 Förberedelser

* Hyper-V är installerat.
* Du har en virtuell maskin, t.ex. **serverA** (Ubuntu 22.04 LTS eller Windows).
* Maskinen är **avstängd** eller **i viloläge** innan du börjar.

---

## 🪄 Steg 1️⃣ – Skapa en Checkpoint

1. Starta **Hyper-V Manager**.
2. Högerklicka på din VM (t.ex. `serverA`).
3. Välj **Checkpoint**.
4. Namnge checkpointen, t.ex. `Före konfigändring`.
5. Vänta tills den syns under VM-listan.

💡 *Tips:* Checkpoints sparar hela maskinens tillstånd – både disk, minne och konfiguration.

---

## ⚙️ Steg 2️⃣ – Gör en konfigurationsändring

1. Starta den virtuella maskinen.
2. Logga in och gör en enkel förändring, t.ex.:

   * Byt hostname
     ```
     sudo hostnamectl set-hostname test-server
     ```
   * Eller ändra en nätverksinställning i Netplan.
3. Kontrollera att ändringen gått igenom:
   ```
   hostname
   ```

---

## 🔁 Steg 3️⃣ – Återställ till Checkpoint

1. Stäng av den virtuella maskinen.
2. Högerklicka på checkpointen du skapade.
3. Välj **Apply** (eller “Revert to this Checkpoint”).
4. Bekräfta med **Apply** igen.

Nu kommer maskinen att återgå till exakt det läge den var i vid skapandet av checkpointen.

---

## 🧪 Steg 4️⃣ – Verifiera

Starta om VM:n och testa:
```
hostname
```
➡️ Den bör visa det **gamla** namnet (innan din ändring).

---

## 💡 Extra utmaning

* Skapa flera checkpoints och testa att hoppa mellan dem.
* Kontrollera var checkpoint-filerna sparas (mapp: `C:\ProgramData\Microsoft\Windows\Hyper-V\Snapshots`).
* Testa att ta bort en checkpoint och observera vad som händer med lagringen.

---

**🎓 Klart!**
Du har nu lärt dig skapa, återställa och hantera checkpoints i Hyper-V – ett viktigt verktyg för säker testning och felsökning.
