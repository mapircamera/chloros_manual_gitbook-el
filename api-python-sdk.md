# API: Python SDK

{% hint style="info" %}
**Ψάχνετε για το πλήρες API;** Αυτή η σελίδα είναι ένα πρακτικό σεμινάριο. Κάθε δημόσια κλάση, μέθοδος, ακριβής υπογραφή και παράδειγμα που μπορείτε να αντιγράψετε και να επικολλήσετε βρίσκεται στην [Αναφορά SDK](reference/sdk-reference.md), η οποία είναι βελτιστοποιημένη για βοηθούς τεχνητής νοημοσύνης.**Χρησιμοποιείτε βοηθό τεχνητής νοημοσύνης;** Επικολλήστε αυτή την URL στο chat, ώστε να έχει πρόσβαση στο πλήρες, τρέχον Chloros 1.3.0 API:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Κάθε σελίδα αυτού του εγχειριδίου είναι διαθέσιμη ως ακατέργαστο markdown με το slug της σε πεζά + `.md`, ενώ ολόκληρο το εγχειρίδιο είναι ευρετηριασμένο στο `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

Το **Chloros Python SDK** (`chloros-sdk` στο PyPI) διαχειρίζεται όλες τις λειτουργίες της εφαρμογής για επιτραπέζιους υπολογιστές μέσω της Python: μαζική επεξεργασία εικόνων, ζωντανό έλεγχο της κάμερας LATTICE και της συστοιχίας, συνεδρίες DAQ με αισθητήρες φωτός, καθώς και αυτοματοποίηση αποθηκευμένων έργων. Πρόκειται για ένα λεπτό στρώμα πάνω από το ίδιο τοπικό backend που χρησιμοποιούν το GUI και το CLI (HTTP στο `127.0.0.1:5000`), οπότε η συμπεριφορά είναι πανομοιότυπη και στις τρεις διεπαφές.

## Εγκατάσταση

Το SDK περιλαμβάνεται στο πακέτο Chloros για επιτραπέζιους υπολογιστές, το οποίο παρέχει επίσης το backend επεξεργασίας και τα περιβάλλοντα εκτέλεσης υλικού, και η έκδοσή του ταιριάζει πάντα με αυτή του πακέτου.

**Βήμα 1 — Εγκατάσταση του Chloros.** Windows: εκτελέστε το πρόγραμμα εγκατάστασης για επιτραπέζιους υπολογιστές (προεπιλεγμένη διαδρομή `C:\Program Files\Chloros\`) από τη σελίδα [Λήψη](download.md). Linux: εγκαταστήστε το πακέτο `.deb` με το `sudo apt install ./chloros_1.3.0_<arch>.deb` ([Εγκατάσταση σε Linux](linux/linux-installation.md)).**Βήμα 2 — Ελέγξτε το SDK** (Python 3.7+). Συνήθως δεν χρειάζεται να εγκαταστήσετε τίποτα:

* **Windows:** το πρόγραμμα εγκατάστασης εγκαθιστά το πακέτο SDK που περιλαμβάνεται στο σύστημα Python (εάν δεν είναι εγκατεστημένο το Python, παραλείπει αυτό το βήμα).
* **Linux:** το `.deb` εγκαθιστά το SDK για το σύστημα `python3`, στο `/usr/lib/python3/dist-packages`. Οι απαιτήσεις του, `python3-numpy` και `python3-requests`, συνιστώνται από το πακέτο: το `apt install` τις περιλαμβάνει, ενώ το `dpkg -i` όχι.
* **Ένα virtualenv** δεν αναγνωρίζει τα πακέτα του συστήματος· εγκαταστήστε το συνοδευτικό αρχείο wheel σε αυτό: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` σε Linux, ή το `chloros_sdk-*.whl` στον φάκελο `sdk` του φακέλου εγκατάστασης των Windows (`C:\Program Files\Chloros\sdk\` από προεπιλογή).

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # Windows: py -c "…"
```

Σε έναν υπολογιστή χωρίς πακέτο Chloros, του οποίου το SDK οδηγεί ένα backend σε άλλον υπολογιστή (βλ. [Τι παρέχει μια εγκατάσταση μόνο με pip](#what-a-pip-only-install-gets)), εγκαταστήστε το από το PyPI: `pip install chloros-sdk`. Το PyPI ενημερώνεται με κάθε νέα έκδοση, οπότε αντιστοιχεί στην πιο πρόσφατη σταθερή έκδοση.

**Βήμα 3 — Συνδεθείτε μία φορά ανά υπολογιστή:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Τα διαπιστευτήρια αποθηκεύονται προσωρινά στο `~/.chloros/` (και στις δύο πλατφόρμες). Στα Windows, μπορείτε να συνδεθείτε με τον ίδιο τρόπο μέσω της καρτέλας «Χρήστης» <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> της εφαρμογής για επιτραπέζιους υπολογιστές. Το SDK απαιτεί ένα πρόγραμμα Chloros+ επί πληρωμή — δείτε την [Απαίτηση άδειας χρήσης](#license-requirement) παρακάτω.

| Απαίτηση | Λεπτομέρειες |
| --- | --- |
| **Εγκατάσταση Chloros** | Windows: πρόγραμμα εγκατάστασης για επιτραπέζιους υπολογιστές· Linux: πακέτο `.deb` (παρέχει το εκτελέσιμο του backend και το SDK) |
| **Python** | 3.7 ή νεότερη έκδοση (αναπτύχθηκε/δοκιμάστηκε στην έκδοση 3.10) |
| **Λειτουργικό σύστημα** | Windows 10/11 64-bit, Ubuntu 22.04 LTS ή νεότερη έκδοση, ή NVIDIA Jetson (JetPack 6) |
| **Άδεια χρήσης** | Ενεργή σύνδεση στο Chloros+, οποιοδήποτε επίπεδο συνδρομής επί πληρωμή (Copper ή ανώτερο) |

## Η επιτυχία σε 60 δευτερόλεπτα

Μία κλήση δημιουργεί ένα έργο, εισάγει έναν φάκελο, διαμορφώνει την επεξεργασία και εκτελεί τον αγωγό — ξεκινώντας αυτόματα το backend αν δεν εκτελείται ήδη:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)
```

(Στο Linux, χρησιμοποιήστε διαδρομές Linux: `/home/user/drone_images/flight001`. Το SDK λειτουργεί με τον ίδιο τρόπο και στις δύο πλατφόρμες.)

Επεξεργάζεστε έναν φάκελο καταγραφής LATTICE; Χρησιμοποιήστε το περιβλήμα συμβατό με το LATTICE — εφαρμόζει τις σωστές προεπιλογές (χωρίς ανίχνευση στόχου πάνελ, τυπικός απο-bayering):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — πλήρης έλεγχος του pipeline

Για οτιδήποτε πέρα από μια εντολή μίας γραμμής, χρησιμοποιήστε το `ChlorosLocal`. Χρησιμοποιεί ένα backend που ήδη εκτελείται (αυτό της εφαρμογής για επιτραπέζιους υπολογιστές) ή ξεκινά το εγκατεστημένο κατά την πρώτη χρήση (`auto_start_backend=True`; στα Windows από τον φάκελο `backend` του φακέλου εγκατάστασης του Chloros, `C:\Program Files\Chloros\backend\` από προεπιλογή), δημιουργεί και διαμορφώνει έργα, παρακολουθεί την πρόοδο και επιστρέφει μια σύνοψη μετά την εκτέλεση.

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
Διατηρήστε το προεπιλεγμένο `http://127.0.0.1:5000` αντί να το αντικαταστήσετε με το `localhost` — στα Windows, το `localhost` αναλύεται πρώτα σε `::1` και κοστίζει περίπου 2 δευτερόλεπτα ανά αίτημα έναντι του backend που υποστηρίζει μόνο IPv4.
{% endhint %}

Χρησιμοποιήστε το ως διαχειριστή περιβάλλοντος. Η έξοδος από το μπλοκ τερματίζει τη χρήση του backend από αυτή την παρουσία, όχι το ίδιο το backend: αυτό παραμένει σε λειτουργία για το επόμενο σενάριο και τερματίζεται αυτόματα μετά από 30 λεπτά αδράνειας. Το `shutdown_backend()` τερματίζει μία παρουσία που έχει ξεκινήσει.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

Το `configure()` δέχεται τις ακόλουθες λέξεις-κλειδιά: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` και `custom_settings`. Οι κύριες τιμές:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (8-bit)"            # 8-bit TIFF; what all-8-bit images get for a deeper format
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

Οι ρυθμιστές που αφορούν συγκεκριμένα το LATTICE (`input_level`, `radiometric_output`, η οικογένεια `array_alignment*`) τεκμηριώνονται με τους πλήρεις πίνακες τιμών τους στο [Εγχειρίδιο αναφοράς SDK](reference/sdk-reference.md#supported-values).

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

Μετά την ολοκλήρωση, το `process()` επισυνάπτει τη σύνοψη επεξεργασίας του backend ως `result["summary"]`. Κάθε καταχώριση στο `summary["hints"]` είναι μια πλήρης πρόταση που εξηγεί οτιδήποτε αξιοσημείωτο — για παράδειγμα, γιατί μια εκτέλεση δεν παρήγαγε κανένα αποτέλεσμα — και κάθε υπόδειξη αναμεταδίδεται επίσης ως Python `UserWarning`, οπότε οι κενές εκτελέσεις αυτοδιαγιγνώσκονται ακόμα και αν δεν ελέγξετε ποτέ το λεξικό:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**Το `process()` δεν ενεργοποιείται όταν μια εκτέλεση δεν παράγει εικόνες.** Αυτό είναι το μόνο σημείο στο οποίο το SDK και το CLI διαφέρουν σκόπιμα: Το `chloros-cli process` αντιμετωπίζει την κατάσταση «ζητήθηκαν προϊόντα, κανένα δεν δημιουργήθηκε» ως αποτυχία και τερματίζεται με μη μηδενικό αποτέλεσμα, ενώ το SDK επιστρέφει κανονικά και αναφέρει την κατάσταση μέσω του `summary` / υποδείξεων. Εάν η ροή εργασιών σας πρέπει να σταματήσει σε περίπτωση κενής εκτέλεσης, ελέγξτε το οι ίδιοι — εξετάστε το `summary` (ή μετρήστε τα αρχεία στον φάκελο του έργου) αντί να βασίζεστε σε μια εξαίρεση.
{% endhint %}

## Smart Connect — ενεργό υλικό

Τρία βοηθητικά προγράμματα ανοίγουν μόνιμες συνεδρίες στην ομάδα υλικού του backend — την ίδια ομάδα που χρησιμοποιούν η CLI, άλλα σενάρια και οι καρτέλες καμερών του GUI, έτσι ώστε τα σενάρια του SDK να συνυπάρχουν μαζί τους χωρίς να ανταγωνίζονται για σειριακές θύρες ή εύρος ζώνης δικτύου. Και τα τρία ξεκινάνε αυτόματα ένα τοπικό backend αν δεν εκτελείται κανένα. Ο πίνακας DAQ του GUI δεν αποτελεί μέρος της ομάδας: κλείστε έναν αισθητήρα φωτός στο GUI πριν τον ανοίξετε από ένα σενάριο.

### Μία κάμερα LATTICE — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

{% hint style="info" %}
**Οι σχετικές διαδρομές καταλήγουν στον τόπο εκτέλεσης του σεναρίου σας (αλλαγή στην έκδοση 1.3.0).** Με τοπικό backend, μια σχετική διαδρομή `output_dir` (συμπεριλαμβανομένης της προεπιλεγμένης `"output"`) — και οι άλλες διαδρομές που ανοίγει το backend (`burst_dir`, `profile_path`, `save_path`, `mask_path`) — αναλύονται σε σχέση με τον κατάλογο εργασίας του σεναρίου σας, οπότε το `cam.capture("output/")` εγγράφει στο `<current folder>/output/`. Στην έκδοση 1.2.0, ένα εγκατεστημένο backend τις αντιστοίχιζε στον δικό του φάκελο εγκατάστασης και παρουσίαζε σφάλμα με το μήνυμα «Η πρόσβαση δεν επιτρέπεται». Σε περίπτωση απομακρυσμένου backend, οι διαδρομές αποστέλλονται όπως έχουν πληκτρολογηθεί.
{% endhint %}

### Συγχρονισμένος πίνακας — `connect_array`

Το `connect_array` είναι το συνιστώμενο σημείο εισόδου για συστοιχίες πολλαπλών καμερών. Εκτελεί την ίδια ροή έξυπνης προετοιμασίας με το GUI: ανάλυση δικτύου, αυτόματη επιλογή επιπέδου συγχρονισμού, χρονικό συγχρονισμό PTP, επιλογή μορφής εικονοστοιχείων ανά κάμερα, αρχικοποίηση AE και ενεργοποίηση σκανδάλης GPIO. Η **πρώτη σειριακή θύρα είναι η κύρια** (απελευθερώνει τον παλμό ενεργοποίησης του υλικού)· οι υπόλοιπες είναι δευτερεύουσες.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Προσθέστε το `smart=True` σε οποιαδήποτε λήψη πίνακα για να περιμένετε να σταθεροποιηθεί η αυτόματη έκθεση σε όλες τις κάμερες πριν από την ενεργοποίηση.

Νέο στην έκδοση 1.3.0, το `ArraySession` σας παρέχει επίσης συγχρονισμένα καρέ απευθείας, χωρίς να δημιουργεί αρχεία:

* Το `arr.grab()` επιστρέφει ένα καρέ ανά κάμερα, όλα από μία ενεργοποίηση, ως πίνακες numpy (`{serial: array}`)· Το `.group_id` είναι η κοινή ενεργοποίηση και το `.errors` ονομάζει οποιαδήποτε κάμερα έχει παραλειφθεί.
* Το `arr.frame_stream(fps=5.0)` παράγει ένα τέτοιο σύνολο ανά tick, για ζωντανή επεξεργασία στη δική σας ροή εργασιών.
* Το `arr.continuous("output/")` εκτελεί τη δική του συνεχή λήψη του backend (η λειτουργία «Continuous» του GUI) και επιστρέφει ένα `ContinuousHandle` με τα `status()`, `wait()` και `stop()` — χρησιμοποιήστε το αντί για βρόχο καταγραφής όταν χρειάζεστε να αποθηκεύονται όλα τα ερεθίσματα με υψηλό ρυθμό.

Για τις λειτουργίες καταγραφής (Ενιαία / Συνεχής / Διαστήματος / Ταχύτερη), τους καταγραφείς, τη λειτουργία «burst-to-video» και την ευθυγράμμιση πινάκων, ανατρέξτε στην [Αναφορά SDK](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### Αισθητήρας φωτός DAQ — `connect_daq_sensor`

Χωρίς ορίσματα, το `connect_daq_sensor()` ανιχνεύει αυτόματα το μέσο μεταφοράς (προτεραιότητα: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Κάθε πλαίσιο μεταφέρει το `spectrum` 135 σημείων (W/m²/nm όταν έχει βαθμονομηθεί), μια σημαία `is_saturated` και CIE `x`, `y`, `z`. Για να καθορίσετε συγκεκριμένο αισθητήρα ή μέσο μεταφοράς, περάστε μία ρητή υπόδειξη:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Σημειώστε ότι τα προφίλ διόρθωσης κορυφών (`cap_id`) **δεν** αποτελούν ρυθμιστικό του SDK — επιλέξτε τα αντί αυτού μέσω των `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Αποθηκευμένα έργα — `open_project`

Ένα αποθηκευμένο έργο Chloros διατηρεί το συνδεδεμένο υλικό του (`cameras.json` + `sensors.json` μαζί με το `project.json`), και το `chloros_sdk.open_project(path)` μπορεί να επανασυνδέσει τις κάμερες, τις συστοιχίες και τους αισθητήρες φωτός ταυτόχρονα και να ενεργοποιήσει τη λήψη εικόνων με βάση το όνομα της συσκευής. Οι αισθητήρες φωτός επανασυνδέονται μέσω ενός backend του Chloros που εκτελείται, με τις αποθηκευμένες ρυθμίσεις και το όριο λήψεων τους. Δείτε το [Project Automation](reference/sdk-reference.md#project-automation--chlorosproject) στην αναφορά.

## Τι παρέχει μια εγκατάσταση μόνο με pip

Ελέγξτε τις σημαίες διαθεσιμότητας σε επίπεδο ενότητας πριν χρησιμοποιήσετε επιφάνειες υλικού:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

Σε έναν κεντρικό υπολογιστή με **μόνο** το `pip install chloros-sdk` και χωρίς το πακέτο επιφάνειας εργασίας Chloros:

* Τα `ChlorosLocal`, `process_folder` και `process_lattice_capture` **δεν** λειτουργούν — χρειάζονται το δυαδικό αρχείο backend που περιλαμβάνεται στο πρόγραμμα εγκατάστασης του desktop.
* Τα βοηθητικά προγράμματα smart-connect (`connect_camera`, `connect_array`, `connect_daq_sensor`) είναι καθαρά προγράμματα-πελάτες HTTP, οπότε λειτουργούν με ένα backend σε άλλο υπολογιστή — ωστόσο, τα παρεχόμενα backend συνδέονται μόνο με loopback, οπότε πρέπει να προωθήσετε τη θύρα μόνοι σας (π.χ. `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) και να περάσετε το `backend_url="http://127.0.0.1:5000"` μαζί με το `auto_start_backend=False`. Δείτε [Λειτουργία απομακρυσμένου backend](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* Οι κλάσεις LATTICE που αφορούν απευθείας το υλικό (`LatticeCamera`, `CameraPool`, …) εισάγονται, αλλά χρειάζονται το περιβάλλον εκτέλεσης του Arena SDK από το πακέτο για επιτραπέζιους υπολογιστές — χωρίς αυτό, το `CAMERA_AVAILABLE` είναι `False`.
* Οι αισθητήρες DAQ λειτουργούν μόνο μέσω ενός (tunneled) backend — `connect_daq_sensor()`, ή του `proj.sensors` ενός έργου· το backend ανοίγει τον αισθητήρα στον δικό του υπολογιστή.

## Βοηθοί τεχνητής νοημοσύνης — `chloros-mcp`

Το SDK περιλαμβάνει το `chloros-mcp`, έναν διακομιστή MCP που επιτρέπει στο Claude Desktop, το Claude Code, το Cursor και άλλους πελάτες MCP να ελέγχουν το Chloros: να επεξεργάζονται εικόνες και να χειρίζονται κάμερες LATTICE, συστοιχίες και αισθητήρες φωτός DAQ. Τα εργαλεία του καλύπτουν τις λειτουργίες αυτού του SDK. Εγκαθίσταται μαζί με το SDK, οπότε για το Claude Code η ρύθμιση γίνεται με μία γραμμή:

```bash
claude mcp add chloros -- chloros-mcp
```

Άλλοι πελάτες: [Χρήση του Chloros με βοηθούς τεχνητής νοημοσύνης](ai-assistants.md). Επιλογές και όλα τα εργαλεία: [MCP Server (Βοηθοί ΤΝ)](reference/sdk-reference.md#mcp-server-ai-assistants).

## Απαιτήσεις άδειας χρήσης

Η πρόσβαση στο SDK απαιτεί ενεργή σύνδεση στο Chloros+ σε οποιοδήποτε επίπεδο επί πληρωμή — **Copper ή ανώτερο**(Copper / Bronze / Silver / Gold). το δωρεάν πακέτο Iron δεν παρέχει πρόσβαση σε SDK/CLI. Ο έλεγχος γίνεται**από την πλευρά του διακομιστή**: κάθε αίτημα SDK πρέπει να συνοδεύεται τόσο από ενεργή συνεδρία όσο και από πακέτο επί πληρωμή, διαφορετικά το backend επιστρέφει τα κωδικούς σφάλματος `403` / `PLAN_UPGRADE_REQUIRED` (που εμφανίζεται ως `ChlorosLicenseError` από το `ChlorosLocal`, και ως `ChlorosConnectError` από τα βοηθητικά προγράμματα `connect_*`). Ένας χρήστης που έχει αποσυνδεθεί λαμβάνει τα σφάλματα `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — η επανεκτέλεση του `chloros-cli login` διορθώνει την πρώτη περίπτωση, αλλά όχι τη δεύτερη.

Η χρήση εκτός σύνδεσης λειτουργεί εντός της περιόδου χάριτος του προγράμματος: το επίπεδο ανάγνωσης προέρχεται από την προσωρινή μνήμη επικύρωσης διακομιστή (5 λεπτά) ή από την προσωρινή μνήμη υπογεγραμμένης, δεσμευμένης σε συσκευή άδειας χρήσης (30 ημέρες για τα μηνιαία προγράμματα· έως τη λήξη της συνδρομής για τα ετήσια). Όταν λήξει η περίοδος χάριτος, το πακέτο μετατρέπεται σε δωρεάν και η πρόσβαση στο SDK διακόπτεται έως ότου ο υπολογιστής συνδεθεί με τον διακομιστή μία φορά. Το `chloros-cli status` παραμένει προσβάσιμο στο δωρεάν επίπεδο, οπότε ο λόγος είναι πάντα ορατός. Δείτε [Σύνδεση στο Chloros+](chloros+-login.md).

## Εξαιρέσεις

Εντοπίστε τη βασική κλάση για να χειριστείτε «οτιδήποτε πήγε στραβά στο Chloros»:

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

Όλες οι εξαιρέσεις του αγωγού (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) προέρχονται από το `ChlorosError`. Μια λεπτομέρεια: το `ChlorosConnectError` — δημιουργείται μόνο από τα `connect_camera` / `connect_array` / `connect_daq_sensor` — προέρχεται από το απλό `Exception`, **όχι** από το `ChlorosError`, οπότε το `except ChlorosError` δεν θα το εντοπίσει. Η πλήρης ιεραρχία βρίσκεται στην [Αναφορά SDK](reference/sdk-reference.md#exceptions).

## Δείτε επίσης

* [Αναφορά SDK](reference/sdk-reference.md) — η πλήρης επιφάνεια API, βελτιστοποιημένη για βοηθούς τεχνητής νοημοσύνης.
* [Αναφορά CLI](reference/cli-reference.md) — κάθε υποεντολή CLI αντιστοιχεί σε μια κλήση SDK.
* [Χρήση του Chloros με βοηθούς τεχνητής νοημοσύνης](ai-assistants.md) — οδηγίες ρύθμισης και προτροπών για το `chloros-mcp`.
* [Λήψη](download.md) — προγράμματα εγκατάστασης για Windows και Linux.
