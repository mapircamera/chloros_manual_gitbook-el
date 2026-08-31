# Χρήση του Chloros με βοηθούς τεχνητής νοημοσύνης

Αυτό το εγχειρίδιο απευθύνεται σε δύο ομάδες: τους ανθρώπους και τους βοηθούς τεχνητής νοημοσύνης, μέσω των οποίων οι άνθρωποι εργάζονται όλο και περισσότερο. Κάθε σελίδα περιλαμβάνει ακριβείς τιμές, προεπιλογές και εντολές που μπορούν να αντιγραφούν και να επικολληθούν, ώστε ένας βοηθός (Claude, ChatGPT, Copilot, ένας πράκτορας προγραμματισμού, …) να μπορεί να δημιουργήσει λειτουργική αυτοματοποίηση Chloros από την πρώτη προσπάθεια.

Έκδοση Chloros: **

1.2.0**. Πλατφόρμες CLI/SDK: Windows 10/11 x64 και Linux (x86\_64 / Jetson aarch64).

## Τι να δώσετε στον βοηθό σας

| Πόρος | URL | Για ποιο σκοπό |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Ευρετήριο κάθε σελίδας αυτού του εγχειριδίου, αναγνώσιμο από υπολογιστή. |
| **CLI Αναφορά** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Η πλήρης επιφάνεια εντολών του `chloros-cli`: κάθε εντολή, σημαία, προεπιλογή, κωδικός εξόδου και κανόνας φακέλου εξόδου. Συντάχθηκε για χρήση από μεγάλα γλωσσικά μοντέλα (LLM). |
| **Αναφορά SDK** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | Το πλήρες `chloros_sdk` Python API: κλάσεις, υπογραφές, εξαιρέσεις και παραδείγματα εφαρμογής. Γραμμένο για χρήση από LLM. |
| **Οποιαδήποτε σελίδα ως ακατέργαστο Markdown** | προσθέστε το `.md` στη σελίδα URL | π.χ. Το `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` επιστρέφει τη σελίδα ως ακατέργαστο Markdown — ιδανικό για επικόλληση σε παράθυρο περιβάλλοντος ή λήψη από έναν πράκτορα. |

Σύνδεσμοι εντός του εγχειριδίου: [CLI Αναφορά](reference/cli-reference.md) · [Αναφορά SDK](reference/sdk-reference.md).

{% hint style="info" %}
Οι δύο σελίδες αναφοράς είναι αυτόνομες: ένας βοηθός που έχει διαβάσει μία από αυτές δεν χρειάζεται το υπόλοιπο εγχειρίδιο για να γράψει ένα σωστό σενάριο.
{% endhint %}

## Έτοιμες συνταγές

Αντιγράψτε, συμπληρώστε το `<placeholders>` και επικολλήστε το στον βοηθό σας.

