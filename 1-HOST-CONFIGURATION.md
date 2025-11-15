# 1. Proxmox Host Configuration

**All steps in this file must be run on the Proxmox host shell, NOT the LXC.**

This guide covers finding the required device files and configuring the LXC to access them.

## 1.1. Find Your Devices

MakeMKV needs access to both the DVD block device (e.g., `/dev/sr0`) and its generic SCSI device (e.g., `/dev/sg0`).

1.  **Install `lsscsi`:**
    This tool helps identify the link between your drive and its generic SCSI device.
    ```bash
    apt install lsscsi
    ```

2.  **Find the `sg` Device:**
    Run `lsscsi` to list your devices. Look for your CD/DVD drive.
    ```bash
    lsscsi -g
    ```
    *Example Output:*
    ```
    [0:0:0:0]    cd/dvd  hp      DVD A  DS8A5LH   1HE4  /dev/sr0   /dev/sg0
    ```
    This shows our drive is `/dev/sr0` and its generic device is `/dev/sg0`.

3.  **Get Device Major/Minor Numbers:**
    We need the "address" (Major, Minor) for both devices.
    
    *For `/dev/sr0` (Block Device):*
    ```bash
    ls -l /dev/sr0
    ```
    *Example Output:*
    ```
    brw-rw---- 1 root cdrom 11, 0 Nov 15 10:33 /dev/sr0
    ```
    The numbers are **`b 11:0`** (b = block device).

    *For `/dev/sg0` (Character Device):*
    ```bash
    ls -l /dev/sg0
    ```
    *Example Output:*
    ```
    crw-rw---- 1 root cdrom 21, 0 Nov 14 14:36 /dev/sg0
    ```
    The numbers are **`c 21:0`** (c = character device).

    **Note:** Your numbers may be different! Use the ones your system shows.

## 1.2. Configure the LXC

1.  **Stop the LXC container** from the Proxmox GUI or shell:
    ```bash
    pct stop <CTID>
    ```

2.  **Edit the LXC config file**. (Replace `<CTID>` with your container's ID, e.g., `101`).
    ```bash
    nano /etc/pve/lxc/<CTID>.conf
    ```

3.  **Add the following lines** to the bottom of the file. These lines disable AppArmor (for permissions), grant the cgroup access to the devices, and mount them into the container.
    
    > **Important:** Use the Major:Minor numbers you found in Step 1. The example below uses `11:0` and `21:0`.

    ```ini
    # Disable AppArmor for full device access
    lxc.apparmor.profile: unconfined
    
    # DVD block device (/dev/sr0)
    lxc.cgroup2.devices.allow: b 11:0 rwm
    lxc.mount.entry: /dev/sr0 dev/sr0 none bind,optional,create=file
    
    # DVD generic SCSI device (/dev/sg0)
    lxc.cgroup2.devices.allow: c 21:0 rwm
    lxc.mount.entry: /dev/sg0 dev/sg0 none bind,optional,create=file
    ```

4.  Save the file and **start the LXC container**:
    ```bash
    pct start <CTID>
    ```
