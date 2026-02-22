# LAB05 - Cheat Sheet

Quick reference for Samba 4 Active Directory administration.

---

## Service Management

```bash
sudo systemctl status samba-ad-dc       # Check status
sudo systemctl restart samba-ad-dc      # Restart
sudo systemctl reload samba-ad-dc       # Reload config (no downtime)
sudo journalctl -u samba-ad-dc -f       # Follow logs
```

## Domain Information

```bash
sudo samba-tool domain level show        # Forest/domain functional level
sudo samba-tool domain info 127.0.0.1    # DC name, site, forest info
sudo samba-tool domain passwordsettings show  # Password policy
```

## User Management

```bash
# List / show
sudo samba-tool user list
sudo samba-tool user show <user>

# Create (password must meet policy)
sudo samba-tool user create <user> '<password>' \
  --given-name=<First> --surname=<Last>

# Delete
sudo samba-tool user delete <user>

# Password operations
sudo samba-tool user setpassword <user> --newpassword='<password>'
sudo samba-tool user setexpiry <user> --noexpiry

# Enable / disable
sudo samba-tool user enable <user>
sudo samba-tool user disable <user>
```

## Group Management

```bash
# List / show
sudo samba-tool group list
sudo samba-tool group listmembers <group>
sudo samba-tool group show <group>

# Create / delete
sudo samba-tool group add <group>
sudo samba-tool group delete <group>

# Members (no spaces between usernames)
sudo samba-tool group addmembers <group> <user1,user2,user3>
sudo samba-tool group removemembers <group> <user>
```

## Organizational Units

```bash
sudo samba-tool ou list
sudo samba-tool ou create "OU=<Name>,DC=lab05,DC=lan"
sudo samba-tool ou delete "OU=<Name>,DC=lab05,DC=lan"

# Create user directly in OU
sudo samba-tool user create <user> '<pass>' \
  --userou="OU=IT_Department,DC=lab05,DC=lan"
```

## Computer Accounts

```bash
sudo samba-tool computer list
sudo samba-tool computer show <name>
sudo samba-tool computer delete <name>
```

## GPO Management

```bash
sudo samba-tool gpo listall
sudo samba-tool gpo listcontainers "<GPO-GUID>"
sudo samba-tool gpo show "<GPO-GUID>"

# Advanced GPO editing: use RSAT from Windows client
# Win+R → gpmc.msc
```

## Password Policy

```bash
sudo samba-tool domain passwordsettings set --min-pwd-length=12
sudo samba-tool domain passwordsettings set --complexity=on
sudo samba-tool domain passwordsettings set --history-length=24
sudo samba-tool domain passwordsettings set --min-pwd-age=1
sudo samba-tool domain passwordsettings set --max-pwd-age=42
sudo samba-tool domain passwordsettings set --account-lockout-threshold=5
sudo samba-tool domain passwordsettings set --account-lockout-duration=30
sudo samba-tool domain passwordsettings set --reset-account-lockout-after=30
```

## Trust Management

```bash
# Create forest trust
sudo samba-tool domain trust create <domain> \
  --type=forest --direction=both -U administrator@<domain>

# Verify
sudo samba-tool domain trust list
sudo samba-tool domain trust show <domain>
sudo samba-tool domain trust validate <domain>

# Delete
sudo samba-tool domain trust delete <domain>
```

## DNS Verification

```bash
host -t A ls05.lab05.lan
host -t SRV _ldap._tcp.lab05.lan
host -t SRV _kerberos._tcp.lab05.lan
nslookup lab05.lan
```

## Kerberos Authentication

```bash
kinit <user>@<REALM>          # Get ticket (realm in UPPERCASE)
klist                         # Show tickets
kdestroy                      # Destroy tickets

# Examples
kinit administrator@LAB05.LAN
kinit alice@LAB05.LAN
kinit testuser@LAB06.LAN      # Cross-domain
```

## LDAP Queries

```bash
# Authenticate first
kinit administrator@LAB05.LAN

# Search users
ldapsearch -Y GSSAPI -H ldap://ls05.lab05.lan \
  -b "DC=lab05,DC=lan" "(objectClass=user)" cn sAMAccountName

# Search groups
ldapsearch -Y GSSAPI -H ldap://ls05.lab05.lan \
  -b "DC=lab05,DC=lan" "(objectClass=group)" cn

# Search OUs
ldapsearch -Y GSSAPI -H ldap://ls05.lab05.lan \
  -b "DC=lab05,DC=lan" "(objectClass=organizationalUnit)" dn

kdestroy
```

## Share Access (smbclient)

```bash
# List shares
smbclient -L localhost -U administrator

# Connect to share
smbclient //ls05.lab05.lan/Public -U alice

# Inside smbclient
smb: \> ls
smb: \> put localfile.txt
smb: \> get remotefile.txt
smb: \> exit
```

## CIFS Mount (Linux Client)

```bash
# Manual mount
sudo mount -t cifs //192.168.1.1/Public /mnt/public \
  -o username=alice,password='P@ssw0rd2026!',domain=LAB05,vers=3.0

# Unmount
sudo umount /mnt/public
```

## ACL Management

```bash
# View
getfacl <path>

# Set for group
sudo setfacl -m "g:LAB05\\<group>:rwx" <path>

# Set default (inherited by new files)
sudo setfacl -d -m "g:LAB05\\<group>:rwx" <path>

# Sticky bit (prevent deleting others' files)
sudo chmod +t <path>

# Remove ACL
sudo setfacl -x g:<group> <path>

# Remove all ACLs
sudo setfacl -b <path>
```

