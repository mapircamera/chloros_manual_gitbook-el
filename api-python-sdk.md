# API : Python SDK

{% hint style="info" %}
**Ψάχνετε για το πλήρες API;** Αυτή η σελίδα είναι ένα πρακτικό σεμινάριο. Κάθε δημόσια κλάση, μέθοδος, ακριβής υπογραφή και παράδειγμα που μπορείτε να αντιγράψετε και να επικολλήσετε βρίσκεται στο [Αναφορά SDK](reference/sdk-reference.md), η οποία είναι βελτιστοποιημένη για βοηθούς τεχνητής νοημοσύνης.**Χρησιμοποιείτε βοηθό τεχνητής νοημοσύνης;** Επικολλήστε αυτό το URL στο chat, ώστε να έχει πρόσβαση στην πλήρη, τρέχουσα έκδοση του Chloros 1.2.0 API:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Κάθε σελίδα αυτού του εγχειριδίου είναι διαθέσιμη ως ακατέργαστο markdown με το slug της σε πεζά + `.md`, ενώ ολόκληρο το εγχειρίδιο είναι ευρετηριασμένο στο `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

Το **Chloros Python SDK** (`chloros-sdk` στο PyPI) διαχειρίζεται όλες τις λειτουργίες της εφαρμογής επιφάνειας εργασίας από το Python: μαζική επεξεργασία εικόνων, ζωντανό έλεγχο κάμερας και συστοιχιών LATTICE, συνεδρίες αισθητήρων φωτός DAQ και αυτοματοποίηση αποθηκευμένων έργων. Πρόκειται για ένα λεπτό στρώμα πάνω από το ίδιο τοπικό backend που χρησιμοποιούν το GUI και το CLI (HTTP στο `127.0.0.1:5000`), οπότε η συμπεριφορά είναι πανομοιότυπη και στις τρεις επιφάνειες.

## Εγκατάσταση

Η εγκατάσταση γίνεται σε δύο βήματα: πρώτα το πακέτο Chloros για επιτραπέζιους υπολογιστές (παρέχει το backend επεξεργασίας και τα περιβάλλοντα εκτέλεσης υλικού) και, στη συνέχεια, το πακέτο Python.

**Βήμα 1 — Εγκατάσταση του Chloros.** Windows: εκτελέστε το πρόγραμμα εγκατάστασης για επιτραπέζιους υπολογιστές (προεπιλεγμένη διαδρομή `C:\Program Files\MAPIR\Chloros\`) από τη σελίδα [Λήψη](download.md). Linux: εγκαταστήστε το πακέτο `.deb` ([Εγκατάσταση Linux](linux/linux-installation.md)).**Βήμα 2 — Εγκατάσταση του SDK** (Python 3.7+):

```bash
pip install chloros-sdk
```

Ίσως να μην χρειαστείτε καν το pip: κάθε πρόγραμμα εγκατάστασης περιλαμβάνει ένα αντίστοιχο wheel SDK. Το πρόγραμμα εγκατάστασης Windows το εγκαθιστά αυτόματα στο σύστημά σας Python· το Linux `.deb` το τοποθετεί στο `/usr/lib/chloros/sdk/` και εμφανίζει την ακριβή εντολή `pip install --user`. Το PyPI ενημερώνεται με κάθε έκδοση, οπότε το `pip install chloros-sdk` αντιστοιχεί στην πιο πρόσφατη σταθερή έκδοση.

**Βήμα 3 — Συνδεθείτε μία φορά ανά υπολογιστή:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Τα διαπιστευτήρια αποθηκεύονται προσωρινά στο `~/.chloros/` (και στις δύο πλατφόρμες). Στο Windows μπορείτε να συνδεθείτε με τον ίδιο τρόπο μέσω της καρτέλας «<img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line">» (Διαχείριση χρηστών) της εφαρμογής για επιτραπέζιους υπολογιστές. Το SDK απαιτεί ένα πρόγραμμα Chloros+ επί πληρωμή — δείτε την [Απαίτηση άδειας χρήσης](#license-requirement) παρακάτω.

| Απαίτηση | Λεπτομέρειες |
| --- | --- |
| **Εγκατεστημένο το Chloros** | Windows: πρόγραμμα εγκατάστασης για επιτραπέζιους υπολογιστές; Linux: πακέτο `.deb` (παρέχει το δυαδικό αρχείο του backend) |
| **Python** | 3.7 ή νεότερη έκδοση (αναπτύχθηκε/δοκιμάστηκε στην έκδοση 3.10) |
| **Λειτουργικό σύστημα** | Windows 10/11 64-bit, Ubuntu 22.04 LTS ή νεότερη έκδοση, ή NVIDIA Jetson (JetPack 6) |
| **Άδεια χρήσης** | Ενεργή σύνδεση Chloros+, οποιοδήποτε επίπεδο επί πληρωμή (Copper ή ανώτερο) |

## Η επιτυχία σε 60 δευτερόλεπτα

Μία μόνο κλήση δημιουργεί ένα έργο, εισάγει έναν φάκελο, διαμορφώνει την επεξεργασία και εκτελεί τον αγωγό — ξεκινώντας αυτόματα το backend, αν δεν εκτελείται ήδη:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)
```

