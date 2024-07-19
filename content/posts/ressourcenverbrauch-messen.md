---
title: "ARBEITSTITEL Wie misst man den Energieverbrauch von Microservices?"
date: 2024-07-11T14:18:38+02:00
lastmod: 2024-07-11T14:18:38+02:00
author: "[Sascha Böhme](https://www.linkedin.com/in/sascha-b%C3%B6hme-24b1782b6/), [Andreas Weber](https://www.linkedin.com/in/andreas-weber-green/) und [Florian Wende](https://www.linkedin.com/in/fwende/)"
type: "post"
image: "ressourcenverbrauch-messen.webp"
tags: [ "Green Software Engineering", "Emissionen", "Ressourcenverbrauch", "Messen", "Tools" ]
draft: true
summary: "Der Artikel stellt drei Messwerkzeuge (LiMo, Green Metrics Tool und Kepler) für den Energieverbrauch von Software vor und vergleicht diese."
---

Im Zuge der Digitalisierung wächst der Energieverbrauch und damit der ökologische Fußabdruck von Software
stetig. Das gilt insbesondere für die in Clouds laufenden Microservices. Mit geeigneten Werkzeugen, von denen wir
drei vorstellen und vergleichen, kann dieser Fußabdruck gemessen werden.

Der Anteil der IT an den globalen Emissionen liegt geschätzt zwischen 2 und 4 Prozent[^1] und damit auf einem
ähnlichen Niveau wie das Flugwesen, das gemeinhin als großer Erzeuger von Emissionen gilt. In der IT entstehen
Emissionen durch die Herstellung der benötigten Hardware sowie über den Nutzungszeitraum der Hardware. Trotz stetiger
Verbesserungen wird der Großteil der weltweit in der IT verbrauchten Energie aktuell noch immer aus fossilen
Energieträgern gedeckt. Daher korrelieren hier Emissionen und Energieverbrauch, und eine Reduzierung des
Energieverbrauchs führt zu niedrigeren Emissionen.

Um die Wirksamkeit von Maßnahmen zur Reduzierung des Energieverbrauchs von Software zu überprüfen, ist es notwendig,
verlässliche Messwerkzeuge an der Hand zu haben, die im besten Fall auch noch angenehm zu bedienen sind. Erst durch
die Messung und den Vergleich verschiedener Referenzimplementierungen der gleichen funktionalen Logik, sind wir in der
Lage, wissenschaftlich fundierte Empfehlungen zur Reduktion von Software verursachten Emissionen zu geben.

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

[GreenFrame](https://github.com/marmelab/greenframe-cli/blob/main/src/model/README.md) ist ein Open-Source-Messwerkzeug
zur Bestimmung des Energieverbrauchs von Webseiten. Es bietet
allerdings auch die Möglichkeit, den Energieverbrauch von containerisierten Anwendungen wie zum Beispiel Microservices
zu messen. Das Kernmodell von GreenFrame haben wir zu einer Konsolenapplikation
namens [LiMo](https://github.com/qaware/microservice-energy-consumption-benchmark), als Abkürzung für „Lineares Modell“,
kondensiert. Die Motivation für die Namenswahl liefert das Vorgehen von GreenFrame, das CPU- und Speicherverbrauch der
Container misst und beide Zahlen mit konstanten, empirischen Werten normalisiert und zu einem Energieverbrauch
aggregiert. Dieser Wert gibt den Verbrauch für ein exemplarisches System wieder und kann nicht als
absolute Kenngröße genutzt werden. Für Gegenüberstellungen ist der Wert allerdings sehr wohl hilfreich.

Als Konsolenapplikation sind LiMo bzw. GreenFrame dafür ausgelegt, lokal auf dem Entwickler-Notebook ausgeführt zu
werden, für kontinuierliche Messungen könnte man sie aber auch in eine CI-Pipeline integrieren. Die zu messende
Anwendung wird dann typischerweise lokal als Container in Docker (oder gemeinsam mit eventuell benötigten
Backend-Services in Docker-Compose) ausgeführt und von einem ebenfalls lokal betriebenen Lastgenerator unter Last
gesetzt. Parallel läuft LiMo und misst den Energieverbrauch.

## Hardwarenahes Messen – Green Metrics Tool

Das [Green Metrics Tool](https://www.green-coding.io/de/projects/green-metrics-tool/) ist ein Open-Source-Messwerkzeug
zur Erfassung des Energieverbrauchs containerisierter
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

## Messen im Kubernetes-Cluster – Kepler

[Kepler](https://sustainable-computing.io/) ist ein Open-Source-Messwerkzeug zur Protokollierung des Energieverbrauchs
von Nodes und Pods in einem
Kubernetes-Cluster. Während der Laufzeit von Microservices sammelt es kontinuierlich Verbrauchsdaten von verschiedenen
Sensoren, insbesondere von Performance-Sensoren im Linux-Kernel. Aus diesen Daten aggregiert Kepler Übersichten, wobei
ein lineares Modell ähnlich dem in LiMo zum Einsatz kommt. Optional kann ein Machine-Learning-Modell aktiviert werden,
was wir allerdings bisher nicht getestet haben.

Wie beim Green Metrics Tool haben wir Kepler in einer Public-Cloud-Umgebung eingesetzt (vgl. Abbildung 8). Genau wie bei
den anderen Set-ups lief der Microservice ohne Replikation in einer Instanz. Sowohl Backend als auch Kepler liefen
ebenso im Cluster, während wir die Lasttests von einem Entwickler-Notebook ausgeführt haben.

## Vergleich der Messwerkzeuge

Die betrachteten Messwerkzeuge unterscheiden sich in ihren Ansätzen. LiMo und das Green Metrics Tool sind vorrangig für
Benchmarks gedacht, während Kepler kontinuierliche Messungen, selbst in produktiven Umgebungen, erlaubt, woraus man
Trends ablesen kann. Dass diese verschiedenen Messansätze durchaus zu unterschiedlichen Ergebnissen führen können,
zeigen wir in unserem Artikel [_The Fast and the Frugal: Microservices at
Race_](https://blog.qaware.de/posts/the-fast-and-the-frugal-microservices-at-race/).

[//]: # (TODO: Fertigen Titel und Link einfügen)

Unserer Erfahrung nach lassen sich alle drei Werkzeuge leicht aufsetzen, wobei LiMo aufgrund seiner Einfachheit einen
Vorteil aufweist. Im Betrieb wirkt Kepler unausgereift, weil wir häufige Aussetzer und Fehler beobachtet haben. LiMo und
das Green Metrics Tool sind dagegen robust. Kepler und das Green Metrics Tool bieten mit einer großen Zahl an
unterstützten Sensoren sowie der Aufzeichnung der Messergebnisse und grafischen Übersichten einen deutlichen Mehrwert
gegenüber LiMo in Bezug auf Benutzerfreundlichkeit.

Wir halten LiMo für ein gutes Werkzeug für initiale, einfache Messungen und Vergleiche. Das Green Metrics Tool wirkt
insgesamt vertrauenswürdiger und erlaubt zudem Analysen vom Zusammenspiel mehrerer Microservices. Von Kepler raten wir
gegenwärtig eher ab aufgrund der regelmäßig auftretenden Probleme.

[^1]: C. Freitag et al., The real climate and transformative impact of ICT: A critique of estimates, trends, and
regulations, Cell Press, 2021
