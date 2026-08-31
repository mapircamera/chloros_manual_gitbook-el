# Εγκατάσταση του Linux

Το Chloros διανέμεται για το Linux ως πακέτα `.deb` που εγκαθιστούν το CLI και τον διακομιστή backend. Το Python SDK είναι ένα ξεχωριστό πακέτο pip (περιλαμβάνεται επίσης στο `.deb` ως wheel με αντίστοιχη έκδοση).

Τα ονόματα των αρχείων των πακέτων περιλαμβάνουν την έκδοση και την αρχιτεκτονική: `chloros_1.2.0_amd64.deb` για x86_64 και `chloros_1.2.0_arm64_jp6.deb` για εκδόσεις JetPack 6 Jetson. Αντικαταστήστε το αρχείο που κατεβάσατε πραγματικά στις παρακάτω εντολές.

***

## Linux amd64 (x86_64)

### Απαιτήσεις συστήματος

| Απαίτηση | Ελάχιστη | Συνιστώμενη |
| --- | --- | --- |
| **Διανομή** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Επεξεργαστής** | x86_64 (Intel/AMD) | Intel Core i7 ή καλύτερος |
| **Μνήμη (RAM)** | 8 GB | 16 GB ή περισσότερο |
| **Κάρτα γραφικών** | Καμία (επεξεργασία από τον επεξεργαστή) | GPU NVIDIA με 4 GB+ VRAM (12 GB+ ξεκλειδώνει το `GPU_PARALLEL`, 7 GB+ διατηρεί το Texture Aware εκτός της διαδρομής μονής εικόνας) |
| **Χώρος αποθήκευσης** | 2 GB ελεύθερου χώρου | SSD με 10 GB+ ελεύθερου χώρου |
| **Python** | Python 3.7+ (για το SDK) | Python 3.10+ |

> **Το Ubuntu 20.04 και το Debian 11 δεν υποστηρίζονται.** Ο κατάλογος εξαρτήσεων του `.deb`
> προέρχεται από αυτά με τα οποία συνδέεται πραγματικά το backend του Chloros, και αυτό περιλαμβάνει
> το `libc6 (>= 2.34)`. Τόσο το Focal όσο και το bullseye διαθέτουν το glibc 2.31, οπότε το `apt` αρνείται την
> εγκατάσταση εξ αρχής, αντί να επιτρέψει την αποτυχία της αργότερα κατά την εκτέλεση.

### Εγκατάσταση

```bash
sudo dpkg -i chloros_1.2.0_amd64.deb
sudo apt-get install -f    # pulls the declared dependencies (libibverbs1, libcap2-bin)
```

{% hint style="info" %}
Το `dpkg -i` δεν επιλύει τις εξαρτήσεις. Εάν αναφέρει πακέτα που λείπουν, το `sudo apt-get install -f` (ή το `sudo apt --fix-broken install`) ολοκληρώνει την εγκατάσταση — αυτή είναι η κανονική διαδικασία, όχι σφάλμα.
{% endhint %}

Επαληθεύστε την εγκατάσταση:



