# LAB05 - Samba 4 Active Directory Implementation

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04%20LTS-E95420?logo=ubuntu)
![Samba](https://img.shields.io/badge/Samba-4.19.5-blue)
![AWS](https://img.shields.io/badge/AWS-EC2-FF9900?logo=amazonaws)
![Status](https://img.shields.io/badge/status-complete-success)

Enterprise Active Directory deployment using Samba 4 on Ubuntu Server 24.04, featuring dual domain controllers with bidirectional forest trust, cross-platform client integration, granular resource management, and AWS EC2 cloud deployment.

## Infrastructure

### On-Premises (Sprints 1–4)

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

### AWS Cloud (Sprint 5)

```
                          Internet
                              │
                    ┌─────────┴─────────┐
                    │   LAB05-VPC        │
                    │   10.0.0.0/16      │
                    │                    │
                    │  ┌─────────────┐   │
                    │  │ ubuntu-server│   │
                    │  │ Samba AD DC  │   │
                    │  │ 10.0.1.229   │   │
                    │  │ Elastic IP   │   │
                    │  │ 54.173.102.89│   │
                    │  └──────┬──────┘   │
                    │         │ VPC      │
                    │  ┌──────┴──────┐   │
                    │  │windows-server│  │
                    │  │ Domain Client│  │
                    │  │ 10.0.14.107  │  │
                    │  │ Elastic IP   │  │
                    │  │54.221.100.222│  │
                    │  └─────────────┘   │
                    └───────────────────┘
                    Security Group: LAB05-SG
```

| Component | On-Premises LAB05 | On-Premises LAB06 | AWS |
|-----------|-------------------|-------------------|-----|
| **Domain** | lab05.lan | lab06.lan | lab05.lan |
| **DC Hostname** | ls05.lab05.lan | ls06.lab06.lan | ls05.lab05.lan |
| **Internal IP** | 192.168.1.1 | 192.168.1.2 | 10.0.1.229 |
| **OS** | Ubuntu Server 24.04 | Ubuntu Server 24.04 | Ubuntu Server 24.04 |

## Active Directory Objects

| Category | Details |
|----------|---------|
| **OUs** | IT_Department, HR_Department, Students |
| **Groups** | IT_Admins, HR_Staff, Students, Finance |
| **Users** | alice, bob, charlie, iosif, karl, lenin, vladimir, liudmila |
| **Clients** | wc-05 (Windows 11), lslc (Ubuntu Desktop), windows-server (AWS) |
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

| Sprint | Topic | Environment | Status |
|--------|-------|-------------|--------|
| **Sprint 1** | Domain Controller Setup | On-premises | ✅ Complete |
| **Sprint 2** | Users, Groups, OUs & GPOs | On-premises | ✅ Complete |
| **Sprint 3** | Shared Folders, ACLs & Cron Backup | On-premises | ✅ Complete |
| **Sprint 4** | Forest Trust Between Domains | On-premises | ✅ Complete |
| **Sprint 5** | AWS EC2 Cloud Deployment | AWS | ✅ Complete |
| **Appendix A** | Ubuntu Desktop Client Join | On-premises | ✅ Complete |
| **Appendix B** | Windows 11 Client Join | On-premises | ✅ Complete |

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
    ├── sprint5/           ← AWS deployment screenshots (9)
    ├── client_ubuntu/     ← Ubuntu client screenshots (7)
    └── client_windows/    ← Windows client screenshots (5)
```

## Quick Start

### On-Premises

**Prerequisites:** VirtualBox/VMware, Ubuntu Server 24.04 ISO, 8 GB RAM total, 80 GB disk.

1. Clone this repository
2. Follow [Sprints.md](Sprints.md) from Sprint 1 through Sprint 4
3. Use [Cheat_Sheet.md](Cheat_Sheet.md) for quick command reference

### AWS Cloud (Sprint 5)

**Prerequisites:** AWS Academy account, `ubuntu-server.pem` key pair, FreeRDP installed locally.

1. Launch Ubuntu Server 24.04 and Windows Server 2025 instances in the same VPC
2. Assign Elastic IPs to both instances
3. Configure LAB05-SG security group with AD ports
4. SSH into Ubuntu and follow the Samba installation steps from Sprint 1
5. Set Windows DNS to Ubuntu's private IP, then join the domain
6. Connect via RDP: `xfreerdp /v:54.221.100.222 /u:Administrator /p:'...' /cert:ignore /dynamic-resolution /clipboard`

## Documentation

| Document | Description |
|----------|-------------|
| [Sprints.md](Sprints.md) | Full step-by-step implementation guide (all sprints + client appendices) |
| [Cheat_Sheet.md](Cheat_Sheet.md) | Command reference, troubleshooting, AWS quick reference |

## Key Services

| Port | Service | Protocol |
|------|---------|----------|
| 22 | SSH | TCP |
| 53 | DNS | TCP/UDP |
| 88 | Kerberos | TCP/UDP |
| 389 | LDAP | TCP |
| 445 | SMB/CIFS | TCP |
| 636 | LDAPS | TCP |
| 3268 | Global Catalog | TCP |
| 3389 | RDP (AWS) | TCP |

## Technologies

Samba 4 · Ubuntu Server 24.04 · Windows Server 2025 · AWS EC2 · Kerberos · LDAP · SSSD · Realmd · ACLs · Netplan · Cron · FreeRDP

## Project Statistics

- **Lines of Configuration**: 600+
- **Commands Documented**: 180+
- **Services Configured**: 8 (DNS, Kerberos, LDAP, LDAPS, SMB, Global Catalog, SSH, RDP)
- **Total Implementation Time**: ~31 hours
- **Documentation Pages**: 70+
- **Screenshots**: 49

## Contributing

This is an educational project. Contributions, suggestions, and improvements are welcome!

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/improvement`)
3. Commit your changes (`git commit -am 'Add new feature'`)
4. Push to the branch (`git push origin feature/improvement`)
5. Open a Pull Request

## Acknowledgments

- **Samba Team** — For creating and maintaining Samba 4
- **Ubuntu Community** — For excellent documentation and support
- **SSSD/Realmd Developers** — For seamless Linux AD integration
- **AWS Academy** — For providing the cloud lab environment

## Related Resources

- [Samba Official Wiki](https://wiki.samba.org/)
- [Samba AD DC HOWTO](https://wiki.samba.org/index.php/Setting_up_Samba_as_an_Active_Directory_Domain_Controller)
- [Ubuntu Server Guide](https://ubuntu.com/server/docs)
- [Active Directory Documentation (Microsoft)](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview)
- [SSSD Documentation](https://sssd.io/)
- [FreeRDP](https://www.freerdp.com/)

---

**Project Status**: ✅ Complete 

## License

MIT License — see [LICENSE](LICENSE) for details.

---

*Last Updated: February 2026*
