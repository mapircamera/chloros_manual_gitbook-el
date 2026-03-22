# Οδηγός NVIDIA Jetson

Το Chloros στο NVIDIA Jetson επιτρέπει την επεξεργασία πολυφασματικών εικόνων στην «άκρη» — στο πεδίο, σε UAV και σε απομακρυσμένες εγκαταστάσεις. Το Chloros αναγνωρίζει αυτόματα το μοντέλο Jetson που διαθέτετε και βελτιστοποιεί τη στρατηγική επεξεργασίας του για το υλικό σας.

***

## Υποστηριζόμενα μοντέλα Jetson

| Μοντέλο                | RAM            | Στρατηγική επεξεργασίας                                   | Συνιστώμενη χρήση                                          |
| -------------------- | -------------- | ----------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64 GB κοινόχρηστη | `GPU_PARALLEL` (4 workers)                            | Μέγιστη απόδοση, μεγάλα σύνολα δεδομένων                      |
| **Jetson Orin NX**   | 8-16 GB κοινόχρηστη  | `GPU_PARALLEL` (3 εργαζόμενοι, 16 GB) / `GPU_SINGLE` (8 GB) | Κύρια σύσταση για ανάπτυξη σε αεροσκάφη και στο πεδίο |
| **Jetson Orin Nano** | 8 GB κοινόχρηστη     | `GPU_SINGLE` (1 εργαζόμενος)                               | Υπολογιστική ισχύς άκρου εισόδου                                 |
| **Jetson Nano**      | 4-8 GB κοινόχρηστη   | `GPU_SINGLE` (1 εργαζόμενος)                               | Εισαγωγικού επιπέδου, με περιορισμούς μνήμης                          |

{% hint style="info" %}
**Τα παλαιότερα μοντέλα Jetson** (TX2, TX1, Xavier NX) ενδέχεται να μην υποστηρίζονται. Η απόδοση θα ποικίλλει ανάλογα με τη διαθέσιμη μνήμη GPU και τις δυνατότητες CUDA.
{% endhint %}

***

## Απαιτήσεις

* **JetPack 6.x** (συνιστάται η πιο πρόσφατη έκδοση)
* **NVIDIA CUDA** (περιλαμβάνεται στο JetPack)
* **Άδεια χρήσης Chloros+** (απαιτείται για πρόσβαση σε CLI/SDK)

## Εγκατάσταση

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros-arm64-jp6.deb

# Verify installation
chloros-cli --version

# Install Python SDK (optional)
pip install chloros-sdk

