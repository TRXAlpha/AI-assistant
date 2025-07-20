# Descriere proiect - Asistent AI personal

## Introducere

Acest proiect reprezintă dezvoltarea unui **asistent AI personal complet local**, orientat spre securitate cibernetică avansată, analiză comportamentală, control al rețelei și autonomie absolută. Proiectul are ca scop crearea unui sistem care nu doar reacționează la comenzi, ci anticipează, protejează și învață în timp, fără a depinde de infrastructuri cloud.

## Arhitectură Software și Inteligență Artificială

Asistentul AI rulează pe un model de limbaj LLM performant (ex. MythoMax L2 13B Q4\_K\_M), optimizat pentru funcționare locală cu consum minim de resurse (\~7 GB RAM). Este încărcat în `llama.cpp` și personalizat intens:

* Ajustarea temperaturii, repetitivității, ponderilor și prompt-urilor
* Tool-calling local prin API-uri Python
* Cod principal în **Python**, cu componente optimizate în **C#** (cross-compiled prin CMake)
* Aplicații auxiliare în **Java/Kotlin** pentru Android și extensii planificate pe Galaxy Watch
* AI-ul este capabil să își editeze promptul și comportamentul pentru eficiență mai mare în execuție
* Integrare cu baze de date externe (ex. VirusTotal) pentru detecția proactivă a semnăturilor de malware

## Securitate Proactivă și Sandbox-uri Inteligente

Fiecare dispozitiv (ex. laptop Dell, ThinkPad) include un mecanism honeypot local care:

* Interceptează execuțiile de aplicații necunoscute
* Le izolează automat într-un mediu sandbox dacă nu sunt semnate corect sau verificate
* Refuză executarea fișierelor cu comportamente neobișnuite sau proveniență incertă

Pe partea mobilă, un modul fizic acționează ca proxy local:

* Analizează traficul generat de aplicații, chiar și în rețele mobile
* Interceptează și izolează automat orice trafic de aplicație necunoscută
* Oferă protecție chiar și în rețele publice sau nesecurizate

## Control Rețea și Firewall Dinamic

În cazul activării unei linii proprii de internet, toată infrastructura va fi securizată prin:

* Containere Docker specializate pentru rutare și analiză de trafic
* Analiză DNS + filtrare comportamentală în timp real
* Reverse proxy cu NGINX și certificare HTTPS automatizată
* Logging complet al traficului
* Alerte avansate la:

  * Exploits de tip injection
  * Activitate beaconing suspectă
  * Tentative de tunneling DNS sau fluxuri anormale

Toate aceste date sunt trimise către AI-ul central, care, după antrenament extensiv și analiză comportamentală, va primi control activ asupra deciziilor (ex: allow/deny, prioritizare, izolare).

## Învățare Continuă și Autonomie Evolutivă

După \~1 an de funcționare și antrenament pe baza activității utilizatorului:

* AI-ul va învăța pattern-uri software specifice
* Va ajusta și genera cod propriu pentru automatizare și optimizare
* Va personaliza rutine, comportamente și interacțiuni în funcție de nevoile reale
* Va putea accepta cereri din rețea doar de la dispozitive autentificate prin scoruri de încredere
* Va accepta și adapta protocoalele de comunicare cu servicii externe (Microsoft, Discord, Steam etc.) doar dacă istoricul de comportament le consideră sigure

## Integrare Hardware și Arhitectură Modulară

Sistemul va include o rețea de module hardware, interconectate prin protocoale cu consum redus de energie (ESP-NOW, BLE etc.).

Se planifică migrarea de la microcontrolere ESP către procesoare **RISC-V**, un standard open-source ce permite definirea exactă a arhitecturii CPU. Această schimbare va permite:

* Personalizarea completă a logicii de procesare
* Optimizarea eficienței energetice și de calcul pentru fiecare modul
* Implementarea de filtre fizice avansate pentru securitate și autonomie
* Adaptarea schemelor electronice la nevoile personale, construind un microcontroller “ideal”

Modulele vor avea roluri multiple: interceptarea traficului, izolare software, analiză de senzori, notificări sau control fizic al altor dispozitive.

## Acces și Control

* Interfață Web locală criptată (HTTPS)
* CLI rapid pentru comenzi directe
* Integrare Telegram pentru notificări și control remote

## Concluzie

Acest sistem nu este doar un asistent AI — este o infrastructură digitală cu auto-protecție, auto-învățare și control activ al întregii experiențe digitale. Cu acces complet la rețea, sisteme și logică internă, devine în timp un administrator autonom, optimizat și ultra-securizat pentru orice scenariu de utilizare.
