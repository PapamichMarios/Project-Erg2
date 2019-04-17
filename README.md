# Project-Erg2
Clustering algorithms K-means &amp; K-medoids using C++.
Υλοποίηση αλγορίθμων για τη συσταδοποίηση διανυσμάτων στο d-διάστατο χώρο R με τη χρήση αποστάσεων Euclidean και Cosine, συμπεριλαμβάνοντας 12 συνδυασμούς:

- Initialisation
  1. Random Selection of k points
  2. K-means ++
- Assignment
  1. Lloyd's assignment.
  2. Assignment by range search with LSH
  3. Assignment by range search with Hypercube
- Update
  1. K-means
  2. Partitioning Around Medoids (PAM)improved like Lloyd's

# Author
- Μάριος Παπαμιχαλόπουλος 
- 1115201400149

# Tools used
- Ubuntu Gnome 17.10
- git
- Valgrind 3.13.0
- C++11
- Google Test C++11 framework

# Compilation
```sh
make
```
# Ενδεικτικές εκτελέσεις
```sh
./cluster -o <output_file> -c cluster.conf -d euclidean -i <input_file>
```

```sh
./cluster -o <output_file> -c cluster.conf -d cosine -i <input_file>
```

```sh
./cluster -o <output_file> -c cluster.conf -d <metric> -i <input_file> -complete
```

# Περιγραφή προγράμματος

Kατά την πρώτη φάση του προγράμματος διαβάζω τα configuratios απο το cluster.conf και τα δεδομένα που δίνονται σαν input. Επίσης, φτιάχνω τις δομές που χρειάζονται για το lsh και το hypercube. Αφού αρχικοποιήσω όλα αυτά ξεκινάω τη διαδικασία του clustering.

Ακολουθεί η διαδικασία του clustering, που αποτελείται από 3 φάσεις. Το initialization, το assignment και το update. Για το clustering συγκρίνω τη μεταβολή των objective function για το εκάστοτε τύπο clustering(k-means, kmedoids) και όταν αυτή είναι μικρότερη του 1% σταματάω τη διαδικασία. Πολλές φορές, στο PAM a la loyds, παρατηρείται η εναλλαγή των ίδιων medoids. Π.χ. έχουμε σε ένα loop 5 centroids, τα κάνουμε swap με 5 medoids και μετά στο επόμενο loop ξαναείναι τα αρχικά centroids. Καταλήγουμε σε έναν ατέρμονα βρόγχο. Γι' αυτό το λόγο πέρα από τη μεταβολή των objective function, σαν συνθήκη τερματισμού του clustering έχω hard coded να τερματίζει η διαδικασία μετά από MAX_PROCESS_LOOPS(=20). 

Mόλις τελειώσει το clustering, υπολογίζω το silhouette για το εκάστοτε combination απο initialization, assignment και update. Στο σύνολο είναι 12, όπως περιγράφεται στην άσκηση. Εκτυπώνω τα αποτελέσματα της συσταδοποίησης στο αρχείο που δίνεται σαν όρισμα κατα την εκτέλεση του προγράμματος και συνεχίζω με τον επόμενο συνδυασμό.

To configuration file μου, cluster.conf περιέχει τις παρακατω ρυθμίσεις, με την ίδια σειρά που περιγράφονται παρακάτω:
- Αριθμός συστάδων.
- Αριθμός των σημείων που επιλέγονται απο το dataset για το initialization++.
- Αριθμός των hash functionas, για το LSH.
- Αριθμός των hash tables L, για το LSH.
- Διάσταση προβολής των σημείων για τον υπερκύβο.
- Μέγιστο επιτρεπόμενο πλήθος των υποψήφιων σημείων που θα ελεγχθούν για τον υπερκύβo.
- Αριθμός probes για τον υπερκύβο. τον υπερκύβο. 

# Περιγραφή αρχείων & διεπαφών

**1. Νέα αρχεία**

- **utilities.cpp:** Aρχείο που περιέχει βοηθητικές συναρτήσεις προκειμένου η main να είναι πιο compact και ευκολοδιαβαστη. 

