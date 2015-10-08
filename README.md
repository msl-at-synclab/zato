# Zato

Appunti su Zato.

## Introduzione

Zato è un _Enterprise Service Bus_. Permette di collegare fra loro diversi servizi aziendali pre-esistenti e con protocolli anche eterogenei. Si tratta inoltre di una piattaforma per la realizzazione ex-novo di servizi in linguaggio Python. Maggiori informazioni sullo scopo ed il contesto Zato [si possono trovare qui](https://zato.io/docs/intro/esb-soa.html).

## Architettura

Zato è basato sul concetto di cluster. Ogni cluster eroga un servizio utilizzando un certo numero di server ridondati (stesso codice) e acceduti tramite un load balancer. In ogni cluster, inoltre, esattamente un server svolge anche il ruolo di task scheduler.

Ai fini di configurazione e raccolta statistiche, ciascun cluster include anche un DBMS SQL e un data store [Redis](http://redis.io/).

Viene supportata anche la comunicazione asincrona mediante code, in particolare attraverso dei connettori compatibili con alcune delle tecnologie principali per le code (es. [AMQP](https://www.amqp.org/)) e l’uso di Redis come message broker.

[Questo articolo](https://zato.io/docs/intro/overview-tech.html) è un’ottima introduzione all’architettura di Zato e contiene link ad articoli altrettanto utili sulle singole componenti.

## Installazione

Zato [si può scaricare da qui](https://zato.io/downloads.html). Ho testato personalmente la procedura indicata per Ubuntu e non ci sono stati problemi di sorta. L'unica nota è che per usare i comandi di Zato bisogna loggarsi come utente `zato`:

``` bash
sudo su - zato
```

Da notare che Redis non è impacchettato insieme a Zato, quindi bisogna scaricarlo e installarlo a parte. Su Ubuntu la [procedura ufficiale](http://redis.io/download) è il download e compilazione del sorgente. Oltre alle istruzioni
ufficiali è utile aggiungere la cartella di Redis alla variabile `PATH`:

``` bash
echo 'export PATH=$PATH:/path/to/redis' >> ~/.bashrc
source ~/.bashrc
```

## Tutorial

Sulla documentazione ufficiale si può trovare [un tutorial piuttosto dettagliato](https://zato.io/docs/tutorial/01.html) e con degli utili riferimenti al resto della documentazione.

Seguendo questo tutorial si possono incontrare alcune difficoltà. Di seguito viene spiegato come risolvere i
problemi che ciascuno step potrebbe porre.

### Creare un cluster

Il tutorial invita a creare il cluster con il comando `zato
quickstart create`. Rispetto al tutorial, io l'ho lanciato in questo modo:

``` bash
zato quickstart create ~/tutorial sqlite 127.0.0.1 6379 --verbose
```

In questo modo ho creato un cluster nella cartella `tutorial` della home dell'utente `zato`
e ho indicato i parametri di connessione a Redis (a differenza del tutorial non ho indicato una password, visto che di default Redis non fa sicurezza).

### Impostare una password per Redis

Alla fine, anche se non si passa una password a `zato
quickstart create`, ci penserà il comando stesso a chiederla interattivamente. Perciò _bisogna_ configurare Redis per richiedere la stessa password:

``` bash
redis-cli
config set requirepass "mia indecifrabilissima password"
```

**Nota:** Questi comandi invece vanno dati dall'utente "regolare" del sistema, non dall'utete `zato`.

**Nota:** Ufficialmente la configurazione di Redis si può fare anche tramite il file `redis.conf`, ma poi al riavvio viene ignorata. Sto investigando su come risolvere la cosa.

### Verificare la configurazione del cluster

Il tutorial dice di usare il comando `zato check-config` seguito dal path del server da verificare. Nel mio caso ho dato:

``` bash
zato check-config ~/tutorial/server1
zato check-config ~/tutorial/server2
```

La prima volta che ci ho provato sono usciti fuori degli errori dovuti di default Redis non usa password mentre Zato vuole a tutti i costi passargliene una. [Vedere la sezione
precedente](#impostare-una-password-per-redis) per risolvere questo problema.

### Lanciare il cluster

Il tutorial fa riferimento ad uno script `zato-qs-start.sh`, senza dire dove trovarlo. Nel mio caso era in `~/tutorial`.
