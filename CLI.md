# CLI : Γραμμή εντολών

> **Πλήρης αναφορά:**[CLI Αναφορά](reference/cli-reference.md) καταγράφει**κάθε παράμετρο κάθε υποεντολής** και είναι βελτιστοποιημένη για βοηθούς τεχνητής νοημοσύνης — επικολλήστε το URL στον βοηθό σας και ζητήστε μια λειτουργική εντολή: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Συμβουλή για εργαλεία τεχνητής νοημοσύνης:** οποιαδήποτε σελίδα αυτού του εγχειριδίου είναι διαθέσιμη ως ακατέργαστο Markdown προσθέτοντας το `.md` στη διεύθυνσή της URL (π.χ. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), ενώ το `https://mapir.gitbook.io/chloros/llms.txt` δημιουργεί ευρετήριο ολόκληρου του εγχειριδίου για χρήση από μοντέλα μεγάλης γλώσσας (LLM).

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.2.0 build so the version line reads "Chloros CLI 1.2.0" -->


## Τι είναι τοCLI


Το `chloros-cli` είναι το περιβάλλον γραμμής εντολών για τον ίδιο μηχανισμό επεξεργασίας που χρησιμοποιεί η εφαρμογή επιφάνειας εργασίαςChloros
. Πρόκειται για έναν «λεπτό» πελάτη (HTTP
) που λειτουργεί πάνω από το backendChloros
(έναν τοπικό διακομιστή στο `127.0.0.1:5000`) — οι περισσότερες εντολές εκκινούν αυτόματα το backend, οπότε μια απλή κλήση του `chloros-cli process …` είναι το μόνο που χρειάζεται ένα σενάριο.

Λειτουργεί σε **Windows
10/11 (x64)**και**Linux
(x86_64, καθώς και NVIDIA Jetson arm64 στο JetPack 6)**, σε οποιοδήποτε τερματικό, χωρίς να απαιτείται GUI. Επαληθεύστε την εγκατάστασή σας με:

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```

Οι οικογένειες εντολών, με μια ματιά:

* **Επεξεργασία &amp; λογαριασμός** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 γλώσσες — δείτε [Υποστηριζόμενες γλώσσες](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (μόνο γιαLinux
/Jetson)
* **Ζωντανό υλικό** — `lattice` (έλεγχος κάμερας LATTICE, 45+ υποεντολές), `daq pool-*` (αισθητήρες φωτός DAQ), `time-sync` (PTP)
* **Αυτοματοποίηση** — `project` (εκτέλεση αποθηκευμένου έργουChloros
χωρίς οθόνη, συμπεριλαμβανομένων συνταγών καταγραφής YAML)

Παράμετροι συστηματικές που αξίζει να γνωρίζετε: `--port N` (θύρα backend, προεπιλογή `5000`), `-v/--verbose`, `--restart` (επιβολή επανεκκίνησης του backend), `--backend-exe PATH`. Ανατρέξτε στην [Αναφορά τουCLI
](reference/cli-reference.md) για τον πλήρη κατάλογο.

***

## Εγκατάσταση

ΤοCLI
**περιλαμβάνεται στο πρόγραμμα εγκατάστασης τουChloros** σε όλες τις πλατφόρμες — δεν υπάρχει ξεχωριστή λήψη τουCLI
. Κατεβάστε το πρόγραμμα εγκατάστασης από τη σελίδα [Λήψη](download.md).

###Windows


Το πρόγραμμα εγκατάστασης τοποθετεί τοCLI
στη διαδρομή:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

και προσθέτει αυτόν τον φάκελο στο σύστημά σας `PATH` — **ανοίξτε ένα νέο τερματικό**μετά την εγκατάσταση, ώστε να αναγνωριστεί το ενημερωμένο `PATH`. Το πρόγραμμα εγκατάστασης τοποθετεί επίσης σενάρια εκκίνησης (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) στον ριζικό κατάλογο εγκατάστασης, καθώς και μια**συντόμευση στο μενού «Έναρξη» (Chloros
)CLI
** συντόμευση στο μενού «Έναρξη», καθεμία από τις οποίες ανοίγει ένα τερματικό με το `chloros-cli` έτοιμο για χρήση.

###Linux


Εγκαταστήστε το `.deb` για την αρχιτεκτονική σας:

```bash
# Linux x86_64
sudo dpkg -i chloros-amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Αυτό εγκαθιστά το `chloros-cli` έως το `/usr/bin/chloros-cli` (ήδη σε `PATH`) και το backend σε `/usr/lib/chloros/chloros-backend`, μαζί με το περιβάλλον εκτέλεσης ArenaSDK
που απαιτείται για τις κάμερες LATTICE. Δείτε την [Εγκατάσταση τουLinux
](linux/linux-installation.md) για λεπτομέρειες.

