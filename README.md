# Simulatore Concettuale di Catene di Markov per la Traduzione

Questo progetto è un **simulatore concettuale** (o modello esemplificativo) di un'interazione utente-macchina basata su una selezione casuale e traduzione di frasi. Sebbene il codice non implementi una vera Catena di Markov nel senso stretto della probabilità di transizione tra stati, il **flusso logico** può essere astratto per descrivere il passaggio da uno "stato" all'altro (dalla selezione della frase all'output tradotto) – un modello semplificato che chiameremo "Catena di Markov di Base" per scopi didattici.

-----

## 1\. Il Caso Studio: "La Catena di Frasi"

Immaginiamo un sistema dove lo "stato" è rappresentato dalla **frase inglese** attualmente selezionata. Il sistema può transitare da uno stato all'altro in modo *casuale* (la selezione) e poi procedere verso lo **stato finale** (la traduzione in italiano).

| Stato Iniziale | Azione (Transizione) | Stato Intermedio | Azione Finale (Transizione) | Stato Finale |
| :---: | :---: | :---: | :---: | :---: |
| Set di Frasi Inglesi | Selezione Casuale | Frase Inglese Singola | Richiesta API di Traduzione | Frase Italiana Tradotta |

Il **motore** del nostro simulatore è il `suggestBtn` che innesca questa transizione.

-----

## 2\. Componenti del Codice

Il codice fornito gestisce la selezione casuale di una frase da un array e la sua successiva traduzione tramite un servizio API esterno.

### 2.1. L'Array degli Stati Iniziali (`englishPhrases`)

```javascript
const englishPhrases = [
  "The sun shines over the hills",
  "A cat sleeps near the window",
  "The wind moves the trees slowly",
  // ... altre frasi ...
];
```

  * **Ruolo:** Rappresenta l'insieme dei possibili **Stati Iniziali** nel nostro modello concettuale di Markov. Ogni elemento è una frase inglese completa e distinta.
  * **Dettaglio:** L'array è l'**alfabeto** delle frasi sorgente del simulatore.

### 2.2. L'Ascoltatore di Eventi (Il "Motore di Transizione")

```javascript
suggestBtn.addEventListener("click", async () => { /* ... logica ... */ });
```

  * **Ruolo:** È l'**Innesco** che avvia la transizione. Quando il bottone (`suggestBtn`) viene cliccato, si esegue il blocco di codice asincrono, passando allo stato successivo.

-----

## 3\. Flusso Dettagliato del Processo (La Catena Passo-Passo)

La funzione associata all'evento `click` è il cuore del processo e si svolge in tre fasi principali: **Selezione**, **Transizione API** e **Output**.

### Fase 1: Selezione Casuale (Transizione $S_{Iniziale} \rightarrow S_{Selezionato}$)

```javascript
const phrase = englishPhrases[Math.floor(Math.random() * englishPhrases.length)];
```

  * **Spiegazione:** Viene generato un **indice casuale** (`Math.random() * englishPhrases.length`) e troncato a un intero (`Math.floor`). Questo indice seleziona una frase specifica dall'array `englishPhrases`.
  * **Modello Markov:** Questa è la prima transizione. La probabilità di selezionare una qualsiasi frase è **uniforme** ($1/n$, dove $n$ è il numero totale di frasi), simulando una transizione casuale verso un unico stato (`phrase`) dal set di stati iniziali.

### Fase 2: Richiesta e Gestione API (Transizione $S_{Selezionato} \rightarrow S_{Tradotto}$)

```javascript
try {
  const res = await fetch(
    "https://api.mymemory.translated.net/get?q=" + encodeURIComponent(phrase) + "&langpair=en|it"
  );
  const data = await res.json();
  const trad = data.responseData.translatedText;
  input.value = trad;
} catch (e) {
  input.value = "Non riesco a contattare il servizio di traduzione.";
}
```

  * **Costruzione URL:** La frase selezionata (`phrase`) viene codificata (`encodeURIComponent`) e inserita in un URL per l'API di traduzione gratuita `mymemory.translated.net`. Il parametro `langpair=en|it` specifica la traduzione da **Inglese** a **Italiano**.
  * **Fetch Asincrono:** Viene effettuata la richiesta (`fetch`). L'uso di `await` mette in pausa l'esecuzione fino a quando la risposta (`res`) non arriva.
  * **Parsing JSON:** La risposta viene convertita da testo a oggetto JavaScript (`data = await res.json()`).
  * **Estrazione:** La traduzione effettiva viene estratta dal percorso `data.responseData.translatedText` e salvata in `trad`.
  * **Gestione Errori (`catch`):** Se la richiesta fallisce (es. problema di rete, API non raggiungibile), l'esecuzione salta al blocco `catch`, e il campo di input riceve un messaggio di errore.
  * **Modello Markov:** Questa è la seconda transizione. In un modello ideale, il successo di questa transizione potrebbe essere visto come una **probabilità di successo dell'API** ($\text{P}_{\text{successo}}$) o una probabilità di fallimento ($\text{P}_{\text{fallimento}}$) a causa di errori di rete, portando a due possibili stati finali: *Traduzione Riuscita* o *Traduzione Fallita*.

### Fase 3: Output (Stato Finale)

```javascript
input.value = trad;
// oppure
input.value = "Non riesco a contattare il servizio di traduzione.";
```

  * **Ruolo:** Il risultato (`trad` o il messaggio di errore) viene assegnato alla proprietà `value` del campo di input (`input`).
  * **Stato Finale:** Questo rappresenta lo **stato finale** del simulatore: l'utente visualizza un testo, che è l'obiettivo ultimo della catena.

-----

## 4\. Requisiti e Limitazioni

### Requisiti

  * **HTML:** Presuppone l'esistenza di un bottone con ID `suggestBtn` (al quale è associato l'evento) e un campo di input (es. `<input>`) identificato come `input` (variabili non definite nel frammento ma necessarie per il funzionamento).
  * **Connessione a Internet:** Richiesta per poter contattare il servizio API esterno.

### Limitazioni

  * **API Esterna:** Il codice dipende totalmente dalla disponibilità e affidabilità dell'API `mymemory.translated.net`.
  * **Non una Vera Catena di Markov:** Il modello è solo concettuale. Per una vera implementazione, sarebbero necessarie matrici di probabilità che definiscono la probabilità di transizione tra **ogni coppia di stati** nell'array `englishPhrases`.
  * 
