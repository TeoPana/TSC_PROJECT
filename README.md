# TSC_PROJECT
                    DIAGRAMA

                 +----------------------+
                 |      Baterie LiPo    |
                 +----------+-----------+
                            |
                            v
                 +----------------------+
                 |   Bloc alimentare    |
                 |  Charger / DC-DC     |
                 |  VBUS / VBAT / 3V3   |
                 +----------+-----------+
                            |
                            v
                 +----------------------+
                 |      nRF52840        |
                 | MCU + BLE + control  |
                 +---+---------+-----+--+
                     |         |     |
          SPI        |         |     | GPIO
                     |         |     |
                     v         v     v
          +---------------+  +-------------+  +-------------+
          |  Display      |  |  Senzori    |  |  Butoane    |
          |  e-paper      |  |  IMU / I2C  |  |  User Input |
          +---------------+  +-------------+  +-------------+

                     |
                     | SWD / test pads
                     v
              +------------------+
              | Debug / Program  |
              | SWDIO / SWDCLK   |
              +------------------+

                     |
                     | RF matching
                     v
              +------------------+
              | Antena 2.4 GHz   |
              +------------------+

USB-C ---> Alimentare / Service / Debug



Descriere Hardware - Proiect Smartwatch
Ce am vrut să fac aici e un smartwatch care are la bază microcontrolerul nRF52840. L-am ales pe ăsta pentru că e destul de puternic, are nucleu ARM Cortex-M4F și vine deja cu partea de radio de 2.4 GHz integrată, deci ne scapă de multe bătăi de cap cu comunicarea. El se ocupă de tot: display, butoane, baterie și partea de bluetooth.

1. Cum e gândit sistemul
Toată treaba e alimentată de o baterie mică Li-Po. Am pus niște test pad-uri pe placă ca să o putem lipi ușor. Tensiunea trece printr-un etaj de conversie (un DC/DC) și apoi pleacă spre restul componentelor.

În mare, piesa centrală (nRF-ul) coordonează:

Ecranul de tip e-paper (care consumă super puțin);

Senzorii de pe magistrala serială;

Butoanele pentru utilizator;

Partea de programare (debug) și antena de radio.

Cam astea ar fi blocurile mari: alimentare, procesorul, afișajul, butoanele și partea de RF.

2. Alimentarea
Bateria Li-Po e legată direct la PCB. Ca să scoatem 3.3 V pentru microcontroler am folosit un convertor DC/DC step-up (mă rog, e de tip step-down pentru că bateria are cam 3.7V-4.2V, dar ideea e că stabilizează rail-ul principal).

Am pus și condensatori de filtrare, de 100 nF, pe care i-am înghesuit cât mai aproape de pinii de alimentare, ca să nu avem zgomot. Traseele de curent le-am făcut mai groase pe cablaj ca să nu se încălzească sau să avem căderi de tensiune. Am mers pe DC/DC în loc de regulator liniar (LDO) ca să țină bateria mai mult, că la un ceas e critică autonomia.

3. Creierul sistemului (nRF52840)
Ăsta e punctul central. nRF-ul face de toate: citește butoanele, trimite date la ecran și se ocupă de stările de „sleep” ca să economisească energie. L-am pus cam pe mijlocul plăcii ca să fie traseele scurte către toate componentele și să nu se bată cap în cap semnalele digitale cu cele de radio.

4. Display-ul și cum se vede imaginea
Ecranul e legat prin SPI. Am ales SPI pentru că e mult mai rapid ca I2C-ul și la un ecran grafic chiar ai nevoie de viteză ca să nu se vadă cum se desenează pixelii.
Semnalele pe care le folosim sunt: MOSI pentru date, SCK (ceasul), CS (ca să știe display-ul că vorbim cu el) și mai sunt pinii de Reset și Busy.

Conectorul pentru display e pus pe margine, ca să poată intra panglica aia flexibilă în carcasă fără să se îndoaie prea tare sau să atingă alte piese.

5. Senzori și alte conexiuni
Pentru restul senzorilor am folosit magistrala I2C. E mai simplu așa că folosim doar doi pini (SDA și SCL) și putem pune mai mulți senzori pe aceleași fire. Am pus și rezistențe de pull-up, că altfel nu merge comunicarea pe I2C, e o greșeală clasică pe care am vrut să o evit.

6. Butoane (Input)
Am pus niște butoane fizice legate la pinii de tip GPIO. Din soft o să facem debouncing, dar pe parte de hardware sunt legate direct. Sunt puse pe margini ca să se pupe cu decupajele din carcasa printată.

7. USB-ul și programarea
Avem un conector USB-C pentru încărcare. Pentru programare n-am mai pus mufă specială că ocupa loc, așa că am scos niște pad-uri de test (SWDIO, SWDCLK, etc) unde ne putem lipi cu un programator extern când facem debug. E mult mai ok așa pentru faza de prototip.

8. Partea de antenă (RF)
Aici e cea mai sensibilă zonă. Am urmat schema de referință cu niște bobine și condensatori (rețeaua de adaptare) ca să avem impedanța de 50 ohmi.
Câteva reguli pe care le-am respectat la layout:

