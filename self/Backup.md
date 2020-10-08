Laptop: Externe Festplatte mit Deja Dup.
Server: Die Hardware steht im 10 Zoll Rack (s.u.) mit restic

# backup.fritz.box
- Raid 1 mit zwei Platten mount auf /media/backup_data
- auch als mediaserver
- user backup hat Zugriff auf sein Home in /media/backup_data/backups
- user backup hat eigenen Key
- user backup hat keinen Zugriff auf alles andere in /media/backup_data/ außer seinem Home (chmod 701).
- Port 22 ist freigegeben, ddns über inwx mit ddns.cyb.re
- Degraded raid? `mdadm --manage /dev/md0 -a /dev/<fehlende Platte>`