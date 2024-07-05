Router r.dziki.pl 172.17.7.1 172.17.17.1 172.17.71.1

Proxmox prox.dziki.pl 172.17.17.2

opnSense fire.dziki.lab:
- WAN: 172.17.17.3 VLAN 17
- LAN02: 172.17.40.1
- VLAN10: 172.17.10.1/28
- VLAN20: 172.17.20.1/28
- VLAN30: 172.17.30.1/28

OpenMediaVault openmediavault.dziki.lab 172.17.10.2 VLAN 10
Alpine linx.dziki.lab 172.17.20.2 VLAN 20
AD-01 ad-01.dziki.lab 172.17.30.2 VLAN 30
AD-02 ad-02.dziki.lab 172.17.30.3 VLAN 30