### Επαλήθευση

```bash
chloros-cli --version    # "Chloros CLI 1.2.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Σύνδεση &amp; Άδεια χρήσης

CLI
(καιPython
SDK
) η πρόσβαση απαιτεί **πληρωμένο πρόγραμμαChloros
+**— διατίθεται σε οποιοδήποτε πληρωμένο επίπεδο, αλλά όχι στο δωρεάν. Το όριο επιβάλλεται**από την πλευρά του διακομιστή** από το backend, όχι από το εκτελέσιμο αρχείοCLI
: μια κλήση χωρίς σύνδεση απορρίπτεται με κωδικό σφάλματος `401 AUTH_REQUIRED`, ενώ μια κλήση από συνδεδεμένο χρήστη στο δωρεάν πακέτο απορρίπτεται με το σφάλμα `403 PLAN_UPGRADE_REQUIRED`, ανεξάρτητα από το αν προέρχεται από το `chloros-cli`, τοSDK
ή έναν χειροκίνητα δημιουργημένο πελάτηHTTP
. Αναβάθμιση στο [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Συνδεθείτε **μία φορά ανά υπολογιστή**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.2.0 build showing the license tier line -->


{% hint style="warning" %}
**Κωδικοί πρόσβασης με ειδικούς χαρακτήρες**(ο κωδικός `$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` αλλοιώνεται από το shell (τοCLI
το ανιχνεύει με κωδικό σφάλματος 401 και επαναλαμβάνει αυτόματα την προσπάθεια, αλλά τα απλά εισαγωγικά αποφεύγουν εντελώς το πρόβλημα).
{% endhint %}

Η συνεδρία αποθηκεύεται προσωρινά στο `~/.chloros/user_session.json` και συνεχίζει να λειτουργεί εκτός σύνδεσης για την περίοδο χάριτος του προγράμματος (30 ημέρες για τα μηνιαία προγράμματα, μέχρι τη λήξη για τα ετήσια). Το `chloros-cli status` λειτουργεί ακόμη και χωρίς πληρωμένο πακέτο, οπότε ο λόγος απόρριψης είναι πάντα ορατός.

{% hint style="danger" %}
**Προγραμματίζετε εργασίες χωρίς διεπαφή χρήστη; Συνδεθείτε πρώτα.**Μια εντολή δημιουργίας backend (`process`, `status`, `export-status`, …) που εκτελείται**χωρίς αποθηκευμένη συνεδρία**δεν αποτυγχάνει γρήγορα — μεταβαίνει σε μια διαδραστική γραμμή εντολών `Email:` / `Password:` στο stdin. Επομένως, μια αυτόματη εργασία cron ή ένα βήμα CI θα**κολλήσει περιμένοντας είσοδο**. Εκτελέστε το `chloros-cli login EMAIL 'PASSWORD'` μία φορά στον υπολογιστή πριν προγραμματίσετε οτιδήποτε.
{% endhint %}

***

## Η πρώτη σας εκτέλεση επεξεργασίας

Κατευθύνετε το `process` σε έναν φάκελο με καταγραφές — ανιχνεύει αυτόματα τα αρχείαSurvey3
(`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` ή ένα μείγμα:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Η πρόοδος μεταδίδεται ζωντανά ανά νήμα του αγωγού (Ανίχνευση, Ανάλυση, Επεξεργασία, Εξαγωγή), και μια επιτυχημένη εκτέλεση ολοκληρώνεται με την αναφορά του αριθμού των προϊόντων εικόνας που αποθηκεύτηκαν (`Image products written: N`).



<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N" summary line -->
### Πού αποθηκεύονται τα αποτελέσματα

Το `process` εγγράφει σε έναν **φάκελο έργου**, όχι στον φάκελο εισόδου σας:

* Χωρίς `-o`: το έργο δημιουργείται στον προεπιλεγμένο φάκελο έργων σας (κοινός με το GUI· διαχειριστείτε τον με `get-project-folder` / `set-project-folder`, εφεδρικό `~/Chloros Projects`), με όνομα που καθορίζεται από το `-n/--project-name` ή μια χρονική σήμανση (`YYYYMMDD_HHMMSS`) όταν παραλείπεται.
* Με το `-o PATH`: αυτός ο φάκελος **είναι** ο φάκελος του έργου. Εάν περιέχει ήδη ένα `project.json`, δημιουργείται ένας παράλληλος φάκελος με κατάληξη `_1`/`_2`… αντί να γίνει αντικατάσταση.

Μέσα στο έργο, τα προϊόντα ομαδοποιούνται **ανά κάμερα και, στη συνέχεια, ανά μορφή αρχείου**:

```
<project>/
├── project.json
├── calibration_data.json
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Ο φάκελος της κάμερας είναι `LATT-<sensor>-<lens>-F<filter>` για το LATTICE (που αντιστοιχεί στο EXIF της λήψης `Model`) και `<model>_<filter>` (π.χ. `Survey3N_RGN`) για τοSurvey3
. Ο φάκελος μορφής ακολουθεί το `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` ή `tiff32` για το `TIFF (32-bit, Percent)`.

