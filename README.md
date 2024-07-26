# NAS

markdownCopy# Backup Nextcloud ke QNAP NAS

Panduan ini menjelaskan langkah-langkah untuk melakukan backup data dari server Nextcloud Anda ke QNAP NAS secara otomatis.

## Prasyarat

1. Akses root atau sudo di server Ubuntu Anda.
2. Nextcloud telah diinstal dan dikonfigurasi.
3. QNAP NAS yang dapat diakses melalui jaringan dengan layanan SMB atau NFS diaktifkan.
4. Pengguna dengan izin untuk mengakses dan menulis ke share di QNAP NAS.

## Langkah 1: Buat Backup Script

### 1.1. Buat direktori untuk script dan backup

```bash
sudo mkdir -p /opt/nextcloud_backup
sudo mkdir -p /var/backups/nextcloud
1.2. Buat script backup
Buat dan edit script backup dengan nano:
bashCopysudo nano /opt/nextcloud_backup/backup.sh
Isi script dengan konten berikut:
bashCopy#!/bin/bash

# Direktori Nextcloud dan direktori backup
NEXTCLOUD_DIR=/var/www/nextcloud
BACKUP_DIR=/var/backups/nextcloud

# Pengaturan database
DB_NAME=nextcloud
DB_USER=nextclouduser
DB_PASSWORD=your_password

# Nama file backup
TIMESTAMP=$(date +"%F")
FILE_BACKUP=nextcloud-file-backup-$TIMESTAMP.tar.gz
DB_BACKUP=nextcloud-db-backup-$TIMESTAMP.sql

# Fungsi untuk log
log() {
    echo "$(date +"%F %T") : $1"
}

log "Mulai backup Nextcloud"

# Backup file Nextcloud
log "Backup file Nextcloud"
tar -czf $BACKUP_DIR/$FILE_BACKUP $NEXTCLOUD_DIR

# Backup database
log "Backup database Nextcloud"
mysqldump -u $DB_USER -p$DB_PASSWORD $DB_NAME > $BACKUP_DIR/$DB_BACKUP

# Mount QNAP NAS (ganti 'nas_share' dengan nama share di QNAP NAS)
log "Mount QNAP NAS"
NAS_IP=nas_ip_address
NAS_SHARE=nas_share
NAS_MOUNT_POINT=/mnt/qnap_nas
sudo mount -t cifs //$NAS_IP/$NAS_SHARE $NAS_MOUNT_POINT -o username=your_nas_username,password=your_nas_password

# Salin backup ke QNAP NAS
log "Salin backup ke QNAP NAS"
sudo cp $BACKUP_DIR/$FILE_BACKUP $NAS_MOUNT_POINT/
sudo cp $BACKUP_DIR/$DB_BACKUP $NAS_MOUNT_POINT/

# Unmount QNAP NAS
log "Unmount QNAP NAS"
sudo umount $NAS_MOUNT_POINT

log "Backup selesai"
Ganti your_password, nas_ip_address, nas_share, your_nas_username, dan your_nas_password dengan nilai yang sesuai.
1.3. Berikan izin eksekusi pada script
bashCopysudo chmod +x /opt/nextcloud_backup/backup.sh
Langkah 2: Uji Backup Script
Jalankan script secara manual untuk memastikan semuanya berfungsi dengan baik:
bashCopysudo /opt/nextcloud_backup/backup.sh
Langkah 3: Otomatiskan Backup dengan Cron
Tambahkan script ke cron untuk menjalankan backup otomatis:
bashCopysudo crontab -e
Tambahkan baris berikut untuk menjalankan backup setiap hari pada pukul 2 pagi:
cronCopy0 2 * * * /opt/nextcloud_backup/backup.sh >> /var/log/nextcloud_backup.log 2>&1
Simpan dan keluar dari editor.
Langkah 4: Verifikasi Backup
Periksa QNAP NAS untuk memastikan backup berhasil disalin. Pastikan file backup ada di share yang telah Anda tentukan.
Dengan langkah-langkah ini, data Nextcloud Anda akan dibackup secara otomatis setiap hari ke QNAP NAS. Anda dapat menyesuaikan frekuensi backup dan pengaturan lainnya sesuai kebutuhan Anda.
Copy
Anda dapat menyimpan konten ini sebagai file dengan nama `backup_nextcloud_to_qnap.md`. Fi
