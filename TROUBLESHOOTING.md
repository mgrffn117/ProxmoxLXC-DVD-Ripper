# Troubleshooting Guide

Here are common errors and their solutions.

---

### Error: `add-apt-repository: command not found`
* **Cause:** You are missing the tool to add PPAs.
* **Solution:** Run `sudo apt install software-properties-common`. (Note: For this project, we don't use the PPA, but this is the fix).

---

### Error: `AttributeError: 'NoneType' object has no attribute 'people'`
* **Cause:** `add-apt-repository` is missing its Python dependencies.
* **Solution:** Run `sudo apt install python3-launchpadlib ca-certificates`.

---

### Error: `E: The repository '... bookworm Release' does not have a Release file.`
* **Cause:** You tried to add an **Ubuntu PPA** (like `ppa:heyarje/makemkv-beta`) to a **Debian** system. They are not compatible.
* **Solution:** Remove the bad PPA (`sudo add-apt-repository --remove ppa:heyarje/makemkv-beta`) and follow [Step 2](./2-LXC-INSTALLATION.md) using the correct Debian repository.

---

### Error: `gpg: keyserver receive failed: No dirmngr`
* **Cause:** The GPG tool is missing its network helper (`dirmngr`) or its root directory (`/root/.gnupg`).
* **Solution:** Run `sudo apt install dirmngr` and `mkdir -m 700 /root/.gnupg`.

---

### Error: `The program can't find any usable optical drives.` or `Unknown device - '/dev/sr0'`
* **Cause:** This is a permissions error. The LXC can *see* the `/dev/sr0` file but is being blocked from *using* it as a drive. This is usually a two-part problem.
* **Solution 1:** Make sure `lxc.apparmor.profile: unconfined` is in your LXC config (see [Step 1](./1-HOST-CONFIGURATION.md)).
* **Solution 2:** Make sure you also passed through the generic SCSI device (e.g., `/dev/sg0`). MakeMKV needs this for low-level commands. See [Step 1](./1-HOST-CONFIGURATION.md).

---

### Error: I ripped the disc but it's just a trailer.
* **Cause:** You ripped the wrong Title ID (e.g., `0`).
* **Solution:** Follow the [Ripping Guide](./3-RIPPING-GUIDE.md) to scan the disc with the `info` command and find the title with the longest duration/largest size.

 ---

### Error: Service starts but does nothing (Stale Lock File)
* **Symptoms:** The service runs but exits immediately. Logs show no new activity. Running the script manually with `bash -x` shows it hitting the lock file check and exiting with `exit 0`.
* **Cause:** The server was rebooted unexpectedly, or the container was restarted while a job was active (or while the tray was open/ejecting). This killed the script before it could delete the `/tmp/autorip.lock` file.
* **Solution:** Manually remove the stale lock file to reset the process:
  ```bash
  rm /tmp/autorip.lock
