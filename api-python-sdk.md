# API : Python SDK

Το **Chloros Python SDK** παρέχει προγραμματιστική πρόσβαση στον κινητήρα επεξεργασίας εικόνων Chloros, επιτρέποντας την αυτοματοποίηση, τις προσαρμοσμένες ροές εργασίας και την απρόσκοπτη ενσωμάτωση με τις εφαρμογές Python και τις ερευνητικές σας διαδικασίες.

### Βασικά χαρακτηριστικά

* 🐍 **Εγγενές Python** - Καθαρό, Pythonic API για επεξεργασία εικόνων
* 🔧 **Πλήρης πρόσβαση στο API** - Πλήρης έλεγχος της επεξεργασίας Chloros
* 🚀 **Αυτοματοποίηση** - Δημιουργία προσαρμοσμένων ροών εργασίας μαζικής επεξεργασίας
* 🔗 **Ενσωμάτωση** - Ενσωματώστε το Chloros σε υπάρχουσες εφαρμογές Python
* 📊 **Έτοιμο για έρευνα** - Ιδανικό για επιστημονικές αναλύσεις
* ⚡ **Παράλληλη επεξεργασία** - Κλιμακώνεται ανάλογα με τους πυρήνες της CPU σας (Chloros+)

### Απαιτήσεις

