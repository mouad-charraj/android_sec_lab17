# LabSec17 - Reverse engineering Android

## Objectif

Ce lab analyse l'application OWASP MSTG UnCrackable Level 3. Le travail couvre l'analyse Java avec Jadx, la decompilation avec apktool, le patch smali, l'analyse native de `libfoo.so`, puis la reconstruction de l'APK.

## Demarche

L'APK est d'abord ouverte dans Jadx-GUI. La classe `MainActivity` montre les controles de protection: verification des librairies, detection de root, detection de debug et variable `tampered`. Le chargement de `libfoo.so` indique que la verification importante du secret se trouve dans le code natif.

L'APK est ensuite decompilee avec apktool pour obtenir les fichiers smali et les librairies natives. Le fichier `MainActivity.smali` est modifie afin de neutraliser le message de blocage lie au root ou au tampering.

La librairie `libfoo.so` est analysee dans un outil de reverse engineering. Le code natif est inspecte pour identifier la logique de verification et les controles anti-debug ou anti-Frida. Apres modification, l'APK est reconstruite, signee et reinstallee sur l'emulateur.

Un script Python sert enfin a verifier la logique retrouvee dans le code natif et a confirmer le secret retrouve.

## Preuves

### MainActivity dans Jadx

![MainActivity](<screenshots/Capture d'écran 2026-05-31 205157.png>)

La classe `MainActivity` est ouverte dans Jadx. Elle contient les controles executes au demarrage de l'application.

### Verification des protections

![Protections](<screenshots/Capture d'écran 2026-05-31 205231.png>)

Le code montre les conditions liees au root, au debug et a l'integrite de l'application.

### Chargement de libfoo

![Libfoo](<screenshots/Capture d'écran 2026-05-31 205333.png>)

La librairie native `libfoo.so` est chargee par l'application. Cette partie oriente l'analyse vers le code natif.

### Inspection Java

![Inspection Java](<screenshots/Capture d'écran 2026-05-31 205352.png>)

Le code Java permet de suivre le chemin d'execution avant l'appel a la verification native.

### Decompilation apktool

![Apktool](<screenshots/Capture d'écran 2026-05-31 205457.png>)

L'APK est decompilee avec apktool afin d'obtenir un dossier modifiable.

### Localisation du message de protection

![Message protection](<screenshots/Capture d'écran 2026-05-31 205756.png>)

Le message de blocage est localise dans le smali. Cette zone est celle qui empeche l'application de continuer dans l'environnement de lab.

### Patch smali

![Patch smali](<screenshots/Capture d'écran 2026-05-31 205930.png>)

Le patch neutralise l'appel au message de protection et permet de poursuivre l'execution de l'application.

### Analyse native

![Analyse native](<screenshots/Capture d'écran 2026-05-31 211227.png>)

La librairie native est inspectee pour retrouver les fonctions impliquees dans la verification.

### Verification du code natif

![Verification native](<screenshots/Capture d'écran 2026-05-31 211408.png>)

Les fonctions natives sont examinees pour confirmer le chemin utilise par l'application.

### Calcul du secret

![Calcul secret](<screenshots/Capture d'écran 2026-05-31 212248.png>)

Le script Python applique la logique retrouvee pendant l'analyse et donne le secret retrouve.

### Fonction native de verification

![Fonction native](<screenshots/Capture d'écran 2026-05-31 212327.png>)

Le code natif confirme la logique utilisee pour valider le secret.

## Reconstruction

Les commandes utilisees pour finaliser le patch sont:

```bash
apktool b uncrackable3 -o UnCrackable-Level3-patched.apk
apksigner sign --ks "%USERPROFILE%\.android\debug.keystore" UnCrackable-Level3-patched.apk
adb uninstall owasp.mstg.uncrackable3
adb install -r UnCrackable-Level3-patched.apk
```

## Resultat

L'application est analysee, patchee, reconstruite et reinstallee. Les controles initiaux sont contournes dans l'environnement de lab et la logique native permet de retrouver le secret.

