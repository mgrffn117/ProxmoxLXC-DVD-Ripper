# 2. LXC Installation & Setup

**All steps in this file must be run *inside* the LXC container's shell.**

This guide covers installing all software needed to decrypt and rip DVDs.

## 2.1. Install Decryption Library

Most DVDs are encrypted. `libdvdcss` is required to read them.

1.  Update your package lists:
    ```bash
    sudo apt update
    ```
2.  Install the helper package:
    ```bash
    sudo apt install libdvd-pkg
    ```
3.  Run the configuration tool. This will download and build `libdvdcss`. Use the **`<Tab>`** key to select **"OK"** in the text-based menus.
    ```bash
    sudo dpkg-reconfigure libdvd-pkg
    ```

## 2.2. Install MakeMKV

We will use the official Debian repository for MakeMKV, not the Ubuntu PPA.

1.  **Install GPG Dependencies:**
    (You may get an error if these are already installed, which is fine).
    ```bash
    sudo apt install dirmngr
    mkdir -m 700 /root/.gnupg
    ```
2.  **Add the Repository GPG Key:**
    This allows `apt` to trust the repository.
    ```bash
    sudo gpg --no-default-keyring --keyring /usr/share/keyrings/makemkv.gpg --keyserver keyserver.ubuntu.com --recv-keys 9E5738E866C5E6B2
    ```
3.  **Add the Repository Source:**
    This command adds the repository to your sources list.
    ```bash
    echo 'deb [signed-by=/usr/share/keyrings/makemkv.gpg] [https://ramses.hjramses.com/deb/makemkv](https://ramses.hjramses.com/deb/makemkv) '$(lsb_release -cs)' main' | sudo tee /etc/apt/sources.list.d/makemkv.list
    ```
4.  **Install MakeMKV:**
    Now update your lists and install the command-line tools.
    ```bash
    sudo apt update
    sudo apt install makemkv-bin makemkv-oss
    ```

You are now ready to rip discs.
