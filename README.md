# PrimeSync

A simple Bash script for syncing save data between devices using `rsync` over SSH.  

Originally built to sync Metroid Prime Trilogy saves (Wii version via PrimeHack) between a Steam Deck and a Linux PC, but the script is **fully adaptable to other games or emulators** by changing the `LOCAL_SAVE_PATH` and `REMOTE_SAVE_PATH` in the config file.

The included `primehack-wrapper` script is also easily modified to support other applications or emulators - just update the launch command to fit your use case.

This setup is lightweight, portable, and free of third-party cloud storage dependencies.

## 🔧 Features

- 📤 Push or 📥 pull save data with a single flag (`--push` or `--pull`)
- 💾 Uses `rsync` with `--update` to protect against overwriting newer saves
- 🔒 Verifies server availability before attempting sync
- ⚠️ Shows a native error dialog (`zenity`) if sync fails
- 🛠 Configurable via an external `.conf` file
- ✅ Designed to work in both Desktop and Steam Game Mode

## 📁 Folder Structure

```
prime-sync/
├── sync-save                # Main script
├── sync-save.conf           # Your private config (create this file yourself)
├── sync-save.conf.example   # Config template (copy this to sync-save.conf and edit)
├── primehack-wrapper        # Game launch script (used in Steam)
├── README.md
├── .gitignore
```

## 🛠 Setup

1. **Clone the repository:**

   ```bash
   git clone https://github.com/StripesAndPlaid/prime-sync
   cd prime-sync
   ```

2. **Copy and edit the config:**

   ```bash
   cp sync-save.conf.example sync-save.conf
   nano sync-save.conf
   ```

   Example config:

   ```bash
   REMOTE_HOST=your.server.address
   REMOTE_USER=yourusername

   LOCAL_SAVE_PATH=$HOME/.var/app/io.github.shiiion.primehack/data/dolphin-emu/Wii/title/00010000/52334d45/data
   REMOTE_SAVE_PATH=/home/$REMOTE_USER/metroid
   ```

3. **(Optional) Set up SSH key auth for password-less sync (recommended):**

   Generate a new `ed25519` key if you don’t already have one:

   ```bash
   ssh-keygen -t ed25519
   ```

   Then copy the public key to your server:

   ```bash
   ssh-copy-id yourusername@your.server.address
   ```

   You should now be able to SSH or `rsync` to your server without entering a password.

## 🚀 Usage

From the command line:

```bash
./sync-save --pull    # Download saves from server
./sync-save --push    # Upload saves to server
```

Or launch the game with save sync via the wrapper:

```bash
./primehack-wrapper /path/to/Metroid-Prime-Trilogy.iso
```

## 🎮 Steam Integration (Steam Deck or Desktop)

1. Add `primehack-wrapper` to Steam or use Steam ROM Manager to generate a shortcut.
2. Ensure the ROM path is passed as an argument (Steam ROM Manager handles this automatically).
3. The wrapper script will:
   - Pull the save from the server
   - Launch PrimeHack (or another application if modified)
   - Push the updated save back to the server after quitting

### Wrapper script contents:

```bash
#!/bin/bash

"$(dirname "$0")/sync-save" --pull
/usr/bin/flatpak run io.github.shiiion.primehack -b -e "$1"
"$(dirname "$0")/sync-save" --push
```

## ❗ Notes

- The script uses `/dev/tcp/$REMOTE_HOST/22` to check if your SSH server is reachable.
- `rsync` uses `--update` to avoid overwriting newer local or remote saves.
- Errors are shown via `zenity` if syncing fails — tested in both Desktop and Game Mode on SteamOS.

## ✅ Requirements

- `rsync`
- `bash`
- `zenity`
- SSH access to your server
