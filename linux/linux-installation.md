# Εγκατάσταση σε Linux

Το Chloros διανέμεται για Linux ως πακέτα `.deb`, τα οποία εγκαθιστούν το CLI, τον διακομιστή backend και το Python SDK (για το σύστημα `python3`, καθώς και ένα wheel αντίστοιχης έκδοσης για virtualenvs).

Τα ονόματα των αρχείων πακέτων περιλαμβάνουν την έκδοση και την αρχιτεκτονική: `chloros_1.3.0_amd64.deb` για x86_64, και `chloros_1.3.0_arm64_jp6.deb` για arm64 (NVIDIA Jetson, JetPack 6). Αντικαταστήστε το αρχείο που κατεβάσατε πραγματικά στις παρακάτω εντολές.

{% hint style="info" %}
**Νέο στην έκδοση 1.3.0: τα προγράμματα διατίθενται αποσυμπιεσμένα.** Το backend και η CLI εγκαθίστανται ως φάκελοι, `/usr/lib/chloros/backend/` και `/usr/lib/chloros/cli/`, αντί για μεμονωμένα αρχεία αυτόματης αποσυμπίεσης. Δεν αποσυμπιέζονται πλέον σε έναν προσωρινό φάκελο σε κάθε εκκίνηση, οπότε ξεκινούν πολύ πιο γρήγορα (στο x86_64 το backend συνήθιζε να χρειάζεται περίπου 100 δευτερόλεπτα για την αποσυμπίεση σε κάθε εκκίνηση) και δεν χρειάζονται πλέον αρκετά GB ελεύθερου χώρου στο `/tmp`. Το amd64 `.deb` έχει επίσης περίπου το μισό μέγεθος από το προηγούμενο.
{% endhint %}

***

## Linux amd64 (x86_64)

### Απαιτήσεις συστήματος

| Απαίτηση | Ελάχιστη | Συνιστώμενη |
| --- | --- | --- |
| **Διανομή** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Επεξεργαστής** | x86_64 (Intel/AMD) | Intel Core i7 ή καλύτερος |
| **Μνήμη (RAM)** | 8 GB | 16 GB ή περισσότερα |
| **Κάρτα γραφικών** | Καμία (επεξεργασία από τον επεξεργαστή) | GPU NVIDIA με 4 GB+ VRAM (12 GB+ ξεκλειδώνει το `GPU_PARALLEL`, 7 GB+ διατηρεί το Texture Aware εκτός της διαδρομής μονής εικόνας) |
| **Χώρος αποθήκευσης** | 2 GB ελεύθερου χώρου | SSD με 10 GB+ ελεύθερου χώρου |
| **Python** | Python 3.7+ (για το SDK) | Python 3.10+ |

> **Τα Ubuntu 20.04 και Debian 11 δεν υποστηρίζονται.** Ο κατάλογος εξαρτήσεων του `.deb`
> προέρχεται από αυτά με τα οποία συνδέεται πραγματικά το backend του Chloros, και αυτό περιλαμβάνει ένα
> `libc6` νεότερο από το glibc 2.31 που διατίθενται στις εκδόσεις «focal» και «bullseye», οπότε το `apt` αρνείται την
> εγκατάσταση εξ αρχής, αντί να επιτρέψει την αποτυχία της αργότερα κατά την εκτέλεση.

