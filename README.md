# Αρχιτεκτονική Υπολογιστών

## *Εργασία #1*

<u>Εισηγητής</u>: 	
Παπαευσταθίου Ιωάννης

<u>Ονόματα</u>: 	  
Χάιδω Πορλού		

Δημήτριος Παππάς

<u>ΑΕΜ</u>:				       
9372						

8391

Ομάδα:			13Α



#### ΕΡΩΤΗΜΑ 1

Ανοίγοντας το αρχείο starter_se.py που χρησιμοποιήσαμε για την εκτέλεση του προγράμματος hello, παρατηρούμε κάποιες παραμέτρους που έχουν περάσει στον gem5 για την προσομοίωση. Παραθέτουμε παρακάτω τις παραμέτρους αυτές, που δείχνουν τα βασικά χαρακτηριστικά του συστήματος, και στη συνέχεια σε ποιό σημείο του κώδικα βρίσκεται.

- cpu type: minor cpu 

  > cpu_types = {
  >  "atomic" : ( AtomicSimpleCPU, None, None, None, None),
  >  "**minor**" : (MinorCPU,
  >             devices.L1I, devices.L1D,
  >             devices.WalkCache,
  >             devices.L2),
  >  "hpi" : ( HPI.HPI,
  >            HPI.HPI_ICache, HPI.HPI_DCache,
  >            HPI.HPI_WalkCache,
  >            HPI.HPI_L2)
  > }

- frequency: 1GHz

  > self.clk_domain = SrcClockDomain(clock="**1GHz**",
  >                                       voltage_domain=self.voltage_domain)

- number of cores: 1

  >parser.add_argument("--num-cores", type=int, default=**1**,
  >                   help="Number of CPU cores")

- memory type

  >  parser.add_argument("--mem-type", default="**DDR3_1600_8x8**",
  >                      choices=ObjectList.mem_list.get_names(),
  >                      help = "type of memory to use")

- number of memory channels: 2 (L1 cache & L2 cache)

  >  parser.add_argument("--mem-channels", type=int, default=**2**,
  >                      help = "number of memory channels")

- memory size: 2GB

  > parser.add_argument("--mem-size", action="store", type=str,
  >                     default="**2GB**",
  >                     help="Specify the physical memory size")

Όπως φαίνεται από τα παραπάνω, τρέχοντας την εντολή 

*./build/ARM/gem5.opt configs/example/arm/starter_se.py --cpu="minor" "tests/test-progs/hello/bin/arm/linux/hello" # Hello world!* 

το μόνο όρισμα που δίνουμε στον gem5 είναι το cpu type  (minor) και όλα τα υπόλοιπα παίρνουν την default τιμή.

Συμπληρωματικά, για τις μνήμες cache, το σύστημα έχει L1 icache (instruction cache), L1 dcache (data cache) και L2 cache, το μέγεθος και γενικότερα τα χαρακτηριστικά των οποίων φαίνονται στο αρχείο devices.py.

- L1 icache

  >class L1I(L1_ICache):
  >tag_latency = 1
  >data_latency = 1
  >response_latency = 1
  >mshrs = 4
  >tgts_per_mshr = 8
  >size = '**48kB**'
  >assoc = 3

- L1 dcache 

  >class L1D(L1_DCache):
  >tag_latency = 2
  >data_latency = 2
  >response_latency = 1
  >mshrs = 16
  >tgts_per_mshr = 16
  >size = '**32kB**'
  >assoc = 2
  >write_buffers = 16

- L2 cache

  >class L2(L2Cache):
  >tag_latency = 12
  >data_latency = 12
  >response_latency = 5
  >mshrs = 32
  >tgts_per_mshr = 8
  >size = '**1MB**'
  >assoc = 16
  >write_buffers = 8
  >clusivity='mostly_excl'

Οι βασικές μονάδες λοιπόν του συστήματος μας είναι η cpu (που περιλαμβάνει και τις μνήμες cache), η κεντρική μνήμη και το shared memory bus.

#### ΕΡΩΤΗΜΑ 2

##### a. 

Φαίνεται πως οι πληροφορίες που παραθέτουν τα αρχεία stats.txt και config.ini ταυτίζονται με αυτές που καταγράψαμε στο προηγούμενο ερώτημα από το αρχείο starter_se.py. Τις παρουσιάζουμε στον παρακάτω πίνακα:

