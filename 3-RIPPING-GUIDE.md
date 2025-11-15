# 3. Ripping Guide

This guide covers the `makemkvcon` commands to scan and rip your disc.

## 3.1. Command Breakdown

The main command we use is:
`makemkvcon -r mkv dev:/dev/sr0 <TITLE_ID> <DESTINATION_PATH>`

* **`makemkvcon`**: The MakeMKV console (command-line) program.
* **`-r`**: "Robot mode." Runs the command without any interactive prompts.
* **`mkv`**: The action. "Make an MKV file."
* **`dev:/dev/sr0`**: The source device.
* **`<TITLE_ID>`**: The specific title on the disc to rip (e.g., `0`, `1`, `all`). **This is the most important part.**
* **`<DESTINATION_PATH>`**: The folder to save the final `.mkv` file (e.g., `/mnt/share/`).

## 3.2. How to Rip Your Movie

1.  **Insert your DVD** into the host's drive.

2.  **Find the Main Movie Title:**
    Run the `info` command to scan the disc.
    ```bash
    makemkvcon -r info dev:/dev/sr0
    ```
    Look at the output for the "Title" list. The main movie is the one with the **longest duration** and **largest file size**.

    *Example Output:*
    ```
    ...
    TINFO:0,9,0,"0:02:13"       (Duration: 2 minutes)
    TINFO:0,10,0,"88.7 MB"      (Size: 88.7 MB)
    ...
    TINFO:1,9,0,"1:21:57"       (Duration: 1 hour 21 mins)
    TINFO:1,10,0,"4.9 GB"        (Size: 4.9 GB)
    ...
    ```
    In this example, **Title 1** is clearly the main movie.

3.  **Run the Rip Command:**
    Use this command, replacing `<TITLE_ID>` with the number you just found (e.g., `1`) and `<PATH_TO_SHARE>` with your destination directory.
    
    ```bash
    makemkvcon -r mkv dev:/dev/sr0 1 /mnt/share/
    ```
    The `.mkv` file will be created in your target directory.

### Ripping All Titles

If you want to rip *everything* (trailers, menus, etc.) as separate files, you can use `all`:

```bash
makemkvcon -r mkv dev:/dev/sr0 all /mnt/share/
