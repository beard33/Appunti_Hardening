# L'HARDENING
Per **hardening** si intende l'insieme di operazioni di configurazione e gestione di un sistema informatico per minimizzare la possibilità di attacchi e i danni da essi derivanti. Diviso principalmente in due categorie:
* _One Time Hardening_ : eseguito soltanto una volta (solitamente alla messa in opera del sistema)
* _Multiple Time Hardening_ : eseguito più volte durante il ciclo di vita di un sistema, solitamente a fronte del rilascio di patch aggiuntive per vulnerabilità 0-day o per l'aggiunta di moduli supplementari a quelli esistenti.

### Attività di hardening
L'hardening non è una procedura univoca che porta alla messa in sicurezza di un sistema, ma una serie di operazioni variabili e talvolta complementari tra le quali:
* Mantenere il sistema aggiornato
* Installare un firewall e controllare le porte di rete
* Installare software antivirus 
* 
* Mantenere dei backup del sistema
* Usare containers o macchine virtuali
* ... 

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

### CAPABILITIES:
Nei sistemi unix le due tipologie di privilegi sono _User_ e _Root_: la prima non ha possibilità di interazione con nessun file che non rientri nelle sue proprietà, la seconda ha accesso e modifica ad ogni elemento del sistema. L'implementazione di una via di mezzo spesso è richiesta, e intervengono le **Capabilities**. Queste dividono l'accesso al sistema in gruppi logici che possono essere aggiunti o rimossi a/da determinati processi. Una lista delle capabilities di default sono `/usr/include/linux/capabilities.h`
La global capability definisce cosa un processo può o non può fare e se una capability arriva direttamente dal sistema non può essere sovrascritta nemmeno dall'utente root

### DISK ENCRYPTION:
Un'ulteriore misura di protezione per il raggiungimento di un buon livello di hardening del sistema èl'utilizzo della crittografia come strumento di protezione del disco e dell'utilizzo di partizioni separate (in modo da permettere un migliore isolamento dei dati e facilitando le operazioni di recovery in caso di errori)

### AIDE:
È un tool di intrusion detection che attraverso la creazione di un database (che altro non è se non uno snapshot delle parti di filesystem selezionate) contenente file e directory per eseguire manualmente (o automaticamente se impostato insieme a `cron`) 
</br>
Il file di configurazione si trova in `etc/aide.conf`. E' possibile utilizzare regole predefinite (`PERMS` per il controllo degli accessi, basandosi su utenti, gruppi, contesti SELinux e attributi file. `CONTENT` per il controllo di contenuto e tipo di file, `DATAONLY` per la detection di cambiamenti in dati all'interno di file e directories)</br>
Attraverso il comando `aide --check` è possibile eseguire un controllo di integrità secondo i parametri specificati. In caso di modifiche è necessario aggiornare lo snapshot di aide attraverso il comando `aide --update`. 

### LOG-CHECK:
È un programma che automatizza la lettura dei log e permette all'amministratore di sistema di leggerli ricevendo una mail. Di default lavora come un cronjob settato ogni ora e ad ogni riavvio. </br>
Supporta 3 livelli di contollo:
* _paranoid_: pensato per macchine ad alt rischio che eseguono pochissimi servizi, estremamente verboso
* _server:_ impostazione di default, contiene impostazioni per la maggior parte dei demoni di sistema
* _workstation:_ filtra la maggior parte dei messaggi, il meno verboso dei 3


