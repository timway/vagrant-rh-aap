# Vagrant Powered AAP Lab Environment

We can use Vagrant to provision a fully fledged AAP environment. This build
is tested in Fedora using libvirt images. The vagrant-registration plugin is
used to automatically manage the licensing of the underlying RHEL VMs.

## Handling .env file

To import the values stored in a .env file run this command:

```bash
export $(cat .env | xargs)
```

The Vagrantfile requires a number of environment variables to get AAP up and
running correctly. The starting point is:

```bash
SPONGE_RHSM_USERNAME=""
SPONGE_RHSM_PASSWORD=""
SPONGE_RHSM_POOL_BASE=""
SPONGE_RHSM_POOL_AAP=""
SPONGE_AAP_CONTROLLER_ADMIN_PASSWORD=""
SPONGE_AAP_CONTROLLER_PG_PASSWORD=""
SPONGE_AAP_REGISTRY_USERNAME=""
SPONGE_AAP_REGISTRY_PASSWORD=""
SPONGE_AAP_HUB_ADMIN_PASSWORD=""
SPONGE_AAP_HUB_PG_PASSWORD=""
SPONGE_AAP_SSO_ADMIN_PASSWORD=""
SPONGE_AAP_SSO_KEYSTORE_PASSWORD=""
```

## The Ansible Automation Platform Installer Collection

Current the installer is not publically available in a way that can directly be
installed via `ansible-galaxy`. If you download and extract the online
installer from Red Hat's web-site the collection can be re-packaged from there.

```bash
tar cJvf ansible-automation_platform_installer-2.2.1-1.tar.xz \
    --directory ansible-automation-platform-setup-2.2.1-1/collections/ansible_collections/ansible/automation_platform_installer \
    FILES.json \
    LICENSE \
    MANIFEST.json \
    Makefile \
    README.md \
    meta \
    playbooks \
    plugins \
    roles
```