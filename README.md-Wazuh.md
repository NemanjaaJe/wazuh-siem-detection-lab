# Wazuh SIEM Detection Lab

Home-lab projekat u kojem je Wazuh SIEM povezan sa dva Windows uređaja (agent na drugom laptopu, manager/monitoring na mom) sa ciljem da se testira detekcija sumnjivih administrativnih i autentifikacionih radnji.

## Cilj projekta

Cilj je bio praktično proveriti kako Wazuh reaguje na tipične indikatore kompromitacije (IOC) na Windows sistemu — kreiranje naloga, eskalaciju privilegija, izvršavanje skripti sa zaobiđenom bezbednosnom politikom i pokušaj brisanja tragova — i dokumentovati koji alert-i se generišu za svaku radnju.

## Arhitektura lab-a

| Komponenta | Uloga |
|---|---|
| Laptop 1  | Wazuh manager / dashboard |
| Laptop 2  | Wazuh agent (Windows) — meta test radnji |

Test komande su izvršavane lokalno na agent mašini, dok je monitoring i provera alert-a rađena preko Wazuh dashboard-a na manager mašini.

## Test scenariji

| # | Test radnja | Komanda | MITRE ATT&CK tehnika |  
|---|---|---|---|---|
| 1 | Kreiranje lokalnog korisničkog naloga | `net user hacker Test123! /add` | T1136.001 – Create Account: Local Account | 
| 2 | Dodavanje naloga u Administrators grupu | `net localgroup administrators hacker /add` | T1098.007 – Account Manipulation: Additional Local or Domain Group Membership | 
| 3 | PowerShell izvršavanje uz zaobilaženje Execution Policy | `powershell -ExecutionPolicy Bypass -Command "Get-Process"` | T1059.001 – Command and Scripting Interpreter: PowerShell |
| 4 | Brisanje Security event log-a | `wevtutil cl Security` | T1070.001 – Indicator Removal: Clear Windows Event Logs | 



## Detaljan opis testova

### 1. Kreiranje sumnjivog korisničkog naloga
Kreiran je novi lokalni nalog `hacker` komandom `net user hacker Test123! /add`. Ova radnja simulira prvi korak napadača koji je dobio pristup sistemu i želi da napravi backdoor nalog za dalji pristup. Wazuh prati promene u lokalnim nalozima preko Windows Event Log kanala (Security log, Event ID 4720).

### 2. Eskalacija privilegija
Novokreirani nalog je dodat u lokalnu Administrators grupu komandom `net localgroup administrators hacker /add`. Ovo predstavlja eskalaciju privilegija — napadač koji ima običan nalog pokušava da sebi obezbedi administratorska prava. Odgovarajući Windows Event ID je 4732 (član dodat u privilegovanu grupu).

### 3. Izvršavanje PowerShell komande uz Bypass politiku
Izvršena je komanda `powershell -ExecutionPolicy Bypass -Command "Get-Process"` koja zaobilazi podrazumevanu PowerShell Execution Policy. Ovo je čest obrazac kod malicioznih skripti i "living-off-the-land" tehnika, jer napadači koriste legitimne alate (PowerShell) da izbegnu detekciju baziranu na fajlovima.

### 4. Brisanje Security event log-a (anti-forenzika)
Komandom `wevtutil cl Security` obrisan je Windows Security event log. Ova radnja simulira pokušaj napadača da prikrije tragove nakon što je izvršio prethodne korake. Wazuh detektuje ovu radnju preko Event ID 1102 ("The audit log was cleared"), što je jedan od najkritičnijih alert-a u SIEM okruženju jer direktno ukazuje na pokušaj prikrivanja aktivnosti.


## Zaključak

Lab je pokazao da Wazuh, uz podrazumevana pravila, uspešno detektuje ključne faze jednostavnog napadačkog scenarija — od kreiranja naloga, preko eskalacije privilegija, do pokušaja prikrivanja tragova. Ovo je osnovni primer kako SIEM alati doprinose vidljivosti (visibility) u mreži i omogućavaju bezbednosnim timovima da reaguju na sumnjive radnje u realnom vremenu.

## Korišćeni alati

- Wazuh (manager + agent)
- Windows 10/11 (test okruženje)
- PowerShell / CMD

## Autor

Nemanja — student, u procesu izgradnje portfolija za cybersecurity oblast.
