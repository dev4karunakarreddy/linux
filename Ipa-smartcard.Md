# Smart Card Authentication with IPA on RHEL 8

## Overview
This document provides step-by-step instructions to set up an Identity, Policy, and Audit (IPA) environment on RHEL 8 for testing smart card authentication. The setup consists of an IPA Server and an IPA Client, both running on separate virtual machines.

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
- `pkcs15-init` for smart card configuration
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

## 4. Configure Smart Card Authentication

### Generate and Execute IPA Configuration Scripts
#### On IPA Server:
```bash
ipa-advise config-server-for-smart-card-auth > server.sh
ipa-advise config-client-for-smart-card-auth > client.sh
kinit admin
sh -x server.sh /etc/ipa/ca.crt
```

#### Copy and Run on IPA Client:
```bash
scp client.sh client.ipa.test:/tmp/client.sh
ssh client.ipa.test "kinit admin && sh -x /tmp/client.sh /etc/ipa/ca.crt"
```

---

## 5. Create a User and Generate Certificate
```bash
ipa user-add ipauser1 --first=ipa --last=user1 --email=ipauser1@ipa.test --password
openssl req -new -newkey rsa:2048 -keyout ipauser1.key -nodes -out ipauser1.csr -subj '/CN=ipauser1'
ipa cert-request ipauser1.csr --principal=ipauser1 --certificate-out=ipauser1.crt
```

---

## 6. Write Key and Certificate to Smart Card
```bash
pkcs15-init --erase-card --use-default-transport-keys
pkcs15-init --create-pkcs15 --use-default-transport-keys --pin redhat --puk redhat --so-pin redhat --so-puk redhat
pkcs15-init --store-pin --auth-id 01 --label sctest --so-pin redhat --pin redhat --puk redhat
pkcs15-init --store-private-key ipauser1.key --auth-id 01 --id 01 --so-pin redhat --pin redhat
pkcs15-init --store-certificate ipauser1.crt --auth-id 01 --id 01 --format pem --so-pin redhat --pin redhat
```

---

## 7. Test Smart Card Authentication
```bash
su - ipauser1 -c 'whoami'
kinit -X X509_user_identity=PKCS11:module_name=/usr/lib64/opensc-pkcs11.so ipauser1
```

---

## 8. Troubleshooting
### Check Smart Card Detection
```bash
certutil -d /etc/pki/nssdb -L -h all
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

