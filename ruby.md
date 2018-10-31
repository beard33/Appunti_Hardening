### VARIABILI

* _Stringhe_ -> " "
* _Numerici_ -> int, float (n.m)
* _Boolean_ -> true/false
* _Costanti_ -> Iniziano con la maiuscola

#### OPERAZIONI BASE

* " + "
* " * "  
* " - "
* ** (exp)
*  /
*  % (modulo)
  
`puts` e `print` differiscono principalmente per il fatto che la prima aggiunge automaticamente una new_line in coda, la seconda no.

**NB** In ruby _tutto_ è oggetto, e pertanto tutto possiede una serie di metodi legati alle proprietà dell'oggetto di cui è parte.
 Ponendo un `!` in coda al metodo vado a modificare direttamente la variabile su cui sto eseguendo il metodo stesso. 
I **commenti** vengono scritti preceduti da `#` nel caso di commenti mono riga, da `=begin` / `=end` nel caso di commenti multiriga

#### INPUT/OUTPUT
Il metodo per leggere l'input dell'utente è `gets`, a cui ruby aggiunge automaticamente una newline in coda. Per eliminare tale riga è necessario concatenare al gets il metodo `chomp` secondo la sintassi `variabile = gets.chomp`.
Per stampare una variabile è necessario utilizzare il _segnaposto_ `#{variabile}` secondo la sintassi `puts "stringa a piacere #{variabile}"`

#### STRINGHE
Le stringhe come tutti gli elementi in Ruby sono oggetti e posseggono nativamente dei metodi ad esse applicabili:
* `length` -> ogni stringa possiede intrinsecamente il metodo `length` attraverso la sintassi `stringa.length`). E' possibile anche concatenare il metodo in dichiarazione, ad esempio `"stringa".length`.
* `reverse` -> inverte l'ordine dei carattaeri nella stringa
* `upcase, downcase, capitalize` -> settano la stinga in maiuscolo/minuscolo/iniziale maiuscola
* `stringa.include?("substring")` -> controlla se la stringa _stringa_ include la _substring_ al suo interno e restituisce un booleano
* `gsub(/char/ , "new_char")` -> permette la **g**lobal **sub**stitution del char selezionato con il _newchar_
* `split("char")` prende in input una stringa e restituisce un array di elementi creato utilizzando il _char_ passato come valore come carattere delimitatore per la suddivisione
  


#### CONTROL FLOW
* **IF** ->   la gestione dell'if avviene secondo la seguente sintassi. Per le valutazioni booleane `==`. Sono inoltre possibili controlli come `&&` e `||`

                if condizione
                    istruzioni
                elsif condizione
                    istruzioni
                else
                    istruzioni
                end
* **UNLESS** -> condizione che non esegue il codice interno finchè la condizione considerata _non è verificata_ ("finchè non"), e segue la seguente sintassi

                unless condizione
                    istruzioni
                else
                    istruzioni
                end
La forma dell'unless lo rende di fatto un'abbreviazione dell'if ed è scrivibile anche nella forma

                istruzioni unless condizione
* **WHILE** -> condizionale uguale ad altri linguaggi, la sintassi è la seguente

                while condizione
                    istruzioni
                end

* **UNTIL** -> condizionale inverso al while, da leggersi come un "finchè non" nella forma
    
                until condizione
                    istruzioni
                end

* **FOR** -> condizionale uguale ad altri linguaggi, nella forma

                for condizione
                    istruzioni
                end

Un modo tipico di ciclare su indici è nella scrittura `for num in min...max` o `for num in min..max` dove le differenze risiedono dell'utilizzo di **..** o **...**, per cui mentre il primo prende anche l'estremo superiore, il secondo ignora l'estremo superiore e si ferma allo step immediatamente precedente

* **TIMES** -> è un sostituto del for in determinati contesti (**?**), esegue un determinanto numero di volte una porzione di codice; nell'esempio 10

                10.times do
                    istruzioni
                end

* **LOOP** -> _loop_ è il più semplice degli iterator ed è possibile ad esempio creare un ciclo infinito scrivendo `loop { puts"Hello World" }`. La sintassi generica è la seguente

                loop {
                    istruzioni
                }

Tenendo a mente che in Ruby _{}_ sono sostituibili da `do` `end`, pertanto

                loop do
                    istruzioni
                end
**NB** È importante inserire una condizione di uscita dal loop, ad esempio inserendo uno statement `break`, accompagnato ad esempio da un `if`

                loop do
                    istruizioni
                    break if  condizione_uscita
                end
Anche in Ruby è presente la keyword `next` per skippare alcuni passi del loop e passare all'iterazione successiva, magari accompagnando anche in questo caso lo statement con una condizione

                loop do
                    next if condizione_next
                    istruzioni
                end

#### ARRAY
Assegnato attraverso parentesi quadre, con una sintassi tipo `my_array = []`

* **EACH** -> simile al loop ma applicato ad ogni elemento di una determinata collezione. Come in java il nome _item_ è soltanto un segnaposto per ogni elemento del ciclo, può avere qualunque nome

                object.each { |item|
                    istruzioni
                }
