# RHEL 9 VMware VM Setup and Networking Journal

## 1. Creating RHEL 9 VMs

I first created a **RHEL 9 VM** in VMware.  
- I chose the **Server version (CLI only)** during installation.  
- After the first VM was ready, I **cloned it** to create a second VM.  
- Both VMs are running **CLI only**, no GUI installed.

---

## 2. Adding a User and Configuring Sudo

I faced a problem where the user I created **could not use sudo**.  

- To fix this, I had to reload the VM and stop it at the **GRUB menu**:  
  1. Press `e` to edit the boot line.  
  2. Add `rd.break` at the end of the kernel line.  
  3. Press `Ctrl+X` to boot into the emergency shell.  

- Remount the system as read/write:  
  ```bash
  mount -o remount,rw /sysroot
  chroot /sysroot
  ```

- Edit the sudoers file safely with visudo:  
  ```bash
  visudo
  ```

- I added my user under the root line:  
  ```
  ## Allow root to run any commands anywhere
  root    ALL=(ALL)       ALL
  myuser  ALL=(ALL)       ALL
  ```
  > **Note:** `myuser` is my actual username.

- I repeated the same process on the second cloned server.

---

## 3. Shutting Down the VM via CLI

Before adding new network adapters, I shut down both VMs:
```bash
sudo shutdown -h now
```
This ensures the VM settings can be changed safely.

---

## 4. Adding Additional Network Adapters in VMware

I wanted the VMs to communicate with each other while keeping internet access.

I edited each VM and added a second network adapter:
- **First adapter** → NAT (for internet).
- **Second adapter** → Host-Only (for VM-to-VM communication).

---

## 5. Configuring Network Interfaces on RHEL 9

Once the VMs started, I used `nmcli` to configure the new Host-Only network adapter with static IPs.

### VM1 (example: Host-Only NIC: enp0s8)
```bash
# Create connection
sudo nmcli con add type ethernet ifname enp0s8 con-name hostonly1 ipv4.method manual ipv4.addresses 192.168.100.10/24

# Bring interface up
sudo nmcli con up hostonly1
```

### VM2 (example: Host-Only NIC: enp0s8)
```bash
sudo nmcli con add type ethernet ifname enp0s8 con-name hostonly2 ipv4.method manual ipv4.addresses 192.168.100.20/24
sudo nmcli con up hostonly2
```

### Test connectivity between VMs
```bash
ping 192.168.100.20  # From VM1
ping 192.168.100.10  # From VM2
```

✅ **Result:**
- Internet access remains available via the NAT adapter.
- VMs can communicate through the Host-Only network.

---

## 6. Summary

I created two RHEL 9 CLI-only VMs using VMware.

1. **Fixed sudo access** for a user using GRUB emergency mode + `visudo`.
2. **Added a second network adapter** (Host-Only) to enable VM-to-VM communication while keeping internet via NAT.
3. **Configured static IPs** with `nmcli` and verified connectivity.

This setup allows me to:
- ✅ Use CLI-only RHEL 9 servers.
- ✅ Run commands as a sudo user.
- ✅ Connect VMs to each other without breaking internet access.