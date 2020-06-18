## Hardware

## Jobs fuer gesunde Prokrastination:
- Platine zusammenstecken
- Arduino-Programm raufladen und mal die ausgabe vom ADC angucken

### Waagzelle PSD-S1 (2000KG)
-  ~~[Datasheet von baugleicher oder aehnlicher Waage](https://cdn.sparkfun.com/assets/parts/1/2/2/3/8/TAS501.pdf)~~
- [Datasheet(lokal)](/uploads/projekte/Waage/061004354176.jpg) und beim
  [Hersteller](http://www.cnpushton.com/en/index.php?c=product&id=36)
- Laut Verkaeufer lineares Verhalten
- Beschriftung auf Seite: `2000KG Y18091972`

Wenn es sich um eine uebliche Waagzelle handelt, dann ist das eine Messbruecke
mit vier Dehnungswiderstaenden:
[[/uploads/projekte/Waage/messbr.png]]

Die Waage wird als symmetrische Bruecke bei Nullast ausgeliefert.
Bei symmetrischer Bruecke gilt mit $$R_1=R_2=R_3=R_4=R$$ die Gleichung $$R_{Quellwiderstand}=R$$ .

Also hat jeder der Widerstaende bei Nullast $$350\Omega$$ (siehe Datasheet).

Spannungsabfall am Spannungsteiler mit dem auf 2T belasteten Dehnungswiderstand ist laut Datasheet $$~2mV/V$$ (rated output). Das kommt hin, wenn ich mich an die Waage haenge, dann misst das Multimeter 0,1mV bei irgendwas um 10V (?) Eingangsspannung.

Fuer unbelasteten Spannungsteiler gilt: $$V_{out}=\frac{R_2}{R_1+R_2}*V_{in}$$.

Umgestellt nach $$R_2$$: $$R_2 = R_1 * \frac{1}{({\frac{V_{in}}{V_{out}}-1})}$$.

### ESP32
- Datasheet: https://www.espressif.com/sites/default/files/documentation/esp32_datasheet_en.pdf
- technical Reference https://www.espressif.com/sites/default/files/documentation/esp32_technical_reference_manual_en.pdf
- Hat Bluetooth auf eigenem Prozessor
- ADC: 12bit-SAR (Successive Apporoximation) -> sinnvoll einfach den zu benutzen?

### HX711 ADC
- ADC: 24bit Sigma-Delta ADC
- Datasheet: https://cdn.sparkfun.com/datasheets/Sensors/ForceFlex/hx711_english.pdf
- Misst Differenzialspannung in der Messbruecke (s.O.).

### AMS1117-3.3
- Standard-Spannungsregler.
- Datasheet http://www.advanced-monolithic.com/pdf/ds1117.pdf

### LM324N
- Standard-Operationsverstaerker.
- [Datasheet](https://www.ti.com/lit/ds/symlink/lm324-n.pdf)
Aus 3 der Dinger kann man einen Instrumentenverstärker (Impamp bauen)

Im Datasheet findet sich da ein Aufbau dazu. $$R_2$$ muss eingestellt werden (siehe Formel Datasheet).

Umgestellt nach R2:
$$R_2=\frac{2*R_1*(V_2-V_1)}{V_o-1}$$

Dazu muss natürlich die Differenzspannung bei Vollauslastung (V2-V1 ) zuvor berechnet werden.

## Design-Entscheidungen
~~Es scheint sinnvoller zu sein, den SAR zu nehmen, da dieser trotz geringerer Aufloesung mehr Samples/S liefert. Die Samples sind hier bei kompletter Ausnutzung der Breite etwa 0,5 Meter breit. Bei dem 24-Bit ADC vom HX711 waere das sehr viel besser.
Eines der Probleme des HX711 ist allerdings die eventuell grosse Unsicherheit bei Peaks, da sich der Kondensator hier erst aufladen muesste.~~

Tatseachlich ist der Potentialunterschied an der Messbruecke so gering, dass wir zunaechst den HX711 nehmen muessen, weil dieser einen Operationsverstaerker eingebaut hat.

Nuetzlicher Link zu Operationsverstaerkern https://predictabledesigns.com/introduction-to-load-cell-conditioning-circuits/

Stand 11.06 ist ein Opamp vorhanden. Mal gucken.


## Offene Fragen
- Was genau ist Bluetooth-Serial eigentlich?
- Ist es sinnvoll, die uebertragung irgendwie zu schuetzen?

## Sonstige Takeaways
- Wenn man in einer Whetonschen Messbruecke versucht einzelne Widerstaende zu messen, indem man mit dem Multimter an zwei Kabeln misst, dann misst man nicht nur diesen Widerstand, sondern auch die drei anderen, die ja quasi parallel mit dem einzelnen in der Messbruecke verschaltet sind. Damit ist der Widerstand geringer, als der Einzelwiderstand. Der Einzelwiderstand ist uebrigens genau der Gesamtwiderstand (gemessen an den beiden Stellen, wo beide Messpunkte gleich weit "entfernt sind"), wenn alle Einzelwiderstaende gleich sind.
