––
# Relazione – [NinjaDB]
Componenti: [Rosa Passaro, IE22700227 - Omar Silano, IE22700120 - Antonio Vitale, IE22700259]

## Dominio e applicazione (max 150 parole)



## 1. Operazioni e workload
| ID  | Operazione                                                                    | Tipo (R/W) | Frequenza stimata (A/M/B) | Campi di filtro/ordinamento | Motivazione della stima (1-2 righe) |
| :-: | :---------------------------------------------------------------------------- | :--------: | :-----------------------: | :-------------------------- | :---------------------------------- |
|     |                                                                               |            |                           |                             |                                     |
| OP1 | Recuperare le migliori speedrun di un gioco                                   |     R      |             A             |                             |                                     |
| OP2 | Confrontare velocità di speedrun con score/recensioni (time to beat)          |     R      |             M             |                             |                                     |
| OP3 | Trovare gli utenti più attivi tra i giochi su cui si fanno Speedrun           |     R      |             B             |                             |                                     |
| OP4 | Trovare i giochi più apprezzati/criticati dal pubblico                        |     R      |            M/A            |                             |                                     |
| OP5 | Analizzare l'utenza dei giochi di nicchia                                     |     R      |             B             |                             |                                     |
| OP6 | Trovare i personaggi che appaiono in più giochi (Cameo giochi non principali) |     R      |             B             |                             |                                     |
| OP7 | Inserimento e aggiornamento di una nuova speedrun verificata                  |     W      |             M             |                             |                                     |

## 2. Modellazione (max 120 parole per relazione)

NOTA: we raga sono Omar questa è una nota da eliminare quando consegnamo, praticamente dobbiamo trovare le relazioni principali (o tutte le relazioni??? DA CHIEDERE ALLA PROF) del nostro modello, e poi dobbiamo scegliere un pattern da applicare tra questi 3:
1. embedding -> inserisci i dati collegati dentro lo stesso documento JSON. Velocizza le query, fai le READ direttamente senza join. Tuttavia abbiamo un vincolo di documenti max. 16 MB su MongoDB, quindi per le relazioni che prevedono unione di troppi dati bisogna fare attenzione
2. referencing -> i documenti in relazione sono separati, ma hanno una chiave esterna che li collega. Quindi, qui risparmi memoria per i singoli documenti, ma devi fare le Join quindi le query sono più lente
3. ibrido -> bellissimoooooo. Fai un po' embedding e un po' referencing, qua sembra la cache perché i dati che più ti interessano li embeddi (quindi quelli che usi di più nelle operazioni), e gli altri fai referencing (quelli che usi di meno, ad esempio le speedrun ormai obsolete diventano dati più rari)

((NON ABBIAMO RELAZIONI N:M AL MOMENTO, MA NE POSSIAMO TROVARE))

Relazione 1: Gioco - Speedrun
Cardinalità: 1-N --> le speedrun si riferiscono ad uno specifico gioco, il quale può avere N speedrun
Scelta: Ibrido --> Embedding delle Top 10 speedrun del gioco (OP1), tutte le altre speedrun di quel gioco sono conservate in un altro documento che fa da storico
Alternativa scartata (eventuale): Embedding -> considerato perché avrebbe velocizzato OP1 che è a frequenza alta, scartato perché abbiamo un limite di max. 16MB per documento
Operazioni che motivano la scelta: OP1, OP7
Rischio al crescere dei dati: _Unbounded Array Anti-pattern_: l'annidamento illimitato causerebbe il superamento del limite dei 16 MB per documento e il degrado della cache RAM.
Esempio:
-------------------------------------------
Relazione 2: Giocatore - Speedrun
Cardinalità: 1-N --> un giocatore può completare N speedrun, e una speedrun si riferisce al giocatore che l'ha completata
Scelta: Referencing --> la speedrun ha l'id del giocatore
Alternativa scartata (eventuale): Ibrido --> Avremmo potuto filtrare le speedrun del giocatore così da ottenere velocemente gli ultimi risultati totalizzati dal giocatore. Tuttavia, non è un'operazione che abbiamo considerato principale nel nostro workload
Operazioni che motivano la scelta: OP3
Rischio al crescere dei dati: Un giocatore molto attivo farebbe crescere l'array di run oltre i 16 MB; il Referencing frammenta le run in documenti indipendenti indicizzati.
Esempio: 
-------------------------------------------

Relazione 3: Gioco - Recensioni e Metriche
Cardinalità: 1-N --> un gioco ha N recensioni, la recensione è riferita ad un gioco
Scelta: Embedding --> giochi e recensioni incorporati nello stesso documento, così che sia immediato il carimento delle recensioni e le valutazioni del gioco interessato
Alternativa scartata (eventuale): Referencing --> opzione valida per quanto riguarda giochi molto popolari, ma sarebbe stato svantaggioso utilizzarlo poiché le join sarebbero state fin troppo costose per giochi con pochi dati
Operazioni che motivano la scelta: OP2, OP4, OP5, OP6
Rischio al crescere dei dati: il sotto-oggetto `metrics` ha occupazione di memoria fissa, mentre l'array `critic_reviews` viene limitato nello script ETL alle sole recensioni rilevanti (es. max 5 elementi per gioco), eliminando il rischio di sforare i 16 MB.
Esempio: 
-------------------------------------------



## 3. Validazione (max 100 parole)
Per ogni collezione, spiegare i vincoli imposti nel validatore e il motivo di ciascuno, ad esempio, quale errore nei dati previene. Indicare anche un vincolo che si è deciso di non imporre, e perché.
Esempio di inserimento rifiutato: [file] → [regola violata] → [messaggio d'errore]
Vincolo NON imposto e motivo:


## 4. CRUD (max 80 parole)
| Operazione | File | OP di riferimento (sezione 1) |
|------------|------|-------------------|
Coerenza dei dati duplicati:



## 5. Indici (max 100 parole per indice)
| Indice | OP servite | Stage prima → dopo | docsExamined prima → dopo | keysExamined | nReturned | ms prima → dopo | File explain |
|--------|------------|--------------------|---------------------------|--------------|-----------|-----------------|--------------|

Indici composti – ordine dei campi (se ESR):
Indice scartato (e perchè):


## 6. Query e aggregation
Per ogni pipeline: | Stage | Ruolo (1 riga) |
L'ordine degli stage influisce sulle prestazioni? (1 riga)


## 7. Riflessione critica e uso dell'AI (max 250 parole)
Parti in cui è stata usata l'AI:
Un suggerimento rifiutato e perché:
La decisione di cui siamo meno convinti:
