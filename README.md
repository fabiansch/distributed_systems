# distributed_systems
Repository contains notes of a distributed system lecture at HAW Hamburg and can be used for preparation of the exam. The notes are ordered within the chapters and subchapters of the book [Distributed Systems](https://www.distributed-systems.net/index.php/books/distributed-systems-3rd-edition-2017/)

Everybody is invited to improve the notes :) Just send a pull request.


Optimierungsdreieck in Verteilten Systemen

           C
         /   \
       P   -   A

**C** (Consistency) 
**A** (Availibility) 
**P** (Tolerant dem Netzwerk)

- CP - Banktransaktion
- CA - Filesystem / LDAP
- AP - DNS

# Introduction

## Was ist ein VS?
Tannenbaum liefert spezielle Definition.

alternativ: System ohne kohärenten Speicher das sich allerdings verhält als ob dieser kohärent wäre.

## Design Goals:
7 Transparenzziele
- Location Transparenz (z.B. REST mit DNS)
- being open
  - (z.B. IDL)
  - problematisch bei heterogenen Systemen => daher minimale Objektmenge
    - in Praktikum war die minimale Objektmenge ein Integer
- being scalable
  - Skalierbarkeit bedingt replizieren.
  - Man kann örtlich skalieren und ... skalieren.
  - Verteilter Algorithmus ist notwendig.
- Pitfalls
  - 8 Annahmen die man nie annehmen sollte.

## Types of DS
Ganz grob drei verschiedene typen:
- Schnelles rechnen
- Informationen verteilen
- IOT

# Architektur:
## Styles
- Layered
  - Funktional
- Object-Based
  - Service Orientiert (Objekte in Hierarchischem Namespace)
- Resource-Based
  - CRUD (machbar mit http)
- Publish Subscribe
  - Message Queue (Kafka für grosse Systeme, RapidMQ und Ross für eher kleinere)

## Middleware organization
- Wrapper
  - Ziel: Gleiche Middleware und anwendungsspezifische Wrapper binden die Applikationslogik an
- Interceptors
  - gutes Bild im Buch
- Modifiable Middleware
  - Was muss man machen damit Middleware erweiterbar ist..

## System Architecture
Publish Subscribe (Bild)

Thin Client, Fat Client

- Zentralisierter Ansatz
  - Client-Server-Server (CSS)
- Dezentraler Ansatz
  - Peer-To-Peer (P2P)
  - Tannenbaum macht immer CORD P2P was ein spezielles P2P ist (Cassandra beruht darauf und ist hoch skalierbar)
- Hybrider Ansatz
  - (google, amazon: relativ homogene und gut verbundene Datacenter und somit teils P2P möglich)

# Processes:

kennen wir schon, aber neue Begrifflichkeiten kommen dazu:
- statefull
  - Nachricht ist **abhängig** zu Vorgängernachrichten
- stateless
  - Nachricht ist **unabhängig** zu Vorgängernachrichten
- zustandvariant
  - Nachricht beeinflusst den internen Zustandsautomaten
  - z.B. Fileserver
- zustandinvariant
  - Nachricht beeinflusst den internen Zustandsautomaten **nicht**
- persistent
  - In nachrichtenbasierten Systemen werden Nachrichten **behalten** bis Abnehmer gefunden wird
- transient
  - In nachrichtenbasierten Systemen werden Nachrichten **verworfen** wenn kein Abnehmer gefunden wird

- virtualisierung **NEIN**
- network user interface **NEIN**
- server **NEIN**

# Communication:
## Foundations
- layered protocols
  - osi
- types of communication
  - synchron und asynchron (Bildchen mit Zeitstrahl)

## RPC
Eine Mittleware die auf Bitcodierung basiert hat den Vorteil, dass sie sehr effizient ist, allerdings
reist sie aus dem Design Muster aus! Daher z.B. **RPC**.

## Message Oriented Communication

- Neben RPCs gibt es Modelle die auf Queues basieren

  - Point-to-Point (P2P)
    - Queues: einer steckt etwas rein, einer holt es raus
    - durch Queue starke Entkopplung: Producer kann schon tot sein bevor Consumer beginnt zu arbeiten.

  - Publish-Subscribe
    - Topics: sobald Konsumenten ein Topic abonnieren werden ihnen alle über eine bestimmte Zeit zwischengespeicherten und auch alle zukünftigen Nachrichten von einem Producer zugesendet bis sie sich wieder abmelden. 

