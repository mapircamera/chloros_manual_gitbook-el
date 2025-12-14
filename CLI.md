# CLI : Γραμμή εντολών

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>Το **Chloros CLI** παρέχει ισχυρή πρόσβαση στη γραμμή εντολών στον κινητήρα επεξεργασίας εικόνων Chloros, επιτρέποντας την αυτοματοποίηση, τη δημιουργία σεναρίων και την λειτουργία χωρίς οθόνη για τις ροές εργασίας σας στον τομέα της απεικόνισης.

### Βασικά χαρακτηριστικά

* 🚀 **Αυτοματοποίηση** - Σενάρια μαζικής επεξεργασίας πολλαπλών συνόλων δεδομένων
* 🔗 **Ενσωμάτωση** - Ενσωμάτωση σε υπάρχουσες ροές εργασίας και pipelines
* 💻 **Λειτουργία χωρίς οθόνη** - Εκτέλεση χωρίς GUI
* 🌍 **Πολυγλωσσικό** - Υποστήριξη 38 γλωσσών
* ⚡ **Παράλληλη επεξεργασία** - Δυναμική κλιμάκωση ανάλογα με την CPU σας (έως 16 παράλληλοι εργαζόμενοι)

### Απαιτήσεις

| Απαίτηση          | Λεπτομέρειες                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Λειτουργικό σύστημα** | Windows 10/11 (64-bit)                                              |
| **Άδεια χρήσης**          | Chloros+ ([απαιτείται πρόγραμμα επί πληρωμή](https://cloud.mapir.camera/pricing)) |
| **Μνήμη**           | Ελάχιστη μνήμη RAM 8 GB (συνιστάται 16 GB)                                  |
| **Διαδίκτυο**         | Απαιτείται για την ενεργοποίηση της άδειας χρήσης                                     |
| **Χώρος στο δίσκο**       | Διαφέρει ανάλογα με το μέγεθος του έργου                                              |

{% hint style=&quot;warning&quot; %}
**Απαιτήσεις άδειας χρήσης**: Το CLI απαιτεί συνδρομή επί πληρωμή στο Chloros+. Τα βασικά (δωρεάν) πακέτα δεν έχουν πρόσβαση στο CLI. Επισκεφθείτε το [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) για αναβάθμιση.
{% endhint %}

## Γρήγορη εκκίνηση

### Εγκατάσταση

Το CLI περιλαμβάνεται αυτόματα στο πρόγραμμα εγκατάστασης Chloros:

1. Κατεβάστε και εκτελέστε το **Chloros Installer.exe**
2. Ολοκληρώστε τον οδηγό εγκατάστασης
3. CLI εγκατεστημένο σε: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style=&quot;success&quot; %}
Το πρόγραμμα εγκατάστασης προσθέτει αυτόματα το `chloros-cli` στο PATH του συστήματός σας. Επανεκκινήστε το τερματικό σας μετά την εγκατάσταση.
{% endhint %}

### Πρώτη εγκατάσταση

Πριν χρησιμοποιήσετε το CLI, ενεργοποιήστε την άδεια χρήσης Chloros+:

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

### Βασική χρήση

Επεξεργασία φακέλου με τις προεπιλεγμένες ρυθμίσεις:

```powershell
chloros-cli process "C:\Images\Dataset001"
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

**Παράδειγμα:**

```powershell
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance
```

#### Επιλογές εντολών επεξεργασίας

| Επιλογή                | Τύπος    | Προεπιλογή        | Περιγραφή                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Διαδρομή    | _Απαιτείται_     | Φάκελος που περιέχει εικόνες RAW/JPG πολλαπλών φασμάτων                                         |
| `-o, --output`        | Διαδρομή    | Ίδια με την είσοδο  | Φάκελος εξόδου για επεξεργασμένες εικόνες                                                     |
| `-n, --project-name`  | String  | Αυτόματη δημιουργία | Προσαρμοσμένο όνομα έργου                                                                    |
| `--vignette`          | Σημαία    | Ενεργοποιημένη        | Ενεργοποίηση διόρθωσης βινιέτας                                                             |
| `--no-vignette`       | Σημαία    | -              | Απενεργοποίηση διόρθωσης βινιέτας                                                            |
| `--reflectance`       | Σημαία    | Ενεργοποιημένη        | Ενεργοποίηση βαθμονόμησης ανακλαστικότητας                                                         |
| `--no-reflectance`    | Σημαία    | -              | Απενεργοποίηση βαθμονόμησης ανακλαστικότητας                                                        |
| `--ppk`               | Σημαία    | Απενεργοποιημένη       | Εφαρμογή διορθώσεων PPK από δεδομένα αισθητήρα φωτός .daq                                      |
| `--format`            | Επιλογή  | TIFF (16-bit)  | Μορφή εξόδου: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Ακέραιος | Αυτόματο           | Ελάχιστο μέγεθος στόχου σε εικονοστοιχεία για την ανίχνευση του πίνακα βαθμονόμησης                          |
| `--target-clustering` | Ακέραιος | Αυτόματο           | Όριο ομαδοποίησης στόχων (0-100)                                                    |
| `--exposure-pin-1`    | String  | Κανένα           | Κλείδωμα έκθεσης για μοντέλο κάμερας (Pin 1)                                                 |
| `--exposure-pin-2`    | String  | Κανένα           | Κλείδωμα έκθεσης για μοντέλο κάμερας (Pin 2)                                                 |
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

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**Ειδικοί χαρακτήρες**: Χρησιμοποιήστε απλά εισαγωγικά γύρω από κωδικούς πρόσβασης που περιέχουν χαρακτήρες όπως `$`, `!` ή κενά.
{% endhint %}

**Έξοδος:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - Εκκαθάριση διαπιστευτηρίων

Εκκαθαρίστε τα αποθηκευμένα διαπιστευτήρια και αποσυνδεθείτε από τον λογαριασμό σας.

**Σύνταξη:**

```bash
chloros-cli logout
```

**Παράδειγμα:**

```powershell
chloros-cli logout
```

**Έξοδος:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

***

### `status` - Έλεγχος κατάστασης άδειας χρήσης

Εμφάνιση της τρέχουσας άδειας χρήσης και της κατάστασης ελέγχου ταυτότητας.

**Σύνταξη:**

```bash
chloros-cli status
```

**Παράδειγμα:**

```powershell
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

Παρακολούθηση της προόδου εξαγωγής του νήματος 4 κατά τη διάρκεια ή μετά την επεξεργασία.

**Σύνταξη:**

```bash
chloros-cli export-status
```

**Παράδειγμα:**

```powershell
chloros-cli export-status
```

**Περίπτωση χρήσης:** Καλέστε αυτήν την εντολή κατά τη διάρκεια της επεξεργασίας για να ελέγξετε την πρόοδο της εξαγωγής.

***

### `language` - Διαχείριση γλώσσας διεπαφής

Προβολή ή αλλαγή της γλώσσας διεπαφής CLI.

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

```powershell
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

| Κωδικός    | Γλώσσα              | Εγγενής ονομασία      |
| ------- | --------------------- | ---------------- |
| `en`    | Αγγλικά               | Αγγλικά          |
| `es`    | Ισπανικά               | Español          |
| `pt`    | Πορτογαλικά            | Português        |
| `fr`    | Γαλλικά                | Français         |
| `de`    | Γερμανικά                | Deutsch          |
| `it`    | Ιταλικά               | Italiano         |
| `ja`    | Ιαπωνικά              | 日本語              |
| `ko`    | Κορεατικά                | 한국어              |
| `zh`    | Κινέζικα (απλοποιημένα)  | 简体中文             |
| `zh-TW` | Κινέζικα (παραδοσιακά) | 繁體中文             |
| `ru`    | Ρωσικά               | Русский          |
| `nl`    | Ολλανδικά                 | Nederlands       |
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
| `pt-BR` | Βραζιλιάνικα πορτογαλικά  | Português Brasileiro |
| `zh-HK` | Καντονέζικα             | 粵語             |
| `ms`    | Μαλαισιανά                 | Bahasa Melayu    |
| `sk`    | Σλοβακικά                | Slovenčina       |
| `bg`    | Βουλγαρικά             | Български        |
| `hr`    | Κροατικά              | Hrvatski         |
| `lt`    | Λιθουανικά            | Lietuvių         |
| `lv`    | Λετονικά               | Latviešu         |
| `et`    | Εσθονικά              | Eesti            |
| `sl`    | Σλοβενικά             | Slovenščina      |

{% hint style=&quot;success&quot; %}
**Αυτόματη διατήρηση**: Η προτίμηση γλώσσας σας αποθηκεύεται στο `~/.chloros/cli_language.json` και διατηρείται σε όλες τις συνεδρίες.
{% endhint %}

***

### `set-project-folder` - Ορισμός προεπιλεγμένου φακέλου έργου

Αλλάξτε την προεπιλεγμένη θέση του φακέλου έργου (κοινή με το GUI).

**Σύνταξη:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Παράδειγμα:**

```powershell
chloros-cli set-project-folder "C:\Projects\2025"
```

***

### `get-project-folder` - Εμφάνιση φακέλου έργου

Εμφάνιση της τρέχουσας προεπιλεγμένης θέσης του φακέλου έργου.

**Σύνταξη:**

```bash
chloros-cli get-project-folder
```

**Παράδειγμα:**

```powershell
chloros-cli get-project-folder
```

**Έξοδος:**

```
ℹ Current project folder: C:\Projects\2025
```

***

### `reset-project-folder` - Επαναφορά στην προεπιλογή

Επαναφορά του φακέλου έργου στην προεπιλεγμένη θέση.

**Σύνταξη:**

```bash
chloros-cli reset-project-folder
```

***

## Παγκόσμιες επιλογές

Αυτές οι επιλογές ισχύουν για όλες τις εντολές:

| Επιλογή          | Τύπος    | Προεπιλογή       | Περιγραφή                                      |
| --------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe` | Διαδρομή    | Αυτόματη ανίχνευση | Διαδρομή προς το εκτελέσιμο αρχείο του backend                       |
| `--port`        | Ακέραιος αριθμός | 5000          | Αριθμός θύρας του backend API                          |
| `--restart`     | Σημαία    | -             | Αναγκαστική επανεκκίνηση του backend (τερματίζει τις υπάρχουσες διεργασίες) |
| `--version`     | Σημαία    | -             | Εμφάνιση πληροφοριών έκδοσης και έξοδος                |
| `--help`        | Σημαία    | -             | Εμφάνιση πληροφοριών βοήθειας και έξοδος                   |

**Παράδειγμα με καθολικές επιλογές:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

***

## Οδηγός ρυθμίσεων επεξεργασίας

### Παράλληλη επεξεργασία

Chloros+ CLI **αυτοματοποιεί την κλιμάκωση** της παράλληλης επεξεργασίας ώστε να ταιριάζει με τις δυνατότητες του υπολογιστή σας:

**Πώς λειτουργεί:**

* Ανιχνεύει τους πυρήνες της CPU και τη μνήμη RAM
* Κατανομή εργαζομένων: **2× πυρήνες CPU** (χρησιμοποιεί hyperthreading)
* **Μέγιστο: 16 παράλληλοι εργαζόμενοι** (για σταθερότητα)

**Επίπεδα συστήματος:**

| Τύπος συστήματος   | CPU        | RAM      | Εργαζόμενοι  | Απόδοση     |
| ------------- | ---------- | -------- | -------- | --------------- |
| **Υψηλής απόδοσης**  | 16+ πυρήνες  | 32+ GB   | Έως 16 | Μέγιστη ταχύτητα   |
| **Μεσαία** | 8-15 πυρήνες | 16-31 GB | 8-16     | Εξαιρετική ταχύτητα |
| **Χαμηλή**   | 4-7 πυρήνες  | 8-15 GB  | 4-8      | Καλή ταχύτητα      |

{% hint style=&quot;success&quot; %}
**Αυτόματη βελτιστοποίηση**: Το CLI ανιχνεύει αυτόματα τις προδιαγραφές του συστήματός σας και διαμορφώνει τη βέλτιστη παράλληλη επεξεργασία. Δεν απαιτείται χειροκίνητη διαμόρφωση!
{% endhint %}

### Μέθοδοι Debayer

Το CLI χρησιμοποιεί **Υψηλή ποιότητα (ταχύτερη)** ως τον προεπιλεγμένο και συνιστώμενο αλγόριθμο debayer:

| Μέθοδος                      | Ποιότητα | Ταχύτητα | Περιγραφή                                 |
| --------------------------- | ------- | ----- | ------------------------------------------- |
| **Υψηλή ποιότητα (ταχύτερη)** ⭐ | ⭐⭐⭐⭐    | ⚡⚡⚡   | Αλγόριθμος ευαισθησίας στις άκρες (προεπιλεγμένος, συνιστώμενος) |

### Διόρθωση βινιέτας

**Τι κάνει:** Διορθώνει την πτώση του φωτός στις άκρες της εικόνας (σκούρες γωνίες που είναι συνηθισμένες στις εικόνες της κάμερας).

* **Ενεργοποιημένη από προεπιλογή** - Οι περισσότεροι χρήστες θα πρέπει να την διατηρήσουν ενεργοποιημένη
* Χρησιμοποιήστε `--no-vignette` για να την απενεργοποιήσετε

{% hint style=&quot;success&quot; %}
**Σύσταση**: Ενεργοποιείτε πάντα τη διόρθωση βινιέτας για να εξασφαλίσετε ομοιόμορφη φωτεινότητα σε όλο το κάδρο.
{% endhint %}

### Βαθμονόμηση ανακλαστικότητας

Μετατρέπει τις ακατέργαστες τιμές του αισθητήρα σε τυποποιημένα ποσοστά ανακλαστικότητας χρησιμοποιώντας πάνελ βαθμονόμησης.

* **Ενεργοποιημένη από προεπιλογή** - Απαραίτητη για την ανάλυση της βλάστησης.
* Απαιτεί πάνελ στόχων βαθμονόμησης στις εικόνες.
* Χρησιμοποιήστε το `--no-reflectance` για να την απενεργοποιήσετε.

{% hint style=&quot;info&quot; %}
**Απαιτήσεις**: Βεβαιωθείτε ότι τα πάνελ βαθμονόμησης είναι σωστά εκτεθειμένα και ορατά στις εικόνες σας για ακριβή μετατροπή της ανακλαστικότητας.
{% endhint %}

### Διορθώσεις PPK

**Τι κάνει:** Εφαρμόζει διορθώσεις μετα-επεξεργασμένης κινηματικής χρησιμοποιώντας δεδομένα καταγραφής DAQ-A-SD για βελτιωμένη ακρίβεια GPS.

* **Απενεργοποιημένο από προεπιλογή**
* Χρησιμοποιήστε το `--ppk` για να το ενεργοποιήσετε
* Απαιτεί αρχεία .daq στο φάκελο του έργου από τον αισθητήρα φωτός MAPIR DAQ-A-SD.

### Μορφές εξόδου

<table><thead><tr><th width="197">Μορφή</th><th width="130.20001220703125">Βάθος bit</th><th width="116.5999755859375">Μέγεθος αρχείου</th><th>Κατάλληλο για</th></tr></thead><tbody><tr><td><strong>TIFF (16-bit)</strong> ⭐</td><td>16-bit ακέραιος</td><td>Μεγάλο</td><td>Ανάλυση GIS, φωτογραμμετρία (συνιστάται)</td></tr><tr><td><strong>TIFF (32-bit, Ποσοστό)</strong></td><td>32-bit float</td><td>Πολύ μεγάλος</td><td>Επιστημονική ανάλυση, έρευνα</td></tr><tr><td><strong>PNG (8-bit)</strong></td><td>8-bit ακέραιος</td><td>Μεσαία</td><td>Οπτική επιθεώρηση, κοινή χρήση στο διαδίκτυο</td></tr><tr><td><strong>JPG (8-bit)</strong></td><td>Ολοκληρωμένος αριθμός 8 bit</td><td>Μικρό</td><td>Γρήγορη προεπισκόπηση, συμπιεσμένη έξοδος</td></tr></tbody></table>***

## Αυτοματοποίηση και δημιουργία σεναρίων

### Μαζική επεξεργασία PowerShell

Αυτόματη επεξεργασία πολλαπλών φακέλων δεδομένων:

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

### Windows Σενάριο μαζικής επεξεργασίας

Απλός βρόχος για μαζική επεξεργασία:

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

### Python Σενάριο αυτοματοποίησης

Προηγμένη αυτοματοποίηση με διαχείριση σφαλμάτων:

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

## Ροή εργασίας επεξεργασίας

### Τυπική ροή εργασίας

1. **Εισαγωγή**: Φάκελος που περιέχει ζεύγη εικόνων RAW/JPG
2. **Ανακάλυψη**: Το CLI σαρώνει αυτόματα τα υποστηριζόμενα αρχεία εικόνων
3. **Επεξεργασία**: Η παράλληλη λειτουργία προσαρμόζεται στους πυρήνες της CPU σας (Chloros+)
4. **Έξοδος**: Δημιουργεί υποφακέλους μοντέλων κάμερας με τις επεξεργασμένες εικόνες

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

Τυπικοί χρόνοι επεξεργασίας για 100 εικόνες (12MP η καθεμία):

| Λειτουργία              | Χρόνος      | Υλικό                                     |
| ----------------- | --------- | -------------------------------------------- |
| **Παράλληλη λειτουργία** | 5-10 λεπτά  | i7/Ryzen 7, 16 GB RAM, SSD (έως 16 εργαζόμενοι) |
| **Παράλληλη λειτουργία** | 10-15 λεπτά | i5/Ryzen 5, 8 GB RAM, HDD (έως 8 εργαζόμενοι)   |

{% hint style=&quot;info&quot; %}
**Συμβουλή απόδοσης**: Ο χρόνος επεξεργασίας ποικίλλει ανάλογα με τον αριθμό των εικόνων, την ανάλυση και τις προδιαγραφές του υπολογιστή.
{% endhint %}

***

## Αντιμετώπιση προβλημάτων

### CLI Δεν βρέθηκε

**Σφάλμα:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Λύσεις:**

1. Επαληθεύστε τη θέση εγκατάστασης:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Χρησιμοποιήστε την πλήρη διαδρομή εάν δεν βρίσκεται στο PATH:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Προσθέστε στο PATH χειροκίνητα:
   * Ανοίξτε τις Ιδιότητες συστήματος → Μεταβλητές περιβάλλοντος
   * Επεξεργαστείτε τη μεταβλητή PATH
   * Προσθέστε: `C:\Program Files\Chloros\resources\cli`
   * Επανεκκινήστε το τερματικό

***

### Αποτυχία εκκίνησης του backend

**Σφάλμα:**

```
Backend failed to start within 30 seconds
```

**Λύσεις:**

1. Ελέγξτε αν το backend εκτελείται ήδη (κλείστε το πρώτα)
2. Ελέγξτε ότι το Windows Firewall δεν το μπλοκάρει
3. Δοκιμάστε διαφορετική θύρα:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

4. Αναγκάστε την επανεκκίνηση του backend:

```powershell
chloros-cli --restart process "C:\Datasets\Field_A"
```

***

### Προβλήματα άδειας χρήσης/αυθεντικοποίησης

**Σφάλμα:**

```
Chloros+ license required for CLI access
```

**Λύσεις:**

1. Βεβαιωθείτε ότι έχετε ενεργή συνδρομή Chloros+.
2. Συνδεθείτε με τα διαπιστευτήριά σας:

```powershell
chloros-cli login user@example.com 'password'
```

3. Ελέγξτε την κατάσταση της άδειας χρήσης:

```powershell
chloros-cli status
```

4. Επικοινωνήστε με την υποστήριξη: info@mapir.camera

***

### Δεν βρέθηκαν εικόνες

**Σφάλμα:**

```
No images found in the specified folder
```

**Λύσεις:**

1. Βεβαιωθείτε ότι ο φάκελος περιέχει υποστηριζόμενες μορφές (.RAW, .TIF, .JPG)
2. Ελέγξτε ότι η διαδρομή του φακέλου είναι σωστή (χρησιμοποιήστε εισαγωγικά για διαδρομές με κενά)
3. Βεβαιωθείτε ότι έχετε δικαιώματα ανάγνωσης για το φάκελο.
4. Ελέγξτε ότι οι επεκτάσεις αρχείων είναι σωστές.

***

### Η επεξεργασία σταματά ή κολλάει

**Λύσεις:**

1. Ελέγξτε τον διαθέσιμο χώρο στο δίσκο (βεβαιωθείτε ότι είναι αρκετός για την έξοδο).
2. Κλείστε άλλες εφαρμογές για να ελευθερώσετε μνήμη.
3. Μειώστε τον αριθμό των εικόνων (επεξεργαστείτε τις σε παρτίδες).

***

### Η θύρα χρησιμοποιείται ήδη

**Σφάλμα:**

```
Port 5000 is already in use
```

**Λύση:**

Καθορίστε μια διαφορετική θύρα:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

***

## Συχνές ερωτήσεις

### Ε: Χρειάζομαι άδεια χρήσης για το CLI;

**Α:** Ναι! Το CLI απαιτεί μια πληρωμένη **άδεια Chloros+**.

* ❌ Πρότυπο (δωρεάν) πρόγραμμα: CLI απενεργοποιημένο
* ✅ Πακέτα Chloros+ (επί πληρωμή): Το CLI είναι πλήρως ενεργοποιημένο

Εγγραφείτε στο: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### Ε: Μπορώ να χρησιμοποιήσω το CLI σε έναν διακομιστή χωρίς GUI;

**Α:** Ναι! Το CLI λειτουργεί εντελώς χωρίς οθόνη. Απαιτήσεις:

* Windows Server 2016 ή νεότερη έκδοση
* Εγκατεστημένο Visual C++ Redistributable
* Επαρκής μνήμη RAM (ελάχιστο 8 GB, συνιστάται 16 GB)
* Εφάπαξ ενεργοποίηση άδειας GUI σε οποιονδήποτε υπολογιστή

***

### Ε: Πού αποθηκεύονται οι επεξεργασμένες εικόνες;

**Α:** Από προεπιλογή, οι επεξεργασμένες εικόνες αποθηκεύονται στον **ίδιο φάκελο με την είσοδο** σε υποφακέλους μοντέλων κάμερας (π.χ. `Survey3N_RGN/`).

Χρησιμοποιήστε την επιλογή `-o` για να καθορίσετε διαφορετικό φάκελο εξόδου:

```powershell
chloros-cli process "C:\Input" -o "D:\Output"
```

***

### Ε: Μπορώ να επεξεργαστώ πολλούς φακέλους ταυτόχρονα;

**Α:** Όχι απευθείας με μία εντολή, αλλά μπορείτε να χρησιμοποιήσετε σενάρια για να επεξεργαστείτε τους φακέλους διαδοχικά. Ανατρέξτε στην ενότητα [Αυτοματοποίηση και σενάρια](CLI.md#automation--scripting).

***

### Ε: Πώς μπορώ να αποθηκεύσω την έξοδο CLI σε ένα αρχείο καταγραφής;

**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

***

### Ε: Τι συμβαίνει αν πατήσω Ctrl+C κατά τη διάρκεια της επεξεργασίας;

**Α:** Το CLI θα:

1. Σταματήσει την επεξεργασία ομαλά
2. Κλείσει το backend
3. Κλείσει με κωδικό 130

Εικόνες που έχουν υποστεί μερική επεξεργασία ενδέχεται να παραμείνουν στον φάκελο εξόδου.

***

### Ε: Μπορώ να αυτοματοποιήσω την επεξεργασία του CLI;

**Α:** Φυσικά! Το CLI έχει σχεδιαστεί για αυτοματοποίηση. Δείτε [Αυτοματοποίηση &amp; Σενάρια](CLI.md#automation--scripting) για παραδείγματα PowerShell, Batch και Python.

***

### Ε: Πώς μπορώ να ελέγξω την έκδοση CLI;

**Α:**

```powershell
chloros-cli --version
```

**Έξοδος:**

```
Chloros CLI 1.0.2
```

***

## Λήψη βοήθειας

### Βοήθεια γραμμής εντολών

Δείτε τις πληροφορίες βοήθειας απευθείας στο CLI:

```powershell
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### Κανάλια υποστήριξης

* **Ηλεκτρονικό ταχυδρομείο**: info@mapir.camera
* **Ιστοσελίδα**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Τιμές**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

## Πλήρη παραδείγματα

### Παράδειγμα 1: Βασική επεξεργασία

Επεξεργασία με προεπιλεγμένες ρυθμίσεις (βινιέτα, ανακλαστικότητα):

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

***

### Παράδειγμα 2: Επιστημονικά αποτελέσματα υψηλής ποιότητας

32-bit float TIFF:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

***

### Παράδειγμα 3: Γρήγορη επεξεργασία προεπισκόπησης

8-bit PNG χωρίς βαθμονόμηση για γρήγορη ανασκόπηση:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

***

### Παράδειγμα 4: Επεξεργασία με διόρθωση PPK

Εφαρμογή διορθώσεων PPK με ανακλαστικότητα:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

***

### Παράδειγμα 5: Προσαρμοσμένη θέση εξόδου

Επεξεργασία σε διαφορετικό δίσκο με συγκεκριμένη μορφή:

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

***

### Παράδειγμα 6: Ροή εργασίας πιστοποίησης

Ολοκλήρωση ροής πιστοποίησης:

```powershell
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
chloros-cli process "C:\Datasets\Field_A"

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Παράδειγμα 7: Χρήση πολλαπλών γλωσσών

Αλλαγή γλώσσας διεπαφής:

```powershell
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
chloros-cli process "C:\Vuelos\Campo_A"

# Change back to English
chloros-cli language en
```
