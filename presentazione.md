# 📝 patterns in Ruby

## Cos'è un design pattern?
---
Un design pattern è un modo specifico di affrontare un problema.
Conoscere i design pattern più ricorrenti offre un aiuto non
indifferente per superare tutti gli ostacoli che ci vengono posti
ogni giorno.

## Origine dei design pattern
Nel 1995 un gruppo di sviluppatori identificò 23 tipi di pattern
per affrontare diverse problematiche e ne scrissero un libro:
**Design Patterns: Elements of Reusable Object-Oriented Software**

## Usare un design pattern piuttosto che un altro
---
Purtroppo non c'è una regola specifica che ci dice quando è meglio
seguire una certa strada rispetto ad un'altra, l'unica regola
d'oro è: **Fai solo quello che serve**
#### Fai solo quello che serve
### Fai solo quello che serve
## Fai solo quello che serve
# Fai solo quello che serve 🤬🔪

# Design pattern: emmocomesefa?

## Template Method Pattern

Il template method pattern ci permette, ereditando una classe, di non duplicare il codice in ogni classe. Un caso d'uso comune è quando dobbiamo elaborare dei dati in maniera diversa:
```ruby
class NotifyDelay
  def initialize(message, person)
    @message = message
    @person = person
  end

  def notify
    print "to: #{@person}\n#{message}"
  end
end

class NotifyDelayViaSlack < NotifyDelay
  def notify
    # complex logic to set up slack
    client.send(to: @person, message: @message)
  end
end

class NotifyDelayViaEmail < NotifyDelay
  def notify
    # complex logic to set up mailer
    smtp.sendmail(@message, 'simonebravo@nebulab.it', @person)
  end
end
```

## Strategy Pattern

E' molto simile al template method pattern ma invece di estendere diverse volte una classe si ha una classe sola che sceglie il miglior modo di compiere una determinata azione runtime. Un esempio molto inerente con il  nostro lavoro è quello di gestire è spedire un ordine utilizzando servizi differenti.

```ruby
class Order
  # order logic

  def ship_with(carrier_class)
    carrier_class.ship(self)
  end
end

class UPSCarrier
  def ship(order)
    # Ship via UPS praying for apis to be responsive
  end
end

class BRTCarrier
  def ship(order)
    # Ship via BRT
  end
end
```

## Observer Pattern
L'observer pattern permette di "avvertire" tutti gli oggetti interessati ad un certo avvenimento permettendogli di non effettuare polling. Per implementarlo la classe che contiene l'azione o il dato interessato deve fornire un metodo per iscriversi a queste notifiche e un secondo metodo per avvertire tutti gli oggetti interessati. In genere il secondo metodo viene chiamato subito dopo la generazione dell'evento. In ruby già è presente il modulo Observable che ci facilita l'implementazione:

```ruby
class Mauretto
  include Observable

  def receive_package(package)
    print_manifest package
    stock_package package
    lose_simones_package package

    notify_observers(Time.now, package)
  end

  private

  # some Mauretto's methods
end

class NebulabEmployee
  def initialize(doorman)
    doorman.add_observer(self)
  end

  def update(time, package)
    print "Raga è arrivato un pacco alle #{time} chi scende a prenderlo?"

    wait_for_cooworker
  end
end
```


## Composite Pattern
Il composite pattern è l'implementazione dell'approccio top-down ovvero la scomposizione di un'azione complessa in diverse azioni più piccole. Questa operazione può essere ripetuta più volte e non c'è un limite di profondità per la scomposizione di un problema. Non ho aggiunto un esempio di codice perché sarebbe venuto un esempio troppo complesso e forviante. Il concetto è di avere due tipo di classi:
- la prima è un contenitore che si occupa solo di chiamare le classi contenute in esso;
- la seconda contiene tutte le operazioni da eseguire per finalizzare un compito minore.

