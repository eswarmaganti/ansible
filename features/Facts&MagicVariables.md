# Discovering variables: facts and magic variables
- With ansible you can retrieve or discover certain variables containing information about your remote systems or about ansible itself.
- Variables related to remote systems are called *facts*. With facts, you can use the behaviour or state of one system as a configuration on other systems.
- Variables related to ansible are called `magic variables`.

## Ansible facts
- Ansible facts are data realted to your remote systems, including operating systems, IP addresses, attached filesystems, and more.
- You can access this data in the `ansible_facts` variable. By default, you can also access some Ansible facts as top-level variables with the `ansible_` prefix.
- You can disable this behaviour using the `INJECT_FACTS_AS_VARS` settings. 
- To see all available facts, we can run this playbook
  - `ansible-playbook -i inventory/hosts.yml playbooks/get_facts.yml`

    ```yaml
    ---
    - name: Playbook to get the facts from remote system
      hosts: vagrantubuntulab0101
      gather_facts: yes
      tasks:
        - name: Print all available facts
          ansible.builtin.debug:
            var: ansible_facts
    ```

- Facts include a large amount of variable data, which may look like this:
    ```json
    "ansible_facts": {
            "all_ipv4_addresses": [
                "192.168.58.101",
                "10.0.2.15"
            ],
            "all_ipv6_addresses": [
                "fdeb:4c62:4b88:355d:a00:27ff:fe33:da79",
                "fe80::a00:27ff:fe33:da79",
                "fd17:625c:f037:2:a00:27ff:fed9:fa93",
                "fe80::a00:27ff:fed9:fa93"
            ],
            "ansible_local": {},
            "apparmor": {
                "status": "enabled"
            },
            "architecture": "aarch64",
            "bios_date": "",
            "bios_vendor": "",
            "bios_version": "",
            "board_asset_tag": "",
            "board_name": "",
            "board_serial": "",
            "board_vendor": "",
            "board_version": "",
            "chassis_asset_tag": "",
            "chassis_serial": "",
            "chassis_vendor": "",
            "chassis_version": "",
            "cmdline": {
                "BOOT_IMAGE": "/vmlinuz-5.15.0-160-generic",
                "autoinstall": true,
                "biosdevname": "0",
                "ds": "nocloud-net",
                "net.ifnames": "0",
                "ro": true,
                "root": "/dev/mapper/ubuntu--vg-ubuntu--lv"
            },
            "date_time": {
                "date": "2025-12-20",
                "day": "20",
                "epoch": "1766207372",
                "epoch_int": "1766207372",
                "hour": "05",
                "iso8601": "2025-12-20T05:09:32Z",
                "iso8601_basic": "20251220T050932720417",
                "iso8601_basic_short": "20251220T050932",
                "iso8601_micro": "2025-12-20T05:09:32.720417Z",
                "minute": "09",
                "month": "12",
                "second": "32",
                "time": "05:09:32",
                "tz": "UTC",
                "tz_dst": "UTC",
                "tz_offset": "+0000",
                "weekday": "Saturday",
                "weekday_number": "6",
                "weeknumber": "50",
                "year": "2025"
            },
            "default_ipv4": {
                "address": "10.0.2.15",
                "alias": "eth0",
                "broadcast": "",
                "gateway": "10.0.2.2",
                "interface": "eth0",
                "macaddress": "08:00:27:d9:fa:93",
                "mtu": 1500,
                "netmask": "255.255.255.0",
                "network": "10.0.2.0",
                "prefix": "24",
                "type": "ether"
            },
            "default_ipv6": {
                "address": "fd17:625c:f037:2:a00:27ff:fed9:fa93",
                "gateway": "fe80::2",
                "interface": "eth0",
                "macaddress": "08:00:27:d9:fa:93",
                "mtu": 1500,
                "prefix": "64",
                "scope": "global",
                "type": "ether"
            },
            "device_links": {
                "ids": {
                    "dm-0": [
                        "dm-name-ubuntu--vg-ubuntu--lv",
                        "dm-uuid-LVM-8OmLUnNfx20r8dswbNNxEcbvxB1VNzvcC26GLeEwsYUJNXEMvOgYQJ6Uehlgh1Sb"
                    ],
                    "sda3": [
                        "lvm-pv-uuid-S2nkSD-7RNB-W1sU-M9T4-MM8V-izts-4eSiCA"
                    ]
                },
                "labels": {},
                "masters": {
                    "sda3": [
                        "dm-0"
                    ]
                },
                "uuids": {
                    "dm-0": [
                        "2bf8029c-4c42-4681-8997-5f2f6c680ea0"
                    ],
                    "sda1": [
                        "3D4A-E5A1"
                    ],
                    "sda2": [
                        "64a2d46d-6818-4f02-b510-a68cb9e0b085"
                    ]
                }
            },
            "devices": {
                "dm-0": {
                    "holders": [],
                    "host": "",
                    "links": {
                        "ids": [
                            "dm-name-ubuntu--vg-ubuntu--lv",
                            "dm-uuid-LVM-8OmLUnNfx20r8dswbNNxEcbvxB1VNzvcC26GLeEwsYUJNXEMvOgYQJ6Uehlgh1Sb"
                        ],
                        "labels": [],
                        "masters": [],
                        "uuids": [
                            "2bf8029c-4c42-4681-8997-5f2f6c680ea0"
                        ]
                    },
                    "model": null,
                    "partitions": {},
                    "removable": "0",
                    "rotational": "1",
                    "sas_address": null,
                    "sas_device_handle": null,
                    "scheduler_mode": "",
                    "sectors": 63905792,
                    "sectorsize": "512",
                    "size": "30.47 GB",
                    "support_discard": "0",
                    "vendor": null,
                    "virtual": 1
                },
                "loop0": {
                    "holders": [],
                    "host": "",
                    "links": {
                        "ids": [],
                        "labels": [],
                        "masters": [],
                        "uuids": []
                    },
                    "model": null,
                    "partitions": {},
                    "removable": "0",
                    "rotational": "1",
                    "sas_address": null,
                    "sas_device_handle": null,
                    "scheduler_mode": "none",
                    "sectors": 122368,
                    "sectorsize": "512",
                    "size": "59.75 MB",
                    "support_discard": "4096",
                    "vendor": null,
                    "virtual": 1
                },
                "loop1": {
                    "holders": [],
                    "host": "",
                    "links": {
                        "ids": [],
                        "labels": [],
                        "masters": [],
                        "uuids": []
                    },
                    "model": null,
                    "partitions": {},
                    "removable": "0",
                    "rotational": "1",
                    "sas_address": null,
                    "sas_device_handle": null,
                    "scheduler_mode": "none",
                    "sectors": 121984,
                    "sectorsize": "512",
                    "size": "59.56 MB",
                    "support_discard": "4096",
                    "vendor": null,
                    "virtual": 1
                },
                "loop2": {
                    "holders": [],
                    "host": "",
                    "links": {
                        "ids": [],
                        "labels": [],
                        "masters": [],
                        "uuids": []
                    },
                    "model": null,
                    "partitions": {},
                    "removable": "0",
                    "rotational": "1",
                    "sas_address": null,
                    "sas_device_handle": null,
                    "scheduler_mode": "none",
                    "sectors": 69032,
                    "sectorsize": "512",
                    "size": "33.71 MB",
                    "support_discard": "4096",
                    "vendor": null,
                    "virtual": 1
                },
                "loop3": {
                    "holders": [],
                    "host": "",
                    "links": {
                        "ids": [],
                        "labels": [],
                        "masters": [],
                        "uuids": []
                    },
                    "model": null,
                    "partitions": {},
                    "removable": "0",
                    "rotational": "1",
                    "sas_address": null,
                    "sas_device_handle": null,
                    "scheduler_mode": "none",
                    "sectors": 158472,
                    "sectorsize": "512",
                    "size": "77.38 MB",
                    "support_discard": "4096",
                    "vendor": null,
                    "virtual": 1
                },
                "loop4": {
                    "holders": [],
                    "host": "",
                    "links": {
                        "ids": [],
                        "labels": [],
                        "masters": [],
                        "uuids": []
                    },
                    "model": null,
                    "partitions": {},
                    "removable": "0",
                    "rotational": "1",
                    "sas_address": null,
                    "sas_device_handle": null,
                    "scheduler_mode": "none",
                    "sectors": 165952,
                    "sectorsize": "512",
                    "size": "81.03 MB",
                    "support_discard": "4096",
                    "vendor": null,
                    "virtual": 1
                },
                "loop5": {
                    "holders": [],
                    "host": "",
                    "links": {
                        "ids": [],
                        "labels": [],
                        "masters": [],
                        "uuids": []
                    },
                    "model": null,
                    "partitions": {},
                    "removable": "0",
                    "rotational": "1",
                    "sas_address": null,
                    "sas_device_handle": null,
                    "scheduler_mode": "none",
                    "sectors": 90800,
                    "sectorsize": "512",
                    "size": "44.34 MB",
                    "support_discard": "4096",
                    "vendor": null,
                    "virtual": 1
                },
                "loop6": {
                    "holders": [],
                    "host": "",
                    "links": {
                        "ids": [],
                        "labels": [],
                        "masters": [],
                        "uuids": []
                    },
                    "model": null,
                    "partitions": {},
                    "removable": "0",
                    "rotational": "1",
                    "sas_address": null,
                    "sas_device_handle": null,
                    "scheduler_mode": "none",
                    "sectors": 0,
                    "sectorsize": "512",
                    "size": "0.00 Bytes",
                    "support_discard": "4096",
                    "vendor": null,
                    "virtual": 1
                },
                "loop7": {
                    "holders": [],
                    "host": "",
                    "links": {
                        "ids": [],
                        "labels": [],
                        "masters": [],
                        "uuids": []
                    },
                    "model": null,
                    "partitions": {},
                    "removable": "0",
                    "rotational": "1",
                    "sas_address": null,
                    "sas_device_handle": null,
                    "scheduler_mode": "none",
                    "sectors": 0,
                    "sectorsize": "512",
                    "size": "0.00 Bytes",
                    "support_discard": "0",
                    "vendor": null,
                    "virtual": 1
                },
                "sda": {
                    "holders": [],
                    "host": "SCSI storage controller: Red Hat, Inc. Virtio SCSI (rev 01)",
                    "links": {
                        "ids": [],
                        "labels": [],
                        "masters": [],
                        "uuids": []
                    },
                    "model": "HARDDISK",
                    "partitions": {
                        "sda1": {
                            "holders": [],
                            "links": {
                                "ids": [],
                                "labels": [],
                                "masters": [],
                                "uuids": [
                                    "3D4A-E5A1"
                                ]
                            },
                            "sectors": 2201600,
                            "sectorsize": 512,
                            "size": "1.05 GB",
                            "start": "2048",
                            "uuid": "3D4A-E5A1"
                        },
                        "sda2": {
                            "holders": [],
                            "links": {
                                "ids": [],
                                "labels": [],
                                "masters": [],
                                "uuids": [
                                    "64a2d46d-6818-4f02-b510-a68cb9e0b085"
                                ]
                            },
                            "sectors": 4194304,
                            "sectorsize": 512,
                            "size": "2.00 GB",
                            "start": "2203648",
                            "uuid": "64a2d46d-6818-4f02-b510-a68cb9e0b085"
                        },
                        "sda3": {
                            "holders": [
                                "ubuntu--vg-ubuntu--lv"
                            ],
                            "links": {
                                "ids": [
                                    "lvm-pv-uuid-S2nkSD-7RNB-W1sU-M9T4-MM8V-izts-4eSiCA"
                                ],
                                "labels": [],
                                "masters": [
                                    "dm-0"
                                ],
                                "uuids": []
                            },
                            "sectors": 127817728,
                            "sectorsize": 512,
                            "size": "60.95 GB",
                            "start": "6397952",
                            "uuid": null
                        }
                    },
                    "removable": "0",
                    "rotational": "1",
                    "sas_address": null,
                    "sas_device_handle": null,
                    "scheduler_mode": "none",
                    "sectors": 134217728,
                    "sectorsize": "512",
                    "size": "64.00 GB",
                    "support_discard": "0",
                    "vendor": "VBOX",
                    "virtual": 1
                }
            },
            "distribution": "Ubuntu",
            "distribution_file_parsed": true,
            "distribution_file_path": "/etc/os-release",
            "distribution_file_variety": "Debian",
            "distribution_major_version": "22",
            "distribution_release": "jammy",
            "distribution_version": "22.04",
            "dns": {
                "nameservers": [
                    "127.0.0.53"
                ],
                "options": {
                    "edns0": true,
                    "trust-ad": true
                },
                "search": [
                    "."
                ]
            },
            "domain": "",
            "effective_group_id": 1001,
            "effective_user_id": 1001,
            "env": {
                "DBUS_SESSION_BUS_ADDRESS": "unix:path=/run/user/1001/bus",
                "HOME": "/home/ansible",
                "LANG": "en_US.UTF-8",
                "LOGNAME": "ansible",
                "MOTD_SHOWN": "pam",
                "PATH": "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin",
                "PWD": "/home/ansible",
                "SHELL": "/bin/bash",
                "SHLVL": "0",
                "SSH_CLIENT": "192.168.58.1 58514 22",
                "SSH_CONNECTION": "192.168.58.1 58514 192.168.58.101 22",
                "SSH_TTY": "/dev/pts/0",
                "TERM": "xterm-256color",
                "USER": "ansible",
                "XDG_RUNTIME_DIR": "/run/user/1001",
                "XDG_SESSION_CLASS": "user",
                "XDG_SESSION_ID": "87",
                "XDG_SESSION_TYPE": "tty",
                "_": "/bin/sh"
            },
            "eth0": {
                "active": true,
                "device": "eth0",
                "features": {
                    "esp_hw_offload": "off [fixed]",
                    "esp_tx_csum_hw_offload": "off [fixed]",
                    "fcoe_mtu": "off [fixed]",
                    "generic_receive_offload": "on",
                    "generic_segmentation_offload": "on",
                    "highdma": "off [fixed]",
                    "hsr_dup_offload": "off [fixed]",
                    "hsr_fwd_offload": "off [fixed]",
                    "hsr_tag_ins_offload": "off [fixed]",
                    "hsr_tag_rm_offload": "off [fixed]",
                    "hw_tc_offload": "off [fixed]",
                    "l2_fwd_offload": "off [fixed]",
                    "large_receive_offload": "off [fixed]",
                    "loopback": "off [fixed]",
                    "macsec_hw_offload": "off [fixed]",
                    "netns_local": "off [fixed]",
                    "ntuple_filters": "off [fixed]",
                    "receive_hashing": "off [fixed]",
                    "rx_all": "off",
                    "rx_checksumming": "off",
                    "rx_fcs": "off",
                    "rx_gro_hw": "off [fixed]",
                    "rx_gro_list": "off",
                    "rx_udp_gro_forwarding": "off",
                    "rx_udp_tunnel_port_offload": "off [fixed]",
                    "rx_vlan_filter": "on [fixed]",
                    "rx_vlan_offload": "on",
                    "rx_vlan_stag_filter": "off [fixed]",
                    "rx_vlan_stag_hw_parse": "off [fixed]",
                    "scatter_gather": "on",
                    "tcp_segmentation_offload": "on",
                    "tls_hw_record": "off [fixed]",
                    "tls_hw_rx_offload": "off [fixed]",
                    "tls_hw_tx_offload": "off [fixed]",
                    "tx_checksum_fcoe_crc": "off [fixed]",
                    "tx_checksum_ip_generic": "on",
                    "tx_checksum_ipv4": "off [fixed]",
                    "tx_checksum_ipv6": "off [fixed]",
                    "tx_checksum_sctp": "off [fixed]",
                    "tx_checksumming": "on",
                    "tx_esp_segmentation": "off [fixed]",
                    "tx_fcoe_segmentation": "off [fixed]",
                    "tx_gre_csum_segmentation": "off [fixed]",
                    "tx_gre_segmentation": "off [fixed]",
                    "tx_gso_list": "off [fixed]",
                    "tx_gso_partial": "off [fixed]",
                    "tx_gso_robust": "off [fixed]",
                    "tx_ipxip4_segmentation": "off [fixed]",
                    "tx_ipxip6_segmentation": "off [fixed]",
                    "tx_lockless": "off [fixed]",
                    "tx_nocache_copy": "off",
                    "tx_scatter_gather": "on",
                    "tx_scatter_gather_fraglist": "off [fixed]",
                    "tx_sctp_segmentation": "off [fixed]",
                    "tx_tcp6_segmentation": "off [fixed]",
                    "tx_tcp_ecn_segmentation": "off [fixed]",
                    "tx_tcp_mangleid_segmentation": "off",
                    "tx_tcp_segmentation": "on",
                    "tx_tunnel_remcsum_segmentation": "off [fixed]",
                    "tx_udp_segmentation": "off [fixed]",
                    "tx_udp_tnl_csum_segmentation": "off [fixed]",
                    "tx_udp_tnl_segmentation": "off [fixed]",
                    "tx_vlan_offload": "on [fixed]",
                    "tx_vlan_stag_hw_insert": "off [fixed]",
                    "vlan_challenged": "off [fixed]"
                },
                "hw_timestamp_filters": [],
                "ipv4": {
                    "address": "10.0.2.15",
                    "broadcast": "",
                    "netmask": "255.255.255.0",
                    "network": "10.0.2.0",
                    "prefix": "24"
                },
                "ipv6": [
                    {
                        "address": "fd17:625c:f037:2:a00:27ff:fed9:fa93",
                        "prefix": "64",
                        "scope": "global"
                    },
                    {
                        "address": "fe80::a00:27ff:fed9:fa93",
                        "prefix": "64",
                        "scope": "link"
                    }
                ],
                "macaddress": "08:00:27:d9:fa:93",
                "module": "e1000",
                "mtu": 1500,
                "pciid": "0000:00:08.0",
                "promisc": false,
                "speed": 1000,
                "timestamping": [],
                "type": "ether"
            },
            "eth1": {
                "active": true,
                "device": "eth1",
                "features": {
                    "esp_hw_offload": "off [fixed]",
                    "esp_tx_csum_hw_offload": "off [fixed]",
                    "fcoe_mtu": "off [fixed]",
                    "generic_receive_offload": "on",
                    "generic_segmentation_offload": "on",
                    "highdma": "off [fixed]",
                    "hsr_dup_offload": "off [fixed]",
                    "hsr_fwd_offload": "off [fixed]",
                    "hsr_tag_ins_offload": "off [fixed]",
                    "hsr_tag_rm_offload": "off [fixed]",
                    "hw_tc_offload": "off [fixed]",
                    "l2_fwd_offload": "off [fixed]",
                    "large_receive_offload": "off [fixed]",
                    "loopback": "off [fixed]",
                    "macsec_hw_offload": "off [fixed]",
                    "netns_local": "off [fixed]",
                    "ntuple_filters": "off [fixed]",
                    "receive_hashing": "off [fixed]",
                    "rx_all": "off",
                    "rx_checksumming": "off",
                    "rx_fcs": "off",
                    "rx_gro_hw": "off [fixed]",
                    "rx_gro_list": "off",
                    "rx_udp_gro_forwarding": "off",
                    "rx_udp_tunnel_port_offload": "off [fixed]",
                    "rx_vlan_filter": "on [fixed]",
                    "rx_vlan_offload": "on",
                    "rx_vlan_stag_filter": "off [fixed]",
                    "rx_vlan_stag_hw_parse": "off [fixed]",
                    "scatter_gather": "on",
                    "tcp_segmentation_offload": "on",
                    "tls_hw_record": "off [fixed]",
                    "tls_hw_rx_offload": "off [fixed]",
                    "tls_hw_tx_offload": "off [fixed]",
                    "tx_checksum_fcoe_crc": "off [fixed]",
                    "tx_checksum_ip_generic": "on",
                    "tx_checksum_ipv4": "off [fixed]",
                    "tx_checksum_ipv6": "off [fixed]",
                    "tx_checksum_sctp": "off [fixed]",
                    "tx_checksumming": "on",
                    "tx_esp_segmentation": "off [fixed]",
                    "tx_fcoe_segmentation": "off [fixed]",
                    "tx_gre_csum_segmentation": "off [fixed]",
                    "tx_gre_segmentation": "off [fixed]",
                    "tx_gso_list": "off [fixed]",
                    "tx_gso_partial": "off [fixed]",
                    "tx_gso_robust": "off [fixed]",
                    "tx_ipxip4_segmentation": "off [fixed]",
                    "tx_ipxip6_segmentation": "off [fixed]",
                    "tx_lockless": "off [fixed]",
                    "tx_nocache_copy": "off",
                    "tx_scatter_gather": "on",
                    "tx_scatter_gather_fraglist": "off [fixed]",
                    "tx_sctp_segmentation": "off [fixed]",
                    "tx_tcp6_segmentation": "off [fixed]",
                    "tx_tcp_ecn_segmentation": "off [fixed]",
                    "tx_tcp_mangleid_segmentation": "off",
                    "tx_tcp_segmentation": "on",
                    "tx_tunnel_remcsum_segmentation": "off [fixed]",
                    "tx_udp_segmentation": "off [fixed]",
                    "tx_udp_tnl_csum_segmentation": "off [fixed]",
                    "tx_udp_tnl_segmentation": "off [fixed]",
                    "tx_vlan_offload": "on [fixed]",
                    "tx_vlan_stag_hw_insert": "off [fixed]",
                    "vlan_challenged": "off [fixed]"
                },
                "hw_timestamp_filters": [],
                "ipv4": {
                    "address": "192.168.58.101",
                    "broadcast": "192.168.58.255",
                    "netmask": "255.255.255.0",
                    "network": "192.168.58.0",
                    "prefix": "24"
                },
                "ipv6": [
                    {
                        "address": "fdeb:4c62:4b88:355d:a00:27ff:fe33:da79",
                        "prefix": "64",
                        "scope": "global"
                    },
                    {
                        "address": "fe80::a00:27ff:fe33:da79",
                        "prefix": "64",
                        "scope": "link"
                    }
                ],
                "macaddress": "08:00:27:33:da:79",
                "module": "e1000",
                "mtu": 1500,
                "pciid": "0000:00:09.0",
                "promisc": false,
                "speed": 1000,
                "timestamping": [],
                "type": "ether"
            },
            "fibre_channel_wwn": [],
            "fips": false,
            "form_factor": "",
            "fqdn": "vagrantubuntulab0101",
            "gather_subset": [
                "all"
            ],
            "hostname": "vagrantubuntulab0101",
            "hostnqn": "",
            "interfaces": [
                "eth0",
                "lo",
                "eth1"
            ],
            "is_chroot": false,
            "iscsi_iqn": "",
            "kernel": "5.15.0-160-generic",
            "kernel_version": "#170-Ubuntu SMP Wed Oct 1 10:12:04 UTC 2025",
            "lo": {
                "active": true,
                "device": "lo",
                "features": {
                    "esp_hw_offload": "off [fixed]",
                    "esp_tx_csum_hw_offload": "off [fixed]",
                    "fcoe_mtu": "off [fixed]",
                    "generic_receive_offload": "on",
                    "generic_segmentation_offload": "on",
                    "highdma": "on [fixed]",
                    "hsr_dup_offload": "off [fixed]",
                    "hsr_fwd_offload": "off [fixed]",
                    "hsr_tag_ins_offload": "off [fixed]",
                    "hsr_tag_rm_offload": "off [fixed]",
                    "hw_tc_offload": "off [fixed]",
                    "l2_fwd_offload": "off [fixed]",
                    "large_receive_offload": "off [fixed]",
                    "loopback": "on [fixed]",
                    "macsec_hw_offload": "off [fixed]",
                    "netns_local": "on [fixed]",
                    "ntuple_filters": "off [fixed]",
                    "receive_hashing": "off [fixed]",
                    "rx_all": "off [fixed]",
                    "rx_checksumming": "on [fixed]",
                    "rx_fcs": "off [fixed]",
                    "rx_gro_hw": "off [fixed]",
                    "rx_gro_list": "off",
                    "rx_udp_gro_forwarding": "off",
                    "rx_udp_tunnel_port_offload": "off [fixed]",
                    "rx_vlan_filter": "off [fixed]",
                    "rx_vlan_offload": "off [fixed]",
                    "rx_vlan_stag_filter": "off [fixed]",
                    "rx_vlan_stag_hw_parse": "off [fixed]",
                    "scatter_gather": "on",
                    "tcp_segmentation_offload": "on",
                    "tls_hw_record": "off [fixed]",
                    "tls_hw_rx_offload": "off [fixed]",
                    "tls_hw_tx_offload": "off [fixed]",
                    "tx_checksum_fcoe_crc": "off [fixed]",
                    "tx_checksum_ip_generic": "on [fixed]",
                    "tx_checksum_ipv4": "off [fixed]",
                    "tx_checksum_ipv6": "off [fixed]",
                    "tx_checksum_sctp": "on [fixed]",
                    "tx_checksumming": "on",
                    "tx_esp_segmentation": "off [fixed]",
                    "tx_fcoe_segmentation": "off [fixed]",
                    "tx_gre_csum_segmentation": "off [fixed]",
                    "tx_gre_segmentation": "off [fixed]",
                    "tx_gso_list": "on",
                    "tx_gso_partial": "off [fixed]",
                    "tx_gso_robust": "off [fixed]",
                    "tx_ipxip4_segmentation": "off [fixed]",
                    "tx_ipxip6_segmentation": "off [fixed]",
                    "tx_lockless": "on [fixed]",
                    "tx_nocache_copy": "off [fixed]",
                    "tx_scatter_gather": "on [fixed]",
                    "tx_scatter_gather_fraglist": "on [fixed]",
                    "tx_sctp_segmentation": "on",
                    "tx_tcp6_segmentation": "on",
                    "tx_tcp_ecn_segmentation": "on",
                    "tx_tcp_mangleid_segmentation": "on",
                    "tx_tcp_segmentation": "on",
                    "tx_tunnel_remcsum_segmentation": "off [fixed]",
                    "tx_udp_segmentation": "on",
                    "tx_udp_tnl_csum_segmentation": "off [fixed]",
                    "tx_udp_tnl_segmentation": "off [fixed]",
                    "tx_vlan_offload": "off [fixed]",
                    "tx_vlan_stag_hw_insert": "off [fixed]",
                    "vlan_challenged": "on [fixed]"
                },
                "hw_timestamp_filters": [],
                "ipv4": {
                    "address": "127.0.0.1",
                    "broadcast": "",
                    "netmask": "255.0.0.0",
                    "network": "127.0.0.0",
                    "prefix": "8"
                },
                "ipv6": [
                    {
                        "address": "::1",
                        "prefix": "128",
                        "scope": "host"
                    }
                ],
                "mtu": 65536,
                "promisc": false,
                "timestamping": [],
                "type": "loopback"
            },
            "loadavg": {
                "15m": 0.00390625,
                "1m": 0.0615234375,
                "5m": 0.01513671875
            },
            "locally_reachable_ips": {
                "ipv4": [
                    "10.0.2.15",
                    "127.0.0.0/8",
                    "127.0.0.1",
                    "192.168.58.101"
                ],
                "ipv6": [
                    "::1",
                    "fd17:625c:f037:2:a00:27ff:fed9:fa93",
                    "fdeb:4c62:4b88:355d:a00:27ff:fe33:da79",
                    "fe80::a00:27ff:fe33:da79",
                    "fe80::a00:27ff:fed9:fa93"
                ]
            },
            "lsb": {
                "codename": "jammy",
                "description": "Ubuntu 22.04.5 LTS",
                "id": "Ubuntu",
                "major_release": "22",
                "release": "22.04"
            },
            "lvm": "N/A",
            "machine": "aarch64",
            "machine_id": "d2cbe610f1d043198604b50ca24eb14c",
            "memfree_mb": 1257,
            "memory_mb": {
                "nocache": {
                    "free": 3072,
                    "used": 710
                },
                "real": {
                    "free": 1257,
                    "total": 3782,
                    "used": 2525
                },
                "swap": {
                    "cached": 0,
                    "free": 3782,
                    "total": 3782,
                    "used": 0
                }
            },
            "memtotal_mb": 3782,
            "module_setup": true,
            "mounts": [
                {
                    "block_available": 5647528,
                    "block_size": 4096,
                    "block_total": 7817692,
                    "block_used": 2170164,
                    "device": "/dev/mapper/ubuntu--vg-ubuntu--lv",
                    "dump": 0,
                    "fstype": "ext4",
                    "inode_available": 1935430,
                    "inode_total": 1998848,
                    "inode_used": 63418,
                    "mount": "/",
                    "options": "rw,relatime",
                    "passno": 0,
                    "size_available": 23132274688,
                    "size_total": 32021266432,
                    "uuid": "2bf8029c-4c42-4681-8997-5f2f6c680ea0"
                },
                {
                    "block_available": 0,
                    "block_size": 131072,
                    "block_total": 477,
                    "block_used": 477,
                    "device": "/dev/loop1",
                    "dump": 0,
                    "fstype": "squashfs",
                    "inode_available": 0,
                    "inode_total": 11879,
                    "inode_used": 11879,
                    "mount": "/snap/core20/2690",
                    "options": "ro,nodev,relatime,errors=continue",
                    "passno": 0,
                    "size_available": 0,
                    "size_total": 62521344,
                    "uuid": "N/A"
                },
                {
                    "block_available": 0,
                    "block_size": 131072,
                    "block_total": 270,
                    "block_used": 270,
                    "device": "/dev/loop2",
                    "dump": 0,
                    "fstype": "squashfs",
                    "inode_available": 0,
                    "inode_total": 648,
                    "inode_used": 648,
                    "mount": "/snap/snapd/21761",
                    "options": "ro,nodev,relatime,errors=continue",
                    "passno": 0,
                    "size_available": 0,
                    "size_total": 35389440,
                    "uuid": "N/A"
                },
                {
                    "block_available": 0,
                    "block_size": 131072,
                    "block_total": 478,
                    "block_used": 478,
                    "device": "/dev/loop0",
                    "dump": 0,
                    "fstype": "squashfs",
                    "inode_available": 0,
                    "inode_total": 12028,
                    "inode_used": 12028,
                    "mount": "/snap/core20/2321",
                    "options": "ro,nodev,relatime,errors=continue",
                    "passno": 0,
                    "size_available": 0,
                    "size_total": 62652416,
                    "uuid": "N/A"
                },
                {
                    "block_available": 0,
                    "block_size": 131072,
                    "block_total": 620,
                    "block_used": 620,
                    "device": "/dev/loop3",
                    "dump": 0,
                    "fstype": "squashfs",
                    "inode_available": 0,
                    "inode_total": 954,
                    "inode_used": 954,
                    "mount": "/snap/lxd/29353",
                    "options": "ro,nodev,relatime,errors=continue",
                    "passno": 0,
                    "size_available": 0,
                    "size_total": 81264640,
                    "uuid": "N/A"
                },
                {
                    "block_available": 0,
                    "block_size": 131072,
                    "block_total": 649,
                    "block_used": 649,
                    "device": "/dev/loop4",
                    "dump": 0,
                    "fstype": "squashfs",
                    "inode_available": 0,
                    "inode_total": 956,
                    "inode_used": 956,
                    "mount": "/snap/lxd/36930",
                    "options": "ro,nodev,relatime,errors=continue",
                    "passno": 0,
                    "size_available": 0,
                    "size_total": 85065728,
                    "uuid": "N/A"
                },
                {
                    "block_available": 436026,
                    "block_size": 4096,
                    "block_total": 498138,
                    "block_used": 62112,
                    "device": "/dev/sda2",
                    "dump": 0,
                    "fstype": "ext4",
                    "inode_available": 130816,
                    "inode_total": 131072,
                    "inode_used": 256,
                    "mount": "/boot",
                    "options": "rw,relatime",
                    "passno": 0,
                    "size_available": 1785962496,
                    "size_total": 2040373248,
                    "uuid": "64a2d46d-6818-4f02-b510-a68cb9e0b085"
                },
                {
                    "block_available": 273040,
                    "block_size": 4096,
                    "block_total": 274657,
                    "block_used": 1617,
                    "device": "/dev/sda1",
                    "dump": 0,
                    "fstype": "vfat",
                    "inode_available": 0,
                    "inode_total": 0,
                    "inode_used": 0,
                    "mount": "/boot/efi",
                    "options": "rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro",
                    "passno": 0,
                    "size_available": 1118371840,
                    "size_total": 1124995072,
                    "uuid": "3D4A-E5A1"
                },
                {
                    "block_available": 0,
                    "block_size": 131072,
                    "block_total": 355,
                    "block_used": 355,
                    "device": "/dev/loop5",
                    "dump": 0,
                    "fstype": "squashfs",
                    "inode_available": 0,
                    "inode_total": 621,
                    "inode_used": 621,
                    "mount": "/snap/snapd/25585",
                    "options": "ro,nodev,relatime,errors=continue",
                    "passno": 0,
                    "size_available": 0,
                    "size_total": 46530560,
                    "uuid": "N/A"
                }
            ],
            "nodename": "vagrantubuntulab0101",
            "os_family": "Debian",
            "pkg_mgr": "apt",
            "proc_cmdline": {
                "BOOT_IMAGE": "/vmlinuz-5.15.0-160-generic",
                "autoinstall": true,
                "biosdevname": "0",
                "ds": "nocloud-net",
                "net.ifnames": "0",
                "ro": true,
                "root": "/dev/mapper/ubuntu--vg-ubuntu--lv"
            },
            "processor": [
                "0",
                "1"
            ],
            "processor_cores": 1,
            "processor_count": 2,
            "processor_nproc": 2,
            "processor_threads_per_core": 1,
            "processor_vcpus": 2,
            "product_name": "",
            "product_serial": "",
            "product_uuid": "",
            "product_version": "",
            "python": {
                "executable": "/usr/bin/python3.10",
                "has_sslcontext": true,
                "type": "cpython",
                "version": {
                    "major": 3,
                    "micro": 12,
                    "minor": 10,
                    "releaselevel": "final",
                    "serial": 0
                },
                "version_info": [
                    3,
                    10,
                    12,
                    "final",
                    0
                ]
            },
            "python_version": "3.10.12",
            "real_group_id": 1001,
            "real_user_id": 1001,
            "selinux": {
                "status": "disabled"
            },
            "selinux_python_present": true,
            "service_mgr": "systemd",
            "ssh_host_key_ecdsa_public": "AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBBOheH3gUd034CkNbrAeqcO8sut/AsSnZF+kuO+frcdaqvsFlgzwJMzmzUTaAW8dNekwPlksWBpUW/fBfQM31bY=",
            "ssh_host_key_ecdsa_public_keytype": "ecdsa-sha2-nistp256",
            "ssh_host_key_ed25519_public": "AAAAC3NzaC1lZDI1NTE5AAAAIMw2zBFN34nuXD31N4qKSBtYxlPiq2B8QGj7hXGWHaOl",
            "ssh_host_key_ed25519_public_keytype": "ssh-ed25519",
            "ssh_host_key_rsa_public": "AAAAB3NzaC1yc2EAAAADAQABAAABgQCYPrDXqB4qafaDN6pzkBl+E+i9ldqPthjt9bzn8Ufrl0xwS+hrfLe+H9kUVAYNeIHyJL10B/oGBcSbBt7V6BhSRWU29OZ1GdpKw/RNs31PveuMpiS8luVdez342ula3eDL9uiCmbfeV1OhfLdlw/beRn0/oOmt2bSUC47kgEzkcjTwwktXmYQq2IRY/qEJYWTDA8Ym3MUQIgwnrLhoB1cVKVTY37iDTH19fmqV06bd1Isd9GdVQpMpTWrYXqs3XGYUPqJGqJ4CouS3csN4QdamF7mqGCktOJQEUN4incoHnXqWpSDNQMgyVFLmInlcRsj5YnYAn/oicDHISh3PCK/kUB8JiuRlTzOfNxskAgapOWD1t/dLocF+tP1qxM4qC2GvdNi9MUSC7rQBf8aAXREP/0c7Ns0Bi3pNZ/fmVK2XMHB8Et4wocKTx8Wke30Cy6daa98TX3H/oevjCSAeIyHd8Ii69J/AB/x8ZheutsNndKW3SWfWtH+B9iBNDpk+BMs=",
            "ssh_host_key_rsa_public_keytype": "ssh-rsa",
            "swapfree_mb": 3782,
            "swaptotal_mb": 3782,
            "system": "Linux",
            "system_capabilities": [
                ""
            ],
            "system_capabilities_enforced": "True",
            "system_vendor": "",
            "systemd": {
                "features": "+PAM +AUDIT +SELINUX +APPARMOR +IMA +SMACK +SECCOMP +GCRYPT +GNUTLS +OPENSSL +ACL +BLKID +CURL +ELFUTILS +FIDO2 +IDN2 -IDN +IPTC +KMOD +LIBCRYPTSETUP +LIBFDISK +PCRE2 -PWQUALITY -P11KIT -QRENCODE +BZIP2 +LZ4 +XZ +ZLIB +ZSTD -XKBCOMMON +UTMP +SYSVINIT default-hierarchy=unified",
                "version": 249
            },
            "uptime_seconds": 98841,
            "user_dir": "/home/ansible",
            "user_gecos": "",
            "user_gid": 1001,
            "user_id": "ansible",
            "user_shell": "/bin/bash",
            "user_uid": 1001,
            "userspace_bits": "64",
            "virtualization_role": "NA",
            "virtualization_tech_guest": [],
            "virtualization_tech_host": [],
            "virtualization_type": "NA"
        }
    }
    ```
