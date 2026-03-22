---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Στόχοι βαθμονόμησης

Η MAPIR προσφέρει διάφορους στόχους βαθμονόμησης που καλύπτουν ένα ευρύ φάσμα εφαρμογών. Το συμπαγές μοντέλο T4-R50 που φαίνεται παρακάτω περιλαμβάνει 4 πάνελ τα οποία έχουν μετρηθεί ως προς την ανακλαστικότητα του φωτός στην περιοχή από 250 έως 2.500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>Οι στόχοι αναφοράς διάχυτης ανάκλασης T4 έχουν τις ακόλουθες καμπύλες ανάκλασης, [λήψη δεδομένων εδώ](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR Αντανάκλαση T4 :: 250-2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR Ανακλαστικότητα T4 :: 400-1000 nm</p></figcaption></figure>Οι στόχοι αναφοράς διάχυτης ακτινοβολίας T4P έχουν τις ακόλουθες καμπύλες ανακλαστικότητας, [λήψη δεδομένων εδώ](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR T4P Ανακλαστικότητα :: 250-2500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR Ανακλαστικότητα T4P :: 400-1000nm</p></figcaption></figure>Κοιτάζοντας το γράφημα ανακλαστικότητας, μπορείτε να δείτε ότι οι τιμές είναι το μήκος κύματος (άξονας x) έναντι του ποσοστού ανακλαστικότητας (άξονας y). Όταν καταγράφουμε μια εικόνα του στόχου βαθμονόμησης, δημιουργούμε στη συνέχεια μια σχέση μεταξύ της τιμής του pixel και του ποσοστού ανακλαστικότητας, εντός του φάσματος στο οποίο είναι ευαίσθητες οι ζώνες του αισθητήρα κάθε κάμερας.

Αυτό σημαίνει ότι με κάθε εικόνα που καταγράφετε με τις κάμερές μας, μπορείτε να χρησιμοποιήσετε μια φωτογραφία των στόχων ανακλαστικότητας μας, όπως το [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) ή το [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125) για να βαθμονομήσετε τις εικόνες ως προς την ανακλαστικότητα. Μόλις βαθμονομηθεί, κάθε pixel στην εικόνα αντιστοιχεί σε ποσοστό ανακλαστικότητας.

Εάν εξάγετε τις βαθμονομημένες εικόνες στο Chloros ως τυπικό JPG ή στο TIFF, τότε το ποσοστό ανακλαστικότητας υπολογίζεται διαιρώντας την τιμή του εικονοστοιχείου με το βάθος bit της μορφής εικόνας. Έτσι, για το JPG διαιρέστε με 255, και για το TIFF διαιρέστε με 65.535. Μπορείτε επίσης να επιλέξετε την έξοδο μορφής PERCENT στο Chloros, και τότε κάθε pixel θα κυμαίνεται από μια τιμή ποσοστού 0,0 έως 1,0 (0% έως 100% ανακλαστικότητα). Απλά λάβετε υπόψη ότι ορισμένες εφαρμογές εικόνων δεν μπορούν να δεχτούν τις εικόνες ποσοστού (κυμαινόμενης υποδιαστολής), και είναι μεγάλες σε μέγεθος από άποψη αποθήκευσης.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
