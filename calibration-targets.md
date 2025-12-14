---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---
# Στόχοι βαθμονόμησης

Το MAPIR προσφέρει διάφορους στόχους βαθμονόμησης για να καλύψει ένα ευρύ φάσμα εφαρμογών. Το συμπαγές T4-R50 που φαίνεται παρακάτω περιέχει 4 πάνελ που έχουν μετρηθεί για ανακλαστικότητα φωτός από 250 έως 2.500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>Οι διάχυτοι στόχοι αναφοράς T4 έχουν τις ακόλουθες καμπύλες ανάκλασης, [λήψη δεδομένων εδώ](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Ανακλαστικότητα :: 250-2500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 Ανακλαστικότητα :: 400-1000nm</p></figcaption></figure>Κοιτάζοντας το γράφημα ανάκλασης, μπορείτε να δείτε ότι οι τιμές είναι μήκος κύματος (άξονας x) έναντι ποσοστού ανάκλασης (άξονας y). Όταν καταγράφουμε μια εικόνα του στόχου βαθμονόμησης, δημιουργούμε μια σχέση μεταξύ της τιμής των εικονοστοιχείων και του ποσοστού ανάκλασης, εντός του φάσματος στο οποίο είναι ευαίσθητες οι ζώνες του αισθητήρα της κάμερας.

Αυτό σημαίνει ότι με κάθε εικόνα που καταγράφετε με τις κάμερές μας, μπορείτε να χρησιμοποιήσετε μια φωτογραφία των στόχων ανάκλασης, όπως το [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) ή το [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125) για να βαθμονομήσετε τις εικόνες ως προς την ανάκλαση. Μετά τη βαθμονόμηση, κάθε εικονοστοιχείο της εικόνας ισούται με το ποσοστό ανάκλασης.

Εάν εξάγετε τις βαθμονομημένες εικόνες σε Chloros ως τυπικό JPG ή TIFF, τότε το ποσοστό ανάκλασης υπολογίζεται διαιρώντας την τιμή του pixel με το βάθος bit της μορφής εικόνας. Έτσι, για JPG διαιρέστε με 255 και για TIFF διαιρέστε με 65.535. Μπορείτε επίσης να επιλέξετε την έξοδο μορφής PERCENT στο Chloros και, στη συνέχεια, κάθε pixel θα κυμαίνεται από μια τιμή ποσοστού 0,0 έως 1,0 (0% έως 100% ανακλαστικότητα). Λάβετε υπόψη ότι ορισμένες εφαρμογές εικόνων δεν μπορούν να δεχτούν εικόνες ποσοστού (κυμαινόμενου σημείου) και ότι έχουν μεγάλο μέγεθος από άποψη αποθήκευσης.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