- You can reference the model of the first disk in the facts shown above in a template or playbook as:
  - `{{ ansible_facts['devices']['xvda']['model] }}`
- To reference the system hostname:
  - `{{ ansible_facts['nodename'] }}`

## Caching facts
- Like registered variables, facts are stored in memory by default. However, unlike registered variables, facts can be gathered independently and cached for repeated use.
- With cached facts, you can refer to facts from one system when configuring a second system, even if Ansible executes the current play on the second system first. For example
  - `{{ hostvars['host.example.com']['ansible_facts']['os_family'] }}`
- Caching is controlled by the cache plugins. By default, Ansible used the memory cache plugin, which stores facts in memory for the duration of the current playbook run.
- Facts caching can improve performance. If you manage thousands of hosts, you can configure facts caching to run nightly, and then manage configuration on a smaller set of servers periodically throughout the day.
- With cached facts, you have access to variables and information about all hosts even when you are only managing a small number of servers.

## Disabling facts
- By default, Ansible gather facts at the beginning of each play. If you don't need to gather facts, you can turn off fact gathering at the play level to improve scalability.
- Disabling facts may particularly improve performance in push mode with very large number of systems, or ig you are using Ansible on experimental platforms.
- To disable fact gathering

    ```yaml
    - name: Playbook
      hosts: all
      gather_facts: false
    ```

