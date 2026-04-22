# Proiect TSC - InkTime Smartwatch
**Student:** Giuglan Denisa-Cristina  
**Grupa:** 334CD  
---

## 1. Viziunea Proiectului si Arhitectura Sistemului

### Diagrama Bloc Hardware
Am structurat conexiunile pentru a separa magistrala rapida SPI (display) de magistrala senzorilor I2C, asigurand astfel o integritate maxima a semnalului.

```text
       ALIMENTARE (POWER)             SISTEM CENTRAL (MCU)                INTERFETE I/O
    _____________________          ___________________________          ____________________
   |                     |        |                           |        |                    |
   |   USB-C Connector   |---5V---|      nRF52840 (SoC)       |--SPI-->|   DISPLAY E-PAPER  |
   |_____________________|        |    Cortex-M4F @ 64MHz     |   Bus  |   (1.54" 24-pin)   |
              |                   |___________________________|        |____________________|
              v                    | P0.02 (SCK) | P0.03 (MOSI)
    _____________________          | P0.15 (DC)  | P0.16 (RST)          ____________________
   |                     |        |___________________________|        |                    |
   |   BQ25180 Charger   |--VBAT--|      MAGISTRALA I2C       |<------>|   ACCEL. BMA421    |
   |  (Li-Po Management) |        |      (SDA / SCL)          |   Bus  |   MAX17048 GAUGE   |
   |_____________________|        |___________________________|        |   DRV2605 HAPTIC   |
              ^                    | P0.26 (SDA) | P0.27 (SCL)         |____________________|
              |                    |___________________________|
    __________|__________          |                           |        ____________________
   |                     |         |   GPIO / PWM CONTROL      |       |                    |
   |   BATERIE LI-PO     |         |                           |------>|   MOTOR VIBRATII   |
   |      250 mAh        |         | P1.11 (PWM) | P0.13 (UP)  |       |____________________|
   |_____________________|         | P0.14 (ENT) | P1.02 (DN)  |<------|    3x BUTOANE      |
              |                    |___________________________|       |____________________|
              v                              |
    _____________________          __________|__________                ____________________
   |                     |        |                     |              |                    |
   |   RT6160A BUCK-B    |--3.3V--|   CRISTAL EXTERN    |------------->|      ANTENA RF     |
   |  (System Power)     |        |    (32.768 kHz)     |              |     (2.4 GHz)      |
   |_____________________|        |_____________________|              |____________________|
```
## 2. Documentatia Componentelor (BOM)

Am selectat componentele bazandu-ma pe catalogul **JLC Parts**, urmarind un raport optim intre dimensiune si performanta. Toate piesele pasive (rezistente/condensatoare) sunt in capsule **0201** si **0402**.

