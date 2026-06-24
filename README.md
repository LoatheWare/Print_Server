# Print Server i konfigurisanje Print servera na Windows Server 2019 u VirtualBox okruženju
## Šta je ovo?
Ovo je vežba za administratore računarskih mreža koju sam radio tokom mog školovanja. Pocetnja stanja i potrebne fajlove nisam u mogucnosti da obezbedim, ali za ovo će Vam sigurno biti potreban VirtualBox, Windows Server 2019 .iso fail, Windows 10 .iso fajl, Linux distribucija po ukusu, mi smo radili sa OpenSuse. Zadak Vam je dat, kao i uputstvo za njegovo rešavanje. Ukoliko Vam nešto nije jasno tokom rešavanja ovih zadataka obratite mi se direktnom porukom na aplikaciji LinkedIn, www.linkedin.com/in/nikola-karanović-397185390

## Pitanja i odgovori — Print Server na Windows Server 2019

### 1. Šta je Print Server i čemu služi?
**Print Server** je server (ili uloga na serveru) koji **centralizuje upravljanje štampačima** u mreži — prihvata print job-ove (zadatke za štampu) od klijentskih računara, redja ih u red čekanja (queue) i prosleđuje ih ka fizičkim ili virtuelnim štampačima. Umesto da se svaki štampač instalira pojedinačno na svaki klijentski računar, klijenti se povezuju na **deljeni (shared) štampač** preko servera, čime se administracija znatno olakšava.

---

### 2. Koje su prednosti korišćenja Print Servera?
Print Server nam omogućava:
1. **Centralizovano upravljanje** — instalacija drajvera, dodavanje/uklanjanje štampača i podešavanje dozvola radi se na jednom mestu, umesto na svakom klijentu pojedinačno.
2. **Lakšu distribuciju štampača korisnicima** — štampači se mogu automatski dodeliti korisnicima/računarima preko **Group Policy**-ja, bez potrebe za ručnom instalacijom na svakoj mašini.
3. **Kontrolu pristupa i dozvola** — moguće je definisati ko ima pravo da štampa, upravlja redom čekanja, ili administrira sam štampač (Print, Manage Documents, Manage Printer dozvole).
4. **Praćenje i upravljanje redom čekanja (queue)** — administrator može centralno da vidi, pauzira, otkaže ili promeni redosled print job-ova za sve korisnike.
5. **Smanjenje opterećenja klijentskih računara** — obradu print job-a (spooling) preuzima server, a ne klijentska mašina.

---

### 3. Koja je uloga (role) potrebna za konfigurisanje Print Servera na Windows Server 2019?
Potrebna je uloga **Print and Document Services**, koja se instalira preko **Server Manager-a** (Add Roles and Features) ili preko PowerShell-a:

```powershell
Install-WindowsFeature -Name Print-Server -IncludeManagementTools
```

Ova uloga uključuje **Print Management** konzolu, koja se koristi za centralizovano upravljanje štampačima, drajverima i serverima.

---

### 4. Šta je Print Spooler servis i zašto je važan?
**Print Spooler** je Windows servis koji **prihvata, redja u red čekanja i upravlja print job-ovima** pre nego što ih prosledi štampaču. Ako ovaj servis nije pokrenut, štampanje **neće biti moguće**, bez obzira na to koliko je sam štampač ispravno konfigurisan. Status servisa se može provjeriti i restartovati preko `services.msc` ili PowerShell-a:

```powershell
Get-Service -Name Spooler
Restart-Service -Name Spooler
```

---

### 5. Kako se dodaje (instalira) štampač na Print Serveru?
Štampač se dodaje kroz **Print Management** konzolu ili kroz **Devices and Printers** u Control Panel-u:
1. Otvoriti **Print Management** (`printmanagement.msc`).
2. Desni klik na **Print Servers > [naziv servera] > Printers**, izabrati **Add Printer**.
3. Izabrati tip porta (npr. **Local port**, **TCP/IP port** za mrežne štampače, ili **Create a new port**).
4. Izabrati/instalirati odgovarajući **drajver** za model štampača.
5. Definisati ime štampača i, ukoliko je potrebno, deliti ga (share) na mreži.

U laboratorijskom (VirtualBox) okruženju, najčešće se koristi **virtuelni (lokalni) štampač** ili **Microsoft-ov generički/PDF drajver**, jer fizički štampač nije dostupan u virtuelnom okruženju.

---

### 6. Kako se deli (share) štampač na mreži?
Da bi klijenti mogli da pristupe štampaču preko mreže, potrebno je da bude **deljen (shared)**. Ovo se podešava u svojstvima štampača:

Devices and Printers > [desni klik na štampač] > Printer Properties > Sharing tab > Share this printer

Pri čemu se definiše i **Share name** — naziv pod kojim će se štampač vidljiv klijentima na mreži (npr. `\\PRINTSERVER\HP-LaserJet`).