- **initialization.cpp:** Αρχείο που περιέχει τις συναρτήσεις που σχετίζονται με το πρώτο βήμα της συσταδοποίησης, δηλαδή αυτό της αρχικοποίησης των κεντροειδών σημείων. Περιέχει 2 συναρτήσεις τη RandomSelection() και τη k_meanspp(). Η πρώτη αρχικοποιεί τα κεντροειδή με τυχαία σημεία απο το dataset, ενώ η δεύτερη ακολουθεί τη διαδικασία initialization++, όπως περιγράφεται στις διαφάνειες.

- **assignment.cpp:** Αρχείο που περιέχει τις συναρτήσεις που σχετίζονται με το δεύτερο βήμα της συσταδοποίησης, δηλαδή αυτό της ανάθεσης των σημείων στις συστάδες στις οποίες βρίσκονται πιο κοντά. Περιέχει 3 σνυναρτήσεις. Η πρώτη είναι η loyds() που υπολογίζει για ένα σημείο τις αποστάσεις από τα κεντροειδή και το κατατάσει στη συστάδα του κεντροειδούς που βρίσκεται πίο κοντά. Η δεύτερη lsh() κοιτάει σε ποιο bucket, hash-άρονται τα κεντροειδή και μέσω range search(ακτίνα που διπλασιάζεται όσο MAX_TIME_RADIUS(=5)) παίρνει αυτά των οποίων η απόσταση είναι μικρότερη απο το range. Σε περίπτωση collision, τα σημεία αναθέτονται στο κοντινότερο κεντροειδές σημείο. Τέλος, η συναρτηση hypercube(), η οποία κοιτάει σε ποια κορυφή hash-άρονται τα centroids και κάνει ουσιαστικά ό,τι και η lsh. Και για τις 2 αυτές συναρτήσεις χρησιμοποιώ μια τροποποιημένη range search συναρτηση απο την πρώτη εργασία.

- **update.cpp:** Άρχείο που περιέχει τις συναρτήσεις που σχετίζονται με το τρίτο και τελευταίο βήμα της συσταδοποίησης, δηλαδή αυτό της ανανέωσης των κεντροειδών. Περιέχει 3 συναρτήσεις. Την k_means() που είναι το κλασικό update που για όλα τα σημεία του cluster, δημιουργείται ένα νέο κεντροειδές ως αποτέλεσμα του μέσου όρου των σημείων. Την PAM_a_la_loys() που είναι η διαδικασία με τα medoids αλλα με παραλλαγή loyds, όπως προτάθηκε στο φροντιστήριο και η Silhouette(), η συνάρτηση για τον υπολογισμό της αποτελεσματικότητας του clustering.


- **metric.h:** Αρχείο που περίεχει μια abstract class, που ανάλογα τον τύπο της μετρικής(euclidean ή cosine) που δίνεται από τη γραμμή εντολών υπολογίζει και την κατάλληλη απόσταση κατα τη διάρκεια του προγράμματος.

- **cluster.conf:** Aρχείο διαφόρων ρυθμίσεων για το πρόγραμμα.
 
- **cluster.cpp:** Το αρχείο που βρίσκεται η main συνάρτηση. Στην αρχή παίρνει διάφορα στοιχεία όπως τα inline args και τα configuration args και συνεχίζει με τη διαδικασία του clustering, εκτυπώνοντας τα αποτελέσματα στο <output_file> που δίνει ο χρήστης.


**2. Αρχεία απο την 1η εργασία:**

- **hash_table.h:** **Template** που περιέχει 3 κλάσεις. Μια κλάση **γονιός HashTable** που περιέχει pure virtual μεθόδους σχετικα με την τοποθέτηση των στοιχείων και την αναζήτηση sto hash table και οι δύο κλάσεις **παιδιά HashTable_EUC και HashTable_COS**. 
- **hash_node.h:** **Template** που περιγράφει τα nodes του hash table.
- **hyper_cube.h:** Ομοίως ότι και στο hash_table.h μόνο που τώρα έχουμε **HyperCube γονιό** κλάση και **HyperCube_EUC, HyperCube_COS παιδιά**.
- **hyper_node.h:** **Template** που περιγράφει τα nodes του hypercube.

-  **hyperplane.cpp:** Αρχείο που περιέχει 3 κλάσεις που χρησιμοποιούνται για τον υπολογισμό των h. Υπάρχει η κλάση γονιός Hyperplane που περιέχει pure virtual μεθόδους και οι δύο κλάσεις παιδία Hyperplane_EUC και Hyperplane_COS. Η κλάση Hyperplane περιέχει πάντα το τυχαίο διάνυσμα που προκύπτει από κανονική κατανομή (0,1). Η Hyperplane_EUC επεκτείνει την Hyperplane αποθηκέυοντας το t[0,w] και το w. 

