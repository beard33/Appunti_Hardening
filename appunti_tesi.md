# L'HARDENING
Per **hardening** si intende l'insieme di operazioni di configurazione e gestione di un sistema informatico per minimizzare la possibilità di attacchi e i danni da essi derivanti. Diviso principalmente in due categorie:
* _One Time Hardening_ : eseguito soltanto una volta (solitamente alla messa in opera del sistema)
* _Multiple Time Hardening_ : eseguito più volte durante il ciclo di vita di un sistema, solitamente a fronte del rilascio di patch aggiuntive per vulnerabilità 0-day o per l'aggiunta di moduli supplementari a quelli esistenti.

### Attività di hardening
L'hardening non è una procedura univoca che porta alla messa in sicurezza di un sistema, ma una serie di operazioni variabili e talvolta complementari tra le quali:
* Mantenere il sistema aggiornato
* Installare un firewall e controllare le porte di rete
* Installare software antivirus 
* Mantenere dei backup del sistema
* Usare containers o macchine virtuali
* ... 
---
## HARDENING E SISTEMI OPERATIVI
L'hardening dipende anche dalla famiglia di sistemi operativi che si vuole rendere più sicura; esistono infatti diverse risorse o checklist da eseguire dipendentemente dal sistema operativo di riferimento:

### Linux:
* AppArmor
* SELinux
* Moduli GRSecurity
* TomoyoLinux
* FirewallD
* Aide 
* ...

### BSD:
* Tamper detection con AIDE (Advanced Intrusion Detection Environment)
* Jails (_FreeBSD_)