| Componenta | Package | Cod JLC Parts | Datasheet |
|:---|:---|:---|:---|
| **nRF52840-QIAA-R** (MCU) | aQFN73 | [C190794](https://jlcpcb.com/partdetail/Nordic_Semicon-NRF52840_QIAA_R/C190794) | [Link](https://docs.nordicsemi.com/) |
| **E-Paper Display 1.54"** | 24-pin FPC | [C359944](https://jlcpcb.com/partdetail/Waveshare-4_2inch_e_PaperModule/C359944) | [Link](https://www.waveshare.com/w/upload/e/e6/1.54inch_e-Paper_Datasheet.pdf) |
| **BQ25180YBGR** (Charger) | DSBGA-9 | [C3682423](https://jlcpcb.com/partdetail/TexasInstruments-BQ25180YBGR/C3682423) | [Link](https://www.ti.com/lit/ds/symlink/bq25180.pdf) |
| **RT6160AWSC** (Buck-Boost) | WLCSP-15 | [C7065276](https://jlcpcb.com/partdetail/Richtek_Tech-RT6160AWSC/C7065276) | [Link](https://www.richtek.com/assets/product_file/RT6160A/DS6160A-05.pdf) |
| **BMA421** (Accel) | LGA-12 | [C5242966](https://jlcpcb.com/partdetail/Bosch_Sensortec-BMA421/C5242966) | [Link](https://www.bosch-sensortec.com/media/boschsensortec/downloads/datasheets/bst-bma421-ds004.pdf) |
| **MAX17048G+T10** (Gauge) | DFN-8 | [C2682616](https://jlcpcb.com/partdetail/AnalogDevices-MAX17048G_T10/C2682616) | [Link](https://www.analog.com/media/en/technical-documentation/data-sheets/MAX17048-MAX17049.pdf) |
| **DRV2605YZFR** (Haptic) | VSSOP-10 | [C81079](https://jlcpcb.com/partdetail/TexasInstruments-DRV2605YZFR/C81079) | [Link](https://www.ti.com/lit/ds/symlink/drv2605.pdf) |
| **Antenna (2450AT18B100E)** | SMD 3216 | [C5179427](https://jlcpcb.com/partdetail/Johanson_Technology-2450AT18B100E/C5179427) | [Link](https://www.johansontechnology.com/datasheets/2450AT18B100/2450AT18B100.pdf) |
| **KH-TYPE-C-16P** (USB) | 16-pin SMD | [C2765186](https://jlcpcb.com/partdetail/Kinghelm-KH_TYPE_C_16P/C2765186) | [Link](https://datasheet.lcsc.com/lcsc/2012121836_Kinghelm-KH-TYPE-C-16P_C2765186.pdf) |
| **Molex 503480-2400** (FPC) | 24-pin SMD | [C2857168](https://jlcpcb.com/partdetail/Molex-5034802400/C2857160) | [Link](https://www.molex.com/en-us/products/part-detail/5034802400) |
| **USBLC6-2SC6Y** (ESD) | SOT-23-6 | [C7519](https://jlcpcb.com/partdetail/Stmicroelectronics-USBLC6_2SC6Y/C7519) | [Link](https://www.st.com/resource/en/datasheet/usblc6-2.pdf) |
| **Si2301CDS** (P-FET) | SOT-23 | [C10487](https://jlcpcb.com/partdetail/Vishay-Si2301CDS-T1-GE3/C10487) | [Link](https://www.vishay.com/docs/68749/si2301cds.pdf) |
| **EVP-AKB31A** (Butoane) | SMD Tactile | [C2845028](https://jlcpcb.com/partdetail/Panasonic-EVP_AKE31A/C2845028) | [Link](https://industrial.panasonic.com/cdbs/www-data/pdf/ATV0000/ATV0000CE3.pdf) |
| **Motor Vibratii** | Wire leads | [C7528006](https://jlcpcb.com/partdetail/7528806-LCM1027B3605F/C7528806) | [Link](https://datasheet.lcsc.com/lcsc/2310131633_LCM-LCM1027B3605F_C7528806.pdf) |
| **Crystal 32 MHz** | 2016 | [C5265841](https://jlcpcb.com/partdetail/Huiyuancrystal-HY32MSMD2016CA1R30/C5265841) | [Link](https://www.lcsc.com/product-detail/C7275071.html) |
| **Crystal 32.768 kHz** | 3215 | [C70509](https://jlcpcb.com/partdetail/NDK-NX3215SA_32_768KHZ_EXS00AMU00466/C70509) | [Link](https://www.ndk.com/images/products/catalog/c_NX3215SA_e.pdf) |
| **Inductor 0.47uH** | 2016 | [C5832368](https://jlcpcb.com/partdetail/6763488-FTC252012SR47MBCA/C5832368) | [Link](https://www.lcsc.com/product-detail/C5832368.html) |
| **Inductor 10uH** | 0402 | [C910650](https://jlcpcb.com/partdetail/MurataElectronics-LQW15DN100M00D/C910650) | [Link](https://www.murata.com/en-us/products/productdetail?partno=LQW15DN100M00%23) |
| **Rezistenta 10k Ω** | 0201 | [C100126](https://jlcpcb.com/partdetail/LIZElec-CR0201FH1002G/C100126) | [Link](https://www.lcsc.com/product-detail/C100126.html) |
| **Rezistenta 5.1k Ω** | 0201 | [C100142](https://jlcpcb.com/partdetail/LIZElec-CR0201FH5101G/C100142) | [Link](https://www.lcsc.com/product-detail/C5266011.html) |
| **Condensator 1uF** | 0402 | [C14445](https://jlcpcb.com/partdetail/15107-CL05A105KP5NNNC/C14445) | [Link](https://weblib.samsungsem.com/) |
| **Condensator 100nF** | 0201 | [C49062](https://jlcpcb.com/partdetail/50070-CL03A104KP3NNNC/C49062) | [Link](https://www.lcsc.com/product-detail/C49062.html) |
| **Condensator 4.7uF** | 0402 | [C23687](https://jlcpcb.com/partdetail/TDK-C1005X5R0J475MTJ00E/C23687) | [Link](https://www.lcsc.com/product-detail/C23687.html) |
| **Condensator 22uF** | 0402 | [C105226](https://jlcpcb.com/partdetail/106441-CL05A226MQ5QUNC/C105226) | [Link](https://www.lcsc.com/product-detail/C105226.html) |
| **Condensator 12pF** | 0201 | [C50391](https://jlcpcb.com/partdetail/51400-0201CG120J500NT/C50391) | [Link](https://www.lcsc.com/product-detail/C50391.html) |
| **LiPo 250mAh** | Baterie | - | [Link](https://www.tme.eu/Document/b9e12bf26ad0ba929a22ab5d58f022cd/AKY0106.pdf) |
| **TC2030-IDC** (Debug) | Footprint | - | [Link](https://www.tag-connect.com/product/tc2030-idc) |

---

## 3. Analiza Functionalitatii Hardware

In aceasta faza a proiectului, m-am concentrat pe implementarea unui sistem care sa poata functiona independent de o sursa de curent externa pentru perioade lungi.

### 3.1 Unitatea Centrala si Conectivitatea
Am optat pentru **nRF52840** datorita arhitecturii sale flexibile. Acesta imi permite sa folosesc USB-ul nativ pentru debug si transfer de date, in timp ce interfata radio BLE asigura sincronizarea cu telefonul. Pentru a asigura o precizie ridicata a timpului, am adaugat un cristal extern de **32.768 kHz**, esential pentru ca ceasul sa nu piarda secunde in modurile sleep.

### 3.2 Sistemul de Afisaj E-Paper
In loc de un ecran clasic care necesita iluminare constanta, am integrat un display E-Paper de 1.54 inch. Acesta utilizeaza particule incarcate electric care isi pastreaza pozitia fara consum de energie. Am conectat ecranul prin **SPI**, trimitand datele la frecventa maxima suportata pentru a finaliza refresh-ul rapid si a permite procesorului sa intre inapoi in starea de consum minim.

### 3.3 Managementul Energiei
Alimentarea este asigurata de o baterie Li-Po de 250mAh. Am integrat un circuit **Buck-Boost RT6160A** pentru a ma asigura ca, indiferent de nivelul de descarcare al bateriei (intre 4.2V si 3.0V), sistemul primeste un 3.3V stabil. Incarcarea este gestionata inteligent de **BQ25180**, iar monitorizarea procentului bateriei se face prin **MAX17048**, care raporteaza datele catre MCU prin magistrala I2C.

---

## 4. Alocarea Pinilor nRF52840 (Justificare Tehnica)

In procesul de layout, am ales pinii pentru a optimiza rutarea si a separa semnalele de mare viteza de cele de control.

| Pin nRF | Conectat la | Ratiune Tehnica / Explicatie |
|:---|:---|:---|
| **P0.02** | E-Paper SCK | Pin de clock SPI de mare viteza, ales pentru capabilitatea High-Drive. |
| **P0.03** | E-Paper MOSI | Linie de date SPI catre display, plasata langa clock pentru a asigura integritatea semnalului. |
| **P0.15** | E-Paper DC | Data/Command select, esential pentru protocolul controller-ului EPD. |
| **P0.26** | I2C SDA | Linie de date pentru magistrala comuna (IMU, Fuel Gauge, Haptic). |
| **P0.27** | I2C SCL | Semnal de ceas pentru magistrala I2C. |
| **P1.11** | PWM Haptic | Controlul motorului de vibratii, plasat pe Portul 1 pentru a izola zgomotul de comutatie. |
| **P0.13** | Button UP | Intrare GPIO cu rezistenta pull-up interna; poate trezi MCU din sleep (SENSE). |
| **P0.14** | Button ENTER| Intrare digitala pentru selectie si confirmare in meniu. |
| **P1.02** | Button DOWN | Navigare interfata utilizator. |
| **SWDIO / SWDCLK**| Debug/Prog | Interfata Serial Wire Debug folosita pentru programare prin header-ul Tag-Connect. |

---

## 5. Estimarea Consumului si Autonomia

Am proiectat InkTime cu un consum in "Deep Sleep" (System ON) estimat la **~84 uA** (MCU sleep, RTC activ, senzori in low-power).
* In timpul unui **refresh de ecran**, avem un varf de consum de pana la **15 mA** pentru o durata scurta.
* In modul **standby**, tehnologia E-Ink consuma **0 mA** pentru a mentine imaginea.

**Calcul Autonomie:** Cu o baterie de 250 mAh, la un regim de utilizare mediu (refresh ceas o data pe minut si notificari BLE), autonomia estimata este de **~30 de zile**, oferind o experienta superioara smartwatch-urilor clasice.

---

## 6. Design Mecanic si Integrare 3D

Integrarea componentelor a fost realizata in **Fusion 360**, adoptand o structura compacta de tip sandwich:
* **Carcasa Superioara:** Protejeaza display-ul si asigura etanseitatea.
* **PCB:** Placa de 2 straturi pe care sunt montate toate componentele SMD.
* **Carcasa Inferioara:** Gazduieste bateria Li-Po si motorul haptic, fiind fixata prin presare si suruburi de precizie.
Am generat fisierele **.step** pentru vederea explodata si **.f3z** pentru proiectul sursa, asigurand compatibilitatea cu procesele de fabricatie.

---

## 📂 Organizare Repository
* **/Hardware**: Schema electronica (.sch) si design PCB (.brd).
* **/Manufacturing**: Arhiva Gerber, fisiere BOM si Pick&Place.
* **/Mechanical**: Modele CAD 3D (.step, .f3z).
* **/Images**: Randari si capturi de ecran.