---

### 7. Kako klijent (Windows 10) pristupa deljenom štampaču na serveru?
Na klijentskoj mašini, štampač se može povezati na nekoliko načina:
- Preko **UNC putanje** direktno u **Run** dijalogu ili **File Explorer**-u:

\PRINTSERVER\HP-LaserJet

- Preko **Settings > Devices > Printers & scanners > Add a printer or scanner**, gde se traži dostupan mrežni štampač.
- Preko **Group Policy**-ja (automatska dodela, bez intervencije korisnika — vidi pitanje 9).

---

### 8. Koje su osnovne dozvole (permissions) koje se mogu dodeliti korisnicima za štampač?
U **Security** tabu svojstava štampača definišu se tri nivoa dozvola:
- **Print** — korisnik može da šalje dokumente na štampanje.
- **Manage Documents** — korisnik može da upravlja redom čekanja (pauziranje, brisanje, promena redosleda) za sve dokumente, ne samo svoje.
- **Manage Printer** — korisnik (administrator) može da menja konfiguraciju samog štampača, dozvole, drajvere i da ga deli/ukloni sa mreže.

---

### 9. Kako se štampač automatski dodeljuje korisnicima preko Group Policy-ja?
Da bi se izbeglo ručno povezivanje na štampač sa svake klijentske mašine, koristi se **Group Policy Deployed Printers**:
1. Otvoriti **Group Policy Management** konzolu na serveru.
2. Kreirati novi GPO (ili koristiti postojeći) i povezati ga (link) sa odgovarajućom OU (Organizational Unit) u kojoj se nalaze korisnici/računari.
3. Otvoriti GPO za uređivanje (Edit) i otići na:

User Configuration > Preferences > Control Panel Settings > Printers

4. Desni klik > **New > Shared Printer**, uneti UNC putanju štampača (npr. `\\PRINTSERVER\HP-LaserJet`) i izabrati akciju **Create** ili **Update**.
5. Sačekati da se Group Policy primeni na klijentima (ili pokrenuti `gpupdate /force` na klijentu radi odmah primene).

---

### 10. Koja je komanda za prinudno ažuriranje Group Policy-ja na klijentu radi provere dodeljenog štampača?
Na klijentskoj (Windows 10) mašini, pokreće se sledeća komanda u Command Prompt-u ili PowerShell-u:

gpupdate /force

Nakon ovoga, dodeljeni štampač preko GPO-a treba da se automatski pojavi u listi instaliranih štampača na klijentu, bez potrebe za ručnim dodavanjem.

---

### 11. Koje su PowerShell komande za upravljanje štampačima na serveru?
Najkorisnije PowerShell komande za administraciju Print Servera:

```powershell
Get-Printer
Get-PrinterDriver
Add-Printer -Name "HP-LaserJet" -DriverName "HP LaserJet" -PortName "LPT1:"
Remove-Printer -Name "HP-LaserJet"
Get-PrintJob -PrinterName "HP-LaserJet"
Remove-PrintJob -PrinterName "HP-LaserJet" -ID 1
```

- `Get-Printer` — prikazuje listu svih instaliranih štampača na serveru.
- `Get-PrinterDriver` — prikazuje listu instaliranih drajvera za štampače.
- `Add-Printer` / `Remove-Printer` — dodaje, odnosno uklanja štampač sa servera.
- `Get-PrintJob` / `Remove-PrintJob` — prikazuje, odnosno briše konkretan zadatak (job) iz reda čekanja štampača.

---

### 12. Koji su najčešći problemi prilikom konfigurisanja Print Servera i kako se rešavaju?
- **Print Spooler servis nije pokrenut** — provjeriti status servisa (`Get-Service Spooler`) i pokrenuti ga ako je zaustavljen; ponekad je potrebno i restartovati servis ako su job-ovi "zaglavljeni" u redu čekanja.
- **Klijent ne može da pristupi deljenom štampaču** — provjeriti da li je štampač zaista deljen (Sharing tab), da li klijent ima odgovarajuću **Print** dozvolu, i da li postoji mrežna konekcija (firewall ne blokira **File and Printer Sharing**).
- **Pogrešan ili nekompatibilan drajver** — ako klijent koristi drugu arhitekturu (npr. 32-bit vs 64-bit) u odnosu na server, potrebno je na serveru dodati **dodatni drajver** za tu arhitekturu (Print Management > Drivers > Add Driver).
- **Štampač dodeljen preko GPO-a se ne pojavljuje na klijentu** — provjeriti da li je GPO povezan sa odgovarajućom OU, da li je korisnik/računar zaista član te OU, i pokrenuti `gpupdate /force` na klijentu radi trenutne primene politike.
- **Print job "zaglavljen" u redu čekanja** — obrisati job preko `Remove-PrintJob`, ili u krajnjem slučaju, restartovati Print Spooler servis (čime se briše ceo red čekanja).

---