## Adding custom facts
- The setup module in Ansible automatically discovers a standard ser of facts about each host. If you want to add custom values to your facts, you can write a custom facts module, set temporary facts with a `ansible.builtin.set_fact` task, or provide custom facts using the facts.d directory.

### facts.d or local facts
- You can add static custom facts by adding static files to facts.d or add dynamic facts by adding executable script to facts.d.
- For example, you can add a list of all users on a host to your facts by creating and running a script in facts.d
- To use facts.d, create an `/etc/ansible/facts.d` directory on the remote host or hosts. 
- If you prefer a different directory, create it and specify it using the `fact_path` play keyword.
- Add files to the directory to supply your custom facts. ALl file names must end with `.fact`. The files can be JSON, INI or executable files returning JSON
- To add static facts, simply add a file with the `.fact` extension. For example, create `/etc/ansible/facts.d/preferences.fact` with the below content
    ```
    [general]
    adfs=1
    bar=2
    foo=3
    ```
- The next time fact gathering runs, your fact will include a hash variable fact named `general` with all the values as members.
- To validate this we can run the ansible adhoc command as below

    ```commandline
    ansible vagrantubuntulab0101 -i inventory/hosts.yml -m ansible.builtin.setup -a "filter=ansible_local" 
    vagrantubuntulab0101 | SUCCESS => {
        "ansible_facts": {
            "ansible_local": {
                "preferences": {
                    "general": {
                        "adfs": "1",
                        "bar": "2",
                        "fpp": "3"
                    }
                }
            }
        },
        "changed": false
    }
    ```
