---
title: "ARBEITSTITEL Ressourcenverbrauch Messen"
date: 2024-07-11T14:18:38+02:00
lsatmod: 2024-07-11T14:18:38+02:00
author: "[Florian Wende](https://www.linkedin.com/in/fwende/)"
type: "post"
image: "TODO"
tags: [ "Green Software Engineering", "Emissionen", "Ressourcenverbrauch", "Messen", "Tools" ]
draft: true
summary: "TODO"
---

Im Zuge der Digitalisierung wächst der Energieverbrauch und damit der ökologische Fußabdruck von Software
stetig. Das gilt insbesondere für die in Clouds laufenden Microservices. Mit geeigneten Werkzeugen, von denen wir
drei vorstellen und vergleichen, kann dieser Fußabdruck gemessen werden.

Der Anteil der IT an den globalen Emissionen liegt geschätzt zwischen 2 und 4 Prozent [Frei21] und damit auf einem
ähnlichen Niveau wie das Flugwesen, das gemeinhin als großer Erzeuger von Emissionen gilt. In der IT entstehen
Emissionen durch die Herstellung der benötigten Hardware sowie über den Nutzungszeitraum der Hardware. Trotz stetiger
Verbesserungen wird der Großteil der weltweit in der IT verbrauchten Energie aktuell noch immer aus fossilen
Energieträgern gedeckt. Daher korrelieren hier Emissionen und Energieverbrauch, und eine Reduzierung des
Energieverbrauchs führt zu niedrigeren Emissionen.

## Energieverbrauch von Microservices messen

Die exakte Messung des Energieverbrauchs einer einzelnen Anwendung ist nicht praktisch machbar, insbesondere nicht in
einem mehrfach virtualisierten Set-up wie bei typischen Cloud-Umgebungen. Stattdessen kommen Vereinfachungen,
Annäherungen und Modelle zum Einsatz. Für praktisch relevante Ansätze müssen dabei mindestens die drei folgenden
Anforderungen erfüllt sein:

* _Messbarkeit_ beschreibt die Auswahl und Messung solcher Kenngrößen, die einerseits während der Laufzeit eines
  Microservice leicht zu bestimmen sind und andererseits in Beziehung zum Energieverbrauch stehen. Üblicherweise sind
  dies die Anteile des Microservice an der Auslastung von CPU und Hauptspeicher, da beide Werte häufig den größten
  Anteil am Energieverbrauch einer Anwendung tragen. Dazu kommt gelegentlich die Menge an über das Netzwerk übertragener
  Daten, wobei hier die Berechnung des Energieverbrauchs deutlich schwieriger fällt.
* _Vergleichbarkeit_ beschreibt, wie man stabile Messungen erzielt. Typisch dafür sind mehrfache Wiederholungen der
  Messungen mit Berechnung von Mittelwerten, um zufällige Schwankungen und Beeinflussungen durch andere Prozesse auf
  einem Computer auszugleichen. Ein anderer Ansatz sind längere Messungen, um unerwünschte Effekte auf die
  Messergebnisse zu verringern.
* _Reproduzierbarkeit_ beschreibt, dass eine Messung wiederholbar und damit nachprüfbar ist. Die Messung sollte auf
  gleicher oder ähnlicher Hardware zu vergleichbaren Ergebnissen kommen. Sie stellt außerdem sicher, dass es möglich
  ist, den Effekt von Änderungen in Implementierungen zu erkennen oder Trends aufzudecken.

Auch wenn der Energieverbrauch von Microservices nur indirekt und potenziell annähernd bestimmt werden kann, erlauben
diese drei Anforderungen Vergleiche zwischen unterschiedlichen Varianten eines Microservice. Mit LiMo, dem Green Metrics
Tool und Kepler stehen drei Messwerkzeuge zur Verfügung, die diese Anforderungen erfüllen. Alle drei verfolgen leicht
unterschiedliche Ansätze mit ihren eigenen Stärken und Schwächen.

## Messen mit einem linearen Modell – LiMo