E similmente con i do/end
                
                object.each do |item|
                    istruzioni
                end

#### DATA STRUCTURES
È possibile dichiarare anche array multidimensionali, ad esempio `my_2d_array = [[1, 2, 1],[2, 4, 1]]`

* **HASH** -> sono l'equivalente dei dizionari di Py, una corrispondenza _chiave-valore_. La loro dichiarazione avviene come segue:

                hash = {
                    "chiave_1" => valore_1
                    "chiave_2" => valore_2
                    ... 
                    "chiave_N" => valore_N
                }
È poi possibile accedere ad un valore corrispondente ad una determinata chiave attraverso la stessa, ad esempio `puts hash["chiave_1"]`  restituirà _valore 1_. 
Per creare un hash vuoto si può fare attraverso le graffe `my_hash = {}` o attraverso l'utilizzo della keyword _new_  associata alla famiglia di oggetti in questione `my_hash = Hash.new`. L'inserimento di una coppia chiave/valore avviene o in dichiarazione o attraverso la sintassi `hash["chiave"] = "valore"`. Per iterare su un'hash stampando sia chiave che valore è possibile utilizzare la sintassi `my_hash.each do |key, val|`. </br>
Esistono anche i metodin `each_key` e `each_value` per iterare solo su chiavi e valori </br>
E' possibile anche definire un default per il valore che deve essere restituito qualora non si trovasse corrispondenza tra la chiave assegnata ed un valore dell'hash attraverso la sintassi `my_hash = Hash.new{"val_restituito"}`.  </br> **Simboli**: in Ruby se non si vogliono utilizzare le stringhe come chiavi, ma i _simboli_ è necessario anteporre i due punti alla dichiarazione

                 hash = {
                    :chiave_1 => valore_1
                    :chiave_2 => valore_2
                    ... 
                    :chiave_N => valore_N
                }

O da Ruby 1.9

                new_hash = { 
                one: 1,
                two: 2,
                three: 3
                }

E' possibile dichiarare un simbolo attraverso la sintassi `my_symbol = :symbol_name` e convertire un simbolo a stringa e viceversa usando `to_s` e `to_sym` (o `intern`)

NB: mentre per le stringhe è possibile avere due o più stringhe che rappresentano lo stesso valore ma con diverso ID, può esistere soltanto un dato simbolo con quell'ID. Ad esempio runnando il codice:

                puts "string".object_id  #=> 17010120
                puts "string".object_id  #=> 16997500

                puts :symbol.object_id   #=> 802268
                puts :symbol.object_id   #=> 802268

Per selezionare invece valori che rispettino determinati criteri si può utilizzare il metodo `select`

                grades = { alice: 100,
                bob: 92,
                chris: 95,
                dave: 97
                }

                grades.select { |name, grade| grade <  97 }


* **SORT**:
Sulle collezioni è possibile chiamare il metodo `sort` che, di default, ordina secondo l'ordine crescente gli elementi. Supporta anche l'operatore `<=>` che restituisce 0, 1 o -1 a sceodna del valore assunto dal confronto tra due elementi. Ad esempio per ordinare in ordine decrescente un _Array_ è possibile fare

                Array.sort do |item1, item2|
                    item2 <=> item1
                end

#### METODI
Vengono definiti come in python, con la differenza che richiedono l'`end` al termine

                def nome_metodo (param1, param2)
                    foo()
                end
E' possibile passare come parametro un numero non specificato di valori (attraverso l'anteposizione dell'asterisco, ad esempio `def saluti (nome1, *nomi)`) e supporta anche i default values, similmente a python. `def metodo(val1, val2 = def_val)`

#### VARIE
* _Assegnazione condizionale_: viene fatta con l'operatore `||=` che assegna il valore alla destra se la variabile alla sinistra è **nil**, ignora l'assegnazione altrimenti

* _Ritorno implicito_: se non viene esplicitamente dichiarato un **return** all'interno di un metodo viene automaticamente ritornato il risultato dell'ultima espressione valutata

* _Valutazione rapida_ nel valutare le condizioni ruby valuta solo quelle necessarie, pertanto se per caso dovesse valutare `a && b` qualora **a** fosse falsa b non verrebbe nemmeno valutata, similmente per una valutazione tipo `a || b` qualora a fosse vera

* _respond\_to?()_ : prende un metodo in input e restituisce se l'elemento su cui è chiamato può eseguire quel metodo. Il metodo da controllare deve essere passato come simbolo (`:metodo`)

* _Push_ può essere sostituita da `<<` e funziona anche sulle stringhe

                "Yukihiro " << "Matsumoto"
                # ==> "Yukihiro Matsumoto"

Questa tipologia di concatenazione inoltre permette di aggiungere ad una stringa una variabile

                drink = "espresso"
                "I love " << drink          # ==> I love espresso

* _Collect_ : prende un blocco e vi applica un'espressione ad ogni elemento. NB: ritorna una copia dell'elemento su cui lavora, non modifica tale elemento

                fibs = [1, 1, 2, 3, 5, 8, 13, 21, 34, 55]
                doubled_fibs = fibs.collect { |num| num*2}