| Απαίτηση          | Λεπτομέρειες                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros Desktop**  | Πρέπει να είναι εγκατεστημένο τοπικά                                           |
| **Άδεια χρήσης**          | Chloros+ ([απαιτείται πρόγραμμα επί πληρωμή](https://cloud.mapir.camera/pricing)) |
| **Λειτουργικό σύστημα** | Windows 10/11 (64-bit)                                              |
| **Python**           | Python 3.7 ή νεότερη έκδοση                                                |
| **Μνήμη**           | Ελάχιστη μνήμη RAM 8 GB (συνιστάται 16 GB)                                  |
| **Διαδίκτυο**         | Απαιτείται για την ενεργοποίηση της άδειας χρήσης                                     |

{% hint style=&quot;warning&quot; %}
**Απαιτήσεις άδειας χρήσης**: Το Python SDK απαιτεί συνδρομή Chloros+ επί πληρωμή για πρόσβαση στο API. Τα βασικά (δωρεάν) πακέτα δεν έχουν πρόσβαση στο API/SDK. Επισκεφθείτε το [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) για αναβάθμιση.
{% endhint %}

## Γρήγορη εκκίνηση

### Εγκατάσταση

Εγκαταστήστε μέσω pip:

```bash
pip install chloros-sdk
```

{% hint style=&quot;info&quot; %}
**Πρώτη ρύθμιση**: Πριν χρησιμοποιήσετε το SDK, ενεργοποιήστε την άδεια χρήσης Chloros+ ανοίγοντας το Chloros, Chloros (Browser) ή το Chloros CLI και συνδεθείτε με τα διαπιστευτήριά σας. Αυτό χρειάζεται να γίνει μόνο μία φορά.
{% endhint %}

### Βασική χρήση

Επεξεργαστείτε ένα φάκελο με λίγες μόνο γραμμές:

```python
from chloros_sdk import process_folder

# One-line processing
results = process_folder("C:\\DroneImages\\Flight001")
```

### Πλήρης έλεγχος

Για προηγμένες ροές εργασίας:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")

# Configure settings
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE", "GNDVI"]
)

# Process images
chloros.process(mode="parallel", wait=True)
```

***

## Οδηγός εγκατάστασης

### Προαπαιτούμενα

Πριν εγκαταστήσετε το SDK, βεβαιωθείτε ότι διαθέτετε:

1. **Chloros Desktop** εγκατεστημένο ([download](download.md))
2. **Python 3.7+** εγκατεστημένο ([python.org](https://www.python.org))
3. **Ενεργή άδεια Chloros+** ([αναβάθμιση](https://cloud.mapir.camera/pricing))

### Εγκατάσταση μέσω pip

**Τυπική εγκατάσταση:**

```bash
pip install chloros-sdk
```

**Με υποστήριξη παρακολούθησης προόδου:**

```bash
pip install chloros-sdk[progress]
```

**Εγκατάσταση ανάπτυξης:**

```bash
pip install chloros-sdk[dev]
```

### Επαλήθευση εγκατάστασης

Ελέγξτε ότι το SDK έχει εγκατασταθεί σωστά:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## Πρώτη ρύθμιση

### Ενεργοποίηση άδειας χρήσης

Το SDK χρησιμοποιεί την ίδια άδεια χρήσης με τα Chloros, Chloros (Browser) και Chloros CLI. Ενεργοποιήστε μία φορά μέσω του GUI ή του CLI:

1. Ανοίξτε το **Chloros ή το Chloros (Browser)** και συνδεθείτε στην καρτέλα Χρήστη <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> . Εναλλακτικά, ανοίξτε το **CLI**.
2. Εισαγάγετε τα διαπιστευτήριά σας Chloros+ και συνδεθείτε
3. Η άδεια αποθηκεύεται τοπικά (διατηρείται μετά την επανεκκίνηση)

{% hint style=&quot;success&quot; %}
**Μία φορά ρύθμιση**: Αφού συνδεθείτε μέσω του GUI ή του CLI, το SDK χρησιμοποιεί αυτόματα την αποθηκευμένη άδεια χρήσης. Δεν απαιτείται πρόσθετη πιστοποίηση!
{% endhint %}

### Δοκιμή σύνδεσης

Επαληθεύστε ότι το SDK μπορεί να συνδεθεί στο Chloros:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK (auto-starts backend if needed)
chloros = ChlorosLocal()

# Check status
status = chloros.get_status()
print(f"Backend running: {status['running']}")
```

***

## Αναφορά API

### ChlorosLocal Class

Κύρια κλάση για τοπική επεξεργασία εικόνας Chloros.

#### Κατασκευαστής

```python
ChlorosLocal(
    api_url="http://localhost:5000",     # Backend URL
    auto_start_backend=True,             # Auto-start backend if not running
    backend_exe=None,                    # Backend path (auto-detected)
    timeout=30,                          # Request timeout (seconds)
    backend_startup_timeout=60           # Backend startup timeout
)
```

**Παράμετροι:**

| Παράμετρος                 | Τύπος | Προεπιλογή                   | Περιγραφή                           |
| ------------------------- | ---- | ------------------------- | ------------------------------------- |
| `api_url`                 | str  | `"http://localhost:5000"` | URL του τοπικού Chloros backend          |
| `auto_start_backend`      | bool | `True`                    | Αυτόματη εκκίνηση του backend αν χρειαστεί |
| `backend_exe`             | str  | `None` (αυτόματη ανίχνευση)      | Διαδρομή προς το εκτελέσιμο backend            |
| `timeout`                 | int  | `30`                      | Χρονικό όριο αιτήματος σε δευτερόλεπτα            |
| `backend_startup_timeout` | int  | `60`                      | Χρονικό όριο για την εκκίνηση του backend (δευτερόλεπτα) |

**Παραδείγματα:**

```python
# Default (auto-start backend)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom timeout
chloros = ChlorosLocal(timeout=60)
```

***

### Μέθοδοι

#### `create_project(project_name, camera=None)`

Δημιουργία νέου έργου Chloros.

**Παράμετροι:**

| Παράμετρος      | Τύπος | Απαιτείται | Περιγραφή                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | Ναι      | Όνομα για το έργο                                     |
| `camera`       | str  | Όχι       | Πρότυπο κάμερας (π.χ. &quot;Survey3N\_RGN&quot;, &quot;Survey3W\_OCN&quot;) |

**Επιστρέφει:** `dict` - Απόκριση δημιουργίας έργου

**Παράδειγμα:**

```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```

***

#### `import_images(folder_path, recursive=False)`

Εισαγωγή εικόνων από φάκελο.

**Παράμετροι:**

| Παράμετρος     | Τύπος     | Απαιτείται | Περιγραφή                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/Path | Ναι      | Διαδρομή προς φάκελο με εικόνες         |
| `recursive`   | bool     | Όχι       | Αναζήτηση υποφακέλων (προεπιλογή: False) |

**Επιστρέφει:** `dict` - Εισαγωγή αποτελεσμάτων με αριθμό αρχείων

**Παράδειγμα:**

```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```

***

#### `configure(**settings)`

Διαμόρφωση ρυθμίσεων επεξεργασίας.

**Παράμετροι:**

| Παράμετρος                 | Τύπος | Προεπιλογή                 | Περιγραφή                     |
| ------------------------- | ---- | ----------------------- | ------------------------------- |
| `debayer`                 | str  | &quot;Υψηλή ποιότητα (ταχύτερη)&quot; | Μέθοδος Debayer                  |
| `vignette_correction`     | bool | `True`                  | Ενεργοποίηση διόρθωσης βινιέτας      |
| `reflectance_calibration` | bool | `True`                  | Ενεργοποίηση βαθμονόμησης ανακλαστικότητας  |
| `indices`                 | λίστα | `None`                  | Δείκτες βλάστησης προς υπολογισμό |
| `export_format`           | str  | &quot;TIFF (16-bit)&quot;         | Μορφή εξόδου                   |
| `ppk`                     | bool | `False`                 | Ενεργοποίηση διορθώσεων PPK          |
| `custom_settings`         | dict | `None`                  | Προηγμένες προσαρμοσμένες ρυθμίσεις        |

**Μορφές εξαγωγής:**

* `"TIFF (16-bit)"` - Συνιστάται για GIS/φωτογραμμετρία
* `"TIFF (32-bit, Percent)"` - Επιστημονική ανάλυση
* `"PNG (8-bit)"` - Οπτική επιθεώρηση
* `"JPG (8-bit)"` - Συμπιεσμένη έξοδος

**Διαθέσιμοι δείκτες:**

NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2 και άλλα.

**Παράδειγμα:**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="High Quality (Faster)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=True,
    export_format="TIFF (32-bit, Percent)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI", "CIG"]
)
```

***

#### `process(mode="parallel", wait=True, progress_callback=None)`

Επεξεργασία των εικόνων του έργου.

**Παράμετροι:**

| Παράμετρος           | Τύπος     | Προεπιλογή      | Περιγραφή                               |
| ------------------- | -------- | ------------ | ----------------------------------------- |
| `mode`              | str      | `"parallel"` | Λειτουργία επεξεργασίας: &quot;parallel&quot; ή &quot;serial&quot;   |
| `wait`              | bool     | `True`       | Αναμονή για ολοκλήρωση                       |
| `progress_callback` | callable | `None`       | Λειτουργία επιστροφής κλήσης προόδου (progress, msg) |
| `poll_interval`     | float    | `2.0`        | Διάστημα δειγματοληψίας για την πρόοδο (δευτερόλεπτα)   |

**Επιστρέφει:** `dict` - Αποτελέσματα επεξεργασίας

{% hint style=&quot;warning&quot; %}
**Παράλληλη λειτουργία**: Απαιτείται άδεια Chloros+. Προσαρμόζεται αυτόματα στους πυρήνες της CPU (έως 16 εργαζόμενους).
{% endhint %}

**Παράδειγμα:**

```python
# Simple processing
results = chloros.process()

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