- **fi.cpp:** Κλάση που χρησιμοποιείται ως **hash_function** για το euclidean LSH και το euclidean binary cube. Για το LSH, αυτή η κλάση χρησιμοποιείται για τον υπολογισμό του hash_value, που βασίζεται στον τυπο **φ** των διαφανειών, ο οποίος είναι το άθροισμα των γινομένων r με τα h mod tableSize. Για να γίνει αυτό καλέι τη συνάρτηση computeH().
Για το cube, αυτή η συνάρτηση απλά μετατρέπει το h σε 0 ή 1.

- **gi.cpp:** Κλάση που χρησιμοποιείται ως **hash_function** για το cosine LSH και το cosine binary cube.  Αυτή η κλάση, χρησιμοποιείται για τον υπολογισμό του hash_value, που βασίζεται σε random projections στο χώρο. Το hash_value είναι concat των 0 ή 1 που προκύπτουν από το εσωτερικό γινόμενο των random projections με το διάνυσμα.

- **lsh.cpp:** 
-- Αρχικά, γίνεται η επεξεργασία των παραμέτρων που χρησιμοποιούνται κατα την κληση τον προγράμματος. 
-- Υπολογίζονται οι διάφορες χρήσιμες μεταβλητές **type, tableSize και dimensions**, με τη βοήθεια κατάλληλων συναρτήσεων του namespace που ορίζεται στο help_function.h. Το type ισούται με **“EUC”** ‘η **“COS”** ανάλογα την πρώτη γραμμή του **<input_file>**. Με βάση το type υπολογίζεται και το tableSize. 
-- Δημιουργείται το hash_table μέσω της χρήσης πολυμορφισμού. Ορίζουμε μια μεταβλητη hash_tableptr που είναι δείκτης σε δείκτη σε HashTable<K>. Ανάλογα το type δημιουργείται και ο κατάλληλος τύπος hash_table.
-- Ύστερα, ξεκινάμε το διάβασμα του **<input_file>** γραμμή γραμμή. Με τη χρήση ενός vector αποθηκεύουμε το σημείο που διαβάζουμε και το τοποθετούμε στη δομή μας.
-- Ξεκινάμε το διάβασμα του **<query_file>** γραμμή γραμμη, όπως και το διάβασμα του **<input_file>**. Κάνουμε τα search που απαιτούνται(NN, ANN, RS) σε κάθε πίνακα L και τα εκτυπώνουμε στο <output_file>. Στο τέλος, εκτυπώνουμε το κλάσμα προσέγγισης και το μέσο χρόνο ANN.

- **cube.cpp:** 
-- Αρχικά, γίνεται η επεξεργασία των παραμέτρων που χρησιμοποιούνται κατα την κληση τον προγράμματος. 
-- Υπολογίζονται οι διάφορες χρήσιμες μεταβλητές **type, tableSize και dimensions**, με τη βοήθεια κατάλληλων συναρτήσεων του namespace που ορίζεται στο help_function.h. Το type ισούται με **“EUC”** ‘η **“COS”** ανάλογα την πρώτη γραμμή του <input_file>. Με βάση το type υπολογίζεται και το tableSize. 
-- Δημιουργείται το hyper_cube μέσω της χρήσης πολυμορφισμού. Ορίζουμε μια μεταβλητη hyper_cubeptr που είναι δείκτης σε δείκτη σε HyperCube<K>. Ανάλογα το type δημιουργείται και ο κατάλληλος τύπος hyper_cube.
-- Ύστερα, ξεκινάμε το διάβασμα του **<input_file>** γραμμή γραμμή. Με τη χρήση ενός vector αποθηκεύουμε το σημείο που διαβάζουμε και το τοποθετούμε στη δομή μας.
-- Ξεκινάμε το διάβασμα του **<query_file>** γραμμή γραμμη, όπως και το διάβασμα του **<input_file>**. Κάνουμε τα search που απαιτούνται(NN, ANN, RS) και τα εκτυπώνουμε στο **<output_file>**. Στο τέλος, εκτυπώνουμε το κλάσμα προσέγγισης και το μέσο χρόνο ANN.