```ruby
class CompositeTask
  def inizialize(tasks)
    @taks << tasks
  end

  def call
    @tasks.each(&:call)
  end
end

class Task
  def call
    # Do something
  end
end
```
Come si può dedurre dall'esempio le classi di tipo composite possono essere nestate teoricamente all'infinito mentre i task sono le parti terminali compiti ben determinati

I problemi più grandi che si possono incontrare utilizzando il composite pattern sono:
- Scomporre il problema eccessivamente
- Non considerare il fatto che la profondità della scomposizione possa essere > 1

## Iterator Pattern
Non c'è tanto da spiegare qui, l'unica cosa figa è il fatto di includere il modulo `Enumerator` per avere tutti i metodi che esso contiene. L'unica cosa da fare è sovrascrivere i metodi `each` e `<=>`.

```ruby
class Prize
  attr_accessor :amount

  def <=>(other)
    amount <=> other.amount
  end
end

class JackPot
  include Enumerator

  attr_accessor :prizes

  def each(&block)
    @prizes.each(&block)
  end
end
```

## Command Pattern
I command pattern ci tornano utile quando abbiamo bisogno di eseuguire delle azioni reversibili. Quindi per ogni azione deve essere previsto un rollback per riportare lo stato come era in precedenza. Un'esempio molto semplice che ci fa capire il concetto di questo pattern sono le migrazioni di rails.

```ruby
class DropManufacturers < ActiveRecord::Migration
  def up
    drop_table :manufacturers
    remove_column :spree_products, :manufacturer_id, :integer
  end

  def down
    add_column :spree_products, :manufacturer_id, :integer
    create_table :manufacturers do |t|
      t.string :name, null: false
      t.string :zip_codes, array: true, default: []

      t.timestamps null: false
    end
  end
end
```

Guardando l'esempio si capsice subito che l'azione non è completamente reversibile. Un grande problema del command patter infatti è che non sempre le azioni possono essere annullate completamente.

## Adapter Pattern
L'adapter pattern viene principalmente utilizzato quando dobbiamo trasformare dei dati presi da un oggetto per permettere un secondo oggetto di poterli elaborare. Un esempio molto semplice potrebbe essere la conversione dei separatori delle cifre decimali dallo standard europeo a quello indiano in una stringa contenente un numero.

```ruby
class TheSalaryIDreamEveryNight
  def amount
    "100.000"
  end
end

class EuropeanToIndianConverter
  def convert_amount
    n = TheSalaryIDreamEveryNight.new.amount
    groups = n.delete('.').scan(/(.*)(\d{3}$)/).flatten
    [groups.first.scan(/(\d{1,2})/), groups.last].flatten.join('.')
  end
end

EuropeanToIndianConverter.new.convert_amount(TheSalaryIDreamEveryNight.new.amount)
# => 1.00.000
```
Quindi adesso se un indiano vuole assumermi per la folle cifra di 1.00.000 rupie o 1.174,66 euro l'anno può farlo.


## Proxy Pattern
Un proxy è semplicemente un involucro che ricopre una classe. Può essere di diversi tipi:
- protection proxy che protegge alcuni metodi nascondendoli o autorizzandoli;
- remote proxy che ci permette di utilizzare una risorsa remota senza dover pensare a come funziona;
- virtual proxy che ci permettono di rimandare grandi carichi di lavoro fino a che non è necessario eseguire delle operazioni su di essi. es: creare un oggetto che richiede una chiamata api alla sua inizializazzione.

Un problema comune che si genera con i proxy è di fare il proxy di metodi che non dovrebbero essere toccati come ad esempio `to_s`, `anchestors`, `tap`...

## Decorator Pattern
Il decorator pattern ci aiuta a customizzare una classe senza il bisogno di sovrascriverla completamente

```ruby
class MostComplexClassInTheWorld
  def ugly_method
    print "🔻"
  end
  # a lot of methods here
end

# I don't think ruby is ugly
module ShowThemWhatsUgly
  def ugly_method
    print "🐍"
  end

  MostComplexClassInTheWorld.prepend self
end
```

