To create an SFTP-only user on your Ubuntu server that can access `/var/ftp/symul` and log in via FileZilla or SFTP Terminal command. Here's a step-by-step guide to do this securely:

---

#### 1️⃣ Create the SFTP user

```bash
sudo adduser symulftp
```
- Set a password when prompted.
- You can skip other info (just press Enter).

---

#### 2️⃣ Create the folder and set permissions
Make sure the folder exists:

```bash
sudo mkdir -p /var/ftp/symul
sudo chown root:root /var/ftp
sudo chmod 755 /var/ftp
```
Give the user access only to the `symul` folder inside:

```bash
sudo mkdir /var/ftp/symul
sudo chown symulftp:symulftp /var/ftp/symul
sudo chmod 755 /var/ftp/symul
```
**Notes:**

- `/var/ftp` must be owned by root and not writable by others — required for `ChrootDirectory`.
- The actual SFTP folder (`symul`) is owned by the user so they can read/write.


#### 🧩 (Optional) Mount Another Directory Inside /var/ftp/symul
If you want to give the SFTP user access to files or directories located elsewhere on the system, you can mount (bind) them inside the user's chroot folder.
**Example:**
Let’s say you have a directory `/root/server-files/symul` that you want the SFTP user symulftp to access inside `/var/ftp/symul`.
```bash
sudo mount --bind /root/server-files/symul /var/ftp/symul
```
**Check ownership and permissions**
```bash
sudo chown -R symulftp:symulftp /var/ftp/symul
sudo chmod 755 /var/ftp/symul
```

**Make it persistent after reboot**
Edit `/etc/fstab`:
```bash
sudo nano /etc/fstab
```
Add this line at the bottom:
```nano
/root/server-files/symul /var/ftp/symul  none  bind  0  0
```
**If you need to unmount**
```bash
sudo umount /var/ftp/symul
```

---

#### 3️⃣ Restrict the user to SFTP only
Edit SSH config:

```bash
sudo nano /etc/ssh/sshd_config
```
Add this at the end:

```bash
Match User symulftp
    ChrootDirectory /var/ftp
    ForceCommand internal-sftp
    AllowTcpForwarding no
    X11Forwarding no
```
*Explanation:*
- `ChrootDirectory /var/ftp` → user sees `/var/ftp` as root (`/` inside SFTP).
- `ForceCommand internal-sftp` → disables SSH shell access.
- `AllowTcpForwarding no` & `X11Forwarding no` → improve security.

💡 **Tip:** If you need to give access to multiple directories, you can create subfolders under `/var/ftp` and mount or symlink them inside, but `/var/ftp` itself must remain root-owned.

---

#### 4️⃣ Restart SSH

```bash
sudo systemctl restart ssh
```

---

#### 5️⃣ Test login in FileZilla or Terminal
Try in **FileZilla**
- Host: your server IP
- Protocol: SFTP
- Username: `symulftp`
- Password: the one you set
- Port: 22

Try in **Terminal**
- sftp symulftp@your_server_IP
- Password: the one you set


The user should land in `/symul` folder and cannot access anything outside `/var/ftp`.

---

✅ Optional: If you want the user *restricted strictly* to `/symul` and no write to `/var/ftp`, the above setup already handles that. `/var/ftp` must be owned by root for SFTP chroot to work.
