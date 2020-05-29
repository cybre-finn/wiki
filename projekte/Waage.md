## Hardware
### Waagzelle PSD-S1 (2000KG)
-  [Datasheet von baugleicher oder aehnlicher Waage](https://cdn.sparkfun.com/assets/parts/1/2/2/3/8/TAS501.pdf)
- Laut Verkaeufer lineares Verhalten
- Beschriftung auf Seite: `2000KG Y18091972`

Wenn es sich um eine uebliche Waagzelle handelt, dann ist das eine Messbruecke
mit vier Dehnungswiderstaenden:
[[/uploads/projekte/Waage/messbr.png]]

### ESP32
- Datasheet: https://www.espressif.com/sites/default/files/documentation/esp32_datasheet_en.pdf
- technical Reference https://www.espressif.com/sites/default/files/documentation/esp32_technical_reference_manual_en.pdf
- Hat Bluetooth auf eigenem Prozessor
- ADC: 12bit-SAR (Successive Apporoximation) -> sinnvoll einfach den zu benutzen?

### HX711 ADC
- ADC: 24bit Sigma-Delta ADC
- Datasheet: https://cdn.sparkfun.com/datasheets/Sensors/ForceFlex/hx711_english.pdf
- Misst Differenzialspannung in der Messbruecke (s.O.).

## Design-Entscheidungen
~~Es scheint sinnvoller zu sein, den SAR zu nehmen, da dieser trotz geringerer Aufloesung mehr Samples/S liefert. Die Samples sind hier bei kompletter Ausnutzung der Breite etwa 0,5 Meter breit. Bei dem 24-Bit ADC vom HX711 waere das sehr viel besser.
Eines der Probleme des HX711 ist allerdings die eventuell grosse Unsicherheit bei Peaks, da sich der Kondensator hier erst aufladen muesste.~~

Tatseachlich ist der Potentialunterschied an der Messbruecke so gering, dass wir zunaechst den HX711 nehmen muessen, weil dieser einen Operationsverstaerker eingebaut hat.