# LAB05 - Samba 4 Active Directory Implementation

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04%20LTS-E95420?logo=ubuntu)
![Samba](https://img.shields.io/badge/Samba-4.19.5-blue)
![Status](https://img.shields.io/badge/status-complete-success)

Enterprise Active Directory deployment using Samba 4 on Ubuntu Server 24.04, featuring dual domain controllers with bidirectional forest trust, cross-platform client integration, and granular resource management.

## Infrastructure

```
                     Internet / External Network
                               │
                         Gateway: 172.30.20.1
                               │
                   ┌───────────┴───────────┐
                   │   Bridge Network       │
                   │   172.30.20.0/25       │
                   └─────┬───────────┬──────┘
                         │           │
                   .41   │           │   .42
                   ┌─────┴─────┐ ┌───┴───────┐
                   │   ls05    │ │   ls06     │
                   │ LAB05 DC  │ │ LAB06 DC   │
                   └─────┬─────┘ └───┬───────┘
                         │           │
                    .1   │           │   .2
                   ┌─────┴───────────┴──────┐
                   │   Internal Network      │
                   │   192.168.1.0/24        │
                   └─────┬───────────┬──────┘
                         │           │
                   ┌─────┴─────┐ ┌───┴───────┐
                   │  wc-05    │ │   lslc     │
                   │ Win11.100 │ │ Ubuntu .10 │
                   └───────────┘ └───────────┘

              Forest Trust (Bidirectional)
          LAB05.LAN ◄────────────► LAB06.LAN
```

| Component | LAB05 | LAB06 |
|-----------|-------|-------|
| **Domain** | lab05.lan | lab06.lan |
| **Realm** | LAB05.LAN | LAB06.LAN |
| **DC Hostname** | ls05.lab05.lan | ls06.lab06.lan |
| **Internal IP** | 192.168.1.1 | 192.168.1.2 |
| **Bridge IP** | 172.30.20.41 | 172.30.20.42 |
| **OS** | Ubuntu Server 24.04 | Ubuntu Server 24.04 |
| **Samba** | 4.19.5 | 4.19.5 |

## Active Directory Objects

| Category | Details |
|----------|---------|
| **OUs** | IT_Department, HR_Department, Students |
| **Groups** | IT_Admins, HR_Staff, Students, Finance |
| **Users** | alice, bob, charlie, iosif, karl, lenin, vladimir, liudmila |
| **Clients** | wc-05 (Windows 11), lslc (Ubuntu Desktop) |
| **Shares** | FinanceDocs (R/W + sticky), HRDocs (R/W), Public (Read-only) |

## Security Policies

| Policy | Value |
|--------|-------|
| Minimum password length | 12 characters |
| Password complexity | Enabled |
| Password history | 24 passwords |
| Max password age | 42 days |
| Account lockout | 5 attempts / 30 min |

## Sprint Summary

| Sprint | Topic | Status |
|--------|-------|--------|
| **Sprint 1** | Domain Controller Setup | ✅ Complete |
| **Sprint 2** | Users, Groups, OUs & GPOs | ✅ Complete |
| **Sprint 3** | Shared Folders, ACLs & Cron Backup | ✅ Complete |
| **Sprint 4** | Forest Trust Between Domains | ✅ Complete |
| **Appendix A** | Ubuntu Desktop Client Join | ✅ Complete |
| **Appendix B** | Windows 11 Client Join | ✅ Complete |

## Repository Structure

```
LAB05-Samba-AD/
├── README.md              ← You are here
├── Sprints.md             ← Complete implementation guide
├── Cheat_Sheet.md         ← Quick reference commands
├── LICENSE
└── images/
    ├── sprint1/           ← DC setup screenshots (10)
    ├── sprint2/           ← Users & GPOs screenshots (7)
    ├── sprint3/           ← Shares & ACLs screenshots (5)
    ├── sprint4/           ← Trust config screenshots (6)
    ├── client_ubuntu/     ← Ubuntu client screenshots (7)
    └── client_windows/    ← Windows client screenshots (5)
```

## Quick Start

**Prerequisites:** VirtualBox/VMware, Ubuntu Server 24.04 ISO, 8 GB RAM total, 80 GB disk.

1. Clone this repository
2. Follow [Sprints.md](Sprints.md) from Sprint 1 through Sprint 4
3. Use [Cheat_Sheet.md](Cheat_Sheet.md) for quick command reference

## Documentation

| Document | Description |
|----------|-------------|
| [Sprints.md](Sprints.md) | Full step-by-step implementation guide (all sprints + client appendices) |
| [Cheat_Sheet.md](Cheat_Sheet.md) | Command reference, troubleshooting, deployment checklist |

## Key Services

| Port | Service | Protocol |
|------|---------|----------|
| 53 | DNS | TCP/UDP |
| 88 | Kerberos | TCP/UDP |
| 389 | LDAP | TCP |
| 445 | SMB/CIFS | TCP |
| 636 | LDAPS | TCP |
| 3268 | Global Catalog | TCP |

## Technologies

Samba 4 · Ubuntu Server 24.04 · Kerberos · LDAP · SSSD · Realmd · ACLs · Netplan · Cron

## 📈 Project Statistics

- **Lines of Configuration**: 500+
- **Commands Documented**: 150+
- **Services Configured**: 6 (DNS, Kerberos, LDAP, LDAPS, SMB, Global Catalog)
- **Total Implementation Time**: 29 hours
- **Documentation Pages**: 60+

## 🤝 Contributing

This is an educational project. Contributions, suggestions, and improvements are welcome!

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/improvement`)
3. Commit your changes (`git commit -am 'Add new feature'`)
4. Push to the branch (`git push origin feature/improvement`)
5. Open a Pull Request

## 🙏 Acknowledgments

- **Samba Team** - For creating and maintaining Samba 4
- **Ubuntu Community** - For excellent documentation and support
- **SSSD/Realmd Developers** - For seamless Linux AD integration

## 📧 Contact

For questions or feedback about this implementation:
- Open an issue in this repository
- Check the troubleshooting guide in the main documentation

## 🔗 Related Resources

- [Samba Official Wiki](https://wiki.samba.org/)
- [Samba Wiki - Active Directory](https://wiki.samba.org/index.php/Active_Directory)
- [Samba AD DC HOWTO](https://wiki.samba.org/index.php/Setting_up_Samba_as_an_Active_Directory_Domain_Controller)
- [Ubuntu Server Guide](https://ubuntu.com/server/docs)
- [Active Directory Documentation (Microsoft)](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/)
- [SSSD Documentation](https://sssd.io/)
- [Ubuntu Server Guide - Samba](https://ubuntu.com/server/docs/samba-active-directory)
---

**Project Status**: ✅ Complete and Production-Ready

## License

MIT License — see [LICENSE](LICENSE) for details.

---

*Last Updated: February 21, 2026*