## Multicast Communication
zum Beispiel Publish-Subscribe (Multicast ist nur eine Begrifflichkeit und kann verschieden implementiert werden)

Interessant wenn Transaktionen ins Spiel kommen (z.B. bei Replikation einer Datenbank)
- tree-based multicasting (nicht so interessant)
- fludding
- gossip

# Naming:

- Warum? für dynamische Bindung der Dienste um Location Transparency zu erreichen

## Definitionen: names, identifiers and addresses

- flat für peer to peer
- hierarchisch für …

# Coordination:
- Ziel: Ordnung in die Nachrichten bringen.

## physikalische Uhr
- Problem ist die gemeinsame Zeitbasis
  - Network-Time-Protocol (NTP) stellt Zeit einer zentralen Atomuhr bereit.
  - Berkeley Algorithmus funktioniert ohne zentrale Atomuhr
    - Wichtig: niemals Zeit zurückstellen!
    
## logische Uhr
beruhen beide auf der Happens Before Relation
- Lamport Uhr
  - Nachteil: wenn Prozesse nicht miteinander kommunizieren, keine totale Ordnung - nur kausale Ordnung - oft okay wie z.B. für WhatsApp aber für sicherheitskritische Dinge inakzeptabel.
  - Warum WhatsApp ab und an falsch sortiert: Sortierung muss gequeued werden und dies wird nur über eine gewisse Zeit gemacht. (Bsp: Client versendet mit timestamp versehene Nachricht erst Tage später. Die anderen Nachrichten derselben Zeit sind schon längst aus der Queue verschwunden und in Datenbanken geschrieben.)
- Vektor Uhr 
  - Nachteil Vektor Uhr: Skaliert nicht

(Algorithmus muss man für Lamport sowie für Vektor zumindest illustrieren können! Ebenso können Verständnisfragen kommen)

## Mutual exclusion
In verteilten Systemen haben wir keine atomaren Zugriffe wie zum Beispiel mit Semaphoren in nicht verteilten Systemen.

- Mutual exclusion mit zentralem Server
- Mutual exclusion Verteilt
  - problematisch wenn Koordinator Tod, aber dies gilt auch bei zentralem Ansatz
- Token-Ring
  - dezentral, jeder muss jeden kennen um bei Ausfällen den Ring aufrecht erhalten zu können 
- Dezentralisierter Algorithmus

## Election algoritms
- Bully
  1. Besten finden z.B. via Broadcast
  1. …
  1. Gewinner informiert alle anderen
- Ring
  - Zettel und jeder trägt sich ein, zweite runde nötig damit jeder erfährt wer der Gewinner ist.

Beide basieren auf ein stabiles Netzwerk…
- Ripple Algorithmus für Netze die instabil sind.

## Location systems
- GPS
  - Genaue Ortsbestimmung benötigt 4 Satelliten für: x, y, z und die Zeit
- wenn kein GPS
  - relative Ortung über Landmarken im Raum (eine Dimension 2+, 2 Dimension 3+,..)
- wenn keine Landmarken
  - relative Ortung über Delays im Netzwerk oder Stärke von Signalen wie dem WLAN

## Gossip Based Coordination
- Wie bekommt man Nachrichten über das Netzwerk verteilt: jeder spricht mit seinen Nachbarn.

# Consistency and Replication
- Bisher: Wie verteilen wir Nachrichten
- Jetzt: Was machen mit den Daten

## Data Centric
Clients bewegen sich nicht
- Continuos mit Commits und maximaler Abweichung.
  - Freeze bei zu großer Abweichung (Transaktionen müssen warten bis Zustände synchronisiert sind)
- Consistent Ordering
  - sequential consistency
  - causal consistency für additive Dinge
  - fifo ohne Beispiele
  
## Client centric (Clients bewegen sich)
Client Anforderung: monotonisches lesen/schreiben

..
ASIC vs BASE


..to be continued
