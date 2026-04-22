# Proiect-TSC

Autor: Giuglan Denisa-Cristina
Grupa: 334CD

1. Diagrama Bloc (Hardware Architecture)
Am proiectat arhitectura sistemului InkTime avand in centru SoC-ul nRF52840. Am optimizat conexiunile pentru un consum minim de curent si am ales protocoale de comunicatie eficiente (SPI pentru display si I2C pentru senzori).

SURSA ENERGIE              SISTEM INTELIGENT INKTIME             PERIFERICE (I/O)
    +-----------------+          +-----------------------+          +-----------------------+
    | USB-C Connector |--- 5V ---|    nRF52840 (SoC)     |          |    E-PAPER DISPLAY    |
    |  (Incarcare)    |          | Bluetooth 5.0 Low En. |--- SPI --|     (1.54" 24-pin)    |
    +--------+--------+          +-----------+-----------+          +-----------------------+
             |                               |
             v                   +-----------+-----------+          +-----------------------+
    +-----------------+          | P0.02 (SCK)           |          |    ACCEL. (BMA421)    |
    | BQ25180 Charger |          | P0.03 (MOSI)          |          |    GAUGE (MAX17048)   |
    | (Mngmnt Baterie)|-- VBAT --| P0.26 (SDA) <---I2C-->|----------|    DRV2605 (Haptic)   |
    +--------+--------+          | P0.27 (SCL)           |          +-----------------------+
             ^                   +-----------+-----------+
             |                               |                      +-----------------------+
    +--------+--------+          | P1.11 (PWM) ----------|--------->|    SHAKER / MOTOR     |
    | Li-Po Battery   |          |                       |          +-----------------------+
    |    250 mAh      |          | P0.13 (UP)            |
    +--------+--------+          | P0.14 (ENTER) <---GPIO|----------+      3x BUTOANE       |
             |                   | P1.02 (DOWN)          |          |      NAVIGATIE        |
             v                   +-----------+-----------+          +-----------------------+
    +-----------------+                      |
    | RT6160A (Buck-B)|<---------------------+                      +-----------------------+
    |  Output: 3.3V   |-------------------------------------------->|   CRYSTAL 32.768kHz   |
    +-----------------+                                             +-----------------------+

2. Bill of Materials (BOM)
Am selectat toate componentele de pe platforma JLC Parts, respectand capsulele SMD impuse (0201 pentru rezistente si condensatori de 100nF, 0402 pentru restul).

3. Functionalitate Hardware si Interfete
In dezvoltarea acestui proiect, am pus accent pe miniaturizare si eficienta energetica:

MCU nRF52840: Gestioneaza stack-ul Bluetooth Low Energy si procesarea datelor de la senzori.
Display E-Paper: Comunicatia se realizeaza prin SPI Bus. Am ales acest afisaj datorita tehnologiei bistabile care pastreaza imaginea fara consum de energie.
Senzorul IMU & Gauge: Sunt conectate pe aceeasi magistrala I2C (P0.26/P0.27) pentru a economisi pinii GPIO ai microcontrolerului.
Haptic Feedback: Shaker-ul este controlat prin driver-ul haptic DRV2605, utilizand semnal PWM de la MCU (P1.11) pentru a genera modele de vibratie complexe.
Sistem de Putere: Am implementat un regulator Buck-Boost RT6160A care asigura o tensiune constanta de 3.3V, chiar si atunci cand bateria Li-Po scade spre pragul de descarcare.

4. Alocare Pini (Pinout Map)
Maparea pinilor a fost realizata pentru a asigura o rutare optima pe stratul TOP si pentru a respecta cerintele mecanice:

P0.02, P0.03, P0.15: SPI (SCK, MOSI, DC) pentru controlul Display-ului.
P0.26, P0.27: Magistrala I2C (SDA, SCL) pentru IMU si Fuel Gauge.
P1.11: PWM pentru controlul vibratiilor (Shaker).
P0.13, P0.14, P1.02: Input-uri GPIO pentru butoanele de navigatie (Up, Enter, Down).
P0.00, P0.01: Conexiuni pentru cristalul extern de 32.768kHz (LFXO Clock).

Alocarea pinilor nRF52840 a fost făcută strategic pentru a optimiza rutarea PCB și pentru a utiliza funcțiile hardware native ale SoC-ului:

Interfața SPI (P0.02 - SCK, P0.03 - MOSI, P0.15 - DC):
De ce: Acești pini au fost grupați în partea stângă a procesorului pentru a avea o conexiune directă și scurtă către conectorul FPC al ecranului E-Paper. Folosirea pinilor P0.02 și P0.03 (care pot fi mapați ca High-Drive) asigură un semnal stabil pentru frecvența de ceas a ecranului.
Magistrala I2C (P0.26 - SDA, P0.27 - SCL):
De ce: Am ales acești pini deoarece sunt plasați pe marginea chip-ului, facilitând rutarea către senzorul BMA421 și Fuel Gauge (MAX17048) fără a traversa alte trasee de date. Acești pini suportă protocolul TWI (Two-Wire Interface) nativ, reducând consumul de energie în modul Sleep.
Controlul Haptic (P1.11 - PWM):
De ce: Pinul P1.11 face parte din portul 1, fiind izolat de semnalele SPI de viteză mare, ceea ce reduce riscul de interferențe electromagnetice între driver-ul haptic și restul sistemului. Este conectat la perifericul PWM hardware pentru a genera vibrații fluide fără a încărca procesorul.
Butoanele de Navigație (P0.13, P0.14, P1.02):
De ce: Acești pini au fost aleși pentru că suportă funcția de "Wake-up on GPIO". Acest lucru permite smartwatch-ului să intre în modul Deep Sleep (consum de micro-amperi) și să fie activat instantaneu la apăsarea oricărui buton.
Oscilatorul Extern (P0.00, P0.01):
De ce: Aceștia sunt pinii dedicați pentru cristalul de 32.768kHz (XL1/XL2). Utilizarea unui cristal extern este critică pentru InkTime pentru a menține precizia ceasului în timp ce sistemul se află în modurile Low Power.

5. Design Log si Constrangeri
Am implementat design-ul respectand urmatoarele reguli de bune practici:

Layout PCB: Am folosit un stackup de 1mm grosime. Traseele de putere (VUSB, VBAT, 3V3) au o latime de 0.3mm, iar semnalele de date 0.15mm.
Antena Radio: Am asigurat o zona de keep-out (decupaj in planul de masa) sub antena PCB a nRF52840 pentru a evita atenuarea semnalului BLE.
Management Termic/Zgomot: Am plasat condensatoarele de decuplare (100nF) cat mai aproape de pinii de alimentare ai procesorului si am adaugat via-stitching pentru planul de masa (GND).
Integrare Mecanica: Am asezat componentele in "sandwich" (Display -> PCB -> Baterie -> Shaker). Am acceptat erorile de tip "Dimension" la butoane si USB-C deoarece acestea trebuie sa strapunga peretele carcasei pentru a fi functionale.