# Fire-and-forget (non-blocking)
chloros.process(wait=False)
```

***

#### `get_config()`

Λήψη τρέχουσας διαμόρφωσης έργου.

**Επιστρέφει:** `dict` - Τρέχουσα διαμόρφωση έργου

**Παράδειγμα:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

Λήψη πληροφοριών κατάστασης backend.

**Επιστρέφει:** `dict` - Κατάσταση του backend

**Παράδειγμα:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
```

***

#### `shutdown_backend()`

Τερματίζει το backend (εάν έχει ξεκινήσει από το SDK).

**Παράδειγμα:**

```python
chloros.shutdown_backend()
```

***

### Λειτουργίες ευκολίας

#### `process_folder(folder_path, **options)`

Λειτουργία ευκολίας μιας γραμμής για την επεξεργασία ενός φακέλου.

**Παράμετροι:**

| Παράμετρος                 | Τύπος     | Προεπιλογή         | Περιγραφή                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Path | Απαιτείται        | Διαδρομή προς φάκελο με εικόνες     |
| `project_name`            | str      | Αυτόματη δημιουργία  | Όνομα έργου                   |
| `camera`                  | str      | `None`          | Πρότυπο κάμερας                |
| `indices`                 | list     | `["NDVI"]`      | Δείκτες για υπολογισμό           |
| `vignette_correction`     | bool     | `True`          | Ενεργοποίηση διόρθωσης βινιέτας     |
| `reflectance_calibration` | bool     | `True`          | Ενεργοποίηση βαθμονόμησης ανακλαστικότητας |
| `export_format`           | str      | &quot;TIFF (16-bit)&quot; | Μορφή εξόδου                  |
| `mode`                    | str      | `"parallel"`    | Λειτουργία επεξεργασίας                |
| `progress_callback`       | callable | `None`          | Επιστροφή κλήσης προόδου              |