- The `ansible_local` namespace separates custom facts created by facts.d from system facts or variables defines else where in the playbook, so variables will not override with each other.

- You can also use facts.d to execute a script on the remote host, gathering dynamic custom facts to the ansible_local namespace.
- For example, you can generate a list of all users that exists on a remote host as a fact about that host. To generate dynamic custom facts using facts.d:
  - Write and test a script to generate the JSON data you want
  - Save the script in your facts.d directory
  - Make sure your script is executable by the Ansible connection user
  - Gather facts to execute the script and add the JSON output to ansible_local

Example:

- I have created a script in my ansible remote machine to gather all the user accounts present.

    ```commandline
    $ ls -l local_users.fact
    -rwxrwxrwx 1 root root 144 Dec 20 10:22 local_users.fact
    
    $ cat local_users.fact
    #! /bin/bash
    
    getent passwd | awk -F : '{print $1, $3}' | \
    jq -Rn '{users: [ inputs | split(" ") | {name: .[0], uid: (.[1] | tonumber)} ] }' 
    ```

- When I gather the facts from my remote node using `ansible.builtin.setup` module, the custom facts will be loaded under `ansible_local.local_users`
    
    ```commandline
    ansible vagrantubuntulab0101 -i inventory/hosts.yml -m ansible.builtin.setup -a "filter=ansible_local"
    vagrantubuntulab0101 | SUCCESS => {
        "ansible_facts": {
            "ansible_local": {
                "local_users": {
                    "users": [
                        {
                            "name": "root",
                            "uid": 0
                        },
                        {
                            "name": "daemon",
                            "uid": 1
                        },
                        {
                            "name": "bin",
                            "uid": 2
                        },
                        {
                            "name": "sys",
                            "uid": 3
                        },
                        {
                            "name": "sync",
                            "uid": 4
                        },
                        {
                            "name": "games",
                            "uid": 5
                        },
                        {
                            "name": "man",
                            "uid": 6
                        },
                        {
                            "name": "lp",
                            "uid": 7
                        },
                        {
                            "name": "mail",
                            "uid": 8
                        },
                        {
                            "name": "news",
                            "uid": 9
                        },
                        {
                            "name": "uucp",
                            "uid": 10
                        },
                        {
                            "name": "proxy",
                            "uid": 13
                        },
                        {
                            "name": "www-data",
                            "uid": 33
                        },
                        {
                            "name": "backup",
                            "uid": 34
                        },
                        {
                            "name": "list",
                            "uid": 38
                        },
                        {
                            "name": "irc",
                            "uid": 39
                        },
                        {
                            "name": "gnats",
                            "uid": 41
                        },
                        {
                            "name": "nobody",
                            "uid": 65534
                        },
                        {
                            "name": "_apt",
                            "uid": 100
                        },
                        {
                            "name": "systemd-network",
                            "uid": 101
                        },
                        {
                            "name": "systemd-resolve",
                            "uid": 102
                        },
                        {
                            "name": "messagebus",
                            "uid": 103
                        },
                        {
                            "name": "systemd-timesync",
                            "uid": 104
                        },
                        {
                            "name": "pollinate",
                            "uid": 105
                        },
                        {
                            "name": "syslog",
                            "uid": 106
                        },
                        {
                            "name": "uuidd",
                            "uid": 107
                        },
                        {
                            "name": "tcpdump",
                            "uid": 108
                        },
                        {
                            "name": "tss",
                            "uid": 109
                        },
                        {
                            "name": "landscape",
                            "uid": 110
                        },
                        {
                            "name": "fwupd-refresh",
                            "uid": 111
                        },
                        {
                            "name": "sshd",
                            "uid": 112
                        },
                        {
                            "name": "vagrant",
                            "uid": 1000
                        },
                        {
                            "name": "lxd",
                            "uid": 999
                        },
                        {
                            "name": "vboxadd",
                            "uid": 998
                        },
                        {
                            "name": "ansible",
                            "uid": 1001
                        },
                        {
                            "name": "mysql",
                            "uid": 113
                        }
                    ]
                },
                "preferences": {
                    "general": {
                        "adfs": "1",
                        "bar": "2",
                        "fpp": "3"
                    }
                }
            }
        },
        "changed": false
    }
    ```

