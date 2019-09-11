Trells CA Certificates
=========

Trellis Ansible Role to add custom CA Certficates to OpenSSL's Trusted Store. Perfect for using with [Digital Ocean Managed Databases](https://m.do.co/c/fc096f22997f) which require SSL/TLS connections for MySQL.

Requirements
------------

- [Trellis](https://github.com/roots/trellis)
- Ubuntu 18.04

Installation
------------

### Add to requirements

```yaml
# trellis/galaxy.yml
- name: trellis_ca_certificates
  src: https://github.com/craigpearson/trellis-ca-certificates
  version: develop
```

### Include in provision playbook

To add this functionality on your staging/production servers include this role in `trellis/server.yml`

```yaml
# trellis/server.yml
...
- name: WordPress Server - Install LEMP Stack with PHP 7.3 and MariaDB MySQL
  hosts: web:&{{ env }}
  become: yes
  roles:
    - { role: common, tags: [common] }
    ...
    - { role: sshd, tags: [sshd] }
    - { role: trellis-ca-certificates, tags: [ca-certificates] } # Recommended inclusion point
    - { role: mariadb, tags: [mariadb] }
```

If you require this functionality on your development server you should also add this to `trellis/dev.yml`

### Set certificates to include

In production we would add to `trellis/group_vars/production/main.yml`

```yaml
# trellis/group_vars/production/main.yml
trellis_ca_certificates_trusted:
  # Local Source:         trellis/certs/production/example-certificate.crt
  # Remote Destination:   /usr/local/share/ca-certificates/database.crt
  - name: database
    src: example-certificate.crt
```

### Include .crt files

By default this role looks in `trellis/certs` for your certificates, example:

```shell
trellis/
├── bin/
├── certs/                            # → Source certificates folder
│   ├── development/                  # → Development certificates
│   ├── staging/                      # → Staging certificates
│   └── production/                   # → Production certificates
│       └── example-certificate.crt
└── deploy-hooks/
```

Now all that's left is to provision

Role Variables
--------------

### Configure custom certificates

The only variable required for configuration is the certificate trusted source list. This should be placed in `trellis/group_vars/{{ env }}/main.yml` where `{{ env }}` is development, staging, or production.

**Note:** Your source certificate is renamed and placed in the destination as specified in `name`, example:

```yaml
# This is an example of possible values, this value defaults to []
trellis_ca_certificates_trusted:
  # Local Source:         trellis/certs/{{ env }}/example-certificate.crt
  # Remote Destination:   /usr/local/share/ca-certificates/database.crt
  - name: database
    src: example-certificate.crt
  # Local Source:         trellis/certs/{{ env }}/db-master.crt
  # Remote Destination:   /usr/local/share/ca-certificates/database/master.crt
  - name: database/master
  - src: db-master.crt
  # Local Source:         trellis/certs/{{ env }}/db-slave.crt
  # Remote Destination:   /usr/local/share/ca-certificates/database/slave.crt
  - name: database/slave
  - src: db-slave.crt
```

### Source directory
To store your certificate files in a folder other than `trellis/certs/{{ env }}`:

```yaml
# Defaults to trellis/certs/env - where env is development, staging or production
trellis_ca_certificates_local_dir: custom-local-directory/{{ env }}/
```

### Remote destination directory
Unless you have configured your remote OpenSSL to look for certs in a different directory you shouldn't need to change this

```yaml
# Defaults to OpenSSL Trusted store on Ubuntu 18.04
trellis_ca_certificates_local_dir: /usr/local/share/ca-certificates
```