**Επιστρέφει:** `dict` - Αποτελέσματα επεξεργασίας

**Παράδειγμα:**

```python
from chloros_sdk import process_folder

# Simple one-liner
results = process_folder("C:\\DroneImages\\Flight001")

# With custom settings
results = process_folder(
    "C:\\DroneImages\\Flight001",
    project_name="Field_A_Survey",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    mode="parallel"
)

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

results = process_folder(
    "C:\\DroneImages\\Flight001",
    progress_callback=show_progress
)
```

***

## Υποστήριξη διαχειριστή περιβάλλοντος

Το SDK υποστηρίζει διαχειριστές περιβάλλοντος για αυτόματο καθαρισμό:

```python
from chloros_sdk import ChlorosLocal

# Auto-cleanup when done
with ChlorosLocal() as chloros:
    chloros.create_project("MyProject")
    chloros.import_images("C:\\Images")
    chloros.configure(indices=["NDVI"])
    chloros.process()
# Backend automatically shut down here
```

***

## Πλήρη παραδείγματα

### Παράδειγμα 1: Βασική επεξεργασία

Επεξεργασία φακέλου με τις προεπιλεγμένες ρυθμίσεις:

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### Παράδειγμα 2: Προσαρμοσμένη ροή εργασίας

Πλήρης έλεγχος της ροής επεξεργασίας:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project with camera template
chloros.create_project("Research_Plot_A", camera="Survey3N_RGN")

# Import images
import_results = chloros.import_images("C:\\Research\\PlotA")
print(f"Imported {len(import_results.get('files', []))} images")

# Configure advanced settings
chloros.configure(
    debayer="High Quality (Faster)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=False,
    export_format="TIFF (16-bit)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI"]
)

