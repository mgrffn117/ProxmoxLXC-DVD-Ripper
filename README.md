# Proxmox DVD Ripper (LXC)

This project provides a detailed guide for passing a Proxmox host's physical DVD/Blu-ray drive to a Debian-based LXC container to rip discs using MakeMKV.

This method uses device mapping to grant the LXC container the necessary low-level access to the drive, which is required by MakeMKV.

## Project Guide

This guide is broken down into several parts. Follow them in order.

1.  **[Proxmox Host Configuration](./1-HOST-CONFIGURATION.md)**
    * Find the drive's device files (`/dev/sr0`, `/dev/sg0`).
    * Get the device Major/Minor numbers.
    * Configure the LXC's `.conf` file to pass through the devices.

2.  **[LXC Installation & Setup](./2-LXC-INSTALLATION.md)**
    * Install `libdvdcss` for decryption.
    * Add the correct MakeMKV repository for Debian.
    * Install the MakeMKV command-line tools.

3.  **[Ripping Guide](./3-RIPPING-GUIDE.md)**
    * How to scan a disc for titles.
    * How to identify the main movie.
    * The `makemkvcon` command to rip the movie.

 4. **[FFmpeg Merging Guide](./4-FFMPEG-GUIDE.md)**
    * How to install FFmpeg on the LXC container (Linux) or Windows host.
    * Generating the `list.txt` concatenation file automatically.
    * The `ffmpeg` command to losslessly merge split video files.
    * Using the MKV container to preserve DVD subtitles and chapter markers.

    ## Troubleshooting

* If you hit an error, please see the **[Troubleshooting Guide](./TROUBLESHOOTING.md)** for a list of common errors and their solutions.

## Contributing

Pull requests and bug reports are welcome! Please see the **[Contributing Guide](./CONTRIBUTING.md)**.
