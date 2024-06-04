---
title: 'Francesco Petrarca'
description: 'Francesco Petrarca summary'
pubDate: 'Jan 08 2022'
author: 'gkkconan'
category: 'study-notes'
subject: 'systems and networks'
tags: ['systems', 'assembly', 'architectures', 'indirizzamento', 'cpu', 'intel', 'x86', 'informatica']
---


# Table of Contents
- [Architettura della CPU](#architettura-della-cpu)
- [Ciclo Macchina](#ciclo-macchina)
- [Pipelining](#pipelining)
- [Set di istruzioni macchina: CISC e RISC](#set-di-istruzioni-macchina-cisc-e-risc)
- [Assembly Intel x86](#assembly-intel-x86)
- [Metodi di indirizzamento Intel x86](#metodi-di-indirizzamento-intel-x86)

---

## Architettura della CPU

### La CPU
La CPU è l'elemento fondamentale di ogni calcolatore/computer, e anche la colonna portante di tutto il modello di Von-Neumann. Essa è può essere **multicore**, ovvero composta da più core (ALU + CU + registri).

Ora visioniamo ogni componente interno di questo microprocessore nel dettaglio:
- **CU** (Control Unit): coordina e gestisce le operazioni interne dei core in base agli input
- **ALU** (Arithmetic Logic Unit): svolge tutte le operazioni logiche e aritmetiche che richiede la CU
- **Registri**: piccole aree di memoria che servono per immagazzinare i dati temporaneamente per eseguire correttamente le operazioni durante l'esecuzione di istruzioni
- **CACHE interna**: aree di memoria in cui ci sono le istruzioni successive a quella in esecuzione per velocizzare le operazioni. possono essere di diverso livello (L1, L2, L3...) in base alla posizione e di conseguenza velocità di accesso e frequenza di utilizzo
- **LCU** (Logic Control Unit): circuiti che convertono impulsi elettrici in segnali per la CU e viceversa (segnali dalla CU verso l'esterno)
- **MI** (Memory Interface): circuiti che forniscono ai bus dati e indirizzi gli impulsi per comunicare con la memoria e con le periferiche, e viceversa (segnali dall'esterno alla CU)
- **BUS interno** (CPU bus): collegamenti elettrici che permettono la comunicazione tra i vari componenti della CPU


### I BUS
Il system bus è composto da altri tre tipi di bus:
- **control bus**: sono trasferiti i comandi (es. read, write...) dalla CPU alle periferiche e le richieste delle periferiche alla CPU (IRQ)
- **data bus**: sono trasferiti i dati
- **address bus**: sono trasferiti gli indirizzi

Il control bus è costituito da segnali logici. Alcuni segnali sono trasmessi dalla CPU alle periferiche, e altri dalle periferiche alla CPU. Il suo ruolo fondamentale è di sincronizzare i trasferimenti su tutti i bus esterni in base al clock di sistema.

L'address bus è costituito da linee su cui la CPU scrive l'indirizzo di cella della memoria in cui vuole accedere (che sia in lettura o in scrittura).

Il data bus è costituito da linee su cui la CPU o la memoria scrivono i dati che devono essere trasferiti. In particolare le direzioni possono essere:
- dalla CPU alla memoria: la CPU scrive sul bus e la memoria legge
- dalla memoria alla CPU: la memoria scrive sul bus e la CPU legge

In alcuni processori, in alternativa, esiste anche un unico bus temporizzato che "unisce" address e data bus.

Solitamente è presente anche una corrispondenza tra la dimensione dei bus e quella dei registri. Ad esempio, un'architettura a 64 bit, ha sia registri della CPU che bus a 64 bit.
Però una CPU può anche avere registri e bus interno a 64 bit ma avere address e data bus a dimensioni differenti.
In quel caso, il numero di bit indicano i bit che la CPU può elaborare contemporaneamente.

*architettura hardware (cpu, bus e registri) ≠ architettura software (os)*

Un computer con architettura effettivamente a 64 bit deve avere CPU, bus e OS tutti a 64 bit.
Però un computer può comunque funzionare anche con le seguenti combinazioni, tuttavia l'ultima è la più performante:
• CPU a 32 bit e OS a 32 bit
• CPU a 64 bit e OS a 32 bit
• CPU a 64 bit e OS a 64 bit

Anche i software devono essere compatibili con l'OS. Per esempio, i programmi a 32 bit sono adatti agli OS a 32 bit, così anche come per i programmi e gli OS a 64 bit.
Tuttavia, la maggior parte dei software a 32 bit funzionano anche sugli OS a 64 bit, ma si potrebbero riscontrare problemi di driver, con dei tools o con gli antivirus.


### I registri
Un registro è una piccola e veloce memoria interna alla CPU.
Negli anni sono anche aumentati in numero e dimensioni, sempre secondo potenze di 2 (8 bit -> 16 bit -> 32 bit -> 64 bit).

Esistono dei registri fondamentali in tutte CPU, seppur differiscono in base ai nomi dati dai produttori.
[Schema architettura CPU generica con registri fondamentali](../../assets/SCHOOL/SISTEMI/01.png)

Di seguito la spiegazione più dettagliata per ogni registro:
- **AR** (Address Register): memorizza gli indirizzi per gli accessi in memoria (solo da CPU verso memoria)
- **DR** (Data Register): memorizza i dati dalla memoria alla CPU e dalla CPU alla memoria
- **IR** (Instruction Register): memorizza l'istruzione e gli operandi che si sta eseguendo
- **PC** (Program Counter): l'indirizzo della prossima istruzione da eseguire
- **SR** (Status Register): memorizza tramite dei flag lo stato della CPU dopo l'ultima operazione eseguita
- **SP** (Stack Pointer): memorizza l'indirizzo in cima allo stack per salvare l'indirizzo di ritorno dalle subroutine (funzioni o procedure) chiamate dal main program. L'ultima subroutine chiamata è la prima a terminare l'esecuzione e per tornare al main program serve l'indirizzo di ritorno, salvato nel SP
- **R0, R1, R2, ..., Rn** (Registri di lavoro, o Registri generali): memorizzano dei risultati temporanei I/O dall'ALU

Oltre a tutti questi registri, la CPU può comprendere anche altri registri per contenere altri dati. Ad esempio, potrebbero esserci altri registri per contenere degli indirizzi iniziali (base register), dei contatori (count register), degli indici (index register) o dei puntatori (pointer register).

I 3 registri più importanti che tengono conto dell'esecuzioni sono:
- **SR** (Status Register)                --> PASSATO
- **IR** (Instruction Register)          --> PRESENTE
- **PC** (Program Counter)             --> FUTURO



## Ciclo Macchina
Il ruolo della CPU è di eseguire un programma (ovvero una sequenza di istruzioni).
Per svolgere il suo obiettivo, la CPU deve prelevare le istruzioni dalla memoria, caricare tutto nei registri, e successivamente l'ALU elabora l'istruzione e restituisce un risultato (in un registro o in memoria).

Tutte queste operazioni che svolge la CPU per eseguire un programma rappresentano il ciclo macchina, chiamato anche **fetch-execute cycle**.
Di seguito uno schema che riassume in modo esemplare tutto il ciclo macchina: [Schema a blocchi ciclo macchina](../../assets/SCHOOL/SISTEMI/02.png)

Riassumendo, le fasi che possiamo schematizzare da tutto il processo sono 5:
- **IF** (Instruction Fetch): lettura dell'istruzione dalla memoria
- **ID** (Instruction Decode): decodifica dell'istruzione e lettura dei dati dai registri
- **EX** (Execution): esecuzione dell'istruzione
- **MEM** (Memory): scrittura del dato in memoria (se necessario)
- **WB** (Write Back): scrittura del risultato nei registri e aggiornamento dello stato

Le CPU più recenti suddividono queste fasi in micro-operazioni con la stessa complessità, in modo da ottimizzare il processo, eseguendo ogni micro-operazione in un ciclo di clock.
La durata di una micro-operazione è definita **cycle time**, ed è indicata con T di CPU.
1/T di CPU è la frequenza di clock della CPU, misurata in *gigahertz* (GHz).

Esempio: [Ciclo macchina sequenziale](../../assets/SCHOOL/SISTEMI/03.png)
	Immaginiamo tre istruzioni in sequenza: ogni intervallo dell'asse del tempo rappresenta T di CPU, che per semplicità facciamo coincidere con la durata di una fase.
	Se ogni fase dura T di CPU che equivale a 0,5ns, serviranno 7,5ns per eseguire tutte le istruzioni.
	Il calcolo sarà il seguente: 3 istruzioni * 5 fasi * 0,5ns

Nei prossimi sotto-capitoli verranno spiegate nel dettaglio tutte le fasi del ciclo macchina, includendo i registri coinvolti nelle rispettive operazioni.

Per fissare meglio i concetti, di seguito un flowchart riguardante tutti i passi principali del ciclo e dei registri coinvolti: [Flowchart Ciclo Macchina](../../assets/SCHOOL/SISTEMI/04.png)


### IF
registri coinvolti: PC, AR, IR, DR
- L'indirizzo della prima istruzione viene caricato nel Program Counter (PC), poi viene letta e trasferita nell'Address Register (AR)
- L'istruzione è prelevata dalla memoria dall'Address bus (fetch) e caricata nell'Instruction Register + eventuali dati nel Data Register della CPU
- Il Program Counter viene incrementato per puntare alla successiva istruzione da eseguire


### ID
registri coinvolti: registri di lavoro
Scopo: Il processore deve capire cosa significano le istruzioni che riceve.
- La CU legge l'Instruction Register, decodifica l'istruzione, carica i dati nei registri di lavoro e definisce i campi dell'istruzione


### EX
registri coinvolti: registri di lavoro e ALU
- La ALU esegue tutti i calcoli aritmetico-logici, preparati in precedenza, con l'ausilio dei registri di lavoro per memorizzare eventuali risultati parziali
- Nel caso in cui non ci siano calcoli da eseguire, quindi non sarebbe necessario utilizzare la ALU, questa fase non viene saltata, piuttosto il processore sta fermo per un giro di clock


### MEM
registri coinvolti: registri di lavoro e memoria
- I dati vengono salvati in memoria, quindi i dati passano dai registri alla memoria


### WB
registri coinvolti: Data Register (facoltativo), Status Register
Scopo: fare un report di quello che è accaduto, usando dei flag contenuti nello SR
- Il risultato dell'esecuzione dell'istruzione viene scritto nel Data Register e lo stato della CPU viene settato nello Status Register



## Pipelining
Il pipelining è un concetto di progettazione hardware e una tecnica che permette di elaborare più istruzioni in parallelo.

In questo modo, le micro-operazioni, viste in precedenza, sono raggruppate in **circuiti mutuamente esclusivi della CPU** specializzati per eseguire velocemente micro-operazioni specifiche. Così facendo, nella CPU si possono elaborare contemporaneamente più micro-operazioni in parti differenti.

Ogni sequenza di elaborazioni indipendenti forma una pipeline (che letteralmente significa "conduttura" o "tubatura").
In pratica più circuiti mutuamente esclusivi eseguono operazioni in parallelo su dati differenti.

Esempio:
	Supponiamo di avere 5 circuiti mutuamente esclusivi e abbiamo la necessità di elaborare un programma di 8 istruzioni.
	In questo modo, dopo 5 fasi eseguite, avremo un periodo (dal 5 all'8) in cui tutte le unità funzionali lavorano contemporaneamente realizzando delle pipeline perfettamente funzionanti.
	Di seguito uno schema che sintetizza questo esempio per comprendere meglio: [Pipelining Ciclo Macchina](../../assets/SCHOOL/SISTEMI/05.png)
	Come possiamo notare, tutto il programma dura 12 fasi. Non utilizzando la tecnica delle pipeline sarebbe durato 40 fasi (5 fasi * 8 istruzioni). Quindi ovviamente questa tecnica è molto utile e ottimizza di molto tutto il processo di esecuzione dei programmi.

Il pipelining quindi si realizza sostituendo ad un unica unità funzionale (complessa e monolitica), più unità più piccole e semplici per ciascun gruppo di micro-operazioni, dette *stadi delle pipeline*.



## Set di istruzioni macchina: CISC e RISC

### Il linguaggio macchina e le architetture
Il linguaggio macchina è l'insieme delle istruzioni macchina che la CPU riesce comprendere ed eseguire.
Ogni istruzione macchina è una sequenza di bit formattata/divisa in campi. Ogni campo ha un certo numero di bit che contengono dati diversi.

Di seguito un'esempio di un'istruzione macchina in un'architettura a 16 bit formattata con 5 campi: [Esempio codice macchina formattato](../../assets/SCHOOL/SISTEMI/06.png)

Il formato dei dati può cambiare in base ad architettura e istruzione, però obbligatoriamente deve avere i campi per:
- gli opcode (istruzioni da svolgere)
- operandi coinvolti (+ rispettivo indirizzo)

Solitamente le CPU moderne di qualsiasi produttore sono retrocompatibili, quindi le istruzioni per le CPU meno recenti girano anche su CPU recenti (anche eseguendole in minor tempo, però per avere una miglior ottimizzazione è necessario utilizzare istruzioni più recenti).

In base al numero di operandi che una CPU può elaborare in un'istruzione esistono diverse architetture:
• architettura a un operando 
• architettura a due operandi
• architettura a tre operandi


#### Architettura a un operando
| opcode | operando |
| :----- | :------- |

Il campo dell'operando può contenere o l'operando direttamente o l'indirizzo in cui si trova.
- *vantaggi*: molti bit per operando o per il suo indirizzo
- *svantaggi*: serve predisporre precedentemente un operando in un registro di lavoro e solo dopo aver eseguito tutte le operazioni coni i dati in quel registro il risultato viene trasferito nella sua destinazione finale


#### Architettura a due operandi
| opcode | operando1/risultato | operando2 |
| :----- | :------------------ | :-------- |

L'operando1/risultato all'inizio contiene il primo operando o il suo indirizzo e alla fine il risultato dell'operazione.
- *vantaggi*:
	- molti bit per operando o per il suo indirizzo
	- non sono necessari registri d'appoggio se si devono svolgere operazioni con due operandi
- *svantaggi*: bisogna salvare il primo operando se si vuole riutilizzarlo successivamente


#### Architettura a tre operandi
| opcode | risultato | operando1 | operando2 |
| :----- | :-------- | :-------- | :-------- |

In questa architettura il terzo operando è il risultato, quindi non essendo in comune con uno degli operandi.
- *vantaggi*: programmazione più semplice (poiché non è necessario alcun spostamento dei dati per le operazioni)
- *svantaggi*: numero di bit per la codifica degli indirizzi di ogni campo limitato

La scelta dell'architettura è fondamentale per le prestazioni finali della CPU poiché dipende dall'insieme di istruzioni (Instruction Set Architecture: **ISA**), dalle modalità di accesso ai dati e alle istruzioni.


### Architetture CISC e RISC
Esistono principalmente due architetture per i processori:
- **CISC** (Complex Instruction Set Computer)
	- Tante istruzioni complesse (ognuna per ogni operazione possibile)
	- Consuma di più
	- Tempi di esecuzione più lunghi
	- La fase di decodifica è complessa e avviene attraverso un firmware nella CU la cui presenza che occupa spazio e aumenta il TDP (Thermal Design Power)
	- Prevalentemente a due operandi
	- Più usati per il desktop e per i server
- **RISC** (Reduced Instruction Set Computer)
	- Poche istruzioni indispensabili
	- Tempi di esecuzione migliori
	- Firmware per la decodifica più semplice e veloce
	- Le operazioni complesse vengono scomposte in operazioni minori e più semplici
	- Prevalentemente a tre operandi
	- Più usati per il mobile

Attualmente qualche processore elabora internamente istruzioni CISC e le trasforma in RISC prima di eseguirle.
Per verificare le prestazioni di un'architettura, solitamente vengono utilizzati i **benchmarks**, ovvero programmi con lo scopo di misurare vari parametri (es. MIPS) durante l'esecuzione di alcune operazioni (es. grafico, gestionale, render).


#### Risc e Pipelining
Le architetture RISC, essendo progettate per avere un'ISA ridotto e svolgendo operazioni più semplici, risultano più adatte per le pipelining (permettendo di scomporre meglio le fasi).



## Assembly Intel x86

### Il linguaggio assembly
Il codice macchina (o codice binario) può essere convertito in un codice mnemonico (linguaggio assembly), quindi molto più facilmente comprensibile.
Esso presenta un livello di astrazione software un gradino più elevata rispetto al codice binario.

Esempio:
```plaintext
codice macchina: 1010 0101 0011 1010 (linguaggio macchina)
codice mnemonico: MOV R0, R1 (linguaggio assembly)
```

L'insieme delle istruzioni di tutti questi codici mnemonici è chiamato **linguaggio assembly**.
Esso prevede, come ogni linguaggio, l'uso di parole chiave, sintassi e semantica.
Ad esempio, per assegnare dei valori a dei registri possiamo usare *MOV*, altrimenti *ADD* per sommare i contenuti di registri o di aree di memoria, o ancora *SUB* per sottrarre...

Il processo che traduce il codice assembly in codice macchina è composto da due fasi:
- **Assembler**: traduce ogni parola mnemonica nel corrispondente codice macchina
- **Linker**: collega moduli e librerie

Esiste anche il processo inverso, ovvero il **disassembly** (o disassemblaggio).

Di seguito uno schema che riassume perfettamente tutto il processo di creazione dell'eseguibile del programma, ovvero traduzione in codice macchina: [Traduzione da codice assembly a codice macchina](../../assets/SCHOOL/SISTEMI/07.png)

Il linguaggio assembly prevede istruzioni a zero, uno, due e tre operandi.

Il componimento di un'istruzione generica per processori Intel x86 sarà il seguente:
- **label**: etichetta opzionale per assegnare un nome all'indirizzo in memoria dove si trova l'istruzione
- **opcode** (codice operativo): codice mnemonico obbligatorio per specificare l'operazione da svolgere
- **operandi** (zero, uno, due o tre): separati da una virgola, specificano i dati su cui operare

| label | opcode | operando1 | operando2 |
| :---- | :----- | :-------- | :-------- |


### Registri processori Intel x86
Il processore Intel x86 prevede 5 tipi di registri (14 totali) a 16 bit:
- **registri di flag** (registri di stato): FR
- **registri di lavoro** (registri generali): AX, BX, CX, DX
- **registri indice**: DI, SI
- **registri di segmento**: DS, ES, CS, SS
- **registri puntatore**: IP, BP, SP

#### Flag Register
Tutte le istruzioni modificano il Flag Register (chiamato anche registro di stato, o FR), e l'esecuzione di alcune istruzioni dipende anche dal contenuto dei flag in questo registro.

I flag di questo registro sono 9 (su 16 bit) e servono per salvare lo stato della CPU dopo l'esecuzione di un'istruzione.
Di seguito sono riportate tutte le posizioni di tutti del registro: [Flag Register](../../assets/SCHOOL/SISTEMI/08.png)

Questi flag possono essere ulteriormente suddivisi in 6 flag di stato e 3 flag di controllo.

Flag di stato:
- **CF** (Carry Flag): viene settato ad 1 quando c'è un riporto nell'operazione precedente
- **PF** (Parity Flag): pari ad uno se il numero di bit è dispari così con quell'uno diventa pari (serve a far diventare pari il numero di uno dello Status Register, se il numero invece è già pari è settato su 0)
- **AF** (Auxiliary Flag): usato per il carry e per il borrow (prestito) nelle operazioni
- **ZF** (Zero Flag): quando l'ultima operazione ha dato risultato 0 viene settato su 1
- **SF** (Sign Flag): quando l'ultima operazione ha dato risultato negativo viene settato su 1
- **OF** (Overflow Flag): quando è presente un'overflow (dato di dimensioni superiori a quelle predisposte) viene settato su 1

Flag di controllo:
- **TF** (Trap Flag): settato a 1 quando si vuole eseguire un programma in debug mode, quindi la CPU cede il controllo all'utente dopo ogni istruzione
- **IF** (Interrupt Flag): settato a 1 se si vogliono prevedere le IRQ
- **DF** (Direction Flag): settato a 1 per indirizzi di memoria decrescenti, per i crescenti è settato a 0

#### Registri di lavoro
i Registri di lavoro sono i registri utilizzati dalla ALU per contenere gli operandi.
Nell'Intel x86 i registri di lavoro sono a 16 bit e sono 4:
- **AX** (Accumulator)
- **BX** (Base)
- **CX** (Count)
- **DX** (Data)

Questi registri a 16 bit possono essere anche usati dividendoli in due parti, ognuna da 8bit: parte alta (più significativa) e parte bassa (meno significativa):
- **AX** -> AH, AL
- **BX** -> BH, BL
- **CX** -> CH, CL
- **DX** -> DH, DL

Ecco un'immagine che spiega la struttura in modo molto semplice e intuitivo: [Suddivisione registri di lavoro](../../assets/SCHOOL/SISTEMI/10.png)

#### Registri segmento
Per ogni programma in esecuzione vengono assegnati 4 segmenti (ovvero aree della memoria da 64kB) che servono a contenere dati, istruzioni e indirizzi.
Quindi allo stesso tempo servono 4 registri che contengono gli indirizzi di questi segmenti:
- **DS** (Data Segment): contiene l'indirizzo in cui iniziano i dati del programma
- **ES** (Extra Segment): contiene l'indirizzo in cui iniziano i dati che necessitano più spazio (es. floating point)
- **CS** (Code Segment): contiene l'indirizzo dell'area riservata al codice
- **SS** (Stack Segment): contiene gli indirizzi di ritorno delle subroutine

*segmenti ≠ registri segmento*

Riassumendo, i registri indice contengono l'indirizzo per puntare ai segmenti. Ecco un'immagine che esprime meglio il concetto: [Registri segmento che puntano ai segmenti](../../assets/SCHOOL/SISTEMI/11.png)

#### Registri indice
Per utilizzare tutte le strutture dati (array o string), la CPU utilizza due registri indice per contenere l'indice dell'elemento che si sta leggendo o scrivendo:
- **SI** (Source Index): contiene indice dell'elemento della struttura che si sta leggendo
- **DI** (Destination Index): contiene indice dell'elemento della struttura che si sta scrivendo
Visto che i dati si trovano nel DS, questi registri lavorano legati a DS.
Quindi in DS troviamo l'indirizzo di inizio di tutti i dati e in SI e DI troviamo l'offset per leggere e scrivere i dati. 

#### Registri puntatore
Il principale registro puntatore è **IP** (Instruction Pointer) che serve a puntare all'istruzione corrente. In poche parole, l'IP "tiene il segno" dell'operazione a cui è arrivato.

Anche in assembly è possibile utilizzare delle subroutine (funzioni), che consentono di raggruppare delle funzioni sotto un nome prettamente simbolico per chiamarle ogni volta che servono.
Per salvare gli indirizzi di ritorno al main, dalle subroutine, serve uno stack.
Questo indirizzo di ritorno viene calcolato grazie ai seguenti registri:
- **BP** (Base Pointer): inizio area memoria riservata nello SS
- **SP** (Stack Pointer): cima area memoria riservata nello SS



##  Metodi di indirizzamento Intel x86

### Indirizzamento
Un fattore importante per la progettazione di un'architettura è la modalità di accesso a un dato, un'istruzione o un registro.
In base ai metodo di indirizzamento le istruzioni possono avere formati diversi.

Per accedere ai dati nella memoria serve utilizzare tre concetti chiave:
- **Base address**: indirizzo dal quale iniziano dati e istruzioni del programma da elaborare
- **Offset**: distanza da sommare al base address per puntare all'istruzione o al dato da elaborare. Alcune architetture contengono l'offset nei registri indice (SI e DI)
- **Displacement**: spostamento aggiuntivo a base address + offset. In alcune architetture si esplicita direttamente nell'istruzione macchina 

*base address + offset + displacement = effective address*

In questa rappresentazione, ogni riga di codice macchina è di 16 bit, ovvero 4 cifre esadecimali: [Rappresentazione effective address](../../assets/SCHOOL/SISTEMI/12.png)

L'istruzione a cui ci possiamo affidare per trasferire dati da una sorgente ad una destinazione è MOV, con la seguente sintassi:
``` asm
MOV destination, source
```

Questa istruzione è a due operandi poiché sposta dei dati da una sorgente ad una destinazione, essi sono separati dalla virgola e possono essere indirizzi di memoria o altri registri.
Volendo tradurre il significato in termini più comprensibili per i comuni mortali, sarebbe l'equivalente italiano per "copia nella destination il dato che sta in source".

I metodi di indirizzamento si suddividono in 3 tipi:
- **immediato**: il dato sta nell'istruzione
- **diretto**: c'è il registro o l'indirizzo della memoria in cui si trova il dato
- **indiretto**: c'è il registro o l'indirizzo della memoria in cui si trova l'indirizzo della memoria dove si trova il dato. Come fosse un doppio indirizzamento. In conclusione il dato può essere solo in memoria e non nei registri o nell'istruzione.

Per i processori Intel x86 esistono principalmente 7 modalità di indirizzamento egregiamente riassunte in questo schema: [Sette principali metodi di indirizzamento Intel x86](../../assets/SCHOOL/SISTEMI/13.png)


### Indirizzamento immediato
Come detto nel sotto-capitolo introduttivo precedente, nell'indirizzamento immediato, il dato è direttamente specificato nell'istruzione.

Esempio: [Rappresentazione grafica indirizzamento immediato](../../assets/SCHOOL/SISTEMI/14.png)
```asm
MOV AL, 4Dh
```
In questo piccolo esempio stiamo semplicemente scrivendo il valore "4D" (in esadecimale, per questo "h") nel registro AL, ovvero nella parte bassa del registro AX (Accumulator).
Questa è un'istruzione abbastanza semplice che non utilizza base, offset o displacement.


### Indirizzamento diretto
#### Da registro a registro
In questo indirizzamento, entrambi gli operandi sono registri.

Esempio: [Indirizzamento diretto da registro a registro](../../assets/SCHOOL/SISTEMI/15.png)
``` assembly
MOV AX, BX
```
Questa istruzione semplicemente copia i dati contenuti all'interno del registro BX nel registro AX.
Anche questa istruzione è molto banale e non utilizza base, offset o displacement.

#### Da registro a memoria e viceversa
In questo indirizzamento un operando è un registro e l'altro è un indirizzo di memoria (specificato attraverso l'offset, quindi tra parentesi quadre).

Esempio: [Indirizzamento diretto da memoria a registro](../../assets/SCHOOL/SISTEMI/16.png)
```asm
MOV AX, [6Ah]
```
Quest'istruzione prevede, ancor prima dell'operazione, il calcolo dell'indirizzo di memoria in cui si trova il dato.
Questo calcolo può avvenire, come spiegato nei sotto-capitoli precedenti, secondo questa struttura:
	*base address + offset = effective address*
Solo dopo aver calcolato l'indirizzo di memoria si può passare alla copia del dato dalla memoria (base address + 6A) al registro (AX).

Si nota che invertendo gli operandi, il risultato dell'operazione sarebbe l'inverso, ovvero si otterrebbe la copia del dato dal registro (AX) nella memoria (base address + 6A). L'istruzione quindi sarebbe stata la seguente:
```asm
MOV [6Ah], AX
```


### Indirizzamento indiretto
#### Tramite registro
Nel momento in cui l'architettura della CPU prevede dei registri indice (SI e DI) o dei registri puntatore (IP), è possibile indirizzare i dati anche tramite questi registri.

Esempio: [Indirizzamento indiretto tramite registro](../../assets/SCHOOL/SISTEMI/17.png)
```asm
MOV AX, [SI]
```
Come negli indirizzamenti diretti da registri a memoria e viceversa, anche in questa tipologia di indirizzamento è necessario prima calcolare l'indirizzo in cui si trova il dato attraverso:
	*base address + source index = effective address*
In questo caso l'offset non è specificato nell'istruzione ma si trova in SI.
Solo successivamente, si può procedere a copiare il dato dalla memoria (base address + SI che sarebbe l'offset) nel registro AX.

Invertendo gli operandi e usando il Destination Index (DI) al posto di SI, il contenuto di AX verrebbe copiato nella memoria con l'offset contenuto in DI. Quindi l'istruzione sarebbe la seguente:
```asm
MOV [DI], AX
```

#### Tramite registro e displacement
Questo tipo di indirizzamento prevede che l'indirizzo della memoria in cui si trova il dato si ottenga spostando di un valore aggiuntivo (displacement) all'offset.

Esempio: [Indirizzamento indiretto tramite registro e displacement](../../assets/SCHOOL/SISTEMI/18.png)
``` asm
MOV AX, 03[SI]
```
Anche in questa operazione è previsto prima il calcolo dell'indirizzo effettivo, attraverso questo calcolo:
	*base address + source index + displacement = effective address*
In cui offset corrisponde al Source Index (SI), e il displacement corrisponde a 03.
In seguito possiamo concludere copiando il dato dall'indirizzo effettivo (base address + SI + 03) nel registro AX.

Invertendo gli operandi (e usando DI al posto di SI), l'istruzione ottenuta copierà il dato dal registro AX in memoria (base address + offset di DI + displacement di 03):
```asm
MOV 03[DI], AX
```