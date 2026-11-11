# Χρήση του Chloros με βοηθούς τεχνητής νοημοσύνης

Αυτό το εγχειρίδιο απευθύνεται σε δύο είδη κοινού: τους ανθρώπους και τους βοηθούς τεχνητής νοημοσύνης (AI) με τους οποίους οι άνθρωποι συνεργάζονται όλο και περισσότερο. Κάθε σελίδα περιλαμβάνει ακριβείς τιμές, προεπιλογές και εντολές που μπορούν να αντιγραφούν και να επικολληθούν, ώστε ένας βοηθός (Claude, ChatGPT, Copilot, ένας πράκτορας προγραμματισμού, …) να μπορεί να γράψει λειτουργική αυτοματοποίηση Chloros από την πρώτη προσπάθεια.

Έκδοση Chloros: **

1.3.0**. Πλατφόρμες CLI/SDK: Windows 10/11 x64 και Linux (x86\_64 / Jetson aarch64).

## Τι να δώσετε στον βοηθό σας

| Πόρος | URL | Για τι χρησιμεύει |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Ευρετήριο αναγνώσιμο από μηχανή για κάθε σελίδα αυτού του εγχειριδίου. |
| **Αναφορά CLI** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Η πλήρης επιφάνεια εντολών `chloros-cli`: κάθε εντολή, σημαία, προεπιλογή, κωδικός εξόδου και κανόνας φακέλου εξόδου. Γράφτηκε για χρήση από το LLM. |
| **Αναφορά SDK** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | Το πλήρες Python API του `chloros_sdk`: κλάσεις, υπογραφές, εξαιρέσεις και παραδείγματα με λύσεις. Συντάχθηκε για χρήση από LLM. |
| **Οποιαδήποτε σελίδα ως ακατέργαστο Markdown** | προσθέστε το `.md` στη διεύθυνση URL της σελίδας | π.χ. το `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` επιστρέφει τη σελίδα ως ακατέργαστο Markdown — ιδανικό για επικόλληση σε παράθυρο περιβάλλοντος ή λήψη από έναν πράκτορα. |

Σύνδεσμοι εντός του εγχειριδίου: [Αναφορά CLI](reference/cli-reference.md) · [Αναφορά SDK](reference/sdk-reference.md).

{% hint style="info" %}
Οι δύο σελίδες αναφοράς είναι αυτόνομες: ένας βοηθός που έχει διαβάσει μία από αυτές δεν χρειάζεται το υπόλοιπο εγχειρίδιο για να γράψει ένα σωστό σενάριο.
{% endhint %}

## Αφήστε τον βοηθό σας να εκτελέσει απευθείας το Chloros: `chloros-mcp`