### Εγκατάσταση

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
```

{% hint style="info" %}
**Χρησιμοποιήστε το `apt install ./…`, όχι το `dpkg -i`.** Το `apt` επιλύει τις εξαρτήσεις του πακέτου και εγκαθιστά επίσης ό,τι προτείνει: τις απαιτήσεις του Python SDK, `python3-numpy` και `python3-requests`. Το `dpkg -i` δεν εγκαθιστά κανένα από τα δύο. Το CLI και το backend είναι αυτόνομα και λειτουργούν χωρίς αυτά, αλλά το `import chloros_sdk` τα χρειάζεται· η έξοδος της εγκατάστασης αναφέρει ό,τι λείπει. Διατηρήστε το `./`, ώστε το `apt` να εγκαταστήσει το αρχείο αντί να αναζητήσει ένα πακέτο με αυτό το όνομα. Το πακέτο δηλώνει επίσης το `libgl1`, το οποίο φορτώνει το ενσωματωμένο OpenCV και ενδέχεται να λείπει από μια ελάχιστη εικόνα διακομιστή.
{% endhint %}

Επαληθεύστε την εγκατάσταση:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo apt install ./chloros_1.3.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block ("chloros_sdk 1.3.0 for python3" and the "In a virtualenv: pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl" line), any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend@<user>.service" hint -->

***

## Linux arm64 (NVIDIA Jetson)

### Απαιτήσεις συστήματος

| Απαίτηση | Ελάχιστη | Συνιστώμενη |
| --- | --- | --- |
| **Πλατφόρμα** | NVIDIA Jetson με JetPack 6 | Jetson Orin NX 16 GB ή AGX Orin |
| **JetPack** | JetPack 6.x | Τελευταία έκδοση JetPack 6 |
| **Μνήμη (RAM)** | 8GB (κοινή χρήση GPU/CPU) | 16GB+ κοινή χρήση (12GB+ είναι το κατώτατο όριο για παράλληλη επεξεργασία) |
| **Αποθήκευση** | 2GB ελεύθερου χώρου | SSD NVMe με 10GB+ ελεύθερου χώρου |
| **Python** | Python 3.7+ (για το SDK) | Python 3.10+ |

### Εγκατάσταση

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Ίδια διάταξη με το amd64 `.deb` (Python SDK, συμπεριλαμβάνονται buffer λήψης και υπηρεσίες backend), για Jetson Orin / Orin NX / Orin Nano στο **JetPack 6** (L4T R36). Το PyTorch του είναι η έκδοση CUDA του JetPack 6 για το Orin, οπότε η επεξεργασία GPU, συμπεριλαμβανομένου του Texture Aware debayer, εκτελείται στην GPU του Jetson. Το πακέτο δεν περιλαμβάνει το ίδιο το CUDA: χρησιμοποιεί τις βιβλιοθήκες CUDA και cuDNN που εγκαθιστά το JetPack 6 (`sudo apt install nvidia-jetpack` σε μια εικόνα που δεν τις διαθέτει). Όπου δεν είναι διαθέσιμη GPU με CUDA, η επεξεργασία εκτελείται στην CPU και το `--debayer texture-aware` χρησιμοποιεί το Standard debayer (το backend καταγράφει μία γραμμή `[DEBAYER] … needs a CUDA GPU …` που εξηγεί τον λόγο). Για πληροφορίες σχετικά με τη μνήμη, τη θερμική συμπεριφορά και την εφαρμογή στο πεδίο του Jetson, ανατρέξτε στον [Οδηγό NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Εγκατάσταση του Python SDK (Όλα τα Linux)**Το `.deb` εγκαθιστά το SDK για εσάς.** Και τα δύο πακέτα (amd64 και Jetson) το εγκαθιστούν στο σύστημα `python3`, οπότε το `import chloros_sdk` λειτουργεί αμέσως μετά την εγκατάσταση, χωρίς pip και χωρίς σύνδεση στο δίκτυο. Τα πακέτα του (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` και οι φορτωτές έργων) τοποθετούνται στο `/usr/lib/python3/dist-packages`, το οποίο ανήκει στο πακέτο: μια αναβάθμιση τα αντικαθιστά, ενώ η κατάργηση του πακέτου τα αφαιρεί. Η έκδοση αντιστοιχεί πάντα στο CLI και στο backend που έχετε εγκαταστήσει.

Οι απαιτήσεις του ίδιου του SDK, τα `python3-numpy` και `python3-requests`, **συνιστώνται, αλλά δεν είναι υποχρεωτικές** από το πακέτο: το `sudo apt install ./chloros_<version>_<arch>.deb` τα εγκαθιστά, ενώ το `dpkg -i` όχι. Η έξοδος της εγκατάστασης αναφέρει την έκδοση του SDK που εισάγει το `python3`, τυχόν απαιτήσεις που λείπουν, καθώς και οποιοδήποτε `chloros_sdk` που βρίσκεται αλλού στη διαδρομή του `python3` και επικαλύπτει το αντίγραφο του πακέτου (όπως ένα που εγκαταστάθηκε νωρίτερα με το `pip install --user`).**Σε ένα virtualenv**, το οποίο δεν έχει πρόσβαση στα πακέτα του συστήματος, εγκαταστήστε το συνοδευτικό αρχείο wheel σε αυτό (εντελώς εκτός σύνδεσης):

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

