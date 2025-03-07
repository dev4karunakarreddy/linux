# Setting up an IPA Environment for Smart Card Authentication

## Basic Setup with IPA Server and Client

### Create Two VMs (RHEL8 Example)
1. Use a virtualization manager to create two VMs: one for the IPA Server and one for the IPA Client.
2. Assign resources:
   - Memory: 3GB
   - CPUs: 2
   - Disk: 10GB
3. Set up hostnames and IP addresses:
   - IPA Server:
     ```bash
     hostnamectl set-hostname server.ipa.test
     nmcli conn modify ens3 ipv4.addresses 192.168.122.231/24
     reboot
     ```
   - IPA Client:
     ```bash
     hostnamectl set-hostname client.ipa.test
     nmcli conn modify ens3 ipv4.addresses 192.168.122.232/24
     reboot
     ```

### Configure Repositories
```bash
cat > /etc/yum.repos.d/rhel-8.2.0-latest-appstream.repo << EOF
[rhel-8.2.0-latest-appstream]
name=rhel-8.2.0-latest-appstream
baseurl=http://download-node-02.eng.bos.redhat.com/rhel-8/nightly/RHEL-8/latest-RHEL-8.2.0/compose/AppStream/x86_64/os/
enabled=1
gpgcheck=0
skip_if_unavailable=1
EOF
```

```bash
cat > /etc/yum.repos.d/rhel-8.2.0-latest-baseos.repo << EOF
[rhel-8.2.0-latest-baseos]
name=rhel-8.2.0-latest-baseos
baseurl=http://download-node-02.eng.bos.redhat.com/rhel-8/nightly/RHEL-8/latest-RHEL-8.2.0/compose/BaseOS/x86_64/os/
enabled=1
gpgcheck=0
skip_if_unavailable=1
EOF
```

## Install IPA Server
```bash
dnf -y module enable idm:DL1
dnf -y module install idm:DL1/dns
ipa-server-install --setup-dns --auto-forwarders --no-reverse --domain ipa.test --realm IPA.TEST --admin-password Secret123 --ds-password Secret123 --mkhomedir --unattended
firewall-cmd --permanent --add-service={ntp,dns,freeipa-ldap,freeipa-ldaps,freeipa-trust}
firewall-cmd --reload
systemctl restart firewalld.service
```

## Install IPA Client
```bash
dnf -y module enable idm:client
dnf -y module install idm:client
dnf -y group install Workstation
systemctl set-default graphical.target
reboot
ipa-client-install --principal admin --password Secret123 --mkhomedir --unattended
```

## Run ipa-advise Scripts
```bash
ipa-advise config-server-for-smart-card-auth > server.sh
ipa-advise config-client-for-smart-card-auth > client.sh
kinit admin
sh -x server.sh /etc/ipa/ca.crt
scp client.sh client.ipa.test:/tmp/client.sh
kinit admin
sh -x /tmp/client.sh /etc/ipa/ca.crt
```

## Create User and Certificate
```bash
kinit admin
ipa user-add ipauser1 --first=ipa --last=user1 --email=ipauser1@ipa.test --password
openssl req -new -newkey rsa:2048 -keyout ipauser1.key -nodes -out ipauser1.csr -subj '/CN=ipauser1'
ipa cert-request ipauser1.csr --principal=ipauser1 --certificate-out=ipauser1.crt
```

## Write Key and Cert to PKCS#15 Card
```bash
pkcs15-init --erase-card --use-default-transport-keys
pkcs15-init --create-pkcs15 --use-default-transport-keys --pin redhat --puk redhat --so-pin redhat --so-puk redhat
pkcs15-init --store-pin --auth-id 01 --label sctest --so-pin redhat --pin redhat --puk redhat
pkcs15-init --store-private-key ipauser1.key --auth-id 01 --id 01 --so-pin redhat --pin redhat
pkcs15-init --store-certificate ipauser1.crt --auth-id 01 --id 01 --format pem --so-pin redhat --pin redhat
pkcs15-init -F
```

## Authenticate with Smart Card
```bash
su - ipauser1 -c 'su - ipauser1 -c whoami'
```

## Increase SSSD Timeouts
```bash
vim /etc/sssd/sssd.conf
# Add the following lines
[domain]
krb5_auth_timeout = 60

[pam]
p11_child_timeout = 60
systemctl stop sssd; rm -rf /var/lib/sss/{db,mc}/*; systemctl start sssd
```

## Check CA Bundles
```bash
openssl crl2pkcs7 -nocrl -certfile /var/lib/ipa-client/pki/ca-bundle.pem | openssl pkcs7 -print_certs -text -noout | egrep 'Issuer:|Subject:'
```

## Troubleshooting
- **Access the Card:**
  ```bash
  certutil -d /etc/pki/nssdb -L -h all
  ```
- **Check Kerberos:**
  ```bash
  kinit -X X509_user_identity=PKCS11:module_name=/usr/lib64/opensc-pkcs11.so ipauser1
  ```
- **Resolve OCSP Issues:**
  ```bash
  p11tool --provider /usr/lib64/opensc-pkcs11.so --list-all-certs
  kinit -X X509_user_identity=PKCS11:module_name=/usr/lib64/opensc-pkcs11.so:certid=01 ipauser1
  ```

## Adding AD Trust
For integrating IPA with Active Directory:
- Setup Active Directory and Certificate Server
- Follow official guides from Red Hat

## References
- [RHEL 7 Smart Card Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/linux_domain_identity_authentication_and_policy_guide/smart-cards)