{% hint style="info" %}
**Κάθε εξαγόμενο προϊόν διατηρεί το όνομα του αρχείου ΠΗΓΗΣ.**Μια εξαγωγή radiance του `capture_..._raw.tif` εξακολουθεί να ονομάζεται `capture_..._raw.tif` — απλώς βρίσκεται στο `tiff32/Radiance_Images/`.**Ο φάκελος προσδιορίζει το προϊόν, όχι το όνομα αρχείου**, οπότε χρησιμοποιήστε γενική αναζήτηση για τον κατάλογο, όχι για την κατάληξη `*radiance*`.
{% endhint %}

### Οι επιλογές που θα χρησιμοποιήσετε στην πράξη

| Επιλογή | Προεπιλογή | Τι κάνει |
| --- | --- | --- |
| `-o, --output PATH` | προεπιλεγμένος φάκελος έργου | Θέση φακέλου έργου (βλ. παραπάνω). |
| `-n, --project-name NAME` | χρονική σήμανση | Όνομα έργου. |
| `--format FMT` | `TIFF (16-bit)` | Ένα από τα `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | κανένας | Δείκτες βλάστησης προς εξαγωγή (βλ. [Δείκτες βλάστησης](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = νευρωνική απο-Bayer, πιο αργή, υψηλότερη ποιότητα (Chloros
+, GPU NVIDIA). |
| `--vignette / --no-vignette` | ενεργοποιημένο | Διόρθωση βινιέτας. |
| `--reflectance / --no-reflectance` | ενεργοποιημένο | Βαθμονόμηση ανακλαστικότητας· για το LATTICE, αυτό είναι επίσης το διακόπτης ενεργοποίησης/απενεργοποίησης του προϊόντος ανακλαστικότητας. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Επιβολή του σημείου εισόδου της ροής επεξεργασίας για αρχεία TIFF του LATTICE. |

Για όλα τα υπόλοιπα — ρύθμιση ανίχνευσης στόχου, PPK, πινέζες έκθεσης, σημαίες ευθυγράμμισης πινάκων — ανατρέξτε στην [ενότητα `process` του Οδηγού αναφοράς τουCLI
](reference/cli-reference.md).

***

## Επιλογή των στοιχείων προς εξαγωγή (Προϊόντα LATTICE)

Η επεξεργασία LATTICE διακλαδώνεται σε **κάθε εφαρμόσιμο προϊόν σε μία μόνο διέλευση**. Οι τέσσερις διακόπτες ανά προϊόν είναι όλοι**ενεργοποιημένοι από προεπιλογή**· χρησιμοποιήστε τη φόρμα `--no-` για να απενεργοποιήσετε έναν:

| Διακόπτης | Προϊόν |
| --- | --- |
| `--debayered` | Γραμμική αποσύνθεση → `Debayered_Images/` |
| `--preview` | Προεπισκόπηση οθόνης (ισορροπία λευκού + γάμμα· επέκταση ψευδοχρωμάτων για πολυφασματικά) → `Preview_Images/` |
| `--radiance` | ακτινοβολία float32, W/m²/sr/nm → `Radiance_Images/` (πάντα `tiff32/`) |
| `--reflectance` | uint16 ανακλαστικότητα, έτοιμη για Pix4D → `Reflectance_Calibrated_Images/` |

RGB
οι κύριες κάμερες εκπέμπουν πάντα μόνο δε-bayered + προεπισκόπηση — η ακτινοβολία/ανακλαστικότητα ανά ζώνη δεν έχει νόημα για έναν ευρυζωνικό αισθητήρα, οπότε αυτές οι επιλογές δεν έχουν καμία επίδραση για αυτές.Survey3
Το `.raw` αγνοεί τις επιλογές και ακολουθεί την τυπική διαδρομή ανακλαστικότητας/στόχου.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (προεπιλογή `auto`) επιλέγει την αναφορά ανακλαστικότητας: το `auto` δημιουργεί έναν [στόχο βαθμονόμησης](calibration-targets.md) εντός του καρέ που πληροί τα κριτήρια QA ως απόλυτη αναφορά και επιστρέφει στη διαίρεση κατερχόμενης ακτινοβολίας του αισθητήρα φωτός του DAQ (ρ = π·L/E) όταν δεν υπάρχει στόχος· το `target` είναι αυστηρό (χωρίς υποκατάσταση από το DAQ)· Το `daq` έχει ως έγκυρη πηγή το DAQ. Μπορούν να παρασχεθούν σαρώσεις στόχων που έχουν μετρηθεί ανά μονάδα με το `--target-reflectance-dir`.

{% hint style="info" %}
**Ανάγνωση εικονοστοιχείων ανακλαστικότητας:**το DN που σημαίνει ρ = 1,0 είναι**ανά πηγή** — Τα αρχεία LATTICE προσθέτουν την ετικέτα `Chloros:PixelScale=32768` στο XMP· τα αρχείαSurvey3
χρησιμοποιούν το 65535 (και δεν περιέχουν ετικέτες `Chloros:*`). Διαβάστε την ετικέτα και διαιρέστε με αυτήν αντί να υποθέτετε μια σταθερά. Λεπτομέρειες και η μία σκόπιμη ακραία περίπτωση χωρίς κλίμακα βρίσκονται στην [ΑναφοράCLI
](reference/cli-reference.md).
{% endhint %}

**Η επεξεργασία ξεκινά πάντα από το `raw`.** Τα παράγωγα προϊόντα (εξαγωγές χωρίς debayering/ακτινοβολίας/ανακλαστικότητας) δεν επιστρέφονται ποτέ στον αγωγό επεξεργασίας — η επανεισαγωγή και η επεξεργασία τους θα εφάρμοζε διπλά τους μαθηματικούς υπολογισμούς βαθμονόμησης, οπότε τοChloros
τα παραλείπει και το δηλώνει. Το `--input-level` αποτελεί τη σκόπιμη «εξώπορτα διαφυγής» όταν χρειάζεται πραγματικά να επιβάλλετε ένα σημείο εισόδου.

***

## Όταν μια εκτέλεση αποτυγχάνει

Από την έκδοση 1.2.0 και μετά, το `process` αποτυγχάνει με εμφανή μήνυμα αντί να «επιτυγχάνει» χωρίς να εμφανίζει τίποτα:

* Μια εκτέλεση που **ζήτησε προϊόντα αλλά δεν έγραψε κανένα**— μόνο τα `project.json` και `calibration_data.json` — εμφανίζει το `Processing finished but wrote no image products.` και**τερματίζεται με μη μηδενική τιμή**, οπότε τα σενάρια μπορούν να το ανιχνεύσουν. Οι συνήθεις αιτίες: ο φάκελος εισόδου δεν αναγνωρίστηκε ως λήψη (ελέγξτε τη διάταξη και το `--input-level`), ή κάθε ζητηθέν προϊόν ήταν ακατάλληλο για αυτές τις κάμερες (π.χ. ζητώντας ακτινοβολία/ανακλαστικότητα από κάμερες που υποστηρίζουν μόνο τοRGB
).
* Μια **σκόπιμη εκτέλεση μόνο με μεταδεδομένα** (όλα τα προϊόντα απενεργοποιημένα, χωρίς `--indices`) εξακολουθεί να θεωρείται επιτυχής — η έξοδος μιας κενής εικόνας είναι το σωστό αποτέλεσμα σε αυτή την περίπτωση.
* Εκτελέστε ξανά με το `--verbose` και ελέγξτε το αρχείο καταγραφής του backend για τις γραμμές `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, οι οποίες εξηγούν τις παραλείψεις ανά κάμερα.

