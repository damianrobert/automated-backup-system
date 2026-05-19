# 🗄️ Automated Backup System

A production-grade automated backup system for Ubuntu servers, featuring remote NFS storage, retention policies, logging, and failure detection. Built as a DevOps portfolio project.

---

## ✨ Features

- 📦 Backs up configurable directories using `tar`
- 🌐 Stores backups on a remote NFS mount
- 🔍 Verifies the remote destination is mounted before running
- 🧹 Automatically deletes backups older than 3 days
- 📝 Logs all activity (start, finish, errors) to a persistent log file
- 🚨 Detects and reports `tar` failures
- ⏰ Scheduled via `cron` to run daily at 08:00

---

## 🏗️ Architecture

```
[ Ubuntu Server ]
      |
      |  1. Check NFS mount
      |  2. Clean old backups
      |  3. Create timestamped .tgz archive
      |  4. Log result
      v
[ NFS Remote Host ] --> /mnt/backup/
```

The script is **storage-agnostic** by design — it operates on a mount point, so the underlying storage (NFS, SMB, or local disk) can be swapped without changing the script.

---

## 📋 Requirements

- Ubuntu Server (tested on 24.04)
- NFS client configured and mount defined in `/etc/fstab`
- `tar`, `find`, `mountpoint` (all included in standard Ubuntu)

---

## 📁 Project Structure

```
.
├── backup-script.sh   # Main backup script
└── README.md          # This file
```

---

## ⚙️ Configuration

Edit the variables at the top of `backup-script.sh`:

| Variable | Description | Default |
|---|---|---|
| `backup_files` | Space-separated list of directories to back up | `/home /var/spool/mail /etc /root /boot /opt` |
| `dest` | Mount point of the remote NFS share | `/mnt/backup` |
| `log_file` | Path to the log file | `$dest/backup.log` |

---

## 🚀 Setup

**1. Clone the repository**
```bash
git clone https://github.com/yourusername/automated-backup.git
cd automated-backup
```

**2. Make the script executable**
```bash
chmod +x backup-script.sh
```

**3. Configure your NFS mount**

Add your NFS share to `/etc/fstab`:
```
<nfs-server-ip>:/path/to/share  /mnt/backup  nfs  defaults  0  0
```

Then mount it:
```bash
sudo mount -a
```

**4. Schedule with cron**

Open the crontab editor:
```bash
crontab -e
```

Add this line to run every day at 08:00:
```
0 8 * * * /path/to/backup-script.sh
```

---

## 🔎 How It Works

### 1. Mount Check
Before doing anything, the script verifies the NFS destination is mounted using `mountpoint -q`. If the mount is down, the script exits immediately with a non-zero status code — preventing silent failures where backups appear to succeed but are written nowhere.

### 2. Retention Policy
Old backups are pruned using `find` with `-mtime +2`, which targets files **strictly older than 2 days** (i.e. 3+ days old). This keeps a rolling 3-day window of backups.

### 3. Archive Creation
`tar czf` creates a compressed `.tgz` archive with a timestamped filename in the format:
```
hostname-Weekday-DD-Mon-HH-MM-SS.tgz
```
Example: `helios-Tuesday-19-May-10-46-59.tgz`

### 4. Logging & Error Detection
All output — including `tar` warnings and errors — is appended to `backup.log`. The script checks `tar`'s exit code and logs either `Backup finished` or `Backup FAILED` accordingly.

---

## 📄 Sample Log Output

```
--------------------------------------------------------------
Backing up /etc to /mnt/backup/helios-Tuesday-19-May-10-46-59.tgz
Tue May 19 10:46:59 AM UTC 2026

tar: Removing leading '/' from member names

Backup finished
Tue May 19 10:46:59 AM UTC 2026
```

> Note: The `tar: Removing leading '/'` message is expected and harmless — tar strips leading slashes so archives can be safely extracted anywhere.

---

## 🛣️ Potential Improvements

- [ ] Move configuration to a `.env` file
- [ ] Add email or webhook alerting on failure
- [ ] Support multiple backup destinations (e.g. S3 via `rclone`)
- [ ] Checksum verification after backup
- [ ] Backup encryption with `gpg`

---

## 📜 License

MIT
