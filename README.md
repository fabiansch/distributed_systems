# distributed_systems
Repository contains notes of a distributed system lecture at HAW Hamburg and can be used for preparation of the exam. The notes are ordered within the chapters and subchapters of the book [Distributed Systems](https://www.distributed-systems.net/index.php/books/distributed-systems-3rd-edition-2017/)

Everybody is invited to improve the notes :) Just send a pull request.


Optimierungsdreieck in Verteilten Systemen

           C
         /     \
       P   -   A

C (Consistency) 
A (Availibility) 
P (Tolerant dem Netzwerk)

CP - Banktransaktion
CA - Filesystem / LDAP
AP - DNS

jrun -cp CaDBase.jar:Server.jar middleware.core.server.Server 10.0.1.1 real

Was ist ein VS? Tannenbaum liefert Definition. Z.B.: System ohne kohärenten Speicher das sich allerdings verhält als ob er kohärent wäre.

Design Goals: 7 Transparenzziele
==> Location Transparenz (z.B. REST mit DNS)
==> being open. (z.B. IDL) problematisch bei heterogenen Systemen => daher Minimale Objektmenge
==> Skalierbarkeit bedingt replizieren. Mach kann örtlich skalieren und ... skalieren. Verteilter Algorithmus ist notwendig.
==> Pitfaults: 8 Annahmen die man nie annehmen sollte.
==> 3 types of DS (Schnelles rechnen, Informationen verteilen, IOT)

Architektur:
==> Style
Layered: Funktional
Object-Based: Service Orientiert (Objekte in Hierarchischem Namespace)
Resource-Based: CRUD (machbar mit http)
Publish Subscribe: Message Queue (Kafka (groß), RapidMQ, Ross)

==> Middleware
Wrapper: Gleiche Middleware und Wrapper binden die Applikationslogik an
Interceptors: gutes Bild im Buch
Modifiable Middleware: Was muss man machen damit Middleware erweiterbar ist..

==> System Architecture
Publish Subscribe (Bild)
Thin Client, Fat Client

Client-Server-Server (CSS) - Zentralisierter Ansatz
Peer-To-Peer - Dezentraler Ansatz. Tannenbaum macht immer CORD P2P was ein spezielles P2P ist (Cassandra beruht darauf und ist hoch skalierbar)
Hybrid (google, amazon: relativ homogene und gut verbundene Datacenter und somit teils P2P möglich)


Processes:

kennen wir schon, aber neue Begrifflichkeit kommt dazu:
statefull und stateless (Nachricht ist unabhängig zu Vorgängernachrichten)
zustandvariant (fileserver) und zustandinvariant (Nachricht beeinflusst nicht den internen Zustandsautomaten)
persistent (Nachrichten werden behalten bis Abnehmer gefunden wird) und transient (Nachrichten werden verworfen wenn sich niemand interessiert) … (Es geht um nachrichtenbasierte Systeme. Was wenn niemand sich für eine Nachricht interessiert)

virtualisierung NEIN
network user interface NEIN
server NEIN

Communication:
layered protocols: osi
types of communication: synchron und asynchron (Bildchen mit Zeitstrahl)

Eine Mittleware die auf Bitcodierung basiert hat den Vorteil, dass sie sehr effizient ist, allerdings
reist sie aus dem Design Muster aus! Daher z.B. RPC.

Neben RPCs gibt es Modelle die auf Queues basieren (Message Oriented Communication)
Point-to-Point (P2P) (Queues - einer steckt etwas rein, einer holt es raus) (durch Queue starke Entkopplung, Producer kann schon tot sein bevor Consumer beginnt zu arbeiten)
Publish-Subscribe (Topics - Sobald Konsumenten ein Topic abonnieren werden ihnen alle über eine bestimmte Zeit zwischengespeicherten und auch alle neuen Nachrichten von einem Producer zugesendet bis sie sich wieder abmelden. 

Multicast Communication
zum Beispiel Publish-Subscribe (Multicast ist nur eine Begrifflichkeit und kann verschieden implementiert werden)

tree-based multicasting (nicht so interessant)
fludding
gossip
Interessant wenn Transaktionen ins Spiel kommen (z.B. bei Replikation einer Datenbank)

Naming;
Definitionen: identity, identifier,..
Warum? für dynamische Bindung der Dienste für Location Transparency
flat fuer peer to peer, hierarchisch für …

Coordination: (Ordnung in die Nachrichten bringen)
physikalische Uhr: Problem ist die gemeinsame Zeitbasis, dafür gibt es das Network-Time-Protocol (NTP) das eine die Zeit einer zentralen Atomuhr beinhaltet. Berkeley Algorithmus funktioniert ohne zentrale Atomuhr, dabei beachten: niemals Zeit zurückstellen!
logische Uhr: Lamport Uhr und Vektor Uhr beruhen beide auf der Happens Before Relation 

Nachteil Lamport: wenn Prozesse nicht miteinander kommunizieren, keine totale Ordnung, nur kausale Ordnung - oft okay wie z.B. für WhatsApp aber für sicherheitskritische Dinge inakzeptabel.
Warum WhatsApp ab und an falsch sortiert: Sortierung muss gequeued werden und dies wird nur über eine gewisse Zeit gemacht. (Bsp: Client versendet mit timestamp versehene Nachricht erst Tage später. Die anderen Nachrichten derselben Zeit sind schon längst aus der Queue verschwunden und in Datenbanken geschrieben.)

Nachteil Vektor Uhr: Skaliert nicht (Algorithmus muss man für Lamport sowie für Vektor zumindest illustrieren können! Ebenso können Verständnisfragen kommen)

Mutual exclusion
In verteilten Systemen haben wir keine atomare zugriffe wie zum Beispiel mit Semaphoren.

Mutual exclusion mit zentralem Server
Mutual exclusion Verteilt (problematisch wenn Koordinator Tod, aber ebenso bei zentralem Ansatz)
Token-Ring (dezentral, jeder muss jeden kennen um bei Ausfällen den Ring aufrecht erhalten zu können) 
…

Election algoritms
Bully (Besten finden z.B. via Broadcast, …, Gewinner informiert alle anderen)
Ring (Zettel und jeder trägt sich ein, zweite runde nötig damit jeder erfährt wer der Gewinner ist)
Beide basieren auf ein stabiles Netzwerk…
Ripple Algorithmus für Netze die instabil sind.

Location
GPS (Genaue Ortsbestimmung benötigt 4 Satelliten: x, y, z und Zeit)
wenn kein GPS - relative Ortung über Landmarken im Raum (eine Dimension 2+, 2 Dimension 3+,..) oder relative Ortung über Delays im Netzwerk oder Stärke von Signalen wie WLAN

Gossip Based Coordination (Wie bekommt man Nachrichten über das Netzwerk verteilt - jeder spricht mit seinen Nachbarn)

Consistency and Replication (Bisher: Wie verteilen wir nachrichten, jetzt: was machen mit den Daten)
Data Centric (Clients bewegen sich nicht)
	Continuos mit Commits und maximaler Abweichung. Freeze bei zu großer Abweichung (Transaktionen müssen warten bis Zustände synchronisiert sind)
	Consistent Ordering (1. sequential consistency - 2. causal consistency für additive Dinge - 3. fifo ohne Beispiele)
Client centric (Clients bewegen sich)
	Client Anforderung: monotonisches lesen/schreiben
ASIC vs BASE

