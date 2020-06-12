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
- Herausgefunden muss werden: Was ist $$Rf$$ fuer unsere Messbruecke?
- Dabei habe ich rausgefunden: $$\delta$$ ist der Faktor minus 1, um den sich R
  maximal veraendert, wenn R sich wegen Zug oder druck veraendert. Vergroessert
  sich R bei zweit Tonnen etwa um 1.42, dann $$\delta=0.42$$
- ~~Eine Moeglichkeit der Messung von $$\delta$$ waere, wieder was an die Waage zu haengen und entweder direkt den Widerstand zu Messen oder die Spannung, um dann den Widerstand zu Messen. $$\delta_{Testgewicht}=R_{Testgewicht}/R_{unbelastet}-1$$.~~
- Mithilfe der Daten aus aufgefundenen Datasheet der Waage kann Rf auch genau berechnet werden.

[[/uploads/projekte/Waage/messbr-opamp.png]]\

### Berechnung Rf
Die Waage wird als symmetrische Bruecke bei Nullast ausgeliefert.
Bei symmetrischer Bruecke gilt mit $$R_1=R_2=R_3=R_4=R$$ $$R_{Quellwiderstand}=R$$.

Also hat jeder der Widerstaende bei Nullast $$350\Omega$$ (siehe Datasheet).

Spannungsabfall am Spannungsteiler mit dem auf 2T belasteten Dehnungswiderstand ist laut Datasheet $$~2mV/V$$ (rated output). Das kommt hin, wenn ich mich an die Waage haenge, dann misst das Multimeter 0,1mV bei irgendwas um 10V (?) Eingangsspannung.

Fuer unbelasteten Spannungsteiler gilt: $$V_{out}=\frac{R_2}{R_1+R_2}*V_{in}$$.

Umgestellt nach $$R_2$$: $$R_2 = R_1 * \frac{1}{({\frac{V_{in}}{V_{out}}-1})}$$.

Wir legen mal fest: $$U_{in}:=1V$$

Beim rechten Spannungsteiler mit festen Widerstaenden
faellt dann eine Spannung von 0.5V ab ($$U_o=0.5*V_i$$ bei gleichen Widerstaenden).
Daher ist unser $$V_{out}:=0.502V$$, damit wir auf die korrekte Brueckenspannung von 2mV kommen.

Also: $$350\Omega * \frac{1}{({\frac{1V}{0.502V}-1})}=352.8112\Omega$$.

Folglich: $$\delta:=350/352.8112-1=0.00803$$

Nun zur Formel auf der Grafik aus dem Datasheet (s.O.)...

Wir suchen Rf, also umstellen: $$R_f=\frac{2V_{out}*R}{\delta * V_{REF}}$$.

Vout ist am besten 3.3V.

Vref kann man mit zwei Werten mal berechnen, einmal zum Testmessen mit 14V (kurz vorm Maximum der Waage)
und einmal mit 3.3V fuer die Anwendung direkt auf dem Board. Ggf packen wir da noch eine andere Exitation Voltage fuer mehr Genaugigkeit rein, dann brauchts aber einen weiteren Spannungsregler.

| Vref | Rf |
|------|----|
| 3.3V | $$\frac{2*3.3V*350\Omega}{0.00803*3.3}=87173\Omega$$ |
| 14V  | $$\frac{2*3.3V*350\Omega}{0.00803*14}=20548\Omega$$ |

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
- Wenn man in einer Whetonschen Messbruecke versucht einzelne Widerstaende zu messen, indem man mit dem Multimter an zwei Kabeln misst, dann misst man nicht nur diesen Widerstand, sondern auch die drei anderen, die ja quasi parallel mit dem einzelnen in der Messbruecke verschaltet sind. Damit ist der Widerstand geringer, als der Einzelwiderstand. Der Einzelwiderstand ist uebrigens genau der Gesamtwiderstand, wenn alle Einzelwiderstaende gleich sind.