- **help_functions.h:** Διάφορες χρήσιμες συναρτήσεις για να ναι το πρόγραμμα πιο ευκολοδιάβαστο.

# Aποτελέσματα clustering για διαφορετικά k & παρατηρήσεις

| k=3 initialization++ points=20  |      I1A1U1      | I1A1U2          | I1A2U1           | I1A2U2          | I1A3U1           | I1A3U2          | I2A1U1           | I2A1U2          | I2A2U1           | I2A2U2          | I2A3U1           | I2A3U2          |
|---------------------------------|:----------------:|-----------------|------------------|-----------------|------------------|-----------------|------------------|-----------------|------------------|-----------------|------------------|-----------------|
| χρόνος                          | 0.245431 seconds | 26.0255 seconds | 0.359637 seconds | 26.4246 seconds | 0.270093 seconds | 33.6215 seconds | 0.275117 seconds | 33.9901 seconds | 0.461018 seconds | 28.7626 seconds | 0.226329 seconds | 27.0111 seconds |
| silhouette                      | 0.0307987        | 0.037378        | 0.0328238        | 0.0231781       | 0.0330254        | 0.0566986       | 0.0376822        | 0.0421266       | 0.0157147        | 0.0387717       | 0.0376367        | 0.0257953       |
|                                 |                  |                 |                  |                 |                  |                 |                  |                 |                  |                 |                  |                 |
| χρόνος                          | 0.355066 seconds | 18.9497 seconds | 1.93974 seconds  | 13.7843 seconds | 0.50629 seconds  | 13.596 seconds  | 0.166568 seconds | 14.4567 seconds | 1.63845 seconds  | 18.6107 seconds | 0.675157 seconds | 21.0766 seconds |
| silhouette                      | 0.043684         | 0.0426254       | 0.0467553        | 0.0304947       | 0.045327         | 0.035634        | 0.0452312        | 0.0440269       | 0.0487827        | 0.0441833       | 0.0499319        | 0.0455252       |

| k=5 initialization++ points=25 |      I1A1U1      | I1A1U2          | I1A2U1           | I1A2U2          | I1A3U1           | I1A3U2          | I2A1U1           | I2A1U2          | I2A2U1           | I2A2U2          | I2A3U1           | I2A3U2          |
|--------------------------------|:----------------:|-----------------|------------------|-----------------|------------------|-----------------|------------------|-----------------|------------------|-----------------|------------------|-----------------|
| χρόνος                         | 0.455391 seconds | 23.7009 seconds | 0.543174 seconds | 30.5392 seconds | 0.546145 seconds | 23.6992 seconds | 0.325556 seconds | 58.3495 seconds | 0.344169 seconds | 56.4401 seconds | 0.385591 seconds | 56.5718 seconds |
| silhouette                     | 0.0452401        | 0.0391287       | 0.0154151        | 0.0334701       | 0.0452401        | 0.0391287       | 0.0395982        | 0.0621793       | 0.0349264        | 0.0611648       | 0.0395982        | 0.0621793       |
|                                |                  |                 |                  |                 |                  |                 |                  |                 |                  |                 |                  |                 |
| χρόνος                         | 0.466975 seconds | 9.20072 seconds | 3.40109 seconds  | 9.90568 seconds | 1.15832 seconds  | 9.30196 seconds | 0.471255 seconds | 9.36874 seconds | 4.24773 seconds  | 9.46121 seconds | 1.27141 seconds  | 9.08807 seconds |
| silhouette                     | 0.0548233        | 0.0435577       | 0.0548233        | 0.0435577       | 0.0548233        | 0.0435577       | 0.0550521        | 0.0466107       | 0.0550521        | 0.0466107       | 0.0550521        | 0.0466107       |

