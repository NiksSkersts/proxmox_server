Proxmox Server
=========

Configures Proxmox server

Requirements
------------

```
collections:
  - name: 'community.general'
    version: '*'
```

Role Variables
--------------

# EXAMPLE VARIABLES

```
# SECTION OS configuration
os_update: false
pkgs:
  - python3-netaddr
  - python3-proxmoxer
  - gnupg
  - sshpass
  - rsync
  - ipmitool

# SECTION User and token setup
    token_save_to_disk: false
    token_save_location: ~/
    proxmox_users:
      - zabbix:
        userid: zabbix@pam
        email: email@email.com
        acl:
          - path: /
            role: PVEAuditor
          - path: /storage
            role: PVEAuditor
          - path: /vms
            role: PVEAuditor
    proxmox_tokens:
      - token0:
        secret: A_SECRET
        userid: zabbix@pam
        acl:
          - path: /
            role: PVEAuditor
          - path: /storage
            role: PVEAuditor
          - path: /vms
            role: PVEAuditor

# SECTION Configure alerting
    disable_mail_to_root: false
    disable_default_matcher: false
    remove_previous: false
    alerting_smtp:
      - test:
        name: NAME
        author: "NAME"
        email_from: email@email.com
        server: "192.168.1.1"
        mailto: root@pam
    alerting_matchers:
      - default-matcher:
        name: "backup-failures"
        comment: "Send notifications about backup failures"
        match: "exact:type=vzdump"
        severity: "error"
        mode: all
        target: NAME

# SECTION Creating Snippets, LXCs, and VMs
    retry_count: 5
    search_domain: example.com
    vm_archive_name: debian-12-generic-amd64.tar.xz
    snippet_definitions:
      - ci-user-1:
        src: aaa/bbb/ci-user-1.j2
        dest: /var/lib/vz/snippets/ci-user-1.yaml
      - ci-vendor: 
        src: aaa/bbb/ci-vendor.j2
        dest: /var/lib/vz/snippets/ci-vendor.yaml
    lxc_definitions:
     vm_definitions:
       - docker: 
         # MAIN VARS
         remove_existing: false
         name: docker
         description: |
           HA Docker VM for services that must be up as much as possible.
         id: "1000"
         ip: 192.168.1.2
         template: "https://cloud.debian.org/images/cloud/bookworm/latest/debian-12-generic-amd64.tar.xz"
         template_destination_path: /var/lib/vz/template
         template_checksum: sha512:fba0dd15297d8cb13efce72c608ff2e5af36682d6fb43b121aca2e72453742cac1c728982b99469a969eaef79fb8ccaf9839380afd1ae525e3a8068504ce6700 
         # CLOUD CONFIG VARS
         ci_update:
         ci_user: 
         ci_pass:
         ci_search_domain: "{{ search_domain }}"
         ci_dns:
         ci_ssh:
         ci_custom:
         # VM VARS
         agent: 1
         onboot: true
         bios: 'ovmf'
         cpu_cores: 4
         vm_memory: 2048
         efidisk:
           storage: 'local-zfs'
           format: raw
           efitype: 4m
           pre_enrolled_keys: 1  # Set this to 0 to disable Secure Boot
         cloudinit:
           ide2: "local-zfs:cloudinit,format=raw"
         drives:
           scsi0: "local-zfs:0,import-from=/root/files/disk.raw,format=raw,iothread=1"
         machine: 'q35'
         scsihw: 'virtio-scsi-single'
         tmp:
           storage: 'local-zfs'
         ostype: "l26"
         serial:
           serial0: 'socket'
         vga: "serial0"
         # DISK VARS
         diskn:
           - id: 1
             disk_key: scsi0
             disk_backup: "true"
             replicate: "true"
             disk_discard: "on"
             disk_dest_pool: "data"
             disk_size: 64G
         # NET VARS
         dns: 192.168.100.1
         netn:
           - id: 0
             bridge: "vmbr0"
             ip: "{{ ip }}"
             gw: 192.168.1.1
             subnet: 24
             mtu: 1
             tag: 1
# SECTION Deploy custom scripts
      custom_scripts:
        - 0:
          cmd: 'aaa/bbb/create_template.sh'
        - 1:
          cmd: 'aaa/bbb/clone_template.sh'
        - 2:
          cmd: 'aaa/bbb/create_ssh.sh'

```

Dependencies
------------

None

Example Playbook
----------------

TODO

License
-------

GPL-2.0-or-later

Author Information
------------------

Authors:
  - Niks Skersts <mail@nikssk.id.lv> , nikssk.id.lv
  - Niks Skersts <niksskersts@rix.tbmarine.group> , SIA TBMarine Shipmanagement (Riga)

