# Projet-Linky-IOT
# Projet Linky IIOT

## Description

Ce projet est une application console en **C#** qui simule la lecture de compteurs électriques Linky via MQTT.  
Elle permet de recevoir les données de capteurs (puissance, température, état TOR), de les traiter et de les afficher dans la console, puis de les sauvegarder périodiquement en fichiers CSV.  

Cette version remplace une ancienne version Windows Forms perdue.

---

## Fonctionnement du code

L’application est organisée autour de quelques concepts clés :

### 1. Structure des données

- La classe `CompteurData` stocke toutes les informations d’un compteur :
  - `Nom` : identifiant du compteur.
  - `PuissanceList` : liste des valeurs de puissance reçues.
  - `Temperature` : température actuelle.
  - `EtatTOR` : état TOR du compteur.
  - `PuissanceMin` et `PuissanceMax` : valeurs min et max de puissance.

---

### 2. Initialisation (`Init`)

- Lit le fichier de configuration (`TestParam.txt`) qui contient :
  1. L’adresse du broker MQTT.
  2. Le ou les topics des compteurs séparés par des `;`.
  3. Le temps entre deux sauvegardes en secondes.
  4. Le nombre de valeurs pour la moyenne glissante.
- Vérifie que toutes les lignes sont présentes et valides.
- Sépare les topics pour gérer plusieurs compteurs.

---

### 3. Connexion MQTT (`ConnexionAbonnementMqtt`)

- Crée un client MQTT et se connecte au broker.
- S’abonne au topic `GeiiNancy/Compteurs/#` pour recevoir tous les compteurs.
- Affiche dans la console :
  - Nom du broker.
  - Liste des compteurs valides.
  - Intervalle de sauvegarde.
  - Nombre de valeurs pour la moyenne glissante.

---

### 4. Filtrage des compteurs (`Filtrage` et `EstCompteurValide`)

- Vérifie que les messages reçus correspondent à des compteurs valides configurés dans le fichier.
- Ignore les messages provenant de compteurs non définis.

---

### 5. Réception des données MQTT (`Client_MqttMsgPublishReceived`)

- Décode le message reçu en texte ASCII.
- Sépare le topic pour identifier le compteur.
- Vérifie et parse les valeurs de puissance, température et état TOR.
- Met à jour la liste de valeurs pour le compteur.
- Calcule le minimum, le maximum et prépare la moyenne glissante.

---

### 6. Calcul de la moyenne glissante (`CalculerMoyenneGlissante`)

- Lisse les données pour obtenir une valeur moyenne sur les `NbMoy` dernières mesures.
- Affiche un avertissement si le nombre de valeurs reçu est insuffisant pour le calcul.

---

### 7. Affichage (`Affichage`)

- Affiche dans la console un tableau avec :
  - Nom du compteur
  - Moyenne de puissance
  - Min / Max de puissance
- Change la couleur du texte pour rendre le tableau plus lisible.
- Affiche un message si une erreur de sauvegarde survient.


---

### 8. Sauvegarde des données (`SauvegardePeriodique`)

- Crée un dossier `Sauvegardes` si nécessaire.
- Enregistre chaque compteur dans un fichier CSV : `NomCompteur.csv`.
- Format CSV : `DateHeure;Temperature;Puissance;ETor`.
- Réinitialise le flag d’erreur si la sauvegarde est réussie, sinon affiche un message d’erreur.

---

### 9. Boucle principale (`Main`)

1. Demande à l’utilisateur le chemin du fichier de configuration et le chemin de sauvegarde.
2. Charge la configuration via `Init()`.
3. Connecte et s’abonne au broker MQTT via `ConnexionAbonnementMqtt()`.
4. Boucle infinie :
   - Si de nouvelles valeurs sont reçues, elles sont sauvegardées périodiquement.
   - Sinon, le programme attend 1 seconde.


