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