# Process with progress monitoring
def show_progress(progress, message):
    print(f"Progress: {progress}% - {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

print("Processing complete!")
```

***

### Παράδειγμα 3: Μαζική επεξεργασία πολλαπλών φακέλων

Επεξεργασία πολλαπλών συνόλων δεδομένων πτήσεων:

```python
from chloros_sdk import ChlorosLocal
from pathlib import Path

# Initialize SDK once
chloros = ChlorosLocal()

# List of flight folders
flights = [
    "C:\\Datasets\\Flight_001",
    "C:\\Datasets\\Flight_002",
    "C:\\Datasets\\Flight_003"
]

for flight_path in flights:
    flight_name = Path(flight_path).name
    print(f"\n{'='*60}")
    print(f"Processing: {flight_name}")
    print('='*60)
    
    try:
        # Create project
        chloros.create_project(flight_name, camera="Survey3N_RGN")
        
        # Import images
        chloros.import_images(flight_path)
        
        # Configure
        chloros.configure(
            vignette_correction=True,
            reflectance_calibration=True,
            indices=["NDVI", "NDRE", "GNDVI"]
        )
        
        # Process
        chloros.process(mode="parallel", wait=True)
        
        print(f"✓ {flight_name} completed successfully")
    
    except Exception as e:
        print(f"✗ {flight_name} failed: {e}")

print("\n" + "="*60)
print("All flights processed!")
```

***

### Παράδειγμα 4: Ενσωμάτωση ροής έρευνας

Ενσωμάτωση του Chloros με ανάλυση δεδομένων:

```python
from chloros_sdk import ChlorosLocal
import pandas as pd
import matplotlib.pyplot as plt

# Initialize Chloros
chloros = ChlorosLocal()

# Field survey data
surveys = [
    {"name": "Plot_A", "folder": "C:\\Research\\PlotA", "biomass": 4500},
    {"name": "Plot_B", "folder": "C:\\Research\\PlotB", "biomass": 3800},
    {"name": "Plot_C", "folder": "C:\\Research\\PlotC", "biomass": 5200}
]

results = []

for survey in surveys:
    # Process with Chloros
    chloros.create_project(survey['name'])
    chloros.import_images(survey['folder'])
    chloros.configure(indices=["NDVI", "NDRE"])
    chloros.process(mode="parallel", wait=True)
    
    # Get results
    config = chloros.get_config()
    
    # Extract NDVI values (example - adjust based on your needs)
    # In real implementation, you would read the processed TIFF files
    
    results.append({
        'plot': survey['name'],
        'biomass': survey['biomass'],
        # Add your NDVI extraction here
    })

# Statistical analysis
df = pd.DataFrame(results)
print("\nResults:")
print(df)

# Create correlation plot
# plt.scatter(df['ndvi'], df['biomass'])
# plt.xlabel('NDVI')
# plt.ylabel('Biomass (kg/ha)')
# plt.title('NDVI vs Biomass Correlation')
# plt.show()
```

***

### Παράδειγμα 5: Προσαρμοσμένη παρακολούθηση προόδου

Προηγμένη παρακολούθηση προόδου με καταγραφή:

```python
from chloros_sdk import ChlorosLocal
from datetime import datetime
import logging

# Setup logging
logging.basicConfig(
    filename=f'processing_{datetime.now():%Y%m%d_%H%M%S}.log',
    level=logging.INFO,
    format='%(asctime)s - %(message)s'
)

# Progress callback with logging
def log_progress(progress, message):
    log_msg = f"[{progress}%] {message}"
    logging.info(log_msg)
    print(log_msg)

# Process with logging
chloros = ChlorosLocal()
chloros.create_project("LoggedProcess")
chloros.import_images("C:\\DroneImages")
chloros.configure(indices=["NDVI", "NDRE"])

logging.info("Starting processing...")
chloros.process(
    mode="parallel",
    progress_callback=log_progress,
    wait=True
)
logging.info("Processing complete!")
```

***

### Παράδειγμα 6: Αντιμετώπιση σφαλμάτων

Σταθερή αντιμετώπιση σφαλμάτων για χρήση σε παραγωγή:

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import (
    ChlorosError,
    ChlorosBackendError,
    ChlorosLicenseError,
    ChlorosProcessingError
)

def process_safely(folder_path):
    """Process with comprehensive error handling"""
    try:
        with ChlorosLocal() as chloros:
            chloros.create_project("SafeProcess")
            chloros.import_images(folder_path)
            chloros.configure(indices=["NDVI"])
            chloros.process()
            
        return True, "Success"
    
    except ChlorosLicenseError as e:
        return False, f"License error: {e}. Upgrade to Chloros+ at cloud.mapir.camera/pricing"
    
    except ChlorosBackendError as e:
        return False, f"Backend error: {e}. Ensure Chloros Desktop is installed."
    
    except ChlorosProcessingError as e:
        return False, f"Processing error: {e}"
    
    except FileNotFoundError as e:
        return False, f"Folder not found: {e}"
    
    except ChlorosError as e:
        return False, f"Chloros error: {e}"
    
    except Exception as e:
        return False, f"Unexpected error: {e}"

# Use the safe function
success, message = process_safely("C:\\DroneImages\\Flight001")
if success:
    print(f"✓ {message}")
else:
    print(f"✗ {message}")
```

***

### Παράδειγμα 7: Εργαλείο γραμμής εντολών

Δημιουργήστε ένα προσαρμοσμένο εργαλείο CLI με το SDK:

```python
#!/usr/bin/env python
"""
Custom Chloros CLI Tool
Process multiple folders from command line
"""

import sys
import argparse
from pathlib import Path
from chloros_sdk import process_folder

def main():
    parser = argparse.ArgumentParser(description='Custom Chloros Processor')
    parser.add_argument('folders', nargs='+', help='Folders to process')
    parser.add_argument('--indices', nargs='+', default=['NDVI'],
                       help='Indices to calculate (default: NDVI)')
    parser.add_argument('--camera', default=None,
                       help='Camera template')
    parser.add_argument('--format', default='TIFF (16-bit)',
                       help='Export format')
    
    args = parser.parse_args()
    
    successful = []
    failed = []
    
    for folder in args.folders:
        folder_path = Path(folder)
        
        if not folder_path.exists():
            print(f"✗ Skipping {folder}: not found")
            failed.append(folder)
            continue
        
        print(f"\nProcessing: {folder_path.name}...")
        
        try:
            process_folder(
                folder_path,
                camera=args.camera,
                indices=args.indices,
                export_format=args.format
            )
            print(f"✓ {folder_path.name} complete")
            successful.append(folder)
        
        except Exception as e:
            print(f"✗ {folder_path.name} failed: {e}")
            failed.append(folder)
    
    # Summary
    print(f"\n{'='*60}")
    print(f"Summary: {len(successful)} successful, {len(failed)} failed")
    
    return 0 if not failed else 1

if __name__ == '__main__':
    sys.exit(main())
```

**Χρήση:**

```bash
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI
```

***

## Χειρισμός εξαιρέσεων

Το SDK παρέχει συγκεκριμένες κατηγορίες εξαιρέσεων για διαφορετικούς τύπους σφαλμάτων:

### Ιεραρχία εξαιρέσεων

```python
ChlorosError                    # Base exception
├── ChlorosBackendError        # Backend startup/connection issues
├── ChlorosLicenseError        # License validation issues
├── ChlorosConnectionError     # Network/connection failures
├── ChlorosProcessingError     # Image processing failures
├── ChlorosAuthenticationError # Authentication failures
└── ChlorosConfigurationError  # Configuration errors
```

### Παραδείγματα εξαιρέσεων

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import *

try:
    chloros = ChlorosLocal()
    chloros.process()

except ChlorosLicenseError:
    print("Chloros+ license required. Upgrade at cloud.mapir.camera/pricing")

except ChlorosBackendError:
    print("Backend failed to start. Ensure Chloros Desktop is installed.")

except ChlorosProcessingError as e:
    print(f"Processing failed: {e}")

except ChlorosError as e:
    print(f"General Chloros error: {e}")
```

***

## Προχωρημένα θέματα

### Προσαρμοσμένη διαμόρφωση backend

Χρησιμοποιήστε μια προσαρμοσμένη θέση ή διαμόρφωση backend:

```python
chloros = ChlorosLocal(
    backend_exe="C:\\Custom\\chloros-backend.exe",
    api_url="http://localhost:5001",  # Custom port
    timeout=60,                        # Longer timeout
    backend_startup_timeout=120        # 2 minutes startup
)
```

### Επεξεργασία χωρίς μπλοκάρισμα

Ξεκινήστε την επεξεργασία και συνεχίστε με άλλες εργασίες:

```python
# Start processing (non-blocking)
chloros.process(wait=False)

# Do other work here...
print("Processing started in background...")

# Check status later
import time
while True:
    status = chloros.get_config()
    if status.get('processing_complete'):
        break
    time.sleep(5)

print("Processing complete!")
```

### Διαχείριση μνήμης

Για μεγάλα σύνολα δεδομένων, επεξεργαστείτε τα σε παρτίδες:

```python
from pathlib import Path

base_folder = Path("C:\\LargeDataset")
batch_size = 100

# Get all image files
images = list(base_folder.glob("*.RAW"))

# Process in batches
for i in range(0, len(images), batch_size):
    batch = images[i:i+batch_size]
    batch_folder = base_folder / f"batch_{i//batch_size}"
    
    # Create batch folder and move images
    # ... (implementation details)
    
    # Process batch
    process_folder(batch_folder)
```

***

## Αντιμετώπιση προβλημάτων

### Το backend δεν ξεκινά

**Πρόβλημα:** Το SDK δεν ξεκινά το backend.

**Λύσεις:**

1. Βεβαιωθείτε ότι το Chloros Desktop είναι εγκατεστημένο:

```python
import os
backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Ελέγξτε ότι το Windows Firewall δεν το μπλοκάρει
3. Δοκιμάστε τη χειροκίνητη διαδρομή του backend:

```python
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")
```

***

### Η άδεια χρήσης δεν ανιχνεύεται

**Πρόβλημα:** Το SDK προειδοποιεί για την έλλειψη άδειας χρήσης

**Λύσεις:**

1. Ανοίξτε το Chloros, το Chloros (πρόγραμμα περιήγησης) ή το Chloros CLI και συνδεθείτε.
2. Βεβαιωθείτε ότι η άδεια χρήσης είναι αποθηκευμένη στην προσωρινή μνήμη:

```python
from pathlib import Path
import os

# Check cache location (Windows)
cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
print(f"Cache exists: {cache_path.exists()}")
```

3. Επικοινωνήστε με την υποστήριξη: info@mapir.camera

***

### Σφάλματα εισαγωγής

**Πρόβλημα:** `ModuleNotFoundError: No module named 'chloros_sdk'`

**Λύσεις:**

```bash
# Verify installation
pip show chloros-sdk

# Reinstall if needed
pip uninstall chloros-sdk
pip install chloros-sdk

# Check Python environment
python -c "import sys; print(sys.path)"
```

***

### Χρονικό όριο επεξεργασίας

**Πρόβλημα:** Χρονικό όριο επεξεργασίας

**Λύσεις:**

1. Αυξήστε το χρονικό όριο:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. Επεξεργαστείτε μικρότερες παρτίδες
3. Ελέγξτε τον διαθέσιμο χώρο στο δίσκο
4. Παρακολουθήστε τους πόρους του συστήματος

***

### Θύρα ήδη σε χρήση

**Πρόβλημα:** Η θύρα 5000 του backend είναι κατειλημμένη

**Λύσεις:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

Ή εντοπίστε και κλείστε τη διεργασία που προκαλεί τη σύγκρουση:

```powershell
# PowerShell
Get-NetTCPConnection -LocalPort 5000
```

***

## Συμβουλές απόδοσης

### Βελτιστοποίηση ταχύτητας επεξεργασίας

1. **Χρησιμοποιήστε την παράλληλη λειτουργία** (απαιτείται Chloros+)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **Μειώστε την ανάλυση εξόδου** (εάν είναι αποδεκτό)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **Απενεργοποιήστε τους περιττούς δείκτες**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **Επεξεργασία σε SSD** (όχι HDD)

***

### Βελτιστοποίηση μνήμης

Για μεγάλα σύνολα δεδομένων:

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***

### Επεξεργασία στο παρασκήνιο

Απελευθερώστε Python για άλλες εργασίες:

```python
chloros.process(wait=False)  # Non-blocking

# Continue with other work
# ...
```

***

## Παραδείγματα ενσωμάτωσης

### Ενσωμάτωση Django

```python
# views.py
from django.http import JsonResponse
from chloros_sdk import process_folder

def process_images_view(request):
    if request.method == 'POST':
        folder_path = request.POST.get('folder_path')
        
        try:
            results = process_folder(folder_path)
            return JsonResponse({'success': True, 'results': results})
        except Exception as e:
            return JsonResponse({'success': False, 'error': str(e)})
```

### Flask API

```python
# app.py
from flask import Flask, request, jsonify
from chloros_sdk import process_folder

app = Flask(__name__)

@app.route('/api/process', methods=['POST'])
def process():
    data = request.get_json()
    folder_path = data.get('folder_path')
    
    try:
        results = process_folder(folder_path)
        return jsonify({'success': True, 'results': results})
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 500

if __name__ == '__main__':
    app.run()
```

### Jupyter Notebook

```python
# notebook.ipynb
from chloros_sdk import ChlorosLocal
import matplotlib.pyplot as plt

# Initialize
chloros = ChlorosLocal()

# Process
chloros.create_project("JupyterTest")
chloros.import_images("C:\\Data")
chloros.configure(indices=["NDVI"])

# Progress in notebook
from IPython.display import clear_output

def notebook_progress(progress, message):
    clear_output(wait=True)
    print(f"Progress: {progress}%")
    print(message)

chloros.process(progress_callback=notebook_progress)

# Visualize results
# ... (your visualization code)
```

***

## Συχνές ερωτήσεις

### Ε: Το SDK απαιτεί σύνδεση στο διαδίκτυο;

**Α:** Μόνο για την αρχική ενεργοποίηση της άδειας χρήσης. Μετά τη σύνδεση μέσω Chloros, Chloros (Browser) ή Chloros CLI, η άδεια αποθηκεύεται τοπικά και λειτουργεί εκτός σύνδεσης για 30 ημέρες.

***

### Ε: Μπορώ να χρησιμοποιήσω το SDK σε έναν διακομιστή χωρίς GUI;

**Α:** Ναι! Απαιτήσεις:

* Windows Server 2016 ή νεότερη έκδοση
* Chloros εγκατεστημένο (μία φορά)
* Άδεια χρήσης ενεργοποιημένη σε οποιονδήποτε υπολογιστή (άδεια χρήσης αποθηκευμένη στο cache και αντιγραμμένη στον διακομιστή)

***

### Ε: Ποια είναι η διαφορά μεταξύ Desktop, CLI και SDK;

| Χαρακτηριστικό         | Desktop GUI | CLI Γραμμή εντολών | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **Διεπαφή**   | Point-click | Εντολή          | Python API  |
| **Κατάλληλο για**    | Οπτική εργασία | Σενάρια        | Ενσωμάτωση |
| **Αυτοματοποίηση**  | Περιορισμένη     | Καλή             | Εξαιρετική   |
| **Ευελιξία** | Βασική       | Καλή             | Μέγιστη     |
| **Άδεια χρήσης**     | Chloros+    | Chloros+         | Chloros+    |

***

### Ε: Μπορώ να διανέμω εφαρμογές που έχουν δημιουργηθεί με το SDK;

**Α:** Ο κώδικας SDK μπορεί να ενσωματωθεί στις εφαρμογές σας, αλλά:

* Οι τελικοί χρήστες πρέπει να έχουν εγκατεστημένο το Chloros.
* Οι τελικοί χρήστες πρέπει να έχουν ενεργές άδειες χρήσης Chloros+.
* Η εμπορική διανομή απαιτεί άδεια χρήσης OEM.

Επικοινωνήστε με το info@mapir.camera για ερωτήσεις σχετικά με το OEM.

***

### Ε: Πώς μπορώ να ενημερώσω το SDK;

```bash
pip install --upgrade chloros-sdk
```

***

### Ε: Πού αποθηκεύονται οι επεξεργασμένες εικόνες;

Από προεπιλογή, στη διαδρομή του έργου:

```
Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### Ε: Μπορώ να επεξεργαστώ εικόνες από σενάρια Python που εκτελούνται σύμφωνα με το πρόγραμμα;

**Α:** Ναι! Χρησιμοποιήστε το Windows Task Scheduler με σενάρια Python:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("C:\\Flights\\Today")
```

Προγραμματίστε μέσω του Task Scheduler να εκτελείται καθημερινά.

***

### Ε: Το SDK υποστηρίζει async/await;

**Α:** Η τρέχουσα έκδοση είναι συγχρονισμένη. Για ασύγχρονη συμπεριφορά, χρησιμοποιήστε το `wait=False` ή εκτελέστε σε ξεχωριστό νήμα:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

## Λήψη βοήθειας

### Τεκμηρίωση

* **Αναφορά API**: Αυτή η σελίδα

### Κανάλια υποστήριξης

* **Email**: info@mapir.camera
* **Ιστότοπος**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Τιμές**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### Δείγμα κώδικα

Όλα τα παραδείγματα που αναφέρονται εδώ έχουν δοκιμαστεί και είναι έτοιμα για παραγωγή. Αντιγράψτε τα και προσαρμόστε τα για τη δική σας περίπτωση χρήσης.

***

## Άδεια

**Ιδιόκτητο λογισμικό** - Πνευματικά δικαιώματα (c) 2025 MAPIR Inc.

Το SDK απαιτεί ενεργή συνδρομή Chloros+. Απαγορεύεται η μη εξουσιοδοτημένη χρήση, διανομή ή τροποποίηση.