Κωδικοί εξόδου: `0` επιτυχία · `1` γενική αποτυχία · `2` σφάλμα παραμέτρου · `130` διακοπή με Ctrl+C.

***

## Δείκτες βλάστησης

Εκτελέστε το `--indices` με ένα ή περισσότερα ονόματα προεπιλογών· κάθε δείκτης αποθηκεύεται στον δικό του φάκελο `<INDEX>_Index_Images/`:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI NDRE GNDVI
```

Τα 22 ονόματα προεπιλογών που δέχεται το `process --indices`:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

{% hint style="warning" %}
**Υπάρχουν τρεις λίστες ευρετηρίου — μην τις συγχέετε.**Το αναπτυσσόμενο μενού «Ρυθμίσεις έργου» του GUI περιέχει 27 τύπους (προσθέτει τους `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` — αυτοί οι πέντε τύποι ισχύουν μόνο για το GUI και**δεν** ισχύουν για το `--indices`). Η εντολή ζωντανής/εκτός σύνδεσης `lattice index --preset` χρησιμοποιεί τη δική της ξεχωριστή λίστα με 22 προεπιλογές. Οι τύποι και οι μαθηματικοί υπολογισμοί των ζωνών τεκμηριώνονται στο [Τύποι πολυφασματικών δεικτών](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## Αισθητήρες φωτός DAQ: Μια γρήγορη γεύση

Η οικογένεια `daq pool-*` ελέγχει τους φασματικούς αισθητήρες DAQ της σειράς «MAPIR
» (DAQ-U μέσω USB, DAQ-M μέσω BLE, DAQ-E μέσω Ethernet) μέσω της μόνιμης ομάδας του backend — το GUI, τοCLI
και τοSDK
μοιράζονται όλα ένα live handle. **Το `pool-*` είναι η υποστηριζόμενη διαδρομή DAQ στο παρεχόμενο αρχείοCLI
**· άλλες υποεντολές `daq` στις οποίες ενδέχεται να αναφερθείτε αποτελούν μια επιφάνεια αποκλειστικά εσωτερικής χρήσης τουMAPIR
και τερματίζονται με ένα ρητό σφάλμα που σας παραπέμπει στο `pool-*`.

```bash
# 1. Open a pooled session (pick the line matching your sensor)
chloros-cli daq pool-connect                              # smart-detect
chloros-cli daq pool-connect --port COM3                  # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF      # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local   # DAQ-E by hostname (reliable)

