
1. Install `mongodb-bin` from the AUR (prebuilt binary) unless you want to build from source (the `mongodb` AUR package). Arch removed MongoDB from the official repos, so AUR is the way. ([ArchWiki][1])
2. Ensure DB directories exist and are owned by the `mongodb` user.
3. Enable & start the `mongodb.service` systemd unit.
4. Use `mongosh` to connect; enable authentication if you expose the DB to networks. ([ArchWiki][1])

---

# Install from AUR 

If you have an AUR helper such as `yay`:

```bash
# (if you don't have yay) install base-devel & git first:
sudo pacman -S --needed base-devel git

# Install mongodb prebuilt binary + shell/tools (adjust package names as needed)
yay -S mongodb-bin mongosh-bin mongodb-tools-bin
```

# Post-install (create directories & start service)

Create the DB & log directories and set ownership (if the package didn't already):

```bash
sudo mkdir -p /var/lib/mongodb /var/log/mongodb
sudo chown -R mongodb:mongodb /var/lib/mongodb /var/log/mongodb
```

Enable & start MongoDB with systemd:

```bash
sudo systemctl enable --now mongodb.service
# check status
sudo systemctl status mongodb.service
```
---

# Connect with the shell

Newer MongoDB uses the `mongosh` shell (recommended):

```bash
mongosh
# or to connect with username (after you enable authentication)
mongosh -u <user> -p
```

If authentication is not configured, MongoDB listens on `127.0.0.1` by default but **does not require authentication** — enable access control before binding to 0.0.0.0. See the config at `/etc/mongodb.conf`. ([ArchWiki][1])

---

# Optional
Install mongodb compass

```txt
yay -S mongodb-compass-bin
```