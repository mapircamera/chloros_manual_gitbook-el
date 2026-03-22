# CLI : Γραμμή εντολών

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>Το **Chloros CLI** παρέχει ισχυρή πρόσβαση μέσω γραμμής εντολών στον μηχανισμό επεξεργασίας εικόνων Chloros, επιτρέποντας την αυτοματοποίηση, τη δημιουργία σεναρίων και τη λειτουργία χωρίς οθόνη για τις ροές εργασίας απεικόνισης.

### Βασικά χαρακτηριστικά

* 🚀 **Αυτοματοποίηση** - Μαζική επεξεργασία πολλαπλών συνόλων δεδομένων μέσω σεναρίων
* 🔗 **Ενσωμάτωση** - Ενσωμάτωση σε υπάρχουσες ροές εργασίας και διαδικασίες
* 💻 **Λειτουργία χωρίς οθόνη** - Εκτέλεση χωρίς GUI
* 🌍 **Πολλαπλές γλώσσες** - Υποστήριξη για 38 γλώσσες
* ⚡ **Παράλληλη επεξεργασία** - Η [Δυναμική Προσαρμογή Υπολογιστικής Ισχύος](processing-architecture/dynamic-compute-adaptation.md) βελτιστοποιεί αυτόματα για το υλικό σας

### Απαιτήσεις