# Run system diagnostics
chloros-cli selftest
```

Για γενικές λεπτομέρειες εγκατάστασης του Linux, ανατρέξτε στην [Εγκατάσταση του Linux](linux-installation.md).

***

## Δυναμική προσαρμογή υπολογισμών στο Jetson

Το Chloros ανιχνεύει αυτόματα το μοντέλο Jetson σας και επιλέγει τη βέλτιστη στρατηγική επεξεργασίας. **Δεν απαιτείται χειροκίνητη ρύθμιση.**

### Πώς λειτουργεί

Κατά την εκκίνηση, το Chloros δημιουργεί προφίλ του συστήματός σας:

1. **Ανιχνεύει το μοντέλο Jetson** μέσω του `/proc/device-tree/model`
2. **Διαβάζει τη διαθέσιμη GPU/κοινόχρηστη μνήμη**

3.**Επιλέγει μια στρατηγική επεξεργασίας** (`GPU_PARALLEL`, `GPU_SINGLE` ή `CPU_PARALLEL`)
4. **Ορίζει αυτόματα τον αριθμό των εργαζομένων, τον τύπο του αγωγού και την κατανομή μνήμης**

### Συμπεριφορά ανά μοντέλο

| Μοντέλο Jetson                | Στρατηγική       | Εργαζόμενοι | Αγωγός                       | Ταυτόχρονη εκτέλεση |
| --------------------------- | -------------- | ------- | ------------------------------ | ----------- |
| **Jetson Nano 8GB**         | `GPU_SINGLE`   | 1       | `tiled_gpu` (αποδοτική χρήση μνήμης) | Σειριακή  |
| **Jetson Orin Nano 8GB**    | `GPU_SINGLE`   | 1       | `tiled_gpu`                    | Σειριοποιημένη  |
| **Jetson Orin NX 8GB**      | `GPU_SINGLE`   | 2       | `tiled_gpu`                    | Σειριοποιημένο  |
| **Jetson Orin NX 16GB**     | `GPU_PARALLEL` | 3       | `fused_gpu` (πλήρης διαδρομή GPU)    | Ταυτόχρονη  |
| **Jetson AGX Orin 32-64GB** | `GPU_PARALLEL` | 4       | `fused_gpu`                    | Ταυτόχρονη  |

{% hint style="success" %}
Το **Jetson Orin NX 16GB** είναι η ιδανική επιλογή για ανάπτυξη σε περιφερειακά συστήματα — ακολουθεί τη στρατηγική `GPU_PARALLEL` με 3 ταυτόχρονους εργαζόμενους, παρέχοντας πραγματική παράλληλη επεξεργασία GPU σε συμπαγή μορφή.
{% endhint %}

Η βασική διαφορά μεταξύ των πλατφορμών είναι η **μνήμη**. Ένα Jetson Nano με 8 GB κοινόχρηστης μνήμης πρέπει να επεξεργάζεται εικόνες μία κάθε φορά χρησιμοποιώντας μια προσέγγιση με πλακίδια που εξοικονομεί μνήμη, ενώ ένα Orin NX με 16 GB μπορεί να επεξεργάζεται 3 εικόνες ταυτόχρονα μέσω της GPU χρησιμοποιώντας τον συνδυασμένο αγωγό υψηλότερης απόδοσης.

Για την πλήρη αναφορά προσαρμογής υπολογιστικής ισχύος, ανατρέξτε στο [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md).

***

## Διαχείριση θερμοκρασίας

Οι συσκευές Jetson έχουν περιορισμένο θερμικό περιθώριο, ειδικά σε κλειστές ή αερομεταφερόμενες εφαρμογές. Το Chloros περιλαμβάνει αυτόματη παρακολούθηση θερμοκρασίας και περιορισμό:

| Θερμοκρασία         | Ενέργεια                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70°C**          | Κανονική λειτουργία — πλήρης ταχύτητα επεξεργασίας          |
| **70°C** (Προειδοποίηση)  | Αυτόματη μείωση μεγέθους παρτίδας                   |
| **80°C** (Κρίσιμη κατάσταση) | Εντατική ρύθμιση — χαμηλότερη ταυτόχρονη επεξεργασία         |
| **90°C** (Τερματισμός λειτουργίας) | Πλήρης διακοπή επεξεργασίας GPU — απαιτείται ψύξη |

{% hint style="warning" %}
**Εξασφαλίστε επαρκή αερισμό και απαγωγή θερμότητας** για συνεχή επεξεργασία, ειδικά σε κλειστά περιβλήματα πεδίου ή αερομεταφερόμενα συστήματα. Η θερμική ρύθμιση θα μειώσει την απόδοση επεξεργασίας για την προστασία του υλικού.
{% endhint %}

***

## Διαχείριση μνήμης

Οι συσκευές Jetson χρησιμοποιούν **ενιαία μνήμη** — η GPU και η CPU μοιράζονται την ίδια φυσική μνήμη RAM. Αυτό σημαίνει ότι η αναφερόμενη VRAM (π.χ., 15,3 GB στο Orin NX 16 GB) δεν είναι αποκλειστική μνήμη GPU· μοιράζεται με το λειτουργικό σύστημα και άλλες διεργασίες.

### Συστάσεις για swap

Για μεγάλα σύνολα δεδομένων ή επεξεργασία Texture Aware debayer, το Chloros ενδέχεται να συστήσει τη δημιουργία χώρου swap:

```bash
# Check current memory and swap
free -h

# Create a swap file (example: 8GB)
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**Εκτιμήσεις μνήμης ανά εικόνα:**

* Τυπική αποσύνθεση: ~10 MB ανά εικόνα
* Αποσύνθεση με αναγνώριση υφής: ~15 MB ανά εικόνα

Το Chloros υπολογίζει αυτόματα τη μνήμη που απαιτείται με βάση το μέγεθος του συνόλου δεδομένων σας και σας προειδοποιεί εάν συνιστάται η χρήση swap.

### Εναλλακτική λύση OOM (Out of Memory)

Εάν εντοπιστεί κατάσταση έλλειψης μνήμης κατά τη διάρκεια της επεξεργασίας:

1. Το Chloros μειώνει αυτόματα τον αριθμό των εργαζομένων GPU
2. Επιστρέφει από το `fused_gpu` στο pipeline `tiled_gpu` (πιο αποδοτικό ως προς τη μνήμη)
3. Συνεχίζει την επεξεργασία με μειωμένη απόδοση αντί να διακοπεί

