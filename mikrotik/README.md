## Winbox 

Przed przystąpieniem do konfiguracji zainstalowano klient WinBox według poradnika:

[![Jak zainstalować WinBox'a na Linuxie?](https://img.youtube.com/vi/2jjtK5Me29I/0.jpg)](https://www.youtube.com/watch?v=2jjtK5Me29I)

Pobrano z repozytorium AUR na dystrybucji Arch Linux:
```bash
yay -S winbox
```

## Początkowa konfiguracja

Skonfigurowano według dokumentacji: https://help.mikrotik.com/docs/display/ROS/First+Time+Configuration

- usunięto domyślną konfiguracje
- nazwa mostka łączącego porty LAN: LAN BRIDGE
- porty połączone w mostek: Ether2-Ether5, Wifi 2.4GHz Wifi5GHz
- konfiguracja mostka: 172.17.7.1/25 (Sieć LAN)
- adres interfejsu WAN (Ether1): Klient DHCP
- adres Wifi 5GHz: 172.17.7.169/29
- DHCP:
    - interfejs: LAN BRIDGE
    - pula adresów: 172.17.7.2-127/25
    - bramę ustawiono jako adres routera
    - DNS ustawiono jako: adres routera i 94.140.14.14 (publiczny serwery AdGuard'a blokujący reklamy)
    - dzierżawa adresu: 8 godzin
- nowy użytkownik: 
    - pełen dostęp
    - dostępny z sieci: 172.17.7.168/29, 10.147.20.222, 10.147.20.150 (ZeroTier - zdalny dostęp przez VPNa)
    - hasło: wygenerowane i zapisane w menedżerze haseł
- konto admin wyłączone
- wyłączono nieużywane porty: Ether2-Ether4
- wyłączono usługi Mac-Telnet, Neighbor Discovery, interfejs webowy, api, bandwith server, UPNP, IPv6
- wyłączono DNS Cache:
```mikrotik
/ip dns set allow-remote-requests=no
```
- SSH:
    - port: 2222 
    - dostępny z sieci: 172.17.7.168/29, 10.147.20.222, 10.147.20.150 (ZeroTier - zdalny dostęp przez VPNa)
    - użycie silniejszej kryptografii:
    ```mikrotik
    /ip ssh set strong-crypto=yes
    ```
    - ustawiono typ klucza SSH: Ed25519
    - ustawiono rozmiar klucza SSH: 4096
    - wygenerowano klucz prywatny i publiczny z hasłem zapisanym w menedżerze haseł:
    ```bash
    ssh-keygen -C "mikrotik" -b 4096 -f /home/zzikson/.ssh/mikrotik
    ```
    - zaimportowano klucz publiczny do routera:
    ```mikrotik
    /user/ssh-keys/ import public-key-file=mikrotik.pub user=WcqwHPdm9AEu
    ```
    - wyłączono logowanie za pomocą hasła
- Winbox:
    - port: 2222 
    - dostępny z sieci: 172.17.7.168/29, 10.147.20.222, 10.147.20.150 (ZeroTier - zdalny dostęp przez VPNa)
- dodano zasady do zapory ogniowej:
```mikrotik
/ip firewall filter
  add chain=forward action=fasttrack-connection connection-state=established,related comment="fast-track for established,related";
  add chain=forward action=accept connection-state=established,related comment="accept established,related";
  add chain=forward action=drop connection-state=invalid
  add chain=forward action=drop connection-state=new connection-nat-state=!dstnat in-interface=ether1 comment="drop access to clients behind NAT from WAN"
  add chain=forward action=fasttrack-connection connection-state=established,related \
    comment="fast-track for established,related";
  add chain=forward action=accept connection-state=established,related \
    comment="accept established,related";
  add chain=forward action=drop connection-state=invalid
  add chain=forward action=drop connection-state=new connection-nat-state=!dstnat \
    in-interface=ether1 comment="drop access to clients behind NAT from WAN"
```
- skonfigurowano PAT:
```mikrotik
/ip firewall nat
  add chain=srcnat out-interface=ether1 action=masquerade
```
- wyłączono IPv6
- zrobiono backup konfigurecji o nazwie
- zaktualizowano RouterOS do wersji 7.14.3

## Sieć bezprzewodowa

- sieć 2.4GHz przeznaczona dla gości:
    - hasło: wygenerowane, zapisane w menedżerze haseł
    - dozwolone szyfrowanie: WPA2-PSK, WPA3-PSK 
    - wyłączono WPS
    - ustawiono kraj jako Polska (ilość kanałów 13)
- sieć 5GHz jest roboczo skonfigurowana jako sieć do zarządzania routerem:
    - hasło: wygenerowane, zapisane w menedżerze haseł
    - rozgłaszanie SSID wyłączone
    - dozwolone szyfrowanie WPA3-PSK 
    - wyłączono WPS
    - ustawiono kraj jako Polska (ilość kanałów 13)

## DNS

- dodano rekordy:
    - A - dziki.pl -> 172.17.7.91
    - A - nas.dziki.pl -> 172.17.7.80
    - A - prox.dziki.pl -> 172.17.7.103
    - A - dns.adguard-dns.com -> 94.140.14.14
    - A - dns.adguard-dns.com -> 94.140.15.15
    - AAAA - dns.adguard-dns.com -> 2a10:50c0::ad1:ff
    - AAAA - dns.adguard-dns.com -> 2a10:50c0::ad2:ff
- skonfigurowano szyfrowany DNS (DoH):
    - pobrano certyfikat .pem serwera DNS dns.adguard-dns.com
    - zaimportowano certyfikat
    - włączono DNS Cache:
    ```mikrotik
    /ip dns set allow-remote-requests=no
    ```
    - ustawiono serwer DoH: https://dns.adguard-dns.com/dns-query
    - usunięto adresy DNS pochodzące z DHCP na interfejsie: Ether1
    - skonfigurowano NAT dla DNS
    ```mikrotik
    /ip firewall nat add chain=dstnat action=redirect protocol=tcp dst-port=53 
    /ip firewall nat add chain=dstnat action=redirect protocol=udp dst-port=53 
    ```
    - skonfigurowano klienta NTP: 185.177.151.86
    - skonfigurowano NAT dla NTP. Zasadę umieszczono na pierwszym miejscu:
    ```mikrotik
    /ip firewall nat add chain=srcnat action=masquerade protocol=udp dst-port=123 
    ```

## Zdalny dostęp (VPN) - ZeroTier

- utworzono konto na stronie ZeroTier'a: https://www.zerotier.com/
- utworzono sieć
- dodano urządzenia do administracji routerem: laptop, telefon
- pobrano pakiet ze strony Mikrotika - https://mikrotik.com/download:
    - ARM64 / AMPERE -> Extra packages 7.xx.x Stable
    - dodano pakiet do RouterOS
- włączono na routerze instacje ZeroTier'a
- dodano wirtualny interfejs: zerotier-laciem
- dodano zasadę na pierwszym miejscu zapory ogniowej:
```mikrotik
/ip firewall filter add action=accept chain=forward in-interface=zerotier1 place-before=0
/ip firewall filter add action=accept chain=input in-interface=zerotier1 place-before=0
```
- dodano router do sieci ZeroTier
- dodano trasę do podsieci dla serwerów: 172.17.17.0/28 -> 10.147.20.138

## Firewall

- dodano listę adresów wykorzystanych jako blacklisty w regułach zapory:
```mikrotik
add address=0.0.0.0/8 comment=RFC6890 list=not_in_internet
add address=172.16.0.0/12 comment=RFC6890 list=not_in_internet
add address=192.168.0.0/16 comment=RFC6890 list=not_in_internet
add address=10.0.0.0/8 comment=RFC6890 list=not_in_internet
add address=169.254.0.0/16 comment=RFC6890 list=not_in_internet
add address=127.0.0.0/8 comment=RFC6890 list=not_in_internet
add address=224.0.0.0/4 comment=Multicast list=not_in_internet
add address=198.18.0.0/15 comment=RFC6890 list=not_in_internet
add address=192.0.0.0/24 comment=RFC6890 list=not_in_internet
add address=192.0.2.0/24 comment=RFC6890 list=not_in_internet
add address=198.51.100.0/24 comment=RFC6890 list=not_in_internet
add address=203.0.113.0/24 comment=RFC6890 list=not_in_internet
add address=100.64.0.0/10 comment=RFC6890 list=not_in_internet
add address=240.0.0.0/4 comment=RFC6890 list=not_in_internet
add address=192.88.99.0/24 comment="6to4 relay Anycast [RFC 3068]" list=\
    not_in_internet
add address=0.0.0.0/8 comment="defconf: RFC6890" list=no_forward_ipv4
add address=169.254.0.0/16 comment="defconf: RFC6890" list=no_forward_ipv4
add address=224.0.0.0/4 comment="defconf: multicast" list=no_forward_ipv4
add address=255.255.255.255 comment="defconf: RFC6890" list=no_forward_ipv4
add address=127.0.0.0/8 comment="defconf: RFC6890" list=bad_ipv4
add address=192.0.0.0/24 comment="defconf: RFC6890" list=bad_ipv4
add address=192.0.2.0/24 comment="defconf: RFC6890 documentation" list=\
    bad_ipv4
add address=198.51.100.0/24 comment="defconf: RFC6890 documentation" list=\
    bad_ipv4
add address=203.0.113.0/24 comment="defconf: RFC6890 documentation" list=\
    bad_ipv4
add address=240.0.0.0/4 comment="defconf: RFC6890 reserved" list=bad_ipv4
add address=0.0.0.0/8 comment="defconf: RFC6890" list=not_global_ipv4
add address=10.0.0.0/8 comment="defconf: RFC6890" list=not_global_ipv4
add address=100.64.0.0/10 comment="defconf: RFC6890" list=not_global_ipv4
add address=169.254.0.0/16 comment="defconf: RFC6890" list=not_global_ipv4
add address=172.16.0.0/12 comment="defconf: RFC6890" list=not_global_ipv4
add address=192.0.0.0/29 comment="defconf: RFC6890" list=not_global_ipv4
add address=192.168.0.0/16 comment="defconf: RFC6890" list=not_global_ipv4
add address=198.18.0.0/15 comment="defconf: RFC6890 benchmark" list=\
    not_global_ipv4
add address=255.255.255.255 comment="defconf: RFC6890" list=not_global_ipv4
add address=224.0.0.0/4 comment="defconf: multicast" list=bad_src_ipv4
add address=255.255.255.255 comment="defconf: RFC6890" list=bad_src_ipv4
add address=0.0.0.0/8 comment="defconf: RFC6890" list=bad_dst_ipv4
add address=224.0.0.0/4 comment="defconf: RFC6890" list=bad_dst_ipv4
```
- wszystkie zasady zapory ogniowej:
```mikrotik
/ip firewall filter
add action=accept chain=forward in-interface=zerotier-laciem
add action=accept chain=forward comment=\
    "accept established,related,untracked" connection-state=\
    established,related,untracked
add action=accept chain=input comment="accept established, related" \
    connection-state=established,related
add action=drop chain=input connection-state=invalid
add action=fasttrack-connection chain=forward comment=\
    "fast-track for established,related" connection-state=established,related \
    hw-offload=yes
add action=drop chain=forward connection-state=invalid
add action=drop chain=forward comment=\
    "drop access to clients behind NAT from WAN" connection-nat-state=!dstnat \
    connection-state=new in-interface=ether1
add action=drop chain=forward comment=\
    "Drop incoming from internet which is not public IP" in-interface=ether1 \
    log=yes log-prefix=!public src-address-list=not_in_internet
add action=drop chain=forward comment=\
    "Drop packets from LAN that do not have LAN IP" in-interface="LAN BRIDGE" \
    log=yes log-prefix=LAN_!LAN src-address=!172.17.7.0/25
add action=drop chain=forward comment="defconf: drop bad forward IPs" \
    src-address-list=no_forward_ipv4
add action=drop chain=forward comment="defconf: drop bad forward IPs" \
    dst-address-list=no_forward_ipv4
add action=add-src-to-address-list address-list=bruteforce_blacklist \
    address-list-timeout=1d chain=input comment=Blacklist connection-state=\
    new dst-port=2222 protocol=tcp src-address-list=connection3
add action=add-src-to-address-list address-list=connection3 \
    address-list-timeout=30m chain=input comment="Third attempt" \
    connection-state=new dst-port=2222 protocol=tcp src-address-list=\
    connection2,!secured
add action=add-src-to-address-list address-list=connection2 \
    address-list-timeout=15m chain=input comment="Second attempt" \
    connection-state=new dst-port=2222 protocol=tcp src-address-list=\
    connection1
add action=add-src-to-address-list address-list=connection1 \
    address-list-timeout=5m chain=input comment="First attempt" \
    connection-state=new dst-port=2222 protocol=tcp
add action=accept chain=input dst-port=2222 protocol=tcp src-address-list=\
    !bruteforce_blacklist
add action=drop chain=input comment="block everything" in-interface=ether1
```

## VLAN'y

Zmienione podsieci i skonfigurowane VLANy:
- 172.17.7.0/25 - VLAN 7 - Guest WIFI
- 172.17.17.0/28 - VLAN 17 - Serwery
- 172.17.71.0/29 - VLAN 71 - Managment VLAN
- VLAN 777 - Blackhole

- utworzono nowy mostek Blackhole dla nieużywanych portów
- porty w mostku LAN BRIDGE: Ether5, Wifi 2.4GHz, Wifi 5GHz
- porty w mostku Blackhole: Ether2-Ether4
- dla każdego portu zmieniono im PVID: Bridge -> Ports -> VLAN -> PVID
- dodano VLANy do mostka LAN BRIDGE w sekcji: Bridge -> VLANs
    - 7, tagged: LAN BRIDGE, untagged: Wifi 2.4GHz
    - 17, tagged: LAN BRIDGE, untagged: Ether5
    - 71, tagged: LAN BRIDGE, untagged: Wifi 5ghz
    - 7, tagged: LAN BRIDGE, untagged: Wifi 2.4GHz
- dodano VLANy do mostka Blackhole w sekcji: Bridge -> VLANs
    - 777, tagged: Blackhole, untagged: Ether2-Ether4
- dodano VLANy do mostka LAN BRIDGE i Blackhole w sekcji: Interface (Konfiguracja tak samo jak w punktach powyżej):
    - LAN BRIDGE:
        - 07-VLAN-Guest-Wifi
        - 17-VLAN-Servers
        - 71-VLAN-Managmen
    - Blackhole:
        - 777-VLAN-Blackhole
- dodano adresy do utworzonych wirtualnych interfejsów:
    - 172.17.7.1/25 - 07-VLAN-Guest-Wifi
    - 172.17.17.1/28 - 17-VLAN-Servers
    - 172.17.71.1/29 - 71-VLAN-Managment
- zmieniono interfejs serwera DHCP z LAN BRIDGE Na 07-VLAN-Guest-Wifi
- włączono opcje VLAN filtering na mostku: LAN BRIDGE

VLANy są skonfigurowane by sieć gości nie miała dostępu do podsieci do zarządzania sprzętem oraz serwerami.

- dodano zasady do zapory ogniowej:
``` mikrotik
/ip/firewall/filter
add action=accept chain=forward \
    dst-address=172.17.17.0/28 src-address=\
    172.17.71.0/29
add action=accept chain=forward \
    dst-address=172.17.71.0/29 src-address=\
    172.17.17.0/28
add action=drop chain=forward \
    dst-address-list=LAN-Subnets \
    src-address-list=LAN-Subnets
```
- dodano trasy statyczne dla podsieci za wirtualną zaporą ogniową OPNsense:
    - 172.17.10.0/28 -> 172.17.17.3
    - 172.17.20.0/28 -> 172.17.17.3
    - 172.17.30.0/28 -> 172.17.17.3

## Żródła 
1. Dokumentacja: https://help.mikrotik.com/docs/
    - budowa zapory ogniowej:
        - https://help.mikrotik.com/docs/display/ROS/Building+Your+First+Firewall
        - https://help.mikrotik.com/docs/display/ROS/Building+Advanced+Firewall
        - https://help.mikrotik.com/docs/display/ROS/Bruteforce+prevention
2. Winbox, pakiet ZeroTier: https://mikrotik.com/download
3. Kanał na YT Mikrotika: https://www.youtube.com/@mikrotik
    - Instalacja Winboxa na Linuxie: https://www.youtube.com/watch?v=2jjtK5Me29I 
    - Konfiguracja DoH: https://www.youtube.com/watch?v=w4erB0VzyIE
    - Konfiguracja ZeroTier: https://www.youtube.com/watch?v=60uIlyF8Z5s
    - Konfiguracja logowania SSH: https://www.youtube.com/watch?v=be-pBwhjRWA