GreenFrame [Gfm] ist ein Open-Source-Messwerkzeug zur Bestimmung des Energieverbrauchs von Webseiten. Es bietet
allerdings auch die Möglichkeit, den Energieverbrauch von containerisierten Anwendungen wie zum Beispiel Microservices
zu messen. Das Kernmodell von GreenFrame haben wir zu einer Konsolenapplikation namens LiMo, als Abkürzung für „Lineares
Modell“, kondensiert [Git]. Die Motivation für die Namenswahl liefert das Vorgehen von Green- Frame, das CPU- und
Speicherverbrauch der Container misst und beide Zahlen mit konstanten, empirischen Werten normalisiert und zu einem
Energieverbrauch aggregiert. Dieser Wert gibt den Verbrauch für ein exemplarisches System wieder und kann nicht als
absolute Kenngröße genutzt werden. Für Gegenüberstellungen ist der Wert allerdings sehr wohl hilfreich.
Der Vergleich der Implementierungen des Beispiel-Microservice nutzt das in Abbildung 4 dargestellte Set-up. Die
jeweilige Microservice-Version wird lokal auf einem Entwickler-Notebook in Docker- Compose ausgeführt und via k6 unter
Last gesetzt. Parallel läuft LiMo und misst den Energieverbrauch. Als Testzeitraum reichen drei Minuten, da innerhalb
dieser Zeit sowohl CPU- als auch Speicherverbrauch weitgehend konstant bleiben. Java-basierte Implementierungen erhalten
eine Minute Vorwärmzeit unter Last, damit JIT-Kompilierungen (Just-in-time) die eigentlichen Messungen nicht verzerren.
Dieses Vorgehen ist fair, da bei längerer Laufzeit eines Microservice die initiale JIT-Zeit nicht ins Gewicht fällt.

Die Messungen erfolgen unter fünf unterschiedlich hohen Lastszenarien, bei denen die drei Endpunkte in jeweils gleicher
Verteilung aufgerufen werden [Git]. Abbildung 5 gibt die Messergebnisse von LiMo wieder. Es zeigt sich bei allen
Implementierungen eine nahezu lineare Korrelation zwischen Aufrufrate und Energieverbrauch. Das bedeutet, dass andere
Aspekte der Implementierungen nur einen geringen Einfluss auf den Energieverbrauch haben und bei zunehmender Last in den
Hintergrund treten. Zudem sind deutliche Unterschiede zwischen den Implementierungen erkennbar. Rust verbraucht deutlich
weniger Energie als alle anderen Technologien. Insbesondere können mit Rust, bei gleichem Energieverbrauch, fünfmal so
viele Requests wie mit Go als zweitbester Technologie verarbeitet werden. Die Java-Implementierungen liegen im
Mittelfeld und sind zum Teil vergleichbar effizient wie Go. Hierfür ist vermutlich die Reife des Garbage Collector (GC)
von Java im Gegensatz zum GC von Go verantwortlich. Der Einsatz von nativem Kompilieren für Java zeigt bei den Messungen
keinen Vorteil und ist zusammen mit Node am Energie-hungrigsten.

## Hardwarenahes Messen – Green Metrics Tool

Das Green Metrics Tool [Gmt] ist ein Open-Source-Messwerkzeug zur Erfassung des Energieverbrauchs containerisierter
Anwendungen. Es benutzt eine detaillierte Ausführungssteuerung, bei der neben Lastzeiten auch Ruhezeiten berücksichtigt
werden. Dabei werden Kenngrößen aus vielen verschiedenen Sensoren, je nach Laufzeitumgebung, gemessen. Für
virtualisierte Umgebungen steht unter anderem ein Sensor zur Verfügung, der mithilfe eines Machine-Learning-Modells den
Energieverbrauch des Prozessors anhand der CPU-Auslastung sowie konfigurierbarer Hardware-Metadaten abschätzt. Dadurch
ergibt sich eine vergleichsweise hardwarenahe Bestimmung des Energieverbrauchs auch für durch Container virtualisierte
Anwendungen. Die während der Messung über die Sensoren erhaltenen Werte werden in einer Datenbank gespeichert. Eine
Benutzeroberfläche ermöglicht die Visualisierung sowie den Vergleich verschiedener Messungen.

Wir haben das Green Metrics Tool in einer Public-Cloud-Umgebung eingesetzt (vgl. Abbildung 6). Hierbei lief nur der
Microservice zusammen mit dem Green Metrics Tool in einer Instanz. Alle anderen Komponenten liefen auf separaten
Instanzen, um die Messung nicht zu beeinflussen.

