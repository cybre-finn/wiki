[[_TOC_]]
# Hardware
## Status
Prototyp 0.3a mit HX711 scheint vernünftig zu funktionieren. Software muss geschrieben werden.

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

#### Der richtige Gain
Die Zelle hat bei Vollauslastung eine Differenzspannung von $$2mV/V$$
Bei 3.3V ist demnach DiffSpannung bei $$6.6mV$$.

Das Datasheet (Seite 1) kann so interpretiert werden, dass bei einer Differenzspannung von 20mV
und einem Gain von 128 die Ausgabe $$2^{24-1}-1$$ ist.
Das ergibt sich aus der Tatsache, dass wir einen 24Bit-ADC haben, der aber auch negative Zahlen ausgeben kann (und die Null vom 2er-Komplement fällt weg).

Der Umrechnungsfaktor zu Kilogramm aus den Rohwerten ist demnach: 
$$\frac{1}{(\frac{6.6mV}{20mV}*(2^{24-1}-1))/2000kg}=0.000722481$$

In der Realität passt aber besser folgendes besser auf die gemessenen Gewichte:
$$\frac{1}{(\frac{6.6mV}{20mV}*(2^{24-1}-1))/1000kg}=0.000361240$$

Der Umrechnungsfaktor a aus einer Vorhandenen Messung mit einer Kalibrierten Waage und Rohwerten d aus dem HX711:
$$a=\frac{m_{Kalibriert}}{d_{Cyberscale}}$$

### AMS1117-3.3
- Standard-Spannungsregler.
- Auf Board als Spannungsversorgung fuer ESP8266
- Datasheet http://www.advanced-monolithic.com/pdf/ds1117.pdf

### LM317
- Datasheet: https://www.ti.com/lit/ds/symlink/lm317.pdf
- [Ein Rechner](https://circuitdigest.com/calculators/lm317-resistor-voltage-calculator) liefert die Werte fuer den Spannungsteiler.
- In diesem Fall bei R1=100R R2=700 sind es genau 10V.

### LM324N
- Standard-Operationsverstaerker.
- [Datasheet](https://www.ti.com/lit/ds/symlink/lm324-n.pdf)
Aus 3 der Dinger kann man einen Instrumentenverstärker (Impamp bauen).

Im Datasheet findet sich da ein Aufbau dazu. $$R_2$$ muss eingestellt werden (siehe Formel Datasheet).
**Wichtig ist hierbei, dass im Fall unserer Waage die beiden Eingaenge des Opamps ganz rechts vertauscht werden muessen, sonst gleichen sich die Gains Gegenseitig aus (als andersrum als auf folgendem Bild).**

[[/uploads/projekte/Waage/Screenshot_2020-06-27 LMx24-N, LM2902-N Low-Power, Quad-Operational Amplifiers datasheet (Rev D) - lm324-n pdf.png]]

Umgestellt nach R2 (Gain-Resistor):
$$R_2=\frac{2*R_1*(V_2-V_1)}{V_o-1}$$

Dazu muss natürlich die Differenzspannung bei Vollauslastung (V2-V1 ) zuvor berechnet werden.
Bei 10V betraegt diese 20mV.

$$R2:= \frac{2*1000000\Omega *0.020V}{3.3-1}=1739.13\Omega$$.

### INA122
- Instrumentenverstaerker
- [Datasheet](https://www.ti.com/lit/ds/symlink/ina122.pdf)
- Gain kann mit $$R_g$$ eingestellt werden
- Errinerung: ADC-Wertebereich: 0-1V
- Bei 2mV/V Querspannung bei Vollast betraegt der Gain bei 10V 20mV. $$g:= 1V/20mV=50$$.
- Formel aus Datasheet: $$Gain=5+\frac{200k\Omega}{R_g}$$.
- Umgestellt nach R_g: $$R_g=\frac{200K\Omega}{Gain-5}$$.
- $$R_g:= \frac{200K\Omega}{50-5}=4.4444K\Omega$$.

## Design-Entscheidungen
Es scheint sinnvoller zu sein, den SAR zu nehmen, da dieser trotz geringerer Aufloesung mehr Samples/S liefert. Die Samples sind hier bei kompletter Ausnutzung der Breite etwa 0,5 Meter breit. Bei dem 24-Bit ADC vom HX711 waere das sehr viel besser.
Eines der Probleme des HX711 ist allerdings die eventuell grosse Unsicherheit bei Peaks, da sich der Kondensator hier erst aufladen muesste.

Aktueller Ansatz auf dem Testboard ist entweder ein INA122 Instrumentenverstaerker oder ein LM324N-4fach-Operationsverstaerker, aus dem dann zur Verstaerkung des Signals ein Inamp gebaut wird.

## Offene Fragen
- Was genau ist Bluetooth-Serial eigentlich?
- Ist es sinnvoll, die uebertragung irgendwie zu schuetzen?

## Sonstige Takeaways
- Wenn man in einer Whetonschen Messbruecke versucht einzelne Widerstaende zu messen, indem man mit dem Multimter an zwei Kabeln misst, dann misst man nicht nur diesen Widerstand, sondern auch die drei anderen, die ja quasi parallel mit dem einzelnen in der Messbruecke verschaltet sind. Damit ist der Widerstand geringer, als der Einzelwiderstand. Der Einzelwiderstand ist uebrigens genau der Gesamtwiderstand (gemessen an den beiden Stellen, wo beide Messpunkte gleich weit "entfernt sind"), wenn alle Einzelwiderstaende gleich sind.
- Es gibt Single-Supply und Dual Supply-Opamps und Inamps. Der Unterschied besteht scheinbar meist darin, ob der Referenzpin problemlos auf die selbe Masse wie die Opamp-Versorgungsspannung gelegt werden kann oder nicht. In unserem Fall muss es ein Single-Supply-Geraet sein, weil die Spannungsversorgung ueber ein Batteriegeraet laeuft und nicht getrennt ist.

## v0.1
- Fehler sind schon während Fertigung aufgefallen, Fertigung rechtzeitig abgebrochen, Ersatz durch v0.2

## v0.2
- Auslesevariante LM324 hat nicht gut funktioniert, genauere Evaluation wird aus zeitgründen nicht erhoben
- INA122 hatte eigenartig grosses Offset
- ADC1 wird vom Radio gebraucht und stand für INA122 nicht zur Verfügung. Hotfix mit Kabel zu einem ADC2-Pin
- ADC aktiv im Bereich 0-1V und nicht 0-3.3V

## v0.3
### Vorüberlegungen
- zweiter Versuch mit HX711
- INA122-Pads bleiben, aber zu ADC2
- LM324 kommt weg

## v0.3a
- mal nur mit HX711 streng nach Datasheet

## v0.3a
- mal nur mit HX711 streng nach Datasheet

### Ergebnis
gutes Ergbnis, HX711 macht Readouts,
ABER 
* die Beinchen vom internen Transistor am Spannungsregler sind leider vertauscht.
* Der Tare-Button liegt auf dem Pin 35. Weil Pin 31-39  aber keinen internenen Interrupt hat, war das nichts für Interrupts.
* Ein 3.3V-Trace fehlt noch

Für nächste Version
* ggf noch bessere Stromversorgung als den neuen Spannungsregler
* Abschaltung akku bei geringer Spannung
* USB-C-Ladeteil

weitere TODO:
- Pin 9 und 10 beim Hx711 Smbol sind evtl. vertaucht

# Software
## Embedded
Erstmal mit Arduino-Framework, bis man da auf Probleme stösst.