- By default, fact gathering runs once at the beginning of each play. If you create a custom fact using facts.d in a playbook, It will be available in the next play that gather facts.
- If you want to use it in the same play whee you created it, you must explicitly re-run the setup module. For example:

    ```yaml
    - name: custom facts example
      hosts: all
      tasks:
        - name: create the directory for ansible custom facts
          ansible.builtin.file:
            state: directory
            path: /etc/ansible/facts.d
            recursive: true
        - name: Copy the custom facts
          ansible.builtin.copy:
            src: ipmi.fact
            dest: /etc/ansible/facts.d
        - name: Re-read facts after adding custom facts
          ansible.builtin.setup:
            filter: ansible_local
    ```
  
## Ansible Magic Variables
- You can access the information about Ansible operations, including the Python version being used, the hosts and groups in inventory, and the directories for playbooks and riles, using "magic" variables.
- Magic variables names are reserved - do not set variables with these names. The variable `environment` is also reserved.
- The below are the list of ansible magic variables
  - `ansible_check_mode`: Boolean that indicates if we are in check mode or not
  - `ansible_collection_name`: The name of the collection the task that is executing is a part of. In the format *namespace.collection*
  - `ansible_config_file`: The full path of used ansible configuration file
  - `ansible_dependent_role_names`: The names of the roles currently imported into the current play as dependencies of other plays
  - `ansible_diff_mode`: Boolean that indicates that you are in diff mode or not
  - `ansible_forks`: Integer reflecting the numbr of maximum forks available to this run
  - `ansible_index_var`: The name of the valur provided to *loop_control.index_var*.
  - `ansible_inventory_sources`: List of sources used as inventory
  - `ansible_limit`: Content of the *--limit* CLI option for the current execution of Ansible
  - `ansible_loop`: A dictionary/map containing extended loop information when enabled through *loop_control.loop_var*
  - `ansible_parent_role_names`: When the current role is executed by means of an `include_role` or `import_role` action, this variable contains a list of all parent roles, with the most recent role being the first item in the list. When multiple inclusions occur, this list lists the last role as the first item in the list. It is also possible that a specific role exists more than once in this list. For example: when role *A* includes role *B*, inside role B, *ansible_parent_role_names* will be equal to *['A']*. If role *B* then includes role *C*, the list becomes *['B'. 'A']* .
  - `ansible_parent_role_paths`: When the current role is being executed by means of an `include_role` or `import_role` action, this variable contains a list of all parent roles paths, with the most recent role being the first item in the list.
  - `ansible_play_batch`: List of active hosts in the current play run limited by serial, aka 'batch'. Failed/Unreachable hosts are not considered 'active'.
  - `ansible_play_hosts`: List of hosts in the current play run, not limited by the serial, Failed/Unreachable hosts are excluded from this list.
  - `ansible_play_hosts_all`: List of all the hosts that were targeted by the play
  - `ansible_play_name`: The name of the currently executed play. (name attribute of the play not filename of the play)
  - `ansible_play_role_names`: The names of the roles currently imported into the current play. This list does not contain the role names that are implicitly included through dependencies.
  - `ansible_playbook_python`: The path to the python interpreter being used by Ansible on the control node
  - `ansible_role_name`: The fully qualified collection role name, in the format *namespace.collection.role_name*
  - `ansible_role_names`: The names of the roles currently imported into the current play, or roles referenced as dependencies of the roles imported into the current play.
  - `ansible_run_tags`: Contents of the --tags CLI option, which specifies which tags will be included for the current run. If the *--tags* is not passed, this variable will default to *["all"]*
  - `ansible_search_path`: Current search path for action plugins and lookups, in other works, where we search for relative paths when you do *template: src=myfile*
  - `ansible_skip_tags`: Contents of the *--skip-tags* CLI option, which specifies which tags will be skipped for the current run.
  - `ansible_verbosity`: Current verbosity settings for Ansible
  - `ansible_version`: Dictionary/map that contains information about the current running version of ansible.
  - `group_names`: List of groups the current host is part of, it always reflects the *inventory_hostname* and ignores delegation
  - `groups`: A dictionary/map with all the groups in inventory and each group has the list of hosts that belong to it.
  - `hostvars`: A dictionary/map with all the hosts in inventory and variables assigned to them
  - `inventory_dir`: The directory of the inventory source in which the *inventory_hostname* was first defines. This always reflects the *inventory_hostname* and ignores delegation.
  - `inventory_hostname`: The inventory name for the 'current' host iterated over in the play. This is not affected by delegation, it always reflects the original host for the task.
  - `inventory_hostname_short`: The short version of *inventory_hostname*, is the first section after splitting it via **.** .
  - `inventory_file`: The file name of the inventory source in which the *inventory_hostname* was first defines.
  - `omit`: Special variable that allows you to 'omit' an option in a task, for example `- user: name=bon home={{ bobs_home | default(omit) }}`
  - `playbook_dir`: The path to the directory of the current playbook being executed.
  