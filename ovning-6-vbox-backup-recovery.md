# 🧪 Övning: Automatiserad Databasbackup och Återställning (3-2-1 Principen)

I den här övningen kommer du att automatisera en backup av en MariaDB-databas med `mysqldump` och `cron`, överföra den till en annan maskin med `scp`, och testa att återställa databasen.
Du lär dig också grunderna i **3-2-1 backup-principen**:
**3 kopior, 2 olika media, 1 offsite.**

---

## 🎯 Mål

Efter övningen ska du kunna:

* Göra en automatiserad databasbackup
* Flytta backupen till en annan maskin
* Testa återställning av databasen
* Förstå 3-2-1 backup-principen

---

## 🧰 Förberedelser

1. Importera system1, system2 och system3 i Virtualbox på samma sätt som vi gjorde i följande övning: [ovning-5-vbox-import-appliances.md](https://github.com/learnwithlandell/vm-ovningar/blob/main/ovning-5-vbox-import-appliances.md)

2. Starta upp alla tre Debian-maskiner:

   * **10.0.2.7** → Källserver (MariaDB)
   * **10.0.2.8** → Backupserver (Offsite)
   * **10.0.2.15** → Testmaskin (Återställning)

3. Kontrollera att du kan pinga mellan dem:
   ```
   ping 10.0.2.8
   ping 10.0.2.15
   ```

4. Kontrollera att MariaDB är installerad på **10.0.2.7** och att SSH-servern är igång på alla maskiner:
   ```
   sudo apt install openssh-server -y
   systemctl enable --now ssh
   ```

5. Om du vill slippa skriva lösenord varje gång du använder `scp`, skapa SSH-nycklar:
   ```
   ssh-keygen
   ssh-copy-id elev@10.0.2.8
   ssh-copy-id elev@10.0.2.7
   ```

---

## 🔹 Steg 1: Skapa databas och backup-användare (10.0.2.7)

1. Logga in på **10.0.2.7**

2. Skapa en testdatabas med lite data:
   ```
   mysql -u root -p
   CREATE DATABASE test_db;
   USE test_db;
   CREATE TABLE data (id INT PRIMARY KEY, value VARCHAR(50));
   INSERT INTO data VALUES (1, 'Originaldata 1'), (2, 'Originaldata 2');
   FLUSH PRIVILEGES;
   \q
   ```

3. Skapa en användare med bara rättigheter för backup:
   ```
   mysql -u root -p
   CREATE USER 'backup_user'@'localhost' IDENTIFIED BY 'SakertLosen123!';
   GRANT SELECT, LOCK TABLES ON *.* TO 'backup_user'@'localhost';
   FLUSH PRIVILEGES;
   \q
   ```

4. Testa att ta en manuell backup:
   ```
   mysqldump -u backup_user -p test_db > /tmp/test_db_backup.sql
   echo "Backup lyckades!"
   ```

---

## 🔹 Steg 2: Skapa backup-skript (10.0.2.7)

1. Skapa katalogen där backupfilerna ska sparas:
   ```
   sudo mkdir -p /var/backups/mariadb
   sudo chown $USER:$USER /var/backups/mariadb
   ```

2. Skapa skriptet:
   ```
   nano /usr/local/bin/db_backup.sh
   ```

3. Klistra in detta innehåll:
   ```
   #!/bin/bash
   DB_USER="backup_user"
   DB_PASS="SakertLosen123!"
   DB_NAME="test_db"
   BACKUP_DIR="/var/backups/mariadb"
   DATE=$(date +%Y%m%d_%H%M%S)

   mysqldump -u ${DB_USER} -p${DB_PASS} ${DB_NAME} | gzip > ${BACKUP_DIR}/${DB_NAME}_${DATE}.sql.gz

   # Radera gamla filer (äldre än 7 dagar)

   find ${BACKUP_DIR} -name "*.sql.gz" -type f -mtime +7 -delete

   # Logga resultat

   if [ $? -eq 0 ]; then
   echo "${DATE}: Backup av ${DB_NAME} lyckades." >> /var/log/db_backup.log
   else
   echo "${DATE}: FEL vid backup av ${DB_NAME}!" >> /var/log/db_backup.log
   fi
   ```

4. Gör skriptet körbart och testa:
   ```
   chmod +x /usr/local/bin/db_backup.sh
   /usr/local/bin/db_backup.sh
   ls -l /var/backups/mariadb/
   cat /var/log/db_backup.log
   ```

---

## 🔹 Steg 3: Kör backupen automatiskt (cron)

1. Öppna crontab:
   ```
   crontab -e
   ```

2. Lägg till en rad som kör backupen varje timme:
   ```
   0 * * * * /usr/local/bin/db_backup.sh > /dev/null 2>&1
   ```
   (Vill du testa snabbare? Kör varje minut i stället: `* * * * *`)

3. Vänta några minuter och kontrollera att nya filer skapats.

---

## 🔹 Steg 4: Skicka backupen till backupservern (10.0.2.8)

1. På **10.0.2.8**, skapa en mapp för inkommande backupfiler:
   ```
   sudo mkdir -p /mnt/backups/10.0.2.7
   sudo chown $USER:$USER /mnt/backups/10.0.2.7
   ```

2. På **10.0.2.7**, öppna ditt backup-skript igen:
   ```
   nano /usr/local/bin/db_backup.sh
   ```

3. Lägg till raden nedan **efter** `mysqldump`-delen men **innan** rensningen:
   ```
   LAST_BACKUP=$(ls -t ${BACKUP_DIR}/*.sql.gz | head -1)
   scp ${LAST_BACKUP} elev@10.0.2.8:/mnt/backups/10.0.2.7/
   ```

4. Spara, kör skriptet igen och kontrollera på **10.0.2.8** att filen kom fram:
   ```
   ls /mnt/backups/10.0.2.7/
   ```

---

## 🔹 Steg 5: Testa återställning (10.0.2.15)

1. Hämta backupfilen från **10.0.2.7** eller **10.0.2.8**:
   ```
   mkdir -p ~/test_restore && cd ~/test_restore
   scp elev@10.0.2.7:/var/backups/mariadb/test_db_YYYYMMDD_HHMMSS.sql.gz .
   ```

2. Skapa en ny testdatabas:
   ```
   mysql -u root -p
   CREATE DATABASE restore_test_db;
   \q
   ```

3. Återställ backupen:
   ```
   gunzip < test_db_YYYYMMDD_HHMMSS.sql.gz | mysql -u root -p restore_test_db
   ```

4. Kontrollera innehållet:
   ```
   mysql -u root -p
   USE restore_test_db;
   SELECT * FROM data;
   \q
   ```

Om du ser “Originaldata 1” och “Originaldata 2” är återläsningen lyckad ✅

---

## 💡 Reflektion

* Hur uppfyller din lösning **3-2-1-principen**?
* Varför är det viktigt att **testa återställningen** och inte bara lita på att backupen finns?
* Vad händer om backupen misslyckas tyst?
* Hur kan du förbättra säkerheten i ditt skript?

---

## 🏁 Bonus: Använd `rsync` istället för `scp`

Om du vill att backupöverföringen ska gå snabbare och bara kopiera ändrade filer kan du byta ut `scp` mot `rsync`.

1. Installera rsync på båda maskinerna:
   ```
   sudo apt install rsync -y
   ```

2. Byt ut raden i skriptet:
   ```
   rsync -avz ${LAST_BACKUP} elev@10.0.2.8:/mnt/backups/10.0.2.7/
   ```

3. Testa körningen:
   ```
   /usr/local/bin/db_backup.sh
   ```

`rsync` känner automatiskt av om en fil redan finns och hoppar över den — perfekt för stora databaser och långsamma nätverk.

---

✍️ *Senast uppdaterad: Oktober 2025*
