# Suivi de Localisation Smartphone 📍

Une solution complète de suivi de localisation en temps réel composée d'une application Android native et d'un backend PHP/MySQL structuré.

## 📁 Structure du Projet

```text
TP11/
├── LocalisationSmartphone/    # Projet Android Studio (Java)
└── localisation/              # Backend PHP
    ├── classe/                # Modèles de données
    ├── connexion/             # Configuration BDD
    ├── dao/                   # Interfaces (Data Access Object)
    ├── service/               # Logique métier et requêtes SQL
    ├── createPosition.php     # Endpoint API (Point d'entrée)
    └── db.sql                 # Script de création de la base
```

## 🛠️ Architecture Backend (PHP)

Le backend est conçu selon une architecture modulaire pour faciliter la maintenance.

### 1. Base de données (`db.sql`)
<img width="922" height="323" alt="LAB11 1" src="https://github.com/user-attachments/assets/805d8f06-14dd-461e-8e2f-ae8a5d931813" />

<img width="918" height="357" alt="LAB11 2" src="https://github.com/user-attachments/assets/fc3f17ca-8e11-4b95-90c1-a294473c4b6b" />


### 2. Connexion PDO (`connexion/Connexion.php`)
Gère la connexion sécurisée à MySQL via PDO.
```php
<?php
class Connexion {
    private $connexion;
    public function __construct() {
        $host = 'localhost';
        $dbname = 'localisation';
        $login = 'root';
        $password = '';
        try {
            $this->connexion = new PDO("mysql:host=$host;dbname=$dbname", $login, $password);
            $this->connexion->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
        } catch (Exception $e) {
            die('Erreur : ' . $e->getMessage());
        }
    }
    public function getConnexion() { return $this->connexion; }
}
?>
```

### 3. Modèle de données (`classe/Position.php`)
```php
<?php
class Position {
    private $id, $latitude, $longitude, $datePosition, $imei;
    public function __construct($id, $lat, $lon, $date, $imei) {
        $this->id = $id;
        $this->latitude = $lat;
        $this->longitude = $lon;
        $this->datePosition = $date;
        $this->imei = $imei;
    }
    // Getters et Setters...
    public function getLatitude() { return $this->latitude; }
    public function getLongitude() { return $this->longitude; }
    public function getDatePosition() { return $this->datePosition; }
    public function getImei() { return $this->imei; }
}
?>
```

### 4. Service et DAO (`service/PositionService.php`)
Contient la logique d'insertion en base de données.
```php
<?php
include_once 'dao/IDao.php';
include_once 'classe/Position.php';
include_once 'connexion/Connexion.php';

class PositionService implements IDao {
    private $connexion;
    public function __construct() { $this->connexion = new Connexion(); }

    public function create($position) {
        $sql = "INSERT INTO position(latitude, longitude, date_position, imei) 
                VALUES(:latitude, :longitude, :date_position, :imei)";
        $stmt = $this->connexion->getConnexion()->prepare($sql);
        $stmt->execute([
            ':latitude' => $position->getLatitude(),
            ':longitude' => $position->getLongitude(),
            ':date_position' => $position->getDatePosition(),
            ':imei' => $position->getImei()
        ]);
    }
    // Autres méthodes de l'interface (update, delete, etc.)
}
?>
```

### 5. API Endpoint (`createPosition.php`)
Le fichier appelé par l'application Android via une requête POST.
```php
<?php
if ($_SERVER["REQUEST_METHOD"] == "POST") {
    include_once 'service/PositionService.php';
    $service = new PositionService();
    
    $latitude = $_POST['latitude'];
    $longitude = $_POST['longitude'];
    $datePosition = $_POST['date_position'];
    $imei = $_POST['imei'];

    $position = new Position(null, $latitude, $longitude, $datePosition, $imei);
    $service->create($position);
    echo "Position enregistrée avec succès";
}
?>
```

## 📱 Partie Android

L'application utilise **Volley** pour envoyer les coordonnées.

### Configuration de l'URL
Dans `MainActivity.java`, adaptez l'URL selon votre environnement :
- **Émulateur** : `http://10.0.2.2/localisation/createPosition.php`
- **Réel** : `http://192.168.x.x/localisation/createPosition.php`

### Code d'envoi (Extrait Volley)
```java
StringRequest request = new StringRequest(Request.Method.POST, insertUrl,
    response -> Toast.makeText(context, response, Toast.LENGTH_SHORT).show(),
    error -> Log.e("Error", error.getMessage())
) {
    @Override
    protected Map<String, String> getParams() {
        Map<String, String> params = new HashMap<>();
        params.put("latitude", String.valueOf(lat));
        params.put("longitude", String.valueOf(lon));
        params.put("date_position", "2024-05-08 12:00:00");
        params.put("imei", deviceImei);
        return params;
    }
};
```

## 📋 Prérequis
- **Android Studio** (pour le mobile)
- **Serveur local** (WAMP, XAMPP ou Laragon)
- **Dépendance Volley** dans `build.gradle` :
  `implementation 'com.android.volley:volley:1.2.1'`



## 📸 Aperçu

<img width="321" height="440" alt="LAB11 3" src="https://github.com/user-attachments/assets/e65e7df1-f4df-4ce0-b334-df58ede886cb" />


<img width="357" height="321" alt="LAB11 4" src="https://github.com/user-attachments/assets/978e068f-c87d-4077-9290-cf34a8b71d2e" />


## 🛡️ Permissions requises
L'application nécessite les permissions suivantes :
- `ACCESS_FINE_LOCATION` : Pour une précision GPS optimale.
- `INTERNET` : Pour envoyer les données au serveur.
- `READ_PHONE_STATE` : Pour récupérer l'IMEI de l'appareil.