* _Yeld_ : permette di "iniettare" del codice all'interno di una funzione (codice che prenderà il posto della keyword)

                def block_test
                puts "We're in the method!"
                puts "Yielding to the block..."
                yield                                #=> qui verrà stampato "We're in the block!"
                puts "We're back in the method!"
                end

                block_test { puts ">>> We're in the block!" }

    E' inoltre possibile passare parametri allo `yeld` 

                def yield_name(name)
                    puts "Hi"
                    yield(name)
                end

                yield_name("B") { |n| puts "My name is #{n}." }

* _Procs_: sono un modo per evitare di ripetere i blocchi e permettono di salvarne l'output come fossero un metodo. Per definirli è sufficiente ciamare un `Proc.new` e passare l'argomento del blocco

                cube = Proc.new { |x| x ** 3 }

    Per passare il proc ad un metodo è necessario convertirlo in blocco anticipandolo con una `&`. Questa converte anche i simboli rappresentanti i metodi in Proc equivalenti </br>
Un Proc può essere chiamato anche grazie al metodo `call`, seguendo la sintassi `my_Proc.call`

                numbers_array = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
                strings_array = numbers_array.collect(&:to_s)
                puts strings_array

* _lambda_ : come i Proc sono oggetti e vengono definite con
  
                lambda { puts "Hello World"}

A differenza dei Proc le lambda controllano il numero di input, così se ad una lambda viene passato un numeri diverso di parametri lancia un errore, un proc ignora gli eccedenti ed assegna **nil** ai macanti. Inoltre quando una lambda ritorna, passa il controllo al metodo chiamante, un proc no

#### CLASSI

Vengono definite attraverso la sintassi

                def my_class
                    foo()
                end

E necessitano di un costruttore, ovvero il metodo `initialize`

                def my_class
                    def initialize (args)
                        foo()
                    end
                end

Per assegnare _variabili di istanza_ si antepone alle stesse il carattere `@`

                def person
                    def inizialize (name, age)
                        @name = name
                        @age = age
                    end
                end

Per istanziare un nuovo oggetto della classe definita si utilizza la sintassi `my_obj = My_class.new(args)`

***Scope***: è fondamentale nelle classi di Ruby, definisce il contesto di visibilità di una classe in un programma (*global, local, class, instance access*)

* `@` -> definisce variabili di istanza
* `$` -> definisce variabili globali
* `@@` -> definisce variabili variabili di classe
</br>

_Nota:_  L'accesso all'ultima tipologia avviene similmente a Java, utilizzando metodi che restituiscano all'esterno della classe il valore altrimenti privato dell'elemento. E' oltretutto necessario inserire `$` anche nella _chiamata_ ad una variabile globale
</br>

Le variabili di classe sono uniche e valide per ogni istanza della classe che le contiene (sono caratteristiche della classe, e.g. `@@n_route = 4` per la classe `auto`) e posso accedervi chiamandole dalla classe, non da sue istanze


                class Person

                    @@people_count = 0
                
                    def initialize(name)
                        @name = name
                        @@people_count += 1
                    end
                    
                    def self.number_of_instances
                        return @@people_count
                    end
                end

                matz = Person.new("Yukihiro")
                dhh = Person.new("David")

                puts "Number of Person instances: #{Person.number_of_instances}"    => 2


L'**EREDITARIETÀ** viene espressa, nella definizione della classe figlia, attraverso la seguente sintassi. Ogni classe può tuttavia avere **soltanto una superclasse**

                class child_class < sup_class
                    foo()
                end

E' ovviamente possibile fare _l'override_ dei metodi, semplicemente andando a ridefinire con lo stesso nome i metodi nella classe derivata
E' inoltre possibile accedere al metodo della superclasse attraverso la keyword `super()` </br>
Per definire _metodi di classe_ è necessario anteporre al nome del metodo quello della classe di apartenenza 

                def ClassName.method ()
                    foo()
                end


##### VISIBILITÀ:
Per dichiarare la vidibilità di un metodo è necessario, prima della sua definizione, anteporre le keywords `public` o `private` . Di default i metodi son pubblici.

E' possibile utilizzare due "scorciatoie" (simili ai getter/setter di Java):
* **attr_reader**: permette di accedere ad una variabile
* **attr_writer**: permette di modificare una variabile
* **attr_accessor**: permette di leggere _e_ modificare la variabile</br>
Per tutti e tre al nome deve seguire il _simbolo_ del nome della variabile


#### MODULI:
Si può pensare ai moduli come toolbox che contengono set di metodi e costanti (non variabili). Per convenzione le costanti sono scritte `ALL_CAPS`.

                module ModuleName
                    foo()
                end

E' possibile definire metodi con lo stesso nome in moduli differenti e chiamarli attraverso lo _scope definitio;_ ad esempio se definisco una costante PI in un modulo Circle posso accedere a essa o a quella di Math `Math::PI` o `Circle::PI`.

Per includere i moduli è necessario anteporre la keyword `require` seguita dal nome del modulo necessario 

                require `module`

E' possibile anche includere 

