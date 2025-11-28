# 🧐 Contesto e Problema
I sistemi di routing standard basati su GTFS (General Transit Feed Specification) modellano la rete di trasporto come una sequenza logica di fermate. Questo approccio presenta due limitazioni critiche per l'analisi urbana avanzata:
  1- Cecità Topologica: Il percorso tra due fermate viene spesso approssimato       come una linea retta, ignorando la reale conformazione della strada.
  2- Neutralità al Rischio: L'algoritmo cerca esclusivamente di minimizzare il      tempo di viaggio, non distinguendo tra un percorso sicuro e uno che            attraversa aree ad alta incidentalità stradale.
Questo progetto mira a colmare questo divario, creando uno strumento di supporto alle decisioni che bilanci efficienza e sicurezza.

# 💡 La Soluzione Proposta
Abbiamo sviluppato un grafo multimodale arricchito che fonde diverse fonti dati:
  -Dati GTFS Statici: Orari, fermate, percorsi logici (Agenzia SETA/AMO           Modena).
  -GTFS Shapes: Coordinate GPS dettagliate per ricostruire la geometria           fisica dei percorsi bus.
  -Dati Raster (GeoTIFF): Mappe di probabilità di incidente variabili per         fascia oraria (Mattina, Pomeriggio, Sera, Notte).
  -Dati Territoriali (GeoJSON): Confini dei quartieri di Modena per analisi       Origine-Destinazione (OD).
  
L'algoritmo di routing utilizza una funzione di costo, che crea un sorta di ritardo virtuale, generalizzata per trovare il percorso ottimo:
  $$\text{Costo Totale} = \text{Tempo di Viaggio} + (\text{Indice di Rischio} \times \text{Fattore di Penalità})$$
  
# ⚙️ Metodologia e ArchitetturaIl progetto esplora e confronta due approcci implementativi distinti per la modellazione del grafo:1. Approccio "Full Geometry" (Modellazione Fisica)Descrizione: Ogni singola coordinata GPS del tracciato stradale viene istanziata nel grafo come un nodo fisico (ShapePoint). Il rischio viene mappato su ogni micro-segmento stradale.Pro: Massima precisione spaziale per query granulari (es. "incidenti in questo specifico incrocio").Contro: Esplosione della cardinalità del grafo (milioni di nodi), routing computazionalmente oneroso.2. Approccio "Semplificato/Edge-Based" (Ottimizzato per Routing)Descrizione: La complessità geometrica viene gestita nella fase di pre-elaborazione (ETL in Python). Un algoritmo di Map Matching calcola la geometria esatta e il rischio medio del segmento tra due fermate e salva queste informazioni come proprietà dell'arco logico (PRECEDES).Pro: Grafo estremamente leggero, routing rapidissimo (millisecondi) su hardware standard, mantenimento dell'informazione geometrica completa sugli archi.Nota: Questo è l'approccio raccomandato per il calcolo delle matrici OD tra quartieri.💻 Stack TecnologicoCore Database: Neo4j Graph Database (Versione 5.x+)Neo4j Plugins (Obbligatori):APOC (Awesome Procedures on Cypher) - Per utility e caricamento dati avanzato.GDS (Graph Data Science Library) - Per l'esecuzione dell'algoritmo di Dijkstra e le proiezioni in memoria.Linguaggio di Scripting: Python 3.8+Librerie Python Chiave:neo4j (Driver ufficiale Bolt)pandas & geopandas (Manipolazione dati tabellari e geospaziali)rasterio (Campionamento dati raster/immagini)shapely (Operazioni geometriche, creazione WKT)