|                  |        config.ini         | line |
| :--------------: | :-----------------------: | :--: |
|     cpu type     |      *type=MinorCPU*      |  65  |
|    frequency     |       *clock=1000*        |  44  |
| number of cores  |   *multi_thread=false*    |  24  |
|    L1 icache     |       *size=49152*        | 929  |
|    L1 dcache     |       *size=32768*        | 183  |
|     L2 cache     |      *size=1048576*       | 1235 |
|   memory size    | *mem_ranges=0:2147483648* |  21  |
| memory type/mode | *mem_mode=timing* (DDR3)* |  20  |

*αρχείο *minor-timing.py & MInorCPU.py*



##### b. 

Το συνολικό νούμερο των committed εντολών είναι 5028 (αρχείο *stats.txt*, σειρά 14)

> sim_insts                                        5028                       # Number of instructions simulated

Το συνολικό νούμερο των committed operations (including micro ops) είναι 5834. 

>sim_ops                                          5834                       # Number of ops (including micro ops) simulated

Η διαφορά τους πιθανότατα να οφείλεται στο γεγονός ότι μια εντολή μπορεί να αντιστοιχεί σε πολλαπλές micro operations, καθώς και στην ύπαρξη κάποιου pipeline (στην περίπτωση της MinorCPU που τρέξαμε).

##### c. 

H L2 cache φαίνεται να προσπελάστηκε 479 φορές. (αρχείο *stats.txt*, σειρά 439)

>system.cpu_cluster.l2.overall_accesses::total   479   # number of overall (read+write) accesses

Οι προσπελάσεις της L2 cache θα μπορούσαν να υπολογιστούν βρίσκοντας των αριθμό των misses της L1 cache (καθώς η πληροφορία αναζητείται στο επόμενο επίπεδο cache αν δεν βρεθεί). Έτσι, από το αρχείο *stats.txt* (σειρές 316, 113, 139), μπορούμε να υπολογίσουμε τα συνολικά accesses της L2 είναι ίσα με το άθροισμα των συνολικών misses της L1 icache και L1 dcache, αφαιρώντας όμως τα συνολικά MSHR hits (δεύτερη προσπάθεια αναζήτησης στα miss status holding registers). 

> system.cpu_cluster.cpus.icache.overall_misses::total      332      # number of overall misses
>
> system.cpu_cluster.cpus.dcache.overall_misses::total    179        # number of overall misses
>
> system.cpu_cluster.cpus.dcache.overall_mshr_hits::total    32    # number of overall MSHR hits

#### ΕΡΩΤΗΜΑ 3

