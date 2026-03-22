# Εγκατάσταση του Linux

Το Chloros διατίθεται για το Linux ως πακέτα `.deb`, τα οποία εγκαθιστούν το CLI και το backend. Το Python SDK εγκαθίσταται ξεχωριστά μέσω του pip.

***

## Linux amd64 (x86_64)

### Απαιτήσεις συστήματος

| Απαίτηση | Ελάχιστη | Συνιστώμενη |
| --- | --- | --- |
| **Διανομή** | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+ |
| **Επεξεργαστής** | x86_64 (Intel/AMD) | Intel Core i7 ή καλύτερος |
| **Μνήμη (RAM)** | 8 GB | 16 GB ή περισσότερο |
| **Κάρτα γραφικών** | Καμία (επεξεργασία CPU) | GPU NVIDIA με 4 GB+ VRAM |
| **Αποθήκευση** | 2 GB ελεύθερου χώρου | SSD με 10 GB+ ελεύθερου χώρου |
| **Python** | Python 3.7+ (για SDK) | Python 3.10+ |

### Εγκατάσταση

Κατεβάστε το πακέτο `.deb` και εγκαταστήστε το:

```bash
sudo dpkg -i chloros-amd64.deb
```

Επαληθεύστε την εγκατάσταση:

```bash
chloros-cli --version
```

***

## Linux arm64 (NVIDIA Jetson)

### Απαιτήσεις συστήματος

| Απαίτηση | Ελάχιστη | Συνιστώμενη |
| --- | --- | --- |
| **Πλατφόρμα** | NVIDIA Jetson με JetPack 6 | Jetson Orin NX 16GB ή AGX Orin |
| **JetPack** | JetPack 6.x | Τελευταία έκδοση JetPack 6 |
| **Μνήμη (RAM)** | 8 GB (κοινή GPU/CPU) | 16 GB+ κοινή |
| **Αποθήκευση** | 2 GB ελεύθερος χώρος | SSD NVMe με 10 GB+ ελεύθερο |
| **Python** | Python 3.7+ (για SDK) | Python 3.10+ |

### Εγκατάσταση

Κατεβάστε το πακέτο JetPack 6 `.deb` και εγκαταστήστε το:

```bash
sudo dpkg -i chloros-arm64-jp6.deb
```

Επαληθεύστε την εγκατάσταση:

```bash
chloros-cli --version
```

Για λεπτομερείς πληροφορίες σχετικά με τη ρύθμιση του Jetson, συμπεριλαμβανομένης της διαχείρισης θερμοκρασίας και της εγκατάστασης στο πεδίο, ανατρέξτε στον [Οδηγό NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Python SDK Εγκατάσταση (Όλα τα Linux)

Το Python SDK εγκαθίσταται ξεχωριστά μέσω pip και λειτουργεί τόσο σε amd64 όσο και σε arm64:

```bash
pip install chloros-sdk
```

Για να συμπεριλάβετε προαιρετική υποστήριξη ροής προόδου:

```bash
pip install chloros-sdk[progress]
```

Επαληθεύστε το SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Το πακέτο `.deb` εγκαθιστά το Chloros CLI και το backend. Το Python SDK είναι ένα ξεχωριστό πακέτο pip που επικοινωνεί με το backend μέσω ενός τοπικού HTTP API.
{% endhint %}

***

## Κατάλογοι διαμόρφωσης

Το Chloros στο Linux ακολουθεί την [Προδιαγραφή βασικού καταλόγου XDG](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html):

| Σκοπός | Διαδρομή Linux | Ισοδύναμο Windows |
| --- | --- | --- |
| **Διαμόρφωση** | `~/.config/chloros/` | `%APPDATA%\Chloros\` |
| **Δεδομένα / Έργα** | `~/.local/share/chloros/` | `%LOCALAPPDATA%\Chloros\` |
| **Cache / Διαπιστευτήρια** | `~/.cache/chloros/` | `%APPDATA%\Chloros\cache\` |

## Θέσεις εκτελέσιμων αρχείων του backend

Το πακέτο `.deb` εγκαθιστά το backend σε μια τυπική θέση. Τα πακέτα CLI και SDK ανιχνεύουν αυτόματα τη διαδρομή του backend:

| Μέθοδος εγκατάστασης | Διαδρομή backend |
| --- | --- |
| Πακέτο `.deb` | `/usr/lib/chloros/chloros-backend` |
| Χειροκίνητη / προσαρμοσμένη | `/opt/mapir/chloros/backend/chloros-backend` |

Μπορείτε να αντικαταστήσετε τη διαδρομή του backend με την επιλογή `--backend-exe` CLI ή την παράμετρο κατασκευαστή `backend_exe` SDK.

***

## Αρχική ρύθμιση

### 1. Ενεργοποιήστε την άδειά σας

Απαιτείται άδεια Chloros+ για πρόσβαση στα CLI και SDK:

```bash
chloros-cli login your@email.com 'your-password'
```

### 2. Ελέγξτε την κατάσταση της άδειας χρήσης σας

```bash
chloros-cli status
```

### 3. Επεξεργαστείτε το πρώτο σας σύνολο δεδομένων

```bash
chloros-cli process ~/datasets/flight001
```

### 4. Εκτελέστε διαγνωστικά συστήματος

Βεβαιωθείτε ότι το σύστημά σας έχει ρυθμιστεί σωστά:

```bash
chloros-cli selftest
```

Αυτό εκτελεί 7 διαγνωστικούς ελέγχους, συμπεριλαμβανομένων της έκδοσης, της εκκίνησης του backend, της συνδεσιμότητας API και της διαθεσιμότητας CUDA/GPU.

***

## Παραδείγματα σεναρίων Bash

### Επεξεργασία πολλαπλών συνόλων δεδομένων

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    chloros-cli process "$dataset" --format tiff-32
    echo "Done: $(basename "$dataset")"
done
```

### Επεξεργασία με προσαρμοσμένες ρυθμίσεις

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

### Αυτοματοποιημένη επεξεργασία με Cron

Προσθέστε στο crontab σας (`crontab -e`) για να επεξεργάζεστε αυτόματα νέα σύνολα δεδομένων:

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

### CLI Δεν βρέθηκε μετά την εγκατάσταση

Εάν το `chloros-cli` δεν βρέθηκε μετά την εγκατάσταση του πακέτου `.deb`:

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# If not in PATH, check the installation
dpkg -L chloros-amd64  # or chloros-arm64-jp6

# Reload your shell
source ~/.bashrc
```

### Άρνηση δικαιώματος

```bash
# Ensure the binary is executable
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
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

### Δεν εντοπίστηκε CUDA

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

### Λείπουν κοινόχρηστες βιβλιοθήκες

```bash
# Install common dependencies
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

***

## Ενημέρωση του Chloros στο Linux

Χρησιμοποιήστε την ενσωματωμένη εντολή ενημέρωσης για να ελέγξετε και να εγκαταστήσετε ενημερώσεις:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

***

## Επόμενα βήματα

* [Οδηγός NVIDIA Jetson](nvidia-jetson-guide.md) — Βελτιστοποίηση και ανάπτυξη ειδικά για Jetson
* [CLI : Γραμμή εντολών](../CLI.md) — Πλήρης αναφορά εντολών CLI
* [API : Python SDK](../api-python-sdk.md) — Πλήρης αναφορά SDK
* [Δυναμική προσαρμογή υπολογισμών](../processing-architecture/dynamic-compute-adaptation.md) — Πώς το Chloros προσαρμόζεται στο υλικό σας
