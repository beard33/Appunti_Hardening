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

## Linux:
* AppArmor
* SELinux
* Aide 
* ...

## BSD:
* Tamper detection con AIDE (Advanced Intrusion Detection Environment)
* Jails (_FreeBSD_)

## Windows:
* Aggiornamenti
* Firewall
* No FTP/Telnet, si SSH/SFTP
* Controllo porte
* Syslog e Active Monitoring Tools
* Riduzione superficie (eliminazione registry entries, file, librerie e servizi non necessari)
* **Mandatory Integrity Control**
[](https://docs.microsoft.com/en-us/windows/desktop/SecAuthZ/mandatory-integrity-control)

## MacOs:
* Vedere file
* TrustedBSD
---
## DAC vs MAC
### DAC:
_Discretionary Access Control_, implementazione di Access Control basata sul concetto di **proprietario** del file in questione (un'implementazione esemplificativa è la gestione dei file e dei relativi permessi nei sistemi Unix, in cui ogni file presenta 3 bit di permessi (rwx) associati a UID, GUID e Others). Ogni proprietario di un file può decidere i permessi da garantire a determinati UID o GUID. Ad esempio può settare permessi rwx per l'utente A, permessi r only per l'utente B e permessi rw per gli utentei del gruppo C. Il suo punto di forza principale è la flessibilità, ed è per questo che è implementato di default su molti sistemi operativi. Ha delle forti limitazioni tuttavia:
* Non vi è controllo di coerenza tra le policies definite dall'utente e le policies globali di sistema
* L'accesso ad una copia potrebbe essere garantito anche se l'accesso al corrispettivo file originale non è garantito
* Le policies del DAC possono essere modificate dall'utente, pertanto un malware che stia girando come _user_ proprietario di un determinato file potrebbe modificarne i permessi DAC

### MAC:
_Mandatory Access Control_, non si basa più sul proprietario di un determinato file ma su policies di sistema decise sulla base di gerarchie di livelli di rischio. I due attori sono un **soggetto** che compie delle azioni su un **oggetto** (ognuno di questi ha attributi relativi alle autorizzazioni e al livello di sicurezza). Gestito da un _policy administrator_ e non dal proprietario (che non ha capacità di modifica delle policies). 
Due regole fondamentali del MAC sono definite dal modello [Bell-Lapadula](https://it.wikipedia.org/wiki/Modello_Bell-LaPadula)(contrapposto al modello BIBA che lavora sull'integrità):
* SS-Property: un soggetto può accedere ad un oggetto solo se il suo livello di sicurezza è maggiore od uguale a quello dell'oggetto (_No ReadUp_)
* S-Property: un soggetto può accedere ad un oggetto in append se ha un livello di sicurezza inferiore rispetto all'oggetto (_No WriteDown_)

### Linux Security Modules:
E' un framework che permette al kernel linux di supportare diverse implementazioni dei moduli di sicurezza (tra cui il controllo MAC), e sono standard dalla versione 2.6 del kernel. A differenza di sistemi come `systrace` che utilizzano una forma di interposizione nelle syscall (poco funzionale in un'ottica di scalabilità), LSM utilizza un sistema di [_hooks_](https://en.wikipedia.org/wiki/Hooking): sono tecniche e interfacce (dette "di hooking") che permettono di intercettare, modificare o fermare il comportamento di una determinata componente intercettando eventi, messaggi o parametri passati tra componenti, trasferendo di fatto il controllo da un processo ad un altro in modo trasparente al processo che perde il controllo dell'esecuzione. Le principali tecniche di hooking consistono nel modificare codice eseguibile a runtime per implementare un modulo e sposterne l'esecuzione, modificare il contenuto di una libreria runtime, utilizzare chiamate ad API definite e  In genere un hook può soltanto ridurre i privilegi di accesso alle risorse.
[](http://www.di-srv.unisa.it/~ads/corso-security/www/CORSO-0304/lsm/index.htm)
Nel caso dell'LSM ogni volta che avviene una syscall e si passa da _user space_ a _kernel space_ dopo che sono avvenuti i controlli sui dati e sulla loro integrità viene eseguito un primo controllo dal DAC. Successivamente a questo controllo entra in gioco l'hook dell'LSM che rimanda all'implementazione il controllo sulla possibilità di esecuzione della syscall. L'LSM restituisce una risposta affermativa/negativa in relazione alla possibilità di esecuzione; se la risposta è negativa viene bloccata la systemcall (**pg 9 libro selinux**). **NB**: Gli LSM di per se non sono uno strumento di sicurezza, ma un framework per l'implementazione di moduli quali SELinux o AA. 

###### Fonti:
* [DAC wikipedia](https://en.wikipedia.org/wiki/Discretionary_access_control)
* [Corso sicurezza Syracuse University](http://www.cis.syr.edu/~wedu/Teaching/cis643/LectureNotes_New/MAC.pdf)

---
## AppArmor

### Cos'è:
Implementazione del MAC introdotto da Novell e costruito sull'interfaccia LSM (Linux Security Modules). Indica una serie di regole (note come _profilo_) su ciascun programma ed esso dipende dal percorso di installazione/esecuzione del programma in esecuzione (al contrario di SELinux le norme non dipendono dall'utente e ogni utente deve sottostare allo stesso insieme di regole quando è in esecuzione lo stesso programma). Definisce cosa le risorse possono accedere e con che permessi e a differenza di SELinux che si basa su etichette poste ai file, AA lavora sul percorso dei file

### Come funziona:
I profili di AppArmor sono memorizzati in `etc/apparmor.d` e contengono un elenco delle regole di controllo d'accesso alla risorse che ogni programma può utilizzare. Attraverso [`apparmor_parser`](http://manpages.ubuntu.com/manpages/trusty/man8/apparmor_parser.8.html) i profili vengono compilati e caricati all'interno del kernel.
Ogni profilo può essere caricato sia in esecuzione(_enforcing_) che in _complaining_ mode: la prima fa rispettare la policy e registra i tentativi di violazione, la seconda non applica la policy ma registra sempre le chiamate di sistema che sarebbero state negate. Il report dei tentativi di elusione dei profili può avvenire sia attraverso syslog che attrverso 
#### Profili:
Sono file di testo leggibili dall'essere umano che descrivono come un binario debba essere trattato quando in esecuzione, con l'accesso alle risorse che è automaticamente rifiutato quando non ci sono riscontri all'interno delle regole di profilo

### Come si installa:
E' disponibile per tutte le distribuzioni GnuLinux (eccetto le RH che di default hanno SELinux praticamente hardened e difficilissimo da sostituire senza conflitti con AA), dipendentemente dalla pachettizzazione della distro può essere necessario scaricare solo i pacchetti `apparmor` `apparmor-profiles` e `apparmor-utils` (Debian) oppure separatamente anche `apparmor-parser` e `apparmor-libapparmor` (Arch)

### Come si configura:

        # aa-status

 permette di mostrare lo stato di AA, i profili caricati (totali e divisi per tipo di profilo).
Ogni profilo può essere portato dalla modalità enforcing alla modalità complain e viceversa attraverso i comandi

         # aa-complain /usr/bit/service-name
         # aa-enforcing /usr/bit/service-name

seguiti come parametro dal path dell'eseguibile o dal path del file di profilo. E' poi possibile disabilitare un profilo interamente attraverso il comando 

        # aa-disable 

sempre seguito dal path o porlo in _audit-mode_ (ovvero affinchè tenga traccia nel log di tutte le chiamate a sistema, anche quelle andate a bun fine) attraverso il comando `aa-audit`

_Esempi:_

        # aa-enforce /usr/bin/service-name

Pone il servizio selezionato in complain mode

### Creare un nuovo profilo:
I programmi più importanti da porre all'interno di un profilo AA sono soprattutto i programmi che interfacciano la rete, poichè sono i target più probabili di un attaccante remoto. AA fornisce il comando `aa-unconfined` che lista tutti i programmi che non hanno n profilo associato ed espongono una socket sulla rete. Con l'opzione aggiuntiva `--paranoid` si ottengono i processi non confinati che hanno almeno una connessione attiva sulla rete. 
Per la creazione di un profilo di utilizza il comando `# aa-genprof service-name` che come prima cosa, dopo aver controllato che non esista un profilo associato, chiederà di avviare in una finestra il servizio di interesse per la profilatura della system-calls.
Le possibilità all'esecuzione sono molteplici:
* Qualora venisse rilevata l'esecuzione di un altro programma si può eseguire il programma con il profilo del processo padre (`inherit`), eseguirlo con un profilo dedicato(`Profile` e `Named`) in cui la differenza è la possibilità di assegnare un nome arbitrario nel secondo caso, eseguirlo con un sottoprifilo del padre (`Child`), lasciarlo senza profilo (`Unconfined`) o di non eseguirlo (`Deny`).
Quando una chiamata di sistema richiede un permesso dell'utente root (definite _capacità_) AA controlla se il profilo consente al chiamante di fare uso di tale permesso.
**Astrazioni:** è possibile fare uso di astrazioni, le quali forniscono un insieme di regole riutilizzabili riunendo più risorse spesso utilizzate insieme

**NB** : nel caso di sandboxing di servizi di rete può risultare utile sostituire gli identificatori del profilo specifico con elementi genericci che permettano l'utilizzo anche in caso di cambio di interfaccia di rete (e.g. `/run/dhclient-eth0.pid` con `/run/ddclient*.pid` per fare in modo che il profilo sia inddipendente dall'interfaccia _eth0_ utilizzata durante la crazione).

_Nota_: `aa-genprof` non è altro che un layer sopra a `aa-logprof`: alla chiamata di genprof viene di fatto creato un profilo vuoto in complain mode e chiamato logprof per aggiornare il profilo.
E' importante, per avere un profilo preciso, utilizzare la risorsa interessata in tutti i modi possibili relativi alla stessa.

### Cheat sheet:
* Mostrare i profili
  

        # aa-status 


* Settare un determinato profilo in _complain mode_

        # aa-complain

* Modificare il tipo di profilo

        # aa-enforce 

* Mostrare i profili esistenti
  
        # ls /etc/apparmor.d/*

* Mostrare i profili disabilitati
  
        # ls /etc/apparmor.d/disable/*
        
* Layer di `logprof` per la generazione di un profilo
  
        # aa-genprof 

* Ricarica di un profilo

        # apparmor_parser -r /etc/apparmor.d/<profile>


###### Fonti:
* [Debian Handbook](https://debian-handbook.info/browse/it-IT/stable/sect.apparmor.html)
* [Arch wiki](https://wiki.archlinux.org/index.php/AppArmor)
* [Wiki ufficiale gitlab](https://gitlab.com/apparmor/apparmor/wikis/Documentation)
---


## SELinux
### Cos'è: 
Come Apparmor è un'implementazione del MAC che sfrutta il framework _Linux Security Modules_, ma a differenza di AA si basa su etichette ai file invece che ai percorsi di questi ultimi. Fu rilasciato per la prima volta nel dicembre 2000 a seguito di lavori dell'NSA (sviluppatrice iniziale del progetto) per dimostrare il valore dei controlli di accesso obbligatori in sistemi Linux (e più in generale Posix).
Da notare che come anche AA, SELinux non può limitare exploit legati a vulnerabilità degli applicativi (può invece evitare che un applicativo exploited si comporti scondo privilegi che non possiede).
A differenza di AA è un'implementazione [_Type Enforcement_](https://wiki.gentoo.org/wiki/SELinux/Type_enforcement), ovvero in cui l'accesso è governato da autorizzazioni basate sulla divisione comportamentale *subject-access-object*

### Come funziona:
SELinux non va a sovrascrivere o modificare le policy definite dal DAC (se un sistema senza SELinux previene un determinato accesso non c'è nulla che SELinux possa fare per sovrascrivere tale comportamento) poichè l'hook dell'LSM viene triggerato _dopo_ il check del DAC permission. Per questo motivo se ad esempio si necessita di garantire l'accesso ad un file ad un nuovo utente non sarà possibile farlo attraverso una policy di SEL ma sarà necessario implementare tale policy attraverso, ad esempio, l'utilizzo delle __access control lists__ tipiche dei sistemi posix.  (`setfacl` e `getfacl` permettono di gestire permessi aggiuntivi sui file e le directories). 
Dove SEL ha possibilità di azione è riguardo alle [**capabilities**](https://www.linuxjournal.com/article/5737)

Similmente ad AA, SELinux ha diverse modalità di funzionamento:
* `Disabled` -> totalmente disabilitato
* `Permissive` -> (paragonabile al `complaining` di AA), registra nei log azioni che sarebbero bloccate in Enforcing, ma al tempo stesso non blocca alcuna azione
* `Enforcing` -> applica in toto le policy bloccando le azioni dei subject che non le rispettano (paragonabile alla omonima di AA)

La decisione di SELinux riguardo al consentire o negare una determinata azione è basata su il **subject** (ovvero chi compie l'azione) e l'**object** (ovvero il terget dell'azione da compiere)

__nota__: SEL non ha alcuna conoscenza dell'owner del processo, di come questo è stato chiamato etc. Ciò che viene analizzato è il **contesto**, rappresentato da una _label_. Un contesto è formato da tre parti principali:
* User
* Role
* Type (quando un contesto si applica ad un processo si parla di "domain" invece che di "type")

_NOTA:_ i tre componenti sono soltanto nomi, è la policy a dar loro significato (di fatto se un oggetto ha lo stesso nome di una policy domain questo ha significato soltanto se è la policy a definire quest'ultimo)

Il context dello user corrente è visionabile attraverso il comando `id -Z`. 
La scelta di utilizzare le _label_ invece di seguire il modello di implementazione di AA in cui si segue il percorso dei binari deriva da più motivazioni:

* L'utilizzo del concetto di **contesto** permette di rendere l'intero processo indipendente dall'eventuale pathname, permettendo di mappare anche hw, porte di rete o pipe
* L'indipendenza permette inoltre l'introduzione di una forma di MLS (_Multi Level Security_), ovvero una gerarchia multilivello basata sul modello Bell-Lapadula (anch'esso basato su oggetti/soggetti e, in generale, su più livelli di gerarchia)

### Come si installa:
_Fortemente_ consigliata per distribuzioni che la supportino a pieno (ancora meglio su distribuzioni che nascono con SELinux; in tal senso le RH sono le più indicate, seguite da OSuse, che lo supporta a pieno, e Gentoo-Hardened, da preferire ad un adattamento di un'installazione classica di Gentoo). 
Su distribuzioni come Ubuntu è possibile installare SELinux attraverso il comando

        # apt install selinux

Ma solo a patto di terminare ed eliminare AA attraverso

        # /etc/init.d/apparmor stop
        # apt purge apparmor

### Come si configura:
È possibile vedere lo stato corrente di SELinux (disabled/permissive/enforcing attraverso il comando:

        # getenforce


Per avere informazioni più dettagliate riguardo allo stato del sistema si può usare:

        # sestatus

 Per modificare lo stato di funzionamento si utilizza il comando

        # setenforce enforcing
        # setenforce permissive

Il file di config principale si trova nella dir `/etc/selinux/config`

Nelle distribuzioni non native come Debian una volta installato SELinux è necessario settare il parametro `enforcing = 1` per evitare che di default SELinux venga avviato in modalità permissive e modificare il file di config del bootloader (nel caso di GRUB in `/etc/grub.conf`) e settare i flag di attivazione e modalità_enforcing a 1

### Come scrivere una policy:
Le policy si dividono in :
* _Strict Policy_ : una policy che cerca di gestire tutte le attività con SELinux, pertanto ogni oggetto esiste all'interno di un contesto di SElinux ed è controllato da quest'ultimo in ogni azione
* _Targeted Policy_ : una policy in cui soltanto determinate applicazioni (o in generale elementi del sistema, e.g. le porte di rete) sono sottoposte ad una policy, mentre tutto ciò che non è all'interno del target non rientra nel controllo SELinux

Le policy sono compilate in userspace, la principale è caricata al boot dall'init di sistema e i vari moduli di controllo dei contesti possono essere aggiunti/rimossi in ogni momento.

#### Sintassi delle policy:
* **Allow** (et similia): 
  - Per garantire il permesso al processo del dominio(type) `Source` sugli oggetti di tipo `Target` e classe `Class`
                
                allow Source Target:Class Permission
  - Per avere un comportamento come `allow` ma con audit nei log si utilizza
                
                auditallow Source Target:Class Permission
  - Per non garantire il permesso e non avere alcun log si utilizza `dontaudit`
  - Per fare in modo che il compiler generi un errore se i permessi specificati sono    garantiti da altre rules (da usare come controllo extra verso errori nelle policy)   si utilizza `neverallow`
  N.B = Ogni oggetto creato ha un sua contesto di default

  - E' possibile anche eseguire un _Type Transition_, ovvero fare in modo che "ogni oggetto della classe Class creato da un processo del dominio Source e che di default avrebbe il tipo Target, ottiene invece il tipo _New Type_"
    (E' importante notare il fatto che non si tratta di una rule, quanto più di una direttiva di azione a SELinux)

               type_transition Source Target:Class new_type; 
* **ROLE**  
    - E' possibile definire una regola del tipo "e' possibile per un contesto di un processo con la regola Role di trovarsi nel dominio Type" attraverso la seguente dichiarazione (il tentativo di ingresso in un dominio con una Role non autorizzata porterebbe ad un `Invalid Context Error` ) :

                 role ROLE types TYPE;

* **DECLARATIONS**             
    - E' possibile dichiarare un _tipo_ con nome _identifier_, a cui è anche possibile aggiungere una _attributelist_ facoltativa

### Moduli
#### Cos'è un modulo:
Un modulo è una serie di dichiarazioni da "iniettare" nel kernel; può essere caricato/scaricato liberamente e solitamente copre le security rules di una certa appllicazione. Una semplice struttura di un m odulo potrebbe essere:
* Definire un _tipo_ per l'eseguibile
* Definire un _tipo_ per il dominio nel quale l'applicazione gira
* Poichè l'applicazione gira in un modulo non definito altrove, ogni accesso ad oggetti esistenti è vietato di default
* Si lancia l'applicazione mentre il sistema è in `Permissive Mode` (gli accessi negati verranno loggati)
* Si utilizza `audit2allow` per creare regole che matchino con i messaggi di log
* Si aggiustano le rules dove necessario
(Se da un lato questo approccio ha il vantaggio di essere molto immediato, dall'altra è applicabile solo alle azioni che l'applicazione ha compiuto durante l'apertura)
#### Creare un modulo
* Viene creata una dir temporanea dove lavorare e si crea un link simbolico al makefile di Selinux ` ln -s /usr/share/selinux/devel/Makefile ` (si trova all'interno del development-package)
* Si prepara un modulo iniziale creando un file con estensione `.te`
* Si apre una shell con privilegi di root e si seguono i log

         tail -f /var/log/audit/audit.log | \ grep -E '^type=(AVC|SELINUX\_ERR)'
(I messaggi _AVC_ avvengono quando viene negato un permesso, i _SELinux\_Err_ riguardano invece tentativi di violare regole e user restrictions)

* **Esempio di un modulo**: 

        module haifux 1.0.0;

        require {
        type unconfined_t;
        class process { transition sigchld };
        class file { read x_file_perms };
        }        

La prima riga dichiara il nome e la versione del modulo, la clausola `require` indica _tipi_ e _permessi_ che il modulo si aspetta siano già esistenti e definiti
* Se dovessimo usare un tipo senza definirlo o richiederlo avremmo un errore del tipo: `haifux.te":24:ERROR 'unknown type haifux_exec_t' at token ';' on line 1028 `
* D'altro canto se dovessimo richiedere una classe non definita il caricamento del modulo fallirebbe: ` libsepol.print_missing_requirements: haifux's global requirements were not met: type/attribute haifux_t `

Continuando con il modulo di prima:


        module haifux 1.0.0;

        require {
        type unconfined_t;
        class process transition;
        }

        type haifux_t;
        type haifux_exec_t;

        role unconfined_r types haifux_t;

        type_transition unconfined_t haifux_exec_t : process haifux_t;

In cui vengono definiti due nuovi tipi, `haifux_t` e `haifux_exec_t`. Specifica inoltre che se un processo si trova in `unconfined_t` e fa girare un eseguibile il cui dominio è `haifux_exec_t`, questo deve continuare la sua esecuzione nel dominio `haifux_t`

Per compilare il modulo è necessario lanciare `make` nella working directory; il modulo binario ha estensione `.pp`
Per caricare il modulo è necessario usare il comando:

        # make load
Per scaricarlo

        # make clean

### Cheat sheet:
* **Basics:**
  - `disabled` SELinux è disabilitato
  - `permissive` logs senza blocchi
  - `enforcing` logs e blocchi
  - LOG: sono salvati in `var/log/audit/audit.log`
* **Contexts**
  - ***Users***:
    - `unconfined_u` utente non protetto
    - `system_u` utente di sistema
    - `user_u` utente standard
  - ***Role***:
    - `object_r` file
    - `system_r` utenti e processi
    - `unconfined_r` utenti e processi non protetti
  - ***Domini***:
    - `unconfined_t` file o prcesso non protetto
    - ...
* **Comandi Utili**:
  - `sestatus` mostra lo stato attuale
  - `getenforce` mostra l'attuale stato enforcing
  - `setenforce` cambia temporaneamente il livello di enforcing
  - `ls -Z` mostra i contesti di sicurezza dei file
  - `ps -Z atd` mostra i contesti di sicurezza del processo atd
  - `sestatus -b` ritorna tutte le opzioni booleane per i servizi
  - `ausarch -ts time | grep AVC (or denied)` lista tutte le violazioni o gli accessi negati a partire da _time_ (nel caso delle date dipende tutto dal `locale`)

##### Fonti:
* [Wikipedia](https://it.wikipedia.org/wiki/Security-Enhanced_Linux)
* [Vermulen - SELinux System Administration (II ed. 2017)]()
* [Writing a targeted SELinux policy](http://www.billauer.co.il/selinux-policy-module-howto.html#SECTION00023000000000000000)
  