# 2. List pooled sensors and their ids
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 3. Read the latest calibrated spectrum (W/m²/nm)
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 4. Record a calibrated .daq file for 60 s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 5. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

Το `pool-record` χωρίς το `--duration` εκτελείται μέχρι το `pool-record --stop`· ο προεπιλεγμένος κατάλογος εξόδου είναι ο `~/Documents/DAQ Live View/` **στον υπολογιστή του backend**. Το προφίλ διόρθωσης κορυφής επιλέγεται κατά τη σύνδεση (`--cap-id`, προεπιλογή backend `sunshine_cosine`) και μπορεί να αντικατασταθεί ζωντανά με `pool-set-cap` — τα προφίλ ανώτατων ορίων και το βαθμονομημένο εύρος του αισθητήρα περιγράφονται στα κεφάλαια DAQ του παρόντος εγχειριδίου.

{% hint style="warning" %}
**DAQ-E σε κεντρικό υπολογιστή με πολλαπλές κάρτες δικτύου (multi-NIC):** η πρώτη αυτόματη ανίχνευση `pool-connect --eth` μετά την εκκίνηση ενδέχεται να αποτύχει ακόμη και με έναν αισθητήρα σε καλή κατάσταση. Το `--eth-host <ip-or-hostname>` είναι η αξιόπιστη μορφή — χρησιμοποιήστε το όποτε η ανίχνευση δεν αποδίδει αποτελέσματα.
{% endhint %}

***

## Κάμερες LATTICE, PTP &amp; Αυτοματοποίηση Έργων

