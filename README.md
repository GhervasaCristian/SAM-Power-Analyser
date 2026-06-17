# Analizor de Putere și Consum de Energie Electrică (Instrument Virtual LabVIEW)

Acest depozit conține proiectul **„Instrument virtual pentru analiza consumului de energie electrică (activ, reactiv, aparent) utilizând cartela de achiziții DAQmx USB 6009”**.

---

## 🔗 Resurse Proiect
* **Prezentare Video (YouTube):** [Vizionare video prezentare](https://youtu.be/vfUwrXhT9B8)
* **Repository GitHub:** [SAM-Power-Analyser](https://github.com/GhervasaCristian/SAM-Power-Analyser)

---

## 📋 Prezentare Generală a Proiectului

Proiectul constă în realizarea unui **analizor de putere monofazat în timp real (Power Analyser)**. Acesta achiziționează semnale analogice de tensiune și curent din rețeaua electrică prin intermediul unei cartele de achiziție de date și calculează parametrii fundamentali ai energiei electrice:
* **Tensiunea și Curentul RMS ($V_{rms}$, $I_{rms}$)**
* **Puterea Activă ($P$)** – puterea utilă transformată în lucru mecanic, căldură sau lumină (exprimată în Wați - W).
* **Puterea Aparentă ($S$)** – puterea totală absorbită de consumator (exprimată în Volți-Amperi - VA).
* **Puterea Reactivă ($Q$)** – puterea asociată schimbului de energie dintre sursă și elementele reactive (bobine/condensatoare), fără a produce lucru util (exprimată în Volți-Amperi Reactivi - VAR).
* **Factorul de Putere ($PF$)** – eficiența utilizării energiei ($PF = P/S$).
* **Analiza Armonică (THD, Frecvență)** – evidențierea spectrului FFT și a perturbațiilor de rețea.

---

## 🛠️ Arhitectura Hardware

Sistemul hardware este proiectat pentru a măsura semnalele AC în condiții de siguranță, oferind izolare galvanică față de rețeaua electrică:

1. **Senzorul de Curent (Talema ASM-010):**
   * Este un transformator de curent (CT) pentru montaj pe PCB, optimizat pentru intervalul 1-10 A (50/60 Hz).
   * **Optimizare Rezoluție (Metoda celor 3 spire):** Deoarece curenții măsurați au o intensitate redusă, tensiunea nativă la ieșirea senzorului este foarte mică (3mV – 40mV). Pentru a îmbunătăți rezoluția de eșantionare a plăcii de achiziție (DAQ), conductorul de fază a fost înfășurat de **3 ori** în jurul miezului magnetic al senzorului. Acest artificiu scalează curentul primar cu un factor multiplicativ de 3 (tensiunea la ieșirea senzorului devenind 3mV – 120mV). În codul LabVIEW se aplică apoi un factor de divizare cu 3 pentru normalizarea valorilor.
2. **Senzorul de Tensiune (Carry-Tech CT-250V):**
   * Transformator de tensiune coborâtor (raport 1:35) utilizat pentru a reduce tensiunea rețelei (230V AC) la un nivel de semnal sigur pentru placa DAQ. Oferă izolare galvanică.
3. **Placa de Achiziție de Date (NI USB-6009 DAQmx):**
   * Dispune de 8 intrări analogice (AI) cu rezoluție de 14 biți și o rată de eșantionare de până la 48 kS/s.
   * **Configurație Diferențială:** Semnalele de la senzori sunt achiziționate în mod diferențial (canalele AI0/AI1). Această abordare elimină zgomotul de mod comun și protejează placa, permițând achiziția corectă a valorilor alternative negative fără a deteriora ADC-ul.

---

## 💻 Aplicația Software (LabVIEW 2013 SP1)

Implementarea logică din spatele instrumentului virtual este realizată în LabVIEW și utilizează biblioteca **NI-DAQmx**:

### 1. Modelul Matematic implementat
Fiecare parametru este calculat în timp real pe baza a $N$ eșantioane:
* **Tensiunea RMS:** $V_{rms} = \sqrt{\frac{1}{N} \sum_{k=1}^{N} v[k]^2}$
* **Curentul RMS:** $I_{rms} = \sqrt{\frac{1}{N} \sum_{k=1}^{N} i[k]^2}$
* **Puterea Activă:** $P = \frac{1}{N} \sum_{k=1}^{N} (v[k] \cdot i[k])$
* **Puterea Aparentă:** $S = V_{rms} \cdot I_{rms}$
* **Puterea Reactivă:** $Q = \sqrt{S^2 - P^2}$
* **Factorul de Putere:** $PF = \frac{P}{S}$

### 2. Caracteristici Cheie ale Interfeței Utilizator (GUI)
* **Grid Scope (Osciloscop Virtual Multi-Grafic):** Suprapune formele de undă de tensiune și curent pe același grafic folosind scale independente, facilitând vizualizarea defazajului.
* **Analiza Spectrală FFT:** Utilizează funcții de fereastră (Hanning / Flat Top) prin blocul *FFT Power Spectrum and PSD* pentru a vizualiza armonicile curentului/tensiunii și a monitoriza zgomotul sau distorsiunile armonice totale (THD).
* **Detector Caracter Rețea:** Comparator logic determină automat dacă sarcina este inductivă sau capacitivă. Indicatorul din interfață devine verde (activ) când rețeaua prezintă caracter capacitiv ($Q > 0$) și se stinge la caracter inductiv ($Q < 0$).
* **Fereastra de Diagnostic DAQ LOGs:** O componentă de gestionare a erorilor bazată pe deconstruirea și reconstruirea clusterului standard de erori din LabVIEW cu blocuri *Unbundle/Bundle*, afișând un istoric al posibilelor probleme hardware.
* **Logare Date în Fișier:** Salvarea automată a datelor achiziționate. Pentru optimizare, scrierea capului de tabel (header) este executată o singură dată în afara buclei principale de calcul, iar scrierea parametrilor se face în mod continuu în interiorul buclei utilizând blocul *Format Into String*.

---

## 📂 Structura Repozitoriului

* `Documentation/` – Documentațiile detaliate pe etape (P1 - Concepție Hardware, P2 - Documentare Componente, P3 - Testare Senzori, P4 - Dezvoltare LabVIEW, P5/P6 - Proiect Final compilate în format PDF).
* `VI/` – Fișierele sursă LabVIEW: instrumentul virtual principal `PowerAnalyser.vi` și controalele personalizate (`.ctl`).
* `Photos/` – Capturi de ecran ale interfeței grafice (Front Panel), scheme electrice și imagini ale montajului hardware.
* `Simulation Models/` – Modele de simulare utilizate pentru validarea teoretică.