**Minor CPU:** H Minor CPU είναι ένα μοντέλο in-order CPU με fixed pipeline, το οποίο χρησιμοποιείται για την μοντελοποίηση επεξεργαστών με αυστηρό in-order execution. Δεν έχει τη δυνατότητα multithreading ("τρέχει'' με ένα πυρήνα), άλλα έχει 4 στάδια pipelining (Fetch1, Fetch2, Decode & Execute), όπως φαίνεται και στην παρακάτω απεικόνιση.

```
------------------------------------------------------------------------------
    Key:

    [] : inter-stage BufferBuffer
    ,--.
    |  | : pipeline stage
    `--'
    ---> : forward communication
    <--- : backward communication

    rv : reservation information for input buffers

                ,------.     ,------.     ,------.     ,-------.
 (from  --[]-v->|Fetch1|-[]->|Fetch2|-[]->|Decode|-[]->|Execute|--> (to Fetch1
 Execute)    |  |      |<-[]-|      |<-rv-|      |<-rv-|       |     & Fetch2)
             |  `------'<-rv-|      |     |      |     |       |
             `-------------->|      |     |      |     |       |
                             `------'     `------'     `-------'
------------------------------------------------------------------------------
```

  (πηγή [mygem5.org](https://www.gem5.org/documentation/general_docs/cpu_models/minor_cpu))

**SimpleCPU:** H Simple CPU είναι μία ακόμη in-order CPU που υποστηρίζει ο gem5, και είναι ένα μοντέλο που χρησιμοποιείται σε περιπτώσεις στις οποίες ένα πιο περίπλοκο μοντέλο δεν είναι απαραίτητο, όπως για παράδειγμα για την επιβεβαίωση της ορθότητας ενός προγράμματος. Υπάρχουν 3 είδη SimpleCPU: BaseSimpleCPU, AtomicSimpleCPU και TimingSimpleCPU, με την BaseSImpleCPU να είναι το βασικό μοντέλο και να μην μπορεί να χρησιμοποιηθεί μόνη (βλ. εικόνα). Η AtomicSimpleCPU χρησιμοποιεί atomic memory accesses, ενώ η TimingSimpleCPU χρησιμοποιεί timing memory accesses.

![](https://i.ibb.co/18wf00v/Screenshot-2020-11-22-Arm-Research-Starter-Kit-System-Modeling-using-gem5-gem5-rsk-pdf.png)

##### a.

Για το συγκεκριμένο ερώτημα γράψαμε ένα απλό πρόγραμμα σε γλώσσα C. Χρησιμοποιώντας διαφορετικά μοντέλα CPU (MinorCPU & TimingSimpleCPU), βλέπουμε μια διαφορά στο χρόνο εκτέλεσης (ticks). 

|    CPU type     | Number of TIcks |
| :-------------: | :-------------: |
|    MinorCPU     |    37787000     |
| TimingSimpleCPU |    43250000     |

##### b.

Οι χρόνοι φαίνεται να είναι συγκρισιμοί (ίδια κλίμακα), αλλά η TimingSimpleCPU είναι λίγο πιο αργή επειδή δεν χρησιμοποιεί pipelines, δηλαδή κάθε χρονική στιγμή "τρέχει" μόνο μία εντολή στον δίαυλο. Αντίθετα, η MinorCPU χρησιμοποιεί 4 στάδια pipelining (Fetch1, Fetch2, Decode & Execute), άρα είναι πιο αποδοτική. 

##### c.

###### TimingSimpleCPU

Τρέχοντας την εντολή 

*./build/ARM/gem5.opt -d MYPROGRAM configs/example/se.py --cpu-type=TimingSimpleCPU --caches --sys-clock=500000000 -c ccc*

δηλαδή μειώνοντας στο μισό τη συχνότητα λειτουργίας, το πρόγραμμα ολοκληρώνεται σαφώς πιο αργά, στα 50064000 ticks, κάτι το οποίο είναι αναμενόμενο. Κάτι που παρατηρήσαμε και είναι αξιοσημείωτο να αναφέρουμε είναι πως, μειώνοντας στο μισό την συχνότητα λειτουργίας, το clock period είναι 2000 ticks, σε αντίθεση με 1000 που ήταν πριν, επίσης αναμενόμενο (αρχείο *stats.txt*, σειρά 104).

Στη συνέχεια, τρέχοντας την εντολή

*./build/ARM/gem5.opt -d MYPROGRAM configs/example/se.py --cpu-type=TimingSimpleCPU --l2cache -c ccc*

δηλαδή δίνοντας εντολή να χρησιμοποιηθεί αποκλειστικά η L2 cache, το πρόγραμμα γίνεται πολύ πιο αργό, ολοκληρώνεται δηλαδή στα 215836500 ticks. Αυτό είναι αναμενόμενο επειδή η L1 cache είναι γρηγορότερη, οπότε αν αφαιρέσουμε την πρόσβαση σε αυτήν, το πρόγραμμα αργεί σημαντικά να τελειώσει.

###### MinorCPU

Τρέχοντας την εντολή 

*./build/ARM/gem5.opt -d MYPROGRAM configs/example/se.py --cpu-type=MinorCPU --caches --sys-clock=500000000 -c ccc*

δηλαδή μειώνοντας και πάλι στο μισό τη συχνότητα λειτουργίας, το πρόγραμμα ολοκληρώνεται σαφώς πιο αργά, στα 44266000 ticks.

Στη συνέχεια, τρέχοντας την εντολή

*./build/ARM/gem5.opt -d MYPROGRAM configs/example/se.py --cpu-type=MinorCPU --l2cache --caches -c ccc*

δηλαδή δίνοντας εντολή να χρησιμοποιηθεί κατά προτεραιότητα η L2 cache, το πρόγραμμα γίνεται πιο αργό, ολοκληρώνεται δηλαδή στα 45766500 ticks. Αυτό είναι αναμενόμενο επειδή η L1 cache είναι γρηγορότερη, οπότε αν δώσουμε προτεραιότητα  στην L2, η ολοκλήρωση του προγράμματος θα καθυστερήσει.

![](https://i.ibb.co/bQtBPNC/diagram.png)

Η μεγάλη διαφορά που παρατηρείται στο χρόνο εκτέλεσης της TimingSimpleCPU με την πρόσθεση του flag --l2cache έναντι του χρόνου εκτέλεσης της MinorCPU σε παρόμοια εντολή οφείλεται στο γεγονός ότι η TimingSimpleCPU χρησιμοποιεί αποκλειστικά την πολύ πιο αργή L2 cache, ενώ η MinorCPU αδυνατεί να τρέξει αποκλειστικά με L2 cache. Εναλλακτικά, επιλέξαμε να τρέξουμε το πρόγραμμα με προτεραιότητα στην L2 cache στην περίπτωση της MinorCPU, απόφαση που είχε ως αποτέλεσμα την αύξηση του χρόνου εκτέλεσης, αλλά σε πολύ μικρότερο βαθμό.

#### ΒΙΒΛΙΟΓΡΑΦΙΑ

[Using the default configuration scripts](http://pages.cs.wisc.edu/~david/courses/cs752/Spring2015/gem5-tutorial/part1/example_configs.html)

[Arm Research Starter Kit: System Modeling using gem5](https://raw.githubusercontent.com/arm-university/arm-gem5-rsk/master/gem5_rsk.pdf)

[gem5.org (SImpleCPU)](https://www.gem5.org/documentation/general_docs/cpu_models/SimpleCPU)

[gem5.org (MInorCPU)](https://www.gem5.org/documentation/general_docs/cpu_models/minor_cpu)



#### ΚΡΙΤΙΚΗ

Από την παραπάνω εργασία μάθαμε τι είναι ο εξομοιωτής gem5 και πως να τον χρησιμοποιούμε σε περιβάλλον Linux. Είναι ένα πρώτο βήμα για την κατανόηση ARM based CPU μοντέλων. Καταφέραμε να κάνουμε στατικά compile και να τρέχουμε ένα C πρόγραμμα μέσω του gem5. Δοκιμάσαμε να "τρέξουμε" το πρόγραμμα κάνοντας τροποποιήσεις στις παραμέτρους της συχνότητας και των μνημών cache και συμπεράναμε ότι τα χαρακτηριστικά αυτά συνδέονται άμεσα με τον χρόνο εκτέλεσης και τον αριθμό των ticks. Μάθαμε για τους διάφορους τύπους cpu που ορίζει ο εξομοιωτής και πρακτικά είδαμε πως λειτουργεί  ο MinorCPU και TimingSimpleCPU, καθώς και ποιες είναι οι διαφορές τους.

Αυτό που μας δυσκόλεψε ήταν η εγκατάσταση του gem5, μιας και σε μερικά συστήματα χρειαζόταν η εγκατάσταση της zlib πρώτα. Η εντολή build που δόθηκε για την εκτέλεση του C προγράμματος δεν δούλευε, γιατί απουσίαζε το flag -c και το --cpu=MinorCPU πρέπει να αντικατασταθεί με το --cpu-type=MinorCPU.

Επίσης, δεν μας ήταν σαφές τι να συγκρίνουμε στο ερώτημα 2.b.

Δεν θεωρώ ότι η εργασία είναι απλοϊκή, ούτε αδικαιολόγητα δύσκολη. Τα ερωτήματα είναι εύστοχα, ώστε να μας ωθήσουν να πειραματιστούμε.

Τέλος, θα βοηθούσε σημαντικά στην κατανόηση και στην ευκολότερη εξαγωγή συμπερασμάτων, αν γινόταν ένα προ-εργαστηριακό μάθημα, με σκοπό να γίνει μία ανάλυση της πληροφορίας που παράγεται στα αρχεία stats και config.

#### FOLDER STRUCTURE

```
.
├── m5out								# Stats of hello world example
│   ├── config
|   ├── config.json
│   └── stats.txt
├── MyProgram MinorCPU
│   ├── config
|   ├── config.json
│   └── stats.txt
├── MyProgram MinorCPU 500MHz
│   ├── config
|   ├── config.json
│   └── stats.txt
├── MyProgram MinorCPU --l2cache
│   ├── config
|   ├── config.json
│   └── stats.txt
├── MyProgram TimingSImpleCPU
│   ├── config
|   ├── config.json
│   └── stats.txt
├── MyProgram TimingSimpleCPU 500MHz
│   ├── config
|   ├── config.json
│   └── stats.txt
├── MyProgram TimingSimpleCPU --l2cache
│   ├── config
|   ├── config.json
│   └── stats.txt
├── cProgram.c							# Example program in C                      
├── ccc                                 # Cross compiled executable                       
└── README.MD                    
```