## Linux Client — Domain Join

```bash
# Install packages
sudo apt install -y realmd sssd sssd-tools libnss-sss libpam-sss \
  adcli samba-common-bin packagekit krb5-user cifs-utils smbclient

# Discover
sudo realm discover lab05.lan

# Join
sudo realm join --verbose --user=administrator lab05.lan

# Verify
realm list

# Enable home directory creation
sudo pam-auth-update
# Check: SSS authentication + Create home directory on login

# Test login
sudo su - alice@lab05.lan
```

## Network Diagnostics

```bash
ip a                                    # Show interfaces
ip route                                # Show routes
sudo ss -tulnp | grep -E ':(53|88|389|445|636|3268)'  # AD ports
cat /etc/resolv.conf                    # DNS config
cat /etc/netplan/50-cloud-init.yaml     # Netplan config
sudo netplan apply                      # Reapply network
```

## Process Signals

```bash
kill -19 <PID>      # SIGSTOP — freeze process
kill -18 <PID>      # SIGCONT — resume process
kill -15 <PID>      # SIGTERM — graceful stop
kill -9 <PID>       # SIGKILL — force kill
```

## Cron (Scheduled Tasks)

```bash
sudo crontab -e                  # Edit root crontab
sudo crontab -l                  # List root crontab

# Syntax: minute hour day month weekday command
# Example: daily at 19:00
0 19 * * * /usr/local/bin/backup_shares.sh
```

---

## Troubleshooting

### DNS Not Resolving

```bash
cat /etc/resolv.conf              # Must have nameserver 127.0.0.1
sudo systemctl status samba-ad-dc
host -t A ls05.lab05.lan          # Should return 192.168.1.1
```

### Port 53 Occupied

```bash
sudo ss -tulnp | grep :53
sudo systemctl disable --now systemd-resolved
sudo unlink /etc/resolv.conf
# Recreate resolv.conf manually, then restart samba-ad-dc
```

### Kerberos "Cannot find KDC"

```bash
sudo cp /var/lib/samba/private/krb5.conf /etc/krb5.conf
kinit administrator@LAB05.LAN
```

### Password Rejected on User Creation

```bash
sudo samba-tool domain passwordsettings show
# Password must be ≥12 chars with uppercase, lowercase, number, symbol
# Example compliant password: P@ssw0rd2026!
```

### Trust Creation Fails

```bash
# Check DNS forwarding
cat /etc/samba/smb.conf | grep "dns forwarder"
# Must include the other DC's IP

# Test resolution
host -t SRV _ldap._tcp.lab06.lan    # From ls05
host -t SRV _ldap._tcp.lab05.lan    # From ls06
```

### CIFS Mount "Permission Denied"

```bash
# Verify password meets domain policy (12+ chars)
# Verify user is member of the share's allowed group
sudo samba-tool group listmembers <group>
```

### SSSD Not Working on Linux Client

```bash
sudo chmod 600 /etc/sssd/sssd.conf
sudo systemctl stop sssd
sudo rm -rf /var/lib/sss/db/*
sudo systemctl start sssd
sudo journalctl -u sssd -f
```

### Network Resets After Reboot

```bash
sudo netplan apply

# If cloud-init overrides netplan:
sudo nano /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
# Add: network: {config: disabled}
```

## Post-Reboot Fixes

### Disable IPv6 (prevents Samba KDC bind errors)
```bash
# Temporary (until reboot)
sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1
sudo sysctl -w net.ipv6.conf.default.disable_ipv6=1

# Permanent
echo "net.ipv6.conf.all.disable_ipv6=1" | sudo tee -a /etc/sysctl.conf
echo "net.ipv6.conf.default.disable_ipv6=1" | sudo tee -a /etc/sysctl.conf
```

### Clear SSSD Cache (fixes stale login on domain clients)
```bash
sudo systemctl stop sssd
sudo rm -rf /var/lib/sss/db/*
sudo systemctl start sssd
```

### Restart GDM (fixes GUI login failures for domain users)
```bash
sudo systemctl restart gdm3
```

### Fix DNS on Ubuntu Client (if resolv.conf reverts)
```bash
echo "nameserver 192.168.1.1" | sudo tee /etc/resolv.conf
echo "search lab05.lan" | sudo tee -a /etc/resolv.conf
```

### Reapply Netplan (if interfaces lose IP after reboot)
```bash
sudo netplan apply
```
---

## Credentials Reference

| Account | Username | Password | Notes |
|---------|----------|----------|-------|
| DC1 Linux | pablo | admin_21 | System user |
| DC2 Linux | pablo | admin_21 | System user |
| Domain Admin LAB05 | Administrator | Admin_21 | Never expires |
| Domain Admin LAB06 | Administrator | Admin_21 | Never expires |
| All domain users | alice, bob, etc. | P@ssw0rd2026! | 42-day expiry |

## Key Paths

| Path | Description |
|------|-------------|
| `/etc/samba/smb.conf` | Samba main config |
| `/etc/krb5.conf` | Kerberos config |
| `/etc/resolv.conf` | DNS resolution |
| `/etc/netplan/50-cloud-init.yaml` | Network config |
| `/var/lib/samba/private/sam.ldb` | AD database |
| `/var/lib/samba/sysvol/` | GPOs and scripts |
| `/mnt/data/shares/` | Shared folders |
| `/backup/samba/` | Automated backups |

---

*Last Updated: February 21, 2026*
