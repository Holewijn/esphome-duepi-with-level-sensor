# ESPHome Duepi with Level Sensor 🔥⛽

Een geavanceerde en modulaire **ESPHome**-configuratie voor slimme pelletkachels die gebruikmaken van een **Duepi EVO / Artel** controllerboard. Dit project combineert de kerncommunicatie van de kachel met een geïntegreerde **HC-SR04** ultrasone afstandssensor om het pelletniveau, live verbruik en onderhoudsstatistieken in real-time te monitoren via **Home Assistant**.

Dit project draait op een **ESP32 WROOM 32 (USB-C)** en maakt gebruik van de officiële ESPHome `packages:` architectuur om modules overzichtelijk en onderhoudsvrij samen te voegen.

---

## 🛠 Hardware Vereisten

*   **Microcontroller:** NodeMCU ESP32 WROOM 32 (met CH340 of CP2102 USB-C chip).
*   **Niveausensor:** HC-SR04 Ultrasone Afstandssensor (kies de 3.3V compatibele variant of de RCWL-9610 chipset).
*   **Pelletkachel:** Ondersteunde Artel / Duepi EVO controller gebaseerde pelletkachels.

---

## 🔌 Bedrading & Pinout

Omdat we een ESP32 gebruiken, is de UART-communicatie naar de kachel verplaatst naar de hardware seriële pinnen **GPIO16** en **GPIO17**. Dit voorkomt conflicten met de USB-chip tijdens het flashen of loggen via USB-C.

### HC-SR04 Ultrasone Niveausensor

| Sensor Pin | ESP32 Pin | Beschrijving |
| :--- | :--- | :--- |
| **VCC** | **3V3** | Stroomvoorziening (Veilig 3.3V voor deze specifieke sensor) |
| **TRIG** | **GPIO23 (G23)** | Trigger Puls signaal |
| **ECHO** | **GPIO22 (G22)** | Echo Return signaal |
| **GND** | **GND** | Massa / Ground |

### Duepi Moederbord (Pelletkachel)

| Duepi Pin | ESP32 Pin | Beschrijving |
| :--- | :--- | :--- |
| **+5V** | **5V / VIN** | Voedt de ESP32 rechtstreeks vanuit de kachel |
| **GND** | **GND** | Massa / Ground |
| **RX** | **GPIO17 (TX2)** | ESP Transmit ➔ Stove Receive |
| **TX** | **GPIO16 (RX2)** | Stove Transmit ➔ ESP Receive |

---

## 💡 Belangrijkste Functies & Modules

Dit project is opgedeeld in losse, krachtige componenten (`packages:`) die intelligent samenwerken binnen ESPHome:

### 1. Slimme Externe Temperatuursensor (met Automatische Failover) 🌡🛡
Standaard kan deze configuratie een draadloze kamer-temperatuursensor uit Home Assistant (bijv. Zigbee, BLE of Wi-Fi) gebruiken om de klimaatthermostaat van de kachel aan te sturen in plaats van de warme sensor direct achter de kachel.
*   **Normale werking:** De ESP32 leest live de status van je externe kamersensor. De kachel moduleert perfect op basis van de plek waar jij je sensor neerlegt.
*   **Emergency Failover:** Mocht de Wi-Fi wegvallen, Home Assistant crashen of de batterij van de externe sensor leegraken? De ESP32 detecteert direct de ontbrekende data (`NAN`) en schakelt **binnen 10 seconden** automatisch en geruisloos terug naar de **interne omgevingssensor** van de kachel. Je huis blijft warm en veilig!
*   **Monitoring:** De actieve bron is live te volgen via de entiteit `sensor.actieve_temperatuurbron`.

### 2. Ultrasone Niveaumeter & Looptijd Berekening ⏳
*   **Pelletniveau Procent:** Zet de gemeten afstand in meters om naar een nauwkeurig percentage (0-100%).
*   **Verbruikssnelheid (%/h):** Berekent lokaal op de ESP32 via een geavanceerde C++ lambda-tijdsmeting hoe snel het percentage per uur daalt.
*   **Resterende Tijd:** Geeft een live prognose in uren/minuten aan hoe lang de kachel nog kan branden voordat de bunker leeg is.

