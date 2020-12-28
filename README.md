# A simple role to use the EntryDNS API to answer ACME challenges

This role is quite simple (just like it's creator!) and simply answers a single-common-name ACME challenge by registering a single TXT record with the excellent DNS provider [EntryDNS](https://entrydns.net).

## Requirements

The role uses the Ansible Galaxy community crypto collection, so you will need to install that, first:

`$ ansible-galaxy collection install community.crypto`

## Usage

This role requires only a few things to work properly. First, you have to manually create the TXT record that you will use for validation. Unfortunately, EntryDNS does not support API-based record creation, only updating. See [EntryDNS Help](https://entrydns.net/help).

At a minimum, define these variables:

 - acme_email       | your email address
 - dns_record_token | the update token for the record that you wish to use, in entryDNS
 - domain_name      | the domain you wish to issue an SSL certificate for
 - cert_cn          | the common name you wish to issue a cert for (often www.{{ domain_name }})

By default, certificates are issued by Let's Encrypt's staging environment, so you will need to change to the production environment, to get valid certificates. Set the following:

 - acme_directory   | https://acme-v02.api.letsencrypt.org/directory

Alternately, you may use any other ACME provder supported by the Ansible community module [acme_certificate](https://docs.ansible.com/ansible/latest/collections/community/crypto/acme_certificate_module.html).

## Example main.yml

```
---

- hosts: localhost
  become: yes

  vars:
    - acme_email: "john.doe@email.com"
    - dns_record_token: "12345ABCDE67890fghij"
    - domain_name: "example.com"
    - cert_cn: "www.example.com"
    - acme_directory: "https://acme-v02.api.letsencrypt.org/directory"

  roles:
    - acme-entrydns

...
```

Optionally, you can retarget the location of the certificate storage, from the default of `/etc/letsencrypt`, and change the ownership, from the default of `root:root`, with something like:

```
  vars:
    - letsencrypt_dir: "~user/letsencrypt"
    - letsencrypt_dir_owner: user
    - letsencrypt_dir_group: user
```

Without any tags being specified, a key will be generated, for an ACME account. Then, a key and SSL CSR are created. That CSR is then sent to your ACME provider, and the provider's challenge is answered, using EntryDNS.

If you add the tag `destroy`, the SSL certificate for the common name is revoked, and the cert key and CSR are moved to `{{ letsencrypt_dir }}/*/revoked`. The account key is left in place.

That's about it. Please let me know if you have any issues.

See `defaults/main.yml` for more possible variables to set.
