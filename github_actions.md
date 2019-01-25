# Github Actions

## Che è
Le action di github sono una nuova feature che permette di organizzare dei workflow composti da diversi task. Ogni task è un'istanza di docker basata su un'immagine presa da dockerhub che esegue degli script creati da noi.

## Come funziona
Github ci fornisce sia un editor grafico che testuale per organizzare i nostri workflow. Per iniziare un workflow ci serve un qualsiasi hook per agganciarci ad un evento. Github ce ne fornisce diversi e di ogni tipo, ad esempio:

- push su un branch
- commento su una issue/pull request
- label aggiunta ad una issue
- ecc..

Ad ogni hook possiamo collegare una serie di azioni basate su un dockerfile che esegue uno o più script.

Per ogni task può essere specificata una dipendenza che potrà essere "soddisfatta" utilizzando l'opzione `resolve` nella creazione del nostro task. Utilizzando questa funzionalità potremo interrompere, riprendere o selzionare(if-else) i nostri workflow.

## Limitazioni
Purtroppo i workflow hanno delle limitazioni:
  - Ogni worflow può essere eseguito per 58 minuti includendo l'accodamento e il tempo di esecuzione
  - Ogni workflow può lanciare massimo 100 azioni
  - Possono essere eseguiti massimo due workflow simultaneamente per ogni progetto
  - I task non possono essere ricorsivi(ad esempio i commit generati dai task non rilanciano workflow legati all'hook di un commit).
  - I log del container sono limitati a 64kb

## Nascondere le 🔑🗝
Dalla gui delle action è possibile configurare fino a 100 secrets da 65kb l'uno. In caso un secret sia più grande di 65kb si può salvare criptato nel progetto e salvare nelle keys la chiave per decriptarlo.

## Come creare una action
Basta creare un repo con un `Dockerfile`, l'esempio riportato sulla documentazione è abbastanza semplice:

```
FROM debian:9.5-slim

LABEL "com.github.actions.name"="Hello World"
LABEL "com.github.actions.description"="Write arguments to the standard output"
LABEL "com.github.actions.icon"="mic"
LABEL "com.github.actions.color"="purple"

LABEL "repository"="http://github.com/octocat/hello-world"
LABEL "homepage"="http://github.com/actions"
LABEL "maintainer"="Octocat <octocat@github.com>"

ADD entrypoint.sh /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
```

Ora puoi creare il tuo file entrypoint. Cambiando le immagini puoi utilizzare qualsiasi linguaggio.
