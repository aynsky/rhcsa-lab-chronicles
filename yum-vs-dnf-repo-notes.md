# RHCSA Lab Chronicles – YUM vs DNF and Repositories

While practicing for RHCSA, I kept bumping into package management doubts.  
Here's my thought process and notes — maybe this helps someone else too.

---

## ❓ Is YUM old, or still used in RHEL 9?

- Yes, `yum` is **old**.  
- RHEL 7 → used the original `yum` (Python 2 based).  
- RHEL 8 and RHEL 9 → switched to **`dnf`**, but they kept `yum` as a **symlink** to `dnf` for backward compatibility.  
- So on RHEL 9 you can type `yum install ...` and it works, but under the hood it's really `dnf`.

---

## ❓ How to install packages from a repository they give us?

You can:  

### 1. Add the repo with `dnf config-manager --add-repo URL`
- But first install the tool:  
  ```bash
  dnf install -y dnf-plugins-core
  ```  
- Example:  
  ```bash
  dnf config-manager --add-repo http://foundation0.ilt.example.com/dvd/BaseOS
  ```  
- Then install packages normally:  
  ```bash
  dnf install -y vim
  ```  

### 2. Or create a repo file manually:
```ini
[BaseOS]
name=BaseOS Repository
baseurl=http://foundation0.ilt.example.com/dvd/BaseOS
enabled=1
gpgcheck=0
```

---

## ❓ How do we know if dnf is actually using the repo we added?

- Run `dnf repolist` → shows enabled repos.
- Run `dnf repolist all` → shows enabled + disabled repos.
- Run `dnf info <package>` → check "From repo" line.
- If you install a package, dnf output will mention the repo name.

---

## ❓ What if dnf itself is not installed at all?

That can happen in a minimal installation. Then:

1. Manually install dnf using `rpm -ivh` from the `BaseOS/Packages/` directory.
2. Order matters because rpm doesn't resolve dependencies. Typical sequence:
   - `dnf-data`
   - `libdnf`
   - `python3-dnf`
   - `dnf`
3. Once dnf works, create a `.repo` file and from then on just use `dnf install ...` as normal.

---

## ❓ Can we tab-complete the RPM name when installing?

- If you're pulling directly over HTTP (like `rpm -ivh http://.../dnf-data-*.rpm`) → ❌ no tab completion, because bash can't see inside a remote server.
- If you're working with a local ISO/DVD mounted on the system → ✅ yes, you can tab-complete because files are on the filesystem.

---

## ❓ In the RHCSA exam, will it be local or remote repos?

- In the exam, repos come from a **local ISO/DVD** provided by Red Hat.
- Usually mounted at `/mnt`, `/dvd`, or `/media/`.
- You will set up BaseOS and AppStream repos using `file:///` URLs.
- That means → tab completion works when using rpm because the RPMs are on disk.

---

## ❓ When it comes to package install order, is it always the same?

- With `rpm -ivh` manually → **yes**, order matters because dependencies must be installed first.
- With `dnf` → **order doesn't matter**, it resolves dependencies automatically.

---

## 📝 Takeaway

- **RHEL 7** → yum (old).
- **RHEL 8/9** → dnf (yum is just a wrapper).
- For **RHCSA exam** → expect to configure local repos from DVD/ISO.
- Use `dnf` whenever possible, but if bootstrapping from scratch → remember the RPM dependency order.