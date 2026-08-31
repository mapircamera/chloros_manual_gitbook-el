# Chloros CLI Αναφορά

**Έκδοση:**

1.2.0**Δημιουργία:**29/07/2026 19:19 ·**Αναθεώρηση:** 30/08/2026**Κοινό:** Βελτιστοποιημένο για χρήση από LLM· αναγνώσιμο από τον άνθρωπο.**Πεδίο εφαρμογής:** Κάθε υποεντολή του `chloros-cli` που απευθύνεται στον χρήστη, με επιλογές και παραδείγματα που μπορούν να αντιγραφούν και να επικολληθούν.

Το παρόν έγγραφο αποτελεί την πλήρη αναφορά για το εργαλείο γραμμής εντολών `chloros-cli` που διατίθεται μαζί με το MAPIR Chloros. Είναι σκόπιμα εξαντλητικό, έτσι ώστε ένα LLM (ή άνθρωπος) να μπορεί να συνθέσει οποιαδήποτε υποστηριζόμενη ροή εργασίας από τις παρακάτω λίστες χωρίς να εξετάσει τον πηγαίο κώδικα.

Αν χρειάζεστε μόνο τα βασικά σημεία, μεταβείτε στα εξής:
- [Γρήγορη εκκίνηση σε πέντε λεπτά](#five-minute-quickstart)
- [Ροή εργασίας πρώτης σύνδεσης κάμερας LATTICE](#lattice-camera-first-connect-workflow)
- [Ροή εργασίας πρώτης σύνδεσης αισθητήρα DAQ](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Λειτουργίες καταγραφής, συσκευές εγγραφής και επανεπεξεργασία εκτός σύνδεσης](#capture-modes-recorders--offline-reprocess)

---

## Συμβάσεις

- Όλες οι εντολές έχουν το πρόθεμα `chloros-cli`. Στο Windows το εκτελέσιμο αρχείο είναι το `chloros-cli.exe`· στο Linux /Jetson είναι το `chloros-cli`.
- Τα προαιρετικά ορίσματα εμφανίζονται ως `--flag`. Τα υποχρεωτικά ονομαστικά ορίσματα εμφανίζονται χωρίς αγκύλες.
- Όπου δίνεται μια προεπιλεγμένη τιμή, η παράλειψη της σημαίας χρησιμοποιεί αυτή την τιμή.
- Το CLI είναι ένας «λεπτός» πελάτης HTTP που λειτουργεί πάνω στο backend Chloros (διακομιστής Flask στο `127.0.0.1:5000`). Το backend ξεκινά αυτόματα από τις περισσότερες εντολές. Το `CHLOROS_BACKEND_URL=<url>` κατευθύνει τις οικογένειες εντολών **`lattice`**,**`project`**και**`daq pool-*`** σε ένα απομακρυσμένο backend — οι βασικές εντολές (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) καθορίζουν σκόπιμα το `http://127.0.0.1:<port>` και το αγνοούν (η κυριολεκτική τιμή IPv4 αποφεύγει την ποινή ~2 δευτερολέπτων ανά αίτημα που επιβάλλει το Windows&#x27; `localhost`→`::1`). Βλ. [Μεταβλητές περιβάλλοντος](#environment-variables).
- Απαιτείται σύνδεση με λογαριασμό Chloros+ για όλες τις κλήσεις SDK / CLI (εκτελέστε το `chloros-cli login` μία φορά ανά μηχανή· αποθηκεύεται στην κρυφή μνήμη στο `~/.chloros/`).
- Τα παραδείγματα χρησιμοποιούν διαδρομές Linux· στο Windows αντικαταστήστε το `/home/user/...` με το `C:/Users/.../...`.

---

## Σύνοψη ανώτατου επιπέδου

```
chloros-cli [global options] COMMAND [command options]
```

### Παγκόσμιες επιλογές

| Σημαία | Περιγραφή |
| --- | --- |
| `--backend-exe PATH` | Παράκαμψη του εκτελέσιμου του backend που ανιχνεύεται αυτόματα. |
| `--port N` | Θύρα του backend HTTP (προεπιλογή: `5000`). |
| `-v, --verbose` | Ενεργοποίηση αναλυτικής εξόδου. |
| `--restart` | Αναγκαστική επανεκκίνηση του backend (τερματίζει κάθε `backend_server.py` που εκτελείται). |
| `--version` | Εμφάνιση έκδοσης (`Chloros CLI 1.2.0`). |
| `--help` | Εμφάνιση βοήθειας πρώτου επιπέδου. |

### Ευρετήριο εντολών

| Εντολή | Σκοπός |
| --- | --- |
| [`process`](#chloros-cli-process) | Επεξεργασία ενός φακέλου με καταγραφές Survey3 ή LATTICE από άκρο σε άκρο. |
| [`login`](#chloros-cli-login) | Πραγματοποίηση ελέγχου ταυτότητας αυτού του υπολογιστή με λογαριασμό Chloros+. |
| [`logout`](#chloros-cli-logout) | Εκκαθάριση των αποθηκευμένων διαπιστευτηρίων. |
| [`status`](#chloros-cli-status) | Εμφάνιση της τρέχουσας κατάστασης άδειας χρήσης / πιστοποίησης. |
| [`export-status`](#chloros-cli-export-status) | Προβολή της τρέχουσας προόδου εξαγωγής του Thread-4 κατά τη διάρκεια εκτέλεσης του `process`. |
| [`language`](#chloros-cli-language) | Ρύθμιση ή εμφάνιση λίστας γλωσσών εμφάνισης του CLI (υποστηρίζονται 38 γλώσσες). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#εντολές-φακέλου-έργου) | Προεπιλεγμένος φάκελος έργου (κοινός με το GUI). |
| [`update`](#chloros-cli-update) | Έλεγχος και εγκατάσταση ενημερώσεων του CLI (Linux /Jetson). |
| [`selftest`](#chloros-cli-selftest) | Διαγνωστικά συστήματος + δοκιμές λειτουργίας. |
| [`time-sync`](#chloros-cli-time-sync) | Κατάσταση / έλεγχος PTP grandmaster. |
| [`lattice`](#chloros-cli-lattice) | Έλεγχος &amp; καταγραφή κάμερας LATTICE (45+ υποεντολές). |
| [`daq`](#chloros-cli-daq) | Έλεγχος φασματικού αισθητήρα DAQ (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Άνοιγμα και εκτέλεση αποθηκευμένου έργου «Chloros» (κάμερες + συσκευές DAQ). |

---

## Εγκατάσταση

Το `chloros-cli` περιλαμβάνεται στο πρόγραμμα εγκατάστασης για επιτραπέζιους υπολογιστές «Chloros» σε κάθε υποστηριζόμενη πλατφόρμα — δεν υπάρχει ξεχωριστή λήψη του CLI. Η εγκατάσταση του πακέτου πλατφόρμας προσθέτει το `chloros-cli` στο `PATH` σας, μαζί με την εφαρμογή για επιτραπέζιους υπολογιστές και το δυαδικό αρχείο του backend που αυτή χρησιμοποιεί.

Πρόσφατες λήψεις: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Το πρόγραμμα εγκατάστασης περιλαμβάνει επίσης σενάρια εκκίνησης για ευκολία χρήσης (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`) που ανοίγουν ένα έτοιμο προς χρήση περιβάλλον εργασίας CLI· αυτά περιγράφονται στον τον [Οδηγό Χρήστη του CLI](../CLI.md) και δεν επαναλαμβάνονται εδώ.

### Windows (.exe)

1. Κατεβάστε το πρόγραμμα εγκατάστασης Windows από τη σελίδα λήψης.
2. Εκτελέστε το `Chloros-Setup-x.y.z.exe` και ακολουθήστε τον οδηγό εγκατάστασης. Η προεπιλεγμένη διαδρομή εγκατάστασης είναι το `C:\Program Files\Chloros\` (το αρχείο «CLI» αποθηκεύεται στο `C:\Program Files\Chloros\cli\`, το οποίο το πρόγραμμα εγκατάστασης προσθέτει στο PATH).
3. Ανοίξτε ένα νέο τερματικό (`cmd.exe`, PowerShell ή τερματικό «Windows») ώστε να αναγνωριστεί το ενημερωμένο `PATH`.

```powershell
chloros-cli --version
```

Το πρόγραμμα εγκατάστασης προσθέτει αυτόματα το `chloros-cli.exe` στο `PATH` του συστήματός σας και περιλαμβάνει το περιβάλλον εκτέλεσης Arena SDK που απαιτείται για τις κάμερες LATTICE.

### Linux amd64 (.deb)

Για σταθμούς εργασίας Ubuntu 22.04 LTS ή νεότερης έκδοσης / βασισμένους στο Debian με αρχιτεκτονική x86_64.

> **Το Ubuntu 20.04 δεν υποστηρίζεται.** Ο κατάλογος εξαρτήσεων του πακέτου προέρχεται από
> αυτό με το οποίο συνδέεται πραγματικά το backend, και αυτό περιλαμβάνει το `libc6 (>= 2.34)`·
> το focal διαθέτει το glibc 2.31. Το `apt` αρνείται την εγκατάσταση αντί να επιτρέψει την αποτυχία της κατά τη
> διάρκεια της εκτέλεσης.

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
```

Το αρχείο .deb εγκαθιστά:
- Το `chloros-cli` έως το `/usr/bin/chloros-cli`
- Το μεταγλωττισμένο backend έως το `/usr/lib/chloros/chloros-backend`
- Το περιβάλλον εκτέλεσης Arena SDK (για κάμερες LATTICE)
- Μοντέλα αποθορυβοποίησης, πακέτα βαθμονόμησης και διαμόρφωση καναλιού ενημέρωσης

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Ίδια διάταξη με το .deb για amd64, με μια έκδοση CUDA βελτιστοποιημένη για Jetson Orin / Orin NX / Orin Nano.

### Αυθεντικοποίηση μία φορά ανά μηχανή

Κάθε πλατφόρμα απαιτεί μια εφάπαξ σύνδεση μέσω Chloros+ πριν λειτουργήσουν οι κλήσεις SDK / CLI:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Τα διαπιστευτήρια αποθηκεύονται προσωρινά στο `~/.chloros/user_session.json`.

### Επαλήθευση της εγκατάστασης

```bash
chloros-cli --version           # prints "Chloros CLI 1.2.0"
chloros-cli selftest            # full 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

> **Απαιτείται συνδρομή στο Chloros+.**Το CLI απαιτεί ενεργό πρόγραμμα Chloros+. Το**Copper**είναι το βασικό επίπεδο Chloros+ — κάθε επίπεδο Chloros+ επί πληρωμή παρέχει πρόσβαση στο CLI / SDK· μόνο το δωρεάν επίπεδο**Iron** δεν παρέχει πρόσβαση. (Αντιστοίχιση αναγνωριστικών προγραμμάτων: `0`=Iron/free, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Αναβάθμιση στο [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Αυτό το κατώτατο όριο επιβάλλεται από το backend, όχι μόνο από το CLI: ένα αίτημα με την ένδειξη SDK / CLI -χωρίς συνδρομή απορρίπτεται με κωδικό σφάλματος `403 PLAN_UPGRADE_REQUIRED`, ανεξάρτητα από το αν προέρχεται από το `chloros-cli`, το Python SDK ή έναν χειροκίνητα δημιουργημένο πελάτη HTTP. Ένας χρήστης που έχει αποσυνδεθεί λαμβάνει αντ&#x27; αυτού το σφάλμα `401 AUTH_REQUIRED`. Η πρόσβαση λειτουργεί εκτός σύνδεσης κατά τη διάρκεια της περιόδου χάριτος του πακέτου (30 ημέρες για τα μηνιαία πακέτα, έως τη λήξη για τα ετήσια) και διακόπτεται όταν αυτή λήξει· Το `chloros-cli status` συνεχίζει να λειτουργεί, οπότε ο λόγος είναι ορατός (είναι η μοναδική διαδρομή SDK / CLI που εξαιρείται από τον έλεγχο επιπέδου — `GET /api/license-status`).

---

## Γρήγορη εκκίνηση σε πέντε λεπτά

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI NDRE GNDVI

# 3. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 4. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 5. Read a spectrum from a connected DAQ-U
chloros-cli daq pool-connect --port COM3
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

---

## `chloros-cli process`

Επεξεργασία ενός φακέλου εικόνων μέσω της πλήρους ροής εργασίας «Chloros» (ανίχνευση στόχου → βαθμονόμηση → βινιέτα → ανακλαστικότητα → εξαγωγή δείκτη).

### Σύνοψη

```
chloros-cli process INPUT [OPTIONS]
```

### Θεσιακά ορίσματα

| Όρισμα | Περιγραφή |
| --- | --- |
| `INPUT` | Διαδρομή προς τον φάκελο εισόδου που περιέχει αρχεία `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE) ή `.dng`. |

### Κοινές επιλογές

| Παράμετρος | Προεπιλογή | Περιγραφή |
| --- | --- | --- |
| `-o, --output PATH` | ένας νέος φάκελος με χρονική σήμανση κάτω από την προεπιλεγμένη διαδρομή του έργου σας (`~/Chloros Projects`, εκτός αν έχει ρυθμιστεί διαφορετικά) | Φάκελος έργου προς δημιουργία ή επαναχρησιμοποίηση. Εάν ο φάκελος περιέχει ήδη ένα αρχείο `project.json`, δημιουργείται ένα αρχείο `_1`/`_2` αντί να γίνει αντικατάσταση. |
| `-n, --project-name NAME` | αυτόματο (χρονοσήμανση) | Όνομα έργου. |
| `--debayer {standard,texture-aware}` | `standard` | Το Το `texture-aware` χρησιμοποιεί έναν νευρωνικό απο-Bayer (Chloros+); πιο αργό αλλά υψηλότερης ποιότητας. |
| `--vignette / --no-vignette` | `--vignette` | Διόρθωση βινιέτας. |
| `--reflectance / --no-reflectance` | `--reflectance` | Βαθμονόμηση ανακλαστικότητας (χρησιμοποιεί στόχο πάνελ αν εντοπιστεί, βαθμονόμηση NIST ανά σειριακό αριθμό για το LATTICE). Για το πολυφασματικό LATTICE, αυτό λειτουργεί επίσης ως διακόπτης **προϊόντος** ανακλαστικότητας — βλ. [Διακόπτες εξαγωγής ανά προϊόν](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | απενεργοποιημένο | Εφαρμογή διορθώσεων PPK GNSS από αρχεία sidecar. |
| `--exposure-pin-1 MODEL` | απενεργοποιημένο | Στερέωση του μοντέλου «pin-1» ενός συστήματος διπλής κάμερας. |
| `--exposure-pin-2 MODEL` | απενεργοποιημένο | Στερεώστε το μοντέλο «pin-2». |
| `--recal-interval SECONDS` | 0 | Επιβολή επανεκτέλεσης των μαθηματικών υπολογισμών βαθμονόμησης κάθε N δευτερόλεπτα του χρόνου καταγραφής. |
| `--timezone-offset HOURS` | τοπική | Παράκαμψη της μετατόπισης ζώνης ώρας που έχει ενσωματωθεί στα μεταδεδομένα εξόδου. |
| `--format FORMAT` | `TIFF (16-bit)` | Ένα από τα `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | κανένα | Δείκτες βλάστησης (`NDVI`, `NDRE`, `GNDVI`, `EVI`, `SAVI`, `OSAVI`, `CIG`, …). |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Επιβολή του σημείου εισόδου του αγωγού για αρχεία LATTICE TIFF (το Survey3 .raw δεν επηρεάζεται). Επίσης, η «εξώδρομος διαφυγής» που επιτρέπει την επεξεργασία μιας λήψης **χωρίς raw** — δείτε [Πώς μοιάζει ένας φάκελος καταγραφών](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | ενεργοποιημένο | Εκπέμπει το γραμμικό προϊόν χωρίς bayering (`Debayered_Images`). Βλέπε [Διακόπτες εξαγωγής ανά προϊόν](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | ενεργοποιημένο | Εκπέμπει την προεπισκόπηση οθόνης (`Preview_Images`): RGB = ισορροπία λευκού (DAQ-illuminant όταν είναι διαθέσιμη, αλλιώς gray-world) + γάμμα; multispec = επέκταση ψευδοχρωμάτων. |
| `--radiance / --no-radiance` | ενεργοποιημένο | Εκπέμπει ακτινοβολία τύπου float32 (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Αναφορά για το προϊόν ανακλαστικότητας LATTICE: `auto` = ο στόχος εντός πλαισίου που έχει περάσει τον έλεγχο ποιότητας (QA) αποτελεί την απόλυτη αναφορά, DAQ-κατερχόμενη ακτινοβολία (ρ = π·L/E) ως εναλλακτική λύση· `target` = αυστηρή (χωρίς υποκατάσταση DAQ)· `daq` = με DAQ ως έγκυρη πηγή. Βλ. [Επιλογές εξαγωγής ανά προϊόν](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | κανένα | Κατάλογος σαρώσεων **μετρημένης** ανακλαστικότητας στόχου ανά μονάδα (`<serial>.csv`)· σε περίπτωση αποτυχίας, χρησιμοποιούνται τα ονομαστικά φάσματα T3/T4P. |
| `--array-alignment / --no-array-alignment` | ενεργοποιημένο | Πίνακες LATTICE: εφαρμόζει την ευθυγράμμιση μεταξύ ενοτήτων που έχει αποτυπωθεί σε κάθε λήψη`Chloros:Alignment*` XMP σε κάθε επεξεργασμένο προϊόν (απο-bayering / προεπισκόπηση / ακτινοβολία / ανακλαστικότητα / δείκτης). Καμία ενέργεια για εικόνες χωρίς τις ετικέτες. |
| `--array-alignment-crop / --no-array-alignment-crop` | περικοπή | Περικοπή των ευθυγραμμισμένων εξαγωγών στην περιοχή κοινής επικάλυψης του πλέγματος, ώστε όλα τα modules να μοιράζονται ένα αποτύπωμα· το `--no-…` διατηρεί τον πλήρη καμβά του αισθητήρα (μαύρο γέμισμα έξω από την πηγή). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Επαναδειγματοληψία για τη στρέβλωση ευθυγράμμισης. Το `nearest` διατηρεί τα ακριβή DN της πηγής (χωρίς ανάμειξη ραδιομετρικών τιμών μεταξύ εικονοστοιχείων). |

### Επιλογές ανίχνευσης στόχου

| Σημαία | Περιγραφή |
| --- | --- |
| `--min-target-size PIXELS` | Ελάχιστο μέγεθος πλαισίου-στόχου (px) για τον ανιχνευτή. |
| `--target-clustering 0-100` | Ευαισθησία ομαδοποίησης. |
| `--target / --targets` | Αντιμετώπιση του φακέλου εισόδου ως αποκλειστικά πάνελ-στόχου (παράλειψη ανίχνευσης έρευνας). |

### Παραδείγματα

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI NDRE GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a custom output location with a project name
chloros-cli process "C:/input" -o "C:/output" -n "Field_A_2026-05-26"
```

### Επιλογές εξαγωγής ανά προϊόν (LATTICE πολυφασματικό)

Η επεξεργασία LATTICE διαχωρίζεται σε **κάθε εφαρμόσιμο προϊόν σε μία μόνο διέλευση**. Οι τέσσερις επιλογές ανά τύπο — `--debayered`, `--preview`, `--radiance`, `--reflectance` — είναι όλες**ενεργοποιημένες από προεπιλογή**· χρησιμοποιήστε τη φόρμα `--no-<type>` για να απενεργοποιήσετε μία. RGB οι κύριες κάμερες εκπέμπουν πάντα μόνο δεδομένα μετά την απο-Bayering + προεπισκόπηση (όχι ακτινοβολία/ανακλαστικότητα ανά ζώνη), οπότε τα `--radiance`/`--reflectance` είναι αδρανείς λειτουργίες για αυτές. Οι διακόπτες αγνοούνται για τα Survey3 `.raw` (που ακολουθεί την τυπική διαδρομή ανακλαστικότητας/στόχου). *(Η παλιά σημαία `--radiometric-output {reflectance,radiance,sensor-response}` **καταργήθηκε** και αντικαταστάθηκε από αυτές τις επιλογές· δεν υπάρχει πλέον επίπεδο `sensor-response`.)*

| Προϊόν | Έξοδος | Απαιτείται DAQ downwelling; |
| --- | --- | --- |
| `--debayered` | Γραμμική αποσυμπίεση (`Debayered_Images`). | Όχι. |
| `--preview` | Προεπισκόπηση οθόνης (`Preview_Images`): RGB = WB + γάμμα; πολυφασματική = επέκταση ψευδοχρωμάτων. | Όχι. |
| `--radiance` | float32 W/m²/sr/nm από την πλήρη ραδιομετρική αλυσίδα (`Radiance_Images`). | Αρ. |
| `--reflectance` | uint16 ανακλαστικότητα ρ (`32768` = 1,0), Pix4D-ready. | **Ναι**, εκτός αν το αγκυρώνει ένας στόχος εντός πλαισίου που έχει περάσει τον έλεγχο ποιότητας (βλ. παρακάτω). |

Το `--reflectance-source` επιλέγει την αναφορά ανακλαστικότητας: το **`auto`**(προεπιλογή) καθιστά έναν στόχο εντός πλαισίου που έχει περάσει τον έλεγχο ποιότητας (QA) ως**απόλυτη αναφορά**— οι αλυσίδες εμπειρικών γραμμών που αγκυρώνονται σε στόχους αξιολογούνται διασταυρωμένα στακαι εφαρμόζεται ο νικητής της μέτρησης — με επιστροφή στο όριο κατερχόμενης ακτινοβολίας του DAQ (ρ = π·L/E) όταν δεν υπάρχει στόχος ή αποτυγχάνει ο έλεγχος ποιότητας·**`target`**είναι αυστηρό (χωρίς υποκατάσταση από το DAQ)·**`daq`**επιλέγει τη συμπεριφορά με βάση το DAQ. Η γεωμετρία του στόχου (ArUco / σταθερή περιοχή ενδιαφέροντος / λωρίδα) προέρχεται από τη διαμόρφωση στόχου του έργου· Το `--target-reflectance-dir DIR` διατηρεί**μετρημένες** σαρώσεις ανά μονάδα (`<serial>.csv`) που αναζητούνται με βάση τον σειριακό αριθμό/QR της μονάδας-στόχου, με τα ονομαστικά φάσματα T3/T4P ως εφεδρικά.

Η διαδρομή ανακλαστικότητας του DAQ προσδιορίζει αυτόματα την **κατεύθυνση προς τα κάτω με αντιστοίχιση χρονικής σήμανσης**από ένα καταγεγραμμένο**`.daq`**(DAQ-U/M/E)**ή ενός εγγενούς `.csv` του DAQ-M**που βρίσκεται μαζί με τις εικόνες. Εάν ένα πακέτο βαθμονόμησης ανά κάμερα ή DAQ δεν έχει αποθηκευτεί τοπικά στην προσωρινή μνήμη, η ροή εργασιών**το ανακτά αυτόματατην ανακτά από το AWS** κατά την πρώτη χρήση (απαιτείται σύνδεση στο διαδίκτυο μία φορά· αποθηκεύεται στην προσωρινή μνήμη ως `~/.chloros/`).

#### Ανάγνωση εικονοστοιχείων ανακλαστικότητας (Pix4D / Metashape / δικά σας σενάρια)

Η ανακλαστικότητα αποθηκεύεται ως ακέραιος αριθμός DN, και **ο αριθμός DN που αντιστοιχεί σε ρ = 1,0 εξαρτάται από την κάμερα προέλευσης**:

| Πηγή | ρ = 1,0 αντιστοιχεί σε | Πώς να το διαπιστώσετε |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (περιθώριο έως ρ 2,0) | Η ετικέτα XMP `Chloros:PixelScale=32768` είναι αποτυπωμένη στο αρχείο. |
| Survey3 | `65535` (περικομμένο στο ρ 1,0) | Δεν υπάρχουν ετικέτες XMP — αυτή η απουσία *είναι* το σήμα. |

**Διαβάστε το `Chloros:PixelScale` και διαιρέστε με αυτό** αντί να υποθέσετε μια σταθερά. Η ετικέτα ορίζεται στον τομέα uint16, οπότε παραμένει `32768` σε όλες τις μορφές εξόδου που αναπροσαρμόζουν την κλίμακα — `TIFF (16-bit)`, `PNG (8-bit)`, `JPG (8-bit)` και `TIFF (32-bit, Percent)` είναι όλες αυτοπεριγραφικές (κανονικοποιήστε πρώτα τον αποθηκευμένο τύπο δεδομένων πίσω σε uint16: ×257 από 8-bit, ×65535 από float).

> **Μία περίπτωση δεν φέρει κλίμακα, εκ σχεδιασμού.** Όταν μια καταγραφή πηγής 8-bit (BayerRG8) εγγράφεται ως 8-bit TIFF, ο αγωγός *περικόπτει* την τιμή στο εύρος 0..255 αντί να την αναπροσαρμόσει, οπότε κάθε τιμή άνω του ρ≈0,008 ισοπεδώνεται στο 255 και καμία κλίμακα δεν περιγράφει το αρχείο. Το Chloros παραλείπει σκόπιμα τόσο το `Chloros:PixelScale` και το ζεύγος `MicaSense:RadiometricCalibration` που υπάρχει εκεί, και καταγράφει τον λόγο. **Εάν η ετικέτα απουσιάζει από ένα αρχείο ανακλαστικότητας LATTICE, μην υποθέσετε την ύπαρξη κλίμακας — επανεξάγετε σε 16-bit ή 32-bit** αντί να διαιρείτε εικονοστοιχεία που δεν ήταν ποτέ διαιρέσιμα.

#### EXIF που μεταφέρεται στην εξαγωγή

Το `process` αντιγράφει το **μπλοκ GPS και το ExifIFD** της αρχικής λήψης σε κάθε προϊόν, οπότε μια
εξαγωγή περιλαμβάνει τα `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` και
`CameraSerialNumber` μαζί με τη γεωαναφορά.

**Το `FocalLength` δεν είναι προαιρετικό για τη φωτογραμμετρίαμετρία.** Το Pix4D υπολογίζει την απόσταση δειγματοληψίας εδάφους από
το εστιακό μήκος συν το υψόμετρο· όταν η ετικέτα απουσιάζει, καταλήγει σε μια εντελώς λανθασμένη κλίμακα. Σε μία
πτήση πορτοκαλί, η ελλείπουσα ετικέτα μετέτρεψε μια περιοχή 411 m × 160 m σε μια ανακατασκευασμένη
47,8 km × 13 km — μια ορθοφωτογραφία 455 MP με κυρίως «nodata», η οποία στη συνέχεια θεωρήθηκε ως πρόβλημα πλακιδίων και
πρόβλημα BigTIFF πριν καν ελέγξει κανείς το GSD. Εάν η ορθοφωτογραφία σας εμφανίζεται σε μια απίθανη
κλίμακα, εκτελέστε πρώτα το `exiftool -FocalLength` στο εξαγόμενο προϊόν.

Το αντίγραφο **δεν** είναι σκόπιμα το `-all:all`: οι δομικές ετικέτες του IFD0 διαταράσσουν την έξοδο του LATTICE όταν
αντιγραφούν, ενώ τα `ExifImageWidth` / `ExifImageHeight` εξαιρούνται επειδή περιγράφουν τη
*πρωτότυπη* λήψη — μια εξαγωγή που έχει υποστεί αλλαγή μεγέθους θα περιείχε διαφορετικές διαστάσεις
που θα έρχονταν σε αντίθεση με το δικό της ράστερ. Το XMP εγγράφεται απευθείας αντί να αντιγράφεται, επειδή το ExifTool
απορρίπτει τις ετικέτες XMP της ίδιας κλήσης όταν αντιγράφεται το μπλοκ XMP (κάτι που θα απέβαλε τις ετικέτες βαθμονόμησης MAPIR
).

### Πού αποθηκεύονται τα αποτελέσματα

Τα αρχεία αποθηκεύονται **στον φάκελο του έργου, ομαδοποιημένα ανά κάμερα και στη συνέχεια ανά μορφή αρχείου**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Ο φάκελος της κάμερας είναι `LATT-<sensor>-<lens>-F<filter>` για το LATTICE (που αντιστοιχεί στο EXIF της λήψης
`Model`) και `<model>_<filter>` για το Survey3 — δύο κάμερες που μοιράζονται τον ίδιο αισθητήρα και το ίδιο φίλτρο, αλλά διαφέρουν
στον φακό, διατηρούν ξεχωριστές δομές, επειδή η βινιέτα, το οπτικό πεδίο και η παραμόρφωση διαφέρουν. Ο φάκελος μορφής
ακολουθεί το `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` ή `tiff32` για το
`TIFF (32-bit, Percent)`.

> **Κάθε εξαγόμενο προϊόν διατηρεί το όνομα του αρχείου ΠΗΓΗΣ.** Μια εξαγωγή radiance του
> `capture_…_raw.tif` εξακολουθεί να ονομάζεται `capture_…_raw.tif` — απλώς βρίσκεται στο
> `tiff32/Radiance_Images/`. **Ο φάκελος προσδιορίζει το προϊόν, όχι το όνομα του αρχείου**, οπότε η αναζήτηση με
> `*radiance*.tif` δεν βρίσκει τίποτα· αντί αυτού, κάντε αντιστοίχιση με βάση τον κατάλογο.

### Καταγραφές αισθητήρα φωτός — βαθμονομημένες `.daq` + `.csv`

Το `process` διαχειρίζεται επίσης τις καταγραφές `.daq` στον φάκελο εισόδου , και **δεν**
χρειάζεται καμία εικόνα για να το κάνει: ένα DAQ-U / DAQ-M / DAQ-E που πετάει μόνο του αποτελεί πλήρη
καταγραφή, και ένας φάκελος που περιέχει μόνο αρχεία `.daq` είναι έγκυρη είσοδος.

Ένα DAQ μπορεί να καταγραφεί **χωρίς** τη βαθμονόμησή του — αυτό ακριβώς κάνουν από προεπιλογή οι δημόσιες
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`) κάνουν από προεπιλογή: καταγράφουν τις ακατέργαστες μετρήσεις του αισθητήρα και προσθέτουν σφραγίδα στο αρχείο, ώστε
το Chloros να ανακτά τη βαθμονόμηση εργοστασίου αυτού του αισθητήρα **μέσω σειριακής** (πρώτα από την τοπική κρυφή μνήμη,
μετά από το Cloud του MAPIR) και την εφαρμόζει. Το `process` εγγράφει ξανά το αποτέλεσμα:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

Το `.csv` περιέχει μία σειρά ανά μέτρηση: χρονική σήμανση UTC, χρόνος ολοκλήρωσης, συνολική ισχύς,
φωτοπικά/σκοτοπικά lux, PPFD (και η κατανομή του σε μπλε/πράσινο/κόκκινο), μέγιστο μήκος κύματος, και στη συνέχεια το
πλήρες φάσμα στο δικό του πλέγμα μηκών κύματος του αισθητήρα. Το `.daq` εισάγεται εκ νέου χωρίς να
υποβληθεί σε δεύτερη βαθμονόμηση.

Σε περίπτωση επιτυχίας, η εκτέλεση αναφέρει το `Light-sensor products written: N (calibrated .daq + .csv)`.
Η παρένθεση περιγράφει τι γράφτηκε στην πραγματικότητα, οπότε διαβάζεται ως
`(RAW COUNTS — this sensor has no calibration bundle)` για έναν αισθητήρα χωρίς δέσμη και
`(N calibrated, M raw counts)` για έναν φάκελο που περιέχει και τα δύο. Οι δικές του επικεφαλίδες του backend
`[DAQ-EXPORT]` και `[RUN-SUMMARY]` αντλούν τη διατύπωσή τους με τον ίδιο τρόπο — καμία από
τις τρεις δεν μπορεί να χαρακτηρίσει μια ακατέργαστη εξαγωγή ως βαθμονομημένη.

Μια καταγραφή DAQ-U / DAQ-M / DAQ-E της οποίας το πακέτο βαθμονόμησης δεν μπορεί να ανακτηθεί — είστε
εκτός σύνδεσης ή ο συγκεκριμένος αισθητήρας δεν έχει καταχωρημένη βαθμονόμηση — **παραλείπεται με αιτιολογία** σε μια
γραμμή `[DAQ-EXPORT]`, και ποτέ δεν καταγράφεται ως αρχείο «βαθμονομημένο» που περιέχει ακατέργαστες μετρήσεις.
Συνδεθείτε στο διαδίκτυο και εκτελέστε ξανά την διαδικασία. Ο λόγος είναι αυτός που ο αναγνώστης
πραγματικά διαπίστωσε για το συγκεκριμένο αρχείο (μη αναγνώσιμο σχήμα, απουσία δέσμης, σφάλμα εγγραφής), και η σύνοψη
της εκτέλεσης παραθέτει **ξεχωριστούς** λόγους — είκοσι αρχεία που παραλείφθηκαν για μία αιτία εμφανίζονται ως μία
αιτία, όχι ως είκοσι επαναλήψεις της ίδιας αιτίας.

#### Οι καταγραφές DAQ-A εξάγονται ως ακατέργαστες μετρήσεις

Η οικογένεια **DAQ-A** προηγείται του συστήματος πακέτων ανά σειριακό αριθμό και δεν διαθέτει πακέτο βαθμονόμησης
για λήψη — αντίθετα, βαθμονομείται στο πεδίο έναντι ενός στόχου ανακλαστικότητας, γι&#x27; αυτό
και δεν χρειάστηκε ποτέ ένα τέτοιο πακέτο. Η απόρριψη αυτών των καταγραφών δεν τους άφησε κανένα τρόπο να
εξάγουν καθόλου τα δεδομένα τους, οπότε εξάγονται με **διαφορετικό όνομα**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Ένα διαφορετικό όνομα αρχείου αντί για μια σημαία μέσα στο αρχείο, επειδή η δήλωση πρέπει να παραμείνει
ακέραιη κατά την αποστολή μέσω email ως απλό όνομα. Η κεφαλίδα `.csv` δηλώνει
`raw spectral sensor counts (NOT irradiance)` και προειδοποιεί ότι οι τιμές είναι συγκρίσιμες
**εντός** του αρχείου — που είναι ακριβώς ο λόγος για τον οποίο τις χρησιμοποιεί η βαθμονόμηση βάσει στόχου — και
όχι μεταξύ αισθητήρων. Οι φωτομετρικές στήλες που εξαρτώνται από την ισχύ (συνολική ισχύς, φωτοπικά και
σκοτοπικά lux, PPFD) αναγράφονται ως **NULL** αντί να υπολογίζονται από τις μετρήσεις, και η
σύνοψη της εκτέλεσης αναφέρει `RAW COUNTS`, οπότε τα δεδομένα που «εξάγονται» σε αρχείο καταγραφής δεν μπορούν να ερμηνευθούν ως ακτινοβολία.

Οι παλαιότερες καταγραφές **v1.01 / v1.02** (που δημιουργούνται από ένα DAQ-A-SD) δεν φέρουν εποχή ανά ανάγνωση,
αλλά μόνο την ώρα εγγραφής του αρχείου. Το πρόγραμμα αντιστοίχισης εικόνας↔κατευθυνόμενης ακτινοβολίας εξακολουθεί να τις απορρίπτει — η αντιστοίχιση ενός
πλαισίου με μια ώρα εγγραφής θα ήταν λανθασμένη χωρίς να φαίνεται — αλλά το πρόγραμμα εξαγωγής τα διαβάζει, και
το CSV εκτυπώνει το `clock=daq_created_on`, οπότε το προϊόν δηλώνει σε ποιο ρολόι βρίσκεται.

### Σημειώσεις

- Το `process` ανιχνεύει αυτόματα αν ο φάκελός σας είναι τύπου «Survey3», «LATTICE» ή μικτός.
- Η πρόοδος μεταδίδεται μέσω Server-Sent Events· η διεύθυνση CLI εμφανίζει ζωντανή πρόοδο ανά νήμα (Ανίχνευση, Ανάλυση, Επεξεργασία, Εξαγωγή).
- Για τα Linux /Jetson, το CLI ελέγχει τη μνήμη swap και ενδέχεται να εμφανίσει προειδοποίηση πριν από την επεξεργασία μεγάλων φακέλων. Το debayer με αναγνώριση υφής εφαρμόζει επίσης αυτόματα ένα όριο συχνότητας GPU στα Jetson χαμηλής- (Nano, Orin Nano).
- Σε περίπτωση επιτυχίας, η εκτέλεση αναφέρει πόσα προϊόντα εικόνας έγραψε (`Image products written: N`).

#### Μια εκτέλεση που δεν εγγράφει εικόνες αποτυγχάνει

Εάν ζητήσατε προϊόντα και η εκτέλεση δεν εγγράφει **κανένα** — μόνο τα `project.json` και
`calibration_data.json` — το `process` το αντιμετωπίζει ως αποτυχία: εμφανίζει το
`Processing finished but wrote no image products.` και **τερματίζεται με μη μηδενική τιμή**, οπότε ένα σενάριο μπορεί
να το εντοπίσει. Το μήνυμα αναφέρει το φάκελο του έργου και τις συνήθεις αιτίες:

- ο φάκελος εισόδου δεν αναγνωρίστηκε ως λήψη (ελέγξτε τη διάταξη και το `--input-level`), ή
- κάθε ζητηθέν προϊόν παραλείφθηκε ως μη εφαρμόσιμο για αυτές τις κάμερες (π.χ. αίτημα για
  ακτινοβολία/ανακλαστικότητα από κάμερες που υποστηρίζουν μόνο το RGB).

Εκτελέστε ξανά με το `--verbose` και ελέγξτε το αρχείο καταγραφής του backend για τις γραμμές `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
οι οποίες εξηγούν τις παραλείψεις ανά κάμερα που διαφορετικά δεν εμφανίζονται στην έξοδο του CLI.

Μια σκόπιμη εκτέλεση μόνο με μεταδεδομένα — με όλα τα προϊόντα απενεργοποιημένα και χωρίς `--indices` — εξακολουθεί να αποτελεί
**επιτυχία**, καθώς η κενή έξοδος εικόνας είναι το σωστό αποτέλεσμα σε αυτή την περίπτωση.

Το ίδιο ισχύει και για μια **εκτέλεση μόνο με αισθητήρα φωτός**: ένας φάκελος με εγγραφές `.daq` δεν περιέχει εικόνες προς εξαγωγή
εξ ορισμού, και η εκτέλεση αξιολογείται με βάση τα βαθμονομημένα `.daq` / `.csv` που δημιούργησε αντ&#x27; αυτού.

---

## `chloros-cli login`

Πραγματοποιήστε έλεγχο ταυτότητας αυτού του υπολογιστή με έναν λογαριασμό cloud Chloros+. Τα διαπιστευτήρια αποθηκεύονται με ασφάλεια στην προσωρινή μνήμη στο `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Παραδείγματα

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (αφαιρώντας τμήματα από τον κωδικό πρόσβασης ή αντιγράφοντας τμήματα του). Σε περίπτωση σφάλματος 401, το CLI επαναλαμβάνει αυτόματα την προσπάθεια με το `$$` και, στη συνέχεια, με το μισό του κωδικού πρόσβασης χωρίς διπλότυπα· αν η επανάληψη επιτύχει, σας συνδέει και εμφανίζει τη σωστή σύνταξη με απλά εισαγωγικά που πρέπει να χρησιμοποιήσετε την επόμενη φορά.

> **Χρήση χωρίς γραφικό περιβάλλον/μέσω σεναρίου: η απουσία αποθηκευμένης συνεδρίας σημαίνει διαδραστική προτροπή, όχι γρήγορη αποτυχία.** Οποιαδήποτε εντολή που εκκινεί το backend (`process`, `status`, `export-status`, `time-sync`, …) που εκτελείται χωρίς αποθηκευμένη άδεια χρήσης/συνεδρία, οδηγεί σε μια διαδραστική γραμμή εντολών `Email:` / `Password:` στο stdin πριν προχωρήσει. Μια εργασία χωρίς επίβλεψη χωρίς αποθηκευμένη συνεδρία θα κολλήσει επομένως περιμένοντας είσοδο — εκτελέστε το `chloros-cli login EMAIL PASSWORD` μία φορά ανά μηχανή πριν προγραμματίσετε εργασίες χωρίς οθόνη.

---

## `chloros-cli logout`

Εκκαθαρίζει την αποθηκευμένη συνεδρία και επιβάλλει νέα σύνδεση στην επόμενη κλήση.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Εμφανίζει το τρέχον επίπεδο άδειας χρήσης (Iron/Copper/Bronze/Silver/Gold), τον πιστοποιημένο χρήστη και τον αριθμό συνδέσεων συσκευών.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Ελέγχει την τρέχουσα πρόοδο εξαγωγής του Thread-4. Μπορεί να κληθεί με ασφάλεια **κατά τη διάρκεια** της εκτέλεσης του `process` από άλλο shell.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Ρύθμιση της γλώσσας εμφάνισης του «CLI» (υποστηρίζονται 38 γλώσσες, συμπεριλαμβανομένων των CJK, RTL και Indic). Σε παλαιότερες κονσόλες που δεν μπορούν να αποδώσουν το σενάριο, η γλώσσα επιστρέφει ομαλά στα Αγγλικά.

```
chloros-cli language [LANG_CODE] [--list]
```

### Παραδείγματα

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Εντολές φακέλου έργου

Αυτές διαχειρίζονται την προεπιλεγμένη θέση του φακέλου έργου (κοινή με το GUI).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

---

## `chloros-cli update`

Linux/ Μόνο για Jetson. Ελέγχει τα `version_url` από τα `/etc/chloros/update.conf` και προσφέρει τη λήψη + εγκατάσταση του αντίστοιχου `.deb`.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

Στο Linux /Jetson, το CLI εκτελεί επίσης **αυτόματο έλεγχο ενημερώσεων σε κάθε εκκίνηση** (χωρίς να εμποδίζει τη λειτουργία, δεν καθυστερεί ποτέ την εντολή): διαβάζει το `/etc/chloros/update.conf`, αποθηκεύει το αποτέλεσμα στην προσωρινή μνήμη για 1 ώρα στο `~/.chloros/update_cache.json` και εμφανίζει το `Update available: vX.Y.Z / Run: chloros-cli update` όταν υπάρχει νεότερη έκδοση. Το Silπαραλείπεται σε περίπτωση σφάλματος και στο Windows.

---

## `chloros-cli selftest`

Εκτελεί μια δοκιμή λειτουργίας 7 βημάτων: έκδοση, διαθεσιμότητα θύρας, εκκίνηση backend, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), παρουσία μοντέλου αποθορυβοποίησης, ετοιμότητα CUDA+αποθορυβοποίησης.

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

Κατάσταση και έλεγχος του grandmaster PTP. Ο κεντρικός υπολογιστής «Chloros» εκτελεί το grandmaster PTP· οι μονάδες LATTICE cams και DAQ-E λειτουργούν ως υποτελείς μονάδες σε αυτό για χρονικές σφραγίδες μεταξύ συσκευών.

| Υποεντολή | Περιγραφή |
| --- | --- |
| `status` | Εμφάνιση κατάστασης grandmaster, προτεραιοτήτων BMCA, ταυτότητας ρολογιού. |
| `peers` | Λίστα υποτελών που ανιχνεύονται μέσω Delay_Req (κάμερες + αισθητήρες DAQ-E). |
| `cameras` | Κατάσταση PTP ανά κάμερα (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Επανεκκίνηση της διαδικασίας grandmaster. |
| `set-priority --priority1 N --priority2 N` | Παράκαμψη προτεραιοτήτων BMCA. |

### Παραδείγματα

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

Έλεγχος κάμερας LATTICE. Κάθε υποεντολή δρομολογείται μέσω του backend Chloros· το backend διαχειρίζεται την ομάδα καμερών, οπότε οι επόμενες κλήσεις CLI επαναχρησιμοποιούν το ίδιο ανοιχτό χειριστή.

### Κοινές επιλογές (κοινές για τις περισσότερες υποεντολές)

| Σημαία | Περιγραφή |
| --- | --- |
| `-d, --device N` | Δείκτης κάμερας (προεπιλογή: 0). |
| `-s, --serial SN` | Συγκεκριμένος σειριακός αριθμός· παρακάμπτει την επιλογή `--device`. |
| `--serials SN1,SN2,…` | Σειριακοί αριθμοί διαχωρισμένοι με κόμμα για λειτουργία πολλαπλών καμερών. |
| `--all` | Λειτουργία σε κάθε κάμερα που έχει εντοπιστεί. |
| `--exposure US` | Χρόνος έκθεσης σε μικροδευτερόλεπτα. |
| `--gain DB` | Κέρδος σε dB. |
| `--pixel-format FMT` | π.χ. `BayerRG8`, `BayerRG12`. |
| `--width N` / `--height N` | Διαστάσεις εικόνας. |
| `--preset {default,high_quality,high_speed,triggered}` | Εφαρμογή προκαθορισμένης ρύθμισης. Όλα λειτουργούν σε ελεύθερη λειτουργία εκτός από το `triggered`, το οποίο θέτει την κάμερα σε κατάσταση ετοιμότητας για σήμα άκρου υλικού στη Γραμμή 2 — εφόσον δεν υπάρχει τίποτα που να ενεργοποιεί αυτή τη γραμμή, θα περιμένει για πάντα αντί να πραγματοποιήσει λήψη. |
| `-o, --output DIR` | Κατάλογος εξόδου (προεπιλογή: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | Μέγεθος πακέτου GVSP. Το `auto` εκτελεί δοκιμές ICMP+GVSP· `jumbo` = 9000· `standard` = 1500. |

### Ροή εργασιών πρώτης σύνδεσης κάμερας LATTICE

```bash
# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Auto-shrink frame size to fit the wire
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Αναφορά υποεντολών

#### Εντοπισμός &amp; Πληροφορίες

| Υποεντολή | Σκοπός |
| --- | --- |
| `lattice info` | Λίστα συνδεδεμένων καμερών (κατασκευαστής, μοντέλο, σειριακός αριθμός, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Ανάλυση του κεντρικού συστήματος για βέλτιστη διαμόρφωση της κάμερας. Η εντολή `--no-discover` παραλείπει την ανίχνευση καμερών (ταχύτερη ανάλυση, μόνο μέσω κάρτας δικτύου). |
| `lattice network [--fix] [--estimate] [--cameras N]` | Έλεγχος/διόρθωση ρυθμίσεων κάρτας δικτύου· εκτίμηση εύρους ζώνης/FPS. |
| `lattice network-analysis --master SN --slaves SN1,SN2,… [--width N] [--height N] [--pixel-format FMT] [--binning N] [--force-tier TIER] [--backend-url URL] [--json]` | Δυνατότητες δικτύου backend σταθερού σχήματος + σύσταση συστοιχίας (επιστρέφει `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). Το `auto_capped_fps` διατηρεί την απαιτούμενη ανάλυση αλλά περιορίζει τα fps προορισμού — διαβάστε το `recommended.recommended_target_fps` και περάστε το ως στόχο σύνδεσης· θεωρήστε το ως επιτυχία, όχι σφάλμα. |
| `lattice analyze-array [--models M1,M2,…] [--binning N] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--json]` | Τιανάλυση «what-if» χωρίς άνοιγμα καμερών. **Το `--n-active` είναι ο συνολικός αριθμός καμερών στο δίκτυο, όχι μόνο αυτού του πίνακα**— αυξήστε τον όταν αυτόνομες κάμερες μεταδίδουν ταυτόχρονα ή όταν ο προϋπολογισμός του δικτύου υπολογίζεται με βάση μια ζήτηση που τις υποεκτιμά (προεπιλογή: `len(--models)`). Εμφανίζει πάντα τις συγκεντρωτικές γραμμές `Wire budget:` (απαιτούμενα MB/s έναντι ανώτατου ορίου χωρίς συγκρούσεις) και `Max cameras:`, και επισημαίνει το `** OVER-SUBSCRIBED**` όταν το array υπερφορτώνει το δίκτυο — βλ. [Μοντέλο fps &amp; burst του array](#array-fps--burst-model). |
| `lattice gpu` | Εμφάνιση κατάστασης GPU. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Έλεγχος ή ενημέρωση του υλικολογισμικού της κάμερας. Η τοπική επιλογή `.fwa` είναι καρφιτσωμένη: το αρχείο στο `firmware/<MODEL_PREFIX>/` που αντιστοιχεί στο `MIN_FIRMWARE_VERSION` της έκδοσης εγκαθίσταται όταν υπάρχει (μόνο η υψηλότερη έκδοση ως εφεδρική), οπότε μια νεότερη εικόνα προμηθευτή που έχει αποθηκευτεί στο δίσκο παραμένει ανενεργή έως ότου καταργηθεί αυτή η καρφίτσα — οι σκόπιμα νεότερες εκδόσεις φτάνουν στις μονάδες μέσω του υπογεγραμμένου καταλόγου AWS, ο οποίος προτιμάται όταν είναι νεότερος. |
| `lattice presets [--apply NAME]` | Λίστα ή εφαρμογή προεπιλογών κάμερας. |
| `lattice status` | Ζωντανή κατάσταση κάμερας. |

#### Λήψη

| Υποεντολή | Σκοπός |
| --- | --- |
| `lattice capture [--format tiff\|png\|jpg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--force-daq]` | Μονό καρέ. **Αποθηκεύει κάθε τύπο εξαγωγής από προεπιλογή** (`--processing all`)· βλ. [Επίπεδα εξαγωγής καταγραφής](#capture-export-levels-the-all-default). Η εντολή `--levels` αποθηκεύει ένα συγκεκριμένο υποσύνολο (υπερισχύει του `--processing`)· το `--force-daq` εγγράφει την εκχωρημένη ένδειξη DAQ ως `.daq` sidecar ακόμη και σε λήψη μόνο ακατέργαστων δεδομένων. `--jpeg-quality` = JPEG ποιότητα 1–100 (προεπιλογή 95). |
| `lattice continuous [--format tiff\|png\|jpg] [--jpeg-quality N] [--queue-depth N]` | Ροή στο δίσκο μέχρι το Ctrl+C. |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Προβολή. Το `--ae-damping` ρυθμίζει την απόσβεση της αυτόματης έκθεσης (0,4–100). |

#### Ρύθμιση αισθητήρα

| Υποεντολή | Σκοπός |
| --- | --- |
| `lattice configure [--get N1 N2…] [--set N=V N=V…] [--dump] [--json]` | Ανάγνωση/εγγραφή σε οποιονδήποτε κόμβο GenICam. |
| `lattice exposure [--auto] [--auto-once] [--off] [--set US] [--brightness N] [--damping F] [--upper-limit US]` | Έκθεση &amp; AE. |
| `lattice gain [--auto] [--off] [--set DB]` | Κέρδος &amp; αυτόματο κέρδος. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N] [--binning-mode Sum\|Average]` | Περιοχή ενδιαφέροντος (ROI) αισθητήρα &amp; binning. |
| `lattice format [--set FMT] [--list]` | Μορφή εικονοστοιχείου. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Σκανδάλη υλικού/λογισμικού. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (χωρίς σημαίες = WB μίας λήψης) | Λειτουργίες WB. RGB /Μόνο για κάμερες Bayer· δεν εκτελείται (παραλείπεται) σε μονοχρωματικές M3M. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct K] [--get]` | Αλυσίδα επεξεργασίας χρωμάτων «RGB». Το `natural` (προεπιλογή) είναι η οικονομική επεξεργασία σε πραγματικό χρόνο· Το `enhanced` προσθέτει απομάκρυνση περιθωριακών φαινομένων + ζωντάνια + τοπική αντίθεση CLAHE για την πλήρη εμφάνιση «hub-parity» με κόστος επεξεργασίας ανά καρέ περίπου διπλάσιο, οπότε χαμηλότερο **ζωντανό** ρυθμό καρέ — οι αποθηκευμένες λήψεις λαμβάνουν πάντα την πλήρη επεξεργασία σε κάθε περίπτωση. RGB /Μόνο για κάμερες Bayer· παραλείπεται σε μονοχρωματικές M3M. |
| `lattice color [--saturation N] [--contrast N] [--reset] [--get]` | Εμφάνιση κορεσμού/αντίθεσης (κάμερες με φίλτρο RGB). Παραλείπεται σε μονοχρωματικές M3M. |
| `lattice filter [--set NAME] [--list]` | Ρύθμιση του(`RGN-IMX265`, `OCN`, `NGB`, …). |
| `lattice power [--sleep]` | Έλεγχος κόμβων ισχύος/θερμοκρασίας· εναλλαγή σε κατάσταση αναμονής χαμηλής ισχύος. |

#### Βαθμονόμηση &amp; Αισθητήρες

| Υποεντολή | Σκοπός |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Βαθμονόμηση από στόχο ανακλαστικότητας. |
| `lattice dls [--connect] [--spectrum] [--irradiance] [--mac MAC] [--filter NAME] [--json]` | Εντολές για τον ενσωματωμένο αισθητήρα κατερχόμενου φωτός. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Εφαρμογή διόρθωσης βινιέτας σε υπάρχουσες εικόνες. |

#### Πολλαπλές κάμερες (Προσωρινές συνεδρίες)

| Υποεντολή | Σκοπός |
| --- | --- |
| `lattice multi-info` | Εμφάνιση λίστας όλων των καμερών με ρόλους συγχρονισμού. |
| `lattice multi-capture [--format FMT] [--jpeg-quality N] [--processing LEVEL]` | Ένα συγχρονισμένο καρέ από κάθε κάμερα. Αποθηκεύει **όλους τους τύπους εξαγωγής από προεπιλογή**όταν είναι συνδεδεμένος ένας μόνιμος πίνακας· η προσωρινή εναλλακτική λύση χωρίς πίνακα**υποβάλλεται μόνο σε απο-bayering** (εκτελέστε πρώτα το `array-connect` για τα υπόλοιπα). |
| `lattice multi-stream [--fps F] [--count N] [--format FMT] [--jpeg-quality N]` | Ροή συγχρονισμένων καρέ (προσωρινή). |
| `lattice multi-test [--count N]` | Δοκιμή χρονισμού συγχρονισμού GPIO. |
| `lattice multi-detect [--line LINE] [--json]` | Αυτόματη ανίχνευση καλωδίωσης κύριου/δευτερεύοντος GPIO. |

#### Ευθυγράμμιση

| Υποεντολή | Σκοπός |
| --- | --- |
| `lattice align-calibrate [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N]` — συν ρυθμιστικά ανιχνευτή/αντιστοίχισης `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, ρυθμιστικά RANSAC `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, συνδυασμός πολλαπλών καρέ `[--averaging mean\|median\|inlier_weighted]`, γεωμετρικοί περιορισμοί `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, χωρικός περιορισμός `[--roi X0,Y0,X1,Y1] [--mask PATH]` και παρακάμψεις ανά slave `[--per-cam-override SN:KEY=VALUE]` (επαναλαμβανόμενο) | Υπολογισμός προφίλ ευθυγράμμισης από ζωντανές κάμερες. Το `--prefilter` έχει ως προεπιλογή το `gradient` (χάρτης ακμών· ταιριάζει με τον ευθυγραμμιστή GUI/συστοιχίας — οι ακμές διατηρούνται σε όλες τις φασματικές ζώνες). Το `--matcher flann` αποδίδει πάνω από ~5000 χαρακτηριστικά· το `--averaging median` είναι ανθεκτικό σε μία κακή λήψη, το `inlier_weighted` σταθμίζει βάσει του αριθμού αντιστοιχιών· το `--lock-scale` προβάλλει στην πλησιέστερη περιστροφή (χωρίς κλίμακα), το `--lock-axis` μηδενίζει μία συνιστώσα μετάθεσης· το `--mask` εφαρμόζεται σε όλες τις κάμερες (χρησιμοποιήστε το `--per-cam-override` για ρυθμίσεις ανά κάμερα, π.χ. `--per-cam-override 214701292:method=phase`). Το `--rms-threshold-px` αρνείται να αποθηκεύσει μια βαθμονόμηση της οποίας το RMS επαναπροβολής υπερβαίνει το όριο. |
| `lattice align-apply --profile PATH [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N]` | Λήψη ενός ευθυγραμμισμένου πλαισίου πολλαπλών ζωνών. Το `--bit-depth` έχει ως προεπιλογή την προσαρμογή στην κάμερα· το `--no-crop` διατηρεί ολόκληρο το καρέ (συμπληρώνει με μαύρο)· το `--interpolation` (προεπιλογή `linear`) και τα `--border-mode`/`--border-value` (προεπιλογή `constant`/0) ελέγχουν την παραμόρφωση της CPU — η διαδρομή της GPU είναι διγραμμική σε κάθε περίπτωση. |
| `lattice align-stream --profile PATH [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N]` | Πλαίσια πολλαπλών ζωνών ευθυγραμμισμένα με το ρεύμα (ίδια ρυθμιστικά παραμόρφωσης με το `align-apply`). |
| `lattice align-info --profile PATH [--json]` | Εμφάνιση λεπτομερειών προφίλ. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Αλλαγή της σειράς των επιπέδων. |

#### Ευρετήριο / Μαθηματικά βλάστησης

```bash
# Offline: compute NDVI from an aligned multi-band TIFF
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: discover array, calibrate alignment, capture, compute index, in one go
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --save-multiband -o output/
```

Πλήρες σύνολο σημαίας: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI / NDRE / EVI / SAVI / GNDVI /…), `--formula EXPR`, `--channel SYM=BAND` (επαναλαμβανόμενο), `--capture-level raw|debayered|radiance|reflectance|unknown` (παράκαμψη του επιπέδου καταγραφής που έχει καταγραφεί στην πηγή TIFF· προεπιλογή: ανάγνωση από τα μεταδεδομένα του TIFF), `--output PATH`, `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Με το `--live` ισχύουν επίσης τα κουμπιά παραμόρφωσης ευθυγράμμισης: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`.

> **Τα σύμβολα `--channel` διακρίνονται ως προς τα κεφαλαία και τα μικρά γράμματα.** Η πλευρά του συμβόλου πρέπει να ταιριάζει ακριβώς με τα ονόματα των καναλιών της προεπιλογής (οι προεπιλογές χρησιμοποιούν μικρά γράμματα, π.χ. NDVI = `red`, `nir` — ελέγξτε το `--list-presets`), ενώ η πλευρά της ζώνης πρέπει να αντιστοιχεί σε ένα όνομα ζώνης στην ευθυγραμμισμένη στοίβα (ή να είναι ένας δείκτης ζώνης με βάση το 0 στη λειτουργία εκτός σύνδεσης). Το `--channel red=Red_660 --channel nir=NIR_850` λειτουργεί· το `--channel RED=660` αποτυγχάνει με σφάλμα `channel_map missing entries`.

#### Μόνιμες συνδέσεις (Smart-Prep, ροή ισοδύναμη με GUI)

Αυτές οι εντολές διατηρούν τις κάμερες ανοιχτές στην ομάδα του backend κατά τη διάρκεια των κλήσεων της εντολής «CLI».

| Υποεντολή | Σκοπός |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Προσθήκη μίας κάμερας στην ομάδα (μονή κάμερα, χωρίς συστοιχία). |
| `lattice cam-disconnect [--serial SN] [--all]` | Απελευθέρωση. |
| `lattice cam-list` | Λίστα καμερών στο pool. |
| **`lattice array-connect`**|**Σύνδεση μόνιμου συγχρονισμένου συστοιχίου (το συνιστώμενο σημείο εισόδου).** Εκτελεί την πλήρη ροή έξυπνης προετοιμασίας μέσω GUI. |
| `lattice array-disconnect [--array-id ID] [--all]` | Απελευθέρωση συστοιχίας. |
| `lattice array-list` | Εμφάνιση λίστας συνδεδεμένων συστοιχιών. |
| `lattice array-status [--array-id ID]` | Ζωντανά fps, PTP, τελευταίο σφάλμα. |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--index\|--no-index] [--force-daq] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Μία συγχρονισμένη λήψη από τη ζωντανή συστοιχία — Ενιαία / Συνεχής / Διαστήματα / Ταχύτερη. **Προεπιλογή στο `all`** (ένα αρχείο ανά ισχύοντα τύπο εξαγωγής ανά κάμερα). Οι κάμερες που παραλείφθηκαν (π.χ. RGB εξαιρούνται από την ακτινοβολία/αντανάκλαση) αναφέρονται με τον κωδικό `Skipped: SN:<serial> (<reason>)`· η ένδειξη DAQ που χρησιμοποιείται για την αντανάκλαση αποθηκεύεται παράλληλα και αναφέρεται με τον κωδικό `DAQ: <path>`. Δείτε [Λειτουργίες καταγραφής, συσκευές εγγραφής &amp; Επεξεργασία εκτός σύνδεσης](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only]` | Καταγράφει τη ζωντανή προβολή συνδυασμένου δείκτη σε βίντεο/GIF (ποιότητας παρακολούθησης· απαιτεί το συνδυασμένο ρεύμα να είναι ανοιχτό). |
| `lattice array-burst [--duration S] [--max-frames N] [--build] [--products …]` | Σειρά λήψεων raw-Bayer υψηλού fps (επιπέδου ανάλυσης· επανεπεξεργασία εκτός σύνδεσης). |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Επανεπεξεργασία μιας αποθηκευμένης σειράς λήψεων raw σε βίντεο με βαθμονόμηση. |

##### Επιλογές `array-connect`

| Επιλογή | Προεπιλογή | Περιγραφή |
| --- | --- | --- |
| `--serials SN1,SN2,…` | αυτόματη ανίχνευση όλων των καμερών LATTICE (απαιτούνται ≥2) | Η πρώτη σειριακή είναι η MASTER. Όταν παραλείπεται, η ανίχνευση φιλτράρει τα μοντέλα LATTICE (`TRI032*`) και συνδέει όλα αυτά. |
| `--line {Line0,Line2,Line3}` | `Line2` | Γραμμή συγχρονισμού GPIO. |
| `--target-fps F` | αυτόματο | Ρυθμός ενεργοποίησης του Master. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Παράκαμψη του επιλογέα επιπέδου. |
| `--wire-ceiling-mbps MB_PER_S` | auto-detected | **Ο διαθέσιμος ρυθμός μεταφοράς δεδομένων του κεντρικού υπολογιστή, σε MB/s — ο αριθμός από τον οποίο εξαρτάται η κατανομή ολόκληρου του πλέγματος.** Μειώστε τον όταν το πλέγμα αναφέρει πλαίσια με αλλοίωση GVSP: η αυτόματη τιμή προκύπτει από τον αναφερόμενο ρυθμό σύνδεσης της κάρτας δικτύου (NIC), ο οποίος υπερεκτιμά τους προσαρμογείς USB, τις «λεπτές» λωρίδες PCIe και τα πολυάσχολα κοινόχρηστα δίκτυα. Αποθηκεύεται στο μπλοκ καταγραφής συστοιχιών του έργου, οπότε η επανασύνδεση μέσω των εντολών «reopen», «CLI» ή «SDK» την επαναφέρει. Δείτε [Κατάσταση συστοιχίας](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | auto | Ομαδοποίηση υλικού. |
| `--no-recommend` | off | Παράλειψη του βήματος-ανάλυσης. |
| `--no-ptp` | απενεργοποιημένο | Απενεργοποίηση PTP (οι χρονικές σημάνσεις μεταξύ καμερών **δεν** είναι τότε συγκρίσιμες). |

### Smart-AE / Smart-Capture

Οι συστοιχίες LATTICE εκτελούν συνεχή AE στο παρασκήνιο μόλις συνδεθούν, αλλά μια σκηνή που έχει μόλις ρυθμιστεί χρειάζεται λίγο χρόνο για να σταθεροποιηθεί. Το `array-capture --smart` είναι η **έτοιμη λύση**: περιμένει να σταθεροποιηθεί η αυτόματη έκθεση (AE) σε όλες τις κάμερες της συστοιχίας και, στη συνέχεια, ενεργοποιεί τη λήψη. Χρησιμοποιήστε το όταν αλλάζετε σκηνές κατά τη διάρκεια της συνεδρίας.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Η πολιτική σταθεροποίησης είναι συντηρητική από προεπιλογή: χρονικό όριο 5 δευτερολέπτων, παράθυρο σταθερότητας 1,5 δευτερολέπτων, ανοχή διακύμανσης έκθεσης ±5 %. Προσαρμόστε τις ρυθμίσεις μέσω του «SDK» (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`) αν χρειάζεστε διαφορετική συμπεριφορά από την αυτοματοποίηση.

### Επίπεδα εξαγωγής λήψεων (η προεπιλογή `all`)

Από αυτήν την έκδοση, `lattice capture`, `lattice multi-capture` και `lattice array-capture` **η προεπιλογή είναι το `--processing all`** — ένα αποθηκευμένο αρχείο ανά τύπο εξαγωγής που ισχύει για κάθε κάμερα, σύμφωνα με τη συμπεριφορά της επιλογής «Capture All» (Λήψη όλων) στο GUI. Τα επίπεδα είναι:

| Επίπεδο | Έξοδος | Ισχύει για |
| --- | --- | --- |
| `raw` | Μονοκαναλικό Bayer (μονοχρωματικές κάμερες: η μοναδική ζώνη) απευθείας από τον αισθητήρα. | Όλες οι κάμερες. |
| `debayered` | Αποσυμπίεση BGR 3 καναλιών (μονοχρωματικές κάμερες: 1 κανάλι σε κλίμακα του γκρι). | Όλες οι κάμερες. |
| `radiance` | float32 W/m²/sr/nm μέσω της πλήρους ραδιομετρικής αλυσίδας. | Μόνο πολυφασματικές (M3C/M3M) — **παραλείπεται για κάμερες με φίλτροRGB**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), έτοιμο για Pix4D. | Μόνο πολυφασματική, και **μόνο όταν υπάρχει σύνδεση με DAQ + η κάμερα είναι βαθμονομημένη**· διαφορετικά παραλείπεται. |
| `preview` / `display` | Πλήρης αλυσίδα προεπισκόπησης GUI (CCM + WB + γάμμα σύμφωνα με το προφίλ της κάμερας). Το `lattice capture` ονομάζει αυτό το `preview`· τα `array-capture`/`multi-capture` χρησιμοποιούν το `display`. | Όλες οι κάμερες. |

Περάστε ένα μόνο επίπεδο για να αποθηκεύσετε μόνο αυτό (`--processing debayered`). Όταν ζητάτε το `all`, τα επίπεδα που δεν ισχύουν για μια συγκεκριμένη κάμερα παραλείπονται (και αναφέρονται), χωρίς να εμφανίζεται σφάλμα — μια κάμερα που δεν είναι συνδεδεμένη ή δεν έχει βαθμονομηθεί εξακολουθεί να λαμβάνει τα `raw` / `debayered` / `preview`.

Για κάθε πλαίσιο ανακλαστικότητας, η ένδειξη κατεύθυνσης προς τα κάτω του DAQ που χρησιμοποιείται στην πραγματικότητα εγγράφεται σε ένα **`.daq`** sidecar δίπλα στην εικόνα (έτσι ώστε η λήψη να μπορεί να υποβληθεί σε επανεπεξεργασία αργότερα) και αναφέρεται σε μια γραμμή `DAQ:`.

### Πώς μοιάζει ένας φάκελος καταγραφών

Κάθε τύπος εξαγωγής καταλήγει στον **δικό του υποφάκελο** κάτω από το `-o`, οπότε μια καταγραφή πολλαπλών επιπέδων δεν αναμιγνύει ποτέ τους τύπους:

```
output/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
└── *.daq          the downwelling reading matched to the capture
```

Το `<ts>` είναι η χρονική σήμανση της καταγραφής και το `<serial>` ο σειριακός αριθμός της κάμερας, οπότε μια συγχρονισμένη ομάδα μοιράζεται μια
χρονική σήμανση μεταξύ των καμερών. **Σημειώστε τη μία ασυμμετρία:** το επίπεδο `display` αποθηκεύεται σε ένα φάκελο
με το όνομα `preview/`, ενώ τα ίδια τα αρχεία διατηρούν το `_display` στο όνομά τους — ο φάκελος και η κατάληξη διαφέρουν
μόνο για αυτό το επίπεδο. Τα άγνωστα επίπεδα καταλήγουν σε φάκελο με το δικό τους όνομα, και αν ο υπουποφάκελος
δεν μπορεί να δημιουργηθεί, το αρχείο εγγράφεται στη ρίζα εξόδου αντί να χαθεί.

**Επανεπεξεργασία ενός φακέλου καταγραφών:**ορίστε το `chloros-cli process` στη**ρίζα των καταγραφών**
(`output/`). Το `process` συνήθως εισάγει μόνο τον φάκελο που ορίζετε, αλλά όταν αυτός ο φάκελος δεν περιέχει
εικόνες και έχει υποφακέλους, κατεβαίνει αυτόματα — έτσι, οιυποφάκελοι της ρίζας και η
ρίζα `.daq` συλλέγονται όλα με μία κίνηση. Κάθε επίπεδο μιας λήψης εισάγεται ως μία ενιαία εικόνα, με
τα άλλα επίπεδα να είναι διαθέσιμα ως λειτουργίες, αντί για μία εικόνα ανά επίπεδο.

Η άμεση ονομασία ενός **υποφακέλου επιπέδου** (π.χ. `output/raw/`) λειτουργεί επίσης. Με αυτόν τον τρόπο, η ρίζα
`.daq` πίσω, οπότε αντιγράψτε ή κατευθύνετε την ανάγνωση DAQ παράλληλα όταν επαναπαράγετε ένα ραδιομετρικό
προϊόν από το `raw/` — διαφορετικά, η αντιστοίχιση της χρονικής σήμανσης δεν έχει τίποτα με το οποίο να συσχετιστεί.

**Η επεξεργασία ξεκινά πάντα από το `raw`.** Σε κάθε καταγραφή, το ακατέργαστο πλαίσιο αποτελεί την πηγή της ροής επεξεργασίας·
τα `debayered`, `radiance`, `reflectance` και `preview` εμφανίζονται ως τρόποι προβολής, αλλά δεν τροφοδοτούνται ποτέ
πίσω στη διαδρομή επεξεργασίας. Η επανεπεξεργασία ενός παράγωγου προϊόντος θα εφάρμοζε εκ νέου το βινιέτ, το CCM και
μαθηματικές πράξεις ακτινοβολίας που έχουν ήδη ενσωματωθεί στα εικονοστοιχεία του, οπότε το Chloros το απορρίπτει αντί να
προχωρήσει σε διπλή επεξεργασία. Δύο συνέπειες που αξίζει να γνωρίζετε:

- Οι αποδόσεις `index/` και `composite/` **ποτέ** δεν υποβάλλονται σε επεξεργασία. Πρόκειται για εξόδους, όχι για καταγραφές —
  μια απόδοση LUT τύπου «NDVI» δεν έχει καμία ουσιαστική ερμηνεία ακτινοβολίας.
- Ένας φάκελος καταγραφών που εξάγεται **χωρίς** το `raw` (π.χ. `array-capture --processing reflectance`) δεν έχει
  έγκυρη πηγή στην αλυσίδα επεξεργασίας. Αυτές οι καταγραφές εισάγονται και εμφανίζονται κανονικά, αλλά το `process` τις παραλείπει
  και το επισημαίνει:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Εάν πραγματικά χρειάζεστε να προωθήσετε ένα παράγωγο προϊόν — μια συνεδρία κόμβου που καταγράφηκε με
  ενεργοποιημένο το `demosaic` ή έναν παλαιό φάκελο — το `--input-level {raw,debayered,processed}` επιβάλλει το σημείο
  εισόδου και παρακάμπτει την παράλειψη. Αυτή η σημαία αποτελεί τη σκόπιμη «εξώπορτα διαφυγής»· το `auto` (η προεπιλογή)
  δεν επεξεργάζεται ποτέ μια καταγραφή που δεν διαθέτει αρχικό υλικό.

### Παραλειπόμενες καταγραφές σε συστοιχίες μεικτών φίλτρων

Όταν συνδυάζετε κάμερες «RGB» και πολυφασματικές κάμερες σε μία συστοιχία, το `array-capture --processing radiance` (ή το `reflectance`) αποθηκεύει τα πολυφασματικά καρέ και **παραλείπει** τις κάμερες τύπου «RGB» — η ακτινοβολία ανά Bayer δεν έχει νόημα για έναν ευρυζωνικό αισθητήρα. Το CLI εκτυπώνει ρητά κάθε αποθηκευμένο αρχείο (μαζί με το επίπεδο εξαγωγής του), κάθε εγγραφή `.daq` και κάθε παράλειψη, οπότε ο αριθμός των αρχείων δεν αποτελεί έκπληξη:

```
  Saved: output/sync_…_SN213800234.tif [reflectance] (SN:213800234, fid:1)
  Saved: output/sync_…_SN214000533.tif [reflectance] (SN:214000533, fid:1)
  Saved: output/sync_…_SN214701288.tif [reflectance] (SN:214701288, fid:1)
  DAQ:   output/sync_…_daq-e-54b5e0.daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Τα διακριτικά αιτιολόγησης παράλειψης ακολουθούν το μοτίβο `<level>-not-applicable-to-rgb-cam`. Η ανακλαστικότητα μπορεί επίσης να παραλειφθεί με `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, καθώς και με το `dls-uncalibrated-band-<nm>` όταν η ζώνη βρίσκεται κυρίως εκτός του ραδιομετρικά βαθμονομημένου εύρους του αισθητήρα φωτός DAQ (~374–974 nm) — μεταξύ των διαθέσιμων κωδικών προϊόντων (SKU) μόνο το F988, του οποίου η υποστηριζόμενη διαδρομή είναι η ροή εργασίας με πάνελ ανακλαστικότητας.

Χρησιμοποιήστε το `--processing debayered` (ή το `display`) για να συμπεριλάβετε κάθε κάμερα ανεξάρτητα από τον τύπο φίλτρου, ή την προεπιλεγμένη τιμή `all` για να λάβετε κάθε εφαρμόσιμο επίπεδο ανά κάμερα με μία μόνο λήψη.

---

## Λειτουργίες καταγραφής, συσκευές εγγραφής &amp; επανεπεξεργασία εκτός σύνδεσης

Όλα αυτά λειτουργούν σε ένα **μόνιμο πίνακα** (εκτελέστε πρώτα το `array-connect`). Αντικατοπτρίζουν τον πίνακα καταγραφής του GUI.

### Λειτουργίες `array-capture`

Το `array-capture` είναι μια μεμονωμένη εντολή με τέσσερις λειτουργίες κλείστρου καθώς και ένα σύνολο εξαγωγών σεggles:

| Λειτουργία | Σημαία | Συμπεριφορά |
| --- | --- | --- |
| **Μονή** *(προεπιλογή)* | (καμία) | Μία συγχρονισμένη ομάδα καταγραφής, και στη συνέχεια έξοδος. |
| **Συνεχής** | `--continuous` | Πέρασμα-προς-πίσω περάσματα μέχρι τα `Ctrl+C`, `--count N` ή `--duration S`. |
| **Διάστημα** | `--interval S` | Μία διέλευση κάθε `S` δευτερόλεπτα (μετρούμενα από την αρχή κάθε διέλευσης), ίδια όρια. |
| **Ταχύτερη** | `--fastest` | Μόνο ακατέργαστα δεδομένα + η καταγεγραμμένη τιμή DAQ + το σύνθετο δείκτη συνδυασμού· παραλείπει τους μαθηματικούς υπολογισμούς ακτινοβολίας/ανακλαστικότητας/απεικόνισης, ώστε το καρέ να εμφανίζεται γρήγορα. Υπονοεί το `--processing raw --force-daq`. Επεξεργαστείτε εκ νέου αργότερα το αποθηκευμένο `.daq` σε βαθμονομημένα προϊόντα. |

Επιλογές εξαγωγής (συνδυάστε με οποιαδήποτε λειτουργία· όλες μοιράζονται το GUI/το τελικό σημείο SDK):

| Σημαία | Αποτέλεσμα |
| --- | --- |
| `--processing LEVEL` | Ενιαίο επίπεδο εξαγωγής, ή `all` (προεπιλογή). |
| `--levels L1,L2,…` | Ρητό υποσύνολο τύπων εξαγωγής (π.χ. `raw,radiance,reflectance`) · **υπερισχύει του `--processing`**. |
| `--aligned` / `--no-aligned` | Προσαρμογή της μη ακατέργαστης εξαγωγής κάθε μέλους στο [προφίλ ευθυγράμμισης](#alignment) του πίνακα (συν-καταχωρημένη). Η ακατέργαστη μορφή παραμένει χωρίς προσαρμογή, αλλά φέρει τον μετασχηματισμό στα μεταδεδομένα. Επιστρέφει σε μη ευθυγραμμισμένη μορφή (με προειδοποίηση) εάν ο πίνακας δεν διαθέτει προφίλ. |
| `--index` / `--no-index` | Αποθήκευση / παράλειψη της επικάλυψης δείκτη βλάστησης ανά κάμερα, όπου έχει ρυθμιστεί. Προεπιλογή: απόδοση της εικόνας. |
| `--force-daq` | Αποθήκευση της εκχωρημένης μέτρησης DAQ/DLS ως αρχείο sidecar `.daq`, ακόμη και όταν κανένα επιλεγμένο επίπεδο δεν το απαιτεί (π.χ. λήψη μόνο σε μορφή raw), ώστε τα καρέ να μπορούν να υποβληθούν σε εκ νέου επεξεργασία σε ανακλαστικότητα/δείκτη εκτός σύνδεσης. |
| `--smart` | Αναμονή για την να σταθεροποιηθεί σε όλες τις κάμερες πριν από την ενεργοποίηση (βλ. [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | Συμπίεση εικονοστοιχείωνTIFF. `deflate` (προεπιλογή) = zlib L1 χωρίς απώλειες + οριζόντιος προγνωστικός αλγόριθμος, ~4,1 MB ανά καρέ πλήρους ανάλυσης· `none` = χωρίς συμπίεση, ~5× ταχύτερη εγγραφή στα ~6,3 MB ανά καρέ — χρησιμοποιήστε το για μέγιστο σταθερό ρυθμό όταν το επιτρέπει ο δίσκος. Και τα δύο είναι χωρίς απώλειες και διαβάζονται πανομοιότυπα κατά την εισαγωγή. |

> **TIFF με μία εγγραφή + το μοντέλο σταθερού ρυθμού.**Οι καταγραφές εγγράφονται σε**μία**διέλευση αρχείου TIFF που μεταφέρει εικονοστοιχεία + XMP + IFD0 Μάρκα/Μοντέλο (μετρημένο σε Mono12 πλήρους ανάλυσης: 36 ms συμπιεσμένο / 6,5 ms ασυμπίεστο, έναντι ~148 ms για την παλιά μέθοδο «εγγραφή και μετά επανεγγραφή με το ExifTool»); η μόνη εναπομένουσα εργασία του ExifTool (τελειοποίηση του υπο-IFD EXIF) εκτελείται σε έναν ασύγχρονο εργάτη παρασκηνίου, και ένα καρέ είναι ολοκληρωμένο και έτοιμο για εισαγωγή ακόμα και αν αυτή η εργασία δεν εκτελεστεί ποτέ. Σημειώστε ότι η συμπίεση DEFLATE διατηρεί το GIL του Python, οπότε οι συμπιεσμένες εγγραφές**δεν**παραλληλοποιούνται στα νήματα εγγραφής ανά κάμερα — η συνεχής λήψη πλήρους ανάλυσης με 8 κάμερες με ρυθμό αισθητήρα (~10,4 fps) απαιτεί το `--compression none`**και** δίσκο κατηγορίας NVMe(~500 MB/s σταθερής ταχύτητας εγγραφής). Ο ίδιος ρυθμιστής εμφανίζεται ως `compression` στο `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --aligned --no-index -o out/
```

### `array-record` — βίντεο/GIF συνδυασμένου δείκτη (ποιότητας παρακολούθησης)

Καταγράφει ό,τι εμφανίζει η **ζωντανή προβολή συνδυασμένου δείκτη** σε ένα `.avi` (και προαιρετικά σε ένα `.gif`). Επειδή αντλεί δεδομένα από το ζωντανό σύνθετο σήμα, η συνδυασμένη ροή πρέπει να είναι ανοιχτή (π.χ. η συστοιχία προβάλλεται στο GUI) για να καταγράφονται τα καρέ. Ελέγχει την πρόοδο κάθε 2 δευτερόλεπτα και σταματά σε `--duration`, στο `Ctrl+C` ή όταν ο καταγραφέας τερματίζεται αυτόματα.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/
```

| Σημαία | Προεπιλογή | Περιγραφή |
| --- | --- | --- |
| `--array-id ID` | μόνο πίνακας | Πίνακας προορισμού (παραλείψτε αν είναι συνδεδεμένος μόνο ένας). |
| `-o, --output DIR` | `output` | Κατάλογος εξόδου (τοπικός στο backend). |
| `--fps F` | `10` | Ρυθμός καρέ εγγραφής. |
| `--duration S` | μέχρι Ctrl+C | Αυτόματη διακοπή μετά από `S` δευτερόλεπτα. |
| `--gif` | απενεργοποιημένο | Εγγραφή κινούμενου GIF επίσης. |
| `--gif-only` | απενεργοποιημένο | Δημιουργία μόνο GIF (χωρίς `.avi`). |

### `array-burst` — ακατέργαστη σειρά λήψεων Bayer υψηλού fps (ποιότητας ανάλυσης)

Διαβάζει απευθείας το buffer συγχρονισμένης ομάδας του βρόχου λήψης — **δεν απαιτείται αλυσίδα βαθμονόμησης, exiftool ή ζωντανή προβολή** — οπότε λειτουργεί με τηνπλήρη ρυθμό λήψης της κάμερας. Γράφει καρέ raw + ένα manifest ανά καρέ + ένα `.daq` για κάθε ξεχωριστή ανάγνωση DLS κάτω από το `<output>/bursts/<base>/`. Επεξεργαστείτε εκ νέου εκτός σύνδεσης (επόμενη εντολή) ή περάστε το `--build` για να γίνει αμέσως μόλις σταματήσει.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Σημαία | Προεπιλογή | Περιγραφή |
| --- | --- | --- |
| `--array-id ID` | μόνο πίνακας | Πίνακας προορισμού. |
| `-o, --output DIR` | `output` | Κατάλογος εξόδου (η ροή καταλήγει στο `<DIR>/bursts/<base>/`). |
| `--duration S` | μέχρι Ctrl+C | Αυτόματη διακοπή μετά από `S` δευτερόλεπτα. |
| `--max-frames N` | χωρίς όριο | Αυτόματη διακοπή μετά από `N` ακατέργαστα καρέ. |
| `--build` | απενεργοποιημένο | Μετά τη διακοπή, άμεση επανεπεξεργασία της ράφας (όπως στο `array-build-video`). |
| `--products …` | `combined:index` | Με το `--build`: ποια βίντεο θα δημιουργηθούν (βλ. παρακάτω). |
| `--fps F` | `10` | Με το `--build`: fps εξόδου βίντεο. |
| `--save-tiffs` | απενεργοποιημένο | Με το `--build`: αποθήκευση επίσης βαθμονομημένων αρχείων TIFF ανά καρέ. |
| `--gif` | απενεργοποιημένο | Με το `--build`: εγγραφή επίσης κινούμενων GIF. |

### `array-build-video` — επανεπεξεργασία εκτός σύνδεσης μιας αποθηκευμένης σειράς

Αντιστοιχίζει χρονικά κάθε ακατέργαστο καρέ με την πλησιέστερη αποθηκευμένη μέτρηση `.daq` και το περνάει μέσα από την **ίδια αλυσίδα ακτινοβολίας / ανακλαστικότητας / δείκτη όπως η ροή εισαγωγής**, δημιουργώντας ένα ή περισσότερα βίντεο.

Το `--products` είναι μια λίστα με κόμματα από στοιχεία `kind:level`, όπου `kind` ∈ `per_cam` | `combined` και το `level` ∈ `radiance` | `reflectance` | `index`. Ένα απλό `level` (χωρίς `kind:`) ορίζεται από προεπιλογή ως `per_cam`. Η προεπιλογή είναι το `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Σημαία | Προεπιλογή | Περιγραφή |
| --- | --- | --- |
| `--burst-dir DIR` | (απαιτείται) | Διαδρομή προς το φάκελο burst (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | Λίστα `kind:level`, όπως παραπάνω. |
| `--fps F` | `10` | FPS βίντεο εξόδου. |
| `--save-tiffs` | απενεργοποιημένο | Αποθήκευση επίσης TIFF με βαθμονόμηση ανά καρέ μαζί με το βίντεο. |
| `--gif` | off | Επίσης, εγγραφή κινούμενων αρχείων GIF. |

> **Επιλέξτε τη σωστή συσκευή εγγραφής.** Το `array-record` είναι *επιπέδου παρακολούθησης* — καταγράφει το ζωντανό σύνθετο σήμα όπως εμφανίζεται και απαιτεί την ροή να είναι ανοιχτή. Το `array-burst` → `array-build-video` είναι *επιπέδου ανάλυσης* — αποθηκεύει ακατέργαστα δεδομένα αισθητήρα με πλήρη ρυθμό και ανασυνθέτει στη συνέχεια βίντεο με βαθμονομημένη ακτινοβολία/ανακλαστικότητα/δείκτη, χωρίς να απαιτείται ζωντανή προβολή.

### Μονοφασικές (M3M) κάμερες μίας ζώνης

Η σειρά **M3M**είναι η μονοχρωματική έκδοση της Bayer**M3C**: ένα φίλτρο παρεμβολών στενής ζώνης ανά κάμερα (`M3M-<lens>-F<wavelength>`, π.χ. `M3M-L87-F685`), οπότε ο αισθητήρας παρέχει**μία μόνο ζώνη κλίμακας του γκρι** χωρίς μωσαϊκό Bayer. Δεν υπάρχει τίποτα να απομοσαϊκοποιηθεί, καμία διασταυρούμενη παρεμβολή μεταξύ καναλιών να διαχωριστεί, ούτε ισορροπία λευκού να ρυθμιστεί — ολόκληρη η διαδικασία επεξεργασίας χρώματος από την οθόνη «RGB» απλά δεν ισχύει.

Τι σημαίνει αυτό για το «CLI»:

- Τα **`lattice white-balance`, `lattice color-profile`, `lattice color`**ανιχνεύουν μια μονοχρωματική κάμερα και**παραλείπουν την επεξεργασία με ένα μήνυμα μίας γραμμής** αντί να εφαρμόζουν άσκοπες ρυθμίσεις. Εξακολουθούν να λειτουργούν κανονικά με μια κάμερα τύπου «RGB» / Bayer M3C στην ίδια συνεδρία.
- Τα **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** εξακολουθούν να λειτουργούν — η ακτινοβολία και η ανακλαστικότητα είναι ραδιομετρικοί χάρτες *ανά ζώνη* και ορίζονται απόλυτα σωστά για μία ζώνη. Τα μονοχρωματικά καρέ φέρουν έναν **ταυτόσημο** πίνακα απόκρισης αισθητήρα (χωρίς αποσύνθεση 3×3), οπότε το επίπεδο περνάει από τους μαθηματικούς υπολογισμούς βαθμονόμησης χωρίς να επηρεάζεται.
- **Μια μεμονωμένη μονοχρωματική κάμερα δεν μπορεί να παράγει δείκτη βλάστησης.**Οι δείκτες NDVI / NDRE / κ.λπ. απαιτούν τουλάχιστον δύο ζώνες (π.χ. Red + NIR). Για να λάβετε έναν δείκτη από μονοχρωματικό εξοπλισμό, στρέψτε**αρκετές** κάμερες M3M σε διαφορετικά μήκη κύματος, ευθυγραμμίστε τις σε μία πολυζωνική στοίβα και υπολογίστε *αυτήν*:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

Τα σύμβολα `--channel` πρέπει να ταιριάζουν **ακριβώς** με τα ονόματα των καναλιών της προεπιλογής (διακρίνεται η χρήση κεφαλαίων-μικρών γραμμάτων· τα NDVI είναι με μικρά γράμματα `red`,`nir` — βλ. `--list-presets`), και τα ονόματα πλευρών ζώνης πρέπει να αντιστοιχούν σε μια ζώνη στην ευθυγραμμισμένη στοίβα (η λειτουργία εκτός σύνδεσης δέχεται επίσης δείκτες ζώνης με βάση το 0, π.χ. `--channel red=0 --channel nir=1`).

Ο διακριτής σε όλη τη στοίβα είναι το διακριτικό `M3M` στη συμβολοσειρά μοντέλου (δεν εμφανίζεται ποτέ σε συμβολοσειρά `M3C`), το οποίο εμφανίζεται στο GUI/SDK ως `is_mono`.

---

## Ρύθμιση και βελτιστοποίηση κάρτας δικτύου κεντρικού υπολογιστή (συστοιχίες LATTICE)

Οι κάμερες LATTICE μεταδίδουν GVSP μέσω της κάρτας δικτύου Ethernet του κεντρικού υπολογιστή, οπότε για συστοιχίες πολλαπλών καμερών το **πρόγραμμα οδήγησης**της κάρτας και το**μέγεθος του δακτυλίου λήψης** έχουν την ίδια σημασία με το ρυθμό σύνδεσης. Λανθασμένες ρυθμίσεις εμφανίζονται ως πύλη `FRAMES WILL DROP` / `Reduce ROI to enable` στον πίνακα «Ρυθμίσεις συστοιχίας» (και στο `lattice network-analysis` / το `analyze_array_network()` του «SDK»), ακόμη και όταν οι ίδιες οι κάμερες λειτουργούν κανονικά.

### Προσαρμογείς USB 10GbE — Realtek RTL8157 («Realtek USB 10GbE Family Controller»)

| Στοιχείο | Απαιτούμενη τιμή | Γιατί έχει σημασία |
| --- | --- | --- |
| **Έκδοση προγράμματος οδήγησης**|**≥ v10.67 (Ιαν. 2026)**, INF `rtump64x64sta.inf` | Το παλαιό πρόγραμμα οδήγησης**του 2016**(v10.65, `rtump64x64.inf`) δεν διαχειρίζεται σωστά την απενεργοποίηση και προκαλεί σφάλματα ελέγχου με**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**κατά το κλείσιμο/την επανεκκίνηση/την κατάσταση αναστολής λειτουργίας. Η μετάβαση κολλάει (χρονικό όριο ~5 λεπτών), ο χρήστης απενεργοποιεί βίαια τη συσκευήκαι οι επαναλαμβανόμενοι μη ομαλοί τερματισμοί**καταστρέφουν το αποθετήριο WMI**(το PowerShell και τα εργαλεία αρχίζουν να παρουσιάζουν σφάλματα με το `Invalid class`) και**μπλοκάρουν τη στοίβα USB** στην επόμενη εκκίνηση (η κάρτα δικτύου δεν ενεργοποιείται· οι μονάδες USB σταματούν να αναγνωρίζονται). Ενημερώστε από το realtek.com (ή τον προμηθευτή του dongle) πριν βασιστείτε σε «καθαρές» επανεκκινήσεις. |
| **Buffers Λήψης**— λέξη-κλειδί `ReceiveBufferLen` |**256**(μέγιστο οδηγού) | Ο δακτύλιος RX της κάρτας δικτύου. Η προεπιλογή του οδηγού στα**32**αφήνει μόνο ~0,26 MB χρησιμοποιήσιμου δακτυλίου — πάρα πολύ μικρό για μια ριπή πολλαπλών καμερών — οπότε ο πίνακας συστοιχιών αναφέρει το σφάλμα `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` και εμποδίζει τη σύνδεση. Στο**256**ο δακτύλιος είναι μεγάλος (**~13,5 MB, όπως μετρήθηκε στον εργαστηριακό κεντρικό υπολογιστή 10GbE**), παρέχοντας στον αγωγό RX πραγματικό περιθώριο για ριπές GVSP από πολλές κάμερες. (Το αν μια δεδομένη διαμόρφωση *συνδέεται* πράγματι αποφασίζεται από δύο ελέγχους — τον έλεγχο εισόδου **drain-aware**και τον έλεγχο**συνολικής υπερ-εγγραφής** — και όχι από μια απλή σύγκριση ράφας έναντι δακτυλίου· βλ. [Μοντέλο fps &amp; ράφας πίνακα](#array-fps--burst-model).) |
| **URB λήψης**— λέξη-κλειδί `PendingReceives` |**64** (μέγ.) | Μπλοκ αιτημάτων USB σε εξέλιξη· αυξάνονται παράλληλα με τα buffer λήψης για απορρόφηση εκρήξεων. |
| **Jumbo Frame** — λέξη-κλειδί `*JumboPacket` | **9014** | Απαιτείται για πακέτα GVSP 9000 byte (6× λιγότερα πακέτα/πλαίσιο από 1500). |

> ⚠️ **Μια ενημέρωση του προγράμματος οδήγησης της κάρτας δικτύου ΕΠΑΝΑΦΕΡΕΙ αυτές τις προηγμένες ιδιότητες στις προεπιλεγμένες τιμές.**Μετά την ενημέρωση ή την αντικατάσταση του προγράμματος οδήγησης της κάρτας,**εφαρμόστε ξανά** τις ρυθμίσεις `ReceiveBufferLen=256` και `PendingReceives=64`, διαφορετικά ο πίνακας συστοιχιών θα ενεργοποιήσει ξανά τον έλεγχο πρόσβασης, παρόλο που «δεν έχει αλλάξει τίποτα στο υλικό». Αυτή είναι η κύρια αιτία για την οποία ένας εξοπλισμός που λειτουργούσε προηγουμένως αρνείται ξαφνικά να συνδεθεί.

Εφαρμόστε από ένα **PowerShell με δικαιώματα διαχειριστή** (αντικαταστήστε το όνομα της κάρτας σας, π.χ. `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **Το `lattice network --fix` καλύπτει τις κάρτες USB 10GbE.** Τώρα ανιχνεύει τον τύπο του προσαρμογέα και ρυθμίζει τη σωστή λέξη-κλειδί για το δακτύλιο λήψης: `*ReceiveBuffers`→2048 για κάρτες δικτύου PCIe (Intel I219 κ.λπ.), ή `ReceiveBufferLen`→256 + `PendingReceives`→64 για τον ελεγκτή **USB** 10GbE της Realtek (ο οποίος δεν εκθέτει το `*ReceiveBuffers`). Οι τιμές-στόχοι περιορίζονται στο μέγιστο που αναφέρει κάθε πρόγραμμα οδήγησης (`NumericParameterMaxValue`), οπότε δεν εγγράφεται ποτέ τιμή εκτός εύρους. Εκτελέστε το από ένα **τερματικό με δικαιώματα διαχειριστή**· όπως κάθε ρύθμιση που βασίζεται στο μητρώο, η αλλαγή τίθεται σε ισχύ μετά την επανεκκίνηση του προσαρμογέα ή την επανεκκίνηση του συστήματος. Οι παραπάνω χειροκίνητες εντολές `Set-NetAdapterAdvancedProperty` παραμένουν μια καλή εναλλακτική λύση — εφαρμόζονται σε πραγματικό χρόνο (επανασύνδεση του προσαρμογέα) χωρίς επανεκκίνηση.

### Βασικές έννοιες δικτύου (όλοι οι σύνδεσμοι LATTICE)

- **Διευθυνσιοδότηση:** link-local `169.254.0.0/16` (GigE Vision LLA). Ο κεντρικός υπολογιστής λαμβάνει μια στατική διεύθυνση `169.254.x.x/16`· οι κάμερες και το DAQ-E αυτοαναθέτουν διευθύνσεις στο ίδιο εύρος. Δεν απαιτείται DHCP/πύλη.
- **Μέγεθος πακέτου:**προτιμήστε το jumbo (9000), αλλά αφήστε την αυτόματη ανίχνευση να το βρει — επαναμετρά σε κάθε σύνδεση και ήδη αγνοεί τοόριο ICMP των 1500 byte μέσω ενός ελέγχου GVSP, οπότε καταλήγει στο jumbo όπου το καλώδιο το μεταφέρει πραγματικά. Ορίστε σταθερή τιμή με το `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` μόνο όταν γνωρίζετε καλύτερα από τον έλεγχο και προτιμήστε την ρύθμιση ανά εντολή αντί για μόνιμη: μια σταθερή τιμή παρακάμπτει τον έλεγχο, οπότε αν η διαδρομή δεν μπορεί πράγματι να μεταφέρει 9000,**κάθε** καταγραφή θα λήξει με `SC_ERR_TIMEOUT -1011` (βλ. [Μεταβλητές Περιβάλλοντος](#environment-variables)).
- **Ο δακτύλιος RX κλιμακώνεται με το `ReceiveBufferLen`:**στην προεπιλεγμένη τιμή `32` ο χρησιμοποιήσιμος δακτύλιος είναι ~0,26 MB (πολύ μικρός για οποιαδήποτε ριπή πολλαπλών καμερών)· στη μέγιστη τιμή `256` είναι μεγάλος (~13,5 MB, όπως μετρήθηκε στον κεντρικό υπολογιστή 10GbE του εργαστηρίου), προσφέροντας πραγματικό περιθώριο. Το αν μια διαμόρφωση θα συνδεθεί αποφασίζεται στη συνέχεια από τον έλεγχο πρόσβασης με γνώμονα την απορρόφηση**και** τον παρακάτω έλεγχο συνολικής υπερ-εγγραφής — όχι από μια απλή σύγκριση ράφας έναντι δακτυλίου.

### Μοντέλο fps &amp; ράφας του Array

Πώς να διαβάσετε τον πίνακα ρυθμίσεων του Array (και το `lattice analyze-array` / το `analyze_array_network` του «SDK»):

- **Η ροή (burst) αθροίζεται ανά κάμερα στην πραγματική μορφή pixel της κάθε κάμερας.**Οι μονοχρωματικές κάμερες**M3M**μεταδίδουν**Mono12 (2 B/px)**· οι κάμερες**M3C**Bayer μεταδίδουν 8- ή 12-bit (το TRI032S εκπέμπει σιωπηρά BayerRG12 ακόμη και όταν ζητείται BayerRG8). Έτσι, ένα καρέ πλήρους ανάλυσης από 4 κάμερες είναι**~12,6 MB αν είναι όλες 8-bit, αλλά ~25 MB με τρεις μονοχρωματικές κάμερες 12-bit**. Η προβολή προσδιορίζει τη μορφή κάθε κάμερας από το μοντέλο της (cache ταυτότητας), οπότε η ροή αντιστοιχεί σε αυτό που μεταφέρει πραγματικά το καλώδιο — και όχι σε μια γενική υπόθεση για BayerRG8.
- **Ένας προσαρμογέας USB Ethernet έχει ανώτατο όριο τα 200 MB/s, ανεξάρτητα από τις τεχνικές προδιαγραφές του.** Ο πίνακας απόδοσης που μετατρέπει τον ρυθμό σύνδεσης σε σταθερή τιμή προέρχεται από το PCIe· μια κάρτα δικτύου USB ανακοινώνει τον *Ethernet* ταχύτητα σύνδεσης, αλλά περιορίζεται από τον δίαυλο USB και τον οδηγό του. Ένα dongle USB 10GbE κατέγραφε ~1063 MB/s «σταθερή» ταχύτητα — ένας αριθμός που δεν επαληθεύτηκε ποτέ — και ο ρυθμός που προέκυψε αλλοίωσε το 6–18 % των πλαισίων, ενώ εξακολουθούσε να αναφέρει ένα ικανοποιητικό fps-στόχο. Οι κάρτες δικτύου που συνδέονται μέσω USB περιορίζονται πλέον στα **200 MB/s** ως απόλυτο όριο (το όριο είναι ο δίαυλος, οπότε δεν εξαρτάται από τα τεχνικά χαρακτηριστικά· ένας προσαρμογέας USB 1 GbE αποδίδει ~80 MB/s και δεν επηρεάζεται). Το `wire_ceiling_source` στην εγγραφή δυνατοτήτων το αναφέρει ρητά, ενώ το `nic_is_usb` το επισημαίνει. Μπορείτε να παρακάμψετε και τις δύο περιπτώσεις με το `--wire-ceiling-mbps`.
- **Η πρόσβαση λαμβάνει υπόψη την εκροή, όχι τη σύγκριση ολόκληρης ριπής έναντι δακτυλίου.** Μια ταυτόχρονη ριπή πρέπει να χωράει μόνο στο *μεταβατικό backlog* = `max(0, Σ per-cam arrival − host drain) × emit_window`, όχι ολόκληρη τη ριπή. Σε δομή γρήγορου κεντρικού υπολογιστή / αργών καμερών (ένας **PCIe**κεντρικός υπολογιστής 10G + 4 κάμερες 1 GbE: άφιξη ≈ 320 MB/s, αποστράγγιση ≈ 1063 MB/s) ο κεντρικός υπολογιστής αποστραγγίζει τα δεδομένα ταχύτερα από ό,τι γεμίζουν οι κάμερες, η καθυστέρηση ≈ 0, οπότε η πλήρης ανάλυση**επιτρέπει**την εκπομπή, παρόλο που η ριπή των 25 MB υπερβαίνει το δακτύλιο των 13,5 MB. Αν τοποθετήσετε τις ίδιες τέσσερις κάμερες πίσω από έναν προσαρμογέα**USB**10GbE, η εκκένωση είναι 200 MB/s, όχι 1063 — η εισερχόμενη ροή την ξεπερνά, και η απώλεια εμφανίζεται ως κατεστραμμένα πλαίσια και όχι ως χαμηλότερο ρυθμό πλαισίων. Σε έναν κεντρικό υπολογιστή 1 GbE, το κατώτατο όριο DLThr των καμερών στα 31,25 MB/s κάνει την εισερχόμενη ροή να ξεπερνά την εκκένωση →**αποκλείει** σωστά (για *αυτή* την κατηγορία αποκλεισμού, μειώστε το ROI ή χρησιμοποιήστε binning ≥ 2). Η είσοδος είναι μία από τις **δύο** πύλες σύνδεσης — η άλλη είναι ο παρακάτω έλεγχος συνολικής υπερ-εγγραφής.
- **Τα προβλεπόμενα fps αποτελούν ένα συντηρητικό ανώτατο όριο σειριακής ανάκτησης.**Ο βρόχος λήψης του κεντρικού υπολογιστή αντλεί επί του παρόντος το buffer κάθε κάμερας**σειριακά**(~ένα παράθυρο εκπομπής ανά κάμερα το καθένα), οπότε ο κύκλος περιορίζεται από το `max(readout+emit, N × emit)` με την εκπομπή ανά-κάμερα να περιορίζεται στο**σύνδεσμο πρόσβασης**της κάμερας (1 GbE ≈ 80 MB/s), και όχι στην ανερχόμενη σύνδεση του κεντρικού υπολογιστή. Για μια συστοιχία 4 καμερών πλήρους ανάλυσης 12-bit, αυτό αντιστοιχεί σε**~2,8 fps**, η τιμή αυτή αντιστοιχεί στις μετρημένες ~2,7–3,0 fps και είναι σκόπιμα**ανεξάρτητη από την έκθεση**, οπότε σε σκηνές με χαμηλό φωτισμό η πραγματική τιμή μπορεί να πέσει ελαφρώς κάτω από το ανώτατο όριο καθώς η έκθεση επιμηκύνεται. Η σειριακή ανάκτηση αποτελεί τον πραγματικό περιοριστή των fps· ο παραλληλισμός της θα αύξανε το ανώτατο όριο προς τον ρυθμό εκπομπής μίας κάμερας.
- **Η συνολική υπερ-εγγραφή αποτελεί σοβαρό εμπόδιο στη σύνδεση.**Η κατανομή εύρους ζώνης ανά κάμερα έχει κατώτατο όριο στα**8 MB/s**(`ARRAY_PER_CAM_FLOOR_BPS`), οπότε μόλις το κατώτατο όριο σταθεροποιηθεί, η συνολική ζήτηση (`per_cam × N`) μπορεί να υπερβεί το**ανώτατο όριο ασφαλούς από συγκρούσεις**(`sustained × sim_emit_factor`). Πρακτικά ανώτατα όρια πλήρους ανάλυσης σε 1 GbE:**6 κάμερες στα 1500 MTU, 9 με jumbo**. Αυτό το ανώτατο όριο εξαρτάται αποκλειστικά από το καλώδιο και το κατώτατο όριο — είναι**ανεξάρτητο από το μέγεθος του πλαισίου**, οπότε**η ομαδοποίηση και η μικρότερη περιοχή ενδιαφέροντος (ROI) ΔΕΝ βοηθούν** (μειώνουν τα byte ανά *πλαίσιο*, όχι τα byte ανά *δευτερόλεπτο* που καθορίζονται από το GevSCPD)· οι μόνες λύσεις είναι λιγότερες κάμερες, πλαίσια jumbo από άκρο-end, ή μια ταχύτερη κάρτα δικτύου. Το σύμπτωμα θα ήταν απώλεια πακέτων GVSP, όχι μια ομαλή μείωση των fps, οπότε το `analyze-array` μηδενίζει τις τιμές των εφικτών fps και εμφανίζει το `**OVER-SUBSCRIBED**`, ενώ το `array-connect` με σταθερή ανάλυση **αρνείται να συνδεθεί** (διαφορετικά, η διαδικασία «walk-down» ομαδοποιεί τα πλαίσια σε χαμηλότερη ανάλυση, κάτι που δεν επιλύει ούτε αυτό το είδος μπλοκ). Το `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` υποβαθμίζει την άρνηση σύνδεσης σε μια έντονη προειδοποίηση για εργασίες σε εργαστηριακό περιβάλλον — βλ. [Μεταβλητές περιβάλλοντος](#environment-variables).

### Κατάσταση συστοιχίας — ποιο υποσύστημα χάνει καρέ

Το `GET /api/camera/array/<array_id>/capability` ενός συνδεδεμένου πίνακα φέρει ένα ενεργό
μπλοκ `health`, το οποίο επαναξιολογείται σε ένα κυλιόμενο παράθυρο **10 δευτερολέπτων**. Διαχωρίζει την απώλεια πλαισίων
στις δύο αιτίες που απαιτούν αντίθετες διορθώσεις, αντί να αναφέρει ένα «ατελές»
ποσοστό που δεν προσδιορίζει καμία από τις δύο:

| Πεδίο | Τι σημαίνει | Ποιο υποσύστημα |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (ανά σειριακό) | Το πλαίσιο **έφτασε αλλά ήταν δομικά ελαττωματικό**— απώλεια πακέτων GVSP. |**Δίκτυο**: διαθέσιμη χωρητικότητα καλωδίου, ρυθμός, δακτύλιος λήψης NIC, MTU |
| `never_arrived_rate_pct` (ανά σειριακό) | Το πλαίσιο **δεν έφτασε καθόλου**— η κάμερα δενενεργοποιήθηκε, ή δεν έστειλε τίποτα. |**Ενεργοποίηση / συγχρονισμός**: καλώδιο M8, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Το χειρότερο ποσοστό κάθε κάμερας. | — |
| `per_cam_rate_pct` | Συνολικό ποσοστό ατελούς λειτουργίας ανά κάμερα (και οι δύο αιτίες μαζί). | — |
| `stable_for_seconds` | Πόσο καιρό κάθε κάμερα παρέμεινε κάτω από το 0,01 %. | — |

Πάνω από το 5 %, το backend καταγράφει μια γραμμή `[array-health <id>] WARN` που ονομάζει τη διαίρεση — κατά την
πρώτη παραβίαση, κατά την αλλαγή ζώνης σοβαρότητας, μία φορά το λεπτό όσο διαρκεί η κατάσταση, και μία φορά όταν
εξαλειφθεί. Το κατεστραμμένο μισό εκτυπώνει το `[gvsp-corrupt <SN>]` κατά την πρώτη εμφάνιση ανά κάμερα και
λόγο, και στη συνέχεια μια συγκεντρωτική καταγραφή κάθε 60 δευτερόλεπτα. Κάθε αξιολόγηση εξακολουθεί να καταγράφεται στο αρχείο καταγραφής του backend·
οι μετρητές αυξάνονται για κάθε buffer ανεξάρτητα από το τι εμφανίζεται.

Η ίδια εγγραφή αναφέρει τον αριθμό από τον οποίο εξαρτάται ολόκληρη η κατανομή:

| Πεδίο | Τι σημαίνει |
| --- | --- |
| `wire_ceiling_mbps` | Ο ισχύων προϋπολογισμός εύρους ζώνης του κεντρικού υπολογιστή, σε MB/s. |
| `wire_ceiling_source` | Από πού προήλθε αυτός ο αριθμός, με λέξεις — π.χ. `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` ή `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` όταν το `--wire-ceiling-mbps` (ή το πεδίο **Wire Budget** του GUI) το έχει ορίσει. |
| `nic_is_usb` | `true` για έναν προσαρμογέα USB Ethernet — δείτε το όριο των 200 MB/s παραπάνω. |

**Ερμηνεία:** ένα μη μηδενικό `gvsp_corrupt_rate_pct` με το `never_arrived_rate_pct` στο 0
σημαίνει ότι η ενεργοποίηση και ο συγχρονισμός του καλωδίου είναι τέλειοι και το 100 % της απώλειας οφείλεται στη διαδρομή του δικτύου — μειώστε το `--wire-ceiling-mbps` και επανασυνδεθείτε. Το αντίστροφο μοτίβο υποδεικνύει το
καλώδιο συγχρονισμού ή τη γραμμή ενεργοποίησης.

> **Το `--target-fps` δεν αποτελεί τον παράγοντα για τα κατεστραμμένα πλαίσια.** Ο ρυθμός GevSCPD καταγράφεται
> μία φορά κατά τη σύνδεση, οπότε η μείωση του ρυθμού ενεργοποίησης αλλάζει τον κύκλο λειτουργίας και όχι τον
> ρυθμό εκπομπής ταυτόχρονων ριπών. Μια μετρημένη μείωση της ζήτησης κατά 5× δεν επέφερε καμία βελτίωση·
> η μείωση του ανώτατου ορίου καλωδίου από 240 σε 200 MB/s μείωσε το ποσοστό κατεστραμμένων δεδομένων στην ίδια διάταξη από 10,4 %
> σε 0,00 %.

> **Η αυτόματη συρρίκνωση εν μέσω ροής δεν είναι διαθέσιμη στο firmware του TRI032S.** Ένας συστοιχία που βρίσκεται σε λειτουργία
> δεν μπορεί να διορθώσει αυτό το πρόβλημα από μόνη της· αποσυνδέστε και επανασυνδέστε, ώστε ο επιλογέας χρόνου σύνδεσης να μπορεί
> να επαναπρογραμματίσει με το νέο ανώτατο όριο.

### Σύμπτωμα → διόρθωση

| Σύμπτωμα (Ρυθμίσεις συστοιχίας / σύνδεση / `analyze_array_network`) | Αιτία | Διόρθωση |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | Το `ReceiveBufferLen` επαναφέρεται στο 32 (συνήθως μετά από ενημέρωση προγράμματος οδήγησης) | Ορίστε το `ReceiveBufferLen`→256, το `PendingReceives`→64· ανοίξτε ξανά τον πίνακα ελέγχου (επανεκκινήστε το backend αν έχει αποθηκεύσει στην κρυφή μνήμη το παλιό μέγεθος δακτυλίου) |
| Κολλάει κατά την επανεκκίνηση/απενεργοποίηση· αργότερα εμφανίζονται σφάλματα WMI `Invalid class`, η κάρτα δικτύου δεν ενεργοποιείται, λείπουν οι μονάδες USB | Παλιό πρόγραμμα οδήγησης Realtek USB 10GbE του 2016 → BSOD `0x9F` → αναγκαστικές απενεργοποιήσεις | Ενημερώστε το πρόγραμμα οδήγησης του προσαρμογέα σε έκδοση ≥ v10.67 (2026) και, στη συνέχεια, εφαρμόστε ξανά τις παραπάνω που αναφέρονται παραπάνω |
| Η σύνδεση επιτυγχάνεται αλλά επιστρέφει ανάλυση χαμηλότερη από την εγγενή | Η λειτουργία Smart-prep συρρίκνωσε αυτόματα το πλαίσιο για να ταιριάζει στο καλώδιο | Αναβάθμιση της σύνδεσης / αποδεχτείτε τη συρρίκνωση / `--force-tier slip-emit-and-capture` |
| Το Array αναφέρει έναν ικανοποιητικό στόχο fps αλλά παρέχει μόνο ένα κλάσμα αυτού· `health.gvsp_corrupt_rate_pct` διαφορετικό από μηδέν, `never_arrived_rate_pct` 0 | Ο υπολογισμένος προϋπολογισμός καλωδίου του κεντρικού υπολογιστή υπερεκτιμά την πραγματική χωρητικότητα (συνηθισμένο σε προσαρμογέα USB Ethernet, στενή λωρίδα PCIe ή κοινόχρηστο δίκτυο) | Επανασυνδεθείτε με χαμηλότερο `--wire-ceiling-mbps` και-ελέγξτε ξανά το μπλοκ κατάστασης. **Όχι** `--target-fps` — ο ρυθμός GevSCPD είναι σταθερός κατά τη σύνδεση |
| Λείπουν κάμερες από τις δημοσιευμένες ομάδες· `health.never_arrived_rate_pct` διαφορετικό από μηδέν, `gvsp_corrupt_rate_pct` 0 | Διαδρομή ενεργοποίησης/συγχρονισμού — οι κάμερες δεν ενεργοποιούνται, δεν είναι πρόβλημα δικτύου | Ελέγξτε το καλώδιο συγχρονισμού M8 και το `--line`· βεβαιωθείτε ότι κάθε μέλος είναι ενεργοποιημένο (`TriggerMode=On`) |
| Τα `**OVER-SUBSCRIBED**` / `Wire budget` έχουν ξεπεραστεί στο `analyze-array`, ή απόρριψη σύνδεσης με καθορισμένη ανάλυση (`array over-subscribes the wire`) | Η συνολική ζήτηση ανά κάμερα (ελάχιστο όριο 8 MB/s × N κάμερες) υπερβαίνει το ανώτατο όριο ασφαλούς λειτουργίας χωρίς συγκρούσεις — 6 κάμερες σε πλήρη ανάλυση σε 1 GbE @1500 MTU, 9 με jumbo | Λιγότερες κάμερες, jumbo frames από άκρο σε άκρο ή ταχύτερη κάρτα δικτύου. **Το ROI/binning ΔΕΝ θα βοηθήσει** (το ανώτατο όριο είναι ανεξάρτητο από το μέγεθος του πλαισίου). Το `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` παρακάμπτει τον περιορισμό στο εργαστήριο (αποδέχεται την απώλεια πακέτων) |

---

## `chloros-cli daq`

Εντολές αισθητήρα φάσματος. Δύο κατηγορίες:
- **`pool-*`**— «λεπτοί» πελάτες HTTP που ελέγχουν τον αισθητήρα μέσω της μόνιμης ομάδας (persistent pool) του backend.**Αυτή είναι η υποστηριζόμενη διαδρομή και η μόνη που υπάρχει στο παρεχόμενο CLI.** Το backend διαχειρίζεται τη μεταφορά, οπότε το GUI, τα σενάρια CLI και SDK μοιράζονται όλα ένα ενεργό χειριστή αντί να ανταγωνίζονται για τη σειριακή θύρα.
- **Όλα τα υπόλοιπα**(`test`, `record`, `live`, `stream`, `connect`, `info`, `net`, `ota`, `sample-rate`, `calibrate`, `serve`, `ws`, `udp`, `mqtt`, `reflectance`, `login`, `logout`, `status`) — άμεση πρόσβαση σε υλικό, που τεκμηριώνεται παρακάτω για λόγους πληρότητας. Αυτά απαιτούν το πακέτο XPython, το οποίο**δεν περιλαμβάνεται σε κανένα διαθέσιμο πακέτο**: το μεταγλωττισμένο CLI το εξαιρεί (το `scripts/Build-CLI.ps1` ορίζει το `--nofollow-import-to=daq`, και τα μεταφορικά `pyserial` / `bleak` / `zeroconf` το περιλαμβάνουν), και ούτε το πακέτο PyPI SDK το περιέχει. Λειτουργούν μόνο από ένα checkout πηγαίου κώδικα, οπότε αντιμετωπίστε τα ως μια εσωτερική διαδρομή ανάπτυξης του MAPIR και όχι ως κάτι προς το οποίο πρέπει να στραφείτε.
- **`discover` / `list`** βρίσκονται μεταξύ των δύο: είναι εντολές απευθείας προς το υλικό από ένα checkout πηγαίου κώδικα, αλλά σε μια έκδοση που έχει κυκλοφορήσει επιστρέφουν στο `pool-discover` και το backend εκτελεί τη σάρωση. Έτσι, η σάρωση λειτουργεί παντού — κάτι που έχει σημασία, καθώς είναι ο μόνος τρόπος για να μάθετε το BLE MAC ενός DAQ-M.

> **`chloros-cli daq --help`** (και `-h` / `help`) παραθέτει τις υποεντολές του `pool-*` — η βοήθεια δρομολογείται σκόπιμα στον πελάτη του pool, ώστε να αντανακλά τις εντολές που εκτελούνται πραγματικά. Εάν καλέσετε μια υποεντολή άμεσης πρόσβασης στο υλικό σε μια έκδοση που έχει διατεθεί, η διαδικασία τερματίζεται με ένα ρητό σφάλμα που αναφέρει το πακέτο που λείπει και σας παραπέμπει πίσω στο `pool-*`· τίποτα δεν αποτυγχάνει σιωπηλά. (Οι εντολές `discover` / `list` αποτελούν εξαίρεση — ανακατευθύνουν στο `pool-discover` και απλώς λειτουργούν.)
>
> **Ό,τι χρειάζεται ένας πελάτης είναι προσβάσιμο μέσω του `pool-*`** — σύνδεση, ροή δεδομένων, εγγραφή βαθμονομημένων αρχείων `.daq` και εναλλαγή προφίλ καπακίων. Το DAQ μπορεί επίσης να ελέγχεται από το Python με το `chloros_sdk.connect_daq_sensor()`, το οποίο χρησιμοποιεί την ίδια κοινή διαδρομή.

### Ροή εργασίας πρώτης σύνδεσης αισθητήρα DAQ

```bash
# 1. Smart-detect any DAQ on this machine (Ethernet → BLE → USB precedence)
chloros-cli daq connect

# 2. Detailed scan: every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U COM port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq discover                      # or: daq pool-discover
chloros-cli daq discover --only ble           # BLE only
chloros-cli daq discover --json               # machine-readable

# 3. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect           # smart-detect
chloros-cli daq pool-connect --port COM3                       # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 4. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 5. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 6. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 7. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### Αναφορά `pool-*`

| Υποεντολή | Σκοπός |
| --- | --- |
| `daq pool-connect` (smart-detect) | Άνοιγμα αισθητήρα στην ομάδα του backend. |
| `daq pool-connect --port PORT` | DAQ-U σε συγκεκριμένη σειριακή θύρα. |
| `daq pool-connect --ble` | DAQ-M μέσω BLE, αυτόματη σάρωση MAC. |
| `daq pool-connect --mac MAC` | DAQ-M μέσω BLE σε γνωστή διεύθυνση MAC (υπονοεί την εντολή `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E μέσω Ethernet σε γνωστό κεντρικό υπολογιστή. |
| `daq pool-connect --eth` | DAQ-E μέσω Ethernet, αυτόματη-εντοπισμός (mDNS + εφεδρικό ARP· λειτουργεί από κενή κρυφή μνήμη ARP στα Windows και Linux). |
| `daq pool-connect --integration-time MS --frame-avg N --no-ae` | Ρύθμιση παραθύρου ενσωμάτωσης / κατάστασης AE. |
| `daq pool-connect --no-stream` | Σύνδεση χωρίς έναρξη ροής δεδομένων (συνδυάζεταιμε το `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Προφίλ διόρθωσης Cap. Η προεπιλογή στο backend είναι το `sunshine_cosine`. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Σάρωση κάθε διαύλου μεταφοράς για αισθητήρες στους οποίους θα μπορούσατε να συνδεθείτε, χωρίς να πραγματοποιήσετε σύνδεση. **Έτσι εντοπίζετε τη διεύθυνση MAC BLE ενός DAQ-M.** `daq discover` / `daq list` δρομολογούνται αυτόματα εδώ στις έτοιμες εκδόσεις. Οι αισθητήρες που είναι ήδη ανοιχτοί στη λίστα δεν εμφανίζονται — ένα συνδεδεμένο DAQ-M σταματά να διαφημίζεται — γι&#x27; αυτό χρησιμοποιήστε το `pool-list` για αυτούς. |
| `daq pool-list` | Εμφάνιση κάθε αισθητήρα στην ομάδα του backend. |
| `daq pool-disconnect --sensor-id ID [--all]` | Απελευθέρωση. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Τα πιο πρόσφατα N πλαίσια φάσματος. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Συνέχιση / παύση ροής δεδομένων. |
| `daq pool-record --sensor-id ID [--duration SEC] [--output DIR] [--device-name NAME] [--stop]` | Έναρξη / διακοπή εγγραφής .daq. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Αλλαγή του προφίλ διόρθωσης κεφαλαίων κατά την εκτέλεση. |

### Υποεντολές άμεσης πρόσβασης στο υλικό (μόνο για έλεγχο πηγαίου κώδικα — δεν περιλαμβάνονται στις διαθέσιμες εκδόσεις)

> Αναφέρονται για λόγους πληρότητας. Αυτές απαιτούν το πακέτο `daq` Python καθώς και τα `pyserial` / `bleak` / `zeroconf`, καμία από τις οποίες δεν περιλαμβάνεται στις συμπιεσμένες εκδόσεις CLI ή στο PyPI SDK — εκτελούνται μόνο από το MAPIR source checkout. **Εάν χρησιμοποιείτε μια κυκλοφορημένη έκδοση Chloros, χρησιμοποιήστε αντ&#x27; αυτού τις παραπάνω εντολές `pool-*`**· αυτές καλύπτουν τη σύνδεση, τη ροή, την εγγραφή και την επιλογή ορίου.

```bash
chloros-cli daq test --port COM3                           # Verify connection
chloros-cli daq connect --eth                              # Smart-detect over ETH
chloros-cli daq info --eth-host daq-e-xxx.local            # Device summary as JSON
chloros-cli daq discover --only usb,ble --timeout 5        # Scan local interfaces
chloros-cli daq list                                       # Alias of discover
# ^ discover/list are the exception in this section: in a shipped build they
#   fall back to `pool-discover` (the backend does the scan), so they work
#   without a source checkout. The only difference is that the fallback needs
#   the Chloros backend running, as all pool-* commands do.

# Streaming JSON Lines to stdout (pipeable)
chloros-cli daq stream --port COM3 --format jsonl --photometrics

# Record to .daq for 60 seconds
chloros-cli daq record --port COM3 --duration 60 -o ~/Documents/spectra/

# Live spectrum visualization in a window
chloros-cli daq live --port COM3 --record

# Dual-sensor reflectance (ambient + object) → JSON Lines
chloros-cli daq reflectance \
  --ambient-eth-host daq-e-field.local \
  --object-eth-host daq-e-canopy.local \
  --record -o ~/Documents/reflectance/

# Convenience: pick integration_time + frame_avg for a target rate
chloros-cli daq sample-rate --port COM3 --target-hz 5

# Calibration profile management
chloros-cli daq calibrate --port COM3 --list
chloros-cli daq calibrate --port COM3 --set field_calibration_2026_05

# DAQ-E network config (mDNS auto-discovers the host)
chloros-cli daq net --eth-host daq-e-xxx.local set-ip --mode static --ip 192.168.2.20
chloros-cli daq net --eth-host daq-e-xxx.local set-name "sky-sensor"
chloros-cli daq net --eth-host daq-e-xxx.local set-ptp --enabled true --domain 0
chloros-cli daq net --eth-host daq-e-xxx.local set-auto-stream true          # auto-stream on boot
chloros-cli daq net --eth-host daq-e-xxx.local set-require-signature         # require factory-signed cal (fw v1.6.0+; refused while the held cal is unsigned)
chloros-cli daq net --eth-host daq-e-xxx.local set-time                      # push host clock (refused when PTP SLAVE)
chloros-cli daq net --eth-host daq-e-xxx.local set-auth-token --current "" --new "s3cret"   # control-channel auth ("" new = disable)
chloros-cli daq net --eth-host daq-e-xxx.local set-ota-password "newpass"    # change OTA password (min 4 chars)
chloros-cli daq net --eth-host daq-e-xxx.local factory-reset                 # clear all NVS settings and reboot
chloros-cli daq net --eth-host daq-e-xxx.local reboot

# OTA firmware update
chloros-cli daq ota --eth-host daq-e-xxx.local \
  --firmware daq_e_1.21.bin --password mapir-daq-e

# Bridge spectra to other protocols
chloros-cli daq serve --port COM3 --tcp-port 9000           # TCP JSON-lines
chloros-cli daq ws    --port COM3 --ws-port 9001            # WebSocket
chloros-cli daq udp   --port COM3 --udp-port 9002           # UDP broadcast
chloros-cli daq mqtt  --port COM3 --broker mqtt.example.com --topic daq/spectrum
```

---

## `chloros-cli project`

Άνοιγμα, σύνδεση, και εκτελέστε ένα αποθηκευμένο έργο Chloros (ένας φάκελος με τα `cameras.json` + `sensors.json` + `project.json`). Όλα δρομολογούνται μέσω του backend, οπότε το GUI και το CLI παράγουν την ίδια κατάσταση υλικού.

### Αναφορά υποεντολών

| Υποεντολή | Σκοπός |
| --- | --- |
| `project open PATH` | Εκτύπωση του καταλόγου συσκευών του έργου (κάμερες, συστοιχίες, αισθητήρες). |
| `project devices PATH [--reconnect]` | Εμφάνιση λίστας ή επανεκτέλεση ανίχνευσης. |
| `project connect PATH [--cameras-only] [--sensors-only]` | Σύνδεση κάθε αποθηκευμένης κάμερας / συστοιχίας / αισθητήρα. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Μία μεμονωμένη λήψη από μια συγκεκριμένη κάμερα ή συστοιχία. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | N-καρέ σε ριπή από μια συγκεκριμένη κάμερα ή συστοιχία (`-n/--count` προεπιλογή 5; `-i/--interval` δευτερόλεπτα μεταξύ καρέ, προεπιλογή 0). Οι ριπές συστοιχιών απομακρύνουν τις επαναλαμβανόμενες συγχρονισμένες ομάδες (ελεγκτής παλαίωσης), ώστε μια συστοιχία με μερικό κύκλο να μην μπορεί να επιστρέψει N αντίγραφα ενός καρέ· εκτυπώνει τα αποτελέσματα ανά επανάληψη. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--poll-interval S]` | Ροή δεδομένων σε δίσκο μέσω εργασίας backend. `--poll-interval` = δευτερόλεπτα μεταξύ των ελέγχων `/stats` (προεπιλογή 2,0). |
| `project sensor read PATH NAME [--json]` | Το πιο πρόσφατο πλαίσιο φάσματος. |
| `project sensor log PATH NAME --seconds SEC [-o DIR] [--device-name NAME]` | Εγγραφή .daq. |
| `project run PATH RECIPE.yaml` | Εκτέλεση συνταγής καταγραφής YAML/JSON. Το `--dry-run` επικυρώνει χωρίς εκτέλεση. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Υπολογισμός ευθυγράμμισης για μια συστοιχία — βλ. [τον πίνακα επιλογών παρακάτω](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Εκτύπωση του τρέχοντος προφίλ ευθυγράμμισης. |
| `project align clear PATH NAME` | Διαγραφή του προφίλ από την προσωρινή μνήμη. |
| `project align tweak PATH NAME --serial SN --dx N --dy N --rotation-deg N --scale N` | Μετατόπιση του μετασχηματισμού ενός δούλου. |
| `project align export PATH NAME --to FILE` | Αποθήκευση του προφίλ στο JSON. |
| `project align import PATH NAME --from FILE [--no-validate]` | Φόρτωση αποθηκευμένου προφίλ. |

#### Επιλογές `project align calibrate`

| Σημαία | Προεπιλογή | Περιγραφή |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Μέθοδος ευθυγράμμισης. **Αυτές οι ονομασίες διαφέρουν από το `lattice align-calibrate`**, η οποία χρησιμοποιεί τις συντομογραφίες `orb` / `akaze` / `phase`· οι δύο εντολές δεν είναι εναλλάξιμες σε αυτήν τη σημαία. |
| `--model {translation, rigid, affine, homography}` | `affine` | Μετασχηματισμός μοντέλου για προσαρμογή. |
| `--frames N` | `1` | Συγχρονισμένα στιγμιότυπα καρέ για υπολογισμό μέσου όρου. |
| `--reference SN` | το κύριο | Σειριακός αριθμός κάμερας αναφοράς· κάθε άλλο μέλος παραμορφώνεται πάνω σε αυτό. |
| `--max-features N` | `5000` | Ανώτατο όριο αριθμού χαρακτηριστικών ORB. |
| `--ratio-threshold F` | `0.75` | Δοκιμή . |
| `--ransac-threshold-px F` | `3.0` | Όριο εσωτερικών σημείων RANSAC. |
| `--min-matches N` | `15` | **Όριο ποιότητας** — απόρριψη της επίλυσης εάν ο αριθμός των αντιστοιχιών εσωτερικών σημείων είναι μικρότερος από αυτόν. |
| `--max-reproj-err-px F` | `4.0` | **Όριο ποιότητας** — απόρριψη της επίλυσης πάνω από αυτό το σφάλμα επαναπροβολής RMS. |
| `--checkerboard RxC` | — | Γεωμετρία πίνακα για το `--method checkerboard`, π.χ. `9x6`. |
| `--name PROFILE` | κενό | Όνομα προφίλ ενσωματωμένο στο αποθηκευμένο JSON. **Δεν είναι το όνομα του πίνακα** — αυτό είναι το θεσιακό `NAME`. |

Οι δύο πύλες ποιότητας είναι ο λόγος για τον οποίο μια βαθμονόμηση μπορεί να επιτύχει στην επίλυση και παρόλα αυτά
να απορριφθεί κατά την αποθήκευση: ένα προφίλ που αποτυγχάνει σε οποιαδήποτε από τις δύο θα προκαλούσε σιωπηρά λανθασμένη καταχώριση σε κάθε
μεταγενέστερη καταγραφή, οπότε απορρίπτεται αντί να αποθηκευτεί.

### Παραδείγματα

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### DSL συνταγών

Το `project run RECIPE.yaml` δέχεται ένα αρχείο YAML ή JSON που περιγράφει μια ακολουθία ενεργειών:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        exposure_auto: "Off"
        gain: 6.0
        gain_auto: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Υποστηριζόμενες ενέργειες: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. Η ενέργεια `burst` απαιτεί τις ενέργειες `name` (απαιτείται), `count` (προεπιλογή 5), `interval` (δευτερόλεπτα, προεπιλογή 0), `output`, `format` και `settings` (ίδια μορφή ρυθμίσεων ανά κάμερα με το `apply`)· οι ράφτες πίνακα χρησιμοποιούν τον ίδιο μηχανισμό παρακολούθησης «fresh-synced-group» με το `project burst`.

Εκτελέστε το:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

---

## Μεταβλητές περιβάλλοντος

| Μεταβλητή | Αποτέλεσμα |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Παρακάμπτει το backend URL (προεπιλογή `http://127.0.0.1:5000`) — **λαμβάνεται υπόψη μόνο από τις οικογένειες εντολών `lattice`, `project` και `daq pool-*`.** Οι βασικές εντολές (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) παρακάμπτουν την `http://127.0.0.1:<port>` και αγνοούν αυτή τη μεταβλητή (η σταθερά IPv4 παρακάμπτει το Windows `localhost`→`::1` ~2 s-per-request penalty), οπότε στοχεύουν πάντα τον τοπικό υπολογιστή. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | Το `1` υποβαθμίζει τη σειρά- απόρριψη σύνδεσης λόγω υπερβολικού αριθμού συνδρομών (συνολική ζήτηση ανά κάμερα &gt; ανώτατο όριο ασφαλούς από συγκρούσεις με το `pin_resolution`) σε μια έντονη προειδοποίηση και συνέχιση, αποδεχόμενο την απώλεια πακέτων GVSP. Μόνο για χρήση σε δοκιμές — δείτε [Μοντέλο fps &amp; burst συστοιχίας](#array-fps--burst-model). |
| `CHLOROS_CLI_MODE` | Ρυθμίζεται από το ίδιο το «CLI»· δίνει εντολή στο backend να ενεργοποιήσει την παράλληλη επεξεργασία. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | Το `0` παραλείπει τον έλεγχο fallback του GVSP (μόνο αποτελέσματα ICMP). **Αυτό απενεργοποιεί τα πακέτα jumbo, δεν περιορίζεται απλώς στη μείωση των καταχωρήσεων στο αρχείο καταγραφής** — η κάμερα απαντά σε ping DF μόνο μέχρι το 1500 σε κάθε διαδρομή, οπότε αυτός ο έλεγχος είναι ο μόνος τρόπος ανίχνευσης πακέτων jumbo. Εξοικονομεί ~1 δευτερόλεπτο ανά κάμερα ανά σύνδεση· κοστίζει ~1,45× το ανώτατο όριο του καλωδίου αν το δίκτυο *μπορούσε* να μεταφέρει πακέτα jumbo. Το SDK προειδοποιεί όταν το ρυθμίζετε. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Καθορίζει το μέγεθος πακέτου GVSP στα N bytes· παραλείπει εντελώς τον έλεγχο. Προτιμήστε τη ρύθμιση ανά εντολή (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) αντί της μόνιμης ρύθμισης: ένα σταθεροποιημένο μέγεθος σταματά να προσαρμόζεται στο δίκτυο που βρίσκεται μπροστά του, και η σταθεροποίηση στα 9000 σε μια διαδρομή που δεν μπορεί να μεταφέρει πακέτα jumbo προκαλεί **κάθε** να λήγει λόγω χρονικού ορίου καταγραφής με το `SC_ERR_TIMEOUT -1011`. |
| `TMPDIR` (Linux) | Παράκαμψη του καταλόγου εξαγωγής ενός αρχείου του Nuitka. Το CLI χρησιμοποιεί αυτόματα το `/mnt/ssd/tmp` αν υπάρχει. |

---

## Κωδικοί εξόδου

| Κωδικός | Σημασία |
| --- | --- |
| `0` | Επιτυχία. |
| `1` | Γενική αποτυχία (τα περισσότερα σφάλματα υποεντολών). |
| `2` | Σφάλμα παραμέτρου. |
| `130` | Διακοπή με Ctrl+C. |

---

## Συμβουλές για την αντιμετώπιση προβλημάτων

- **«Απαιτείται σύνδεση»** → Εκτελέστε το `chloros-cli login EMAIL PASSWORD` μία φορά σε αυτόν τον υπολογιστή.
- **&quot;backend unreachable&quot;** → Ξεκινήστε την εφαρμογή επιφάνειας εργασίας Chloros, ή εκτελέστε απευθείας το εκτελέσιμο του backend (`chloros-backend`), ή ελέγξτε το `CHLOROS_BACKEND_URL` αν πρόκειται για απομακρυσμένη σύνδεση.
- **Οι εντολές `lattice` αποτυγχάνουν με το μήνυμα «Δεν βρέθηκαν προγράμματα οδήγησης κάμερας LATTICE»** → Το περιβάλλον εκτέλεσης Arena SDK δεν είναι εγκατεστημένο· το CLI διατίθεται με το `win32api` ενσωματωμένο στο Windows, αλλά το περιβάλλον εκτέλεσης C αποτελεί μέρος του προγράμματος εγκατάστασης GUI.
- **Στην ενότητα «Σύνδεση πλέγματος» / «Ρυθμίσεις πλέγματος» εμφανίζεται το μήνυμα «FRAMES WILL DROP» ή «Reduce ROI to enable»** → Ο δακτύλιος λήψης της κάρτας δικτύου του κεντρικού υπολογιστή είναι πολύ μικρός (συνήθως επαναρυθμίζεται στο 32 μετά από ενημέρωση του προγράμματος οδήγησης της κάρτας δικτύου). Δείτε [Ρύθμιση &amp; Βελτιστοποίηση Κάρτας Δικτύου Κεντρικού Υπολογιστή](#host-nic-setup--tuning-lattice-arrays) — ορίστε τα `ReceiveBufferLen=256`, `PendingReceives=64`.
- **Ο υπολογιστής κολλάει κατά την επανεκκίνηση/το κλείσιμο, και στη συνέχεια το WMI `Invalid class` / η κάρτα δικτύου δενενεργοποιηθεί / λείπουν μονάδες USB** → Παρωχημένο πρόγραμμα οδήγησης προσαρμογέα USB 10GbE που προκαλεί το σφάλμα `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Ενημερώστε το πρόγραμμα οδήγησης του προσαρμογέα — δείτε [Ρύθμιση &amp; Βελτιστοποίηση Κάρτας Δικτύου Κεντρικού Υπολογιστή](#host-nic-setup--tuning-lattice-arrays).
- **Προειδοποίηση ανταλλαγής Jetson** → Προσθέστε ανταλλαγή που υποστηρίζεται από αρχείο· το αρχείο CLI εμφανίζει τις ακριβείς εντολές `fallocate` / `swapon`.
- **Λείπουν οι άμεσες εντολές DAQ** → Όπως αναμενόταν: το παρεχόμενο `chloros-cli` αποκλείει σκόπιμα το πακέτο `daq`, οπότε υπάρχει μόνο το `pool-*` (ούτε το PyPI SDK το περιλαμβάνει). Χρησιμοποιήστε το `pool-*`, το οποίο ελέγχει τον ίδιο αισθητήρα μέσω του backend, ή το `chloros_sdk.connect_daq_sensor()` από το Python.

---

## Δείτε επίσης

- [Python SDK Αναφορά](sdk-reference.md) — προγραμματιστικό ισοδύναμο κάθε εντολής CLI.
- [Οδηγός αισθητήρων DAQ](../daq/README.md) — καλωδίωση και βαθμονόμηση ανά αισθητήρα.
- Ηλεκτρονική τεκμηρίωση: `https://mapir.gitbook.io/chloros/cli`