<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo dpkg -i chloros_1.2.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block naming the bundled wheel path under /usr/lib/chloros/sdk/, any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend.service" hint -->

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```***

## Linux arm64 (NVIDIA Jetson)

### Απαιτήσεις συστήματος

| Απαίτηση | Ελάχιστη | Συνιστώμενη |
| --- | --- | --- |
| **Πλατφόρμα** | NVIDIA Jetson με JetPack 6 | Jetson Orin NX 16 GB ή AGX Orin |
| **JetPack** | JetPack 6.x | Τελευταία έκδοση JetPack 6 |
| **Μνήμη (RAM)** | 8 GB (κοινή χρήση GPU/CPU) | 16 GB+ κοινή χρήση (12 GB+ είναι το κατώτατο όριο για παράλληλους εργαζόμενους GPU) |
| **Αποθήκευση** | 2GB ελεύθερου χώρου | SSD NVMe με 10GB+ ελεύθερου χώρου |
| **Python** | Python 3.7+ (για το SDK) | Python 3.10+ |

### Εγκατάσταση

```bash
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Ίδια διάταξη με το amd64 `.deb`, με έκδοση CUDA βελτιστοποιημένη για Jetson Orin / Orin NX / Orin Nano. Για πληροφορίες σχετικά με τη μνήμη, τη θερμική συμπεριφορά και τη συμπεριφορά κατά την εγκατάσταση στο πεδίο του Jetson, ανατρέξτε στον [Οδηγό NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Εγκατάσταση Python SDK (Όλα τα Linux)

Το SDK είναι ένας καθαρά Python HTTP πελάτης για το backend, οπότε το ίδιο πακέτο λειτουργεί σε amd64 και arm64. Δύο πηγές:**Από το PyPI** — η δημοσιευμένη σταθερή έκδοση:

```bash
pip install chloros-sdk
```

**Από το συνοδευτικό αρχείο wheel** — εγγυημένη συμβατότητα με το CLI/backend που μόλις εγκαταστήσατε (χρησιμοποιήστε το όταν το `.deb` σας είναι νεότερο από αυτό του PyPI):

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

{% hint style="warning" %}
**Οι διανομές PEP 668** (Ubuntu 23.10+, Debian 12+) δεν επιτρέπουν εγκαταστάσεις pip σε επίπεδο συστήματος. Χρησιμοποιήστε το `pip install --user …`, ένα εικονικό περιβάλλον ή το `sudo pip install --break-system-packages …`. Το πρόγραμμα εγκατάστασης πακέτων δεν εγκαθιστά ποτέ αυτόματα το SDK στο Python του συστήματός σας — αυτή η επιλογή επαφίεται σε εσάς.
{% endhint %}

Προαιρετικά πρόσθετα:

| Πρόσθετο | Εντολή | Προσθέτει |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` για ζωντανή ροή προόδου |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` για μεταφορά μέσω BLE (DAQ-M) |

Επαληθεύστε το SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Το `.deb` εγκαθιστά το Chloros CLI και το backend. Το Python SDK επικοινωνεί με αυτό το backend μέσω ενός τοπικού HTTP API (`http://127.0.0.1:5000`) και το εκκινεί αυτόματα όταν χρειάζεται. Χρησιμοποιείτε πάντα τη συγκεκριμένη διεύθυνση IPv4 αντί για το `localhost` — το `localhost` μπορεί να αναλυθεί σε `::1` και να κοστίζει περίπου δύο δευτερόλεπτα ανά αίτημα.
{% endhint %}

***

## Αρχική ρύθμιση

### 1. Σύνδεση

Η πρόσβαση στα CLI και SDK απαιτεί επίπεδο συνδρομής Chloros+ (**Copper** ή ανώτερο), το οποίο επιβάλλεται από την πλευρά του διακομιστή: ένας καλών που έχει αποσυνδεθεί λαμβάνει το `401 AUTH_REQUIRED`, ενώ ένας καλών με δωρεάν πακέτο (Iron) λαμβάνει το `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Τα διαπιστευτήρια αποθηκεύονται προσωρινά στο `~/.chloros/user_session.json`.

{% hint style="warning" %}
**Πρέπει να συνδεθείτε ξανά μετά από κάθε εγκατάσταση ή αναβάθμιση.** Το σενάριο `prerm` του πακέτου διαγράφει σκόπιμα το `~/.chloros/user_session.json` και την αποθηκευμένη άδεια χρήσης για κάθε χρήστη στον υπολογιστή, έτσι ώστε μια νέα έκδοση να επικυρώνει πάντα εκ νέου την άδεια χρήσης αντί να βασίζεται σε μια παλιά προσωρινή μνήμη.
{% endhint %}

### 2. Ελέγξτε την κατάσταση της άδειας χρήσης σας

```bash
chloros-cli status
```

Το `chloros-cli status` λειτουργεί σε οποιοδήποτε επίπεδο (συμπεριλαμβανομένου του δωρεάν), οπότε μπορείτε πάντα να δείτε γιατί η πρόσβαση είναι ή δεν είναι διαθέσιμη.

### 3. Εκτελέστε διαγνωστικά συστήματος

```bash
chloros-cli selftest
```

Εκτελούνται επτά έλεγχοι με τη σειρά, και η εντολή επιστρέφει μη μηδενικό αποτέλεσμα αν κάποιος από αυτούς αποτύχει:

| # | Έλεγχος | Τι αποδεικνύει |
| --- | --- | --- |
| 1 | **Έκδοση** | Το CLI αναφέρει την έκδοσή του (`v1.2.0`). |
| 2 | **Θύρα διαθέσιμη** | Η θύρα 5000 είναι ελεύθερη, *ή* έχει ήδη απαντηθεί από ένα υγιές backend Chloros (αυτό θεωρείται επιτυχία). |
| 3 | **Εκκίνηση backend** | Το εκτελέσιμο του backend εκκινείται. |
| 4 | **Δοκιμή API (`/api/test`)** | Το backend απαντά με το `status: ok`. |
| 5 | **Πληροφορίες συστήματος** | Εμφανίζει το `GPU: <name>, CUDA: <bool>, PyTorch: <version>` από το `/api/system-info`. |
| 6 | **Μοντέλα αποθορυβοποίησης** | Εντοπίζει μοντέλα `*.pth.enc` (στο Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + Αποθορυβωτής**| Το Texture Aware είναι πράγματι χρησιμοποιήσιμο — απαιτεί CUDA**και** τουλάχιστον ένα αρχείο μοντέλου. |

Η εκτέλεση ολοκληρώνεται με το `N/7 checks passed`, αναφέροντας τυχόν αποτυχίες κατά όνομα.

### 4. Επεξεργασία του πρώτου σας συνόλου δεδομένων

```bash
chloros-cli process ~/datasets/flight001
```

***

## Αρχεία και κατάλογοι

### Ανά χρήστη

Το Chloros αποθηκεύει τα διαπιστευτήριά του και τη διαμόρφωση του CLI σε έναν ενιαίο κατάλογο για όλες τις πλατφόρμες, **`~/.chloros/`** (στο Windows, `%USERPROFILE%\.chloros\`). Δύο προσωρινές μνήμες που αφορούν συγκεκριμένα το Linux ακολουθούν αντίθετα τις συμβάσεις XDG — αυτές τηρούν τις ρυθμίσεις των `XDG_CONFIG_HOME` / `XDG_CACHE_HOME` όταν έχουν οριστεί.

| Διαδρομή | Σκοπός |
| --- | --- |
| `~/.chloros/user_session.json` | Προσωρινή μνήμη συνεδρίας σύνδεσης που δημιουργείται από το `chloros-cli login` (διαγράφεται σε κάθε εγκατάσταση/αναβάθμιση πακέτου) |
| `~/.chloros/working_directory.txt` | Προεπιλεγμένη παράκαμψη φακέλου έργου (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Προτίμηση γλώσσας CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Ρύθμιση γλώσσας που μοιράζεται με το Windows GUI — ένα `language` εδώ έχει προτεραιότητα έναντι του `cli_language.json` |
| `~/.chloros/update_cache.json` | Προσωρινή αποθήκευση διάρκειας μίας ώρας για τον έλεγχο ενημερώσεων εκκίνησης του Linux/Jetson |
| `~/.chloros/backend.log` | Αρχείο καταγραφής του backend όταν αυτό ξεκίνησε από το CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Πακέτα βαθμονόμησης LATTICE ανά κάμερα που έχουν αποθηκευτεί στην προσωρινή μνήμη, ταξινομημένα με βάση τον σειριακό αριθμό και το hash του πακέτου |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Προαιρετικές παρακάμψεις χρήστη για προφίλ διόρθωσης ορίου DAQ |
| `~/.config/chloros/system_config.json` | Προφίλ υλικού αποθηκευμένο στην κρυφή μνήμη από τη Δυναμική Προσαρμογή Υπολογιστικής Ισχύος — διαγράψτε το για να επιβάλετε νέα ανίχνευση υλικού |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Αρχεία καταγραφής διακομιστή backend, ένα αρχείο ανά εκκίνηση |
| `~/Chloros Projects/` | Προεπιλεγμένος φάκελος έργου όταν δεν έχει οριστεί καμία παράκαμψη |

### Σε επίπεδο συστήματος

| Διαδρομή | Σκοπός |
| --- | --- |
| `/usr/bin/chloros-cli` | Σενάριο περιτύλιξης — ορίζει το `LD_LIBRARY_PATH` για τις ενσωματωμένες εγγενείς βιβλιοθήκες και, στη συνέχεια, εκτελεί το πραγματικό δυαδικό αρχείο |
| `/usr/bin/chloros-backend` | Σενάριο περιτύλιξης — το ίδιο, συν το `CHLOROS_PRODUCTION=1`, ώστε η πύλη ελέγχου ταυτότητας του backend να μην μπορεί ποτέ να απενεργοποιηθεί σιωπηλά |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Τα μεταγλωττισμένα εκτελέσιμα αρχεία |
| `/usr/lib/chloros/arena_runtime/` | Περιβάλλον εκτέλεσης Arena SDK που απαιτείται από τις κάμερες LATTICE |
| `/usr/lib/chloros/models/*.pth.enc` | Κρυπτογραφημένα μοντέλα αποθορυβοποίησης που χρησιμοποιούνται από το Texture Aware debayer |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python SDK wheel που αντιστοιχεί ακριβώς σε αυτή την έκδοση |
| `/usr/lib/chloros/exiftool` | Ενσωματωμένο exiftool (με συμβολικό σύνδεσμο προς το `/usr/local/bin/exiftool` μόνο αν δεν υπάρχει exiftool στο σύστημα) |
| `/etc/chloros/update.conf` | Διαμόρφωση καναλιού ενημέρωσης που διαβάζεται από το `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Ρυθμίζει το `net.ipv4.ip_unprivileged_port_start = 319` έτσι ώστε ομπορεί να συνδέσει τις θύρες PTP χωρίς δικαιώματα root |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Κατευθύνει τον δυναμικό φορτωτή στο `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Παρέχει στον συνδεδεμένο χρήστη πρόσβαση στη σειριακή γέφυρα USB DAQ-U (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend.service` | Υπηρεσία backend που παραμένει πάντα ενεργή (εγκατεστημένη, **δεν είναι ενεργοποιημένη**) |
| `/usr/share/applications/chloros-cli.desktop` | Είσοδος στο μενού εφαρμογών «Chloros CLI» που ανοίγει ένα τερματικό |

## Θέση του εκτελέσιμου αρχείου του backend

Τα CLI και SDK ανιχνεύουν αυτόματα το backend:

| Συστατικό | Διαδρομή |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` |

Παράκαμψη της διαδρομής του backend με τη σημαία `--backend-exe` CLI ή την παράμετρο κατασκευαστή `backend_exe` SDK, και τη θύρα με το `--port` (προεπιλογή `5000`).

{% hint style="info" %}
Το `CHLOROS_BACKEND_URL` κατευθύνει τις οικογένειες εντολών **`lattice`**,**`project`**και**`daq pool-*`** σε ένα απομακρυσμένο backend. Οι βασικές εντολές (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) το αγνοούν σκόπιμα και στοχεύουν πάντα το `http://127.0.0.1:<port>`.
{% endhint %}

***

## Κάμερες LATTICE και αισθητήρες φωτός DAQ στο Linux

Όλες οι οικογένειες εντολών live-hardware λειτουργούν στο Linux (amd64 και Jetson):

* **`chloros-cli lattice`** — ανίχνευση, σύνδεση, διαμόρφωση και καταγραφή από κάμερες LATTICE και συγχρονισμένες συστοιχίες. Το `.deb` ενσωματώνει το περιβάλλον εκτέλεσης Arena SDK που απαιτείται και το καταχωρεί στον δυναμικό φορτωτή.
* **`chloros-cli daq pool-*`** — σύνδεση αισθητήρων φωτός DAQ-U/M/E μέσω του backend pool, ροή βαθμονομημένων φασμάτων και καταγραφή αρχείων `.daq`. Το μεταγλωττισμένο CLI περιλαμβάνει μόνο την οικογένεια `pool-*`: `pool-connect`, `pool-disconnect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`.
* **`chloros-cli project`** — εκτέλεση ενός αποθηκευμένου έργου (με τις κάμερες, τους αισθητήρες και τις ρυθμίσεις επεξεργασίας του) χωρίς οθόνη.
* **`chloros-cli time-sync`** — ελέγξτε τον κεντρικό διακομιστή PTP στον οποίο εκτελείται το backend Chloros για κάμερες LATTICE και αισθητήρες DAQ-E.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

Το `--sensor-id` απαιτείται από τα `pool-latest`, `pool-stream`, `pool-record` και `pool-set-cap`· το `pool-list` εμφανίζει τα αναγνωριστικά που βρίσκονται επί του παρόντος στη δεξαμενή.

{% hint style="info" %}
**Προτιμήστε το `--eth-host` για την πρώτη σύνδεση DAQ-E σε έναν υπολογιστή με πολλαπλές συνδέσεις.** Η αυτόματη ανίχνευση πραγματοποιεί αναζήτηση στο mDNS και ενδέχεται να μην εντοπίσει τη διεπαφή του αισθητήρα λόγω κενής κρυφής μνήμης ARP, οπότε το πρώτο `pool-connect --eth` μετά την εκκίνηση ενδέχεται να αποτύχει ακόμη και αν ο αισθητήρας είναι απολύτως λειτουργικός. Η παροχή της διεύθυνσης IP ή του ονόματος υπολογιστή του αισθητήρα παρακάμπτει εντελώς τη διαδικασία ανίχνευσης.
{% endhint %}

**Τα δικαιώματα σειριακής θύρας DAQ-U** διαχειρίζονται από τον εγκατεστημένο κανόνα udev (`uaccess` + ομάδα `dialout`). Εάν ένας αισθητήρας που ήταν ήδη συνδεδεμένος παραμένει μη προσβάσιμος, φορτώστε εκ νέου τους κανόνες ή συνδέστε τον ξανά:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Δείτε την [αναφορά CLI](../CLI.md) για το πλήρες σύνολο εντολών.

### PTP συνεχούς λειτουργίας για κεντρικούς υπολογιστές χωρίς οθόνη

Κατά την πρώτη εγκατάσταση, δημιουργείται η μονάδα systemd `chloros-backend.service`, αλλά **δεν είναι ενεργοποιημένη**. Σε έναν Jetson χωρίς οθόνη ή σε έναν διακομιστή που πρέπει να διατηρεί τον συγχρονισμό χρόνου PTP σε συνεχή λειτουργία για αισθητήρες DAQ-E και κάμερες LATTICE, ενεργοποιήστε την:

```bash
sudo systemctl enable --now chloros-backend.service
sudo systemctl status chloros-backend.service
```

Χωρίς αυτήν, το PTP λειτουργεί μόνο όσο εκτελείται το backend Chloros — δηλαδή, κατά τη διάρκεια μιας ενεργής συνεδρίας CLI/SDK.

Η μονάδα συνδέει το backend με το `127.0.0.1:5000` (ρυθμίσεις περιβάλλοντος `CHLOROS_HOST` / `CHLOROS_PORT` εντός της μονάδας· αντικαταστήστε με το `sudo systemctl edit chloros-backend.service`) και το επανεκκινεί σε περίπτωση αποτυχίας μετά από 5 δευτερόλεπτα.

**Πώς το PTP αποκτά τις θύρες του.** Το PTP χρησιμοποιεί τις θύρες UDP 319/320, και οι δύο κάτω από το κανονικό κατώτατο όριο των 1024 προνομιούχων θυρών. Το `postinst` του πακέτου γράφει το `/etc/sysctl.d/60-chloros-ptp.conf` με το `net.ipv4.ip_unprivileged_port_start = 319`, πράγμα που επιτρέπει στο backend να τις δεσμεύσει ενώ εκτελείται ως χρήστης σας. Εφαρμόζει επίσης το `setcap cap_net_bind_service,cap_net_raw=+ep` στο εκτελέσιμο του backend ως μέτρο ασφαλείας — γι’ αυτό το `libcap2-bin` αποτελεί δηλωμένη εξάρτηση του πακέτου.***

## Παραδείγματα σεναρίων Bash

{% hint style="info" %}
**Κωδικοί εξόδου φιλικοί προς τα σενάρια.**Το `chloros-cli process` τερματίζεται με κωδικό `0` σε περίπτωση επιτυχίας και**με κωδικό διαφορετικό από μηδέν σε περίπτωση αποτυχίας — συμπεριλαμβανομένης μιας εκτέλεσης που ζήτησε προϊόντα εικόνας αλλά δεν έγραψε κανένα** (εμφανίζει το `Processing finished but wrote no image products.` και αναφέρει το όνομα του φακέλου του έργου καθώς και τις συνήθεις αιτίες). Οι επιτυχημένες εκτελέσεις αναφέρουν πόσα προϊόντα εικόνας δημιουργήθηκαν (`Image products written: N`). Κωδικοί εξόδου: `0` επιτυχία, `1` αποτυχία, `2` σφάλμα παραμέτρου, `130` διακοπή.
{% endhint %}

### Επεξεργασία πολλαπλών συνόλων δεδομένων

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    if chloros-cli process "$dataset" --format "TIFF (32-bit, Percent)"; then
        echo "Done: $(basename "$dataset")"
    else
        echo "FAILED: $(basename "$dataset")" >&2
    fi
done
```

### Επεξεργασία με προσαρμοσμένες ρυθμίσεις

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

Οι έγκυρες τιμές του `--format` είναι ακριβώς τέσσερις και περιέχουν κενά — πάντα να τις τοποθετείτε σε εισαγωγικά:

| Τιμή `--format` | Φάκελος εξόδου |
| --- | --- |
| `TIFF (16-bit)` *(προεπιλογή)* | `tiff16` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

Το `--debayer` δέχεται το `standard` (προεπιλογή) ή το `texture-aware` (Chloros+).

### Αυτοματοποιημένη επεξεργασία με Cron

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Python SDK Παράδειγμα

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "NDRE"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Αντιμετώπιση προβλημάτων

### Το CLI δεν εντοπίστηκε μετά την εγκατάσταση

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# List everything the package installed
dpkg -L chloros

# Reload your shell
source ~/.bashrc
```

### Άρνηση πρόσβασης

```bash
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### «setcap failed» κατά την εγκατάσταση

Το `.deb` εφαρμόζει το `cap_net_bind_service` στο `/usr/lib/chloros/chloros-backend`, ώστε να μπορεί να δεσμεύσει τις θύρες PTP 319/320 χωρίς δικαιώματα root. Εάν το `libcap2-bin` απουσίαζε κατά την εγκατάσταση, η κλήση παραλείπεται. Εγκαταστήστε το και επανεγκαταστήστε το πακέτο:

```bash
sudo apt install libcap2-bin
sudo apt reinstall chloros
```

### Το PTP δεν ξεκινά / Δεν είναι δυνατή η δέσμευση της θύρας 319

Βεβαιωθείτε ότι το όριο των θυρών χωρίς δικαιώματα έχει μειωθεί και, αν δεν έχει γίνει, εφαρμόστε το εκ νέου για την τρέχουσα εκκίνηση:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Στη συνέχεια, ελέγξτε το grandmaster:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### «Δεν βρέθηκαν προγράμματα οδήγησης κάμερας LATTICE»

Το περιβάλλον εκτέλεσης Arena SDK δεν αναγνωρίζεται. Βεβαιωθείτε ότι η διαμόρφωση του φορτωτή που δημιουργεί το πακέτο υπάρχει και έχει ενημερωθεί:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
sudo ldconfig
ls /usr/lib/chloros/arena_runtime | head
```

### Αποτυχία εκκίνησης του backend

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

Τα αρχεία καταγραφής του backend για την αποτυχημένη εκκίνηση βρίσκονται στο `~/.cache/chloros/logs/`.

### Δεν εντοπίστηκε CUDA

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

Το `chloros-cli selftest` αναφέρει το ίδιο πράγμα σε μία γραμμή: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Λείπουν κοινόχρηστες βιβλιοθήκες

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

### Αργή εκκίνηση σε συστήματα με κάρτα SD

Τα μεταγλωττισμένα εκτελέσιμα αρχεία αποσυμπιέζονται σε έναν προσωρινό κατάλογο σε κάθε εκκίνηση. Εάν υπάρχει το `/mnt/ssd/tmp`, το Chloros το χρησιμοποιεί αυτόματα· διαφορετικά, ορίστε το `TMPDIR` σε ένα γρήγορο σύστημα αρχείων:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Ενημέρωση του Chloros στο Linux

Η εντολή `update` διατίθεται μόνο στο Linux/Jetson. Ελέγχει την έκδοση που έχει δημοσιευτεί στο κανάλι ενημέρωσης που έχει ρυθμιστεί στο `/etc/chloros/update.conf` και προτείνει τη λήψη και την εγκατάσταση της αντίστοιχης έκδοσης `.deb`:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

Στο Linux/Jetson, η εντολή CLI εκτελεί επίσης έναν έλεγχο ενημέρωσης χωρίς αναμονή σε κάθε εκκίνηση (το αποτέλεσμα αποθηκεύεται προσωρινά για μία ώρα στο `~/.chloros/update_cache.json`) και εμφανίζει το μήνυμα `Update available: vX.Y.Z` όταν υπάρχει νεότερη έκδοση. Οι ρυθμίσεις και τα έργα σας διατηρούνται μετά την ενημέρωση· θα χρειαστεί να συνδεθείτε ξανά στη συνέχεια.

## Απεγκατάσταση

```bash
sudo apt remove chloros
```

Η απεγκατάσταση σταματά το `chloros-backend.service`, επαναφέρει το προεπιλεγμένο κατώτατο όριο θυρών χωρίς δικαιώματα (1024), αφαιρεί τον συμβολικό σύνδεσμο του ενσωματωμένου exiftool και τη διαμόρφωση του φορτωτή Arena, καθώς και διαγράφει τα αποθηκευμένα διαπιστευτήρια. Τα έργα σας και τα αρχεία δεδομένων `~/.chloros/` παραμένουν ανέπαφα.

***

## Επόμενα βήματα

* [Οδηγός NVIDIA Jetson](nvidia-jetson-guide.md) — Βελτιστοποίηση και ανάπτυξη ειδικά για το Jetson
* [CLI : Γραμμή εντολών](../CLI.md) — ο οδηγός CLI
* [API : Python SDK](../api-python-sdk.md) — ο οδηγός SDK
* [Αναφορά CLI](../reference/cli-reference.md) και [Αναφορά SDK](../reference/sdk-reference.md) — εξαντλητικές λίστες εντολών για την έκδοση 1.2.0
* [Δυναμική προσαρμογή υπολογιστικής ισχύος](../processing-architecture/dynamic-compute-adaptation.md) — πώς το Chloros προσαρμόζεται στο υλικό σας