(Στο Linux, χρησιμοποιήστε τις διαδρομές Linux: `/home/user/drone_images/flight001`. Το SDK λειτουργεί με τον ίδιο τρόπο και στις δύο πλατφόρμες.)

Επεξεργάζεστε έναν φάκελο καταγραφής LATTICE; Χρησιμοποιήστε το περιβλητικό πρόγραμμα συμβατό με το LATTICE — εφαρμόζει τις σωστές προεπιλογές (χωρίς ανίχνευση στόχου πάνελ, τυπικός απο-bayering):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — πλήρης έλεγχος της ροής εργασιών

Για οτιδήποτε πέρα από μια εντολή μίας γραμμής, χρησιμοποιήστε το `ChlorosLocal`. Δημιουργεί το backend κατά την πρώτη χρήση (`auto_start_backend=True`), δημιουργεί και διαμορφώνει έργα, παρακολουθεί την πρόοδο και επιστρέφει μια σύνοψη μετά την εκτέλεση.

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

{% hint style="info" %}
Διατηρήστε το προεπιλεγμένο `http://127.0.0.1:5000` αντί να το αντικαταστήσετε με το `localhost` — στο Windows, το `localhost` μετατρέπεται πρώτα σε `::1` και κοστίζει περίπου 2 δευτερόλεπτα ανά αίτημα στο backend που υποστηρίζει μόνο IPv4.
{% endhint %}

Χρησιμοποιήστε το ως διαχειριστή περιβάλλοντος για εγγυημένο καθαρισμό:

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

