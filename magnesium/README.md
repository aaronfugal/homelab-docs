# Magnesium

**Magnesium** is the container that runs my Discord bot, which is fully documented in a dedicated repository due to its size and complexity.

📎 [View the Discord Bot Repository](https://github.com/aaronfugal/discord-bot)  

---

This bot integrates with multiple services in my homelab, including:
- **Plex** (via Radarr/Sonarr APIs)
- **SQL databases** for game tracking, approval flow, and user interaction
- **Web scraping** for real-time data
- **Authorization and approval systems** for media automation

Given the standalone nature and development effort behind this bot, it is kept in its own repository to allow for clearer issue tracking, updates, and version control.

---

## Deployment Setup

The Discord bot is hosted inside an LXC container on my Proxmox environment. Setup and maintenance is handled as follows:

1. **Development**
   - Code is written locally on `plutonium` using VS Code.

2. **Deployment to Container**
   - Code is transferred via `scp` to `magnesium` in the appropriate bot directory.

3. **Python Virtual Environment**
   - A virtual environment manages the dependencies.
   - Requirements are defined in `requirements.txt`.

4. **Service Management**
   - The bot runs via a systemd service:
     ```bash
     systemctl status discordbot.service
     journalctl -u discordbot.service
     ```

5. **Automated Backups**
   - A cron job runs daily to back up the three SQLite databases.
   - The backup script logs success/failure for each DB:

     #### 📄 `backup_games_db.sh`
     ```bash
     #!/bin/bash
     # Overwrite daily backups for games.db, upcoming.db, and users.db

     SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
     PROJECT_DIR="$(dirname "$SCRIPT_DIR")"

     LOG_FILE="$PROJECT_DIR/backup_log.txt"
     BACKUP_DIR="$PROJECT_DIR/backups"

     declare -A databases=(
       ["games.db"]="games_backup.db"
       ["upcoming.db"]="upcoming_backup.db"
       ["users.db"]="users_backup.db"
     )

     echo "[$(date)] Starting multi-database backup..." >> "$LOG_FILE"

     for db in "${!databases[@]}"; do
       SRC="$PROJECT_DIR/$db"
       DEST="$BACKUP_DIR/${databases[$db]}"
       
       if cp -f "$SRC" "$DEST"; then
         echo "[$(date)] Backup successful: $db → ${databases[$db]}" >> "$LOG_FILE"
       else
         echo "[$(date)] Backup failed: $db not found or inaccessible." >> "$LOG_FILE"
       fi
     done
     ```

---

## Optional Visual

A topology diagram can be added to show how `magnesium` connects to:
- Radarr and Sonarr APIs
- Plex server (`aluminum`)
- Client interaction via Discord

---
