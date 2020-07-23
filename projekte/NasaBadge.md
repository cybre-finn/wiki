Eine leuchtende Nasa-Badge, die Blinkt, wenn die ISS zu sehen ist.

## Features
- PCB mit Nasa-Logo-Form
- roter Bereich soll durch umgedrehte SMD-LEDs zum leuchten gebracht werden
- 4 Sterne sollen leuchten können
- Button
- Wifi-Sync der Flugbahn
- Leuchten bei ISS-Überflug

## Bauteile
- **[ESP32](https://www.espressif.com/sites/default/files/documentation/esp32_datasheet_en.pdf)**
- **[PZT3904](https://www.mouser.com/datasheet/2/149/2N3904-82270.pdf)** NPN BJT-Transistor (Die SMD-Variante (SOT-223) vom 2N3904)
- **[FT232RL](https://www.ftdichip.com/Support/Documents/DataSheets/ICs/DS_FT232R.pdf)** USB to RS232 interface von FTDI
- **[0603 rote LEDS](https://www.aliexpress.com/item/32886268527.html?spm=a2g0s.9042311.0.0.9b824c4dOkheNj)**
- **0805 Widerstände**
- evtl. **[QuectelL80-R](https://www.quectel.com/UploadFile/Product/Quectel_L80-R_GPS_Specification_V1.2.pdf)**

## LED ansteuern
- [Anleitung](https://www.dummies.com/programming/electronics/components/electronics-components-use-a-transistor-as-a-switch/)
- [Transistor auswählen](https://www.baldengineer.com/the-best-4-transistors-to-keep-in-your-parts-kit.html)

0605 Rote LED von Aliexpress:
- Nennspannung $$U_f = ~1.8V$$
- Nennstrom $$A = ~20mA$$
- Widerstand bei 3.3V: $$75\Omega$$

PZT3904 Transistor für LEDs
- 1.5K Widerstand vor Basis