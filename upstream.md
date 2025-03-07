## Smartcard Authentication Setup Notes

### Environment Setup

```bash
mkdir smartcard_upstream
pushd smartcard_upstream
python -m venv .venv
source .venv/bin/activate
```

### Clone and Configure sssd-ci-containers

```bash
git clone -b build_ansible_opts https://github.com/spoore1/sssd-ci-containers
pushd sssd-ci-containers

cat > .env <<EOF
REGISTRY=localhost/sssd
TAG=fedora-42
EOF

sudo make down
sudo make build BASE_IMAGE="registry.fedoraproject.org/fedora:42-x86_64" TAG="fedora-42" \
    ANSIBLE_OPTS="--extra-vars virt_smartcard=yes"

sudo make up
popd
```

### Clone and Prepare sssd

```bash
git clone -b test-authentication-alternative https://github.com/krishnavema/sssd.git
pushd sssd/src/tests/system
pip install -r requirements.txt
popd
```

### Clone and Prepare sssd-test-framework

```bash
git clone -b smatcard https://github.com/krishnavema/sssd-test-framework.git
pushd sssd-test-framework
pip install -r requirements.txt
```

#### Modify `virtual_smartcard.py`

In `sssd_test_framework/utils/virtual_smartcard.py`, add:

**Imports:**

```python
from typing import TYPE_CHECKING
if TYPE_CHECKING:
    from ..roles.client import Client
```

**New Method:**

```python
def setup_local_card(self, client: Client, username: str) -> None:
    (key, cert) = self.generate_self_signed_cert()
    self.init()
    self.add_key(key)
    self.add_cert(cert)
    self.reset_service()

    client.sssd.sssd["domains"] = "local"
    client.sssd.common.local()
    client.sssd.dom("local")["local_auth_policy"] = "only"
    client.sssd.section(f"certmap/local/{username}")["matchrule"] = "<SUBJECT>.*CN=Test Cert*"
    client.sssd.pam["pam_cert_auth"] = "True"
    client.host.conn.run("echo >> /etc/sssd/pki/sssd_auth_ca_db.pem")
    client.host.conn.run(f"cat {cert} >> /etc/sssd/pki/sssd_auth_ca_db.pem")
    client.sssd.start()

    client.local.user(f"{username}").add()
```

```bash
pip install .
popd
```

### Run Pytest

```bash
pushd sssd/src/tests/system
pytest --mh-config=mhc.yaml --log-file=./pytest-run.log --log-file-level=INFO --log-file-format='%(asctime)s [%(name)s] %(levelname)s %(message)s' --log-file-date-format='%Y-%m-%dT%H:%M:%S%z' -vsx tests/test_smartcard.py -k test_smart_card_setup
```
