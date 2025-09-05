# RHCSA Lab Chronicles 🖥️

A collection of my practice Q&A notes while preparing for the RHCSA exam.  
I've kept them conversational, like how I asked and learned.

---

## 🔑 SSH Keys

**Q:** When sharing SSH key, do we give pub?  
**A:** Yes ✅ — you only share the **public key (`.pub`)**, never the private key.  
- Private key → stays with you, never shared.  
- Public key → copied to the server (`~/.ssh/authorized_keys`).  

Command to copy the pub key:  
```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub student@servera
```

---

## 🌐 Hostname and Networking

**Q:** `/etc/sysconfig/network` we need remember this right?  
**A:**
- On RHEL 6/7 → used for system-wide network configs.
- On RHEL 8/9 (exam relevant) → mostly deprecated.
- Use `nmcli`, `nmtui`, `/etc/hostname`, and `/etc/hosts`.

**Q:** `/etc/hostname` is empty file?  
**A:** Yes, that's possible. If it's empty, the system falls back to `localhost.localdomain`.  
Set hostname persistently with:
```bash
hostnamectl set-hostname servera.example.com
```
This writes into `/etc/hostname` automatically.

**Q:** `sudo hostnamectl set-hostname servera.example.com` — if we just run this it not persistent?  
**A:** It **is** persistent ✅. It updates both the transient hostname and `/etc/hostname`.  
Just remember to also add an entry in `/etc/hosts` if needed:
```
192.168.0.10   servera.example.com   servera
```

---

## 📡 NetworkManager (nmcli)

**Q:** `nmcli status`?  
**A:** No plain `nmcli status`. Use instead:
- `nmcli device status` → shows devices and states.
- `nmcli connection show` → lists saved profiles.

**Q:** Using nmcli to add ipv4 address in there `ipv4.method auto` mean DHCP?  
**A:** Yes ✅.
- `ipv4.method auto` → DHCP
- `ipv4.method manual` → static IP
- `disabled` → no IPv4

Example static:
```bash
nmcli con mod eth0 ipv4.method manual ipv4.addresses 192.168.1.100/24 ipv4.gateway 192.168.1.1 ipv4.dns 8.8.8.8
nmcli con up eth0
```

**Q:** Can we skip `con add` part in there and change existing IP?  
**A:** Yes 👍 you can just `nmcli con mod` an existing connection instead of adding new.

**Q:** Is there config file to do this?  
**A:** Yes. Stored in:
```bash
/etc/NetworkManager/system-connections/
```
Files are INI-style (`.nmconnection`) and are updated automatically when you use nmcli.

**Q:** `uuid=1234abcd-...` mean dev?  
**A:** No 🙂. UUID identifies the connection profile, not the device.  
The actual device is defined in `interface-name=` inside the config.

**Q:** How add new interface without this?  
**A:** You don't need to write UUID yourself — nmcli auto-generates it. Example:
```bash
nmcli con add type ethernet ifname eth1 con-name eth1 ipv4.method auto
```

**Q:** Can't we check its status without up?  
**A:** Yes ✅.
- `nmcli device status` → see state of devices
- `nmcli connection show` → see saved profiles
- `ip addr show eth1` → show interface details

**Q:** 
```bash
nmcli connection modify enp0s3 autoconnect yes ipv4.method manual \
ipv4.addresses 172.25.250.100 ipv4.gateway 172.25.250.254 ipv4.dns 172.25.250.254
```
I know IP is wrong (need prefix) but is autoconnect necessary?

**A:**
- `autoconnect yes` → brings the interface up automatically on boot.
- Not required to make it work now, but recommended so it survives reboot.

Correct form:
```bash
nmcli con mod enp0s3 autoconnect yes ipv4.method manual \
ipv4.addresses 172.25.250.100/24 ipv4.gateway 172.25.250.254 ipv4.dns 172.25.250.254
```

**Q:** If `nmcli con add` command run as unsequence?  
**A:** Order doesn't matter ✅. You can shuffle parameters around; nmcli parses them fine.  
What matters:
- Required options are present.
- Correct syntax.
- If you miss something, fix with `nmcli con mod`.

---

## ✅ Summary

- **SSH** → share pub key only.
- **Hostname** → `hostnamectl set-hostname`, persistent in `/etc/hostname`.
- **Network configs** → manage with `nmcli` (files live in `/etc/NetworkManager/system-connections/`).
- `ipv4.method auto` = DHCP, `manual` = static.
- `autoconnect yes` ensures config survives reboot.
- Sequence of arguments in `nmcli` doesn't matter.