**Από το PyPI** — μόνο για υπολογιστή στον οποίο δεν είναι εγκατεστημένο το πακέτο Chloros, του οποίου το SDK οδηγεί ένα backend σε άλλο υπολογιστή (βλ. [Λειτουργία απομακρυσμένου backend](../reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

Προαιρετικά πρόσθετα, για εγκατάσταση μέσω pip (σε virtualenv ή PyPI):

| Πρόσθετο | Εντολή | Προσθέτει |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` για ζωντανή ροή προόδου |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` για μεταφορά μέσω BLE (DAQ-M) |

Επαληθεύστε το SDK:

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Το SDK της Python επικοινωνεί με το backend που εγκαθιστά το `.deb` μέσω ενός τοπικού HTTP API (`http://127.0.0.1:5000`). Το `ChlorosLocal` και τα βοηθητικά προγράμματα σύνδεσης το εκκινούν αυτόματα όταν χρειάζεται· το `connect_all()` ενός αποθηκευμένου έργου δεν το κάνει ποτέ, οπότε σε έναν υπολογιστή χωρίς οθόνη εκτελέστε το backend ως [υπηρεσία](#always-on-ptp-for-headless-hosts). Χρησιμοποιείτε πάντα τη συγκεκριμένη διεύθυνση IPv4 αντί για το `localhost` — το `localhost` μπορεί να αναλυθεί σε `::1` και να κοστίζει περίπου δύο δευτερόλεπτα ανά αίτημα.
{% endhint %}

***

## Αρχική ρύθμιση

### 1. Σύνδεση

Η πρόσβαση στο CLI και στο SDK απαιτεί επίπεδο Chloros+ επί πληρωμή (**Copper** ή υψηλότερο), το οποίο επιβάλλεται από την πλευρά του διακομιστή: ένας χρήστης που έχει αποσυνδεθεί λαμβάνει την διεύθυνση `401 AUTH_REQUIRED`, ενώ ένας χρήστης του δωρεάν επιπέδου (Iron) λαμβάνει την διεύθυνση `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Τα διαπιστευτήρια αποθηκεύονται προσωρινά στο `~/.chloros/user_session.json`.

{% hint style="info" %}
**Οι αναβαθμίσεις διατηρούν τα στοιχεία σύνδεσής σας· η κατάργηση τα διαγράφει.** Η εγκατάσταση μιας νεότερης έκδοσης `.deb` πάνω σε μια υπάρχουσα εγκατάσταση (συμπεριλαμβανομένης της έκδοσης `chloros-cli update`) διατηρεί το `~/.chloros/user_session.json` και την αποθηκευμένη άδεια χρήσης, οπότε οι εργασίες CLI/SDK χωρίς γραφικό περιβάλλον συνεχίζουν να λειτουργούν. Η κατάργηση του πακέτου (`sudo apt remove chloros`) διαγράφει τα αποθηκευμένα στοιχεία σύνδεσης και την προσωρινή μνήμη της άδειας χρήσης για κάθε χρήστη στον υπολογιστή, οπότε μια νέα εγκατάσταση απαιτεί ξανά το `chloros-cli login`.
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
| 1 | **Έκδοση** | Το CLI αναφέρει την έκδοσή του (`v1.3.0`). |
| 2 | **Θύρα διαθέσιμη** | Η θύρα 5000 είναι ελεύθερη, *ή* έχει ήδη απαντηθεί από ένα υγιές backend του Chloros (το οποίο θεωρείται επιτυχία). |
| 3 | **Εκκίνηση backend** | Το εκτελέσιμο του backend ξεκινά. |
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

### Ανά χρήστη Το

Chloros αποθηκεύει τα διαπιστευτήριά του και τη διαμόρφωση της CLI σε έναν ενιαίο κατάλογο για όλες τις πλατφόρμες, τον **`~/.chloros/`** (στα Windows, `%USERPROFILE%\.chloros\`). Δύο προσωρινές μνήμες ειδικές για το Linux ακολουθούν τις συμβάσεις XDG — αυτές χρησιμοποιούν τους καταλόγους `XDG_CONFIG_HOME` / `XDG_CACHE_HOME` όταν έχουν οριστεί.

| Διαδρομή | Σκοπός |
| --- | --- |
| `~/.chloros/user_session.json` | Προσωρινή μνήμη συνεδρίας σύνδεσης που δημιουργείται από το `chloros-cli login` (διατηρείται κατά τις αναβαθμίσεις πακέτων· διαγράφεται όταν αφαιρείται το πακέτο) |
| `~/.chloros/working_directory.txt` | Προεπιλεγμένη παράκαμψη φακέλου έργου (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Προτίμηση γλώσσας CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Ρύθμιση γλώσσας που μοιράζεται με το γραφικό περιβάλλον χρήστη των Windows — μια τιμή `language` εδώ έχει προτεραιότητα έναντι της `cli_language.json` |
| `~/.chloros/update_cache.json` | Προσωρινή μνήμη διάρκειας μίας ώρας για τον έλεγχο ενημερώσεων κατά την εκκίνηση του Linux/Jetson |
| `~/.chloros/backend.log` | Αρχείο καταγραφής του backend όταν αυτό ξεκίνησε από το CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Προσωρινά αποθηκευμένα πακέτα βαθμονόμησης LATTICE ανά κάμερα, ταξινομημένα βάσει σειριακού αριθμού και κατακερματισμού δέσμης |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Προαιρετικές παρακάμψεις χρήστη για προφίλ διόρθωσης ορίου DAQ |
| `~/.config/chloros/system_config.json` | Προφίλ υλικού που έχει αποθηκευτεί στην προσωρινή μνήμη από τη Δυναμική Προσαρμογή Υπολογιστικής Ικανότητας — διαγράψτε το για να επιβάλετε νέα ανίχνευση υλικού |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Αρχεία καταγραφής διακομιστή backend, ένα αρχείο ανά εκκίνηση |
| `~/Documents/MAPIR/Chloros Projects/` | Προεπιλεγμένος φάκελος έργου όταν δεν έχει οριστεί καμία παράκαμψη |

### Σε επίπεδο συστήματος

| Διαδρομή | Σκοπός |
| --- | --- |
| `/usr/bin/chloros-cli` | Σενάριο περιτύλιξης — ορίζει το `LD_LIBRARY_PATH` για τις ενσωματωμένες εγγενείς βιβλιοθήκες (και το `TMPDIR` σε `/mnt/ssd/tmp` όταν υπάρχει αυτός ο φάκελος), και στη συνέχεια εκτελεί το πραγματικό εκτελέσιμο |
| `/usr/bin/chloros-backend` | Σενάριο περιτύλιξης — το ίδιο για το backend, το οποίο επισημαίνει ως εγκατεστημένη έκδοση παραγωγής |
| `/usr/lib/chloros/cli/`, `/usr/lib/chloros/backend/` | Το μεταγλωττισμένο CLI και το backend, εγκατεστημένα χωρίς συμπίεση (νέο στην έκδοση 1.3.0)· τα προγράμματα είναι τα `cli/chloros-cli` και `backend/chloros-backend` |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Σενάρια εκκίνησης δύο γραμμών που εκκινούν τα παραπάνω πραγματικά προγράμματα — διατηρούνται σε αυτές τις διαδρομές επειδή το SDK, το CLI και τα παλαιότερα σενάρια τα αναζητούν εδώ |
| `/usr/lib/chloros/arena_runtime/` | Περιβάλλον εκτέλεσης Arena SDK που απαιτείται από τις κάμερες LATTICE |
| `/usr/lib/chloros/models/*.pth.enc` | Κρυπτογραφημένα μοντέλα αποθορυβοποίησης που χρησιμοποιούνται από το Texture Aware debayer |
| `/usr/lib/python3/dist-packages/chloros_sdk/` (και `lattice_sdk`, `lattice_cam`, `arena_api`, οι φορτωτές έργων) | Το Python SDK, εγκατεστημένο για το σύστημα `python3` και το οποίο ανήκει στο πακέτο |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python SDK wheel που αντιστοιχεί ακριβώς σε αυτή την έκδοση, για ένα virtualenv |
| `/usr/lib/chloros/exiftool` | Ενσωματωμένος εκκινητής του ExifTool (με συμβολικό σύνδεσμο προς το `/usr/local/bin/exiftool` μόνο αν δεν υπάρχει exiftool στο σύστημα). Από την έκδοση 1.3.0 και μετά, το σενάριο ExifTool διατίθεται με τη δική του βιβλιοθήκη Perl στο `/usr/lib/chloros/exiftool-dist/`, οπότε εκτελείται χωρίς το πακέτο `libimage-exiftool-perl` του συστήματος |
| `/etc/chloros/update.conf` | Διαμόρφωση του καναλιού ενημέρωσης που διαβάζεται από το `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Ρυθμίζει το `net.ipv4.ip_unprivileged_port_start = 319` ώστε το backend να μπορεί να δεσμεύσει τις θύρες PTP χωρίς δικαιώματα root |
| `/etc/sysctl.d/60-chloros-gige.conf` | Αυξάνει τα buffer λήψης socket του πυρήνα (`net.core.rmem_max`, `net.core.rmem_default`) στα 32 MB για ροές κάμερας LATTICE και τις εφαρμόζει κατά την εγκατάσταση (μια τιμή που έχει ήδη οριστεί υψηλότερη παραμένει ως έχει). Με την προεπιλογή του πυρήνα, τα πακέτα της κάμερας απορρίπτονται στον πυρήνα και ένας πίνακας χάνει καρέ και ρυθμό καρέ· το `chloros-cli lattice network` αναφέρει τα buffer όταν είναι χαμηλά. Δείτε [Κεντρικοί υπολογιστές Linux — buffer λήψης socket](../reference/cli-reference.md#linux-hosts--socket-receive-buffers) |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Κατευθύνει τον δυναμικό φορτωτή στο `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Παρέχει στον συνδεδεμένο χρήστη πρόσβαση στη σειριακή γέφυρα USB DAQ-U (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend@.service` | Υπηρεσία backend «always-on» με δυνατότητα ενεργοποίησης, η οποία εκτελεί το backend ως τον χρήστη που ορίζετε (εγκατεστημένη, **δεν είναι ενεργοποιημένη**) — αυτή που πρέπει να ενεργοποιήσετε, δείτε [Always-On PTP](#always-on-ptp-for-headless-hosts) |
| `/lib/systemd/system/chloros-backend.service` | Η ίδια υπηρεσία που εκτελεί το backend ως **root**(εγκατεστημένη,**δεν είναι ενεργοποιημένη**), διατηρείται ώστε ένας υπολογιστής που την έχει ενεργοποιήσει να συνεχίζει να λειτουργεί |
| `/var/log/chloros/` | Το λεπτομερές αρχείο καταγραφής της υπηρεσίας root, στο οποίο έχει δικαίωμα ανάγνωσης η ομάδα `adm` (διαγράφεται κατά την πλήρη κατάργηση του πακέτου) |
| `/usr/share/applications/chloros-cli.desktop` | Είσοδος στο μενού εφαρμογών «Chloros CLI» που ανοίγει ένα τερματικό |

**Τείχος προστασίας (νέο στην έκδοση 1.3.0).** Εάν το `ufw` ή το `firewalld` είναι ενεργό κατά την εγκατάσταση, το πακέτο ανοίγει τις θύρες UDP `5002` (δεδομένα DAQ-E), `5353` (ανακάλυψη mDNS), `319` και `320` (PTP), και καταργεί αυτούς τους κανόνες όταν αφαιρεθεί το πακέτο. Εάν υπάρχει εγκατεστημένο τείχος προστασίας αλλά δεν είναι ενεργό, το πακέτο δεν προσθέτει τίποτα και εμφανίζει τις θύρες που πρέπει να επιτραπούν σε περίπτωση που το ενεργοποιήσετε αργότερα. Δεν ανοίγει καμία θύρα TCP: μια ενημέρωση υλικολογισμικού DAQ-E, κατά την οποία ο αισθητήρας επανασυνδέεται με αυτόν τον υπολογιστή μέσω TCP για τη συλλογή της εικόνας, αποτυγχάνει με το `… device never connected back on TCP <port> …`, ενώ ένα ενεργό τείχος προστασίας εμποδίζει αυτή τη σύνδεση.

## Θέση του εκτελέσιμου προγράμματος του backend

Το CLI και το SDK ανιχνεύουν αυτόματα το backend:

| Συστατικό | Διαδρομή |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` (ένα πρόγραμμα εκκίνησης για το `/usr/lib/chloros/backend/chloros-backend`) |

Αντικαταστήστε τη διαδρομή του backend με την επιλογή CLI `--backend-exe` ή την παράμετρο κατασκευαστή SDK `backend_exe`, και τη θύρα με το `--port` (προεπιλογή `5000`).

{% hint style="info" %}
Το `CHLOROS_BACKEND_URL` κατευθύνει τις οικογένειες εντολών **`lattice`**,**`project`**και**`daq pool-*`** σε ένα απομακρυσμένο backend. Οι βασικές εντολές (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) το αγνοούν σκόπιμα και στοχεύουν πάντα στο `http://127.0.0.1:<port>`.
{% endhint %}

***

## Κάμερες LATTICE και αισθητήρες φωτός DAQ στο Linux

Όλες οι οικογένειες εντολών live-hardware λειτουργούν στο Linux (amd64 και Jetson):

* **`chloros-cli lattice`** — ανίχνευση, σύνδεση, διαμόρφωση και καταγραφή από κάμερες LATTICE και συγχρονισμένες συστοιχίες. Η εντολή `.deb` ενσωματώνει το περιβάλλον εκτέλεσης του Arena SDK που απαιτείται και το καταχωρεί στον δυναμικό φορτωτή.
* **`chloros-cli daq pool-*`** — σύνδεση αισθητήρων φωτός DAQ-U/M/E μέσω του backend pool, ροή βαθμονομημένων φασμάτων, καταγραφή αρχείων `.daq` και εναλλαγή προφίλ καλυμμάτων, καθώς και, για ένα DAQ-E, αλλαγή των ρυθμίσεων δικτύου, επανεκκίνηση και ενημέρωση του firmware του. Το `chloros-cli daq discover` αναζητά αισθητήρες που μπορείτε να συνδέσετε, ενώ το `chloros-cli daq --help` παραθέτει τις υποεντολές του `pool-*`.
* **`chloros-cli project`** — εκτελεί ένα αποθηκευμένο έργο (τις κάμερες, τους αισθητήρες και τις ρυθμίσεις επεξεργασίας του) χωρίς οθόνη.
* **`chloros-cli time-sync`** — Ελέγξτε τον «grandmaster» PTP που εκτελεί το backend Chloros για κάμερες LATTICE και αισθητήρες DAQ-E.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

Η εντολή `--sensor-id` απαιτείται από τις εντολές `pool-latest`, `pool-stream`, `pool-record` και `pool-set-cap`· το `pool-list` εμφανίζει τα αναγνωριστικά που βρίσκονται επί του παρόντος στην ομάδα.

{% hint style="info" %}
**Το `--eth-host` παραλείπει την αναζήτηση.** Το `pool-connect --eth` εντοπίζει ένα DAQ-E μέσω mDNS, με εφεδρική λειτουργία ARP που λειτουργεί από μια κρύα κρυφή μνήμη ARP· η διαβίβαση της διεύθυνσης IP ή του ονόματος κεντρικού υπολογιστή του αισθητήρα με το `--eth-host` πραγματοποιεί άμεση σύνδεση με αυτόν.
{% endhint %}

**Τα δικαιώματα σειριακής θύρας DAQ-U** διαχειρίζονται από τον εγκατεστημένο κανόνα udev (`uaccess` + ομάδα `dialout`). Εάν ένας αισθητήρας που ήταν ήδη συνδεδεμένος παραμένει απρόσιτος, φορτώστε εκ νέου τους κανόνες ή συνδέστε τον ξανά:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Δείτε την [αναφορά CLI](../CLI.md) για το πλήρες σύνολο εντολών.

### PTP συνεχούς λειτουργίας για κεντρικούς υπολογιστές χωρίς οθόνη

Το πακέτο εγκαθιστά δύο υπηρεσίες systemd για το backend, καμία από τις οποίες δεν είναι ενεργοποιημένη. Σε ένα Jetson χωρίς οθόνη ή σε έναν διακομιστή που πρέπει να διατηρεί τον συγχρονισμό χρόνου PTP σε συνεχή λειτουργία για αισθητήρες DAQ-E και κάμερες LATTICE, ή όπου σενάρια SDK συνδέουν αποθηκευμένα έργα (το `connect_all()` δεν ξεκινά ποτέ ένα backend), ενεργοποιήστε την υπηρεσία **ως χρήστης σας**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

Το `chloros-backend@<user>.service` εκτελεί το backend ως αυτόν τον χρήστη: οι καταγραφές που εγγράφει ανήκουν σε αυτόν, το λεπτομερές αρχείο καταγραφής του βρίσκεται στο `~/.cache/chloros/logs` του, και χρησιμοποιεί το `chloros-cli login` (`~/.chloros`), οπότε συνδεθείτε πρώτα μία φορά ως αυτός ο χρήστης. Η εγκατάσταση του πακέτου εμφανίζει αυτήν την εντολή με το όνομα χρήστη σας συμπληρωμένο.

Χωρίς υπηρεσία, το backend (και μαζί με αυτό ο PTP grandmaster) εκτελείται μόνο όταν το έχει ξεκινήσει κάτι — μια εντολή `chloros-cli` ή ένα σενάριο SDK.

Η μονάδα συνδέει το backend με τις ρυθμίσεις περιβάλλοντος `127.0.0.1:5000` (ρυθμίσεις περιβάλλοντος `CHLOROS_HOST` / `CHLOROS_PORT` εντός της μονάδας· αντικαταστήστε τις με το `sudo systemctl edit chloros-backend@.service`) και το επανεκκινεί σε περίπτωση αποτυχίας μετά από 5 δευτερόλεπτα.

**Η υπηρεσία root.**Το `chloros-backend.service` εκτελεί το backend ως**root**. Παραμένει εγκατεστημένο, ώστε ένας υπολογιστής που το έχει ενεργοποιήσει να συνεχίζει να λειτουργεί, αλλά ό,τι γράφει ένα backend με δικαιώματα root ανήκει στον root, οπότε ένας χρήστης δεν μπορεί να διαγράψει τις δικές του καταγραφές· το αρχείο καταγραφής του βρίσκεται στο `/var/log/chloros`, το οποίο είναι αναγνώσιμο από την ομάδα `adm`. Για να μεταβείτε στον λογαριασμό χρήστη σας:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

**Το backend μιας υπηρεσίας είναι κοινόχρηστο και δεν διακόπτεται ποτέ από έναν πελάτη.** Τα σενάρια `chloros-cli` και SDK το χρησιμοποιούν όπως το βρίσκουν: το `process` εκτελείται πάνω του, φορτώνοντας το δικό του έργο στη θέση του τελευταίου. Εκεί όπου η CLI, σε διαφορετική περίπτωση, θα επανεκκινούσε το backend (`--restart`, ή ένα `process` του οποίου η εισαγωγή δεν βρήκε τίποτα), ονομάζει τη μονάδα, εμφανίζει το `sudo systemctl restart <unit>` και τερματίζει το `1`, επειδή ένα backend που θα σταματούσε με αυτόν τον τρόπο θα παρέμενε εκτός λειτουργίας. Επανεκκινήστε το μόνοι σας με το `sudo systemctl restart chloros-backend@$USER.service`. Το backend μιας υπηρεσίας δεν σταματά ποτέ από μόνο του όταν βρίσκεται σε κατάσταση αδράνειας, και το `chloros-cli backend stop` το απορρίπτει, αναφέροντας την εντολή `systemctl stop`.**Πώς το PTP αποκτά τις θύρες του.** Το PTP χρησιμοποιεί τις θύρες UDP 319/320, και οι δύο κάτω από το κανονικό κατώτατο όριο των 1024 προνομιακών θυρών. Το `postinst` του πακέτου γράφει το `/etc/sysctl.d/60-chloros-ptp.conf` με το `net.ipv4.ip_unprivileged_port_start = 319`, το οποίο επιτρέπει στο backend να τις δεσμεύσει ενώ εκτελείται ως χρήστης σας. Από την έκδοση 1.3.0 και μετά, το πακέτο δεν ορίζει δυνατότητες αρχείου (`setcap`) στο backend, και το `libcap2-bin` δεν αποτελεί πλέον εξάρτηση: στο αποσυμπιεσμένο backend, μια δυνατότητα αρχείου θα εμπόδιζε την εκκίνησή του, οπότε μην προσθέσετε καμία μόνοι σας.

***

## Παραδείγματα σεναρίων Bash

{% hint style="info" %}
**Κωδικοί εξόδου φιλικοί προς τη δημιουργία σεναρίων.**Το `chloros-cli process` τερματίζει το `0` σε περίπτωση επιτυχίας και**με μη μηδενική τιμή σε περίπτωση αποτυχίας — συμπεριλαμβανομένης μιας εκτέλεσης που ζήτησε προϊόντα εικόνων αλλά δεν έγραψε κανένα** (εμφανίζει το `Processing finished but wrote no image products.` και αναφέρει το όνομα του φακέλου του έργου και τις συνήθεις αιτίες). Οι επιτυχημένες εκτελέσεις αναφέρουν πόσα προϊόντα εικόνας δημιουργήθηκαν (`Image products written: N`, συν το `(plus M clamp masks)` όταν δημιουργήθηκαν μάσκες). Κωδικοί εξόδου: `0` επιτυχία, `1` αποτυχία, `2` σφάλμα παραμέτρου, `130` διακοπή.
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
    --indices NDVI GNDVI OSAVI \
    --debayer texture-aware \
    --no-vignette
```

Οι έγκυρες τιμές `--format` είναι ακριβώς τέσσερις και περιέχουν κενά — να τις τοποθετείτε πάντα σε εισαγωγικά:

| Τιμή `--format` | Φάκελος εξόδου |
| --- | --- |
| `TIFF (16-bit)` *(προεπιλογή)* | `tiff16` |
| `TIFF (8-bit)` | `tiff8` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

Το `--debayer` δέχεται το `standard` (προεπιλογή) ή το `texture-aware` (Chloros+).

### Αυτοματοποιημένη επεξεργασία με Cron

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Παράδειγμα SDK για Python

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "OSAVI"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Αντιμετώπιση προβλημάτων

### Δεν εντοπίστηκε το CLI μετά την εγκατάσταση

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
sudo chmod +x /usr/lib/chloros/chloros-backend /usr/lib/chloros/backend/chloros-backend
```

### Το backend δεν ξεκινά μετά την προσθήκη δυνατοτήτων αρχείων

Μην εκτελέσετε το `setcap` στο backend. Από την έκδοση 1.3.0 και μετά, το πακέτο δεν ορίζει δυνατότητες αρχείων, και οι θύρες 319/320 του PTP προέρχονται από τη ρύθμιση `sysctl` που περιγράφεται στην ενότητα [Πώς το PTP αποκτά τις θύρες του](#always-on-ptp-for-headless-hosts). Μια δυνατότητα αρχείου θέτει το αποσυμπιεσμένο backend σε λειτουργία ασφαλούς εκτέλεσης, στην οποία δεν μπορεί να εντοπίσει τις δικές του βιβλιοθήκες, με αποτέλεσμα να μην ξεκινά. Εάν προσθέσατε κάποια, αφαιρέστε την:

```bash
sudo setcap -r /usr/lib/chloros/backend/chloros-backend
```

### Το PTP δεν ξεκινά / Δεν μπορεί να δεσμεύσει τη θύρα 319

Βεβαιωθείτε ότι το κατώτατο όριο των θυρών χωρίς προνόμια έχει μειωθεί και, αν δεν έχει γίνει, εφαρμόστε το εκ νέου για την τρέχουσα εκκίνηση:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Στη συνέχεια, ελέγξτε το grandmaster:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### Οι εντολές `lattice` αποτυγχάνουν με το μήνυμα «Δεν βρέθηκαν προγράμματα οδήγησης κάμερας LATTICE»

Το backend δεν μπόρεσε να φορτώσει το runtime της κάμερας LATTICE με το οποίο έχει εγκατασταθεί. Κάθε εντολή `lattice` εκτελείται στο backend· το ίδιο το CLI δεν περιέχει προγράμματα οδήγησης κάμερας. Επανατοποθετήστε το Chloros (`sudo apt install --reinstall ./chloros_1.3.0_<arch>.deb`): το `.deb` εγκαθιστά το περιβάλλον εκτέλεσης δίπλα στο backend, στο `/usr/lib/chloros/arena_runtime`, και το καταχωρεί στον δυναμικό φορτωτή. Για να ελέγξετε αυτήν την καταχώριση:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
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

Τα αρχεία καταγραφής του backend για την αποτυχημένη εκκίνηση βρίσκονται στο `~/.cache/chloros/logs/`. Εάν η θύρα 5000 ανήκει σε μια υπηρεσία backend (`systemctl status chloros-backend@$USER.service`), επανεκκινήστε την υπηρεσία αντί να τερματίσετε τη διαδικασία της.

### Δεν εντοπίστηκε CUDA

```bash
# Check NVIDIA driver installation (x86_64)
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check the L4T release (JetPack 6 is R36) and that JetPack's CUDA libraries are installed
cat /etc/nv_tegra_release
sudo apt install nvidia-jetpack
```

Το `chloros-cli selftest` αναφέρει το ίδιο πράγμα σε μία γραμμή: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Λείπουν κοινόχρηστες βιβλιοθήκες

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries (the program itself, not the launcher script)
ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"
```

### Αργή εκκίνηση σε συστήματα με κάρτα SD

Από την έκδοση 1.3.0 και μετά, το backend και το CLI εγκαθίστανται χωρίς συμπίεση, οπότε δεν αποσυμπιέζονται πλέον σε έναν προσωρινό κατάλογο σε κάθε εκκίνηση. Τα προσωρινά αρχεία μεταφέρονται αυτόματα στο `/mnt/ssd/tmp`, εφόσον υπάρχει αυτός ο φάκελος· διαφορετικά, μπορείτε να τα κατευθύνετε σε ένα γρήγορο σύστημα αρχείων:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Ενημέρωση του Chloros σε Linux

Η εντολή `update` είναι διαθέσιμη μόνο για Linux/Jetson. Ελέγχει την έκδοση που έχει δημοσιευτεί στο κανάλι ενημέρωσης που έχει ρυθμιστεί στο `/etc/chloros/update.conf` και προσφέρει τη λήψη και εγκατάσταση της αντίστοιχης έκδοσης `.deb`:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

Στο Linux/Jetson, η CLI εκτελεί επίσης έναν έλεγχο ενημέρωσης χωρίς αναμονή σε κάθε εκκίνηση (το αποτέλεσμα αποθηκεύεται προσωρινά για μία ώρα στο `~/.chloros/update_cache.json`) και εμφανίζει το μήνυμα `Update available: vX.Y.Z` όταν υπάρχει νεότερη έκδοση. Οι ρυθμίσεις, τα έργα και τα στοιχεία σύνδεσής σας διατηρούνται μετά την ενημέρωση.

Η αναβάθμιση του πακέτου διακόπτει κάθε υπηρεσία backend κατά τη διάρκεια της εγκατάστασης, οπότε ένα σενάριο που εκτελείται χάνει τις κάμερες και τους αισθητήρες του, και στη συνέχεια ξεκινά ξανά κάθε μία από αυτές που είναι ενεργοποιημένες (μία που ξεκίνησε χειροκίνητα αλλά δεν ενεργοποιήθηκε ποτέ παραμένει διακοπμένη, όπως μετά από επανεκκίνηση).

## Απεγκατάσταση

```bash
sudo apt remove chloros
```

Η κατάργηση σταματά και απενεργοποιεί όλες τις υπηρεσίες backend, επαναφέρει το προεπιλεγμένο κατώτατο όριο θυρών χωρίς δικαιώματα (1024), αφαιρεί το αρχείο ρυθμίσεων του buffer λήψης (οι τρέχουσες τιμές παραμένουν μέχρι την επανεκκίνηση), τον συμβολικό σύνδεσμο του ενσωματωμένου exiftool, τη διαμόρφωση του φορτωτή Arena, τους κανόνες τείχους προστασίας που πρόσθεσε το πακέτο και το SDK Python που εγκατέστησε, και διαγράφει τα αποθηκευμένα διαπιστευτήρια. Τα έργα σας και τα αρχεία δεδομένων `~/.chloros/` παραμένουν ανέπαφα. Το `sudo apt purge chloros` διαγράφει επίσης το `/var/log/chloros`.

***

## Επόμενα βήματα

* [Οδηγός NVIDIA Jetson](nvidia-jetson-guide.md) — Βελτιστοποίηση και ανάπτυξη ειδικά για το Jetson
* [CLI: Γραμμή εντολών](../CLI.md) — ο οδηγός CLI
* [API: Python SDK](../api-python-sdk.md) — ο οδηγός SDK
* [Αναφορά CLI](../reference/cli-reference.md) και [Αναφορά SDK](../reference/sdk-reference.md) — εξαντλητικές λίστες εντολών/API για την έκδοση 1.3.0
* [Δυναμική προσαρμογή υπολογιστικής ισχύος](../processing-architecture/dynamic-compute-adaptation.md) — πώς το Chloros προσαρμόζεται στο υλικό σας
