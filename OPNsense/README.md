# Konfiguracja NGFW - OPNsense

![Logo OPNsense](https://www.landitec.com/wp-content/uploads/2019/12/opnsense.png)

## Wstępna konfiguracja

**System:**
- access:
    - utworzono nowego użytkownika z uprawnienia administratora: rootkowski
    - wyłączono użytkownika root
    - dodano serwer TFA (Two Factor Autentication): TOTP Server
    - skonfigurowano logowanie dwuetapowe
- firmware:
    - status:
        - zaktualizowano do najnowszej wersji
    - plugins:
        - zainstalowano wtyczki:
           - os-collectd
           - os-crowdsec
           - os-dmidecode
           - os-haproxy
           - os-maltrail
           - os-zerotier
- settings:
    - skonfigurowano dostęp do panelu administracyjnego przez protokół HTTPS
    - zmieniono port panelu na 8000
    - skonfigurowano automatyczne wylogowanie użytkownika po 5 minutach nieaktywności
    - wyłączono ssh
    - uwierzytelnienie przez serwer TOTP
    - zmieniono serwery DNS na serwery AdGuard'a: 94.140.14.14, 94.140.15.15
- interfaces:
    - dodano interfejs administracyjny: Lan02 
- firewall:
    - nat:
        - wyłączono NAT
    - rules:
        - Lan02 - dodano zasadę pozwalająca się połączyć z interfejem

## VLAN'y
- dodano VLANy: 10 - NAS, 20 - Linux, 30 - Windows:
    - na panelu na lewo: interfaces -> other types -> VLAN -> "+"
- przypisano wirtualne interfejsy, włączono je i nadano im adresy IP:
    - na panelu na lewo: interfaces -> assignments
    - adresy IP:
        - 172.17.10.1/28 - NAS
        - 172.17.20.1/28 - Linux
        - 172.17.30.1/28 - Windows
    - na serwerze Proxmox:
        - dodano VLAN tag dla interfejsów maszyn:
            - OPNsense-Firewall-01: Lan - 10, 20, 30
            - wykorzystane polecenie:
            ```bash
            qm set 102 -net0 "virtio=66:64:C9:BC:13:F1,bridge=vmbr0,trunks=10;20;30"
            ```
            - WinServ2k19-AD-01/02: Lan - 30 
            - OpenMediaVault-NAS-01: Lan - 10 

        - na wirtualnym mostku sieciowym włączono opcję: Vlan Aware
            - na panelu na lewo: Datacenter -> \[Nazwa serwera Proxmox] -> Network -> PPM na mostek

        - dodano zasady do zapory na wirtualnych interfejsach w celu filtracji VLANów:
        - VLAN 10 (NAS) - dostęp ma tylko serwery Windows
        - VLAN 20 (Linux) - dostęp tylko do serwerów Windows
        - VLAN 30 (Windows) - dostęp do serwera NAS i Linux

## OpenVPN - Remote Access:
- utworzono wewnętrzny Server Of Authority - CA-Dziki-Lab:
    - na panelu na lewo: System -> Trust -> Authorities 
    - klucz szyfrowania: RSA 4096 bit
    - algorytm szyfrowania: SHA512
    - czas ważności: 10 lat
- utworzono wewnętrzny certyfikat dla serwera VPN - OpenVPN Server Cert:
    - na panelu na lewo: System -> Trust -> Certificates 
    - klucz szyfrowania: RSA 4096 bit
    - algorytm szyfrowania: SHA512
    - czas ważności: 10 lat
- dodano użytkownika example-ovpn-user:
    - na panelu na lewo: System -> Access -> Users 
    - powłoka: /usr/sbin/nologin
    - wygenerowano certyfikat użytkownika: example-ovpn-user:
        - klucz szyfrowania: RSA 4096 bit
        - algorytm szyfrowania: SHA512
        - czas ważności: 10 lat
- wygenerowano klucz statyczny - OpenVPN-key-01:
    - na panelu na lewo: VPN -> OpenVPN -> zakładka Static Keys
- dodano instację serwera OpenVPN:
    - port: 1194
    - server (Pula adresów dla VPN): 10.17.77.0/24
    - certificate: OpenVPN Server Cert 
    - certificate depth: One (Client + Server)
    - tls static key: OpenVPN-key-01
    - Strict User/CN Matching: \[x]
    - Local Network:
        - 172.17.40.1/32
        - 172.17.10.0/28
        - 172.17.20.0/28
        - 172.17.30.0/28
    - DNS Servers: 172.17.17.1
    - options: client-to-client
    - redirect gateway: default
- przypisano, włączono interfejs ovpn1:
    - na panelu na lewo: interfaces -> assignments
- dodano zasadę do zapory ogniowej na interfejsie WAN pozwalający na dostęp do serwera VPN:
    - na panelu na lewo: firewall -> Rules -> WAN
- dodano zasadę do zapory ogniowej na interfejsie Ovpn01 pozwalający na dostęp do sieci lokalnych za firewallem:
    - na panelu na lewo: firewall -> Rules -> Ovpn01

**Źródła:**
- Konfiguracja VLANów: https://youtu.be/UI5tO1hP2q8?t=455
- Polecenie pozwalające dodać kilka VLAN tagów na jeden wirtualny interfejs: https://forum.proxmox.com/threads/how-do-i-assign-multiple-vlans-to-a-vm.130435/
- Konfiguracja OpenVPN: https://youtu.be/3A5eIYs6adk