| k=10 initialization++ points=50 |      I1A1U1      | I1A1U2          | I1A2U1           | I1A2U2          | I1A3U1           | I1A3U2          | I2A1U1           | I2A1U2          | I2A2U1           | I2A2U2          | I2A3U1           | I2A3U2          |
|--------------------------------|:----------------:|-----------------|------------------|-----------------|------------------|-----------------|------------------|-----------------|------------------|-----------------|------------------|-----------------|
| χρόνος                         | 0.401843 seconds | 45.9895 seconds | 0.567501 seconds | 29.2313 seconds | 0.503881 seconds | 46.0574 seconds | 0.521479 seconds | 31.2825 seconds | 0.581116 seconds | 28.2363 seconds | 0.613612 seconds | 31.2448 seconds |
| silhouette                     | 0.0565317        | 0.0548776       | 0.0218279        | 0.0413856       | 0.0565317        | 0.0548776       | 0.0507464        | 0.510102        | 0.0305604        | 0.280020        | 0.0507464        | 0.302010        |
|                                |                  |                 |                  |                 |                  |                 |                  |                 |                  |                 |                  |                 |
| χρόνος                         | 0.545651 seconds | 8.02452 seconds | 9.79417 seconds  | 10.291 seconds  | 3.50877 seconds  | 8.72695 seconds | 0.653569 seconds | 8.93452 seconds | 10.3215 seconds  | 10.6163 seconds | 3.7219 seconds   | 9.13238 seconds |
| silhouette                     | 0.0635473        | 0.0417903       | 0.0635473        | 0.0417903       | 0.0635473        | 0.0417903       | 0.0645577        | 0.0467737       | 0.0645577        | 0.0467737       | 0.0645577        | 0.0467737       |

| k=50 initialization++ points=150 |      I1A1U1     | I1A1U2          | I1A2U1          | I1A2U2          | I1A3U1          | I1A3U2          | I2A1U1          | I2A1U2          | I2A2U1          | I2A2U2          | I2A3U1          | I2A3U2          |
|----------------------------------|:---------------:|-----------------|-----------------|-----------------|-----------------|-----------------|-----------------|-----------------|-----------------|-----------------|-----------------|-----------------|
| χρόνος                           | 2.24152 seconds | 41.1907 seconds | 2.18276 seconds | 37.3612 seconds | 2.92735 seconds | 44.6688 seconds | 2.45408 seconds | 34.8005 seconds | 2.16663 seconds | 30.2727 seconds | 2.87485 seconds | 34.816 seconds  |
| silhouette                       | 0.0849015       | 0.058257        | 0.0645039       | 0.0629769       | 0.084315        | 0.058257        | 0.0933014       | 0.0646677       | 0.0669441       | 0.0545661       | 0.0933014       | 0.0646677       |
|                                  |                 |                 |                 |                 |                 |                 |                 |                 |                 |                 |                 |                 |
| χρόνος                           | 2.16465 seconds | 2.41262 seconds | 225.501 seconds | 149.378 seconds | 73.9312 seconds | 50.271 seconds  | 2.1396 seconds  | 1.7759 seconds  | 183.926 seconds | 51.6082 second  | 61.2698 seconds | 19.1377 seconds |
| silhouette                       | 0.129579        | 0.0916887       | 0.12056         | 0.090251        | 0.129579        | 0.092510        | 0.14126         | 0.102926        | 0.14126         | 0.1020510       | 0.15202         | 0.102951        |

Από τις διάφορες μετρήσεις που έκανα και όχι μόνο αυτές που παρουσίασα παρατηρώ ότι:
- Τις περισσότερες φορές, το clustering με αρχικοποίηση τον αλγόριθμο initialization++ δίνει καλύτερα αποτελέσματα από το να διαλέξουμε τυχαία κάποια σημεία απο το dataset. Βέβαια επειδή και οι δύο αρχικοποιήσεις είναι τυχαιοκρατικές, μπορούν να τύχουν και κακά αρχικά κεντροειδή. Αυτήν την τυχαιότητα προσπαθεί να φτιάξει και η μεθοδος initialization++. Eπίσης, μεγάλη διαφορά παρατηρείται μεταξύ του k-means με τον k-medoids σε μικρά clusters. 
- Το PAM a la loyds είναι πολύ πιο χρονοβόρο απο το απλό k-means.
- Σε μικρά clusters, τα silhouette είναι σχεδόν ίδια και για τα 2 updates. Σε μεγαλύτερα, παρατηρώ ότι το update με k-means είναι καλύτερο από το k-medoids, άρα πιστεύω ότι τα στοιχεία του dataset δεν χαρακτηρίζονται από ακραία σημεία στο χώρο, γιατί αυτά ειναι ένας από τους λόγους που επηρεάζουν τη λειτουργια του k-means και δίνει κακά αποτελέσματα.