Antena e fix pe marginea plăcii;

Nu am pus masă (ground) sub antenă că altfel nu mai emite nimic;

Traseul e cât mai scurt, fără unghiuri dubioase care să bage paraziți.

9. Designul cablajului (Layout)
Am încercat să grupez piesele logic. Condensatorii de decuplare sunt „lipiți” de pinii de alimentare. N-am folosit unghiuri de 90 de grade la trasee (am mers pe 45) ca să arate profi și să evităm problemele de reflexie a semnalului. Am pus plan de masă pe ambele fețe și multe via-uri (via stitching) ca să fie totul ecranat bine, mai ales lângă zona de radio.

10. Bateria și consumul
Ceasul trebuie să stea aprins mult, deci am setat nRF-ul să stea în low-power cât mai mult timp. Ecranul e-paper ajută enorm aici, că el consumă curent doar când schimbi imaginea, în rest stă „degeaba”. Dacă nu facem refresh des, bateria ar trebui să țină destul de bine.

11. Cum se asamblează toate (Mecanică)
În modelul 3D am verificat dacă încap toate: PCB-ul, bateria Li-Po (care e destul de grasă) și ecranul. Mufa USB-C trebuie să iasă fix prin gaura din carcasă, altfel nu putem băga cablul la încărcat. Per total, totul pare că se aliniază ok și nu există piese care să se lovească între ele.



## Bill of Materials (BOM)

| Ref | Componentă | Valoare | Package | JLC Part | Datasheet |
|-----|------------|--------|--------|----------|-----------|
| C5, C7, C8, C12, C19 | Condensator decuplare | 100nF | 0201 | C1525 | https://datasheet.lcsc.com/lcsc/1811091111_Murata-Electronics-GRM011R60J104KE01D_C1525.pdf |
| C23, C27, C34, C42 | Condensator | 100nF | 0201 | C1525 | https://datasheet.lcsc.com/lcsc/1811091111_Murata-Electronics-GRM011R60J104KE01D_C1525.pdf |
| C15 | Condensator | 1µF | 0402 | C52923 | https://datasheet.lcsc.com/lcsc/1811151714_Samsung-Electro-Mechanics-CL05A105KO5NNNC_C52923.pdf |
| C24, C39 | Condensator bulk | 10µF | 0402 | C19702 | https://datasheet.lcsc.com/lcsc/1811091111_Samsung-Electro-Mechanics-CL05A106MQ5NUNC_C19702.pdf |
| C11 | Condensator RF | 100pF | 0201 | C57112 | https://datasheet.lcsc.com/lcsc/1811151714_Samsung-Electro-Mechanics-CL03C101JB3NNNC_C57112.pdf |
| R5, R7, R8 | Rezistor | 10kΩ | 0201 | C25744 | https://datasheet.lcsc.com/lcsc/1811090913_Yageo-RC0201FR-0710KL_C25744.pdf |
| R9, R_PWR_EPD | Rezistor | 10kΩ | 0201 | C25744 | https://datasheet.lcsc.com/lcsc/1811090913_Yageo-RC0201FR-0710KL_C25744.pdf |
| L2 | Inductor | 10µH | 0402 | C1046 | https://datasheet.lcsc.com/lcsc/1811091211_Sunlord-SWPA4020S100MT_C1046.pdf |
| SJ1 | Jumper | - | SMD | - | - |
| U1 | Microcontroller | nRF52840 | QFN/BGA | C87453 | https://infocenter.nordicsemi.com/pdf/nRF52840_PS_v1.1.pdf |
| USB | Conector USB-C | - | SMD | C165948 | https://datasheet.lcsc.com/lcsc/1811151713_TYPE-C-16PIN_C165948.pdf |
Toate componentele au fost selectate din biblioteca JLCPCB pentru a asigura compatibilitatea cu procesul de asamblare automată.





PINII nRF52840

Pinii i-am ales în funcție de cum dădea mai bine pe layout-ul PCB-ului și cum se optimiza mai ușor rutarea. Perifericele, gen SPI și I2C, le-am mapat pe pinii care erau cei mai aproape de componentele lor, special ca să ținem traseele scurte și să nu avem zgomot pe semnal.

Pinii cu funcții speciale (SWD și resetul) i-am lăsat doar pentru partea de debug, nu i-am folosit la alte semnale de uz general ca să nu ne încurcăm. Pentru semnalele de viteză mare, cum e SPI-ul, am mers pe conexiuni cât mai scurte și directe, în timp ce pe alea mai lente (I2C sau GPIO-urile) le-am plasat mai flexibil, pe unde am mai găsit loc pe placă.

Aveam si componenta 3D a TC2030-IDC ului, dar dupa ce am vorbit cu laborantul despre problema ca nu ar incapea in carcasa toate acele cabluri, mi s-a spus ca trebuie sa sterg acel model, si doar sa pun un rectangle deasupra lui ca sa acopar acele gauri. 