###### Fonti:
* [DAC wikipedia](https://en.wikipedia.org/wiki/Discretionary_access_control)
* [Corso sicurezza Syracuse University](http://www.cis.syr.edu/~wedu/Teaching/cis643/LectureNotes_New/MAC.pdf)
* [Capabilities](https://www.linuxjournal.com/article/5737)
* [Capabilities 2](https://linux-audit.com/linux-capabilities-101/)
* [Logcheck](http://logcheck.org/)
* [AIDE](https://www.tecmint.com/check-integrity-of-file-and-directory-using-aide-in-linux/)
* [AIDE_2](http://aide.sourceforge.net/)

---
--- 


# Linux (security modules):
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
  
---
---
# FreeBSD:

## L'hardening su BSD:
Prima di utilizzare tools di qualsiasi tipo, è necessario innanzitutto seguire delle accortezze per portare il sistema di base alla miglior configuraione di sicurezza possibile:

* Prevenire i login, attraverso comandi come `pw lock id-name`  o cambiando la shell in `chsh /usr/sbin/nologin id-name`.
* Gestire al meglio le escalation di privilegi, ad esempio quando un utente non root necessita di permessi root; è sconsigliabile aggiungere l'utente al gruppo condiviso `wheel`  (grazie al quale un utente può ottenere i privilegi digitando **su** + **wheel-pwd** ). E' preferibile invece utilizzare il pacchetto `sudo`, il quale permette di controllare in modo più fine i permessi di un determinato utente, permettendo anche di controllare quali comandi il suddetto utente possa eseguire con privilegi e quali no. Per la modifica di tali privilegi è necessario editare `/usr/local/etc/sudoers`.
* Utilizzare delle policy restrittive riguardo le password, riguardo la loro gestione e il loro riutilizzo attraverso l'inmplementazione del modulo built-in PAM; esso permette di spacificare diversi parametri riguardo alla lunghezza e complessità di una password, riguardo alla sua durata, al suo poter somigliare ad altre password precedenti etc etc. Per l'attivazione è necessario scommmentare in `/etc/pam.d/passwd` la riga contenente `pam_passwdqc.so` e andando a dichiarare comandi che specifichino come una pwd debba essere gestita.
* Controllare eventuali vulnerabilità all'interno dei pacchetti installati attraverso il comando

                pkg audit -f
Esso consente di confrontare le applicazioni **di terze parti** installate sulla propria macchina con un database fornito direttamente dal team di sicurezza di FreeBSD, ottenendo ad esempio un output di questo tpo

                Affected package: cups-base-1.1.22.0_1
                Type of problem: cups-base -- HPGL buffer overflow vulnerability.
                Reference: <https://www.FreeBSD.org/ports/portaudit/40a3bca2-6809-11d9-a9e7-0001020eed82.html>
                1 problem(s) in your installed packages found.
                You are advised to update or deinstall the affected package(s) immediately.


</br>
Per mantenere un buon livello di sicurezza su BSD è poi possibile utilizzare tool e strumenti ulteriori quali:

* **AIDE** (Advanced Intrusion Detection Environment): come già visto su Linux, prende snapshot dello stato del sistema, registra gli hash, i timestamp delle modifiche e altri parametri personalizzabili. Gli snapshot vengono utilizzati per creare un DB;
* **Logcheck**: lo stesso visto su linux, permette una scansione periodica dei log di sistema e un report secondo diversi gradi di filtro dei risultati dello scan  
* **ACL** (Access Control List): sono un'estensione del meccanismo per esprimere regole che determinano l'accesso ad alcune risorse (che si trovano anche all'itnerno di un sistema linux) con una garanzia di maggior controllo all'amministratore. A differenza delle _capabilities_, che descrivono cosa un utente/processo può/non può fare e con che permessi, le ACL descrivono regole associate ad ogni oggetto in relazione a quali azioni possono essere eseguite da quali utenti. ACL e capabilities, insieme, formano la _Access Matrix_. Per consentire l'abilitazione delle ACL all'avvio è necessario settare in `/etc/fstab` il parametro **acls**. Usando `tunefs` è possibile impostare il flag automaticamente ed in modo permanente (di fatto modifica un superblocco nell'ACL flag dell'header di sistema); così facendo si è sicuri che non possano avvenire modifiche al flag di acls e che il sistema si avvii sempre con le ACL attive (anche senza alcun parametro all'interno di _fstab_). Una volta settate le ACL i filesystem con le ACL abilitate presentaranno un `+` a fianco delle informazioni a loro relative.
  </br>
Per visualizzare le ACL abilitate su un determinato elemento è sufficiente dare il comando:

                getfacl nome_el


   Per cambiare le impostazioni riguardo un determinato elemento si utilizza `setfacl`, con il flag `-k` qualora si volessero rimuovere tutte le policy esistenti o il flag `-m` per modificare le entries di default. Al netto dei flag che si possono definire in chiamata a `setacls`, i parametri standard richiesti sono sostanzialmente 5:

   1. Un **ACL_TAG**: specifica il tipo di entry (_u_ per User, _g_ per Group, _owner@_, _everyone@_ (NB != da _others_ ) o _group@_ per indicare il gruppo owner )
   2. Un **ACL_QUALIFIER**: specifica il gruppo o l'utente associato, e pertanto consiste in un _gid_ (o group name) o un _uid_ (o username).
   3. Un elenco di **ACCESS_PERMISSIONS**: rappresentano i permessi che il gid/uid ha sull'elemento considerato, e sono  read_data
   

	     w		 write_data
	     x		 execute
	     p		 append_data
	     D		 delete_child
	     d		 delete
	     a		 read_attributes
	     A		 write_attributes
	     R		 read_xattr
	     W		 write_xattr
	     c		 read_acl
	     C		 write_acl
	     o		 write_owner
	     s		 synchronize

       A cui si possono aggiungere descrizioni globali come:

             Set	         Permissions
	         full_set	 all permissions, as shown above
	         modify_set	 all permissions except	write_acl and write_owner
	         read_set	 read_data, read_attributes, read_xattr	and read_acl
	         write_set	 write_data, append_data, write_attributes and
		                	 write_xattr


  4. Un **ACL_INEHRITANCE_FLAG**: definiscono l'ereditariet dei permessi specificati, e possono essere:


	     f	    file_inherit
	     d	    dir_inherit
	     i	    inherit_only
	     n	    no_propagate
	     I	    inherited

  5. Una definizione dell'**ACL_TYPE**: può essere "deny" o "allow"

* **MAC** (Mandatory Access Control): è un'unione del framework del MAC con le ACL, in modo da fornire moduli che possano essere caricati per gestire delle policy di sicurezza. L'essere _mandatory_ implica che il rinforzo dei controlli è eseguito dal sistema direttamente. 
</br>
Le parole chiave quando si parla di MAC in ambito BSD sono:
        
  * _Compartment:_ un set di programmi o dati che vengono partizionati o separati, in cui gli utenti ricevono un'assegnazione di determinate risorse e componenti di un sistema
  * _Integrity:_ livello di fiducia che si può riporre in un dato.
  * _Level:_ l'impostazione di un attributo di sicurezza; al salire del livello sale anche la sua sicurezza
  * _Label:_ attributo di sicurezza che può essere applicato ai file, alle dir o ad altri elementi del sistema. Quando è applicato ad un file ne descrive le proprietà di sicurezza, garantendo l'acesso solo a utenti e processi con label simili. 
  * _Policies:_ collezione di regole che descrivono e definiscono il flusso di dati e informazioni tra gli elementi sottoposti a MAC
  * _Sensitivity:_ descrive quanto un dato deve essere mantenuto confidenziale in relazione ad un utente o un processo.

  **APPLICARE LE LABEL**: </br>
  Sono attributi di sicurezza che vengono applicate a soggetti e oggetti e usate come parte delle decisioni di controllo d'accesso effettuate da una policy (possono essere parte di una **rule** più complessa o essere usate direttamente affinchè la policy prenda una decisione). Esistono due tipologie di policies:
    1. _Single Label:_ di default del sistema, permette di usare solo una label per soggetto/oggetto del sistema (diminuendo l'overhead per l'amministratore, ma riducendo anche la flessibilità garantita da un'implementazione Multi Label del MAC).
    2. _Multi Label:_ permette ad ogni oggetto di avere una propria etichetta MAC indipendente. Questo aumenta il carico per l'amministratore (in quanto tutto ha un'etichetta, compresi nodi di sistema e cartelle) ma aumenta sensibilimente la flessibilità che ne deriva. Per abilitare la modalità multilabel è necessario utilizzare il comando `# tunefs -l enable /`.
  </br>

  Per configurare le label si utilizza il comando `# setfmac` seguito dal modello di riferimento e dal flag voluto; ad esempio per utilizzare il modello _Biba_ (caratterizzato da "write up, read down" in contrasto con il BLM) su un elemento test 

                # setfmac biba/high test
  Un ammonistratore di sistema può utilizzare `setpmac` per modificare il modulo di una policy (essendo in "read down")

                # setpmac biba/low setfmac biba/high test
  Alcune policy module, tra cui Biba, MLS, e LOMac, prevedono etichettatura su 3 livelli:
    * _Low_: è il livello più basso possibile, bloccano l'accesso a livelli superiori secondo il modello utilizzato
    * _Equal:_ oggetti che vengono considerati esenti dalle policy, ignorati o disabilitati
    * _High:_ livello più alto del modello Biba e MLS, nel primo ad esempio blocca il "write down"
  
  Entrambi i modelli suddetti inoltre supportano una etichettatura numerica che consente di specificare esattamente il livello di appartenenza. Gli oggetti di sistema inoltre hanno un grado e un compartimento, i soggetti riflettono il range di permessi disponibili.
  </br>
  Gradi e compartimenti in oggetti e soggetti formano quelle che si chiamano **dominance**, in cui dipendentemente da livello e compartimento un soggetto _domina_ un oggetto, viceversa, o nessuno domina nessuno (stesso livello).
 
  Anche agli utenti è richiesto di avere un'etichetta, cosìcche i loro file e processi possano interfacciarsi con le policies definite nel sistema e sono configurabili all'interno di `/etc/login.conf` utilizzandole login classes. (ad esempio con una riga di questo tipo)

        	:label=partition/13,mls/5,biba/10(5-15),lomac/10[2]:
   
   Le label possono inoltre essere applicate anche alle interfacce di rete per aiutare il controllo dei dati attraverso la rete stessa. Ad esempio qualora volessimo settare una label **biba/equal** all'interfaccia `bge0` dovremo dare un comando del tipo:

                # ifconfig bge0 maclabel biba/equal


  **GESTIRE LE POLICY** </br>
  Ogni modulo contenuto nel framework MAC **possa essere caricato come modulo kernel a runtime** attraverso `kldload`.
  Definito il modulo, questo viene aggiunto a `/boot/loader.conf` affinchè venga avviato al boot. Le policy princcipali principali sono:
   * _mac\_seeotheruids\.ko_ 
   * _mac\_bsdextended\.ko_  -> fornisce un'estensione al normale modello di gestione degli accessi al filesysyem, permettendo all'amministratore di creare un set di regole simil-firewall epr gestire file e directories all'interno del fs. Si usa `ugidfw` seguito da flag per vedere e settare le policy
   * _mac\_portacl_ ->  permette il controllo di determinate porte da parte dell'utente (anche non root) anche all'interno del range delle well-known-ports.
   * _mac\_partition.ko_-> quando questa policy è abilitata, gli utenti possono solo vedere i loro proessi e quelli confinati alla loro partizione, ma non hanno permessi per lavorare con elementi al di fuori della partizione. 
   * _mac\_mls.ko_ -> implementa il _MultiLevel-Security_ module, che garantisce un controllo definendo regole tra **oggetti e soggetti**. Tutto ciò che ha una lebel **low** ha il livello di _clearance_ più basso, e non ha accesso a informazioni di livello più alto. Previene anche che oggetti di livello superiore abbiano capacità di scrittura su livelli inferiori. **Equal** prende gli oggetti esenti dalla policy mentre **high** definisce  le _clearance_ più elevate, con dominanza sugli altri oggeti del s.o. e inmpossibilità di "write down". MLS definisce policy di **"No read up, No write down"** (al contrario del Bibba)
   * _mac\_biba_


* **JAILS**: Si basano su quello che è il `chroot`, che modifica la directory di root di una serie di processi, creando un ambiente sicuro isolato dal resot del s.o. Le jails tuttavia migliorano ed estendono il concetto di chroot sotto vari aspetti: in un chroot tradizionale i processi sono vincolati alla parte di filesystem a cui possono accedere, con il resto dei processi che è condiviso dai processi chrooted e da quelli di sistema. Le jails espandono questo concetto virtualizzando l'accesso al fs, garantendo un controllo più fine e granulare per quel che riguarda l'accesso all'environment jailed. Sono da cosiderarsi a tutti gli effetti un tool di virtualizzazione a lievllo os al pari, ad esempio, di docker. </br>
Ogni jail è caratterizzata da 4 elementi:
  * Una _subtree directory:_ starting point da cui si ha accesso allal jail
  * un _hostname:_ usato dalla jail
  * Un _IP-address:_ assegnato alla jail, spesso è un alias per un'interfaccia di rete esistente
  * Un _comando:_ path name ad un eseguibile da eseguire all'interno della jail, relativo alla root della jail stessa
Ogni jail ha il proprio utente root, il cui potere al di fuori della jail di lavoro è nullo.

  **CREAZIONE JAILS:** </br>
  Prima di tutto è necessario andare a definiretramite `DESTDIR` quale sarà la root della jail

        # export DESTDIR=/here/is/the/jail
  
  e succcessivamente montare la iso

        # mount -t cd9660 /dev/`mdconfig -f cdimage.iso` /mnt

  Si deve poi estrarre la tarball nella destinazione DESTDIR

        # tar -xf /mnt/usr/freebsd-dist/base.txz -C $DESTDIR

  E successivamente costruire la jail

        # setenv D /here/is/the/jail
        # mkdir -p $D      
        # cd /usr/src
        # make buildworld  
        # make installworld DESTDIR=$D  
        # make distribution DESTDIR=$D  
        # mount -t devfs devfs $D/dev 

  I cui comandi eseguono, in ordine:
    * Selezione della **locazione fisica** della jail (dove effettivamente risiede). Una buona scelta è una dir tipo `/user/jails/nomejail` dove _nomejail_ è l'hostname.
    * `buildworld` permette di creare l'userland all'interno della jail
    * `installworld` permette di installare all'interno della dir scelta come subtree tutti i man, le librerie etc 
    * `make` installa invece ogni file aggiuntivo contenuto in `usr/src/etc/`. 

  Una volta installata può essere gestita tramite l'utility `jails` o avviata al boot grazie a rc e un flie `jail.conf` che contiene i parametri e all'edit del file `rc.conf` che contiene la direttiva all'init

##### Fonti
* [Access Matrix](https://prosuncsedu.wordpress.com/2014/08/21/comparing-object-centric-access-control-mechanisms-acl-capability-list-attribute-based-access-control/)
* [ACL](https://www.freebsd.org/cgi/man.cgi?query=setfacl&sektion=1&manpath=freebsd-release-ports)
* [MAC](https://www.freebsd.org/doc/handbook/mac.html)
* [MAC-BIBA](https://www.freebsd.org/cgi/man.cgi?mac_biba)
* [BIBA-MODEL](https://en.wikipedia.org/wiki/Biba_Model)