Το `configure()` δέχεται τις ακόλουθες λέξεις-κλειδιά: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` και `custom_settings`. Οι κύριες τιμές:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

Τα ειδικά για τη LATTICE κουμπιά (`input_level`, `radiometric_output`, η οικογένεια `array_alignment*`) τεκμηριώνονται με τους πλήρεις πίνακες τιμών τους στο [SDK Reference](reference/sdk-reference.md#supported-values).

### Παρακολούθηση της προόδου

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Ανάγνωση της σύνοψης μετά την εκτέλεση — και εντοπισμός κενών εκτελέσεων

Μετά την ολοκλήρωση, το `process()` επισυνάπτει τη σύνοψη επεξεργασίας του backend ως `result["summary"]`. Κάθε καταχώριση στο `summary["hints"]` αποτελεί μια πλήρη πρόταση που εξηγεί οτιδήποτε αξιοσημείωτο — για παράδειγμα, γιατί μια εκτέλεση παρήγαγε μηδενικό αποτέλεσμα — και κάθε υπόδειξη αναμεταδίδεται επίσης ως Python `UserWarning`, οπότε οι κενές εκτελέσεις αυτοδιαγιγνώσκονται ακόμα και αν δεν ελέγξετε ποτέ το λεξικό:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**Το `process()` δεν εμφανίζεται όταν μια εκτέλεση δεν παράγει εικόνες.** Αυτό είναι το μόνο σημείο στο οποίο το SDK και το CLI διαφέρουν σκόπιμα: Το `chloros-cli process` αντιμετωπίζει την κατάσταση «ζητήθηκαν προϊόντα, αλλά δεν γράφτηκε κανένα» ως αποτυχία και τερματίζεται με μη μηδενικό αποτέλεσμα, ενώ το SDK επιστρέφει κανονικά και αναφέρει την κατάσταση μέσω του `summary` / υποδείξεων. Εάν ο αγωγός σας πρέπει να σταματήσει σε περίπτωση κενής εκτέλεσης, ελέγξτε το οι ίδιοι — εξετάστε το `summary` (ή μετρήστε τα αρχεία στον φάκελο του έργου) αντί να βασίζεστε σε μια εξαίρεση.
{% endhint %}

## Smart Connect — ζωντανό υλικό

Τρία βοηθητικά προγράμματα ανοίγουν μόνιμες συνεδρίες στην ομάδα υλικού του backend — την ίδια ομάδα που χρησιμοποιεί το GUI, οπότε τα σενάρια SDK συνυπάρχουν με την εφαρμογή επιφάνειας εργασίας χωρίς να ανταγωνίζονται για σειριακές θύρες ή εύρος ζώνης δικτύου. Και τα τρία εκκινούν αυτόματα ένα τοπικό backend αν δεν εκτελείται κανένα.

### Μία κάμερα LATTICE — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

### Συγχρονισμένη συστοιχία — `connect_array`

Το `connect_array` είναι το συνιστώμενο σημείο εκκίνησης για συστοιχίες πολλαπλών καμερών. Εκτελεί την ίδια ροή έξυπνης προετοιμασίας με το GUI: ανάλυση δικτύου, αυτόματη επιλογή επιπέδου συγχρονισμού, χρονικό συγχρονισμό PTP, επιλογή μορφής εικονοστοιχείου ανά κάμερα, αρχική ρύθμιση AE και ενεργοποίηση σκανδάλης GPIO. Η **πρώτη σειριακή θύρα είναι η κύρια** (αυτή ενεργοποιεί τον παλμό ενεργοποίησης του υλικού)· οι υπόλοιπες είναι δευτερεύουσες.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Προσθέστε το `smart=True` σε οποιαδήποτε λήψη συστοιχίας για να περιμένετε να σταθεροποιηθεί η αυτόματη έκθεση σε όλες τις κάμερες πριν από την ενεργοποίηση. Για τις λειτουργίες λήψης (Μονή / Συνεχής / Διαστήματα / Ταχύτερη), τους καταγραφείς, τη λήψη ραφής σε βίντεο και την ευθυγράμμιση συστοιχιών, ανατρέξτε στο [Εγχειρίδιο αναφοράς SDK](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### Αισθητήρας φωτός DAQ — `connect_daq_sensor`

Χωρίς ορίσματα, το `connect_daq_sensor()` ανιχνεύει αυτόματα το μέσο μεταφοράς (προτεραιότητα: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Κάθε πλαίσιο μεταφέρει την τιμή 135 μονάδων `spectrum` (W/m²/nm όταν έχει βαθμονομηθεί), μια σημαία `is_saturated` και CIE `x`, `y`, `z`. Για να προσδιορίσετε συγκεκριμένο αισθητήρα ή μέσο μεταφοράς — η αξιόπιστη επιλογή σε κεντρικούς υπολογιστές με πολλαπλές διεπαφές δικτύου, όπου η αυτόματη ανίχνευση Ethernet ενδέχεται να παραλείψει ένα λειτουργικό DAQ-E κατά την πρώτη προσπάθεια — περάστε μία ρητή υπόδειξη:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Σημειώστε ότι τα προφίλ διόρθωσης κεφαλαίων (`cap_id`) **δεν** αποτελούν ρυθμιστικό τύπου SDK — επιλέξτε τα μέσω του `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Αποθηκευμένα έργα — `open_project`

Ένα αποθηκευμένο έργο Chloros διατηρεί το συνδεδεμένο υλικό του (`cameras.json` + `sensors.json` μαζί με το `project.json`), και το `chloros_sdk.open_project(path)` μπορεί να επανασυνδέσει τα πάντα ταυτόχρονα και να πραγματοποιήσει καταγραφές με βάση το όνομα της συσκευής. Δείτε την ενότητα [Αυτοματοποίηση έργου](reference/sdk-reference.md#project-automation--chlorosproject) στην αναφορά.

## Τι παρέχει μια εγκατάσταση μόνο μέσω pip

Ελέγξτε τις σημαίες διαθεσιμότητας σε επίπεδο ενότητας πριν χρησιμοποιήσετε επιφάνειες υλικού:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)       # True iff daq_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

Σε έναν κεντρικό υπολογιστή με **μόνο** το `pip install chloros-sdk` και χωρίς το πακέτο επιφάνειας εργασίας Chloros:

