# Chloros Python SDK Αναφορά

**Έκδοση:**

1.2.0**Δημιουργήθηκε:**29/07/2026 19:19 ·**Αναθεωρήθηκε:** 30/08/2026**Πακέτο:** `chloros-sdk` (PyPI)**Κοινό:** Βελτιστοποιημένο για χρήση από LLM· αναγνώσιμο από τον άνθρωπο.**Πεδίο εφαρμογής:** Κάθε δημόσια κλάση, συνάρτηση και βοηθητική συνάρτηση που παρέχεται από το `import chloros_sdk`, με παραδείγματα που μπορούν να αντιγραφούν και να επικολληθούν και καλύπτουν την επεξεργασία εικόνων, τον έλεγχο μίας κάμερας, τους συγχρονισμένους πίνακες, τους αισθητήρες DAQ και την αυτοματοποίηση έργων.

Αν θέλετε μόνο τα βασικά σημεία, μεταβείτε στα εξής:
- [Εγκατάσταση &amp; Γρήγορη εκκίνηση](#installation)
- [Smart-Connect για συστοιχίες LATTICE](#smart-connect-for-lattice-cameras)
- [Συνεδρίες αισθητήρων DAQ](#daq-sensor-sessions)
- [Αυτοματοποίηση έργου](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)

---

## Η αρχιτεκτονική σε 60 δευτερόλεπτα

Το SDK είναι ένα λεπτό στρώμα Python πάνω από το backend Chloros (ο ίδιος διακομιστής Flask που χρησιμοποιούν το γραφικό περιβάλλον εργασίας για επιτραπέζιους υπολογιστές και το CLI). Για την αυτοματοποίηση, εισάγετε το `chloros_sdk` και καλείτε μεθόδους υψηλού επιπέδου· στο παρασκήνιο, κάθε κλήση μετατρέπεται σε αίτημα HTTP προς το τοπικό backend στη θύρα 5000 — `http://127.0.0.1:5000/api/...` (σκόπιμα όχι `localhost`, το οποίο αναλύεται πρώτα σε `::1` στο Windows και κοστίζει ~2 s ανά αίτημα έναντι ενός backend που υποστηρίζει μόνο IPv4). Το backend διαχειρίζεται το σύνολο του υλικού — κάμερες, αισθητήρες DAQ, προφίλ ευθυγράμμισης, frame buffers — έτσι ώστε τα σενάρια SDK να μπορούν να συνυπάρχουν με το GUI χωρίς να ανταγωνίζονται για σειριακές θύρες ή εύρος ζώνης δικτυακής κάρτας.

Υπάρχουν τρεις επιφάνειες εργασίας που θα χρησιμοποιήσετε:

1. **`ChlorosLocal` + ελεύθερες συναρτήσεις** (`process_folder`, `process_lattice_capture`) — Αγωγός επεξεργασίας εικόνων. Εκτελέστε έναν ολόκληρο φάκελο μέσω βαθμονόμησης / απο-Bayer / εξαγωγής ευρετηρίου με μία μόνο κλήση της συνάρτησης `Python`.
2. **Χειριστές Smart-connect** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Άνοιγμα μόνιμης συνεδρίας backend για ενεργό υλικό. Ίδια ροή «smart-prep» με το GUI: έλεγχος δικτύου, αυτόματη επιλογή επιπέδου, PTP, αρχική ρύθμιση AE, διαμόρφωση ενεργοποίησης GPIO.
3. **`ChlorosProject` / `open_project`** — Φόρτωση αποθηκευμένου έργου (φάκελος με `cameras.json` + `sensors.json` + `project.json`), σύνδεση όλων ταυτόχρονα και καταγραφή δεδομένων με ονομασμένους χειριστές.

Οι επιφάνειες 1 και 2 **αυτόματη εκκίνηση ενός τοπικού backend**, εάν δεν υπάρχει ήδη κάποιο σε κατάσταση ακρόασης (το ίδιο ενσωματωμένο εκτελέσιμο που δημιουργούν το GUI/CLI) — έτσι ώστε ένα απλό σενάριο να λειτουργεί από ένα καινούργιο shell χωρίς να χρειαστεί να ξεκινήσετε πρώτα ένα backend. Περάστε το `auto_start_backend=False` για να το απενεργοποιήσετε (π.χ. όταν παραπέμπετε σε ένα απομακρυσμένο backend, το οποίο δεν εκκινείται ποτέ). Δείτε [Αυτόματη εκκίνηση backend](#backend-auto-start). Η επιφάνεια 3 συμπεριφέρεται διαφορετικά: το `open_project()` δεν δέχεται την παράμετρο `auto_start_backend`, και το `connect_all()` δεν εκκινεί ποτέ backend — πραγματοποιεί μία δοκιμή σύνδεσης με το `http://127.0.0.1:5000` και, αν δεν λάβει απάντηση, επιστρέφει σιωπηλά στον άμεσο (χωρίς backend) έλεγχο συσκευής `lattice_sdk`. Μόνο οι `proj.process()` και `stream(..., overlays=True)` δημιουργούν με καθυστέρηση ένα `ChlorosLocal()` (το οποίο εκτελεί αυτόματη-εκκίνηση).

Και οι τρεις απαιτούν έλεγχο ταυτότητας: εκτελέστε το `chloros-cli login` μία φορά στον υπολογιστή ή συνδεθείτε μέσω του γραφικού περιβάλλοντος εργασίας. Οι κλήσεις του SDK χωρίς έγκυρη συνεδρία προκαλούν το σφάλμα `ChlorosAuthenticationError`.

Απαιτήσεις:
- Python 3.7+ (όπως δηλώνεται από το πακέτο· αναπτύχθηκε/δοκιμάστηκε στην έκδοση 3.10)
- Το Chloros Desktop εγκατεστημένο τοπικά (το εκτελέσιμο του backend περιλαμβάνεται στο πρόγραμμα εγκατάστασης)
- Ενεργή σύνδεση στο Chloros+. Το ελάχιστο επίπεδο πρόσβασης στο SDK / CLI είναι το επίπεδο **Copper**ή υψηλότερο (Copper / Bronze / Silver / Gold). Το δωρεάν επίπεδο**Iron**δεν παρέχει πρόσβαση στο SDK / CLI. Αυτό επιβάλλεται**από την πλευρά του διακομιστή**: κάθε αίτημα με την ένδειξη SDK / CLI πρέπει να συνοδεύεται τόσο από ενεργή συνεδρία όσο και από πληρωμένο πακέτο, διαφορετικά το backend επιστρέφει το `403` με το `error_code: PLAN_UPGRADE_REQUIRED` (που εμφανίζεται ως `ChlorosLicenseError` από το `ChlorosLocal`, και ως `ChlorosConnectError` από τους βοηθητικούς κώδικες `connect_*`). Ένας καλών που έχει αποσυνδεθεί λαμβάνει τα σφάλματα `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) αντ’ αυτού — τα δύο είναι διαφορετικά, επειδή η επανεκτέλεση του `chloros-cli login` διορθώνει το πρώτο, αλλά δεν μπορεί να διορθώσει το δεύτερο.
- Η χρήση εκτός σύνδεσης υποστηρίζεται εντός της περιόδου χάριτος του προγράμματος: η κατηγορία διαβάζεται από την προσωρινή μνήμη επικύρωσης διακομιστή (5 λεπτά) ή από την υπογεγραμμένη, συνδεδεμένη με τη συσκευή προσωρινή μνήμη άδειας χρήσης (30 ημέρες για τα μηνιαία προγράμματα, έως τη λήξη της συνδρομής για τα ετήσια). Όταν λήξει αυτή η περίοδος χάριτος, το πρόγραμμα μετατρέπεται σε δωρεάν και η πρόσβαση στα SDK / CLI διακόπτεται έως ότου ο υπολογιστής μπορέσει να συνδεθεί με τον διακομιστή μία φορά. Το `chloros-cli status` (`GET /api/license-status`) παραμένουν προσβάσιμα στο δωρεάν επίπεδο, οπότε ο λόγος είναι εμφανής — είναι η μόνη διαδρομή SDK / CLI που εξαιρείται από τον περιορισμό του επιπέδου.
- Windows 10/11 64-bit, **Ubuntu 22.04 LTS ή νεότερη έκδοση**, ή Jetson (JetPack 6). Το Ubuntu 20.04**δεν** υποστηρίζεται: οι εξαρτήσεις του `.deb` προέρχονται από αυτά με τα οποία συνδέεται το backend, συμπεριλαμβανομένου του `libc6 (>= 2.34)`, και η έκδοση Focal διαθέτει το glibc 2.31.

---

## Εγκατάσταση

Το Python SDK είναι ένα λεπτό στρώμα Python πάνω από το backend Chloros. Για οτιδήποτε πέρα από μερικές ροές εργασίας που αφορούν αποκλειστικά τη συλλογή δεδομένων (DAQ), χρειάζεστε το **πακέτο επιφάνειας εργασίας Chloros εγκατεστημένο τοπικά** (πρόγραμμα εγκατάστασης Windows ή Linux `.deb`) — αυτό παρέχει το εκτελέσιμο του backend, το περιβάλλον εκτέλεσης Arena SDK για κάμερες LATTICE και τα πακέτα βαθμονόμησης.

Πρόσφατες λήψεις: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Βήμα 1 — Εγκατάσταση του πακέτου πλατφόρμας «Chloros»

#### Windows (.exe)

1. Κατεβάστε το `Chloros-Setup-x.y.z.exe` από τη σελίδα λήψεων.
2. Εκτελέστε το πρόγραμμα εγκατάστασης και ακολουθήστε τον οδηγό. Η προεπιλεγμένη διαδρομή εγκατάστασης είναι το `C:\Program Files\MAPIR\Chloros\`.
3. Εκκινήστε το Chloros τουλάχιστον μία φορά και συνδεθείτε με τον λογαριασμό σας στο Chloros+.

#### Linux amd64 (.deb)

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

### Βήμα 2 — Εγκατάσταση του Python SDK

**Το πρόγραμμα εγκατάστασης Chloros περιλαμβάνει ένα αντίστοιχο wheel SDK.** Κάθε πρόγραμμα εγκατάστασης Windows και κάθε αρχείο .deb Linux τοποθετεί στον δίσκο ένα `chloros_sdk-X.Y.Z-py3-none-any.whl` που αντιστοιχεί ακριβώς στην έκδοση του GUI / CLI / backend. Δεν χρειάζεται να παρακολουθείτε το PyPI για να παραμείνετε συγχρονισμένοι.

#### Windows

Το πρόγραμμα εγκατάστασης εκτελεί αυτόματα το `pip install` με βάση το συνημμένο αρχείο wheel, χρησιμοποιώντας το Python του συστήματός σας (προτιμάται το πρόγραμμα εκκίνησης `py.exe`, ενώ σε περίπτωση αποτυχίας χρησιμοποιείται το `python -m pip`). Δεν απαιτείται καμία ενέργεια — το `import chloros_sdk` λειτουργεί στο περιβάλλον σας Python μετά από επιτυχή εγκατάσταση. Εάν δεν υπάρχει το Python στον υπολογιστή, το πρόγραμμα εγκατάστασης παραλείπει αυτόματα αυτό το βήμα και το GUI + το CLI συνεχίζουν να λειτουργούν.

#### Linux (.deb)

Το αρχείο .deb τοποθετεί το wheel στο `/usr/lib/chloros/sdk/`. Το `postinst` εμφανίζει την ακριβή εντολή — οι διανομές PEP 668 απορρίπτουν εξ ορισμού τις εγγραφές στο global pip, οπότε δεν πραγματοποιούμε αυτόματη εγκατάσταση:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Για εγκαταστάσεις Jetson σε περιβάλλον χωρίς σύνδεση με το διαδίκτυο (air-gapped), η διαδικασία είναι πλήρως εκτός σύνδεσης — το wheel βρίσκεται ήδη στο δίσκο.

#### Δημόσιο PyPI

Για κεντρικούς υπολογιστές που χρησιμοποιούν μόνο το pip (δεν έχει εγκατασταθεί το πακέτο επιφάνειας εργασίας Chloros· ροές εργασίας απομακρυσμένου backend ή μόνο DAQ):

```bash
pip install chloros-sdk
```

Το PyPI ενημερώνεται στις εκδόσεις του προγράμματος εγκατάστασης με την έκδοση κυκλοφορίας, οπότε το δημοσιευμένο αρχείο wheel αντιστοιχεί στην πιο πρόσφατη σταθερή έκδοση. Οι εκδόσεις ανάπτυξης (π.χ. `1.1.4.dev1`) διατίθενται μόνο μέσω του ενσωματωμένου wheel του προγράμματος εγκατάστασης.

#### Επαλήθευση

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Απαιτείται συνδρομή στο Chloros+.** Όλες οι κλήσεις προς το SDK απαιτούν ενεργή σύνδεση στο Chloros+. Εκτελέστε το `chloros-cli login user@example.com 'YourPassword'` μία φορά ανά υπολογιστή· τα διαπιστευτήρια αποθηκεύονται προσωρινά στο `~/.chloros/`.

### Χρειάζομαι το πακέτο για επιτραπέζιους υπολογιστές;

Το πακέτο pip από μόνο του **δεν** αρκεί για τις περισσότερες ροές εργασίας. Ακολουθούν οι απαιτήσεις για κάθε επιφάνεια SDK:

| ΕπιφάνειαSDK | Χρειάζεται το πακέτο Desktop; | Γιατί |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Ναι** | Εκκινεί αυτόματα το δυαδικό αρχείο του backend στο `/usr/lib/chloros/chloros-backend` (Linux) ή στο `C:\Program Files\MAPIR\Chloros\…` (Windows). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Ναι**(τοπικό)**/ Όχι**(απομακρυσμένο) | Αμιγείς πελάτες HTTP μέσω του backend. Τοπικό backend → απαιτείται πακέτο για επιτραπέζιους υπολογιστές. Απομακρυσμένο backend → `backend_url=`**μέσω σήραγγας** (βλ. Λειτουργία απομακρυσμένου backend — τα ενσωματωμένα backends συνδέονται μόνο με loopback). |
| `ChlorosProject` / `open_project` | **Ναι** | Διαχειρίζεται αποθηκευμένα έργα μέσω του backend. |
| Άμεσες κλάσεις LATTICE (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Ναι** | Απαιτείται το εγγενές περιβάλλον εκτέλεσης Arena SDK που περιλαμβάνεται στο πακέτο για επιτραπέζιους υπολογιστές. Διαφορετικά, το `CAMERA_AVAILABLE` είναι το `False` κατά την εισαγωγή. |
| Άμεσες κλάσεις DAQ (`DAQUSensor`, `DAQMSensor`, `DAQESensor`, `SensorFleet`, `discover_all`) | **Όχι** | Αμιγώς Python μέσω pyserial/bleak/zeroconf. Ένα περιβάλλον που χρησιμοποιεί αποκλειστικά το pip μπορεί να ελέγχει τις συσκευές DAQ από άκρο σε άκρο. |

### Λειτουργία απομακρυσμένου backend (κεντρικός υπολογιστής που χρησιμοποιεί αποκλειστικά το pip, μέσω σήραγγας)

> **Το ενσωματωμένο backend δεν είναι προσβάσιμο μέσω του LAN.** Οι εκδόσεις
> παραγωγής συνδέονται μόνο με loopback (και στις δύο οικογένειες loopback) και απορρίπτουν κατηγορηματικά τη
> μόνη λειτουργία χωρίς loopback (`CHLOROS_CLOUD_MODE`), οπότε
> το `backend_url="http://<lan-ip>:5000"` **δεν μπορεί να λειτουργήσει με ένα εγκατεστημένο
> Chloros** — αυτό το μοτίβο λειτουργούσε πάντα μόνο με backend τύπου source/dev
> . Για να ελέγξετε ένα backend σε άλλο μηχάνημα, προωθήστε τη θύρα loopback
> του μηχανήματος αυτού και κατευθύνετε το SDK προς το τούνελ:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"   # the tunnel endpoint

chloros_sdk.connect_camera("213800234", backend_url=BACKEND)
chloros_sdk.connect_array(serials, backend_url=BACKEND)
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local", backend_url=BACKEND)
```

Οι κεντρικοί υπολογιστές χωρίς οθόνη / CI / ρομποτικής μπορούν να διατηρήσουν έναν υπολογιστή με την πλήρη εγκατάσταση επιφάνειας εργασίας ως «Chloros server» και `pip install chloros-sdk` παντού αλλού — αλλά η μεταφορά μεταξύ τους γίνεται μέσω της παραπάνω σήραγγας που έχει ρυθμίσει ο χρήστης, όχι μέσω άμεσης σύνδεσης LAN URL.

> **Γνωστός περιορισμός — το `ChlorosLocal` δεν υποστηρίζει αποκλειστικά το pip.** Το `ChlorosLocal(backend_url=BACKEND)` επί του παρόντος επιλύει ένα τοπικό δυαδικό αρχείο backend στον κατασκευαστή του *πριν* διερευνήσει το URL και εμφανίζει το σφάλμα `ChlorosBackendError` («Δεν βρέθηκε το backend του Chloros…») όταν δεν είναι εγκατεστημένο κανένα πακέτο επιφάνειας εργασίας — ακόμη και με ένα προσβάσιμο απομακρυσμένο backend. Μόνο η επιφάνεια «smart-connect» παραπάνω (`connect_camera` / `connect_array` / `connect_daq_sensor`, καθώς και τα βοηθητικά προγράμματα `analyze_array_network` και `list_*` / `discover_*`) λειτουργούν από έναν κεντρικό υπολογιστή που διαθέτει μόνο το pip.

### Ροή εργασίας μόνο για DAQ (κεντρικός υπολογιστής μόνο με το πακέτο pip)

Εάν χρειάζεστε μόνο αισθητήρες DAQ και δεν χρησιμοποιείτε κάμερες LATTICE ή επεξεργασία εικόνας, το πακέτο pip είναι αυτόνομο:

```bash
pip install chloros-sdk
```

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

sensor = DAQUSensor(port="/dev/ttyUSB0")
sensor.connect()
sensor.start_streaming()
```

Χωρίς backend, χωρίς .deb, χωρίς Chloros+ απαιτείται σύνδεση για εργασία DAQ απευθείας με το υλικό.

---

## Γρήγορη εκκίνηση

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## Ευρετήριο κορυφαίου επιπέδου API

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()

# === Project lifecycle ===
chloros_sdk.open_project(path)                    # → ChlorosProject
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk / daq_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics
chloros_sdk.DAQUSensor, DAQMSensor, DAQESensor, SensorFleet, discover_all

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.DAQ_AVAILABLE        # True iff daq_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True iff ChlorosProject deps available
```

---

## Επεξεργασία εικόνων — `ChlorosLocal`

Η κύρια κλάση του pipeline. Εκκινεί το backend κατά την πρώτη χρήση, δημιουργεί και διαμορφώνει έργα, παρακολουθεί την πρόοδο και επιστρέφει περιλήψεις μετά την εκτέλεση.

### Κατασκευαστής

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

### Μέθοδοι

| Μέθοδος | Περιγραφή |
| --- | --- |
| `create_project(project_name, camera=None)` | Δημιουργία νέου έργου (προαιρετικά με πρότυπο κάμερας όπως το `"Survey3N_RGN"`). |
| `import_images(folder_path, recursive=False)` | Εισαγωγή εικόνων RAW/TIF/JPG/DNG **και καταγραφών αισθητήρα φωτός `.daq`**. Επιστρέφει `count` (εικόνες) και `scan_count` (καταγραφές). Εμφανίζει προειδοποίηση μόνο αν ο φάκελος δεν περιέχει κανένα από τα δύο. |
| `export_light_sensor(daq=True, csv=True)` | Εγγραφή βαθμονομημένων `.daq` + `.csv` για κάθε καταγραφή αισθητήρα φωτός του έργου, στο `<project>/Light Sensor/`. Βλ. [Καταγραφές αισθητήρα φωτός](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Ρύθμιση παραμέτρων επεξεργασίας. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Εκτέλεση της ροής επεξεργασίας. Επιστρέφει το `{"status": "complete", "async": False}`, καθώς και ένα `summary` όταν το backend παρέχει ένα — βλ. [Σύνοψη &amp; Συμβουλές μετά την εκτέλεση](#post-run-summary--hints). |
| `get_config()` / `get_status()` / `status()` | Ελέγξτε την κατάσταση του backend. |
| `logout()` | Εκκαθάριση αποθηκευμένων διαπιστευτηρίων. |
| `shutdown_backend()` | Τερματισμός του backend (εάν ξεκίνησε με την επιλογή `SDK`). |
| `discover_cameras()` | Εντοπισμός καμερών LATTICE **μέσω του backend αυτής της παρουσίας** (`/api/camera/discover`). Επιστρέφει μια λίστα λεξικών (`serial`, `model`, `ip`, …) — με την ίδια μορφή που βλέπει το GUI/CLI. Κενή λίστα αν δεν βρέθηκε καμία ή αν το backend είναι απρόσιτο. |
| `camera_capture(output_dir, format="tiff", **settings)` | Λαμβάνει ένα μεμονωμένο καρέ**μέσω του backend**(αυτόματη εκκίνηση από αυτό το handle), ώστε να λάβει την ίδια προετοιμασία με το GUI/ CLI (προεπιλογή 12-bit, επαναχρησιμοποίηση pool, ενσωματωμένα μεταδεδομένα βαθμονόμησης). Επεξεργαστείτε τον στόχο με τα `serial=` ή `device_index=`· περάστε τα `exposure`/`gain`/`pixel_format`/`preset` ως `**settings`. Επιστρέφει το παλαιό λεξικό μεταδεδομένων (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Παραγωγή καρέ προεπισκόπησης με σύνθετη επικάλυψη από μια συγκεντρωμένη κάμερα — ελαφρύς πελάτης MJPEG μέσω της διαδρομής `/api/camera/<serial>/stream-annotated` του backend (ζέβρα / πλέγμα / σταυρόνημα / ιστόγραμμα / κορυφή / σημείο που σχεδιάζονται από την πλευρά του διακομιστή). Το `decode=True` παράγει πίνακες BGR· το `False` παράγει ακατέργαστα bytes τύπου «JPEG». Προσβάσιμο επίσης ανά έργο ως `ChlorosProject.stream(overlays=True)`. |

Χρησιμοποιήστε το ως διαχειριστή περιβάλλοντος για εγγυημένο καθαρισμό:

```python
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

### Καταγραφές αισθητήρα φωτός — βαθμονομημένες `.daq` + `.csv`

Ένα DAQ-U / DAQ-M / DAQ-E μπορεί να καταγραφεί **χωρίς** το πακέτο βαθμονόμησής του. Αυτό είναι
αυτό που οι δημόσιες [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`): καταγράφουν τις ακατέργαστες μετρήσεις του αισθητήρα και προσθέτουν σφραγίδα στο
αρχείο, ώστε το Chloros να ανακτά την εργοστασιακή βαθμονόμηση του συγκεκριμένου αισθητήρα **ανά σειριακό αριθμό** — πρώτα από την τοπική προσωρινή μνήμη
και μετά από το MAPIR Cloud — και την εφαρμόζει κατά την εισαγωγή.

Το Chloros αποθηκεύει το αποτέλεσμα ως δύο προϊόντα ανά καταγραφή, υπό το
`<project>/Light Sensor/`:

| Προϊόν | Τι είναι |
| --- | --- |
| `<name>_calibrated.daq` | Το αρχείο που μπορεί να υποβληθεί σε επανεπεξεργασία — ίδιο σχήμα με μια ζωντανή καταγραφή, που τώρα δηλώνει το πακέτο που το παρήγαγε. Η επανεισαγωγή του **δεν** το βαθμονομεί για δεύτερη φορά. |
| `<name>_calibrated.csv` | Φασματική ακτινοβολία σε W/m²/nm στο δικό του πλέγμα μηκών κύματος του αισθητήρα, μία σειρά ανά μέτρηση, συν φωτομετρικές στήλες (συνολική ισχύς, φωτοπικά/σκοτοπικά lux, PPFD και η μπλε/πράσινη/κόκκινο, μέγιστο μήκος κύματος). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Μόνο για αισθητήρες χωρίς δέσμη δεδομένων (DAQ-A).** Ακατέργαστες φασματικές μετρήσεις αισθητήρα — *όχι* ακτινοβολία. Βλ. παρακάτω. |

Το `process()` εκτελεί αυτή την εξαγωγή ως ένα από τα στάδιά του. **Δεν** απαιτεί εικόνες:
ένας αισθητήρας φωτός που πετάει μόνος του αποτελεί ροή εργασίας πρώτης κατηγορίας, και ένα τέτοιο έργο έχει μηδέν
εικόνες εκ φύσεως.

**Οι καταγραφές DAQ-A εξάγονται ως ακατέργαστες μετρήσεις.** Η οικογένεια DAQ-A προηγείται του συστήματος
δέσμης ανά σειριακό αριθμό και δεν διαθέτει δέσμη προς ανάκτηση — αντίθετα, βαθμονομείται στο πεδίο έναντι ενός
στόχου ανακλαστικότητας, γι’ αυτό και δεν χρειάστηκε ποτέ ένα. Αυτές οι καταγραφές εξάγονται
με ρίζα `_raw` αντί για `_calibrated`: ένα διαφορετικό όνομα αρχείου αντί για μια σημαία
μέσα του αρχείου, επειδή η ένδειξη πρέπει να διατηρηθεί κατά την αποστολή μέσω ηλεκτρονικού ταχυδρομείου ως απλό όνομα. Η
κεφαλίδα `.csv` αναγράφει `raw spectral sensor counts (NOT irradiance)` και προειδοποιεί ότι οι
τιμές είναι συγκρίσιμες **εντός** του αρχείου — ακριβώς για τον σκοπό για τον οποίο τις χρησιμοποιεί η βαθμονόμηση βάσει στόχου
— — και όχι μεταξύ αισθητήρων. Οι φωτομετρικές στήλες που εξαρτώνται από την ισχύ (συνολική ισχύς,
φωτοπικά/σκοτοπικά lux, PPFD) επιστρέφουν **NULL** αντί να υπολογίζονται από τις μετρήσεις.

Ένα DAQ-U / DAQ-M / DAQ-E του οποίου η δέσμη απλά δεν μπόρεσε να ανακτηθεί εξακολουθεί να **παραλείπεται**,
δεν εγγράφεται σε μορφή raw: σε αυτή την περίπτωση το πακέτο υπάρχει και η «επανασύνδεση και επανεπεξεργασία» αποτελεί πραγματική συμβουλή.

Οι παλαιότερες καταγραφές **v1.01 / v1.02** (ένα DAQ-A-SD τις εγγράφει) δεν φέρουν εποχή ανά ανάγνωση,
μόνο τον χρόνο εγγραφής του αρχείου. Ο μηχανισμός αντιστοίχισης εικόνας↔κατερχόμενου σήματος εξακολουθεί να τις απορρίπτει — η αντιστοίχιση ενός
πλαισίου με μια ώρα εγγραφής θα ήταν λανθασμένη χωρίς να γίνεται αντιληπτό — αλλά ο εξαγωγέας τις διαβάζει, και το
CSV εκτυπώνει `clock=daq_created_on`, ώστε το προϊόν να δηλώνει σε ποιο ρολόι βρίσκεται.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

Μια εγγραφή της οποίας το πακέτο βαθμονόμησης δεν μπορεί να ανακτηθεί (εκτός σύνδεσης ή αισθητήρας χωρίς
βαθμονόμηση στο αρχείο) αναφέρεται υπό τον κωδικό `skipped` **με την αιτία**. Δεν
αποθηκεύεται ως «βαθμονομημένο» αρχείο που περιέχει ακατέργαστες μετρήσεις — συνδεθείτε στο διαδίκτυο και
εκτελέστε ξανά την διαδικασία, και η εξαγωγή θα ολοκληρωθεί.

### Ειδοποιήσεις προόδου

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Περίληψη μετά την εκτέλεση &amp; Συμβουλές

Μετά την ολοκλήρωση, το `process()` ανακτά το `GET /api/processing-summary` και επισυνάπτει το κύριο κείμενο ως `result["summary"]`. Η ανάκτηση γίνεται με βάση την αρχή της «βέλτιστης προσπάθειας» και δεν εμποδίζει ποτέ την επιτυχή επιστροφή — εάν η σύνοψη δεν είναι διαθέσιμη, το `process()` επιστρέφει στην απλή μορφή του `{"status": "complete", "async": False}`. Κάθε καταχώριση στο `summary["hints"]` — πλήρεις προτάσεις με την προτεινόμενη διόρθωση, π.χ. γιατί μια εκτέλεση παρήγαγε μηδενική έξοδο — αναμεταδίδεται επίσης ως «Python» (αναφορά σφάλματος) `UserWarning`, οπότε οι εκτελέσεις με μηδενική έξοδο αυτοδιαγιγνώσκονται ακόμη και αν δεν ελέγξετε ποτέ το λεξικό:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]`Το X είναι το μισό που είναι αναγνώσιμο από τον υπολογιστή:

| Κλειδί | Τι μετράει |
| --- | --- |
| `models` | Ομάδες καμερών στην εκτέλεση. |
| `images_in_groups` | Εικόνες πηγής σε αυτές τις ομάδες. |
| `targets_found` | Ανιχνευθέντες στόχοι ανακλαστικότητας. |
| `images_calibrated` | Εικόνες που βαθμονομήθηκαν κατά τη διάρκεια της εκτέλεσης. |
| `exported_files` | **Αρχεία προϊόντων εικόνων που δημιουργήθηκαν κατά τη διάρκεια της εκτέλεσης.** |
| `daq_recordings_exported` / `daq_recordings_skipped` | Καταγραφές αισθητήρα φωτός, που καταμετρήθηκαν ξεχωριστά σκόπιμα — προέρχονται από διαφορετικό στάδιο και υπάρχουν για εκτελέσεις χωρίς καθόλου εικόνες, οπότε η συμπερίληψή τους θα έκανε μια εκτέλεση μόνο DAQ να φαίνεται σαν να εξήγαγε εικόνες. |

Μαζί με αυτά: `summary["output_dirs"]` (κάθε κατάλογος στον οποίο έγινε εγγραφή),
`summary["light_sensor_export"]`, `summary["stopped"]` (ισχύει όταν ο χρήστης διέκοψε την
εκτέλεση, ώστε οι μερικές μετρήσεις να μην εμφανίζονται ως ολοκληρωμένη εκτέλεση με ανεπαρκή παραγωγή), και
`summary["groups"]` (η ανάλυση ανά ομάδα).

Το `exported_files` καταγράφεται από τον αγωγό **καθώς γράφει**, και δεν αντλείται από τα
αντικείμενα εικόνας του έργου εκ των υστέρων. Οι στρατηγικές παράλληλης επεξεργασίας και GPU δημιουργούν τα δικά τους αντικείμενα
εικόνας (σε υποδιαδικασίες εργαζομένων για τις διαδρομές GPU), οπότε η παλιά σάρωση ανέφερε
`0 file(s) written` για κάθε τέτοια εκτέλεση και στη συνέχεια εξέδωσε την υπόδειξη μηδενικών εξαγωγών — σε εκτελέσεις
όπου όλα είχαν λειτουργήσει. Εάν δημιουργήσετε σενάριο με βάση αυτόν τον αριθμό, μια υγιής παράλληλη εκτέλεση τώρα
αναφέρει έναν μη μηδενικό αριθμό.

Οι παραλείψεις του αισθητήρα φωτός αναφέρουν τον λόγο που ο αναγνώστης διαπίστωσε πράγματι για κάθε αρχείο — ένα
μη αναγνώσιμο σχήμα, ένα πακέτο που λείπει, ένα σφάλμα εγγραφής — **χωρίς διπλές καταχωρήσεις**, οπότε είκοσι αρχεία
που παραλείφθηκαν για έναν λόγο εμφανίζονται ως ένας λόγος και όχι ως είκοσι επαναλήψεις του.

> **Το `process()` δεν εμφανίζεται όταν μια εκτέλεση δεν παράγει εικόνες.** Αυτό είναι το μόνο σημείο στο οποίο το SDK και
> το CLI διαφέρουν σκόπιμα: το `chloros-cli process` αντιμετωπίζει το «ζητήθηκαν προϊόντα, κανένα δεν
> γράφτηκε» ως αποτυχία και τερματίζεται με μη μηδενικό αποτέλεσμα, ενώ το SDK επιστρέφει κανονικά και αναφέρει την
> κατάσταση μέσω του `summary` / υπαινιγμούς. Εάν η ροή εργασιών σας πρέπει να σταματήσει σε περίπτωση κενής εκτέλεσης, ελέγξτε την
> οι ίδιοι — ελέγξτε το `summary` (ή μετρήστε τα αρχεία στον φάκελο του έργου) αντί να βασίζεστε στην
> απουσία εξαίρεσης. Οι συνήθεις αιτίες είναι ένας φάκελος εισόδου που δεν αναγνωρίστηκε ως
> καταγραφή και προϊόντα που παραλείφθηκαν ως μη εφαρμόσιμα για τις υπάρχουσες κάμερες (π.χ. ακτινοβολία μόνο από κάμερες τύπου RGB
>).

### Λειτουργίες ευκολίας

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

### Υποστηριζόμενες τιμές

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as already-debayered BGR
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Ραδιομετρική έξοδος (ακολουθία επεξεργασίας πολυφασματικών δεδομένων LATTICE)

Το επίπεδο εξαγωγής πολυφασματικών δεδομένων LATTICE (M3C/M3M) της διαδικασίας `process` — `reflectance` (προεπιλογή), `radiance`, `sensor-response` ή `all` (κάθε εφαρμόσιμη λειτουργία ανά εικόνα) — αντιστοιχεί στη ρύθμιση επεξεργασίας **«Ραδιομετρική έξοδος»** του έργου. Το `configure()` διαθέτει μια ειδική λέξη-κλειδί για αυτό:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

Η προηγμένη λύση διαφυγής — η εγγραφή του κλειδιού `"Radiometric output"` του έργου μέσω του `custom_settings` — εξακολουθεί να λειτουργεί, αλλά να θυμάστε ότι αντικαθιστά ολόκληρο το μπλοκ ρυθμίσεων (δείτε την παρακάτω προειδοποίηση):

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Radiometric output": "radiance"},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

Το `reflectance` (η προεπιλογή) διαιρεί την ακτινοβολία της κάμερας με το **χρονικό σήμα-που αντιστοιχεί στο DAQ downwelling**, η οποία προσδιορίζεται αυτόματα από ένα καταγεγραμμένο `.daq` (DAQ-U/M/E)**ή ένα εγγενές `.csv`* του DAQ-M* που βρίσκεται μαζί με τις εικόνες· οποιοδήποτε πακέτο βαθμονόμησης ανά κάμερα ή DAQ που λείπει τοπικά **ανακτάται αυτόματα από το AWS** κατά την πρώτη χρήση. Το CLI το παρουσιάζει ως επιλογές ανά τύπο προϊόντος στο `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.

> Το `custom_settings` **αντικαθιστά** ολόκληρο το μπλοκ υπολογισμένων ρυθμίσεων (παρακάμπτει τις άλλες λέξεις-κλειδιά και την επικύρωση του `configure()`, σύμφωνα με τον σχεδιασμό). Όταν το χρησιμοποιείτε, συμπεριλάβετε κάθε κλειδί `Project Settings` που σας ενδιαφέρει, όπως στο παραπάνω παράδειγμα.

---

## Smart-Connect για κάμερες LATTICE

Μόνιμες συνεδρίες backend για ζωντανό υλικό. Χρησιμοποιούνται τα ίδια τελικά σημεία με αυτά του GUI, οπότε η συμπεριφορά είναι πανομοιότυπη στα SDK / CLI / GUI.

### Μία κάμερα — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

#### Υπογραφή `connect_camera()`

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

#### `CameraSession` Μέθοδοι

| Μέθοδος | Περιγραφή |
| --- | --- |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Ανάγνωση κόμβων GenICam· επιστρέφει `{nodes, errors, enums, device}`. |
| `set_settings(**kwargs)` | Εγγραφή κόμβων με φιλικό όνομα (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, settings=None, timeout=None)` | Λήψη **ενός** καρέ. Επιστρέφει μια λίστα ενός στοιχείου με λεξικά μεταδεδομένων καρέ. (Η λήψη σε ριπή/πολλαπλών καρέ καταργήθηκε — καλέστε το `capture()` σε βρόχο αν χρειάζεστε μια σειρά.) |
| `disconnect()` | Απελευθέρωση από την ομάδα. Χωρίς αποτέλεσμα αν έχουμε συνδεθεί σε μια ήδη ανοιχτή συνεδρία. |

`capture()` έλεγχοι εξαγωγής (ίδιο μοντέλο με τον πίνακα + GUI):

- `processing` / `levels` — Το `processing="all"` αποθηκεύει κάθε εφαρμόσιμο τύπο εξαγωγής· Το `levels=["raw","radiance"]` αποθηκεύει μόνο αυτά (παρακάμπτει το `processing`). Παραλείψτε και τα δύο για την προεπιλογή του backend.
- `force_daq=True` — αποθήκευση της εκχωρημένης ανάγνωσης DAQ/DLS ως sidecar `.daq` ακόμη και σε λήψη μόνο σε μορφή raw, ώστε το καρέ να μπορεί να υποβληθεί σε επανεπεξεργασία σε ανακλαστικότητα/δείκτη αργότερα. Δεν εκτελείται καμία ενέργεια αν δεν είναι συνδεδεμένο DAQ.

### Συγχρονισμένη συστοιχία — `ArraySession` (Smart-Prep)

Το `connect_array` είναι **το συνιστώμενο σημείο εισόδου** για διαμορφώσεις πολλαπλών καμερών. Εκτελεί εσωτερικά την πλήρη ροή smart-prep μέσω του γραφικού περιβάλλοντος χρήστη (GUI):

1. **Ανάλυση δικτύου** (`/api/camera/array/recommend`) — εντοπίζει το μεγαλύτερο μέγεθος πλαισίου που ταιριάζει στο επίπεδο simεπιπέδου sim-emit χωρίς απώλεια καρέ.
2. **Αυτόματη επιλογή επιπέδου** — `sim-capture-sim-emit` εάν το καλώδιο μπορεί να το υποστηρίξει· διαφορετικά, `sim-capture-ftd-stagger` ή `slip-emit-and-capture`.
3. **Αυτόματη συρρίκνωση**— συρρικνώνει αθόρυβα το μέγεθος του πλαισίου / αυξάνει την ομαδοποίηση όταν η γραμμή δεν μπορεί να υποστηρίξει την απαιτούμενη ανάλυση.**Αυτό το δίχτυ ασφαλείας δεν καλύπτει τη συνολική υπερ-εγγραφή**: ο υπερβολικός αριθμός καμερών για το καλώδιο δεν μπορεί να διορθωθεί με τη συρρίκνωση των καρέ — βλ. [Υπερ-εγγραφή](#over-subscription-the-per-cam-floor).
4. **PTP ενεργοποιημένο**από προεπιλογή — οι χρονικές σημάνσεις μεταξύ καμερών συγχρονίζονται σε ένα κοινό ρολόι με ακρίβεια**~1 ms**. Η ταυτόχρονη έκθεση προέρχεται από το υλικό trigger του M8 (**&lt; 100 µs** μεταξύ μονάδων), όχι από το PTP: το PTP ευθυγραμμίζει τις *χρονικές σημάνσεις*, όχι τις εκθέσεις.
5. **Αυτόματη επιλογή μορφής εικονοστοιχείων ανά κάμερα** — Κάμερες RGB → `BayerRG8`, πολυφασματικές → `BayerRG12`.
6. **Αρχική ρύθμιση AE** — καταγράφει την τρέχουσα κατάσταση AE κάθε κάμερας, ώστε η σύνδεση να μηννα μηδενιστεί η έκθεση εν μέσω λειτουργίας.
7. **Διαμόρφωση ενεργοποίησης GPIO** — Το `connect_array` ενεργοποιεί κάθε κάμερα (`TriggerMode=On`, `TriggerSource=Line2`), ώστε ο παλμός του κύριου να οδηγεί τους δευτερεύοντες μέσω μέσω του καλωδίου M8. Αυτό είναι ένα βήμα που ισχύει μόνο για συστοιχίες: μια μεμονωμένη κάμερα που ενεργοποιείται με το `LatticeCamera` λειτουργεί ανεξάρτητα.

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

#### Υπογραφή `connect_array()`

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # master trigger fire rate (auto if None)
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=180.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

Τιμές `force_tier`:
- `"sim-capture-sim-emit"` — πραγματικά ταυτόχρονη λειτουργία (όλες οι κάμερες ενεργοποιούνται στην ίδια ακμή ρολογιού).
- `"sim-capture-ftd-stagger"` — ευέλικτη χρονική κλιμάκωση (οι κάμερες εκπέμπουν σε ελαφρώς μετατοπισμένους χρόνους, ώστε τα πακέτα να σειριοποιούνται στο καλώδιο).
- `"slip-emit-and-capture"` — διαδοχική καταγραφή ανά κάμερα (χωρίς χρονικό συγχρονισμό· η μόνη επιλογή όταν κανένα μέγεθος πλαισίου δεν ταιριάζει με την ταυτόχρονη λειτουργία).

Η τιμή `wire_ceiling_mbps` παρακάμπτει τον **συνεχή προϋπολογισμό καλωδίου του κεντρικού υπολογιστή** σε MB/s — τον μοναδικό
αριθμό από τον οποίο εξαρτάται ολόκληρη η κατανομή της συστοιχίας. Αφήστε το `None` για να χρησιμοποιηθεί η αυτόματα ανιχνευόμενη
τιμή. Μειώστε το όταν η συστοιχία αναφέρει κατεστραμμένα πλαίσια GVSP: η αυτόματη τιμή προκύπτει
από τον διαφημιζόμενο ρυθμό σύνδεσης της κάρτας δικτύου (NIC), ο οποίος υπερεκτιμά τους προσαρμογείς USB, τις λεπτές λωρίδες PCIe και
απασχολημένα κοινόχρηστα δίκτυα — και αυτή η υπερεκτίμηση εμφανίζεται ως κατεστραμμένα πλαίσια και όχι ως
ορατά αργή σύνδεση. Η τιμή αποθηκεύεται μόνιμα στο μπλοκ καταγραφής συστοιχιών του έργου, οπότε μια
επανέναρξη ή μια μεταγενέστερη ρύθμιση του `connect_array` την επαναφέρει όπως οποιαδήποτε άλλη ρύθμιση συστοιχίας.
Δείτε [Κατάσταση συστοιχίας](#array-health--which-subsystem-is-losing-frames).

#### Υπερ-εγγραφή (το κατώτατο όριο ανά κάμερα)

Ο ρυθμός εκπομπής προσομοίωσης (Sim-emit pacing) κατανέμει σε κάθε κάμερα ένα μερίδιο του προϋπολογισμού καλωδίου με ασφάλεια από συγκρούσεις, με κατώτατο όριο τα **8 MB/s ανά κάμερα**(`per_cam_floor_bps`). Μόλις το `N × floor` υπερβεί το ανώτατο όριο ασφαλούς από συγκρούσεις, το array**υπερ-εγγραφεί το δίκτυο**— ο τρόπος αστοχίας είναι η απώλεια πακέτων GVSP, όχι χαμηλότερος ρυθμός καρέ — και δεν υπάρχει λύση που να αφορά το μέγεθος του καρέ:**η ομαδοποίηση και η περιοχή ενδιαφέροντος (ROI) μειώνουν τα byte ανά καρέ, όχι τα ρυθμιζόμενα byte ανά δευτερόλεπτο**που συγκρίνει ο συνολικός έλεγχος. Πρακτικά ανώτατα όρια πλήρους ανάλυσης σε κεντρικό υπολογιστή 1 GbE:**6 κάμερες @ 1500 MTU, 9 με jumbo frames** (το `max_cams_collision_safe` στην απάντηση της ανάλυσης αναφέρει το ανώτατο όριο για τη γραμμή σας). Λύσεις: λιγότερες κάμερες, jumbo frames από άκρο σε άκρο ή ταχύτερη κάρτα δικτύου (NIC).

- Οι απαντήσεις `analyze_array_network()` και `/api/camera/array/connect` περιλαμβάνουν τις απαντήσεις `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` και `per_cam_floor_bps`. Όταν το `oversubscribed` είναι αληθές, η προβολή **μηδενίζει τα πεδία fps** (`achievable_fps_max` / `fps_bright` / `fps_dark`) αντί να αναφέρει έναν παραπλανητικό αργό-αλλά λειτουργικό ρυθμό.
- Το `POST /api/camera/array/connect` δέχεται μια παράμετρο σώματος `pin_resolution` (**μόνο για HTTP — δεν είναι παράμετρος SDK**· το `connect_array` δεν την εκθέτει). Η «καρφώση» (pinning) αφαιρεί το δίχτυ ασφαλείας της σταδιακής μείωσης (walk-down), οπότε μια σύνδεση με υπερβολικό αριθμό συνδρομητών και ρυθμισμένο το `pin_resolution`**απορρίπτεται**απορρίπτεται** με ένα σφάλμα που αναφέρει κάθε δυνατή λύση. Χωρίς το pinning, η σύνδεση προχωρά με τη διαδικασία walk-down, αλλά προειδοποιεί ότι η συρρίκνωση δεν μπορεί να εκκαθαρίσει το σύνολο.
- Εναλλακτική λύση για εργαστηριακές δοκιμές: ορίστε το `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` στο περιβάλλον του backend για να υποβαθμίσετε την απόρριψη σε μια έντονη προειδοποίηση — θα συνδεθείτε ούτως ή άλλως και αποδέχεστε την απώλεια πακέτων.

#### Κατάσταση συστοιχίας — ποιο υποσύστημα χάνει πλαίσια

Το `GET /api/camera/array/<array_id>/capability` φέρει ένα ενεργό μπλοκ `health` σε μια
συνδεδεμένη συστοιχία, το οποίο επαναξιολογείται σε ένα κυλιόμενο **παράθυρο 10 δευτερολέπτων** . Διαχωρίζει την απώλεια πλαισίων
σε δύο αιτίες που απαιτούν αντίθετες διορθώσεις, αντί για ένα «ατελές» ποσοστό που
δεν προσδιορίζει καμία από τις δύο:

| Πεδίο | Τι σημαίνει | Ποιο υποσύστημα |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (ανά σειριακό) | Το πλαίσιο **έφτασε και ήταν δομικά ελαττωματικό**— απώλεια πακέτων GVSP. |**Δίκτυο**: χωρητικότητα καλωδίου, ρυθμός, δακτύλιος λήψης NIC, MTU |
| `never_arrived_rate_pct` (ανά σειριακό) | Το πλαίσιο **δεν έφτασε καθόλου**— η κάμερα δεν ενεργοποιήθηκε ή δεν εξήλθε τίποτα από αυτήν. |**Ενεργοποίηση / συγχρονισμός**: καλώδιο M8, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Χειρότερος ρυθμός ανά κάμερα για κάθε μία. | — |
| `per_cam_rate_pct` | Συνολικό ποσοστό ατελούς λειτουργίας ανά κάμερα (και οι δύο αιτίες μαζί). | — |
| `stable_for_seconds` | Πόσο καιρό κάθε κάμερα παρέμεινε κάτω από το 0,01 %. | — |

Παράλληλα με το `health`, το ίδιο αρχείο αναφέρει τον αριθμό για τον οποίο παραμένει σε εκκρεμότητα ολόκληρη η κατανομή:

| Πεδίο | Τι σημαίνει |
| --- | --- |
| `wire_ceiling_mbps` | Ο ισχύων προϋπολογισμός εύρους ζώνης του κεντρικού υπολογιστή, MB/s. |
| `wire_ceiling_source` | Από πού προήλθε ο αριθμός αυτός, με λέξεις — π.χ. `USB-capped 200 MB/s (was theoretical 1062; …)` ή `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` όταν το έθεσε το `wire_ceiling_mbps=`. |
| `nic_is_usb` | `true` για έναν προσαρμογέα USB Ethernet. |

Δεν υπάρχει SDK wrapper για αυτό το endpoint — διαβάστε το απευθείας:

```python
import requests, chloros_sdk

arr = chloros_sdk.attach_array(["213800234", "214000533"])
h = requests.get(
    f"http://127.0.0.1:5000/api/camera/array/{arr.array_id}/capability",
    timeout=10).json()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    arr.disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Ανάγνωση:** μημηδενική τιμή `gvsp_corrupt_rate_pct` με το `never_arrived_rate_pct` στο 0 σημαίνει ότι
η ενεργοποίηση και ο συγχρονισμός καλωδίου είναι τέλειοι και το 100 % της απώλειας οφείλεται στη διαδρομή του δικτύου — μειώστε το
`wire_ceiling_mbps` και επανασυνδέστε. Το αντίστροφο μοτίβο υποδεικνύει το καλώδιο συγχρονισμού ή τη
γραμμή ενεργοποίησης.

> **Το `target_fps` δεν αποτελεί παράγοντα για κατεστραμμένα πλαίσια.** Ο ρυθμός GevSCPD καθορίζεται μία φορά κατά τη
> τη σύνδεση, οπότε η μείωση του ρυθμού ενεργοποίησης αλλάζει τον κύκλο λειτουργίας και όχι τον
> ρυθμό εκπομπής ταυτόχρονων ριπών. Μια μετρημένη μείωση της ζήτησης κατά 5× δεν επέφερε καμία βελτίωση, ενώ
> η μείωση του ανώτατου ορίου καλωδίου από 240 σε 200 MB/s μείωσε το ποσοστό κατεστραμμένων δεδομένων της ίδιας διάταξης από 10,4 % σε
> 0,00 %.

> **Η αυτόματη συρρίκνωση εν μέσω ροής δεν είναι διαθέσιμη στο firmware του TRI032S.** Μια συστοιχία που βρίσκεται σε λειτουργία δεν μπορεί
> να το διορθώσει από μόνη της· αποσυνδέστε και επανασυνδέστε, ώστε ο επιλογέας χρόνου σύνδεσης να επαναπρογραμματίσει με βάση
> το νέο ανώτατο όριο.

Ένας **προσαρμογέας Ethernet USB περιορίζεται στα 200 MB/s** από τον αισθητήρα, ανεξάρτητα από την
ονομαστική του τιμή: ο πίνακας απόδοσης που μετατρέπει έναν ρυθμό σύνδεσης σε σταθερή τιμή
παράγεται από το PCIe, ενώ μια κάρτα δικτύου USB ανακοινώνει τον ρυθμό σύνδεσης Ethernet της, αλλά περιορίζεται από το
διαύλο USB και το πρόγραμμα οδήγησής της. Το όριο είναι απόλυτο, όχι κλασματικό — ένας προσαρμογέας USB 1 GbE
παράγει ~80 MB/s και δεν επηρεάζεται.

#### Μέθοδοι `ArraySession`

| Μέθοδος | Περιγραφή |
| --- | --- |
| `status(timeout=10.0)` | Ζωντανό `{fps, ptp, frame_count, last_error, …}`. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, smart=False, timeout=300.0)` | Μία συγχρονισμένη ομάδα καταγραφής. Επιστρέφει ένα `CaptureResult` (λίστα λεξικών πλαισίων + `.skipped`). Έλεγχοι εξαγωγής παρακάτω. |
| `capture(..., smart=True)` | **Έξυπνη καταγραφή** — περιμένει να σταθεροποιηθεί το AE σε όλες τις κάμερες και, στη συνέχεια, ενεργοποιείται. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, timeout=120.0)` | Ταχύτερη λήψη: μόνο ακατέργαστα δεδομένα + η εκχωρημένη ένδειξη DAQ (+ ο ελεύθερος συνδυασμένος δείκτης). Αντιστοιχεί στο κουμπί «Fastest Capture» του GUI. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Ενιαία / Συνεχής / Διάστημα σε έναν περιορισμένο βρόχο. Επιστρέφει το `list[CaptureResult]`.**Απαιτεί το `count` και/ή το `duration_s`** ώστε να τερματιστεί (το «SDK» δεν διαθέτει Ctrl+C). |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, timeout=30.0)` | Ξεκινήστε την εγγραφή της ζωντανής προβολής συνδυασμένου δείκτη σε βίντεο/GIF → `RecorderHandle`. Μία συσκευή εγγραφής σύνθετων εικόνων ανά πίνακα. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, timeout=30.0)` | Έναρξη ραφής ακατέργαστων εικόνων Bayer υψηλού fps → `RecorderHandle`. Επεξεργασία εκ νέου εκτός σύνδεσης με το `build_video()`. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Επεξεργασία εκ νέου εκτός σύνδεσης μιας αποθηκευμένης σειράς λήψεων σε μορφή raw σε βίντεο με βαθμονόμηση. Παύει τη λειτουργία μέχρι να ολοκληρωθεί (`wait=True`) και επιστρέφει το `{outputs, errors, combined}`. |
| `build_video_status(job_id, timeout=15.0)` | Έλεγχος κατάστασης μιας εργασίας δημιουργίας εκτός σύνδεσης: `{running, result, error, burst_dir}`. |
| `disconnect()` | Απελευθέρωση ολόκληρου του πίνακα. |

Έλεγχοι εξαγωγής `capture()` (ίδιο τελικό σημείο με αυτό που χρησιμοποιούν το GUI/CLI):

- `processing` / `levels` — Το `processing="all"` (ή το `levels=["raw","radiance",…]`) αποθηκεύει κάθε ισχύοντα τύπο εξαγωγής ανά κάμερα· μια μεμονωμένη τιμή `processing` αποθηκεύει μόνο αυτό το επίπεδο.
- `aligned=True` — προσαρμόζει την εξαγωγή κάθε μέλους που δεν είναι ακατέργαστη στο [προφίλ ευθυγράμμισης](#array-alignment) του πίνακα (με κοινή καταχώριση)· η ακατέργαστη παραμένει χωρίς προσαρμογή αλλά φέρει τη μετασχηματισμένη μορφή στα μεταδεδομένα. Επιστρέφει σε μη ευθυγραμμισμένη κατάσταση (με προειδοποίηση που εμφανίζεται στο `alignment` του αποτελέσματος) εάν η συστοιχία δεν διαθέτει προφίλ.
- `render_index=False` — παραλείπει την επικάλυψη δείκτη βλάστησης ανά κάμερα· η προεπιλογή την αποδίδει όπου έχει ρυθμιστεί.
- `force_daq=True` — αποθηκεύει την εκχωρημένη ένδειξη DAQ/DLS ως παράρτημα `.daq`, ακόμη και όταν κανένα επιλεγμένο επίπεδο δεν το απαιτεί.

**ΣυμπίεσηTIFF (ρυθμιστικό «HTTP» μόνο):** Το `ArraySession.capture()` δεν στέλνει κλειδί `compression`, οπότε ισχύει η προεπιλογή του backend — το `POST /api/camera/array/capture` διαβάζει μια παράμετρο σώματος `compression`, `"deflate"` από προεπιλογή (zlib L1 χωρίς απώλειες + οριζόντιος προγνωστικός, ~4,1 MB ανά καρέ πλήρους ανάλυσης). Το `"none"` εγγράφει χωρίς συμπίεση (~6,3 MB/καρέ) με**~5× ταχύτερη εγγραφή** — και τα δύο είναι χωρίς απώλειες και διαβάζονται πανομοιότυπα κατά την εισαγωγή. Το `SDK` δεν εκθέτει καμία παράμετρο `kwarg` γι&#x27; αυτό· η εναλλακτική λύση είναι το ``chloros-cli lattice array-capture --compression none`` ή το ακατέργαστο `HTTP`. Το DEFLATE διατηρεί επίσης το GIL του `Python`, οπότε οι συμπιεσμένες εγγραφές δεν παραλληλοποιούνται στα νήματα εγγραφής ανά κάμερα — Η συνεχής καταγραφή πλήρους ανάλυσης από 8 κάμερες με ρυθμό αισθητήρα απαιτεί το `compression: "none"`. Λεπτομέρειες: [CLI Αναφορά → array-capture](cli-reference.md).**Παρακάμψεις εξαγωγής ανά μέλος (μόνο HTTP):**το ίδιο τελικό σημείο δέχεται επίσης το `exclude_serials` (λίστα — αφαίρεση μελών από το αποθηκευμένο σύνολο· ο πίνακας εξακολουθεί να ενεργοποιείται ως μία συγχρονισμένη ομάδα και τα εξαιρούμενα μέλη επιστρέφονται στο `excluded`), το `serial_levels` (αντικαταστάσεις ανά κάμερα `{serial: [level tokens]}`) και `serial_index` (αντικαταστάσεις επικάλυψης δείκτη ανά κάμερα `{serial: bool}`). Αυτές είναι παράμετροι σώματος με ισοδυναμία GUI και**δεν είναι ακόμα kwargs του SDK**· τα μέλη που απουσιάζουν από τους χάρτες ανατρέχουν στις τιμές `levels` / `render_index` που ισχύουν για ολόκληρο τον πίνακα.

##### Επιθεώρηση παραλειφθέντων καμερών — `CaptureResult.skipped`

Η `ArraySession.capture()` επιστρέφει μια `CaptureResult`, η οποία είναι υποκατηγορία της `list`: επαναλάβετε το, δημιουργήστε ευρετήριο για αυτό, χρησιμοποιήστε το `len()` — κάθε υπάρχον πρότυπο συνεχίζει να λειτουργεί. Ο νέος κώδικας μπορεί να ελέγξει το χαρακτηριστικό `.skipped` για να δει ποιες κάμερες αποκλείστηκαν και για ποιο λόγο. Η πιο συνηθισμένη περίπτωση είναι οι κάμερες τύπου «RGB» σε έναν πίνακα μικτών φίλτρων όταν ζητάτε `processing="radiance"` ή `"reflectance"` — η ακτινοβολία ανά Bayer δεν έχει νόημα για έναν ευρυζωνικό αισθητήρα, οπότε το backend παραλείπει αυτές τις κάμερες αντί να παράγει ανοησίες.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Τα διακριτικά αιτιολόγησης ακολουθούν το μοτίβο `<level>-not-applicable-to-rgb-cam` (μία καταχώριση ανά παραλειπόμενο επίπεδο, η καθεμία με το `level`). Οι παραλείψεις που αφορούν συγκεκριμένα την ανακλαστικότητα είναι `reflectance-skipped-no-fresh-dls` (δεν υπάρχει διαθέσιμη νέα μέτρηση κατερχόμενης ακτινοβολίας), `reflectance-skipped-bound-daq-unavailable (…)` (δεν ήταν δυνατή η πρόσβαση στο συνδεδεμένο DAQ) και `dls-uncalibrated-band-<nm>` — η ζώνη βρίσκεται ως επί το πλείστον εκτός του ραδιομετρικά βαθμονομημένου εύρους του αισθητήρα φωτός του DAQ (~374–974 nm), οπότε η απόλυτη διαίρεση ανακλαστικότητας βάσει DAQ απορρίπτεται και το καρέ υποβαθμίζεται σαφώς στην απόκριση του αισθητήρα. Μεταξύ των διαθέσιμων μοντέλων, μόνο το F988 το ενεργοποιεί· η υποστηριζόμενη διαδρομή αυτής της κάμερας είναι η ροή εργασίας με πάνελ ανακλαστικότητας.

Επίπεδα `processing`:

| Επίπεδο | Έξοδος |
| --- | --- |
| `"raw"` | Μονοκαναλικό Bayer (μονοχρωματικές κάμερες: η μονή ζώνη) απευθείας από τον αισθητήρα. |
| `"debayered"` *(προεπιλογή SDK)* | 3-καναλικό BGR μέσω διγραμμικής απομοσαϊκής επεξεργασίας (μονοχρωματικές κάμερες: 1 κανάλι σε κλίμακα του γκρι). |
| `"radiance"` | float32 W/m²/sr/nm μέσω της πλήρους ραδιομετρικής αλυσίδας. Μόνο πολυφασματικό — οι κάμερες «RGB» παραλείπονται. |
| `"reflectance"` | uint16 0..32768 (έτοιμο για Pix4D)· απαιτεί ζωντανό ζεύγος DAQ για απόλυτη αναφορά. Μόνο πολυφασματικό. |
| `"display"` | Πλήρης αλυσίδα που αντιστοιχεί στην προεπισκόπηση του GUI (CCM + WB + γάμμα σύμφωνα με το προφίλ της κάμερας). |
| `"all"` | **Ένα αρχείο ανά εφαρμόσιμο επίπεδο** για κάθε κάμερα (που αντιστοιχεί στην προεπιλογή «Capture All» / «CLI» του GUI). Το `CaptureResult` που επιστρέφεται περιέχει στη συνέχεια ένα λεξικό καρέ ανά `(cam, level)`, με το επίπεδο σε κάθε λεξικό· τα μη εφαρμόσιμα επίπεδα εμφανίζονται στο `.skipped`. Η μέτρηση DAQ που χρησιμοποιείται για οποιοδήποτε καρέ ανακλαστικότητας αποθηκεύεται ως παράρτημα `.daq`. |

> **Σημείωση — η προεπιλογή διαφέρει από το «CLI».** Το `ArraySession.capture()` έχει ως προεπιλογή το `processing="debayered"`· η εντολή `chloros-cli lattice array-capture` έχει ως προεπιλογή το `processing="all"`. Περάστε ρητά το `processing="all"` από το SDK για να αντικατοπτρίσετε την αποθήκευση πολλαπλών επιπέδων του CLI /GUI.

### Λειτουργίες καταγραφής &amp; συσκευές εγγραφής

Η επιφάνεια του πίνακα αντανακλά τον πίνακα καταγραφής του GUI: Λειτουργίες κλείστρου «Μονή» / «Συνεχής» / «Διάστημα» / «Ταχύτερη», καθώς και δύο συσκευές εγγραφής (ζωντανό σύνθετο βίντεο και ακατέργαστη ριπή → εκ νέου επεξεργασία εκτός σύνδεσης).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Combined-index video/GIF recorder (needs the combined live view streaming)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- Το **`capture_repeated`**είναι η λειτουργία «Συνεχής» του «SDK» /Διαστήματος του . Επειδή δεν υπάρχει `Ctrl+C` για να το διακόψετε από ένα σενάριο,**πρέπει** να περάσετε το `count` και/ή το `duration_s` (σταματά όταν επιτευχθεί οποιοδήποτε από τα δύο). Το `interval_s` μετράται από την αρχή κάθε περάσματος (όπως και στο GUI). Τα υπόλοιπα kwargs μεταβιβάζονται απευθείας στο `capture()`.
- **Το**`record`** είναι *επιπέδου παρακολούθησης*: καταγράφει το ζωντανό σύνθετο δείκτη όπως εμφανίζεται, οπότε η συνδυασμένη ροή πρέπει να είναι ανοιχτή για να καταγράφονται τα καρέ. Μία συσκευή καταγραφής σύνθετων δεδομένων ανά πίνακα (εμφανίζει σφάλμα αν υπάρχει ήδη μία σε λειτουργία).
- **`burst` → `build_video`** είναι *επιπέδου ανάλυσης*: το `burst` εγγράφει ακατέργαστα καρέ + ένα manifest ανά καρέ + ένα `.daq` ανά ξεχωριστή ανάγνωση DLS στο `<output>/bursts/<base>/` με τον πλήρη ρυθμό του βρόχου λήψης (χωρίς αλυσίδα, χωρίς exiftool, χωρίς ζωντανή προβολή). Το `build_video` αντιστοιχίζει χρονικά κάθε καρέ με το πλησιέστερο `.daq` και εκτελεί εκ νέου την αλυσίδα ακτινοβολίας/ανακλαστικότητας/δείκτη του αγωγού εισαγωγής. Το `products` είναι μια λίστα με `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (προεπιλογή: ο συνδυασμένος δείκτης). Το `burst().stop()` εκκινεί επίσης αυτόματα μια δημιουργία συνδυασμένου δείκτη με βάση την καλύτερη δυνατή προσπάθεια, η οποία επιστρέφεται ως `build_job` στο αποτέλεσμα τερματισμού.

#### `RecorderHandle`

Επιστρέφεται από τα `ArraySession.record()` και `ArraySession.burst()`. Χρησιμοποιήστε το ως διαχειριστή περιβάλλοντος για αυτόματη διακοπή κατά την έξοδο από το πεδίο εφαρμογής ή χειριστείτε το χειροκίνητα.

| Μέλος | Περιγραφή |
| --- | --- |
| `job_id` | Αναγνωριστικό εργασίας backend (str). |
| `kind` | `"composite"` (από το `record`) ή `"raw"` (από το `burst`). |
| `start_stats` | Το λεξικό που επιστρέφεται από την κλήση `start`. |
| `result` | `None` κατά τη διάρκεια της εκτέλεσης· το τελικό λεξικό αποτελεσμάτων διακοπής μετά τη διακοπή. |
| `stats(timeout=10.0)` | Ζωντανά στατιστικά εργασίας (πλαίσια που έχουν γραφτεί, πραγματικά fps, χρόνος που έχει παρέλθει). |
| `stop(timeout=60.0)` | Διακοπή της συσκευής εγγραφής· επιστρέφει και αποθηκεύει προσωρινά το τελικό αποτέλεσμα. Ιδηπότατη (μια δεύτερη κλήση επιστρέφει το αποθηκευμένο αποτέλεσμα). |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

### Σύνδεση σε ήδη συνδεδεμένο πίνακα — `attach_array`

Εάν ο πίνακας είναι ήδη ενεργός (το GUI τον άνοιξε ή μια προηγούμενη συνεδρία του SDK κάλεσε την `connect_array`), χρησιμοποιήστε το `attach_array` για να λάβετε έναν δείκτη πρόσβασης σε αυτόν αντί να επανασυνδεθείτε. Το `connect_array` εμφανίζει πάντα το σφάλμα «Η κάμερα βρίσκεται<sn> ήδη στο πλέγμα<id>» σε αυτή την περίπτωση, επειδή η αποστολή POST του `/array/connect` για ένα μέλος της ομάδας δεν είναι ιδεμπότενο· το `attach_array` διαβάζει το `/api/camera/array/list` και πραγματοποιεί αντιστοίχιση είτε με βάση το array_id είτε με βάση τους σειριακούς αριθμούς.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Πρότυπο: SDK Τα σενάρια που λειτουργούν παράλληλα με το γραφικό περιβάλλον εργασίας (GUI) θα πρέπει να δοκιμάζουν πρώτα το `attach_array` και να καταφεύγουν στο `connect_array` εάν δεν υπάρχει ακόμη κανένας πίνακας στο pool.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Σημαντικό — η έξοδος από το context-manager ΠΡΑΓΜΑΤΙΚΑ διακόπτει τη σύνδεση.**Το `ArraySession.disconnect()` εκτελεί πάντα POST στο `/array/disconnect`· δεν υπάρχει φρουρός «attached-not-owned» όπως υπάρχει για τα `CameraSession` / `DAQSensorSession`. Εάνχρησιμοποιείτε το GUI σε συν-ενοικίαση και δεν θέλετε να διαλύσετε τον πίνακα κατά την έξοδο από το πεδίο εφαρμογής,**μην χρησιμοποιείτε το μπλοκ `with`** — διατηρήστε το χειριστή σε μια κανονική μεταβλητή και παραλείψτε το ρητό `disconnect()`:
>
> ```python
> arr = chloros_sdk.attach_array(serials)
> arr.capture("output/", processing="reflectance")
> # … script ends; array stays up for the GUI
> ```

### Βοηθητικό εργαλείο ανάλυσης δικτύου

Χρήσιμο πριν από το άνοιγμα του πίνακα — προβλέπει αν οι προτεινόμενες ρυθμίσεις σας θα ταιριάζουν:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    print(f"Shrink to {r['out_width']}x{r['out_height']} binning={r['binning']}")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

Το `status` είναι ένα από τα `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (διαφορετικά είναι το `error`). Το `auto_capped_fps` σημαίνει ότι η ζητούμενη ανάλυση ταιριάζει στο δακτύλιο RX μόνο με περιορισμένο ρυθμό ενεργοποίησης — διατηρήστε την ανάλυση και περάστε το `target_fps=result["recommended"]["recommended_target_fps"]` στο `connect_array` (βλ. [Παράδειγμα 6](#6-capability-probe-before-connecting-a-4-cam-array)).

**Πώς να διαβάσετε την προβολή** (ίδιο μοντέλο με τον πίνακα «Ρυθμίσεις συστοιχίας» του GUI):

- **Η ροή (`frame_bytes_total`) αθροίζεται ανά κάμερα στην πραγματική μορφή εικονοστοιχείων κάθε κάμερας.**Οι κάμερες Mono**M3M**μεταδίδουν Mono12 (2 B/px) ανεξάρτητα από την τιμή του `pixel_format` που ορίζετε, οπότε ένα καρέ πλήρους ανάλυσης από 4 κάμερες έχει μέγεθος**~25 MB** με τρεις μονοχρωματικές κάμερες, και όχι τα ~12,6 MB που υποθέτει η περίπτωση όπου όλες είναι 8-bit . Το backend προσδιορίζει τη μορφή κάθε κάμερας με βάση το μοντέλο της.
- **Η πρόσβαση (`burst_fits_nic_ring`) λαμβάνει υπόψη την εκκένωση**, όχι τη σύγκριση ολόκληρης ριπής έναντι δακτυλίου: η προσομοίωση εκπομπής ταιριάζει όταν ο κεντρικός υπολογιστής εκκενώνει τον δακτύλιο RX γρηγορότερα από ό,τι τον γεμίζουν οι κάμερες. Ένας κεντρικός υπολογιστής 10G + κάμερες 1 GbE**δέχεται** πλήρη ανάλυση ακόμη και όταν το burst υπερβαίνει το ring· ένας κεντρικός υπολογιστής 1 GbE μπλοκάρει (`needs_force_slip` / `auto_shrunk`).
- **Το `achievable_fps_max` είναι ένα συντηρητικό ανώτατο όριο σειριακής ανάκτησης** — το `max(readout+emit, N×emit)` με την εκπομπή ανά κάμερα περιορισμένη στο σύνδεσμο κάμερας 1 GbE, ανεξάρτητο από την έκθεση. Π.χ. ~2,8 fps για συστοιχία 4 καμερών πλήρους ανάλυσης 12 bit (αντιστοιχεί στα μετρημένα ~2,7–3,0 του χρόνου εκτέλεσης). Πλήρες μοντέλο: [Αναφορά CLI → Μοντέλο fps &amp; ριπής συστοιχίας](cli-reference.md#array-fps--burst-model).
- **Η υπερ-εγγραφή (`oversubscribed: true`) σημαίνει ότι το κατώτατο όριο N × ανά κάμερα υπερβαίνει το ανώτατο όριο ασφαλούς σύγκρουσης** — τα πεδία fps (`achievable_fps_max` / `fps_bright` / `fps_dark`) εμφανίζουν τιμή 0, και η αυτόματη συρρίκνωση/ομαδοποίηση δεν μπορεί να το διορθώσει (μειώνουν τα byte ανά πλαίσιο, όχι τον ρυθμό των byte ανά δευτερόλεπτο). Λύσεις είναι η μείωση του αριθμού των καμερών, η χρήση jumbo frames ή μια ταχύτερη κάρτα δικτύου (NIC). Το `max_cams_collision_safe` αναφέρει το ανώτατο όριο (6 κάμερες πλήρους ανάλυσης σε 1 GbE με MTU 1500, 9 με jumbo). Η απόκριση περιλαμβάνει επίσης τα `aggregate_demand_bps`, `collision_safe_ceiling_bps` και `per_cam_floor_bps` (8 MB/s). Δείτε [Υπερ-εγγραφή](#over-subscription-the-per-cam-floor).

### Ανακάλυψη &amp; Καταχώριση

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

---

## Smart-AE / Smart-Capture

Οι συστοιχίες LATTICE εκτελούν συνεχή ρύθμιση έκθεσης (AE) στο παρασκήνιο μόλις συνδεθούν, αλλά μια σκηνή που έχει μόλις εστιάσει χρειάζεται λίγο χρόνο για να σταθεροποιηθεί. Το **Smart-capture** είναι η έτοιμη λύση: ελέγχει την έκθεση κάθε κάμερας, περιμένει έως ότου το σύστημα σταθεροποιηθεί σε ένα παράθυρο και, στη συνέχεια, ενεργοποιεί τη λήψη. Είναι ισοδύναμο με το GUI: το κουμπί «smart» capture της εφαρμογής για επιτραπέζιους υπολογιστές καλεί το ίδιο τελικό σημείο του backend.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Όταν χρησιμοποιείτε μέσω `ChlorosProject` (επόμενη ενότητα) έχετε στη διάθεσή σας περισσότερες ρυθμίσεις:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

Η πολιτική «smart-AE» είναι συντηρητική από προεπιλογή. Σφίξτε το `exposure_tolerance_pct` για απαιτητικές ραδιομετρικές εργασίες· χαλαρώστε το για σκηνές που αλλάζουν γρήγορα, όπου απλώς θέλετε ένα αποτέλεσμα «αρκετά κοντά».

---

## Συνεδρίες αισθητήρων DAQ

Μόνιμη ομάδα backend για φασματικούς αισθητήρες (DAQ-U μέσω USB, DAQ-M μέσω BLE, DAQ-E μέσω Ethernet). Αντικατοπτρίζει την επιφάνεια της κάμερας: έξυπνη ανίχνευση, επαναχρησιμοποίηση ομάδας, ιδεμπότενη σύνδεση.

### Έξυπνη ανίχνευση (Zero-Config)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Προτεραιότητα: Ethernet → BLE → USB. Δώστε οποιαδήποτε ρητή υπόδειξη για να καθορίσετε τη μέθοδο μεταφοράς.

### Καθορισμένη μεταφορά

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

### Μέθοδοι `DAQSensorSession`

| Μέθοδος | Περιγραφή |
| --- | --- |
| `status(timeout=10.0)` | Σύνοψη καταχώρησης ομάδας (κατάσταση ροής/εγγραφής, εύρος μήκους κύματος, sha βαθμονόμησης, χρόνος ολοκλήρωσης, frame_avg, κατάσταση AE). |
| `latest(n=1, timeout=10.0)` | Επιστροφή έως και των N πιο πρόσφατων πλαισίων φάσματος. |
| `stream_start()` / `stream_stop()` | Συνέχιση / παύση ροής (ο χειριστής παραμένει ανοιχτός). |
| `record_start(output_dir=None, device_name=None)` | Έναρξη εγγραφής αρχείου .daq. Επιστρέφει τη διαδρομή του αρχείου. Απορρίπτεται για DAQ-U/M χωρίς δέσμη βαθμονόμησης AWS (το DAQ-E εξαιρείται). |
| `record_stop()` | Διακοπή εγγραφής. Επιστρέφει το `{path, rows}`. |
| `disconnect()` | Απελευθέρωση από την ομάδα. Δεν εκτελείται για χειριστές που είναι συνδεδεμένοι αλλά δεν ανήκουν στον χρήστη. |

> **Τα προφίλ διόρθωσης κορυφής (`cap_id`) δεν αποτελούν ρυθμιστικό τουSDK.** Οι `connect_daq_sensor()` / `DAQSensorSession` δεν εκθέτουν καμία παράμετρο `cap_id` ή μέθοδο `set_cap`. Επιλέξτε ένα προφίλ διόρθωσης ορίου στόλου μέσω των CLI (`chloros-cli daq pool-connect --cap-id …` / `chloros-cli daq pool-set-cap …`) ή τις διαδρομές «HTTP» (`/api/daq`) του backend (`/api/daq/connect` και `/api/daq/<id>/cap-id` δέχονται το `cap_id`).

### Ανακάλυψη — εύρεση μιας διεύθυνσης για σύνδεση

Το `discover_daq_sensors()` σαρώνει τα USB / BLE / ETH για αισθητήρες που *θα μπορούσατε* να ανοίξετε. Αποτελεί το αντίστοιχο του `discover_lattice_cameras()` για το DAQ και τον μοναδικό τρόπο για να λάβετε το **MAC BLE ενός DAQ-M** — ένα DAQ-E διαθέτει όνομα κεντρικού υπολογιστή και ένα DAQ-U θύρα COM, αλλά η διεύθυνση MAC δεν αναγράφεται ούτε στη συσκευή ούτε εμφανίζεται από το λειτουργικό σύστημα.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               None   {'manufacturer': 'Intel'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Πεδίο | Περιγραφή |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | Θύρα COM / BLE MAC / όνομα κεντρικού υπολογιστή — μεταβιβάζεται στα `connect_daq_sensor` ως `port=` / `mac=` / `eth_host=`. |
| `display` | Ετικέτα αναγνώσιμη από τον άνθρωπο. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E`, ή `None` για μια θύρα που η σάρωση δεν μπορεί να αναγνωρίσει (οι σειριακοί προσαρμογείς USB δεν μπορούν να διακριθούν χωρίς ανιχνευτή, οπότε τα άγνωστα εμφανίζονται αντί να κρύβονται). |
| `extra` | Λεπτομέρειες ανά μέσο μεταφοράς (Δηλωμένο όνομα BLE, κατασκευαστής USB, IP/FW/… του DAQ-E). Οι κενές τιμές παραλείπονται. |

| Παράμετρος | Προεπιλογή | Περιγραφή |
| --- | --- | --- |
| `transports` | και τα τρία | Ακολουθία (ή συμβολοσειρά csv) που περιορίζει τη σάρωση. Αξίζει να την ορίσετε όταν γνωρίζετε τι θέλετε — το BLE είναι το πιο αργό τμήμα. |
| `scan_timeout` | 5 | Παράθυρο σάρωσης ανά μέσο μεταφοράς σε δευτερόλεπτα· το backend το περιορίζει στο εύρος 1–20. |
| `timeout` | 60,0 | Ανώτατο όριο «HTTP» για ολόκληρη την κλήση (όπως και αλλού στο «SDK»). |
| `auto_start_backend` | `True` | Δημιουργία τοπικού backend εάν δεν εκτελείται κανένα. Δεν δημιουργείται ποτέ για απομακρυσμένο `backend_url`. |

> **Οι αισθητήρες που είναι ήδη ανοιχτοί στο pool δεν εμφανίζονται.** Μια συνδεδεμένη περιφερειακή συσκευή BLE σταματά να διαφημίζεται και μια ανοιχτή θύρα COM δεν μπορεί να ανιχνευθεί, οπότε η αναζήτηση παραθέτει ό,τι είναι *διαθέσιμο για σύνδεση*. Είναι αναμενόμενο να εμφανιστεί ένα κενό αποτέλεσμα αμέσως μετά τη σύνδεση μιας συσκευής — χρησιμοποιήστε το `list_daq_sensors()` για ό,τι έχετε ήδη στη διάθεσή σας. Τα πρωτόκολλα μεταφοράς των οποίων η σάρωση δεν μπορεί να εκτελεστεί (δεν έχει εγκατασταθεί το bleak / zeroconf) παραλείπονται αντί να εμφανίζονται ως σφάλματα, οπότε ένας υπολογιστής χωρίς Bluetooth εξακολουθεί να λαμβάνει τις απαντήσεις για τα USB και ETH.

### Κατάλογος

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Συνύπαρξη με GUI / CLI

Εάν το GUI έχει ήδη ανοιχτό έναν αισθητήρα, η κλήση του `connect_daq_sensor(port="COM3")` από το Python επιστρέφει ένα χειριστή με την ένδειξη `already_connected=True`. Το `disconnect()` της συνεδρίας είναι τότε δεν εκτελεί καμία ενέργεια, ώστε το σενάριό σας SDK να μην αποσυνδέει τον αισθητήρα από το GUI κατά την έξοδο από το scope.

### Κλάσεις άμεσης πρόσβασης σε υλικό (Χωρίς Backend)

Το `daq_sdk` επανεξάγεται από το `chloros_sdk`, ώστε να μπορείτε επίσης να ελέγχετε τους αισθητήρες από άκρο σε άκρο κατά τη διάρκεια της διαδικασίας χωρίς το backend:

> **Διαθεσιμότητα:**Το `daq_sdk` διατίθεται με την εγκατάσταση του Chloros για επιτραπέζιους υπολογιστές,**όχι** με το πακέτο PyPI — Το `pip install chloros-sdk` σας παρέχει το `lattice_sdk`, αλλά αφήνει το `chloros_sdk.DAQ_AVAILABLE == False`. Ελέγξτε αυτήν την ένδειξη πριν χρησιμοποιήσετε αυτές τις κλάσεις· σε έναν κεντρικό υπολογιστή που διαθέτει μόνο το pip, ελέγξτε τον αισθητήρα μέσω του [`connect_daq_sensor()`](#daq-sensor-sessions), ο οποίος δεν απαιτεί τοπικές βιβλιοθήκες μεταφοράς.

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

# Discovery
for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

# Direct DAQ-U
sensor = DAQUSensor(port="COM3")
sensor.connect()
sensor.start_streaming()
# ... use sensor.add_spectrum_callback(...) ...
sensor.stop()
```

Προτιμήστε τη διαδρομή «smart-connect» (`connect_daq_sensor`) όταν επιθυμείτε κοινή ιδιοκτησία με το GUI· χρησιμοποιήστε τις άμεσες κλάσεις για σενάρια χωρίς διεπαφή χρήστη που κατέχουν αποκλειστικά τον αισθητήρα.

---

## Αυτοματοποίηση έργου — `ChlorosProject`

Ένα αποθηκευμένο έργο «Chloros» είναι ένας φάκελος που περιέχει τα αρχεία `cameras.json` + `sensors.json` + `project.json`. Το `open_project` φορτώνει το manifest, και το `connect_all` συνδέει κάθε αποθηκευμένη συσκευή στο διαδίκτυο με τις αποθηκευμένες ρυθμίσεις της — στην ίδια κατάσταση υλικού που θα δημιουργούσε το GUI.

### Ελάχιστο παράδειγμα

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Read a DAQ
spectrum = proj.sensors["Sky"].read()

# Trigger every device simultaneously
proj.capture_all("./out")

proj.disconnect_all()
```

Ή ως διαχειριστής περιβάλλοντος:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### Μέθοδοι `ChlorosProject`

| Μέθοδος | Περιγραφή |
| --- | --- |
| `connect_all(cameras=True, arrays=True, sensors=True, verbose=False, align=None)` | Εντοπίζει και συνδέει κάθε αποθηκευμένη συσκευή. Επιστρέφει μια αναφορά σύνδεσης ανά κλάση. Χρησιμοποιεί ένα τρέχον backend όταν υπάρχει κάποιο σε κατάσταση ακρόασης στο `127.0.0.1:5000`· διαφορετικά, επιστρέφει σιωπηλά στον άμεσο (χωρίς backend) έλεγχο συσκευών `lattice_sdk` — δεν δημιουργεί ποτέ backend. |
| `disconnect_all()` | Τερματισμός όλων των συνδέσεων. |
| `capture_all(output_dir=".")` | Ένα πλαίσιο από κάθε κάμερα + πίνακα + φάσμα από κάθε αισθητήρα. |
| `stream(camera, overlays=False, fps=10.0)` | Γεννήτρια που παράγει καρέ BGR `numpy` από μια κατονομαζόμενη κάμερα (ή συστοιχία). Το `overlays=False` είναι ένας άμεσος βρόχος λήψης `lattice_sdk` (οι συστοιχίες παράγουν λεξικά `{serial: frame}`). Το `overlays=True` δρομολογείται μέσω του `ChlorosLocal.camera_stream()` → τη ροή MJPEG του backend `/api/camera/<serial>/stream-annotated`, με το αποθηκευμένο μπλοκ της κάμερας `ui.overlay` να διαβιβάζεται ως παράμετροι ερωτήματος. Απαιτεί λειτουργία backend και **αυτόνομη κάμερα**: μια κάμερα σε άμεση λειτουργία προκαλεί το σφάλμα `RuntimeError` (το backend δεν μπορεί να λάβει μια κάμερα που ανήκει σε αυτή τη διαδικασία) και ένας πίνακας προκαλεί το σφάλμα `NotImplementedError` (επικαλύψεις σύνθετων εικόνων ανά κάμερα — ροή ενός μέλους με βάση το όνομα). Ισοδύναμο μιας εκτέλεσης: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Εκτέλεση ευθυγράμμισης σε κάθε πίνακα που είναι συνδεδεμένος αυτή τη στιγμή. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Εκτέλεση της διαδικασίας βαθμονόμησης / ευρετηρίου στις εικόνες του έργου (περιλαμβάνει το `ChlorosLocal.process`· αυτά τα τέσσερα είναι τα **μόνα** αποδεκτά kwargs — τα `indices=` κ.λπ. προκαλούν σφάλμα `TypeError`· ορίστε δείκτες μέσω του `ChlorosLocal.configure()`). Δημιουργεί με καθυστέρηση ένα `ChlorosLocal()`, το οποίο εκκινεί αυτόματα ένα backend. |

Χαρακτηριστικά:
- `proj.cameras` — `Dict[str, CameraHandle]` με κλειδί το όνομα ΚΑΙ τον σειριακό αριθμό.
- `proj.arrays` — `Dict[str, ArrayHandle]` με κλειδί το όνομα ΚΑΙ το array_id.
- `proj.sensors` — `Dict[str, SensorHandle]` με κλειδί το όνομα ΚΑΙ το slot_id.
- `proj.config` — `project.json["config"]` λεξικό.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Επίπεδα επεξεργασίας.** Τα `capture()`, `grab()` και `frame_stream()` δέχονται όλα το ίδιο `processing`
και η αλυσίδα είναι αθροιστική — κάθε επίπεδο εκτελεί όλα τα επίπεδα που βρίσκονται πάνω από αυτό:

| Επίπεδο | Έξοδος | Σημειώσεις |
| --- | --- | --- |
| `raw` | Bayer 1 καναλιού, εγγενής αισθητήρας-εγγενής | Χωρίς απομοσαϊκό. Οι επικαλύψεις δεν είναι διαθέσιμες σε αυτό το επίπεδο. |
| `debayered` | BGR 3 καναλιών (**προεπιλογή**) | Διγραμμικό απομοσαϊκό. Το μόνο επίπεδο που λειτουργεί χωρίς τη λειτουργία backend. |
| `radiance` | float32, W/m²/sr/nm | Πλήρης ραδιομετρική αλυσίδα: απομοσαϊκό + 3×3 αποσύνθεση (πολυφασματική) + DSNU + flat-field + κλίμακα NIST, με την έκθεση × την ενίσχυση να έχουν αφαιρεθεί, ώστε οι τιμές να είναι απόλυτες. |
| `reflectance` | uint16, 32768 = 1.0 | Ακτινοβολία διαιρούμενη με την προς τα κάτω ακτινοβολία (ρ = π·L/E). Απαιτείται ένδειξη DLS/DAQ — δείτε τη σημείωση παρακάτω. |
| `display` | 8-bit sRGB-ish | Απόδοση ισοδύναμη με GUI: CCM + ισορροπία λευκού + γάμμα μέσω του ενεργού προφίλ χρώματος της κάμερας. |

Οτιδήποτε άλλο εκτός από το `debayered` απαιτεί λειτουργία backend· μια κάμερα σε άμεση λειτουργία ενεργοποιεί
το `NotImplementedError`. Το `reflectance` απαιτεί μια χρησιμοποιήσιμη μέτρηση κατερχόμενης ακτινοβολίας — το τελικό σημείο του καρέ μεταφέρει
αυτόματα το συγκεντρωμένο DAQ στην υποδοχή DLS της κάμερας, αλλά χωρίς δεσμευμένο DAQ η αλυσίδα απορρίπτει την
έξοδο ανακλαστικότητας και σημειώνει ειλικρινά την υποβάθμιση στα επιστρεφόμενα μεταδεδομένα αντί να
επιστρέφει σιωπηλά ένα κατώτερο προϊόν.

> **Κλίμακα DN ανακλαστικότητας — μην την κωδικοποιείτε σταθερά.** Η ανακλαστικότητα LATTICE χρησιμοποιεί το `32768` = ρ 1,0 και καταγράφει
> XMP `Chloros:PixelScale=32768`· η ανακλαστικότητα Survey3 χρησιμοποιεί το `65535` = ρ 1,0 και δεν φέρει
> ετικέτες `Chloros:*`. Διαβάστε την ετικέτα και διαιρέστε με αυτήν. Ορίζεται στον χώρο τιμών uint16, οπότε παραμένει
> `32768` για κάθε μορφή που αλλάζει κλίμακα (TIFF 16-bit, 8-bit PNG /JPG, 32-bit percent) — ομαλοποιήστε
> πρώτα τον αποθηκευμένο τύπο δεδομένων (dtype) πίσω σε uint16 (×257 από 8-bit, ×65535 από float). Η μόνη εξαίρεση:
> μια λήψη με πηγή 8-bit που έχει γραφτεί ως 8-bit TIFF «περικοπείται» (*clipped*), δεν αναπροσαρμόζεται, οπότε καμία κλίμακα δεν την περιγράφει
> — το Chloros παραλείπει εντελώς το `PixelScale` και την τούπλα της MicaSense σε αυτή την περίπτωση. Αντιμετωπίστε μια ελλείπουσα
> ετικέτα σε ένα αρχείο ανακλαστικότητας LATTICE ως «καμία έγκυρη κλίμακα», όχι ως προεπιλογή.

> **Τα στοιχεία EXIF μεταφέρονται κατά την εξαγωγή.** Το `process()` αντιγράφει το μπλοκ GPS της αρχικής λήψης
> **και το ExifIFD** σε κάθε προϊόν, οπότε οι εξαγωγές περιλαμβάνουν τα `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` και `CameraSerialNumber`, καθώς και τη
> γεωαναφορά. Το `FocalLength` είναι αυτό από το οποίο το Pix4D υπολογίζει την απόσταση δειγματοληψίας εδάφους — χωρίς αυτό
> η ανακατασκευή καταλήγει σε μια εντελώς λανθασμένη κλίμακα (μια μετρημένη περίπτωση μετέτρεψε μια τοποθεσία 411 m
> σε μια 47,8 km). Το αντίγραφο δεν είναι σκόπιμα το `-all:all`: οι δομικές ετικέτες του IFD0 διαταράσσουν
> την έξοδο του LATTICE, και τα `ExifImageWidth`/`Height` εξαιρούνται επειδή περιγράφουν τη λήψη της πηγής
> και όχι το εξαγόμενο ράστερ.

Υπο-ετικέτες σταδίου λήψης (ισχύουν για τα ραδιομετρικά επίπεδα — `radiance`, `reflectance`, `display`):

| Σημαία | Προεπιλογή | Έννοια |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + πεδίο-πεδίο + αποσύνθεση 3x3 + ραδιομετρική κλίμακα NIST. |
| `apply_white_balance` | `True` | WB LUT. Υποστηρίζει DLS όταν ένα DAQ είναι συνδεδεμένο με την κάμερα. |
| `apply_index` | `False` | Αξιολόγηση δείκτη βλάστησης. |
| `index_expression` | `None` | Παράκαμψη τύπου. Αν δεν είναι κενό → ενεργοποίησηενεργοποίηση δείκτη. |
| `annotated` | `False` | Επικάλυψη διακοσμητικών στοιχείων GUI (zebra/grid/peaking). Δεν διατίθεται για το `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **Ο τύπος επιστροφής είναι `CapturePathMap`, όχι `Dict[str, str]`.**
> Το `chloros_sdk.CapturePathMap` είναι το `Dict[str, Union[str, List[str]]]`: ενός επιπέδου
> Το `processing` δίνει σε κάθε σειριακό αριθμό μία διαδρομή, ενώ ένα πολλαπλών επιπέδων (`"all"`, ή μια
> ρητή λίστα `levels`) του δίνει τη **ταξινομημένη λίστα** κάθε προϊόντος που έχει αποθηκευτεί για την εν λόγω
> κάμερα. Ένα ζωντανό συνδυασμένο σύνθετο, αν υπάρχει ροή, φτάνει κάτω από το επιπλέον
> κλειδί `"combined"` και όχι κάτω από έναν σειριακό κωδικό. Ο κώδικας που υποθέτει το `str` παρουσιάζει σφάλμα στη
> μορφή της λίστας χωρίς να υπάρχει αντίρρηση από τον έλεγχο τύπων — η σημείωση ανέφερε το `Dict[str, str]`
> για κάποιο διάστημα μετά την κυκλοφορία της μορφής λίστας, γι’ αυτό και υπάρχει το ψευδώνυμο. Ομαλοποιήστε
> όταν θέλετε τη «επίπεδη» μορφή:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

### Ευθυγράμμιση πινάκων

Το `ArrayHandle` εκθέτει την πλήρη επιφάνεια ευθυγράμμισης. Τα προφίλ ισχύουν μόνο για τη συνεδρία από προεπιλογή — καλέστε ρητά το `export_alignment()` για να τα διατηρήσετε.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Ευθυγράμμιση κατά τη σύνδεση

Το `connect_all(align=...)` μπορεί να ευθυγραμμίσει αυτόματα κάθε πίνακα κατά τη σύνδεση:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Εάν δεν καθοριστεί, χρησιμοποιείται ως εναλλακτική η συνάρτηση `project.json["config"]["auto_align_on_connect"]`.

### `SensorHandle`

```python
spectrum = proj.sensors["Sky"].read()
# (spectrum_list, is_saturated, integration_time, x, y, z) — matches the
# daq_sdk add_spectrum_callback signature.
```

---

## Άμεση χρήση υλικού (χωρίς backend)

Όταν θέλετε μηδενική εξάρτηση από το backend (CI, ρομπότ χωρίς διεπαφή χρήστη, ενσωματωμένα συστήματα), εισάγετε απευθείας τα `lattice_sdk` και `daq_sdk` — και τα δύο επανεξάγονται από το `chloros_sdk`. Προσοχή στα `CAMERA_AVAILABLE` / `DAQ_AVAILABLE`: το `lattice_sdk` περιλαμβάνεται στο πακέτο PyPI (αλλά απαιτεί την παρουσία του περιβάλλοντος εκτέλεσης Arena SDK), ενώ το `daq_sdk` διατίθεται μόνο με την εγκατάσταση για επιτραπέζιους υπολογιστές.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Προεπιλογές και ενεργοποίηση

Τρεις από τις τέσσερις προεπιλογές είναι **ελεύθερης λειτουργίας**: η κάμερα εκθέτει συνεχώς και το
`capture()` επιστρέφει το επόμενο καρέ. Το `triggered` αποτελεί την εξαίρεση — θέτει την
κάμερα σε κατάσταση ετοιμότητας για ένα σήμα άκρου υλικού στη Γραμμή 2, οπότε δεν καταγράφει τίποτα μέχρι να εμφανιστεί ένα.

| Προεπιλογή | Σκανδάλη | Χρησιμοποιήστε την όταν |
| --- | --- | --- |
| `default` | ελεύθερη λειτουργία | γενική χρήση |
| `high_speed` | ελεύθερη λειτουργία | 8-bit, όριο 60 fps, σύντομη έκθεση |
| `high_quality` | ελεύθερη λειτουργία | 12-bit, χωρίς όριο fps — η συνήθης επιλογή για φωτογραφίες |
| `triggered` | **ενεργοποιημένη, Γραμμή 2** | η κάμερα είναι συνδεδεμένη με καλώδιο συγχρονισμού M8 και ενεργοποιείται από κάποιον άλλο παράγοντα |

Αν επιλέξετε το `triggered` (ή ρυθμίσετε μόνοι σας το `trigger_mode="On"`) χωρίς
να ενεργοποιεί τη Γραμμή 2, κάθε `capture()` θα λήξει — σωστά, αφού ζητήσατε
από την κάμερα να περιμένει. Το SDK εξηγεί αυτό όταν συμβαίνει· δείτε
[SC_ERR_TIMEOUT κατά τη λήψη](#direct-hardware-backend-free).

> **Σημείωση — Τα μηνύματα «GVSP probe» / `SC_ERR_TIMEOUT -1011` κατά τη σύνδεση δεν αποτελούν σφάλματα.**&gt; Κατά τη σύνδεση, το SDK προσπαθεί να διαπραγματευτεί**jumbo frames** (πακέτα GVSP 9000-byte) για υψηλότερη απόδοση. Σε μια απευθείας σύνδεση NIC από σημείο σε σημείο (π.χ. μια τοπική διεύθυνση `169.254.x.x`), το δίκτυο συνήθως δεν μπορεί να μεταφέρει jumbo frames, οπότε αυτή η δοκιμή λήγει και καταγράφει γραμμές όπως:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Αυτή είναι η **προβλεπόμενη εναλλακτική λύση**: το SDK επανέρχεται αυτόματα στα τυπικά πακέτα των 1500 byte και η κάμερα συνεχίζει να συνδέεται κανονικά (οι γραμμές `[chunk-enable …]` που ακολουθούν αποτελούν μέρος της κανονικής ακολουθίας σύνδεσης ). Η καταγραφή εξακολουθεί να λειτουργεί.
>
> Μπορείτε να παραλείψετε αυτόν τον έλεγχο, αλλά **δεν είναι απλώς ένας τρόπος για να μην εμφανίζονται μηνύματα καταγραφής — απενεργοποιεί τα jumbo frames.** Η κάμερα απαντά σε ping τύπου «Don&#x27;t-Fragment» μόνο έως 1500 byte, ανεξάρτητα από το πόσο καλό είναι το δίκτυό σας, οπότε η δοκιμή ping από μόνη της δεν μπορεί ποτέ να εντοπίσει τα jumbo frames· αυτή η δοκιμή είναι το μόνο που μπορεί. Απενεργοποιήστε τον και η κάμερα θα στέλνει για πάντα τυπικά πακέτα των 1500 byte, σε οποιοδήποτε δίκτυο:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Αξίζει να το κάνετε μόνο σε δίκτυο που *γνωρίζετε* ότι δεν υποστηρίζει jumbo, όπου εξοικονομεί περίπου ένα δευτερόλεπτο χρόνου σύνδεσης ανά κάμερα. Δεδομένου ότι πρόκειται για πραγματική ανταλλαγή και όχι για κάτι επιφανειακό, η εντολή «SDK» το επισημαίνει πλέον όταν τη χρησιμοποιείτε:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Μην την αλλάξετε, εκτός αν έχετε λόγο.** Αν παραμείνει ενεργοποιημένη, κάθε σύνδεσημετρά το δίκτυο που διαθέτετε πραγματικά: συνδέστε το σε ένα διακόπτη με δυνατότητα jumbo και η επόμενη σύνδεση θα αναγνωρίσει αυτόματα το jumbo, χωρίς να χρειαστεί καμία ρύθμιση και χωρίς επανεκκίνηση.
>
> Αν *θέλετε* τη διακίνηση δεδομένων jumbo, ενεργοποιήστε το jumbo από άκρο σε άκρο (MTU κάρτας δικτύου 9000 + διακόπτης που τα μεταδίδει), ή καθορίστε το με το `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` όταν γνωρίζετε ότι η σύνδεση το υποστηρίζει — αν και προτιμήστε το `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` ανά εντολή αντί για μόνιμη ρύθμιση, καθώς ένα σταθερό μέγεθος παρακάμπτει τον έλεγχο και σταματά να προσαρμόζεται στο δίκτυο που βρίσκεται μπροστά του. **Κάθε** συσκευή στη διαδρομή πρέπει να μεταφέρει πακέτα jumbo — συμπεριλαμβανομένων τυχόν διαχωριστών ή εγχυτήρων PoE, κάτι που αποτελεί τον συνήθη λόγο για τον οποίο μια κατά τα άλλα συμβατή με jumbo εγκατάσταση δεν μπορεί να τα μεταφέρει.

> **Το `SC_ERR_TIMEOUT -1011` κατά τη διάρκεια των `capture()` / `grab*()` αποτελεί διαφορετικό πρόβλημα — αυτό είναι πραγματικό σφάλμα.**&gt; Η παραπάνω σημείωση αφορά μόνο το `-1011` που καταγράφηκε από τον**αισθητήρα χρόνου σύνδεσης**. Το ίδιο σφάλμα που προκύπτει από μια**συλλογή δεδομένων** σημαίνει ότι η κάμερα συνδέθηκε κανονικά αλλά δεν στέλνει εικόνες:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Το αποκαλυπτικό στοιχείο είναι μια κάμερα της οποίας το κανάλι *ελέγχου* λειτουργεί κανονικά — η ανίχνευση λειτουργεί, οι ρυθμίσεις και οι εγγραφές `[chunk-enable …]` ολοκληρώνονται με επιτυχία — ενώ *κάθε* καρέ καταλήγει σε χρονικό όριο.
>
> **Η συνήθης αιτία είναι ότι η κάμερα έχει ενεργοποιηθεί για ενεργοποίηση μέσω υλικού.** Με τα `trigger_mode="On"` και `trigger_source="Line2"`, η κάμερα δεν εκπέμπει απολύτως τίποτα μέχρι να φτάσει ένα ηλεκτρικό σήμα στο καλώδιο συγχρονισμού M8. Εάν δεν υπάρχει καλώδιο που να τροφοδοτεί αυτή τη γραμμή, κάθε λήψη περιμένει επ’ αόριστον. Η κάμερα δεν είναι χαλασμένη και το δίκτυο λειτουργεί κανονικά — κάνει ακριβώς αυτό που της έχει δοθεί εντολή.
>
> Τα `CameraSettings()` και τα `default` / `high_speed` / `high_quality` εκτελούνται ελεύθερα-run, και μια λήψη που λήγει λόγω χρονικού ορίου ενώ είναι ενεργοποιημένη παρέχει εξήγηση αντί να εμφανίζει απλώς ένα `-1011`. Το `PRESETS["triggered"]` ενεργοποιεί τη Γραμμή 2, σύμφωνα με τον σχεδιασμό.
>
> Για να αναγκάσετε οποιαδήποτε κάμερα να λειτουργήσει σε ελεύθερη λειτουργία:
>
> ```python
> settings = PRESETS["high_quality"]
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Εάν εξακολουθεί να εμφανίζει χρονικό όριο με το `trigger_mode="Off"`, η κάμερα πραγματικά δεν μεταδίδει δεδομένα — στείλτε μας το αρχείο καταγραφής και το `ip link show`.

#### Προφίλ χρωμάτων (ζωντανή προεπισκόπηση RGB) — `set_color_profile`

Το `LatticeCamera.set_color_profile(profile, custom_cct_k=None)` επιλέγει το προφίλ χρωμάτων της οθόνης για τη **ζωντανή προεπισκόπηση** στις κάμερες RGB (οι κάμερες multispec αγνοούν τη ρύθμιση):

| Προφίλ | Σημασία |
| --- | --- |
| `raw` | Παράκαμψη της ραδιομετρικής αλυσίδας εντελώς. |
| `linear` | DSNU + flat + WB, χωρίς CCM, χωρίς γάμμα. |
| `natural` | Γραμμική + μετρημένο CCM + γάμμα sRGB, μόνο με το απλό φινίρισμα (εξομάλυνση χρωματικότητας + αποκορεσμός φωτεινών σημείων) — η ρεαλιστική προεπιλογή. |
| `enhanced` | `natural` συν την πλήρη επεξεργασία «hub-parity» (αφαίρεση περιθωριακών χρωμάτων, ζωντάνια, τοπική αντίθεση CLAHE). Πλουσιότερη εμφάνιση με περίπου **διπλάσιο κόστος επεξεργασίας ανά καρέ**, οπότε χαμηλότερο ρυθμό καρέ LIVE. |
| `custom_temp` | `natural` αλλά η ισορροπία λευκού (WB) σταθεροποιημένη στα `custom_cct_k` Κέλβιν (το DLS αγνοείται· περιορίζεται στα 2000–10000 K από την πλευρά του backend). |

Το προφίλ είναι ένας **ρυθμιστής ταχύτητας/εμφάνισης μόνο για ζωντανή προεπισκόπηση**: οι αποθηκευμένες λήψεις λαμβάνουν πάντα την πλήρη, πλούσια επεξεργασία ανεξάρτητα από το επιλεγμένο προφίλ, οπότε η επιλογή του `natural` για την εξοικονόμηση χρόνου καρέ δεν μειώνει την ποιότητα του τελικού αρχείου που αποθηκεύεται στο δίσκο. Ένα άγνωστο προφίλ ενεργοποιεί το `ValueError`· όταν ένα backend chloros είναι προσβάσιμο, η αλλαγή αποστέλλεται (POST) και σε αυτό, ώστε το επόμενο καρέ προεπισκόπησης να την αντανακλά (οι χρήστες του direct-SDK χωρίς backend εξακολουθούν να λαμβάνουν την αλλαγή στις ρυθμίσεις).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Μονοχρωματικές (M3M) κάμερες και `Calibration`

Μια μονοχρωματική κάμερα **M3M** (`M3M-<lens>-F<wavelength>`) είναι μονοζωνική: ένα επίπεδο κλίμακας του γκρι, χωρίς μωσαϊκό Bayer, χωρίς μήτρα φασματικής διασταύρωσης 3×3. Το `Calibration` την αναγνωρίζει και εκθέτει μια σημαία `is_mono`. Η ανακλαστικότητα εξακολουθεί να ισχύει ως ραδιομετρικός χάρτης ανά ζώνη (η αποσύνθεση είναι ο πίνακας ταυτότητας), αλλά οι μαθηματικοί υπολογισμοί πολλαπλών ζωνών σε μία μόνο κάμερα δίνουν αποτελέσματα αντί να επιστρέφουν ασυναρτησίες:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

Για να δημιουργήσετε έναν δείκτη βλάστησης από μονοφασικό υλικό, συνδυάστε αρκετές κάμερες M3M σε διαφορετικά μήκη κύματος σε μια ευθυγραμμισμένη στοίβα πολλαπλών ζωνών (βλ. [Ευθυγράμμιση Συστοιχιών](#array-alignment)) και υπολογίστε τον δείκτη σε όλη τη στοίβα αντί για μία κάμερα.

Άμεση λειτουργία DAQ:

```python
from chloros_sdk import (
    DAQUSensor, DAQMSensor, DAQESensor,
    SensorFleet, discover_all, DiscoveredSensor,
    apply_sensor_settings, SensorSettings,
)

for d in discover_all(timeout=3.0):
    print(d)

sensor = DAQUSensor(port="COM3")
sensor.connect()
apply_sensor_settings(sensor, settings={"integration_time_ms": 64, "frame_avg": 20})
sensor.start_streaming()
# ... sensor.add_spectrum_callback(your_callback) ...
sensor.stop()
```

> **`apply_sensor_settings` αποδεκτά κλειδιά**— ακριβώς `integration_time_ms`, `frame_avg`, `ae_enabled`, `sunshine_diffuser_installed` (DAQ-E· έχει καταργηθεί υπέρ του `cap_id`), `filter_model` (DAQ-M), και `cap_id` (όλοι οι τύποι DAQ· `None`/`""`/`"none"` = γυμνός αισθητήρας, χωρίς διόρθωση καλύμματος). Τα άγνωστα κλειδιά**αγνοούνται σιωπηρά** — π.χ. το `{"integration_time": 64}` δεν κάνει τίποτα (πρέπει να είναι `integration_time_ms`). Επιστρέφει το `{"applied": [...], "errors": {...}}` και δεν δημιουργεί ποτέ σφάλμα.

Το `chloros_sdk` επανεξάγει μόνο την επιφάνεια πυρήνα που χρησιμοποιήθηκε παραπάνω. Το πλήρες δημόσιο «API» του `daq_sdk` (22 ονόματα) προσθέτει τα ακόλουθα — εισάγετέ τα απευθείας από το `daq_sdk`:

```python
from daq_sdk import (
    DAQULogger, DAQMLogger, DAQELogger,     # rotating-file recorders (the ones the GUI uses)
    ConnectResult, FleetRecordResult,       # SensorFleet result types
    discover_all_detailed, build_sensor,    # detailed discovery + build-by-descriptor
    scan_eth_devices, DaqEControl,          # DAQ-E Ethernet scan + control channel
    scan_ble_devices, detect_ble_device, list_ble_devices,   # DAQ-M BLE discovery
    detect_port, list_serial_ports,         # DAQ-U serial-port discovery
    TcpSerial,                              # serial-over-TCP transport shim
)
```

---

## Εξαιρέσεις

Εντοπίστε την βασική κλάση για να χειριστείτε «οτιδήποτε πήγε στραβά Chloros»:

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

> Τα `ChlorosAuthenticationError` και `ChlorosConfigurationError` εξάγονται σε ανώτατο επίπεδο μαζί με τα υπόλοιπα· μπορούν επίσης να εισαχθούν από το `chloros_sdk.exceptions`, όπως φαίνεται.

Ιεραρχία:

```

ChlorosError
├── ChlorosBackendError           (backend failed to start / unreachable)
├── ChlorosConnectionError        (HTTP transport failure)
├── ChlorosLicenseError           (subscription / tier gate)
├── ChlorosAuthenticationError    (login required)
├── ChlorosConfigurationError     (bad configure() / open_project() inputs)
└── ChlorosProcessingError        (pipeline failed)

ChlorosConnectError                (raised by connect_camera / connect_array /
                                    connect_daq_sensor only — derives from
                                    plain Exception, NOT from ChlorosError,
                                    so `except ChlorosError` will not catch it)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## Παραδείγματα από άκρο σε άκρο

### 1. Επεξεργασία φακέλου με προσαρμοσμένη γραμμή προόδου

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Ζωντανός πίνακας LATTICE → Ανακλαστικότητα + Αναφορά DAQ

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Εκστρατεία καταγραφής βάσει έργου

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Ροή καρέ από πολλαπλές κάμερες → Αγωγός NumPy

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Σενάριο καταγραφής χωρίς διεπαφή χρήστη με άμεση σύνδεση στο υλικό (χωρίς backend)

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Έλεγχος δυνατοτήτων πριν τη σύνδεση συστοιχίας 4 καμερών

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["out_width"], height=r["out_height"],
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Ισοδύναμο «Recipe» καταγραφής (αμιγώς «Python»)

Η γλώσσα DSL «Recipe» του «CLI» έχει ένα άμεσο ισοδύναμο στο «Python»:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply
    # (CameraHandle has no direct apply method; use the underlying lattice_sdk
    #  helper or the backend's /api/camera/<sn>/apply-settings via requests)
    # For most cases just use cam.cam.set_exposure(...) in direct mode or
    # the GUI's saved settings via project.connect_all().

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## Αυτόματη εκκίνηση backend

Τα έξυπνα-connect — `connect_camera`, `connect_array`, `connect_daq_sensor` και `discover_lattice_cameras` — είναι «λεπτοί» HTTP πελάτες που υποθέτουν ότι ένα backend να ακούει στη διεύθυνση `127.0.0.1:5000` (η προεπιλεγμένη διεύθυνση URL της επιφάνειας smart-connect). Όταν το GUI ή το CLI εκτελείται ήδη, υπάρχει ένα. Από ένα απλό σενάριο, μπορεί να μην υπάρχει — οπότε αυτές οι συναρτήσεις **εκκινούν αυτόματα το συνοδευτικό εκτελέσιμο του backend** (χωρίς παράθυρο-less, με τον ίδιο τρόπο που το κάνει το `ChlorosLocal`) πριν από την πρώτη κλήση τους και, στη συνέχεια, περιμένουν έως και `backend_startup_timeout` μέχρι να ξεκινήσει.

Κανόνες:

- **Δημιουργείται πάντα μόνο ένα τοπικό URL.** Ένα `backend_url` που δείχνει προς τα `localhost` / `127.0.0.1` / `[::1]` είναι επιλέξιμο· οποιοσδήποτε άλλος κεντρικός υπολογιστής θεωρείται ότι ανήκει σε κάποιον άλλο και δεν δημιουργείται ποτέ.
- **Το backend παραμένει σε λειτουργία για επαναχρησιμοποίηση** (όπως και το CLI) — δεν υπάρχει σιωπηρό κλείσιμο όταν το σενάριό σας τερματιστεί. Η εκ νέου εκτέλεση του σεναρίου επαναχρησιμοποιεί το ενεργό backend.
- **Αποκλείστε τη χρήση με το `auto_start_backend=False`** σε οποιαδήποτε από αυτές τις κλήσεις (π.χ. όταν έχετε ορίσει ένα απομακρυσμένο backend ή διαχειρίζεστε εσείς ο ίδιος τον κύκλο ζωής του backend).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Εάν το ενσωματωμένο εκτελέσιμο δεν μπορεί να εντοπιστεί ή να ξεκινήσει, η επακόλουθη κλήση HTTP δημιουργεί ένα ανιχνεύσιμο, **εξαρτώμενο από την πλατφόρμα** `ChlorosConnectError` αντί για ένα απλό ίχνος απόρριψης σύνδεσης — στο Windows σας κατευθύνει προς την εφαρμογή επιφάνειας εργασίας ή μια εντολή `chloros-cli`· στο Linux (χωρίς GUI) σας κατευθύνει προς μια εντολή `chloros-cli` ή το `.deb`.

---

## Περιβάλλον &amp; Κεφαλίδες

Το SDK επισημαίνει κάθε κλήση HTTP του backend με το `X-Chloros-Client: sdk`. Το backend εφαρμόζει τους κανόνες αδειοδότησης SDK / CLI (απαιτείται σύνδεση **και** ένα επί πληρωμή πακέτο Chloros+) αντί για τη διαδρομή δωρεάν επιπέδου του GUI. Αυτό ρυθμίζεται αυτόματα κατά την εισαγωγή — δεν χρειάζεται να κάνετε τίποτα.

Τα `http://localhost` και `http://127.0.0.1` αναγνωρίζονται ως τοπικό backend. Οι κλήσεις προς άλλους κεντρικούς υπολογιστές (π.χ. η δική σας υπηρεσία ανάλυσης) παραμένουν ως έχουν.

Παρακάμψτε το backend URL περνώντας το `backend_url=` (ή το `api_url=` στο `ChlorosLocal`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(Ένα `backend_url` χωρίς loopback φτάνει μόνο σε backend τύπου source/dev — τα προεγκατεστημένα backends συνδέονται μόνο με loopback· δείτε τη Λειτουργία Απομακρυσμένου Backend για το μοτίβο του τούνελ.)

---

## Έκδοση &amp; Συμβατότητα

- Η έκδοση SDK εμφανίζεται ως `chloros_sdk.__version__`.
- Το SDK περιορίζει τη συμπεριφορά στην έκδοση του ενσωματωμένου backend. Η ανάμειξη ενός παλαιότερου SDK με ένα νεότερο backend συνήθως λειτουργεί (τερματικά με συμβατότητα προς τα εμπρός), αλλά η ανάμειξη ενός νεότερου SDK με ένα παλαιότερο backend ενδέχεται να προκαλέσει σφάλματα `404` σε νέα τερματικά — αναβαθμίστε την εφαρμογή για επιτραπέζιους υπολογιστές ώστε να ταιριάζει.
- Η επιφάνεια «smart-connect» (`connect_camera` / `connect_array` / `connect_daq_sensor`) και το τελικό σημείο ανάλυσης δικτύου επιστρέφουν σταθερά σχήματα JSON· τα νέα πεδία προστίθενται.

---

## Οδηγίες αντιμετώπισης προβλημάτων

- **`ChlorosAuthenticationError: Login required`** → Εκτελέστε το `chloros-cli login EMAIL PASSWORD` μία φορά σε αυτόν τον υπολογιστή ή συνδεθείτε μέσω της εφαρμογής για επιτραπέζιους υπολογιστές Chloros.
- **`ChlorosConnectError: No Chloros backend is running …`** → Οι κλήσεις smart-connect ενεργοποιούν αυτόματα ένα τοπικό backend, οπότε αυτό εμφανίζεται μόνο όταν το συμπεριλαμβανόμενο εκτελέσιμο δεν μπορεί να βρεθεί/ξεκινήσει (π.χ. ένας κεντρικός υπολογιστής που χρησιμοποιεί μόνο pip χωρίς πακέτο επιφάνειας εργασίας). Το μήνυμα εξαρτάται από την πλατφόρμα: σε Windows ανοίξτε την εφαρμογή επιφάνειας εργασίας ή εκτελέστε οποιαδήποτε εντολή `chloros-cli`· σε Linux εκτελέστε μια εντολή `chloros-cli` (δεν υπάρχει GUI) ή εγκαταστήστε το `.deb`. Για ένα απομακρυσμένο backend, περάστε το `backend_url=` (και το `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** κατά την εισαγωγή → Η φόρτωση του `lattice_sdk` απέτυχε (συνήθως οι DLL χρόνου εκτέλεσης του Arena SDK δεν είναι εγκατεστημένες). Η επιφάνεια εκτός κάμερας εξακολουθεί να λειτουργεί.
- **Η σύνδεση πίνακα επιστρέφει ανάλυση χαμηλότερη από την εγγενή**→ Η αυτόματη προετοιμασία (smart-prep) του backend-μειώνει το μέγεθος του καρέ για να ταιριάζει στο wire. Χρησιμοποιήστε το `analyze_array_network()` για να δείτε τον λόγο και, στη συνέχεια, είτε αναβαθμίστε τη σύνδεση, είτε αποδεχτείτε τη μείωση, είτε περάστε το `force_tier="slip-emit-and-capture"` για διαδοχική καταγραφή. Το δίχτυ ασφαλείας της μείωσης**δεν** καλύπτει τη συνολική-συνδρομής (`oversubscribed: true`, πεδία fps 0): ο υπερβολικός αριθμός καμερών για το καλώδιο δεν μπορεί να διορθωθεί με binning/ROI — μειώστε τον αριθμό των καμερών, ενεργοποιήστε τα jumbo frames ή μεταβείτε σε μια ταχύτερη κάρτα δικτύου (βλ. [Υπερ-εγγραφή](#over-subscription-the-per-cam-floor)).
- **Το `analyze_array_network()` αναφέρει ότι ο δακτύλιος λήψης (RX ring) της κάρτας δικτύου είναι πολύ μικρός (~0,26 MB) / εμφανίζονται μηνύματα &quot;FRAMES WILL DROP&quot;** → Ο δακτύλιος λήψης της κάρτας δικτύου του κεντρικού υπολογιστή βρίσκεται στην προεπιλεγμένη τιμή (συχνά επαναρυθμίζεται στο 32 μετά από ενημέρωση του προγράμματος οδήγησης της κάρτας δικτύου). Σε προσαρμογέα Realtek USB 10GbE, ορίστε τα `ReceiveBufferLen=256` και `PendingReceives=64` (με δικαιώματα διαχειριστή) και, στη συνέχεια, επανεκκινήστε το backend ώστε να διαβάσει εκ νέου το δακτύλιο. Πλήρης διαδικασία: [Αναφορά CLI → Ρύθμιση &amp; Βελτιστοποίηση Κάρτας Δικτύου Κεντρικού Υπολογιστή](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Ο κεντρικός υπολογιστής «κολλάει» κατά την επανεκκίνηση/το κλείσιμο, και αργότερα εμφανίζονται σφάλματα WMI `Invalid class` / η κάρτα δικτύου δεν ενεργοποιείται** → Παρωχημένο πρόγραμμα οδήγησης USB 10GbE που προκαλεί το `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Ενημερώστε το πρόγραμμα οδήγησης του προσαρμογέα σε μια τρέχουσα έκδοση (≥ 2026) και εφαρμόστε εκ νέου τις ρυθμίσεις του δακτυλίου λήψης. Ανατρέξτε στην [Αναφορά CLI → Ρύθμιση και βελτιστοποίηση κάρτας δικτύου κεντρικού υπολογιστή](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Η αντανάκλαση απορρίφθηκε** → Για την αντανάκλαση σε απόλυτη κλίμακα, πρέπει να έχει αντιστοιχιστεί μια ενεργή συσκευή DAQ στην κάμερα (ή στη συστοιχία). Αντιστοιχίστε τη μέσω του GUI ή χρησιμοποιήστε το `processing="radiance"` (W/m²/sr/nm), το οποίο δεν απαιτεί ζευγαρωμένο αισθητήρα.
- **Η καταγραφή με το `smart=True` διαρκεί περισσότερο από το αναμενόμενο** → Η σύγκλιση AE εξαρτάται από τη δυναμική της σκηνής· αυξήστε το `exposure_tolerance_pct` ή μειώστε το `stability_window_s` αν θέλετε ένα ταχύτερο (λιγότερο σταθερή) ενεργοποίηση.

---

## Δείτε επίσης

- [Αναφορά CLI](cli-reference.md) — κάθε υποεντολή CLI αντιστοιχεί σε μια κλήση SDK.
- [Οδηγός αισθητήρων DAQ](../daq/README.md) — κανόνες καλωδίωσης, βαθμονόμησης και καταγραφής για κάθε αισθητήρα.
- Ηλεκτρονική τεκμηρίωση: `https://mapir.gitbook.io/chloros/api-python-sdk`</id></sn>