Όλα τα παραπάνω απαιτούν από τον βοηθό σας να *γράψει* ένα σενάριο ή μια εντολή. Με το `chloros-mcp`, ένας βοηθός που χρησιμοποιεί το [Model Context Protocol](https://modelcontextprotocol.io) (MCP), όπως το Claude Desktop, το Claude Code ή το Cursor, χειρίζεται το ίδιο το Chloros. Ζητάτε με απλή γλώσσα και αυτό καλεί τα εργαλεία του διακομιστή:

* **Επεξεργασία εικόνων:** δημιουργήστε ένα έργο, εισαγάγετε έναν φάκελο με λήψεις, επιλέξτε ρυθμίσεις, επεξεργαστείτε και αναφέρετε τα αποτελέσματα της εκτέλεσης.
* **Λειτουργία υλικού:** σύνδεση καμερών LATTICE, συστοιχιών και αισθητήρων φωτός DAQ, προεπισκόπηση, ρύθμιση έκθεσης και χρώματος, λήψη, καταγραφή και ενημέρωση υλικολογισμικού.

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.
```

Τα εργαλεία του διακομιστή καλύπτουν τις λειτουργίες του [Python SDK](api-python-sdk.md). Εγκαθίσταται μαζί με το SDK, οπότε συνήθως δεν χρειάζεται να εγκαταστήσετε τίποτα άλλο: στα Windows, το πρόγραμμα εγκατάστασης του Chloros τοποθετεί το `chloros-mcp.exe` στο φάκελο `Scripts` του Python σας, ενώ στο Linux, το `.deb` τοποθετεί το `chloros-mcp` στο `/usr/bin`.

**Κώδικας Claude:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Claude Desktop:**προσθέστε αυτό στο `claude_desktop_config.json` (το ανοίγετε από**Ρυθμίσεις &gt; Προγραμματιστής &gt; Επεξεργασία διαμόρφωσης**), και στη συνέχεια επανεκκινήστε το Claude Desktop. Το**Cursor** χρησιμοποιεί την ίδια καταχώριση στο `~/.cursor/mcp.json`.

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

Εάν ο πελάτης αναφέρει ότι ο διακομιστής δεν μπόρεσε να ξεκινήσει, το `chloros-mcp` πιθανώς δεν βρίσκεται στο `PATH` που βλέπει: δώστε την πλήρη διαδρομή αντ&#x27; αυτού (`where chloros-mcp` στα Windows, `which chloros-mcp` στο Linux).

Πρέπει να έχετε εγκαταστήσει το Chloros στον ίδιο υπολογιστή και να έχετε συνδεθεί εκεί, είτε μέσω της εφαρμογής είτε με το `chloros-cli login`, σε οποιοδήποτε επί πληρωμή πακέτο Chloros+ (Copper ή ανώτερο). Ο διακομιστής δεν μπορεί να συνδεθεί εκ μέρους σας.

{% hint style="info" %}
Ρυθμίσεις πελάτη για κάθε εφαρμογή, επιλογές, κάθε εργαλείο και πώς λειτουργεί μια εκτέλεση: [MCP Server (AI Assistants)](reference/sdk-reference.md#mcp-server-ai-assistants) στην Αναφορά SDK.
{% endhint %}

## Προτεινόμενες εντολές

Αντιγράψτε, συμπληρώστε το `<placeholders>` και επικολλήστε το στον βοηθό σας.

### 1. Επεξεργασία φακέλου πτήσης σε NDVI

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

### 3. Σύνδεση συστοιχίας LATTICE και καταγραφή

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. Καταγραφή φασμάτων αισθητήρα φωτός DAQ

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
Η δημιουργία σεναρίων DAQ από τη γραμμή εντολών πραγματοποιείται πάντα μέσω της οικογένειας `daq pool-*` (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Άλλες υποεντολές του τύπου `daq` που ενδέχεται να επινοήσει ο βοηθός σας δεν είναι διαθέσιμες στις έκδοσεις που διατίθενται και οδηγούν σε σφάλμα.
{% endhint %}

## Γιατί τα σενάρια που γράφονται από τεχνητή νοημοσύνη λειτουργούν καλά με το Chloros

Κάθε μία από αυτές είναι μια πραγματική, επαληθευμένη συμπεριφορά του Chloros 1.3.0 — εξαλείφουν τους κλασικούς τρόπους αποτυχίας της αυτοματοποίησης που γράφεται από μηχανές:

* **Χωρίς περίπλοκες ρυθμίσεις.**Οι βοηθητικές λειτουργίες έξυπνης σύνδεσης του SDK (`connect_camera`, `connect_array`, `connect_daq_sensor`) και τα σημεία εισόδου επεξεργασίας (`ChlorosLocal`, `process_folder`)**εκκινούν αυτόματα το τοπικό backend**. Ένα παραγόμενο σενάριο δεν χρειάζεται ανοιχτό GUI ή διακομιστή που έχει ξεκινήσει χειροκίνητα — χρειάζεται μόνο να είναι εγκατεστημένο το πακέτο για επιτραπέζιους υπολογιστές ή CLI.
* **Ολόκληρη η ροή αποτελείται από μία μόνο κλήση.** Το `chloros_sdk.process_folder("path", indices=["NDVI"])` εκτελεί από την αρχή έως το τέλος τις διαδικασίες εισαγωγής → βαθμονόμησης → ανακλαστικότητας → εξαγωγής δείκτη. Μικρότερη επιφάνεια, λιγότερα σημεία όπου ένα δημιουργημένο σενάριο μπορεί να παρουσιάσει σφάλματα.
* **Οι εκτελέσεις χωρίς έξοδο αυτοδιαγιγνώσκονται.** Μετά το `process()`, η σύνοψη της εκτέλεσης επισυνάπτεται στο αποτέλεσμα, και κάθε υπόδειξη επεξεργασίας (π.χ. *γιατί* μια εκτέλεση δεν παρήγαγε έξοδο) αναμεταδίδεται επίσης ως Python `UserWarning` — έτσι, ακόμη και ένα σενάριο που δεν ελέγχει ποτέ το λεξικό αποτελεσμάτων εμφανίζει τη διάγνωση.
* **Η διεπαφή γραμμής εντολών (CLI) αποτυγχάνει με εμφανή τρόπο.**Μια εκτέλεση `chloros-cli process` που ζήτησε αποτελέσματα αλλά δεν έγραψε κανένα, εκτυπώνει το `Processing finished but wrote no image products.` και**τερματίζεται με μη μηδενικό κώδικα εξόδου**, οπότε τα σενάρια shell και το CI το ανιχνεύουν με έναν απλό έλεγχο του κώδικα εξόδου. Οι επιτυχημένες εκτελέσεις αναφέρουν το `Image products written: N`, καθώς και το `(plus M clamp masks)` όταν γράφτηκαν μάσκες. Οι εντολές καταγραφής αναφέρουν επίσης κάθε αρχείο που δεν μπόρεσαν να γράψουν και τερματίζονται με το `1`.
* **Οι σχετικές διαδρομές έχουν την έννοια που αναμένει ένα σενάριο.** Με τοπικό backend, μια σχετική διαδρομή εξόδου (`-o out/` στο CLI, `output_dir="out/"` στο SDK) αναλύεται σε σχέση με τον φάκελο από τον οποίο εκτελείται η εντολή ή το σενάριο.

Μια ασυμμετρία που πρέπει να γνωρίζει ένας βοηθός: το `process()` του SDK **δεν** προκαλεί σφάλμα σκόπιμα σε εκτέλεση χωρίς αποτέλεσμα — αντ’ αυτού, το αναφέρει μέσω της σύνοψης/των υποδείξεων. Εάν ένας αγωγός Python πρέπει να σταματήσει σε μια κενή εκτέλεση, ελέγξτε τη σύνοψη (όπως κάνει η συνταγή 2).

## Προειδοποιήσεις

* **Απαιτείται σύνδεση στο Chloros+.**Το CLI και το SDK απαιτούν**πληρωμένο** επίπεδο Chloros+, το οποίο επιβάλλεται από την πλευρά του διακομιστή: οι αιτήσεις αποτυγχάνουν με το σφάλμα `401 AUTH_REQUIRED` όταν δεν υπάρχει σύνδεση και με το σφάλμα `403 PLAN_UPGRADE_REQUIRED` στο δωρεάν επίπεδο. Εκτελέστε το `chloros-cli login` μία φορά ανά μηχανή πριν εκτελέσετε τα παραγόμενα σενάρια. Δείτε [Σύνδεση στο Chloros+](chloros+-login.md).
* **Οι εντολές καταγραφής ελέγχουν πραγματικό υλικό.** `lattice` / `daq` / `project`, τα αντικείμενα συνεδρίας του SDK και τα εργαλεία του `chloros-mcp` συνδέουν, μεταδίδουν ροή και ενεργοποιούν φυσικές κάμερες και αισθητήρες. Ελέγξτε ένα σενάριο που έχει δημιουργηθεί πριν από την πρώτη εκτέλεσή του και εκτελέστε το με την παρουσία του υλικού.
* **Ελέγξτε δειγματοληπτικά τα αποτελέσματα.** Επαληθεύστε τους φακέλους των προϊόντων και μερικές τιμές εικονοστοιχείων πριν από τη δημοσίευση των αποτελεσμάτων. Ειδικότερα, τα αρχεία TIFF ανακλαστικότητας κλιμακώνονται ανά πηγή — διαβάστε την ετικέτα XMP του `Chloros:PixelScale` (τα αρχεία LATTICE την περιέχουν: 32768 = 1,0 ανακλαστικότητα· τα αρχεία Survey3 δεν την περιέχουν και χρησιμοποιούν 65535 = 1,0) αντί να υποθέσετε έναν διαιρέτη. Και οι δύο σελίδες αναφοράς το τεκμηριώνουν στην ενότητα «Ανάγνωση εικονοστοιχείων ανακλαστικότητας».
* **Μικρές παγίδες που προκαλούν σφάλματα στον παραγόμενο κώδικα:**Το `pool-record` εγγράφει στο σύστημα αρχείων του**κεντρικού υπολογιστή του backend** (προεπιλογή `~/Documents/DAQ Live View/`)· σε μηχανές με πολλές διεπαφές δικτύου, προτιμήστε το `daq pool-connect --eth-host <ip-or-hostname>` αντί της αυτόματης ανίχνευσης· χρησιμοποιήστε το `http://127.0.0.1:5000` (ποτέ το `localhost`) οπουδήποτε εμφανίζεται μια URL του backend· ένα σενάριο που περιηγείται στο δέντρο εξόδου ενός έργου θα πρέπει να παραλείπει τους φακέλους `clamp_masks/` (τα αρχεία `*_clampmask.tif` σε αυτούς είναι μάσκες, όχι εικόνες — βλ. [Μορφές εικόνων εξόδου](output-image-formats.md#clamp-masks))· και στα Windows, όπου οι μακριές διαδρομές είναι απενεργοποιημένες, διατηρήστε τους φακέλους του έργου σύντομους — ένας φάκελος έργου με βάθος μεγαλύτερο από περίπου 130 χαρακτήρες απορρίπτεται πριν ξεκινήσει η επεξεργασία.