| Απαίτηση          | Λεπτομέρειες                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Λειτουργικό σύστημα** | Windows 10/11 (64-bit), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Άδεια χρήσης**          | Chloros+ ([απαιτείται πρόγραμμα επί πληρωμή](https://cloud.mapir.camera/pricing)) |
| **Μνήμη**           | Ελάχιστη μνήμη RAM 8 GB (συνιστάται 16 GB)                                  |
| **Διαδίκτυο**         | Απαιτείται για την ενεργοποίηση της άδειας                                     |
| **Χώρος στο δίσκο**       | Διαφέρει ανάλογα με το μέγεθος του έργου                                              |

{% hint style="warning" %}
**Απαιτήσεις άδειας**: Το CLI απαιτεί συνδρομή Chloros+ επί πληρωμή. Τα βασικά (δωρεάν) πακέτα δεν έχουν πρόσβαση στο CLI. Επισκεφθείτε το [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) για να πραγματοποιήσετε αναβάθμιση.
{% endhint %}

## Γρήγορη εκκίνηση

### Εγκατάσταση

#### Windows

Το CLI περιλαμβάνεται αυτόματα στο πρόγραμμα εγκατάστασης του Chloros:

1. Κατεβάστε και εκτελέστε το **Chloros Installer.exe**

2. Ολοκληρώστε τον οδηγό εγκατάστασης
3. Το CLI εγκαταστάθηκε στο: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style="success" %}
Το πρόγραμμα εγκατάστασης προσθέτει αυτόματα το `chloros-cli` στο PATH του συστήματός σας. Επανεκκινήστε το τερματικό σας μετά την εγκατάσταση.
{% endhint %}

#### Linux

Εγκαταστήστε το πακέτο `.deb` για την αρχιτεκτονική σας:

```bash
# Linux amd64
sudo dpkg -i chloros-amd64.deb

# Linux arm64 (NVIDIA Jetson, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Για λεπτομερείς οδηγίες εγκατάστασης του Linux, ανατρέξτε στην ενότητα [Εγκατάσταση του Linux](linux/linux-installation.md).

### Αρχική ρύθμιση

Πριν χρησιμοποιήσετε το CLI, ενεργοποιήστε την άδεια χρήσης Chloros+:

**Windows:**

```powershell
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process ~/images/dataset001
```

### Βασική χρήση

Επεξεργαστείτε ένα φάκελο με τις προεπιλεγμένες ρυθμίσεις:

**Windows:**

```powershell
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
chloros-cli process ~/images/dataset001
```

***

## Αναφορά εντολών

### Γενική σύνταξη

```
chloros-cli [global-options] <command> [command-options]
```

***

## Εντολές

### `process` - Επεξεργασία εικόνων

Επεξεργασία εικόνων σε ένα φάκελο με βαθμονόμηση.

**Σύνταξη:**

```bash
chloros-cli process <input-folder> [options]
```

**Παραδείγματα:**

```bash
# Windows
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance

# Linux
chloros-cli process ~/datasets/survey_001 --vignette --reflectance
```

#### Επιλογές εντολής επεξεργασίας

| Επιλογή                | Τύπος    | Προεπιλογή        | Περιγραφή                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Διαδρομή    | _Απαιτείται_     | Φάκελος που περιέχει πολυφασματικές εικόνες RAW/JPG                                         |
| `-o, --output`        | Διαδρομή    | Ίδια με την είσοδο  | Φάκελος εξόδου για τις επεξεργασμένες εικόνες                                                     |
| `-n, --project-name`  | Συμβολοσειρά  | Αυτόματη δημιουργία | Προσαρμοσμένο όνομα έργου                                                                    |
| `--vignette`          | Επιλογή    | Ενεργοποιημένη        | Ενεργοποίηση διόρθωσης βινιέτας                                                             |
| `--no-vignette`       | Επιλογή    | -              | Απενεργοποίηση διόρθωσης βινιέτας                                                            |
| `--reflectance`       | Σημαία    | Ενεργοποιημένη        | Ενεργοποίηση βαθμονόμησης ανακλαστικότητας                                                         |
| `--no-reflectance`    | Σημαία    | -              | Απενεργοποίηση βαθμονόμησης ανακλαστικότητας                                                        |
| `--ppk`               | Σημαία    | Απενεργοποιημένη       | Εφαρμογή διορθώσεων PPK από δεδομένα αισθητήρα φωτός .daq                                      |
| `--format`            | Επιλογή  | TIFF (16-bit)  | Μορφή εξόδου: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Ακέραιος | Αυτόματο           | Ελάχιστο μέγεθος στόχου σε εικονοστοιχεία για την ανίχνευση του πίνακα βαθμονόμησης                          |
| `--target-clustering` | Ακέραιος | Αυτόματο           | Όριο ομαδοποίησης στόχων (0-100)                                                    |
| `--debayer`           | Επιλογή  | `standard`     | Μέθοδος Debayer: `standard` ή `texture-aware` (μόνο Chloros+)                          |
| `--target`, `--targets` | Σημαία  | Απενεργοποιημένη       | Αναζήτηση στόχων βαθμονόμησης μόνο σε υποφάκελο &quot;target&quot; ή &quot;targets&quot; (επιταχύνει την επεξεργασία) |
| `--indices`           | Λίστα    | Κανένα           | Δείκτες βλάστησης προς υπολογισμό (π.χ., `--indices NDVI NDRE GNDVI`)                    |
| `--exposure-pin-1`    | Συμβολοσειρά  | Κανένα           | Κλείδωμα έκθεσης για μοντέλο κάμερας (Pin 1)                                                 |
| `--exposure-pin-2`    | Συμβολοσειρά  | Κανένα           | Κλείδωμα έκθεσης για μοντέλο κάμερας (Pin 2)                                                 |
| `--recal-interval`    | Ακέραιος | Αυτόματο           | Διάστημα επαναβαθμονόμησης σε δευτερόλεπτα                                                      |
| `--timezone-offset`   | Ακέραιος | 0              | Διαφορά ζώνης ώρας σε ώρες                                                               |

***

### `login` - Πιστοποίηση λογαριασμού

Συνδεθείτε με τα διαπιστευτήριά σας Chloros+ για να ενεργοποιήσετε την επεξεργασία CLI.

**Σύνταξη:**

```bash
chloros-cli login <email> <password>
```

**Παράδειγμα:**

```bash
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Ειδικοί χαρακτήρες**: Χρησιμοποιήστε απλά εισαγωγικά γύρω από κωδικούς πρόσβασης που περιέχουν χαρακτήρες όπως `$`, `!` ή κενά.
{% endhint %}

**Έξοδος:**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - Διαγραφή διαπιστευτηρίων

Διαγράψτε τα αποθηκευμένα διαπιστευτήρια και αποσυνδεθείτε από τον λογαριασμό σας.

**Σύνταξη:**

```bash
chloros-cli logout
```

**Παράδειγμα:**

```bash
chloros-cli logout
```

**Έξοδος:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style="info" %}
**Χρήστες SDK**: Το Python SDK παρέχει επίσης μια προγραμματιστική μέθοδο `logout()` για τη διαγραφή διαπιστευτηρίων εντός των σεναρίων Python. Δείτε την [τεκμηρίωση Python SDK](api-python-sdk.md#logout) για λεπτομέρειες.
{% endhint %}

***

### `status` - Έλεγχος κατάστασης άδειας χρήσης

Εμφάνιση της τρέχουσας κατάστασης άδειας χρήσης και πιστοποίησης.

**Σύνταξη:**

```bash
chloros-cli status
```

**Παράδειγμα:**

```bash
chloros-cli status
```

**Έξοδος:**

```
╔══════════════════════════════════════╗
║     LICENSE & ACCOUNT INFORMATION    ║
╚══════════════════════════════════════╝

📧 Email: user@example.com
📋 Plan: Chloros+ Professional
🔓 API/CLI Access: Enabled
✓ Status: Active
```

***

### `export-status` - Έλεγχος προόδου εξαγωγής

Παρακολούθηση της προόδου εξαγωγής του Thread 4 κατά τη διάρκεια ή μετά την επεξεργασία.

**Σύνταξη:**

```bash
chloros-cli export-status
```

**Παράδειγμα:**

```bash
chloros-cli export-status
```

**Περίπτωση χρήσης:** Καλέστε αυτήν την εντολή ενώ εκτελείται η επεξεργασία για να ελέγξετε την πρόοδο εξαγωγής.***

### `language` - Διαχείριση γλώσσας διεπαφής

Προβάλετε ή αλλάξτε τη γλώσσα διεπαφής CLI.

**Σύνταξη:**

```bash
# Show current language
chloros-cli language

# List all available languages
chloros-cli language --list

# Set a specific language
chloros-cli language <language-code>
```

**Παραδείγματα:**

```bash
# View current language
chloros-cli language

# List all 38 supported languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Change to Japanese
chloros-cli language ja
```

#### Υποστηριζόμενες γλώσσες (38 συνολικά)

| Κωδικός    | Γλώσσα              | Εγγενές όνομα      |
| ------- | --------------------- | ---------------- |
| `en`    | Αγγλικά               | English          |
| `es`    | Ισπανικά               | Español          |
| `pt`    | Πορτογαλικά            | Português        |
| `fr`    | Γαλλικά                | Français         |
| `de`    | Γερμανικά                | Deutsch          |
| `it`    | Ιταλικά               | Italiano         |
| `ja`    | Ιαπωνικά              | 日本語              |
| `ko`    | Κορεατικά                | 한국어              |
| `zh`    | Κινέζικα (Απλοποιημένα)  | 简体中文             |
| `zh-TW` | Κινέζικα (Παραδοσιακά) | 繁體中文             |
| `ru`    | Ρωσικά               | Русский          |
| `nl`    | Ολλανδικά                | Nederlands       |
| `ar`    | Αραβικά                | العربية          |
| `pl`    | Πολωνικά                | Polski           |
| `tr`    | Τουρκικά               | Türkçe           |
| `hi`    | Χίντι                 | हिंदी            |
| `id`    | Ινδονησιακά            | Bahasa Indonesia |
| `vi`    | Βιετναμέζικα            | Tiếng Việt       |
| `th`    | Ταϊλανδικά                  | ไทย              |
| `sv`    | Σουηδικά               | Svenska          |
| `da`    | Δανικά                | Dansk            |
| `no`    | Νορβηγικά             | Norsk            |
| `fi`    | Φινλανδικά               | Suomi            |
| `el`    | Ελληνικά                 | Ελληνικά         |
| `cs`    | Τσεχικά                 | Čeština          |
| `hu`    | Ουγγρικά             | Magyar           |
| `ro`    | Ρουμανικά              | Română           |
| `uk`    | Ουκρανικά             | Українська       |
| `pt-BR` | Βραζιλιάνικα Πορτογαλικά  | Português Brasileiro |
| `zh-HK` | Καντονέζικα             | 粵語             |
| `ms`    | Μαλαισιανά                 | Bahasa Melayu    |
| `sk`    | Σλοβακικά                | Slovenčina       |
| `bg`    | Βουλγαρικά             | Български        |
| `hr`    | Κροατικά              | Hrvatski         |
| `lt`    | Λιθουανικά            | Lietuvių         |
| `lv`    | Λετονικά               | Latviešu         |
| `et`    | Εσθονικά              | Eesti            |
| `sl`    | Σλοβενικά             | Slovenščina      |

{% hint style="success" %}
**Αυτόματη διατήρηση**: Η προτίμησή σας για τη γλώσσα αποθηκεύεται στο `~/.chloros/cli_language.json` και διατηρείται σε όλες τις συνεδρίες.
{% endhint %}

***

### `set-project-folder` - Ορισμός προεπιλεγμένου φακέλου έργου

Αλλάξτε τη θέση του προεπιλεγμένου φακέλου έργου (κοινή με το GUI στο Windows).

**Σύνταξη:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Παραδείγματα:**

```bash
# Windows
chloros-cli set-project-folder "C:\Projects\2025"

# Linux
chloros-cli set-project-folder ~/projects/2025
```

***

### `get-project-folder` - Εμφάνιση φακέλου έργου

Εμφανίζει την τρέχουσα θέση του προεπιλεγμένου φακέλου έργου.

**Σύνταξη:**

```bash
chloros-cli get-project-folder
```

**Παράδειγμα:**

```bash
chloros-cli get-project-folder
```

**Έξοδος:**

```

# Windows
ℹ Current project folder: C:\Projects\2025

# Linux
ℹ Current project folder: /home/user/.local/share/chloros/projects
```

***

### `reset-project-folder` - Επαναφορά στις προεπιλεγμένες ρυθμίσεις

Επαναφορά του φακέλου έργου στην προεπιλεγμένη θέση.

**Σύνταξη:**

```bash
chloros-cli reset-project-folder
```

***

### `selftest` - Εκτέλεση διαγνωστικών ελέγχων συστήματος

Εκτελέστε 7 διαγνωστικούς ελέγχους για να επαληθεύσετε τη διαμόρφωση του συστήματός σας.

**Σύνταξη:**

```bash
chloros-cli selftest
```

**Διαγνωστικοί έλεγχοι που εκτελούνται:**

1. Έλεγχος έκδοσης
2. Διαθεσιμότητα θύρας (5000)
3. Εκκίνηση backend
4. Δοκιμή συνδεσιμότητας API
5. Πληροφορίες συστήματος και ανίχνευση GPU
6. Επαλήθευση μοντέλων αποθορυβοποίησης
7. Έλεγχος διαθεσιμότητας CUDA

{% hint style="info" %}
**Χρήσιμο για την αντιμετώπιση προβλημάτων**: Εκτελέστε το `selftest` μετά την εγκατάσταση για να επαληθεύσετε ότι το σύστημά σας έχει ρυθμιστεί σωστά, ειδικά σε Linux/Jetson όπου η ρύθμιση της GPU και του CUDA ενδέχεται να χρειαστεί επαλήθευση.
{% endhint %}

***

### `update` - Έλεγχος για ενημερώσεις (μόνο για Linux)

Ελέγξτε και εγκαταστήστε τις ενημερώσεις CLI σε συστήματα Linux.

**Σύνταξη:**

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

| Επιλογή    | Περιγραφή                        |
| --------- | ---------------------------------- |
| `--check` | Μόνο έλεγχος για ενημερώσεις, χωρίς εγκατάσταση |

{% hint style="info" %}
Αυτή η εντολή είναι διαθέσιμη μόνο στο Linux. Στο Windows, οι ενημερώσεις παρέχονται μέσω του προγράμματος εγκατάστασης.
{% endhint %}

***

## Παγκόσμιες επιλογές

Αυτές οι επιλογές ισχύουν για όλες τις εντολές:

| Επιλογή            | Τύπος    | Προεπιλογή       | Περιγραφή                                      |
| ----------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe`   | Διαδρομή    | Αυτόματη ανίχνευση | Διαδρομή προς το εκτελέσιμο του backend                       |
| `--port`          | Ακέραιος | 5000          | Αριθμός θύρας του backend API                          |
| `--restart`       | Σημαία    | -             | Επιβολή επανεκκίνησης του backend (τερματισμός υπαρχουσών διεργασιών) |
| `--version`       | Σημαία    | -             | Εμφάνιση πληροφοριών έκδοσης και έξοδος                |
| `--help`          | Σημαία    | -             | Εμφάνιση πληροφοριών βοήθειας και έξοδος                   |

{% hint style="info" %}
**Αυτόματη ανίχνευση backend**: Η διαδρομή `--backend-exe` ανιχνεύεται αυτόματα ανά πλατφόρμα:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (χειροκίνητα)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

**Παράδειγμα με παγκόσμιες επιλογές:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

**Linux:**

```bash
chloros-cli --port 5001 process ~/datasets/survey_001
```

***

## Οδηγός ρυθμίσεων επεξεργασίας

### Παράλληλη επεξεργασία &amp; Δυναμική προσαρμογή υπολογισμών

Το Chloros 1.1.0 περιλαμβάνει [Δυναμική προσαρμογή υπολογισμών](processing-architecture/dynamic-compute-adaptation.md) — ο μηχανισμός επεξεργασίας **ανιχνεύει αυτόματα το υλικό σας** και επιλέγει τη βέλτιστη στρατηγική:

| Πλατφόρμα | Στρατηγική | Εργαζόμενοι | Αγωγός | Σημειώσεις |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` | Αποδοτική χρήση μνήμης, σειριακή |
| **Jetson Orin NX 16GB** | `GPU_PARALLEL` | 3 | `fused_gpu` | Ταυτόχρονη επεξεργασία GPU |
| **Επιτραπέζιος υπολογιστής με GPU 8 GB** | `GPU_SINGLE` | 3 | `tiled_gpu` | Καλή απόδοση επιτραπέζιου υπολογιστή |
| **Επιτραπέζιος υπολογιστής με GPU 12GB+** | `GPU_PARALLEL` | 3-4 | `fused_gpu` | Βέλτιστη απόδοση επιτραπέζιου υπολογιστή |
| **Σύστημα μόνο με CPU** | `CPU_PARALLEL` | πυρήνες - 1 | `cpu_fallback` | Δεν απαιτείται GPU |

{% hint style="success" %}
**Δεν απαιτείται χειροκίνητη διαμόρφωση!** Το Chloros ανιχνεύει αυτόματα την CPU, την GPU, τη μνήμη RAM και (στο Jetson) τους θερμικούς αισθητήρες, και στη συνέχεια διαμορφώνει αυτόματα τη βέλτιστη ροή επεξεργασίας.
{% endhint %}

### Μέθοδοι Debayer

| Μέθοδος | Σημαία CLI | Ποιότητα | Ταχύτητα | Άδεια |
| --- | --- | --- | --- | --- |
| **Standard (Γρήγορη, Μέτρια ποιότητα)** | `--debayer standard` | Καλή | Γρήγορη | Δωρεάν / Chloros+ |
| **Texture Aware (Αργή, Υψηλότερη ποιότητα)** | `--debayer texture-aware` | Υψηλότερη | Αργή | Μόνο Chloros+ |

Η προεπιλεγμένη μέθοδος debayer είναι η **Standard**. Η μέθοδος**Texture Aware** χρησιμοποιεί ένα μοντέλο αποθορυβοποίησης AI/ML για έξοδο υψηλότερης ποιότητας, αλλά απαιτεί άδεια Chloros+ και GPU NVIDIA.

```bash
# Use Texture Aware debayer (Chloros+ only)
chloros-cli process ~/datasets/field_a --debayer texture-aware
```

### Διόρθωση βινιέτας

**Τι κάνει:** Διορθώνει την πτώση φωτεινότητας στα άκρα της εικόνας (σκοτεινότερες γωνίες που είναι συνηθισμένες στις εικόνες από κάμερες).

* **Ενεργοποιημένη από προεπιλογή** - Οι περισσότεροι χρήστες θα πρέπει να την διατηρήσουν ενεργοποιημένη
* Χρησιμοποιήστε το `--no-vignette` για να την απενεργοποιήσετε

{% hint style="success" %}
**Σύσταση**: Ενεργοποιείτε πάντα τη διόρθωση βινιέτας για να εξασφαλίσετε ομοιόμορφη φωτεινότητα σε όλο το κάδρο.
{% endhint %}

### Βαθμονόμηση ανακλαστικότητας

Μετατρέπει τις ακατέργαστες τιμές του αισθητήρα σε τυποποιημένα ποσοστά ανακλαστικότητας χρησιμοποιώντας πίνακες βαθμονόμησης.

* **Ενεργοποιημένη από προεπιλογή** - Απαραίτητη για την ανάλυση της βλάστησης
* Απαιτεί πίνακες στόχους βαθμονόμησης στις εικόνες
* Χρησιμοποιήστε το `--no-reflectance` για απενεργοποίηση

{% hint style="info" %}
**Απαιτήσεις**: Βεβαιωθείτε ότι τα πάνελ βαθμονόμησης είναι σωστά εκτεθειμένα και ορατά στις εικόνες σας για ακριβή μετατροπή της ανακλαστικότητας.
{% endhint %}

### Διορθώσεις PPK

**Τι κάνει:** Εφαρμόζει διορθώσεις Post-Processed Kinematic χρησιμοποιώντας δεδομένα καταγραφής DAQ-A-SD για βελτιωμένη ακρίβεια GPS.

* **Απενεργοποιημένο από προεπιλογή**
* Χρησιμοποιήστε το `--ppk` για ενεργοποίηση
* Απαιτεί αρχεία .daq στο φάκελο του έργου από τον αισθητήρα φωτός DAQ-A-SD MAPIR.

### Μορφές εξόδου

<table><thead><tr><th width="197">Μορφή</th><th width="130.20001220703125">Βάθος bit</th><th width="116.5999755859375">Μέγεθος αρχείου</th><th>Ιδανικό για</th></tr></thead><tbody><tr><td><strong>TIFF (16-bit)</strong> ⭐</td><td>Ακέραιος 16-bit</td><td>Μεγάλο</td><td>Ανάλυση GIS, φωτογραμμετρία (συνιστάται)</td></tr><tr><td><strong>TIFF (32-bit, Ποσοστό)</strong></td><td>32-bit κινητής υποδιαστολής</td><td>Πολύ μεγάλο</td><td>Επιστημονική ανάλυση, έρευνα</td></tr><tr><td><strong>PNG (8-bit)</strong></td><td>8-bit ακέραιος</td><td>Μεσαίο</td><td>Οπτική επιθεώρηση, κοινή χρήση στο διαδίκτυο</td></tr><tr><td><strong>JPG (8-bit)</strong></td><td>Ακέραιος 8-bit</td><td>Μικρό</td><td>Γρήγορη προεπισκόπηση, συμπιεσμένη έξοδος</td></tr></tbody></table>***

## Αυτοματοποίηση &amp; Σενάρια

### Μαζική επεξεργασία PowerShell (Windows)

Επεξεργαστείτε αυτόματα πολλούς φακέλους συνόλων δεδομένων στο Windows:

```powershell
# process_all_datasets.ps1

$datasets = Get-ChildItem "C:\Datasets\2025" -Directory

foreach ($dataset in $datasets) {
    Write-Host "Processing $($dataset.Name)..." -ForegroundColor Cyan
    
    chloros-cli process $dataset.FullName `
        --vignette `
        --reflectance
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "✓ $($dataset.Name) complete" -ForegroundColor Green
    } else {
        Write-Host "✗ $($dataset.Name) failed" -ForegroundColor Red
    }
}

Write-Host "All datasets processed!" -ForegroundColor Green
```

### Σενάριο μαζικής επεξεργασίας Windows (Windows)

Απλός βρόχος για μαζική επεξεργασία στο Windows:

```batch
@echo off
echo Starting batch processing...

for /d %%i in (C:\Datasets\2025\*) do (
    echo.
    echo ========================================
    echo Processing: %%i
    echo ========================================
    chloros-cli process "%%i"
    
    if %ERRORLEVEL% EQU 0 (
        echo SUCCESS: %%i processed
    ) else (
        echo ERROR: %%i failed
    )
)

echo.
echo All datasets processed!
pause
```

### Μαζική επεξεργασία Bash (Linux)

Επεξεργασία πολλαπλών φακέλων συνόλων δεδομένων στο Linux:

```bash
#!/bin/bash
# process_all_datasets.sh

for dataset in ~/datasets/2026/*/; do
    name=$(basename "$dataset")
    echo "Processing $name..."

    chloros-cli process "$dataset" \
        --vignette \
        --reflectance

    if [ $? -eq 0 ]; then
        echo "✓ $name complete"
    else
        echo "✗ $name failed"
    fi
done

echo "All datasets processed!"
```

### Σενάριο αυτοματοποίησης Python (διαπλατφορμικό)

Προηγμένη αυτοματοποίηση με διαχείριση σφαλμάτων (λειτουργεί σε Windows και Linux):

```python
import subprocess
import os
import sys
from pathlib import Path
from datetime import datetime

def process_dataset(input_folder):
    """Process a folder using Chloros CLI"""
    cmd = ['chloros-cli', 'process', str(input_folder)]
    
    # Execute command
    result = subprocess.run(
        cmd, 
        capture_output=True, 
        text=True,
        encoding='utf-8'
    )
    
    return result.returncode == 0, result.stdout, result.stderr

def main():
    """Process all datasets in a directory"""
    # Adjust path for your platform
    # Windows: Path('C:/Datasets/2025')
    # Linux:   Path.home() / 'datasets' / '2025'
    datasets_dir = Path('C:/Datasets/2025')
    log_file = Path('processing_log.txt')
    
    successful = []
    failed = []
    
    # Start processing
    print(f"Starting batch processing: {datetime.now()}")
    print(f"Scanning: {datasets_dir}")
    print("=" * 60)
    
    for dataset_folder in sorted(datasets_dir.iterdir()):
        if not dataset_folder.is_dir():
            continue
        
        print(f"\nProcessing: {dataset_folder.name}")
        
        success, stdout, stderr = process_dataset(dataset_folder)
        
        if success:
            print(f"✓ {dataset_folder.name} - SUCCESS")
            successful.append(dataset_folder.name)
        else:
            print(f"✗ {dataset_folder.name} - FAILED")
            failed.append(dataset_folder.name)
            
            # Log error details
            with open(log_file, 'a', encoding='utf-8') as f:
                f.write(f"\n=== {dataset_folder.name} - {datetime.now()} ===\n")
                f.write(f"STDOUT:\n{stdout}\n")
                f.write(f"STDERR:\n{stderr}\n")
    
    # Print summary
    print("\n" + "=" * 60)
    print(f"SUMMARY - Completed: {datetime.now()}")
    print(f"  Successful: {len(successful)}")
    print(f"  Failed: {len(failed)}")
    
    if failed:
        print(f"\nFailed folders:")
        for folder in failed:
            print(f"  - {folder}")
        print(f"\nCheck {log_file} for error details")
        sys.exit(1)
    else:
        print("\nAll datasets processed successfully!")
        sys.exit(0)

if __name__ == '__main__':
    main()
```

***

## Ροή εργασιών επεξεργασίας

### Τυπική ροή εργασιών

1. **Εισαγωγή**: Φάκελος που περιέχει ζεύγη εικόνων RAW/JPG
2. **Ανίχνευση**: Το CLI σαρώνει αυτόματα για υποστηριζόμενα αρχεία εικόνων
3. **Επεξεργασία**: Η παράλληλη λειτουργία προσαρμόζεται στους πυρήνες της CPU σας (Chloros+)
4. **Έξοδος**: Δημιουργεί υποφακέλους ανά μοντέλο κάμερας με τις επεξεργασμένες εικόνες

### Παράδειγμα δομής εξόδου

```

MyProject/
├── project.json                             # Project metadata
├── 2025_0203_193056_008.JPG                # Original JPG
├── 2025_0203_193055_007.RAW                # Original RAW
└── Survey3N_RGN/                           # Processed outputs ✓
    ├── 2025_0203_193056_008_Reflectance.tif   # Calibrated reflectance
    ├── 2025_0203_193056_008_Target.tif        # Target detection
    └── ...
```

### Εκτιμήσεις χρόνου επεξεργασίας

Τυπικοί χρόνοι επεξεργασίας για 100 εικόνες (12 MP η καθεμία):

| Πλατφόρμα | Λειτουργία | Εκτιμώμενος χρόνος | Σημειώσεις |
| --- | --- | --- | --- |
| **Επιτραπέζιος υπολογιστής με GPU 12GB+** | `GPU_PARALLEL` | 5-10 λεπτά | Ταχύτερη επιλογή |
| **Επιτραπέζιος υπολογιστής με GPU 8GB** | `GPU_SINGLE` | 10-15 λεπτά | Καλή απόδοση |
| **Jetson Orin NX 16GB** | `GPU_PARALLEL` | 15-25 λεπτά | Υπολογιστική ισχύς στην άκρη |
| **Jetson Nano 8GB** | `GPU_SINGLE` | 30-60 λεπτά | Περιορισμένη μνήμη |
| **Μόνο CPU** | `CPU_PARALLEL` | 20-40 λεπτά | Δεν απαιτείται GPU |

{% hint style="info" %}
**Συμβουλή απόδοσης**: Ο χρόνος επεξεργασίας ποικίλλει ανάλογα με τον αριθμό των εικόνων, την ανάλυση, τη μέθοδο αποσυμπίεσης και το υλικό. Η αποσυμπίεση με αναγνώριση υφής διαρκεί σημαντικά περισσότερο από την τυπική. Δείτε [Δυναμική προσαρμογή υπολογισμών](processing-architecture/dynamic-compute-adaptation.md) για λεπτομέρειες.
{% endhint %}

***

## Αντιμετώπιση προβλημάτων

### CLI Δεν βρέθηκε

**Windows Σφάλμα:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Windows Λύσεις:**

1. Επαληθεύστε τη θέση εγκατάστασης:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Χρησιμοποιήστε την πλήρη διαδρομή αν δεν βρίσκεται στο PATH:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Προσθέστε στο PATH χειροκίνητα:
   * Ανοίξτε Ιδιότητες συστήματος → Μεταβλητές περιβάλλοντος
   * Επεξεργαστείτε τη μεταβλητή PATH
   * Προσθέστε: `C:\Program Files\Chloros\resources\cli`
   * Επανεκκινήστε το τερματικό

**Linux Σφάλμα:**

```
chloros-cli: command not found
```

**Λύσεις για το Linux:**

1. Επαληθεύστε την εγκατάσταση:

```bash
which chloros-cli
dpkg -L chloros-amd64  # or chloros-arm64-jp6
```

2. Επανεκκινήστε το shell σας:

```bash
source ~/.bashrc
```

3. Ελέγξτε τα δικαιώματα:

```bash
sudo chmod +x /usr/bin/chloros-cli
```

***

### Αποτυχία εκκίνησης του backend**Σφάλμα:**

```

Backend failed to start within 30 seconds
```

**Λύσεις:**

1. Ελέγξτε αν το backend εκτελείται ήδη (κλείστε το πρώτα)
2. Ελέγξτε αν το τείχος προστασίας δεν το μπλοκάρει (Windows) ή ελέγξτε τη διαθεσιμότητα της θύρας (Linux: `lsof -i :5000`)
3. Δοκιμάστε διαφορετική θύρα:

```bash
# Windows
chloros-cli --port 5001 process "C:\Datasets\Field_A"

# Linux
chloros-cli --port 5001 process ~/datasets/field_a
```

4. Επιβάλετε επανεκκίνηση του backend:

```bash
# Windows
chloros-cli --restart process "C:\Datasets\Field_A"

# Linux
chloros-cli --restart process ~/datasets/field_a
```

5. Στο Linux, ελέγξτε αν υπάρχει το εκτελέσιμο του backend:

```bash
ls -la /usr/lib/chloros/chloros-backend
```

***

### Προβλήματα άδειας χρήσης / πιστοποίησης**Σφάλμα:**

```

Chloros+ license required for CLI access
```

**Λύσεις:**

1. Βεβαιωθείτε ότι διαθέτετε ενεργή συνδρομή Chloros+
2. Συνδεθείτε με τα διαπιστευτήριά σας:

```bash
chloros-cli login user@example.com 'password'
```

3. Ελέγξτε την κατάσταση της άδειας χρήσης:

```bash
chloros-cli status
```

4. Επικοινωνήστε με την υποστήριξη: info@mapir.camera

***

### Δεν βρέθηκαν εικόνες**Σφάλμα:**

```

No images found in the specified folder
```

**Λύσεις:**

1. Βεβαιωθείτε ότι ο φάκελος περιέχει υποστηριζόμενες μορφές (.RAW, .TIF, .JPG)
2. Ελέγξτε ότι η διαδρομή του φακέλου είναι σωστή (χρησιμοποιήστε εισαγωγικά για διαδρομές με κενά)
3. Βεβαιωθείτε ότι έχετε δικαιώματα ανάγνωσης για το φάκελο
4. Ελέγξτε ότι οι επεκτάσεις αρχείων είναι σωστές

***

### Η επεξεργασία σταματά ή κολλάει**Λύσεις:**

1. Ελέγξτε τον διαθέσιμο χώρο στο δίσκο (βεβαιωθείτε ότι υπάρχει αρκετός για την έξοδο)
2. Κλείστε άλλες εφαρμογές για να ελευθερώσετε μνήμη
3. Μειώστε τον αριθμό των εικόνων (επεξεργαστείτε σε παρτίδες)

***

### Η θύρα χρησιμοποιείται ήδη**Σφάλμα:**

```

Port 5000 is already in use
```

**Λύσεις:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

**Linux:**

```bash
# Find what's using port 5000
lsof -i :5000

# Use a different port
chloros-cli --port 5001 process ~/datasets/field_a
```

***

## Συχνές ερωτήσεις

### Ε: Χρειάζομαι άδεια χρήσης για το CLI;

**Α:**Ναι! Το CLI απαιτεί μια πληρωμένη**άδεια χρήσης Chloros+**.

* ❌ Πακέτο Standard (δωρεάν): Το CLI είναι απενεργοποιημένο
* ✅ Πακέτα Chloros+ (επί πληρωμή): Το CLI είναι πλήρως ενεργοποιημένο

Εγγραφείτε στο: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### Ε: Μπορώ να χρησιμοποιήσω το CLI σε έναν διακομιστή χωρίς GUI;**Α:** Ναι! Το CLI λειτουργεί εντελώς χωρίς οθόνη. Αυτή είναι η κύρια περίπτωση χρήσης στο Linux.**Windows Server:**
* Windows Server 2016 ή νεότερη έκδοση
* Εγκατεστημένο Visual C++ Redistributable

**Linux Server:**
* Ubuntu 20.04+ / Debian 11+ (amd64) ή JetPack 6 (arm64)
* Εγκατάσταση μέσω του πακέτου `.deb`

**Και στις δύο πλατφόρμες:**
* Ελάχιστη μνήμη RAM 8 GB (συνιστάται 16 GB)
* Εφάπαξ ενεργοποίηση άδειας χρήσης: `chloros-cli login user@example.com 'password'`

***

### Ε: Πού αποθηκεύονται οι επεξεργασμένες εικόνες;**Α:**Από προεπιλογή, οι επεξεργασμένες εικόνες αποθηκεύονται στον**ίδιο φάκελο με τις εικόνες εισόδου**, σε υποφακέλους με το μοντέλο της κάμερας (π.χ., `Survey3N_RGN/`).

Χρησιμοποιήστε την επιλογή `-o` για να καθορίσετε έναν διαφορετικό φάκελο εξόδου:

```bash
# Windows
chloros-cli process "C:\Input" -o "D:\Output"

# Linux
chloros-cli process ~/input -o ~/output
```

***

### Ε: Μπορώ να επεξεργαστώ πολλούς φακέλους ταυτόχρονα;**Α:** Όχι απευθείας με μία εντολή, αλλά μπορείτε να χρησιμοποιήσετε σενάρια για να επεξεργαστείτε τους φακέλους διαδοχικά. Ανατρέξτε στην ενότητα [Αυτοματοποίηση &amp; Σενάρια](CLI.md#automation--scripting).***

### Ε: Πώς αποθηκεύω την έξοδο του CLI σε ένα αρχείο καταγραφής;**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

**Linux Bash:**

```bash
chloros-cli process ~/datasets/field_a 2>&1 | tee processing.log
```

***

### Ε: Τι συμβαίνει αν πατήσω Ctrl+C κατά τη διάρκεια της επεξεργασίας;**Α:** Το CLI θα:

1. Σταματήσει την επεξεργασία ομαλά
2. Κλείσει το backend
3. Βγει με κωδικό 130

Εικόνες που έχουν υποστεί μερική επεξεργασία ενδέχεται να παραμείνουν στον φάκελο εξόδου.

***

### Ε: Μπορώ να αυτοματοποιήσω την επεξεργασία του CLI;**Α:** Φυσικά! Το CLI έχει σχεδιαστεί για αυτοματοποίηση. Δείτε [Αυτοματοποίηση &amp; Σενάρια](CLI.md#automation--scripting) για παραδείγματα PowerShell (Windows), Batch (Windows), Bash (Linux) και Python (διαπλατφορμικό).***

### Ε: Πώς μπορώ να ελέγξω την έκδοση του CLI;**Α:**

```bash
chloros-cli --version
```

**Έξοδος:**

```

Chloros CLI 1.1.0
```

***

## Λήψη βοήθειας

### Βοήθεια γραμμής εντολών

Προβάλετε πληροφορίες βοήθειας απευθείας στο CLI:

```bash
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### Κανάλια υποστήριξης

* **Email**: info@mapir.camera
* **Ιστοσελίδα**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Τιμές**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)***

## Πλήρη παραδείγματα

### Παράδειγμα 1: Βασική επεξεργασία

Επεξεργασία με προεπιλεγμένες ρυθμίσεις (βινιέτα, ανακλαστικότητα):

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a_2025_01_15
```

***

### Παράδειγμα 2: Επιστημονικά αποτελέσματα υψηλής ποιότητας

32-bit float TIFF:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "TIFF (32-bit, Percent)" \
  --vignette \
  --reflectance
```

***

### Παράδειγμα 3: Γρήγορη επεξεργασία προεπισκόπησης

8-bit PNG χωρίς βαθμονόμηση για γρήγορη ανασκόπηση:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "PNG (8-bit)" \
  --no-vignette \
  --no-reflectance
```

***

### Παράδειγμα 4: Επεξεργασία με διόρθωση PPK

Εφαρμογή διορθώσεων PPK με ανακλαστικότητα:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --ppk \
  --reflectance
```

***

### Παράδειγμα 5: Προσαρμοσμένη τοποθεσία εξόδου

Επεξεργασία σε διαφορετική θέση με συγκεκριμένη μορφή:

**Windows:**

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

**Linux:**

```bash
chloros-cli process ~/input/raw_images \
  -o ~/output/processed \
  --format "TIFF (16-bit)"
```

***

### Παράδειγμα 6: Ροή εργασιών ελέγχου ταυτότητας

Ολοκλήρωση ροής ελέγχου ταυτότητας (ίδια σε όλες τις πλατφόρμες):

```bash
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
# Windows: chloros-cli process "C:\Datasets\Field_A"
# Linux:   chloros-cli process ~/datasets/field_a
chloros-cli process ~/datasets/field_a

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Παράδειγμα 7: Χρήση πολλαπλών γλωσσών

Αλλαγή γλώσσας διεπαφής (ίδια σε όλες τις πλατφόρμες):

```bash
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
# Windows: chloros-cli process "C:\Vuelos\Campo_A"
# Linux:   chloros-cli process ~/vuelos/campo_a
chloros-cli process ~/vuelos/campo_a

# Change back to English
chloros-cli language en
```
