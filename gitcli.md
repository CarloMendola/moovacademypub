# GIT: una panoramica dalla command line
### + 400 pt

## GIT: il controllo di versione distribuito
### ~ 10 min
Git e' un sistema di controllo di versione distribuito organizzato a repository.
Ciascun repository e' presente in un server centrale (e.g. github, gitlab, ...), di solito in un formato chiamato **bare**, che non permette di visualizzare a filesystem i file versionati, ma che e' clonabile da chiunque ne faccia richiesta e abbia i permessi per accedere.
L'accesso ai repository centrale git puo' essere effettuato nei seguenti modi:
- interazione via file 
- interazione via ssh
- interazione via http/https
Dal momento che il repository centrale viene clonato, ciascuno dei cloni ha una versione completa del repository (di tutti i file in tutti i branch e tutte le versioni), e alla fine del proprio lavoro, il delta prodotto rispetto al repository di partenza deve essere pushato sul repository centrale, eventualmente riconciliando i conflitti che possono essersi creati.

Per approfondire le modalita' di creazione di un server git fare riferimento a [Git on the server](Git-on-the-Server-Getting-Git-on-a-Server) :fire:

## Setup Git Bash
### ~ 10 min
Prima di muovere i primi passi sulla commandline di Git e' necessario installare gli eseguibili di git.
- Per scaricare GIT fare riferimento alla pagina [Git Download](https://git-scm.com/downloads)
Se l'ambiente scelto per l'installazione e' Windows, il package di installazione per questo sistema operativo include anche la **git-bash**, un emulatore di terminale unix ove e' possibile eseguire git da riga di comando.

Per validare l'ambiente appena installato, aprire la gitbash ed eseguire il comando:
```git --version```

![](images/git-version.png)

## GIT: le operazioni piu' usate viste dalla command line
### ~ 30 min

#### Clone repository remoto
Per clonare un repository remoto e' necessario posizionarsi nella cartella di lavoro e eseguire dalla gitbash il comando:
``` git clone https://github.com/CarloMendola/git-example.git ```

![](images/git-clone.png)

Da questo momento, entrando nella cartella git-example si puo' operare sul repository locale.

#### Creazione branch locale e remoto
Per creare un branch e selezionarlo e' sufficiente eseguire il comando: 
``` git checkout -b my-new-branch ```

Come detto in precedenza, essendo git un software completamente distribuito, tutte le operazioni effettuate sul repository locale, sono disaccoppiate dal repository remoto.
Per tanto, se non esegui esplicitamente un operazione di push del branch locale appena creato, questo non esistera' mai sul repository remoto.

Per eseguire la push, si esegue il comando:
``` git push origin my-new-branch ```

Con la keyword **origin** si intende il riferimento remoto del server da cui il repository e' stato clonato.
Per approfondire sul significato di **origin**, fare riferimento a [Git Basics Working with Remotes](https://git-scm.com/book/en/v2/Git-Basics-Working-with-Remotes) :fire:

#### Aggiornamento metadati repository da remoto
Uno dei comandi che viene usato molto spesso dagli sviluppatori, a volte in maniera trasparente dagli IDE grafici come Eclipse, VisualStudio, ecc... e' il **fetch**, un operazione che consente di recuperare i metadati remoti del repository.
Il caso d'uso tipico per cui viene utilizzato questo comando, consiste nel gestire le modifiche parallele ad un repository (quando piu' persone lavorano sullo stesso repository ed effettuano **push** delle loro modifiche locali in remoto).
Eseguendo il fetch, e' possibile capire se la propria baseline (versione di partenza clonata dal repository centrale) e'diventata obsoleta e deve essere ribasata.
Eseguendo il comando fetch, e' possibile avere un idea delle operazioni necessarie per procedere.
``` git fetch --all ```

![](images/git-fetch-all.png)

In questo caso si puo' osservare che il fetch segnala una modifica remota, a questo punto l'utente puo' scegliere se ribasarsi subito o continuare il proprio lavoro posticipando il rebase. Per poter rientrare con le proprie modifiche sul server remoto, le modifiche locali devono includere tutte quelle introdotte da altri sviluppatori in remoto tramite rebase o merge.

#### Visualizzazione testuale delle versioni del repository
Per visualizzare tramite gitbash il version-tree completo del repository e' necessario eseguire il comando:
``` git log --all --graph --oneline ```

![](images/git-log-all1.png)

Dall'immagine si puo' vedere che in questo caso, il repository locale seleziona il branch my-new-branch (vedi puntatore **HEAD**), che pero' e' meno aggiornato di quanto disponibile sul repository remoto.

Per riallineare le modifiche locali a quelle remote si puo' procedere in 2 modi, tramite **Merge** o **Rebase**.

Ove possibile e' sempre meglio procedere tramite rebase del proprio branch locale (riscrittura del proprio branch locale a partire da una nuova baseline). Il beneficio di perseguire questa modalita', consiste nell'avere modo di provare le proprie modifiche dopo il rebase prima del push in remoto.

Per effettuare il rebase (in modalita' FastForward) si utilizza il comando:
``` git pull --rebase --ff-only origin my-new-branch ```

![](images/git-pull-rebase-ff.png)

Eseguendo nuovamente il comando per visualizzare il version-tree (git log) e' possibile osservare lo spostamento del puntatore **HEAD**

**NOTA:** il puntatore *HEAD* permette di selezionere (visualizzare) a filesystem una precisa versione (commit, branch, tag, ecc..) del repository. ::

![](images/git-log-all2.png)

#### Visualizzazione Differenze tra versioni
Per visualizzare tramite command line quali sono le differenze nei contenuti dei file tra 2 versioni, si puo' eseguire il comando:
``` git diff f366d2f fe824a7 ```

![](images/git-diff1.png)

Quella sopra e' solo una delle tante modalita' in cui si puo' interagire con git diff, per approfondire le altre modalita' fare riferimento a [Git Diff](https://git-scm.com/docs/git-diff) :fire:

#### Merge - due modalita' operative
TODO: ...
