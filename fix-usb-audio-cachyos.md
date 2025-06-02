
# 🎧 Fix: USB Audio Interface Stops Working After Login on CachyOS

On CachyOS (or other PipeWire-based systems), some USB audio interfaces stop working after logging back in or resuming from suspend. You often have to manually switch the profile in `pavucontrol` or use tools like `pw-cli` or `pactl`.

This guide shows you how to **automatically fix this by restarting WirePlumber** on login using a systemd user service and timer.

---

## ✅ Quick Manual Fix

If your audio stops working after login, this command will usually fix it:

```bash
systemctl --user restart wireplumber
```

---

## 🛠️ Automate the Fix with systemd

To make this automatic after login:

### 1. Create the systemd service

```bash
mkdir -p ~/.config/systemd/user
nano ~/.config/systemd/user/restart-wireplumber.service
```

Paste the following:

```ini
[Unit]
Description=Restart WirePlumber after login
After=graphical-session.target

[Service]
ExecStart=/usr/bin/systemctl --user restart wireplumber
Type=oneshot
```

---

### 2. Create the systemd timer

```bash
nano ~/.config/systemd/user/restart-wireplumber.timer
```

Paste the following:

```ini
[Unit]
Description=Restart WirePlumber after login

[Timer]
OnBootSec=15
Persistent=false

[Install]
WantedBy=default.target
```

---

### 3. Enable the timer

Run:

```bash
systemctl --user daemon-reexec
systemctl --user enable --now restart-wireplumber.timer
```

---

## 🧪 (Optional) Test It

Reboot your system or log out and back in. Your audio interface should now work automatically without any manual toggling.

You can verify the service ran using:

```bash
systemctl --user status restart-wireplumber.service
```

---

## 📝 Notes

- This fix assumes your audio interface is fully detected but just not linked properly by PipeWire/WirePlumber.
- If you suspend/resume and still face issues, this approach can be extended to trigger after resume using `sleep.target` or a udev rule.
- This approach is safe and works well on most PipeWire-based setups including Arch, CachyOS, Fedora, and more.

---

## ✅ Done!

You no longer have to toggle audio profiles manually — enjoy smooth audio after every login 🎉
