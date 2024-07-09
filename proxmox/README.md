## Konfiguracja hosta

- nazwa node'a: proxie
- adresacja:
    - adres IP: Pobierany przez DHCP
    - brama: Router
    - domena: dziki.lab
- zaaktualizowano system
- dodano użytkownika persefona
- dodano uwierzytelnienie dwu-etapowe dla użytkowników: root, persefona
- skonfigurowano serwer OpenSSH:
    - logowanie jedynie kluczem prywatnym
    - port: 2223
    - wyłączono logowanie za pomocą roota, puste hasła
- skonfigurowano zdalne połączenie za pomocą zero-tier:
- dodano mostek sieciowy vmbr1
    
## Obrazy dysków systemów

**Alpine Linux:** https://dl-cdn.alpinelinux.org/alpine/v3.20/releases/x86_64/alpine-virt-3.20.0-x86_64.iso

**OpenMediaVault:** https://sourceforge.net/projects/openmediavault/files/iso/7.0-32/openmediavault_7.0-32-amd64.iso

**OPNSense:** https://mirror.ams1.nl.leaseweb.net/opnsense/releases/24.1/OPNsense-24.1-dvd-amd64.iso.bz2

**Windows Server 2019:** https://go.microsoft.com/fwlink/p/?LinkID=2195167&clcid=0x409&culture=en-us&country=US

## Maszyny wirtualne

- każda maszyna startuje wraz z uruchomieniem Node'a
- w każdej maszynie włączona jest emulacja SSD
- hardware nie pozwala by wszystkie maszyny na raz mogły płynnie działać

- Alpine-K8s-01:
    - cpu: 2
    - ram: 8GB
    - dysk: 200GB
    - VLAN: 10
- OpenMediaVault-NAS-01
    - cpu: 1
    - ram: 4GB
    - dysk:
        - system: 100GB
        - raid 5: 3x50GB
    - VLAN: 20
- OPNsense-Firewall-01:
    - cpu: 1
    - ram: 4GB
    - dysk: 100GB
    - VLAN: 17
    - interfejsy sieciowe:
        - net0 - vmbr0 - LAN
        - net1 - vmbr1 - WAN
- WinServ2k19-AD-01
    - cpu: 1
    - ram: 4GB
    - dysk: 100GB
    - VLAN: 30
- WinServ2k19-AD-02
    - cpu: 2
    - ram: 4GB
    - dysk: 85GB
    - VLAN: 30

**Źródła:**
- https://pve.proxmox.com/pve-docs/pve-admin-guide.html
- obrazy iso:
    - Alpine: https://www.alpinelinux.org/downloads/
    - OpenMediaVault: https://www.openmediavault.org/download.html
    - OPNsense: https://opnsense.org/download/
    - Windows: https://www.microsoft.com/en-us/evalcenter/download-windows-server-2019