Der gemessene Energieverbrauch der verschiedenen Implementierungen ist in Abbildung 7 dargestellt. Für Spring brach das
Szenario unter hoher Last wiederholt ab, sodass nur Werte für geringe bis mittlere Last vorliegen. Die vom Green Metrics
Tool gemessenen Zahlen ähneln denen von LiMo. Insbesondere zeigt sich Rust als klar effizienteste Implementierung. Go
und Java, sowohl mit Spring als auch mit Quarkus, liegen im Mittelfeld. Node und die native Java-Implementierung folgen
dahinter.

## Messen im Kubernetes-Cluster – Kepler

Kepler [Kep] ist ein Open-Source-Messwerkzeug zur Protokollierung des Energieverbrauchs von Nodes und Pods in einem
Kubernetes-Cluster. Während der Laufzeit von Microservices sammelt es kontinuierlich Verbrauchsdaten von verschiedenen
Sensoren, insbesondere von Performance-Sensoren im Linux-Kernel. Aus diesen Daten aggregiert Kepler Übersichten, wobei
ein lineares Modell ähnlich dem in LiMo zum Einsatz kommt. Optional kann ein Machine-Learning-Modell aktiviert werden,
was wir in unseren Tests allerdings nicht benutzt haben.

Wie beim Green Metrics Tool haben wir Kepler in einer Public-Cloud-Umgebung eingesetzt (vgl. Abbildung 8). Genau wie bei
den anderen Set-ups lief der Microservice ohne Replikation in einer Instanz. Sowohl Backend als auch Kepler liefen
ebenso im Cluster, während wir die Lasttests von einem Entwickler-Notebook ausgeführt haben.

Abbildung 9 zeigt die von Kepler gemessenen Werte für die Microservice-Implementierungen. Im Gegensatz zu den Messungen
mit den beiden anderen Werkzeugen zeigen sich hier deutliche Schwankungen statt einem klaren linearen Verhalten. Die
Tests liefen nur jeweils drei Minuten, wie auch bei den anderen beiden Werkzeugen. Längere Laufzeiten könnten eine
Glättung der Ergebnisse bewirken. Node fiel unter hoher Last wiederholt aus, sodass dafür keine Messwerte vorliegen.
Unter geringer bis mittlerer Last schlägt sich Node allerdings vergleichbar gut wie Rust, das erneut als sparsamste
Implementierung auffällt. Für Go und die verschiedenen Java- Implementierungen ist unter Ausblenden der Schwankungen
kein signifikanter Unterschied erkennbar. Hier ist allerdings die native Version von Java vergleichbar mit den
JVM-Versionen.

## Vergleich der Messwerkzeuge

Die Messungen mit Kepler im Vergleich mit den anderen beiden Werkzeugen zeigen, dass unterschiedliche Messansätze zu
durchaus unterschiedlichen Ergebnissen führen können. Die betrachteten Messwerkzeuge unterscheiden sich in ihren
Ansätzen. LiMo und das Green Metrics Tool sind vorrangig für Benchmarks gedacht, während Kepler kontinuierliche
Messungen, selbst in produktiven Umgebungen, erlaubt, woraus man Trends ablesen kann.

Unserer Erfahrung nach lassen sich alle drei Werkzeuge leicht aufsetzen, wobei LiMo aufgrund seiner Einfachheit einen
Vorteil aufweist. Im Betrieb wirkt Kepler unausgereift, weil wir häufige Aussetzer und Fehler beobachtet haben. LiMo und
das Green Metrics Tool sind dagegen robust. Kepler und das Green Metrics Tool bieten mit einer großen Zahl an
unterstützten Sensoren sowie der Aufzeichnung der Messergebnisse und grafischen Übersichten einen deutlichen Mehrwert
gegenüber LiMo in Bezug auf Benutzerfreundlichkeit.

Wir halten LiMo für ein gutes Werkzeug für initiale, einfache Messungen und Vergleiche. Das Green Metrics Tool wirkt
insgesamt vertrauenswürdiger und erlaubt zudem Analysen vom Zusammenspiel mehrerer Microservices. Von Kepler raten wir
gegenwärtig eher ab aufgrund der regelmäßig auftretenden Probleme.