* Τα `ChlorosLocal`, `process_folder` και `process_lattice_capture` **δεν** λειτουργούν — χρειάζονται το δυαδικό αρχείο backend που περιλαμβάνεται στο πρόγραμμα εγκατάστασης επιφάνειας εργασίας.
* Τα βοηθητικά προγράμματα smart-connect (`connect_camera`, `connect_array`, `connect_daq_sensor`) είναι καθαρά προγράμματα-πελάτες HTTP, οπότε λειτουργούν με ένα backend σε άλλο μηχάνημα — ωστόσο, τα παρεχόμενα backends συνδέονται μόνο με loopback, οπότε πρέπει να προωθήσετε τη θύρα μόνοι σας (π.χ. `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) και να περάσετε το `backend_url="http://127.0.0.1:5000"` μαζί με το `auto_start_backend=False`. Δείτε [Λειτουργία απομακρυσμένου backend](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* Οι κλάσεις LATTICE άμεσης σύνδεσης με το υλικό (`LatticeCamera`, `CameraPool`, …) εισάγονται, αλλά χρειάζονται το περιβάλλον εκτέλεσης Arena SDK από το πακέτο για επιτραπέζιους υπολογιστές — χωρίς αυτό, το `CAMERA_AVAILABLE` είναι το `False`.
* Το `daq_sdk` (οι κλάσεις άμεσης DAQ) διατίθεται με την εγκατάσταση για επιτραπέζιους υπολογιστές, όχι με το πακέτο PyPI, οπότε το `DAQ_AVAILABLE` είναι το `False` σε έναν κεντρικό υπολογιστή που χρησιμοποιεί μόνο το pip — ελέγξτε τους αισθητήρες DAQ μέσω του `connect_daq_sensor()` σε ένα (διοχετευμένο) backend αντ&#x27; αυτού.

## Απαιτήσεις άδειας χρήσης

Η πρόσβαση στο SDK απαιτεί ενεργή σύνδεση Chloros+ σε οποιοδήποτε επίπεδο επί πληρωμή — **Copper ή υψηλότερο**(Copper / Bronze / Silver / Gold). το δωρεάν πακέτο Iron δεν παρέχει πρόσβαση στο SDK/CLI. Η επιβολή γίνεται**από την πλευρά του διακομιστή**: κάθε αίτημα SDK πρέπει να συνοδεύεται τόσο από ενεργή συνεδρία όσο και από επί πληρωμή πακέτο, διαφορετικά το backend επιστρέφει `403` / `PLAN_UPGRADE_REQUIRED` (που δημιουργείται ως `ChlorosLicenseError` από το `ChlorosLocal`, και ως `ChlorosConnectError` από τα βοηθητικά προγράμματα `connect_*`). Ένας χρήστης που έχει αποσυνδεθεί λαμβάνει τα σφάλματα `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — η επανεκτέλεση του `chloros-cli login` διορθώνει την πρώτη περίπτωση, αλλά όχι τη δεύτερη.

Η χρήση εκτός σύνδεσης λειτουργεί εντός της περιόδου χάριτος του προγράμματος: το επίπεδο ανάγεται από την προσωρινή μνήμη επικύρωσης διακομιστή (5 λεπτά) ή από την προσωρινή μνήμη υπογεγραμμένης, συνδεδεμένης με τη συσκευή άδειας χρήσης (30 ημέρες για τα μηνιαία προγράμματα· έως τη λήξη της συνδρομής για τα ετήσια). Όταν λήξει η περίοδος χάριτος, το πρόγραμμα μετατρέπεται σε δωρεάν και η πρόσβαση μέσω του SDK διακόπτεται έως ότου ο υπολογιστής συνδεθεί με τον διακομιστή μία φορά. Το `chloros-cli status` παραμένει προσβάσιμο στο δωρεάν επίπεδο, οπότε ο λόγος εμφανίζεται πάντα. Δείτε [Chloros+ Σύνδεση](chloros+-login.md).

## Εξαιρέσεις

Χρησιμοποιήστε την βασική κλάση για να χειριστείτε «οτιδήποτε Chloros πήγε στραβά»:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

Όλες οι εξαιρέσεις του pipeline (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) προέρχονται από το `ChlorosError`. Μια λεπτομέρεια: το `ChlorosConnectError` — δημιουργείται μόνο από τα `connect_camera` / `connect_array` / `connect_daq_sensor` — προέρχεται από το απλό `Exception`, **όχι** από το `ChlorosError`, οπότε το `except ChlorosError` δεν θα το εντοπίσει. Η πλήρης ιεραρχία βρίσκεται στην [Αναφορά SDK](reference/sdk-reference.md#exceptions).

## Δείτε επίσης

* [Αναφορά SDK](reference/sdk-reference.md) — η πλήρης επιφάνεια API, βελτιστοποιημένη για βοηθούς τεχνητής νοημοσύνης.
* [Αναφορά CLI](reference/cli-reference.md) — κάθε υποεντολή CLI αντιστοιχεί σε μια κλήση SDK.
* [Λήψη](download.md) — προγράμματα εγκατάστασης για τα Windows και Linux.