***

## Εγκατάσταση στο πεδίο

### Θέματα ισχύος

| Μοντέλο Jetson     | Τυπική κατανάλωση ισχύος | Σημειώσεις                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Nano      | 5-10W              | USB-C ή υποδοχή τύπου barrel    |
| Jetson Orin Nano | 7-15W              | Υποδοχή DC τύπου barrel          |
| Jetson Orin NX   | 10-25W             | Υποδοχή DC τύπου barrel          |
| Jetson AGX Orin  | 15-60W             | USB-C PD ή υποδοχή τύπου barrel |

Προγραμματίστε τον προϋπολογισμό ισχύος σας για συνεχή επεξεργασία — η μέγιστη κατανάλωση ισχύος συμβαίνει κατά τη διάρκεια του Thread 3 (Επεξεργασία) που απαιτεί εντατική χρήση της GPU.

### Συστάσεις αποθήκευσης

* **NVMe SSD** συνιστάται έντονα για εφαρμογές arm64
* Οι κάρτες SD είναι πολύ αργές για επεξεργασία — χρησιμοποιήστε τις μόνο ως μέσο εκκίνησης
* Προγραμματίστε χώρο 2-3 φορές μεγαλύτερο από το μέγεθος των ακατέργαστων δεδομένων εικόνας για τα επεξεργασμένα αποτελέσματα

### Λειτουργία χωρίς οθόνη μέσω SSH

Το Chloros CLI είναι ιδανικό για εγκαταστάσεις Jetson χωρίς οθόνη:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format tiff-32

# Monitor export progress
chloros-cli export-status
```

### Αυτοματοποιημένη επεξεργασία με systemd

Δημιουργήστε μια υπηρεσία systemd για αυτοματοποιημένη επεξεργασία:

```ini
# /etc/systemd/system/chloros-process.service
[Unit]
Description=Chloros Automated Processing
After=network.target

[Service]
Type=oneshot
User=chloros
ExecStart=/usr/bin/chloros-cli process /data/incoming --output /data/processed
StandardOutput=append:/var/log/chloros-process.log
StandardError=append:/var/log/chloros-process.log

[Install]
WantedBy=multi-user.target
```

Συνδυάστε με ένα χρονόμετρο systemd για προγραμματισμένη επεξεργασία:

```ini
# /etc/systemd/system/chloros-process.timer
[Unit]
Description=Run Chloros Processing Every Hour

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl enable chloros-process.timer
sudo systemctl start chloros-process.timer
```

***

## Παραδείγματα ροών εργασίας

### Βασική επεξεργασία Jetson

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI
```

### Python SDK στο Jetson

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (32-bit, Percent)",
        reflectance_calibration=True
    )
    chloros.process(mode="parallel")

print("Processing complete!")
```

### Μαζική επεξεργασία πολλαπλών πτήσεων

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format tiff-32 \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Συστήματα Jetson που συνιστώνται για χρήση στο πεδίο

Για εφαρμογές στο πεδίο και στον αέρα, εξετάστε αυτές τις επιλογές πλακέτας φορέα Jetson Orin NX 16GB:

* **Αερομεταφερόμενα/δρόνοι**: Συστήματα με βαθμολογία αντοχής σε κραδασμούς (MIL-STD), ελαφριά (κάτω από 300g), παθητική ψύξη
* **Ανθεκτικό πεδίο**: Αδιάβροχα περιβλήματα IP67/IP69K με συνδεσιμότητα κάμερας PoE GigE
* **Ελάχιστο/οικονομικό**: Κιτ προγραμματιστή με πρόσθετα περιβλήματα

Επικοινωνήστε με την [Υποστήριξη MAPIR](https://www.mapir.camera/community/contact) για συγκεκριμένες συστάσεις υλικού για το σενάριο ανάπτυξης σας.

***

## Επόμενα βήματα

* [Linux Εγκατάσταση](linux-installation.md) — Γενικές λεπτομέρειες εγκατάστασης του Linux
* [Δυναμική προσαρμογή υπολογιστικής ισχύος](../processing-architecture/dynamic-compute-adaptation.md) — Πλήρης αναφορά στρατηγικής υπολογιστικής ισχύος
* [Αγωγός επεξεργασίας](../processing-architecture/processing-pipeline.md) — Κατανόηση του αγωγού 4 νημάτων
* [CLI : Γραμμή εντολών](../CLI.md) — Πλήρης αναφορά CLI
* [API : Python SDK](../api-python-sdk.md) — Πλήρης αναφορά SDK
