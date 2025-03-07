# Smart Card Bug Testing Guide

## Overview

This document outlines the steps to reproduce and investigate a smart card authentication bug described in [RHEL-4981](https://issues.redhat.com/browse/RHEL-4981). It builds upon an existing test case in the SSSD repository and leverages Red Hat's IDM CI infrastructure.

## Objective

Validate smart card authentication with soft OCSP configuration under network failure. If the client prompts for a password instead of a PIN under these conditions, the bug is present.

## Resources

* JIRA: [RHEL-4981](https://issues.redhat.com/browse/RHEL-4981)
* PR Comment: [SSSD PR #7890](https://github.com/SSSD/sssd/pull/7890#issuecomment-2847810271)
* IDM-CI Guide: [IDM CI User Guide](https://docs-idmci.psi.redhat.com/user_docs/guide.html#idmci_container)

## High-Level Steps

1. Setup IPA server and client
2. Configure systems for smart card authentication using `ipa-advise`
3. Enable virtual smart cards on the client
4. Generate a key and certificate via IPA
5. Confirm normal smart card authentication
6. Add `[sssd] certificate_verification = soft_ocsp` to `sssd.conf`
7. Add incorrect, unpingable IP for `ipa-ca.<domain.test>` in `/etc/hosts` on the client
8. Attempt authentication with the smart card

## Expected Behavior

Step 8 should prompt for a PIN. If a password prompt appears, the bug is triggered.

## Test Environment Setup (Using IDM-CI)

### Container Setup

```bash
podman pull quay.io/idmops/idm-ci:latest

podman --runtime /usr/bin/crun run --name idm_ci -it \
  -v /home/spoore/container_home/idm-ci/tews:/home/jenkins/tews:Z \
  -v /home/spoore/.config/openstack/clouds.yaml:/root/.config/openstack/clouds.yaml:Z \
  quay.io/idmops/idm-ci:latest /bin/bash
```

### Configuration File

Create `clouds.yaml` under `~/.config/openstack/` using the IDM-CI documentation.

## Metadata and Test Setup

```bash
mkdir /home/jenkins/tews/smartcard_test
cd /home/jenkins/tews/smartcard_test
wget https://gitlab.cee.redhat.com/spoore/smartcard/-/raw/RHEL10.0/metadata/ipa_smartcard_auth_virtcacard_tier1_2.yaml

kinit <kerberos-id>@REDHAT.COM
te ipa_smartcard_auth_virtcacard_tier1_2.yaml --upto prep
```

## Running the Test

From the smartcard test directory:

```bash
pytest -vsx \
  --no_ad \
  --pdb \
  --cardtype='virtcacard' \
  --multihost-config=/home/jenkins/tews/smartcard_test/config/pytest-multihost-config.yaml \
  -m 'tier1_2 and not ad' test_4001_misc.py::TestSssdSoftFailOCSP::test_0004
```

Insert `pytest.set_trace()` before the call to `chk_journal_after_auth()` in the test to pause execution for manual inspection.

## Notes

* Steps 1–4 are automated using IDM-CI and metadata from `ipa_smartcard_auth_virtcacard_tier1_2.yaml`
* Step 6 is already part of the test configuration
* Modify the existing test case to simulate the incorrect IP entry for `ipa-ca.<domain.test>`

## Follow-up

Try setting this up as described. If issues arise, reach out for clarification or a copy of the necessary `clouds.yaml` configuration.
