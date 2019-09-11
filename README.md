Trells CA Certificates
=========

Trellis Ansible Role to add custom CA Certficates to OpenSSL's Trusted Store. Ideal for using alongside [Digital Ocean Managed Databases](https://m.do.co/c/fc096f22997f) which require SSL/TLS connections for MySQL.

Requirements
------------

This role is specifically built for [Trellis](https://github.com/roots/trellis) - an opinionated WordPress stack developed by [Roots.io](https://roots.io) running on Ubuntu 18.04.

Installation
------------

### Add this role to your requirements

```yaml
# trellis/galaxy.yml
- name: trellis_ca_certificates
  src: https://github.com/craigpearson/trellis-ca-certificates
  version: develop
```

### Include this role in your provision tasks

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

### Set the source and remote destination

To copy CA Certificates to the production server and trust them we would add the following to `trellis/group_vars/production/main.yml`

```yaml
# trellis/group_vars/production/main.yml
trellis_ca_certificates_trusted:
  # Local Source:         trellis/certs/production/example-certificate.crt
  # Remote Destination:   /usr/local/share/ca-certificates/database.crt
  - name: database
    src: example-certificate.crt
```

### Add your .crt files to the relevant directory

By default this role looks in `trellis/certs` for your certificates, in the above example our directory structure would look like:

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

### Configure custom certificates to install

The only variable required for configuration is the certificate trusted source list. This should be placed in `trellis/group_vars/{{ env }}/main.yml` where `{{ env }}` is development, staging or production.

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

### Source directory for `.crt` files
If you wish to store your certificate files in a folder other than `trellis/certs/{{ env }}` you can do that like so:

```yaml
# Defaults to trellis/certs/env - where env is development, staging or production
trellis_ca_certificates_local_dir: source-certificate-director/{{ env }}/
```

### Remote destination directory for `.crt` files
Unless you have configured OpenSSL to look for certs in a different directory you shouldn't need to change this

```yaml
# Defaults to OpenSSL Trusted store on Ubuntu 18.04
trellis_ca_certificates_local_dir: /usr/local/share/ca-certificates
```

### Handler for updating target machines
After the `.crt` file is copied a command to update trusted certificates is ran. Unless you're using a custom certificate store other than the default you shouldn't need to change this.
```yaml
# Command to run when updating trusted certificate store
trellis_ca_certificates_local_dir: /usr/sbin/update-ca-certificates
```
