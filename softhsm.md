# SoftHSM Authentication with IPA on RHEL 8

## Overview
This document provides step-by-step instructions to set up an Identity, Policy, and Audit (IPA) environment on RHEL 8 for testing authentication using SoftHSM. The setup consists of an IPA Server and an IPA Client, both running on separate virtual machines.

---

## Prerequisites
### System Requirements
- Two Virtual Machines (VMs):
  - **IPA Server**
  - **IPA Client**
- Each VM should have:
  - **3GB RAM**, **2 CPUs**, **10GB disk space**
  - **Static IP configuration**
  - **RHEL 8 installed**

### Software Requirements
- RHEL 8.2 or later
- `freeipa-server` and `freeipa-client` packages
- `openssl` for certificate management
- `softhsm` for virtual smart card configuration
- `firewalld` for managing firewall rules

---

## 1. Setting Up the Virtual Machines

### Configure Hostnames and IP Addresses
#### On IPA Server:
```bash
hostnamectl set-hostname server.ipa.test
nmcli conn modify ens3 ipv4.addresses 192.168.122.231/24
reboot
```

#### On IPA Client:
```bash
hostnamectl set-hostname client.ipa.test
nmcli conn modify ens3 ipv4.addresses 192.168.122.232/24
reboot
```

### Configure YUM Repositories
```bash
cat > /etc/yum.repos.d/rhel-8.repo << EOF
[rhel-8.2.0-latest-appstream]
name=rhel-8.2.0-latest-appstream
baseurl=http://download-node-02.eng.bos.redhat.com/rhel-8/nightly/RHEL-8/latest-RHEL-8.2.0/compose/AppStream/x86_64/os/
enabled=1
gpgcheck=0
skip_if_unavailable=1
EOF
```

---

## 2. Install and Configure IPA Server
```bash
dnf -y module enable idm:DL1
dnf -y module install idm:DL1/dns
ipa-server-install --setup-dns --auto-forwarders --no-reverse --domain ipa.test --realm IPA.TEST --admin-password Secret123 --ds-password Secret123 --mkhomedir --unattended
```

### Configure Firewall
```bash
firewall-cmd --permanent --add-service=ntp
firewall-cmd --permanent --add-service=dns
firewall-cmd --permanent --add-service=freeipa-ldap
firewall-cmd --permanent --add-service=freeipa-ldaps
firewall-cmd --permanent --add-service=freeipa-trust
firewall-cmd --reload
systemctl restart firewalld.service
```

---

## 3. Install and Configure IPA Client
```bash
dnf -y module enable idm:client
dnf -y module install idm:client
dnf -y group install Workstation
systemctl set-default graphical.target
reboot
```

### Join IPA Client to IPA Server
```bash
ipa-client-install --principal admin --password Secret123 --mkhomedir --unattended
```

---

## 4. Configure SoftHSM Authentication

### Install SoftHSM
```bash
dnf install -y softhsm
```

### Initialize SoftHSM Token
```bash
softhsm2-util --init-token --label "IPA_Token" --so-pin 1234 --pin 5678
```

### Verify SoftHSM Setup
```bash
softhsm2-util --show-slots
```

---

## 5. Create a User and Generate Certificate
```bash
ipa user-add ipauser1 --first=ipa --last=user1 --email=ipauser1@ipa.test --password
openssl req -new -newkey rsa:2048 -keyout ipauser1.key -nodes -out ipauser1.csr -subj '/CN=ipauser1'
ipa cert-request ipauser1.csr --principal=ipauser1 --certificate-out=ipauser1.crt
```

---

## 6. Store Key and Certificate in SoftHSM
```bash
pkcs11-tool --module /usr/lib64/pkcs11/libsofthsm2.so -l --pin 5678 --write-object ipauser1.crt --type cert --id 01
pkcs11-tool --module /usr/lib64/pkcs11/libsofthsm2.so -l --pin 5678 --write-object ipauser1.key --type privkey --id 01
```

### Verify Stored Objects
```bash
pkcs11-tool --module /usr/lib64/pkcs11/libsofthsm2.so -l --pin 5678 --list-objects
```

---

## 7. Test Authentication with SoftHSM

### Export PKCS#11 Module Path
```bash
export PKCS11_MODULE=/usr/lib64/pkcs11/libsofthsm2.so
```

### Authenticate Using SoftHSM
```bash
kinit -X X509_user_identity=PKCS11:${PKCS11_MODULE} ipauser1
```

### Verify Authentication
```bash
klist
```

---

## 8. Troubleshooting

### Check SoftHSM Token Information
```bash
softhsm2-util --show-slots
```

### Increase SSSD Timeouts
Modify `/etc/sssd/sssd.conf`:
```bash
krb5_auth_timeout = 60
p11_child_timeout = 60
```
Restart SSSD:
```bash
systemctl stop sssd
rm -rf /var/lib/sss/{db,mc}/*
systemctl start sssd
```

---

## 9. Additional References
- [RHEL Smart Card Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/linux_domain_identity_authentication_and_policy_guide/smart-cards)
- [SSSD Smart Card Support](https://access.redhat.com/articles/4253861)

