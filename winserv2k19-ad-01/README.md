![Logo Windowsa](https://www.startpage.com/av/proxy-image?piurl=https%3A%2F%2Fgitlab.ics.muni.cz%2Fuploads%2F-%2Fsystem%2Fproject%2Favatar%2F1939%2Fwindows-server-logo.png&sp=1717330678T9010a351fd91c27d60f1a010627868c1d70bca6a7aa781d64ce282dbf1ae36c4)

# Konfiguracja Windows Server 

## Wstępna konfiguracja obu hostów (Przed dołączniem do domeny)

**System był konfigurowany w języku angielskim**

**Panel sterowania:**
- zmieniono opcje UAC na Always notify (System and Security -> Security and Maintenance -> Security -> Change settings)
- zmieniono strefę czasową na: (UTC+01:00) Amsterdam, Berlin, Bern, Rome, Stockholm, Vienna (win + s (skrót klawiszowy) -> (wpisz) timedate.cpl -> ctrl + shift + enter(by uruchomić przystawkę jako administrator)
- wyłączono network discovery (Network and Internet -> Network and Sharing Center -> Change advanced sharing settings -> Private, Guest or Public, Domain, zaznaczyć Turn off network discovery)
- utworzono nowego użytkownika do administracji: user (Computer Managment -> Local Users and Groups -> Users -> PPM -> New User):
    - odhaczono: User must change password at next logon
    - zaznaczono: User cannot change password

**Właściwości systemu (win + s -> View Advanced system settings):**
- Computer Name:
    - zmieniono nazwę serwera na: AD-01/AD-02:
        - Change -> 
- Remote:
    - pozwolono na zdalne połączenie za pomocą RDP użytkownika: user
        - Allow remote connections to this computer
        - Allow connections only from computers running Remote Desktop with Network Level Authentication
        - Select Users -> Add -> OK


**Konfiguracja IP (win + x -> Network Connections -> Change adapter options -> PPM na adapter -> Properties)**
- wyłączono na interfejsie IPv6 (generowanie adresu link-local) (odhaczenie checkboxa: Internet Protocol Version 6)
- wyłączono LLDP:
    - odhaczenie:
        -Microsoft LLDP Protocol Driver
        -Link-Layer Topology Discovery Responder
        -Link-Layer Topology Discovery Mapper I/O Driver
**Adresacja IP (Double click -> Internet Protocol Version 4):**
- ustawiono:
    - IP: 172.17.30.\[2-3]
    - Maska: 255.255.255.240
    - Brama: 172.17.30.1
    - DNS: 127.0.0.1, 172.17.17.1
**W zakładce advanced -> WINS:**
    - wyłączono NetBIOS over TCP/IP (Enable LMHOSTS lookup - odhaczono, Disable NetBIOS over TCP/IP)
- wyłączono automount (CMD): diskpart.exe -> automount disable
- zainstalowano przeglądarkę Firefox
- tylko podstawowe dane diagnostyczne będą przesyłane do Microsoftu

## AD 

**Instalacja AD (Server Manager -> Manage -> Add Roles and Features):**
- Server Roles -> Active Directory Domain Services
- Zainstalowano rolę i zrestartowano hosta

**Konfiguracja domeny (Server Manager -> Flaga -> Promote this server to a domain controller):**
- Deployment configuration
    - Add a new forrest
    - nazwa domeny: dziki.lab

**Dodanie elementów do domeny:**
- dołączono do domeny:
    - laptop DESKTOP-ADN0RLH
    - serwer NAS: OPENMEDIAVAULT
- dodano jednostkę organizacyjną: new-users
- dodano użytkownika zzikson w OU new-users:
    - kreator użytkownika:
        - odhaczono: User must change password at next logon
        - zaznaczono: User cannot change password
    - właściwości:
    - godziny logowania Pon-Piąt 06:00-16:00 (Account -> Logon Hours)
    - logowanie jedynie za pomocą: DESKTOP-ADN0RLH (Account -> Log On To)
    - profil i folder domowy podłączony pod udostępniony folder SMB z serwera OpenMediaVault
    - odznaczono Enable remote control (Remote control)
- dodano użytkownika server-share:
    - kreator użytkownika:
        - odhaczono: User must change password at next logon
        - zaznaczono: User cannot change password, Password never expires
    - właściwości:
        - logowanie jedynie za pomocą: OPENMEDIAVAULT (Account -> Log On To)
        - odznaczono: Enable remote control (Remote control)

## DNS

**Konfiguracja:**
- aging/scavenging time (w panelu na lewo: PPM \[Nazwa serwera DNS] -> Set Aging/Scavenging for All Zones):
    - No-refresh interval: 4 days
    - Refresh interval: 4 days
- właściwości serwera (PPM \[Nazwa serwera DNS] -> Properties):
    - dodano forwarder: 172.17.17.1 (zakładka Forwarders)
- właściwości strefy wyszukiwania do przodu (DNS -> AD-01 -> Forward Lookup Zones -> PPM \[Nazwa strefy] -> Properties)
    - wyłączono WINS (zakładka WINS):
        - odhaczono: Use WINS forward lookup
**Dodano rekordy DNS:**
- dzikus -> 10.0.0.1 - A
- dzikuss -> dzikus - CNAME

## Shadow Copy

**Dodanie partycji przeznaczonej na snapshot'y:**
- zarządzanie dyskami:
    - system plików: NTFS
    - etykieta: Shaddow Copz

**Konfiguracja Shadow Copy (Eksplorator plików -> PPM na dysk C: -> Configure Shadow Copies):**
- zaznaczono dysk F:
    - Settings:
        - Use limit: 1000 MB
        - Schedule:
            - Schedule Task: Daily
            - Start time: 7:00 AM
            - Schedule Task Daily: Every 1 day
            - Advanced:
                - Repeat task:
                    - Every 2 Hours
                    - Duration 9 hours 0 minutes
    - Enable

## FSRM (File Server Resource Manager)

**Instalacja FSRM (Server Manager -> Manage -> Add Roles and Features):**
- Server Roles -> File And Storage Services -> File Server Resource Manager

**Dodanie partycji przeznaczonej na udostępnione foldery:**
- zarządzanie dyskami:
    - system plików: NTFS
    - etykieta: Shared Folders

**Utworzono i udostępniono folder (PPM na utworzony folder -> Właściwości):**
- W zakładce Sharing -> Advanced Sharing:
    - nazwa udostępnionego folderu: FS-01$ (Folder ukryty)
    - uprawnienia: zapis i odczyt u użytkownika zzikson

**Konfiguracja FSRM:**
- dodanie Quoty do poprzednio utworzonego albumu(FSRM -> Quota Managment -> Quotas)
    - limit 5MB
    - powiadomenie przy zajęciu 85% miejsca na dysku
- Ograniczenie 
- konfiguracja white-listy dla obrazków (File Screening Managment):
    - File Groups - nowe-zdjecia:
        - Files to Include: *
        - Files to Exclude: *.png, *.jpeg
    - File Screen Templates - nowe-zdjecia:
        - Screening type:
            - Active Screening
        - File groups: nowe-zdjecia
    - File Screens:
        - Deriver properties from this file screen template: nowe-zdjecia

## GPO 

**Obiekty dla wszystkich hostów (Forest: dziki.lab -> Domains -> dziki.lab):**
- Administrative Templates - Computer Configuration (Computer Configuration -> Policies -> Administrative Templates):
    - Control Panel:
        - Personalization:
            - Prevent changing lock screen and logon image: Enabled
            - Prevent changing start menu background: Enabled
            - Do not display the lock screen: Disabled
            - Prevent enabling lock screen camera: Enabled 
        - Regional and Language Options:
            - Allow users to enable online speech recognition services: Disabled
        - Network Connections:
            - Prohibit installation and configuration on Network Bridge on your DNS domain network: Enabled
            - Prohibit use of Internet Connection Sharing on your DNS domain network: Enabled
        - Windows Components:
            - Add features to Windows 10:
                - Prevent the wizard form running: Enabled
            - Camera:
                - Allow Use of Camera: Disable

- Security Settings (Computer Configuration -> Policies -> Windows Settings -> Security Settings):
    - Account Policies:
        - Password Policies:
            - Enforce password history: 24 passwords remembered
            - Maximium password age: 31 days
            - Minimum password age: 31 days
            - Minimum password length: 14 characters
            - Minimum password age: 31 days
            - Password must meet complexity requirements: Enabled
        - Account Lockout Policy:
            - Account lockout duration: 10 minutes
            - Account lockout threshold: 3 invalid logon
            - Allow Administrator account lockout: Enabled
            - Reset account lockout counter after: 10 minutes
    - Local Policies:
        - Audit Policy:
            - Audit account logon events: Success, Failure
            - Audit account management: Success, Failure
            - Audit directory service access: Success, Failure
            - Audit logon events: Success, Failure
            - Audit privilege use: Success 
        - User Rights Assignment:
            - Allow log on through Remote Desktop Services: user
            - Back up files and directories: Administrators
            - Deny log on through Remote Desktop Services: Local account, Guests
            - Increase scheduling priority: Administrators
            - Manage auditing and security log: Administrators
            - Restore files and directories: Administrators
        - Security Options:
            - Accounts: Administrator Account status: Disabled
            - Accounts: Block Microsoft accounts: Users can't add or log on with Microsoft accounts 
            - Accounts: Guest account status: Enabled
            - Interactive logon: Machine inactivity limit: 300 seconds
            - User Account Control: Admin Approval Mode for the Built-in Administrator account: Enabled 
    - Event Log:
        - Prevent local guest group from accessing application log: Enabled

## Serwer drukowania

**Instalacja serwera (Server Manager -> Manage -> Add Roles and Features):**
- Server Roles -> Print and Document Services
- Role Services -> Print Server

Pobrano sterowniki do drukarki: https://ftp.hp.com/pub/softlib/software13/printers/SS/SL-M3370FD/SamsungUniversalPrintDriver3.exe

Zainstalowano na serwerze sterowniki

**Dodano sterowniki do serwera wydruku (Print managment -> Print Servers -> \[Nazwa serwera wydruku] -> Drivers):**
- Kreator dodawania sterownika (PPM na środkowym panelu -> Add driver)
    - Printer Driver Selection -> Samsung -> Samsung Universal Print Driver 3
**Dodano drukarkę (Print managment -> Print Servers -> \[Nazwa serwera wydruku] -> Printers):**
- Kreator dodawania drukarki (PPM na środkowym panelu -> Add Printer):
    - Printer Installation:
        - Add a new printer using an existing port: USB001
    - Printer Driver:
        - Use an existing printer driver on the computer: Samsung Universal Print Driver 3

**Wdrożono drukarki za pomocą GPO:**
- Utworzono i podłączono nowy obiekt - Printer Deployment (Forest: dziki.lab -> Domains -> dziki.lab -> new-users): 
- Edycja GPO (User Configuration -> Control Panel Settings -> Printers)
    - New Shared Printer Properties (PPM na panel na prawo):
        - Action: Create
        - Share Path -> Search result -> \[Nazwa udostępnionej drukarki]
        - Set this printer as the default printer

## WSUS

**Instalacja WSUS (Server Manager -> Manage -> Add Roles and Features):**
- Server Roles -> Windows Server Update Services
- Role Services -> Print Server 

**Konfiguracja WSUS (Kreator):**
- Microsoft Update Improvement Program:
    - odhaczono: Yes I would like to join the Microsoft Update Improvement Program
- Specify proxy server:
    - Start Connecting
- Choose Language:
    - Download updates only in these languages -> English, Polish
- Choose Products:
    - odhaczono:
        - aktualizacje dla systemów poniżej wersji windows 10 i windows server 2016
        - Antigen
        - Azure IoT Edge for Linux on Windows
        - Bing
        - BizTalk Server
        - Exchange
        - Expression
        - Forefront
        - Internet Security and Acceleration Server
        - Microsoft Online Services
        - Microsoft StreamInsight
        - Office Communications Server And Office Communicator
        - Office (Oprócz Office 2016)
        - Silverlight
        - Skype for Business
        - Virtual Server 
        - Windows Embedded
        - Windows Essential Business Server
        - Windows Live
        - Windows Small Business Server
        - Windows Subsystem for Linux
        - Works
- Choose Classification:
    - dodatkowo zaznaczono:
        - Feature Packs
        - Service Packs
        - Tools
        - Update Rollups
        - Updates
- Configure Sync Schedule:
    - Synchronize automatically:
        - First synchronization: 2:00:00 AM
        - Synchronizations per day 2
**Konfiguracja GPO**
- Utworzono i podłączono nowy obiekt - Printer Deployment (Forest: dziki.lab -> Domains -> dziki.lab): 
- Edycja GPO (Computer Configuration -> Policies -> Administrative Templates -> Windows Components -> Windows Update)
    - Specify intranet Microsoft update service location:
        - Enabled
        - Set the intranet update service for detecting updates: http://AD-01:8530
        - Set the intranet statistics server: http://AD-01:8530
    - Enable client-side targeting
        - Utworzono grupę WSUS-Computers i dodano hosta: DESKTOP-ADN0RLH
        - Target group name for this computer: WSUS-Computers

## DFS

**Instalacja DFS (Server Manager -> Manage -> Add Roles and Features)**:
- Server Roles -> File And Storage Services -> File and iSCSI Services -> DFS Namespaces

Dodanie przstrzeni nazw Distributed File System (PPM na DFS Managment -> New Namespace):
- Namespace Server - Server: AD-01
- Namespace Name and Settings - Name: Miejsce-Na-Zdjecia

## Instalacja i konfiguracja zapasowego serwera AD

Po instalacji serwera, podłączono go do domeny. Instalacja AD przebiegła tak jak przy serwerze AD-01.

**Konfiguracja serwera (Server Manager -> Flaga -> Promote this server to a domain controller):**
- Deployment configuration
    - Add a domain controller to an existing domain
    - Domain: dziki.lab
- Additional Options:
    - Relicate from AD-01.dziki.lab

**Źródła:**
- zmiana strefy czasowej: https://youtu.be/ZQmkBY3SgK8
- konfiguracja licznika aging, scavenging: https://youtu.be/4EmeijbM8X4
- FSRM: https://youtu.be/eTCTGCi3uHg https://youtu.be/Xv7WOII9wt8
- serwer WSUS: https://youtu.be/VTCzszyiFz4
- serwer wydruku: https://piped.adminforge.de/watch?v=xZY4C4zMHlw
- sterowniki do drukarki: https://support.hp.com/pl-pl/drivers/samsung-ml-1675-laser-printer-series/19134532