### 3. Module: Verbruiks-AI (Gewicht & Live Grafieken) 🧠📊
*   **Actueel Pelletverbruik (Gewicht):** Vertaalt de procentuele daling per uur naar daadwerkelijke kilo's per uur (`kg/h`) op basis van de bunkerinhoud. Detecteert automatisch wanneer de kachel uitstaat of bijgevuld wordt (verbruik springt direct naar `0.0 kg/h`).
*   **Live Verbruik (Gram/Minuut):** Rekent het verbruik om naar grammen per minuut, ideaal voor real-time updates en vloeiende live-grafieken in je Home Assistant dashboard.
*   **AI Schakelaar:** Eenvoudig in- of uit te schakelen via een handige template switch (`switch.module_verbruiks_ai`).

### 4. Pellet & Zakken Teller 🪵🔢
*   Houdt via globale variabelen bij hoeveel kilo pellets er in totaal zijn verbruikt.
*   Biedt een handige teller waarmee je virtueel 'zakken pellets' (bijv. 15kg per stuk) kunt toevoegen in Home Assistant om je voorraad digitatief te beheren.

### 5. Kachel Onderhoud & Ventilator Tuner 🔧💨
*   **Onderhoudsstatistieken:** Houdt branduren en cycli bij om te signaleren wanneer de kachel een reinigingsbeurt nodig heeft.
*   **Ventilator Tuner:** Maakt het mogelijk om de ventilatiesnelheden en curves nauwkeurig bij te stellen voor een optimale en stille verbranding.

---

## 🚀 Aan de slag & Kalibratie

### 1. Installatie
1. Kloon deze repository naar je lokale ESPHome-omgeving.
2. Pas in `pelletstove.yaml` je Wi-Fi-gegevens, wachtwoorden en gewenste encryptiesleutels aan.
3. Wijs je externe Home Assistant temperatuursensor toe bovenin bij de `substitutions:` block:
   ```yaml
   substitutions:
     externe_ha_temperatuursensor: "sensor.woonkamer_temperatuur"
   ```
4. Flash de code op je ESP32.

### 2. Niveausensor Kalibreren
Open de **Logs** van je ESPHome-node in het dashboard om de exacte diepte van je pelletbunker te kalibreren:
1.  **Lege Bunker:** Kijk welke afstand (in meters) de sensor meet als de bunker leeg is (bijv. `0.45 m`). Pas dit aan in de filters van je niveausensor in `sensor.yaml`.
2.  **Volle Bunker:** Kijk welke afstand de sensor meet bij een maximaal gevulde bunker (bijv. `0.10 m`).
3.  Pas de `calibrate_linear` mapping aan naar jouw waarden (bijv. `0.10 -> 100%` en `0.45 -> 0%`) en re-flash de ESP32.

---

## 📂 Project Structuur

De configuratie maakt gebruik van een opgesplitste architectuur onder de `packages:` hoofdmap om wildgroei van code te voorkomen:

```bash
├── pelletstove.yaml          # Hoofdconfiguratiebestand (Wi-Fi, OTA, Substitutions & Packages)
└── components/
    ├── uart.yaml             # Seriële hardware-communicatie met het Duepi-board
    ├── sensor.yaml           # Basissensoren (Niveau in meters, procenten en kachel-data)
    ├── level_sensor_time.yaml# Resterende tijd en verloopsnelheid berekeningen
    ├── pellet_efficiency.yaml# Verbruiks-AI (omzetting naar kg/h en g/min)
    ├── stove_maintenance.yaml# Branduren, onderhoudscycli en statistieken
    ├── temperature_failover.yaml # C++ logica voor de automatische thermostaat-backup
    ├── fan_tuner.yaml        # Regelaars voor ventilatiesnelheden
    ├── globals.yaml          # Globale variabelen voor geheugenopslag (tellers)
    └── ... (overige hardware componenten zoals switch, number, climate, script)
```

---

## 🤝 Credits & Dankwoord

*   **@mvroosmalen1970** - Voor de uitstekende basiscomponenten en het ontcijferen van de Duepi EVO / Artel seriële communicatie.
*   **SmartHomeScene** - Voor de inspiratie achter de logica en wiskundige berekeningen van de ultrasone brandstofniveaus.

---
*Gemaakt met 🔥 en passie voor een slimmer en efficiënter thuis.*
