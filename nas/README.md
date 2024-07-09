## Konfiguracja

Poniżej użyte sekcje panelu administracyjnego do konfiguracji serwera

**System:**
- panel administracyjny używa certfikatu SSL
- ustawiono automatyczne wylogowywanie po 5 minutach bezczynności
- ustawiono strefę czasową: Europe/Warsaw
- zainstalowano pluginy:
    - filebrowser - interfejs webowy 
    - flashmemory
    - lvm
    - omvextrasorg

**Services:**
- skonfigurowano logowanie ssh przez klucz prywatny

**Storage:**
- LVM:
    - utworzono fizyczny wolumin: /dev/md0
    - utworzono grupę woluminów: VG-01
    - utworzono logiczne woluminy:
        - LV-01-Family-Fileshare - 25GB
        - LV-02-AD-Profiles - 10GB
        - LV-03-WSUS - 10GB
- Software RAID:
    - utworzono z 3 wirtualnych dysków 50GB macierz w RAID 5. Łączny rozmiar to 100GB
- File Systems:
    - zformatowano wszystkie logiczne woluminy systemem BTRFS
- Shared Folders:
    - AD-Profiles
    - Family-Fileshare
    - WSUS

**Services:**
- File Browser:
- dodano dwa udziały: AD-Profiles, WSUS
    - port: 3670
    - wygenerowany certyfikat SSL
- SMB:
    - włączono SMB 
    - dodano dwa udziały: AD-Profiles, WSUS
- SSH:
    - port: 2223
    - włączone jedynie uwierzytelnienie przez klucz publiczny  

**Users:**
- dodano nowych użytkowników:
    - family-share - dla File Browser
    - server-share - dla SMB
    - zzikson - do administracji serwera przez SSH

- skonfigurowano adresacje IP:
    - IP i maska podsieci: 172.17.10.2/28
    - brama domyślna: 172.17.10.1
    - IPv6: wyłączone
    - DNS:
        - 172.17.30.2 
        - 172.17.30.3 
        - 172.17.17.1 

- dodano serwer do Active Directory według poradnika: https://forum.openmediavault.org/index.php?thread/42307-omv-6-x-rc1-active-directory/


**Źródła:**
- dołączenie serwera do domeny: https://forum.openmediavault.org/index.php?thread/42307-omv-6-x-rc1-active-directory/
