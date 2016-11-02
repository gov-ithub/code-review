---
title: 'Code Review'

layout: nil
---

# General

- Codul poate avea review de la mai multe persoane, cel puțin una (dar ideal mai mult de o persoană)
- Unora le poate place mai mult să facă review, altora nu
- Review-urile ar trebui să fie prioritizate, de la câteva minute până la câteva ore după pull request
- Ca și contribuitor, vei fi selectat ca și reviewer îndeosebi pe proiectele tale, dar e posibil să fi selectat pentru ceva total în afara proiectului/proiectelor tale

## Pentru revieweri
- Nu încerca să transformi codul la care faci review în propriul cod (vezi https://en.wiktionary.org/wiki/bikeshedding)
- Job-ul tău ca și reviewer este la fel de important ca și acela de a contribui cod
- E mai bine să iei o decizie rapidă (ie: nu accepta) decât să ții un pull request în "limbo".  Cand ai primit un cod review inseamna ca un coleg a terminat, si asteapta dupa tine.
- E indicat să te retragi dintr-un review dacă nu ești persoana potrivită sau nu ai timp să faci review.
- Pentru orice probleme cu codul la care ai făcut review împarți responsabilitatea cu autorul codului.
- Indica clar daca esti de acord ca codul sa fie checked-in sau trebuie modificat. Daca ai intrebari, dar codul este ok pentru check-in, foloseste un alt canal de comunicare cu autorul, nu code review.

## Pentru cei care primesc review:
- Este un review pe cod, nu e personal
- Încearcă să-ți ajuți review-erii cât poți de mult. Asta înseamnă un summary adecvat, test plan, revert plan și evident rulează suita de teste înainte de pull request
- Fă un self-review înainte de pull request, asigură-te că nu ai uitat cod comentat și că respecți convențiile
- Un pull request ar trebui să introducă un singur feature, nu mai multe. Încearcă să optimizezi pentru mai multe pull request-uri mai mici, nu pentru câteva pull request-uri mari.
- Fiecare pull request care introduce un feature nou sau cod netestat, ar trebui să fie însoțit de cel puțin un unit test sau modificările necesare la suita de teste existentă.
- Când îți alegi review-erii, încearcă să-i găsești pe cei care sunt cei mai familiarizați cu produsul la care faci pull request (git blame FTW)
- Un număr de 2 până la 4-5 review-eri dă cele mai bune rezultate (eviți bikeshedding și ai mai multe șanse la un review rapid)

# Niște elemente comune pentru orice review
- Există o posibilitate de SQL Injection în noul cod? Nu contează dacă sursa de input este 'intern' sau 'extern', cod care este susceptibil de SQL Injection nu poate fi acceptat. La fel și pentru alte găuri de securitate (XSS, etc.)
- Pentru cazurile care se aplică, verifică de două ori codul care poate avea  buffer overflow/underflow și value overflow/underflow. Codul nativ trebuie să folosească librării safe, toate funcțiile care acceptă buffer trebuie să accepte și length etc.
- Codul submis conține secrete? API keys, user passwords, internal IPs etc. Secretele trebuiesc totdeauna citite din input, config sau environment.
- În buclele for și while, condițiile de  exit sunt corecte? Aici sunt adesea erori off-by-one, null terminator sau infinite loop
- Există bucle care au în interior bucle? Algoritmii O(N<sup>2</sup>) sau O(N<sup>3</sup>) au probleme când numărul de iterații crește.
- Alocările din noul cod sunt bounded? Se poate ca noul cod să aloce memorie ad-nauseam? Poate crea un fișier enorm? etc.
- Apelurile recursive (inclusiv indirect) sunt bounded? Verifică condiția de exit.
- Input-ul codului este bounded? Dacă citește un fișier, cum va funcționa cu un fișier de 5Gb? Citește dintr-o tabelă, cum va reacționa la un result set de 1 milion de recorduri? Când manipularea se poate face prin stream-uri, să nu se folosească copii (citește fișier în memorie) dacă dimensiunile nu sunt clar controlate (și destul de mici).
- Artefactele create de noul cod, cum vor fi gestionate/șterse/recuperate? Nimic nu are capacitate infinită. De ex:
  - Creează un nou fișier de log, cine/când îl va recicla? S-a adăugat o nouă regulă în logrotate? 
  - Dacă inserează un row într-o bază de date, când va fi șters?
- Codul folosește API-uri care cer permisii elevate? Sunt acestea necesare? Poate funcționa ca non-admin/non-root?
- Codul care manipulează timp, folosește UTC? Dacă timpul e local, cum va funcționa codul în zilele când se face schimbarea de DST? În martie ceasul local sare cu o oră în față, cum reacționează codul? În noiembrie aceeași oră apare de două ori, cum va reacționa codul?  
- Cum reacționează codul la erori? Orice API care accesează resurse fizice (File IO, Network, DB access etc.) poate să dea eroare. Cum reacționează programul?
- În caz de eroare, este clar logată? Se poate investiga fără debugging că un access fizic a dat eroare, este capturat codul și mesajul de eroare? Dar stack-trace-ul? Important mai ales pentru procese în background.
- Noul cod necesită măsurare în producție? Cum va fi monitorizat și cum știm că are probleme?  În Windows, se pot emite performance counteri.
- Dacă noul cod necesită monitorizare, cine o va face? Există un panel? Alerts? Auto-mitigation bots? Etc.
- Dacă codul necesită modificări de schemă la baza de date, acestea cum sunt deployed? Modificări manuale nu sunt acceptabile, trebuiesc folosite migrații.

## Niște elemente de still
- Noul codul respectă stilul existent al fișierului modificat, iar fișierele noi respectă stilul proiectului? Chiar dacă stilul existent e "greșit", e mai important să păstrezi consistența. Discuțiile legate de convențiile de stil nu se poartă în code review.
- Noul cod să nu facă modificări de white-space (ex: replace tabs with spaces și vice-versa). Acestea adaugă zgomot la history (git blame) și complică merge-urile.
- În general se recomandă evitarea mixării de commit-uri funcționale (new feature) vs. commit-uri stilistice (fixing bracketing în cod-ul existent). Mixarea creează probleme dacă partea funcțională are probleme și necesită undo sau trebuie portată cherry-pick într-un alt branch. Ideal, izolați în commit-uri self-contained și independente. Dacă fișierul are probleme de stil, fixează-le într-un commit separat.
`
