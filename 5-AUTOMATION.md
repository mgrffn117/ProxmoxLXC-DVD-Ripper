4. Automation Guide
This guide covers automating 

### File: `4-AUTOMATION.md`

````markdown
# 4. Automation (Headless Auto-Ripper)

**All steps in this file must be run *inside* the LXC container's shell.**

This guide turns your manual ripping station into a fully automated "robot." It sets up a system that:
1.  Checks the drive for a disc every 60 seconds.
2.  Identifies the disc by its unique byte size (preventing infinite re-ripping loops).
3.  Automatically scans and identifies the main movie title.
4.  Rips the movie to your defined share.
5.  **Unlocks and Ejects** the tray when finished.

## 4.1. Install Dependencies

We need `gawk` for text processing and `sg3-utils` to forcefully unlock and eject the drive (standard `eject` commands often fail inside containers due to permission locking).

```bash
sudo apt update
sudo apt install gawk eject sg3-utils
````

## 4.2. Create the Automation Script

We will create the bash script that handles the logic.

1.  Create the file:

    ```bash
    nano /usr/local/bin/autorip.sh
    ```

2.  Paste the following content into the file.

    > **Configuration Note:** \> \* Update `OUT_DIR` to match your Samba share path.

    >   * Ensure `DRIVE_DEV` (usually `/dev/sr0`) and `SG_DEV` (usually `/dev/sg0`) match the devices you identified in [Part 1](https://www.google.com/search?q=./1-HOST-CONFIGURATION.md).

    ```bash
    #!/bin/bash

    # --- CONFIGURATION ---
    DRIVE_DEV="/dev/sr0"
    SG_DEV="/dev/sg0"     # Generic SCSI device (Required for unlocking tray)
    OUT_DIR="/mnt/share"  # <--- CHANGE THIS to your actual output path
    MIN_LENGTH="3600"     # Minimum duration in seconds (3600 = 1 hour)
    LOG_FILE="/var/log/autorip.log"
    LOCK_FILE="/tmp/autorip.lock"
    STATE_FILE="/tmp/last_ripped_disc_id"

    # --- FUNCTIONS ---

    log() {
        echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" >> "$LOG_FILE"
    }

    cleanup() {
        rm -f "$LOCK_FILE"
    }

    # --- MAIN LOGIC ---

    # 1. Check if a job is already running
    if [ -f "$LOCK_FILE" ]; then
        exit 0
    fi

    # 2. Check if a disc is present AND get its Unique Size
    # We use the size in blocks as a unique fingerprint to prevent loops.
    DISC_SIZE=$(cat /sys/class/block/${DRIVE_DEV##*/}/size)

    if [ "$DISC_SIZE" -eq 0 ]; then
        # Drive is empty. Reset state so we are ready for a NEW disc.
        if [ -f "$STATE_FILE" ]; then
            rm -f "$STATE_FILE"
            log "Drive empty. Resetting state for next disc."
        fi
        exit 0
    fi

    # 3. RE-RIP PROTECTION
    # Check if this is the exact same disc size we just finished.
    if [ -f "$STATE_FILE" ]; then
        LAST_SIZE=$(cat "$STATE_FILE")
        if [ "$DISC_SIZE" == "$LAST_SIZE" ]; then
            # We already ripped this disc. Exit silently.
            exit 0
        fi
    fi

    # 4. LOGGING LABEL
    # Try to get a readable label for the log file (e.g. JURASSIC_PARK).
    CURRENT_LABEL=$(lsblk -n -o LABEL "$DRIVE_DEV")
    if [ -z "$CURRENT_LABEL" ]; then
        CURRENT_LABEL="Unknown_Disc"
    fi

    # 5. START THE JOB
    touch "$LOCK_FILE"
    trap cleanup EXIT

    log "New Disc detected: [$CURRENT_LABEL] (Size: $DISC_SIZE blocks). Starting scan..."

    # 6. SCAN FOR TITLES
    # Scan the disc and parse output to find the longest title.
    SCAN_OUTPUT=$(makemkvcon -r info dev:"$DRIVE_DEV" --minlength="$MIN_LENGTH")

    BEST_TITLE=$(echo "$SCAN_OUTPUT" | gawk -F, '
    /TINFO:.*,9,0,/ {
        gsub(/"/, "", $4)
        split($4, t, ":")
        if (length(t) == 3) seconds = t[1]*3600 + t[2]*60 + t[3]
        else seconds = t[1]*60 + t[2]
        
        split($1, id_arr, ":")
        title_id = id_arr[2]

        if (seconds > max_seconds) {
            max_seconds = seconds
            best_title = title_id
        }
    }
    END { print best_title }
    ')

    if [ -z "$BEST_TITLE" ]; then
        log "Error: No title found longer than $MIN_LENGTH seconds."
        exit 1
    fi

    log "Main Feature identified: Title ID $BEST_TITLE"
    log "Starting rip..."

    # 7. RIP THE FILE
    makemkvcon -r mkv dev:"$DRIVE_DEV" "$BEST_TITLE" "$OUT_DIR"
    EXIT_CODE=$?

    if [ $EXIT_CODE -eq 0 ]; then
        log "Rip completed successfully."
        
        # SAVE STATE: Mark this DISC SIZE as "Done"
        echo "$DISC_SIZE" > "$STATE_FILE"
        
        # --- EJECT SEQUENCE ---
        log "Waiting 10 seconds for drive buffer to release..."
        sleep 10
        
        # FORCE UNLOCK: Tell drive to allow medium removal (Fixes "Illegal Request" errors)
        log "Unlocking drive tray..."
        sg_prevent --allow "$SG_DEV"
        
        # Send SCSI Eject
        log "Sending SCSI eject command to $SG_DEV..."
        sg_start --eject "$SG_DEV"
        
        # Fallback standard eject (backup method)
        eject "$DRIVE_DEV" > /dev/null 2>&1
        
        log "Eject sequence complete."
    else
        log "Rip failed with exit code $EXIT_CODE."
    fi
    ```

3.  **Make the script executable:**

    ```bash
    chmod +x /usr/local/bin/autorip.sh
    ```

## 4.3. Configure Systemd Service

We use a systemd **Timer** to run this script every 60 seconds. This ensures the ripper survives reboots and crashes automatically.

1.  **Create the Service File:**

    ```bash
    cat <<EOF > /etc/systemd/system/autorip.service
    [Unit]
    Description=Auto DVD Ripper Service

    [Service]
    Type=simple
    ExecStart=/usr/local/bin/autorip.sh
    EOF
    ```

2.  **Create the Timer File:**

    ```bash
    cat <<EOF > /etc/systemd/system/autorip.timer
    [Unit]
    Description=Run Auto Ripper every minute

    [Timer]
    OnBootSec=1min
    OnUnitActiveSec=1min
    Unit=autorip.service

    [Install]
    WantedBy=timers.target
    EOF
    ```

3.  **Enable and Start:**

    ```bash
    systemctl daemon-reload
    systemctl enable autorip.timer
    systemctl start autorip.timer
    ```

## 4.4. Monitoring & Maintenance

  * **View Logs:**
    To see what the ripper is doing in real-time:

    ```bash
    tail -f /var/log/autorip.log
    ```

  * **Force a Reset:**
    If you re-insert the exact same disc and want to rip it again immediately (bypassing the re-rip protection):

    ```bash
    rm /tmp/last_ripped_disc_id
    ```

  * **Check Timer Status:**

    ```bash
    systemctl status autorip.timer
    ```

<!-- end list -->