### 1. Επεξεργασία ενός φακέλου πτήσεων στο NDVI

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Μαζική παρακολούθηση ενός καταλόγου καταγραφών

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Σύνδεση πίνακα LATTICE και καταγραφή

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. Καταγράψτε φάσματα αισθητήρα φωτός DAQ

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
Η δημιουργία σεναρίων DAQ από τη γραμμή εντολών πραγματοποιείται πάντα μέσω της οικογένειας `daq pool-*` (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Άλλες υποεντολές `daq` που μπορεί να επινοήσει ο βοηθός σας δεν είναι διαθέσιμες στις εκδόσεις που διατίθενται και τερματίζονται με σφάλμα.
{% endhint %}

## Γιατί τα σενάρια που γράφονται από AI λειτουργούν καλά με το Chloros

Κάθε μία από αυτές είναι μια πραγματική, επαληθευμένη συμπεριφορά του Chloros 1.2.0 — εξαλείφουν τους κλασικούς τρόπους αποτυχίας της αυτοματοποίησης που γράφεται από μηχανές:

* **Χωρίς περίπλοκες ρυθμίσεις.**Τα βοηθητικά προγράμματα έξυπνης σύνδεσης του SDK (`connect_camera`, `connect_array`, `connect_daq_sensor`) και τα σημεία εισόδου επεξεργασίας (`ChlorosLocal`, `process_folder`)**εκκινούν αυτόματα το τοπικό backend**. Ένα παραγόμενο σενάριο δεν χρειάζεται το GUI ανοιχτό ή έναν διακομιστή που έχει ξεκινήσει χειροκίνητα — χρειάζεται μόνο να είναι εγκατεστημένο το πακέτο desktop/CLI.
* **Ολόκληρη η ροή αποτελείται από μία μόνο κλήση.** Το `chloros_sdk.process_folder("path", indices=["NDVI"])` εκτελεί εισαγωγή → βαθμονόμηση → ανακλαστικότητα → εξαγωγή δείκτη από την αρχή έως το τέλος. Λιγότερες επιφάνειες, λιγότερα σημεία όπου ένα δημιουργημένο σενάριο μπορεί να παρουσιάσει σφάλμα.
* **Οι εκτελέσεις χωρίς έξοδο αυτοδιαγιγνώσκονται.** Μετά το `process()`, η περίληψη της εκτέλεσης επισυνάπτεται στο αποτέλεσμα, και κάθε υπόδειξη επεξεργασίας (π.χ. *γιατί* μια εκτέλεση δεν παρήγαγε έξοδο) αναμεταδίδεται επίσης ως Python `UserWarning` — έτσι, ακόμη και ένα σενάριο που δεν ελέγχει ποτέ το λεξικό αποτελεσμάτων εμφανίζει τη διάγνωση.
* **Το CLI αποτυγχάνει με εμφανή τρόπο.**Μια εκτέλεση `chloros-cli process` που ζήτησε αποτελέσματα αλλά δεν έγραψε κανένα, εκτυπώνει το `Processing finished but wrote no image products.` και**τερματίζεται με μη μηδενικό κώδικα εξόδου**, οπότε τα σενάρια shell και το CI το ανιχνεύουν με έναν απλό έλεγχο του κώδικα εξόδου. Οι επιτυχημένες εκτελέσεις αναφέρουν το `Image products written: N`.

Μια ασυμμετρία που πρέπει να γνωρίζει ένας βοηθός: το `process()` του SDK **δεν** ενεργοποιείται σκόπιμα σε μια εκτέλεση χωρίς προϊόντα — αντ&#x27; αυτού, αναφέρει το αποτέλεσμα μέσω της σύνοψης/των υποδείξεων. Εάν ένας αγωγός Python πρέπει να σταματήσει σε μια κενή εκτέλεση, ελέγξτε τη σύνοψη (η συνταγή 2 το κάνει).

## Προειδοποιήσεις

* **Απαιτείται σύνδεση στο Chloros+.**Τα CLI και SDK απαιτούν**πληρωμένο** επιπέδου Chloros+, το οποίο επιβάλλεται από την πλευρά του διακομιστή: οι αιτήσεις αποτυγχάνουν με κωδικό σφάλματος `401 AUTH_REQUIRED` όταν δεν έχετε συνδεθεί και με κωδικό σφάλματος `403 PLAN_UPGRADE_REQUIRED` στο δωρεάν επίπεδο. Εκτελέστε το `chloros-cli login` μία φορά ανά μηχανή πριν εκτελέσετε τα δημιουργημένα σενάρια. Δείτε [Chloros+ Σύνδεση](chloros+-login.md).
* **Οι εντολές καταγραφής ελέγχουν πραγματικό υλικό.** Οι εντολές `lattice` / `daq` / `project` και τα αντικείμενα συνεδρίας SDK συνδέουν, μεταδίδουν ροή και ενεργοποιούν φυσικές κάμερες και αισθητήρες. Ελέγξτε ένα σενάριο που έχει δημιουργηθεί πριν από την πρώτη του εκτέλεση και εκτελέστε το με την παρουσία του υλικού.
* **Ελέγξτε δειγματοληπτικά τα αποτελέσματα.** Επαληθεύστε τους φακέλους των προϊόντων και μερικές τιμές εικονοστοιχείων πριν δημοσιεύσετε τα αποτελέσματα. Ειδικότερα, τα αρχεία TIFF ανακλαστικότητας κλιμακώνονται ανά πηγή — διαβάστε την ετικέτα XMP του `Chloros:PixelScale` (LATTICE: 32768 = 1,0 ανακλαστικότητα; Survey3: 65535) αντί να υποθέσετε έναν διαιρέτη. Και οι δύο σελίδες αναφοράς το τεκμηριώνουν στην ενότητα «Ανάγνωση εικονοστοιχείων ανακλαστικότητας».
* **Μικρές παγίδες που προκαλούν σφάλματα στον παραγόμενο κώδικα:**Το `pool-record` εγγράφει στο σύστημα αρχείων του**κεντρικού υπολογιστή του backend** (προεπιλογή `~/Documents/DAQ Live View/`)· σε μηχανές με πολλές διεπαφές δικτύου, προτιμήστε το `daq pool-connect --eth-host <ip-or-hostname>` αντί της αυτόματης ανίχνευσης· και χρησιμοποιήστε το `http://127.0.0.1:5000` (ποτέ το `localhost`) οπουδήποτε εμφανίζεται ένα backend URL.
