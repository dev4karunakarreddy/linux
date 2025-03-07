## Troubleshooting SSH Connection Issues

### 1️⃣ Check if SSH is Running on the Remote Server
Run this command on `dc.ad.test` (or access it directly via another method):
```sh
sudo systemctl status sshd
```
If it's inactive, start it:
```sh
sudo systemctl start sshd
```
Enable it on boot:
```sh
sudo systemctl enable sshd
```

### 2️⃣ Verify SSH is Listening on Port 22
On `dc.ad.test`, check if SSH is listening on port 22:
```sh
sudo netstat -tulpn | grep ssh
```
or
```sh
sudo ss -tulpn | grep 22
```
If it's listening on another port (e.g., `2222`), update your SSH command:
```sh
ssh -p 2222 Administrator@dc.ad.test
```

### 3️⃣ Check Firewall Rules
On `dc.ad.test`, ensure SSH traffic is allowed:
```sh
sudo firewall-cmd --list-all
```
If SSH is not listed, allow it:
```sh
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --reload
```

### 4️⃣ Check SELinux (if applicable)
On `dc.ad.test`, run:
```sh
sestatus
```
If it's enforcing, temporarily disable it to test:
```sh
sudo setenforce 0
```
If SSH works afterward, you'll need to permanently allow SSH in SELinux:
```sh
sudo semanage port -a -t ssh_port_t -p tcp 22
```

### 5️⃣ Test Connection via IP
From your Fedora machine, try connecting directly via IP:
```sh
ssh Administrator@192.168.32.133
```