### Windows:
* Aggiornamenti
* Firewall
* No FTP/Telnet, si SSH/SFTP
* Controllo porte
* Syslog e Active Monitoring Tools
* Riduzione superficie (eliminazione registry entries, file, librerie e servizi non necessari)
* **Mandatory Integrity Control**
[](https://docs.microsoft.com/en-us/windows/desktop/SecAuthZ/mandatory-integrity-control)

### MacOs:
* Vedere file
* TrustedBSD
---
### DAC vs MAC
#### DAC:
_Discretionary Access Control_, implementazione di Access Control basata sul concetto di **proprietario** del file in questione (un'implementazione esemplificativa è la gestione dei file e dei relativi permessi nei sistemi Unix, in cui ogni file presenta 3 bit di permessi (rwx) associati a UID, GUID e Others). Ogni proprietario di un file può decidere i permessi da garantire a determinati UID o GUID. Ad esempio può settare permessi rwx per l'utente A, permessi r only per l'utente B e permessi rw per gli utentei del gruppo C. Il suo punto di forza principale è la flessibilità, ed è per questo che è implementato di default su molti sistemi operativi. Ha delle forti limitazioni tuttavia:
* Non vi è controllo di coerenza tra le policies definite dall'utente e le policies globali di sistema
* L'accesso ad una copia potrebbe essere garantito anche se l'accesso al corrispettivo file originale non è garantito
* Le policies del DAC possono essere modificate dall'utente, pertanto un malware che stia girando come _user_ proprietario di un determinato file potrebbe modificarne i permessi DAC

#### MAC:
_Mandatory Access Control_, non si basa più sul proprietario di un determinato file ma su policies di sistema decise sulla base di gerarchie di livelli di rischio. I due attori sono un **soggetto** che compie delle azioni su un **oggetto** (ognuno di questi ha attributi relativi alle autorizzazioni e al livello di sicurezza). Gestito da un _policy administrator_ e non dal proprietario (che non ha capacità di modifica delle policies). 
Due regole fondamentali del MAC sono definite dal modello [Bell-Lapadula](https://it.wikipedia.org/wiki/Modello_Bell-LaPadula)(contrapposto al modello BIBA che lavora sull'integrità):
* SS-Property: un soggetto può accedere ad un oggetto solo se il suo livello di sicurezza è maggiore od uguale a quello dell'oggetto (_No ReadUp_)
* S-Property: un soggetto può accedere ad un oggetto in append se ha un livello di sicurezza inferiore rispetto all'oggetto (_No WriteDown_)

##### Linux Security Modules
E' un framework che permette al kernel linux di supportare diverse implementazioni dei moduli di sicurezza (tra cui il controllo MAC), e sono standard dalla versione 2.6 del kernel. A differenza di sistemi come `systrace` che utilizzano una forma di interposizione nelle syscall (poco funzionale in un'ottica di scalabilità), LSM utilizza un sistema di [_hooks_](https://en.wikipedia.org/wiki/Hooking): sono tecniche e interfacce (dette "di hooking") che permettono di intercettare, modificare o fermare il comportamento di una determinata componente intercettando eventi, messaggi o parametri passati tra componenti, trasferendo di fatto il controllo da un processo ad un altro in modo trasparente al processo che perde il controllo dell'esecuzione. Le principali tecniche di hooking consistono nel modificare codice eseguibile a runtime per implementare un modulo e sposterne l'esecuzione, modificare il contenuto di una libreria runtime, utilizzare chiamate ad API definite e  In genere un hook può soltanto ridurre i privilegi di accesso alle risorse.
[](http://www.di-srv.unisa.it/~ads/corso-security/www/CORSO-0304/lsm/index.htm)
Nel caso dell'LSM ogni volta che avviene una syscall e si passa da _user space_ a _kernel space_ dopo che sono avvenuti i controlli sui dati e sulla loro integrità viene eseguito un primo controllo dal DAC. Successivamente a questo controllo entra in gioco l'hook dell'LSM che rimanda all'implementazione il controllo sulla possibilità di esecuzione della syscall. L'LSM restituisce una risposta affermativa/negativa in relazione alla possibilità di esecuzione; se la risposta è negativa viene bloccata la systemcall (**pg 9 libro selinux**). **NB**: Gli LSM di per se non sono uno strumento di sicurezza, ma un framework per l'implementazione di moduli quali SELinux o AA. 

###### Fonti:
* [DAC wikipedia](https://en.wikipedia.org/wiki/Discretionary_access_control)
* [Corso sicurezza Syracuse University](http://www.cis.syr.edu/~wedu/Teaching/cis643/LectureNotes_New/MAC.pdf)

---
## AppArmor

#### Cos'è:
Implementazione del MAC introdotto da Novell e costruito sull'interfaccia LSM (Linux Security Modules). Indica una serie di regole (note come _profilo_) su ciascun programma ed esso dipende dal percorso di installazione/esecuzione del programma in esecuzione (al contrario di SELinux le norme non dipendono dall'utente e ogni utente deve sottostare allo stesso insieme di regole quando è in esecuzione lo stesso programma). Definisce cosa le risorse possono accedere e con che permessi e a differenza di SELinux che si basa su etichette poste ai file, AA lavora sul percorso dei file

#### Come funziona:
I profili di AppArmor sono memorizzati in `etc/apparmor.d` e contengono un elenco delle regole di controllo d'accesso alla risorse che ogni programma può utilizzare. Attraverso [`apparmor_parser`](http://manpages.ubuntu.com/manpages/trusty/man8/apparmor_parser.8.html) i profili vengono compilati e caricati all'interno del kernel.
Ogni profilo può essere caricato sia in esecuzione(_enforcing_) che in _complaining_ mode: la prima fa rispettare la policy e registra i tentativi di violazione, la seconda non applica la policy ma registra sempre le chiamate di sistema che sarebbero state negate. Il report dei tentativi di elusione dei profili può avvenire sia attraverso syslog che attrverso 
##### Profili:
Sono file di testo leggibili dall'essere umano che descrivono come un binario debba essere trattato quando in esecuzione, con l'accesso alle risorse che è automaticamente rifiutato quando non ci sono riscontri all'interno delle regole di profilo

#### Come si installa:
E' disponibile per tutte le distribuzioni GnuLinux (eccetto le RH che di default hanno SELinux praticamente hardened e difficilissimo da sostituire senza conflitti con AA), dipendentemente dalla pachettizzazione della distro può essere necessario scaricare solo i pacchetti `apparmor` `apparmor-profiles` e `apparmor-utils` (Debian) oppure separatamente anche `apparmor-parser` e `apparmor-libapparmor` (Arch)

#### Come si configura:
`# aa-status` permette di mostrare lo stato di AA, i profili caricati (totali e divisi per tipo di profilo).
Ogni profilo può essere portato dalla modalità enforcing alla modalità complain e viceversa attraverso i comandi `aa-enforcing` `aa-complain` seguiti come parametro dal path dell'eseguibile o dal path del file di profilo. E' poi possibile disabilitare un profilo interamente attraverso il comando `aa-disable` sempre seguito dal path o porlo in _audit-mode_ (ovvero affinchè tenga traccia nel log di tutte le chiamate a sistema, anche quelle andate a bun fine) attraverso il comando `aa-audit`
_Esempi:_
`# aa-enforce /usr/bin/service-name` pone il servizio selezionato in complain mode

##### Creare un nuovo profilo:
I programmi più importanti da porre all'interno di un profilo AA sono soprattutto i programmi che interfacciano la rete, poichè sono i target più probabili di un attaccante remoto. AA fornisce il comando `aa-unconfined` che lista tutti i programmi che non hanno n profilo associato ed espongono una socket sulla rete. Con l'opzione aggiuntiva `--paranoid` si ottengono i processi non confinati che hanno almeno una connessione attiva sulla rete. 
Per la creazione di un profilo di utilizza il comando `# aa-genprof service-name` che come prima cosa, dopo aver controllato che non esista un profilo associato, chiederà di avviare in una finestra il servizio di interesse per la profilatura della system-calls.
Le possibilità all'esecuzione sono molteplici:
* Qualora venisse rilevata l'esecuzione di un altro programma si può eseguire il programma con il profilo del processo padre (`inherit`), eseguirlo con un profilo dedicato(`Profile` e `Named`) in cui la differenza è la possibilità di assegnare un nome arbitrario nel secondo caso, eseguirlo con un sottoprifilo del padre (`Child`), lasciarlo senza profilo (`Unconfined`) o di non eseguirlo (`Deny`).
Quando una chiamata di sistema richiede un permesso dell'utente root (definite _capacità_) AA controlla se il profilo consente al chiamante di fare uso di tale permesso.
**Astrazioni:** è possibile fare uso di astrazioni, le quali forniscono un insieme di regole riutilizzabili riunendo più risorse spesso utilizzate insieme

**NB** : nel caso di sandboxing di servizi di rete può risultare utile sostituire gli identificatori del profilo specifico con elementi genericci che permettano l'utilizzo anche in caso di cambio di interfaccia di rete (e.g. `/run/dhclient-eth0.pid` con `/run/ddclient*.pid` per fare in modo che il profilo sia inddipendente dall'interfaccia _eth0_ utilizzata durante la crazione).

_Nota_: `aa-genprof` non è altro che un layer sopra a `aa-logprof`: alla chiamata di genprof viene di fatto creato un profilo vuoto in complain mode e chiamato logprof per aggiornare il profilo.
E' importante, per avere un profilo preciso, utilizzare la risorsa interessata in tutti i modi possibili relativi alla stessa.

#### Cheat sheet:
* `# aa-status` // mostra i profili attivi
* `# aa-complain`
* `# aa-enforce` // permettono la modifica del tipo di profilo
* `# ls /etc/apparmor.d/*` // mostra i profili esistenti
* `# ls /etc/apparmor.d/disable/*` // mostra i profili disabilitati
* `# aa-genprof` // layer di logprof per la generazione di un profilo


###### Fonti:
* [Debian Handbook](https://debian-handbook.info/browse/it-IT/stable/sect.apparmor.html)
* [Arch wiki](https://wiki.archlinux.org/index.php/AppArmor)
* [Wiki ufficiale gitlab](https://gitlab.com/apparmor/apparmor/wikis/Documentation)
---


### SELinux
#### Cos'è: 
Come Apparmor è un'implementazione del MAC che sfrutta il framework _Linux Security Modules_, ma a differenza di AA si basa su etichette ai file invece che ai percorsi di questi ultimi. Fu rilasciato per la prima volta nel dicembre 2000 a seguito di lavori dell'NSA (sviluppatrice iniziale del progetto) per dimostrare il valore dei controlli di accesso obbligatori in sistemi Linux (e più in generale Posix).
Da notare che come anche AA, SELinux non può limitare exploit legati a vulnerabilità degli applicativi (può invece evitare che un applicativo exploited si comporti scondo privilegi che non possiede)
#### Come funziona:
SELinux non va a sovrascrivere o modificare le policy definite dal DAC (se un sistema senza SELinux previene un determinato accesso non c'è nulla che SELinux possa fare per sovrascrivere tale comportamento) poichè l'hook dell'LSM viene triggerato _dopo_ il check del DAC permission. Per questo motivo se ad esempio si necessita di garantire l'accesso ad un file ad un nuovo utente non sarà possibile farlo attraverso una policy di SEL ma sarà necessario implementare tale policy attraverso, ad esempio, l'utilizzo delle __access control lists__ tipiche dei sistemi posix.  (`setfacl` e `getfacl` permettono di gestire permessi aggiuntivi sui file e le directories). 
Dove SEL ha possibilità di azione è riguardo alle [**capabilities**](https://www.linuxjournal.com/article/5737)

La decisione di SELinux riguardo al consentire o negare una determinata azione è basata su il **subject** (ovvero chi compie l'azione) e l'**object** (ovvero il terget dell'azione da compiere)

__nota__: SEL non ha alcuna conoscenza dell'owner del processo, di come questo è stato chiamato etc. Ciò che viene analizzato è il contesto, rappresentato da una _label_
#### Come si installa:
#### Come si configura:
#### Differenze con AA:
#### Cheat sheet:
###### Fonti:
* [Wikipedia](https://it.wikipedia.org/wiki/Security-Enhanced_Linux)