## Singleton Pattern
Il concetto di singleton è quello di permettere un'istanza sola di una determinata classe in tutta l'applicazione e di rendere tale istanza globale. Questo approccio risulta molto efficace quando una classe non ha motivo di avere più istanze o se la risorsa è condivisa: ad esempio una configurazione, un logger o un orm. Ruby ci fornisce il modulo `Singleton` da includere nella nostra classe.

```ruby
require 'singleton'

class NebulabsToilet
  include Singleton

  def initialize
    @tenant = nil
  end

  def occupy(tenant)
    raise "Sto bagno è sempre pieno!" if @tenant

    @tenant = tenant
  end

  def free
    @tenant = nil
  end
end

NebulabsToilet.instance.occupy 'Andrea'
#=> "Andrea"
NebulabsToilet.instance.occupy 'Matteo'
# RuntimeError: Sto bagno è sempre pieno!
NebulabsToilet.instance.free
#=> nil
NebulabsToilet.instance.occupy 'Matteo'
#=> "Matteo"
```

Come puoi vedere il nostro bagno è uno solo(singola istanza) ed è condiviso in tutto l'ufficio(applicazione)... è l'esempio perfetto di singleton. Ci sono anche altre alternative al modulo singleton, per esempio usare una classe con tutti metodi e variabili di classe e bloccando l'accesso al metodo initialize.

## Factory Method Pattern

Il factory method pattern è un ibrido tra il template e lo strategy pattern, esso risulta molto utile quando bisogna creare degli oggetti senza sapere esplicitamente quale bisogna creare.

```ruby
class Pond
  def initialize(number_animals)
    @animals = number_animals.times.inject([]) do |animals, i|
      animals << new_animal("Animal#{i}")
      animals
    end
  end

  def simulate_one_day
    @animals.each {|animal| animal.speak}
    @animals.each {|animal| animal.eat}
    @animals.each {|animal| animal.sleep}
  end
end

class FrogPond < Pond
  def new_animal(name)
    Frog.new(name)
  end
end

pond = FrogPond.new(3)
pond.simulate_one_day
```

## Builder Pattern

Il builder pattern è molto utile per generare oggetti complessi in modo semplice e utilizzando comandi più verbosi:

```ruby
class OfficeChair
  attr_accessor :seat, :back, :legs, :arms
end

class OfficeChairBuilder
  def initialize
    @chair = OfficeChair.new
  end

  def add_seat(seat)
    @chair.seat = seat
    self
  end

  def add_back(back)
    @chair.back = back
    self
  end

  def add_legs(legs)
    @chair.legs = legs
    self
  end

  def add_arms(arms)
    @chair.arms = arms
    self
  end
end

OfficeChairBuilder.new.add_seat('comoda')
                      .add_back('dritto')
                      .add_legs('resistenti')
                      .add_arms('basse')
```
## Interpreter Pattern

```ruby
class Or < Expression
  def initialize(expression1, expression2)
    @expression1 = expression1
    @expression2 = expression2
  end

  def evaluate(dir)
    result1 = @expression1.evaluate(dir)
    result2 = @expression2.evaluate(dir)
    (result1 + result2).sort.uniq
  end
end

class Bigger < Expression
  def initialize(size)
    @size = size
  end

  def evaluate(dir)
    results = []
    Find.find(dir) do |p|
      next unless File.file?(p)
      results << p if( File.size(p) > @size)
    end
    results
  end
end


class FileName < Expression
  def initialize(pattern)
    @pattern = pattern
  end

  def evaluate(dir)
    results= []
    Find.find(dir) do |p|
      next unless File.file?(p)
      name = File.basename(p)
      results << p if File.fnmatch(@pattern, name)
    end
    results
  end
end

big_or_mp3_expr = Or.new( Bigger.new(1024), FileName.new('*.mp3') )
big_or_mp3s = big_or_mp3_expr.evaluate('test_dir')
```
