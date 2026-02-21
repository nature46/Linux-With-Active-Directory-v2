# LAB05 - Complete Implementation Guide

**Samba 4 Active Directory on Ubuntu Server 24.04**

---

## Table of Contents

- [Sprint 1: Domain Controller Setup](#sprint-1-domain-controller-setup)
- [Sprint 2: Users, Groups, OUs & GPOs](#sprint-2-users-groups-ous--gpos)
- [Sprint 3: Shared Folders, ACLs & Automated Backup](#sprint-3-shared-folders-acls--automated-backup)
- [Sprint 4: Forest Trust Between Domains](#sprint-4-forest-trust-between-domains)
- [Appendix A: Ubuntu Desktop Client](#appendix-a-ubuntu-desktop-client)
- [Appendix B: Windows 11 Client](#appendix-b-windows-11-client)

---

## Sprint 1: Domain Controller Setup

**Duration:** 6 hours
**Objective:** Install and configure a Samba 4 Active Directory Domain Controller on Ubuntu Server 24.04.

### System Information

| Parameter | Value |
|-----------|-------|
| **Hostname** | ls05.lab05.lan |
| **Domain** | lab05.lan |
| **Realm** | LAB05.LAN |
| **NetBIOS** | LAB05 |
| **Internal IP** | 192.168.1.1/24 |
| **Bridge IP** | 172.30.20.41/25 |
| **System user** | pablo |
| **Admin password** | Admin_21 |

### Step 1: Set Hostname

```bash
sudo hostnamectl set-hostname ls05.lab05.lan
hostname -f
```

### Step 2: Configure /etc/hosts

```bash
sudo nano /etc/hosts
```

Content:
```
127.0.0.1 localhost
192.168.1.1 ls05.lab05.lan ls05

::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

### Step 3: Update System

```bash
sudo apt update && sudo apt upgrade -y
```

![Initial IP configuration](images/sprint1/1.0_ip_initial.png)

### Step 4: Network Configuration (Netplan)

The DC uses two network interfaces: one bridged for internet access, and one on the internal network for domain traffic.

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

Configuration:
```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: no
      addresses:
        - 172.30.20.41/25
      routes:
        - to: default
          via: 172.30.20.1
      nameservers:
        addresses: [127.0.0.1, 10.239.3.7]
        search: [lab05.lan]
    enp0s8:
      dhcp4: no
      addresses:
        - 192.168.1.1/24
```

Apply:
```bash
sudo netplan apply
ip a
ip route
```

> **Note:** The bridge IP and DNS forwarder depend on your environment. For a home lab, use your router IP as gateway and `8.8.8.8` as forwarder.

![Hostname configuration](images/sprint1/1.1_hostname.png)
![/etc/hosts](images/sprint1/1.2_hosts.png)
![Netplan configuration](images/sprint1/1.3_netplan.png)

### Step 5: Disable systemd-resolved

⚠️ **CRITICAL:** Samba needs full control of port 53 (DNS). If systemd-resolved is running, Samba cannot start its internal DNS server.

```bash
sudo systemctl disable --now systemd-resolved
sudo unlink /etc/resolv.conf
sudo nano /etc/resolv.conf
```

Content:
```
nameserver 127.0.0.1
nameserver 10.239.3.7
search lab05.lan
```

Verify port 53 is free:
```bash
sudo ss -tulnp | grep :53
```

![systemd-resolved disabled](images/sprint1/1.4_resolved_disabled.png)

### Step 6: Install Samba and Dependencies

```bash
sudo apt install -y acl attr samba samba-dsdb-modules samba-vfs-modules \
  winbind libpam-winbind libnss-winbind libpam-krb5 krb5-config krb5-user \
  dnsutils ldap-utils smbclient
```

When prompted for Kerberos configuration:
- **Default realm:** `LAB05.LAN`
- **Kerberos servers:** `ls05.lab05.lan`
- **Admin server:** `ls05.lab05.lan`

Stop the default Samba services (we will use the AD DC service instead):
```bash
sudo systemctl disable --now smbd nmbd winbind
```

![Kerberos configuration](images/sprint1/1.5_kerberos_config.png)

### Step 7: Provision the Domain

Remove the default configuration and provision interactively:

```bash
sudo rm -f /etc/samba/smb.conf
sudo samba-tool domain provision --use-rfc2307 --interactive
```

Provisioning answers:

| Question | Answer |
|----------|--------|
| Realm | LAB05.LAN |
| Domain | LAB05 |
| Server Role | dc |
| DNS backend | SAMBA_INTERNAL |
| DNS forwarder | 10.239.3.7 |
| Administrator password | Admin_21 |

![Domain provisioning](images/sprint1/1.6_provision.png)

### Step 8: Configure Kerberos and Start Service

```bash
sudo cp /var/lib/samba/private/krb5.conf /etc/krb5.conf
sudo systemctl unmask samba-ad-dc
sudo systemctl enable samba-ad-dc
sudo systemctl start samba-ad-dc
sudo systemctl status samba-ad-dc
```

![Samba AD DC running](images/sprint1/1.7_samba_running.png)

### Step 9: Verification

Run all verifications to confirm the domain is operational:

```bash
# Domain level
sudo samba-tool domain level show

# Domain info
sudo samba-tool domain info 127.0.0.1

# DNS records
host -t A ls05.lab05.lan
host -t SRV _ldap._tcp.lab05.lan
host -t SRV _kerberos._tcp.lab05.lan

# Kerberos authentication
kinit administrator@LAB05.LAN
klist
kdestroy

# Default users
sudo samba-tool user list

# Listening ports
sudo ss -tulnp | grep -E ':(53|88|389|445|636|3268)'
```

![Full verification](images/sprint1/1.8_verification.png)

### Bonus: Task and Process Management

Understanding process control is essential for system administration. This exercise demonstrates how to pause and resume processes using signals.

Install the `sl` command (a steam locomotive animation):
```bash
sudo apt install -y sl
```

In **terminal 1**, run the animation:
```bash
sl
```

In **terminal 2**, find the PID and send signals:
```bash
# Find the process
pgrep -x sl

# SIGSTOP (19) — freeze the process
kill -19 $(pgrep -x sl)

# SIGCONT (18) — resume the process
kill -18 $(pgrep -x sl)
```

The locomotive freezes mid-screen when SIGSTOP is sent, then resumes exactly where it stopped with SIGCONT. The process stays in memory throughout — it is paused, not terminated.

![Process signals demonstration](images/sprint1/1.9_process_signals.png)

### Sprint 1 Summary

✅ System configured with dual network interfaces
✅ systemd-resolved disabled for Samba DNS
✅ Samba 4 installed and domain provisioned
✅ DNS, Kerberos, LDAP all verified
✅ Process management demonstrated

---

## Sprint 2: Users, Groups, OUs & GPOs

**Duration:** 6 hours
**Objective:** Create the organizational structure and configure domain security policies.

### Concepts: OUs vs Groups

| Feature | Organizational Unit (OU) | Security Group |
|---------|--------------------------|----------------|
| **Purpose** | Organization + GPO application | Assign permissions to resources |
| **Contains** | Users, Groups, Computers, OUs | Only members (users) |
| **Permissions** | Cannot assign to resources | Assigned to files, shares, etc. |
| **GPOs** | Applied to OUs | Not applied to groups |

### Step 1: Create Organizational Units

```bash
sudo samba-tool ou create "OU=IT_Department,DC=lab05,DC=lan"
sudo samba-tool ou create "OU=HR_Department,DC=lab05,DC=lan"
sudo samba-tool ou create "OU=Students,DC=lab05,DC=lan"
```

Verify:
```bash
sudo samba-tool ou list
```

![OUs created](images/sprint2/2.1_ous.png)

### Step 2: Create Security Groups

```bash
sudo samba-tool group add IT_Admins
sudo samba-tool group add HR_Staff
sudo samba-tool group add Students
sudo samba-tool group add Finance
```

Verify:
```bash
sudo samba-tool group list | grep -E "(IT_Admins|HR_Staff|Students|Finance)"
```

![Groups created](images/sprint2/2.2_groups.png)

### Step 3: Create Domain Users

All users are created with a password that meets the domain policy (12+ characters, complexity enabled): `P@ssw0rd2026!`

**Students:**
```bash
sudo samba-tool user create alice 'P@ssw0rd2026!' \
  --given-name=Alice --surname=Wonderland
sudo samba-tool user create bob 'P@ssw0rd2026!' \
  --given-name=Bob --surname=Marley
sudo samba-tool user create charlie 'P@ssw0rd2026!' \
  --given-name=Charlie --surname=Sheen
```

**IT Admins:**
```bash
sudo samba-tool user create iosif 'P@ssw0rd2026!' \
  --given-name=Iosif --surname=Stalin
sudo samba-tool user create karl 'P@ssw0rd2026!' \
  --given-name=Karl --surname=Marx
sudo samba-tool user create lenin 'P@ssw0rd2026!' \
  --given-name=Vladimir --surname=Lenin
```

**HR Staff:**
```bash
sudo samba-tool user create vladimir 'P@ssw0rd2026!' \
  --given-name=Vladimir --surname=Malakovsky
sudo samba-tool user create liudmila 'P@ssw0rd2026!' \
  --given-name=Liudmila --surname=Pavlichenko
```

Verify:
```bash
sudo samba-tool user list
```

![Users created](images/sprint2/2.3_users.png)

### Step 4: Assign Users to Groups

```bash
sudo samba-tool group addmembers Students alice,bob,charlie
sudo samba-tool group addmembers IT_Admins iosif,karl,lenin
sudo samba-tool group addmembers HR_Staff vladimir,liudmila
```

> **Note:** Users are separated by commas with no spaces.

Verify memberships:
```bash
sudo samba-tool group listmembers Students
sudo samba-tool group listmembers IT_Admins
sudo samba-tool group listmembers HR_Staff
sudo samba-tool group listmembers Finance
```

![Group memberships](images/sprint2/2.4_memberships.png)

### Step 5: Configure Password Security Policy

The password policy is applied domain-wide. This is equivalent to editing the Default Domain Policy GPO in Windows Server.

```bash
# Minimum 12 characters
sudo samba-tool domain passwordsettings set --min-pwd-length=12

# Require uppercase, lowercase, numbers, symbols
sudo samba-tool domain passwordsettings set --complexity=on

# Remember last 24 passwords
sudo samba-tool domain passwordsettings set --history-length=24

# Minimum 1 day before changing password again
sudo samba-tool domain passwordsettings set --min-pwd-age=1

# Force change every 42 days
sudo samba-tool domain passwordsettings set --max-pwd-age=42

# Lock account after 5 failed attempts
sudo samba-tool domain passwordsettings set --account-lockout-threshold=5

# Lockout duration: 30 minutes
sudo samba-tool domain passwordsettings set --account-lockout-duration=30

# Reset counter after 30 minutes
sudo samba-tool domain passwordsettings set --reset-account-lockout-after=30
```

Verify:
```bash
sudo samba-tool domain passwordsettings show
```

![Password policy](images/sprint2/2.5_password_policy.png)

Test the policy — a weak password should be rejected:
```bash
sudo samba-tool user create testgpo weak123
# ERROR: password too short / doesn't meet complexity

sudo samba-tool user create testgpo 'SecureP@ss2026!'
# Should succeed

sudo samba-tool user delete testgpo
```

![Policy test](images/sprint2/2.6_policy_test.png)

Set Administrator password to never expire:
```bash
sudo samba-tool user setexpiry Administrator --noexpiry
```

### Step 6: GPO Management

Samba 4 creates two default GPOs automatically. To inspect them:

```bash
# List all GPOs
sudo samba-tool gpo listall

# View SYSVOL structure (where GPOs are stored on disk)
sudo ls -la /var/lib/samba/sysvol/lab05.lan/Policies/

# See which container each GPO is linked to
sudo samba-tool gpo listcontainers "{31B2F340-016D-11D2-945F-00C04FB984F9}"
sudo samba-tool gpo listcontainers "{6AC1786C-016F-11D2-945F-00C04FB984F9}"
```

Default GPOs:
- **Default Domain Policy** → linked to the domain root (`DC=lab05,DC=lan`), controls password policy
- **Default Domain Controllers Policy** → linked to the Domain Controllers OU

> **Important:** Samba has limited GPO editing from the command line. For advanced GPO management (desktop restrictions, software deployment, drive mapping via GPO), use **Windows RSAT** from a domain-joined Windows client: press `Win+R` → `gpmc.msc`.

GPO processing order (LSDOU): **L**ocal → **S**ite → **D**omain → **OU**. The last applied has highest priority.

![GPO list and password policy](images/sprint2/2.7_gpo_list.png)

### Sprint 2 Summary

✅ 3 Organizational Units created
✅ 4 Security Groups created
✅ 8 Domain users created with compliant passwords
✅ Users assigned to appropriate groups
✅ Password security policy configured and tested
✅ GPO structure verified

---

## Sprint 3: Shared Folders, ACLs & Automated Backup

**Duration:** 6 hours
**Objective:** Configure shared folders on a dedicated disk with granular permissions and automated backup.

### Concepts: Two Permission Layers

Samba applies two layers of permissions, just like Windows Server:

1. **Share-level permissions** — defined in `smb.conf` (who can connect)
2. **Filesystem permissions** — POSIX + ACLs on disk (what they can do)

**The most restrictive permission wins.**

### Permission Matrix

| Share | Finance | HR_Staff | Students | Domain Admins |
|-------|---------|----------|----------|---------------|
| **FinanceDocs** | R/W (no delete others) | ✗ | ✗ | Full Control |
| **HRDocs** | ✗ | R/W | ✗ | Full Control |
| **Public** | R | R | R | Full Control |

### Step 1: Add and Mount Dedicated Disk

Using a separate disk for shares keeps data isolated from the OS.

```bash
# Identify the new disk
lsblk

# Create partition
sudo fdisk /dev/sdb
# n → p → 1 → Enter → Enter → w

# Format with ext4
sudo mkfs.ext4 /dev/sdb1

# Create mount point and mount
sudo mkdir -p /mnt/data
sudo mount /dev/sdb1 /mnt/data

# Make permanent — add to /etc/fstab
echo '/dev/sdb1 /mnt/data ext4 defaults 0 2' | sudo tee -a /etc/fstab
```

![Disk setup](images/sprint3/3.1_disk_setup.png)

### Step 2: Create Directory Structure

```bash
sudo mkdir -p /mnt/data/shares/{finance,hr,public}
sudo chown -R root:"LAB05\domain users" /mnt/data/shares
sudo chmod -R 770 /mnt/data/shares
ls -la /mnt/data/shares/
```

![Directories created](images/sprint3/3.1_directories.png)

### Step 3: Configure Samba Shares

Edit `/etc/samba/smb.conf` and add after the `[netlogon]` section:

```bash
sudo nano /etc/samba/smb.conf
```

```ini
#===================== Share Definitions =====================

[FinanceDocs]
    comment = Finance Department Documents
    path = /mnt/data/shares/finance
    valid users = @Finance, @"Domain Admins"
    read only = no
    browseable = yes
    create mask = 0660
    directory mask = 0770

[HRDocs]
    comment = HR Department Documents
    path = /mnt/data/shares/hr
    valid users = @HR_Staff, @"Domain Admins"
    read only = no
    browseable = yes
    create mask = 0660
    directory mask = 0770

[Public]
    comment = Public Shared Documents (Read-Only)
    path = /mnt/data/shares/public
    valid users = @"Domain Users"
    read only = yes
    browseable = yes
    write list = @"Domain Admins"
```

Verify syntax and reload:
```bash
testparm
sudo systemctl reload samba-ad-dc
smbclient -L localhost -U administrator
```

![Samba shares configured](images/sprint3/3.2_shares.png)

### Step 4: Configure ACLs

ACLs (Access Control Lists) provide granular permissions beyond basic POSIX. Domain groups are referenced with the `LAB05\` prefix.

**FinanceDocs — R/W with sticky bit (prevents deleting other users' files):**
```bash
sudo setfacl -m "g:LAB05\\finance:rwx" /mnt/data/shares/finance
sudo setfacl -d -m "g:LAB05\\finance:rwx" /mnt/data/shares/finance
sudo chmod +t /mnt/data/shares/finance
```

**HRDocs — normal R/W:**
```bash
sudo setfacl -m "g:LAB05\\hr_staff:rwx" /mnt/data/shares/hr
sudo setfacl -d -m "g:LAB05\\hr_staff:rwx" /mnt/data/shares/hr
```

**Public — read only:**
```bash
sudo setfacl -m "g:LAB05\\domain users:rx" /mnt/data/shares/public
sudo setfacl -d -m "g:LAB05\\domain users:rx" /mnt/data/shares/public
```

Verify all ACLs:
```bash
for dir in finance hr public; do
    echo "=== $dir ==="
    sudo getfacl /mnt/data/shares/$dir
    echo
done
```

> **What does the sticky bit do?** Users can create files, but can only delete files they own. Root and Domain Admins can delete anything. The `T` flag appears in `ls -la`.

![ACL configuration](images/sprint3/3.3_acls.png)

### Step 5: Automated Backup with Cron

Create a backup script that compresses the shares daily and keeps the last 7 backups:

```bash
sudo nano /usr/local/bin/backup_shares.sh
```

Content:
```bash
#!/bin/bash
# Automated backup for Samba shared folders
BACKUP_DIR="/backup/samba"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/samba_backup_$DATE.tar.gz" -C /mnt/data shares

# Keep only last 7 backups
cd "$BACKUP_DIR" && ls -t *.tar.gz 2>/dev/null | tail -n +8 | xargs -r rm

echo "Backup completed: $DATE" >> "$BACKUP_DIR/backup.log"
```

Make executable and test:
```bash
sudo chmod +x /usr/local/bin/backup_shares.sh
sudo /usr/local/bin/backup_shares.sh
ls -lh /backup/samba/
```

Schedule with cron (daily at 19:00):
```bash
(sudo crontab -l 2>/dev/null; echo "0 19 * * * /usr/local/bin/backup_shares.sh >> /var/log/samba_backup.log 2>&1") | sudo crontab -
sudo crontab -l
```

Cron syntax:
```
*    *    *    *    *
│    │    │    │    │
│    │    │    │    └── Day of week (0-7, 0=Sunday)
│    │    │    └─────── Month (1-12)
│    │    └──────────── Day of month (1-31)
│    └───────────────── Hour (0-23)
└────────────────────── Minute (0-59)
```

![Cron backup configured](images/sprint3/3.4_cron_backup.png)

### Sprint 3 Summary

✅ Dedicated disk mounted for shares at /mnt/data
✅ 3 shared folders configured in smb.conf
✅ Granular ACLs with domain group names
✅ Sticky bit on Finance (prevent unauthorized deletion)
✅ Automated daily backup with cron

---

## Sprint 4: Forest Trust Between Domains

**Duration:** 6 hours
**Objective:** Create a second AD domain and establish bidirectional forest trust.

### Architecture

```
Forest 1: lab05.lan               Forest 2: lab06.lan
├── DC: ls05.lab05.lan            ├── DC: ls06.lab06.lan
├── IP: 192.168.1.1               ├── IP: 192.168.1.2
└── Users: 8                      └── Users: 2
            │
            │ ◄──── Forest Trust (Bidirectional) ────►
            │
```

### Domain Comparison

| Parameter | lab05.lan | lab06.lan |
|-----------|-----------|-----------|
| **Hostname** | ls05.lab05.lan | ls06.lab06.lan |
| **Realm** | LAB05.LAN | LAB06.LAN |
| **Internal IP** | 192.168.1.1 | 192.168.1.2 |
| **Bridge IP** | 172.30.20.41 | 172.30.20.42 |
| **DNS Secondary** | 192.168.1.2 | 192.168.1.1 |

### Step 1: Create Second Domain Controller

Create a new VM and repeat Sprint 1 with the following differences:

- Hostname: `ls06.lab06.lan`
- Realm: `LAB06.LAN`, Domain: `LAB06`
- Internal IP: `192.168.1.2/24`
- Bridge IP: `172.30.20.42/25`

Netplan for ls06:
```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: no
      addresses:
        - 172.30.20.42/25
      routes:
        - to: default
          via: 172.30.20.1
      nameservers:
        addresses: [127.0.0.1, 10.239.3.7]
        search: [lab06.lan]
    enp0s8:
      dhcp4: no
      addresses:
        - 192.168.1.2/24
      nameservers:
        addresses: [127.0.0.1, 192.168.1.1]
        search: [lab06.lan]
```

Update `/etc/hosts` on ls06:
```
127.0.0.1 localhost
192.168.1.2 ls06.lab06.lan ls06
192.168.1.1 ls05.lab05.lan ls05
```

Provision the domain and start Samba as in Sprint 1.

![DC2 network configuration](images/sprint4/4.1_dc2_network.png)
![DC2 Samba running](images/sprint4/4.2_dc2_samba.png)

### Step 2: Configure DNS Forwarding

Each DC must be able to resolve the other domain. Add the other DC's IP as DNS forwarder.

**On ls05:**
```bash
sudo nano /etc/samba/smb.conf
# Change: dns forwarder = 192.168.1.2 10.239.3.7
sudo systemctl reload samba-ad-dc
```

**On ls06:**
```bash
sudo nano /etc/samba/smb.conf
# Change: dns forwarder = 192.168.1.1 10.239.3.7
sudo systemctl reload samba-ad-dc
```

Test cross-resolution from ls05:
```bash
host -t A ls06.lab06.lan
host -t SRV _ldap._tcp.lab06.lan
```

![DNS forwarding verified](images/sprint4/4.3_dns_forwarding.png)

### Step 3: Create Forest Trust

From ls05:
```bash
sudo samba-tool domain trust create lab06.lan \
  --type=forest \
  --direction=both \
  -U administrator@lab06.lan
```

Password: `Admin_21`

Verify:
```bash
sudo samba-tool domain trust list
sudo samba-tool domain trust show lab06.lan
sudo samba-tool domain trust validate lab06.lan
```

> **Note:** A netlogon error during validation is normal in Samba. The important lines are `OK: LocalValidation ... TRUST[WERR_OK]`.

![Trust created](images/sprint4/4.4_trust_create.png)
![Trust verified](images/sprint4/4.5_trust_verify.png)

### Step 4: Test Cross-Domain Authentication

Create test users on ls06:
```bash
sudo samba-tool user create testuser 'P@ssw0rd2026!' --given-name=Test --surname=User
sudo samba-tool user create testuser2 'P@ssw0rd2026!' --given-name=Test2 --surname=User
```

From ls05, authenticate as a LAB06 user:
```bash
kinit testuser@LAB06.LAN
klist
kdestroy
```

From ls06, authenticate as a LAB05 user:
```bash
kinit alice@LAB05.LAN
klist
kdestroy
```

![Cross-domain authentication](images/sprint4/4.6_cross_auth.png)

### Sprint 4 Summary

✅ Second DC (ls06.lab06.lan) created and provisioned
✅ DNS forwarding configured on both DCs
✅ Bidirectional forest trust established
✅ Trust validated successfully
✅ Cross-domain Kerberos authentication working

---

## Appendix A: Ubuntu Desktop Client

**Hostname:** lslc
**IP Address:** 192.168.1.10
**Domain:** lab05.lan
**OS:** Ubuntu Desktop 24.04

### Phase 1: Install Packages (With Internet)

Before switching to the internal network, install all required packages:

```bash
sudo apt update
sudo apt install -y realmd sssd sssd-tools libnss-sss libpam-sss \
  adcli samba-common-bin packagekit krb5-user cifs-utils smbclient
```

Kerberos prompts:
- **Default realm:** `LAB05.LAN`
- **Kerberos servers:** `ls05.lab05.lan`
- **Admin server:** `ls05.lab05.lan`

### Phase 2: Configure Network (Internal Only)

Switch the VM to internal network only (intnet). Configure static IP via GUI:

- **Address:** 192.168.1.10
- **Netmask:** 255.255.255.0
- **Gateway:** 192.168.1.1
- **DNS:** 192.168.1.1

![Network configuration](images/client_ubuntu/cu_1_network.png)

Configure hostname and DNS resolution:
```bash
sudo hostnamectl set-hostname lslc
```

Edit `/etc/hosts`:
```
127.0.0.1 localhost
127.0.1.1 lslc
192.168.1.10 lslc.lab05.lan lslc
```

Edit `/etc/resolv.conf`:
```
nameserver 192.168.1.1
search lab05.lan
```

Verify connectivity:
```bash
ping -c 2 192.168.1.1
nslookup lab05.lan
nslookup ls05.lab05.lan
```

![Connectivity verified](images/client_ubuntu/cu_2_connectivity.png)

### Phase 3: Discover and Join Domain

```bash
sudo realm discover lab05.lan
```

Expected output should show `type: kerberos` and `server-software: active-directory`.

```bash
sudo realm join --verbose --user=administrator lab05.lan
```

Password: `Admin_21`

The output should end with `Successfully enrolled machine in realm`.

![Realm join](images/client_ubuntu/cu_3_realm_join.png)

Verify:
```bash
realm list
```

Should show `configured: kerberos-member`.

![Realm list](images/client_ubuntu/cu_4_realm_list.png)

### Phase 4: Configure PAM Authentication

Enable domain authentication and automatic home directory creation:

```bash
sudo pam-auth-update
```

Ensure these are checked:
- **SSS authentication**
- **Create home directory on login**

![PAM configuration](images/client_ubuntu/cu_5_pam_config.png)

### Phase 5: Test Domain Login

Test via terminal:
```bash
sudo su - alice@lab05.lan
whoami
id
pwd
exit
```

![Alice login](images/client_ubuntu/cu_5_alice_login.png)

Test via GUI: log out and log in as `alice@lab05.lan` with password `P@ssw0rd2026!`. Use "Not listed?" if the user doesn't appear.

### Phase 6: Access Shared Folders

Mount a share using CIFS:
```bash
sudo mkdir -p /mnt/public
sudo mount -t cifs //192.168.1.1/Public /mnt/public \
  -o username=alice,password='P@ssw0rd2026!',domain=LAB05,vers=3.0
ls -la /mnt/public/
mount | grep cifs
```

![Share access](images/client_ubuntu/cu_7_share_access.png)

Alternatively, from the file manager: press `Ctrl+L` and type `smb://192.168.1.1/Public`.

### Appendix A Summary

✅ SSSD + Realmd installed and configured
✅ Ubuntu Desktop joined to lab05.lan domain
✅ PAM configured for domain users and home directory creation
✅ Domain login working (terminal and GUI)
✅ Shared folder access verified via CIFS mount

---

## Appendix B: Windows 11 Client

**Hostname:** wc-05
**IP Address:** 192.168.1.100
**Domain:** lab05.lan
**OS:** Windows 11 Professional

> **Note:** Windows 11 Home cannot join domains. You need Pro or Enterprise.

### Step 1: Configure Network

Set the VM to internal network only. Configure static IP:

- **IP:** 192.168.1.100
- **Mask:** 255.255.255.0
- **Gateway:** 192.168.1.1
- **DNS:** 192.168.1.1

![Network configuration](images/client_windows/cw_1_network.png)

Verify from cmd:
```cmd
ping 192.168.1.1
nslookup lab05.lan
```

![Connectivity verified](images/client_windows/cw_2_connectivity.png)

### Step 2: Join Domain

1. Press `Win+R` → type `sysdm.cpl` → **Computer Name** tab → **Change**
2. Select **Domain** and type: `lab05.lan`
3. Enter credentials: `administrator` / `Admin_21`
4. "Welcome to the lab05.lan domain" message appears
5. Restart the computer

![Domain join](images/client_windows/cw_3_domain_join.png)

### Step 3: Login as Domain User

After reboot:

1. Click **Other user** on the login screen
2. Enter: `LAB05\alice` / `P@ssw0rd2026!`
3. First login will be slow (profile creation)

Verify from cmd:
```cmd
whoami
net user alice /domain
```

![User info](images/client_windows/cw_5_user_info.png)

### Step 4: Access Shared Folders

Open File Explorer and type in the address bar:
```
\\ls05.lab05.lan
```

Available shares will appear: FinanceDocs, HRDocs, Public (depending on group membership).

![Shares visible](images/client_windows/cw_6_shares.png)

### Appendix B Summary

✅ Windows 11 Pro joined to lab05.lan
✅ Domain users can log in
✅ Network shares accessible from File Explorer
✅ DNS resolution and authentication working

---

## Document Information

| Field | Value |
|-------|-------|
| **Project** | LAB05 - Samba 4 Active Directory |
| **Version** | 3.0 |
| **Last Updated** | February 21, 2026 |
| **Environment** | Laboratory / Testing |
| **Total Duration** | ~29 hours |

---

**End of Implementation Guide**