Η οικογένεια `lattice` (45+ υποεντολές) καλύπτει τη λειτουργία των καμερών LATTICE από την αρχή έως το τέλος: ανίχνευση, μεμονωμένες λήψεις, μόνιμες συγχρονισμένες συστοιχίες με τη ροή σύνδεσης «smart-prep» του GUI, ζωντανή προεπισκόπηση στο πρόγραμμα περιήγησης, ευθυγράμμιση, μαθηματικές πράξεις ευρετηρίου και διαγνωστικά του δικτυακού επιτραπέζιου υπολογιστή-NIC. Ένα δείγμα:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Παράλληλα: το `chloros-cli time-sync` παρέχει αναφορές σχετικά με τον «grandmaster» PTP που εκτελεί ο κεντρικός υπολογιστήςChloros
(οι κάμερες LATTICE και οι αισθητήρες DAQ-E λειτουργούν ως «slave» σε αυτόν για χρονικές σημάνσεις μεταξύ συσκευών), ενώ το `chloros-cli project` ανοίγει ένα αποθηκευμένο έργο «Chloros
» και ελέγχει τις κάμερες, τις συστοιχίες και τους αισθητήρες του χωρίς οθόνη — συμπεριλαμβανομένων των προγραμματισμένων συνταγών καταγραφής YAML.

Αυτές οι τρεις οικογένειες (`lattice`, `project`, `daq pool-*`) είναι επίσης οι μόνες που υποστηρίζουν το `CHLOROS_BACKEND_URL` για τον έλεγχο ενός **απομακρυσμένου** backend· οι βασικές εντολές στοχεύουν πάντα στον τοπικό υπολογιστή.

Πλήρεις οδηγίες βρίσκονται στα κεφάλαια για το LATTICE αυτού του εγχειριδίου· κάθε παράμετρος περιλαμβάνεται στην [ΑναφοράCLI
](reference/cli-reference.md).

***

## Αντιμετώπιση προβλημάτων: Τα 5 πιο συνηθισμένα

| Σύμπτωμα | Λύση |
| --- | --- |
| Το `Login required` ή μια προγραμματισμένη εργασία κολλάει σε μια προτροπή `Email:` | Εκτελέστε το `chloros-cli login EMAIL 'PASSWORD'` μία φορά σε αυτόν τον υπολογιστή — οι εντολές χωρίς αποθηκευμένη συνεδρία εκτελούνται διαδραστικά αντί να αποτυγχάνουν αμέσως. |
| `backend unreachable` | Εκκινήστε την εφαρμογή επιφάνειας εργασίας «Chloros
» ή εκτελέστε απευθείας το δυαδικό αρχείο του backend (`chloros-backend`). Εάν κατευθύνετε το `lattice`/`project`/`daq pool-*` σε ένα απομακρυσμένο backend, ελέγξτε το `CHLOROS_BACKEND_URL`. |
| Η σύνδεση του πίνακα έχει αποκλειστεί: `FRAMES WILL DROP` / `Reduce ROI to enable` | Η δακτυλιοειδής ένδειξη λήψης της κάρτας δικτύου (NIC) του κεντρικού υπολογιστή επαναφέρθηκε στις προεπιλεγμένες ρυθμίσεις — η #1 αιτία για την άρνηση σύνδεσης ενός συστήματος που λειτουργούσε προηγουμένως, συνήθως μετά από ενημέρωση του προγράμματος οδήγησης της κάρτας δικτύου. Εκτελέστε το `chloros-cli lattice network --fix` από ένα **τερματικό με δικαιώματα διαχειριστή** (ή ορίστε τα `ReceiveBufferLen=256`, `PendingReceives=64`)· ανατρέξτε στην ενότητα *Ρύθμιση &amp; Βελτιστοποίηση Κάρτας Δικτύου Κεντρικού Υπολογιστή* του εγχειριδίου αναφοράς. |
| Η υποεντολή `daq` τερματίζεται: «απαιτείται το πλήρες πακέτο daq…» | Αναμένεται σε έτοιμες εκδόσεις — το μεταγλωττισμένοCLI
παρέχει μόνο την οικογένεια `daq pool-*`, η οποία καλύπτει τη σύνδεση, τη ροή, την καταγραφή και την επιλογή cap. Χρησιμοποιήστε το `pool-*` (ή το `chloros_sdk.connect_daq_sensor()` από τοPython
). |
| Το Jetson εμφανίζει μια προειδοποίηση σχετικά με την ανταλλαγή πριν από μεγάλους φακέλους | Προσθέστε ανταλλαγή με υποστήριξη αρχείου — τοCLI
εμφανίζει τις ακριβείς εντολές `fallocate`/`swapon` που πρέπει να εκτελεστούν. |

***

## Λήψη βοήθειας

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Κάθε παράμετρος, κάθε υποεντολή:** [CLI
Αναφορά](reference/cli-reference.md)
* **ΙσοδύναμοPython
:** [Python
SDK
](api-python-sdk.md) και η [SDK
Αναφορά](reference/sdk-reference.md)
* **Υποστήριξη:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
