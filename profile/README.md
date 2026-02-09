## Hi there 👋
# ☕ FutureKawa - IoT & Coffee Monitoring System

> **Solution de surveillance distribuée (IoT) et centralisée pour la traçabilité du stockage de café vert.**
> *Projet réalisé dans le cadre de la MSPR - Bloc 4 (Concevoir et développer des solutions applicatives).*

---

## 🌍 Contexte du Projet

**FutureKawa** est une entreprise internationale de caféiculture (Brésil, Équateur, Colombie).
**Problème :** Pertes de qualité dues à des conditions de stockage mal maîtrisées (température/humidité) et manque de traçabilité.
**Solution :** Un système distribué complet permettant de :
1. **Collecter** en temps réel les données environnementales via des capteurs IoT (ESP32).
2. **Stocker** localement les données dans chaque pays (Brésil, Équateur, Colombie).
3. **Consolider** les stocks et les alertes au siège via une architecture centralisée.

---

## 👥 L'Équipe Projet

| Membre | Rôle Principal | Responsabilités Clés |
| :--- | :--- | :--- |
| **FATHALLAH Ayoub** | -- | -- |
| **COPPIN Mattheo** | -- | -- |
| **BOUNOUR Zied** | -- | -- |
| **GNINGUE Papa Cheikh** |-- | -- |

---

## 🏗️ Schéma d'Architecture Globale

```mermaid
flowchart LR
    %% --- Couche IoT ---
    subgraph IoT_Entrepot["Entrepôt local (pays)"]
        ESP32["ESP32 + Capteur (C++/Arduino)"]
    end

    %% --- Backend local pays ---
    subgraph LocalPays["Backend local - Pays (ex : Équateur) - Docker Compose"]
        direction TB
        Broker["Broker MQTT<br/>(Mosquitto)"]
        API_Local["API REST locale<br/>(Spring Boot, Java)"]
        DB_Local["Base SQL locale<br/>(PostgreSQL)"]
    end

    %% --- Siège FutureKawa ---
    subgraph Siege["Siège FutureKawa"]
        direction TB
        API_Central["Backend central<br/>(Spring Boot, Java)"]
        Frontend["Frontend Web<br/>(React + TypeScript)"]
    end

    %% Flux IoT
    ESP32 -- "Publie mesures<br/>(MQTT)" --> Broker
    Broker -- "Notify / Subscribe<br/>(MQTT)" --> API_Local
    API_Local -- "INSERT / SELECT<br/>(JPA / SQL)" --> DB_Local

    %% Flux pays -> siège
    API_Local -- "HTTP/REST JSON" --> API_Central
    API_Central -- "Données consolidées<br/>(JSON)" --> Frontend

    %% Styles
    classDef iot fill:#fff4c2,stroke:#d6a300,stroke-width:1px;
    classDef local fill:#e3f2fd,stroke:#1565c0,stroke-width:1px;
    classDef siege fill:#e8f5e9,stroke:#2e7d32,stroke-width:1px;

    class ESP32 iot;
    class Broker,API_Local,DB_Local local;
    class API_Central,Frontend siege;
```


### 🛠️ Stack Technologique
*   **IoT :** ESP32, Capteurs DHT22, C++ (Arduino Core)
*   **Communication :** MQTT (Mosquitto)
*   **Backend :** Java Spring Boot (API REST)
*   **Base de Données :** PostgreSQL
*   **Frontend :** React.js + TypeScript
*   **DevOps :** Docker, Docker Compose, Jenkins

---

## 💾 Base de Données

Chaque pays dispose de sa propre base de données locale **PostgreSQL** pour garantir l'autonomie en cas de coupure réseau.

### Modèle Relationnel (MCD)
La structure respecte la hiérarchie : **Configuration → Entrepôts → Lots → Mesures/Alertes**.

```mermaid
erDiagram
    CONFIGURATION ||--o{ ENTREPOTS : "definit_les_seuils_pour"
    ENTREPOTS ||--o{ LOTS : "stocke"
    LOTS ||--o{ MESURES : "contient"
    LOTS ||--o{ ALERTES : "genere"

    CONFIGURATION {
        int id PK "ID unique (toujours 1)"
        varchar nom_pays "Nom du pays (ex: Brésil)"
        varchar code_pays "Code ISO (ex: BRA)"
        decimal temp_ideale "Température idéale en °C"
        decimal temp_tolerance "Tolérance ±°C"
        decimal humidite_ideale "Humidité idéale en %"
        decimal humidite_tolerance "Tolérance ±%"
        int duree_conservation "Durée max en jours (ex: 365)"
    }

    ENTREPOTS {
        int id PK "ID unique de l'entrepôt"
        varchar nom "Nom de l'entrepôt (ex: Entrepôt Santos)"
        varchar adresse "Adresse de l'entrepôt"
        varchar responsable "Nom du responsable"
        varchar email_responsable "Email pour les alertes"
        decimal latitude "Latitude GPS (ex: -23.9608)"
        decimal longitude "Longitude GPS (ex: -46.3339)"
    }

    LOTS {
        varchar id PK "ID unique du lot (ex: LOT-BRA-2024-001)"
        int entrepot_id FK "Référence à l'entrepôt"
        varchar type_cafe "Type: ARABICA / ROBUSTA / PREMIUM / STANDARD"
        date date_stockage "Date d'entrée en stockage"
        decimal poids_kg "Poids du lot en kg"
        varchar statut "CONFORME / ALERTE / PERIME"
        timestamp date_maj "Dernière mise à jour du statut"
    }

    MESURES {
        int id PK "ID auto-incrémenté"
        varchar lot_id FK "Référence au lot"
        varchar sensor_id "ID du capteur IoT (ESP32)"
        decimal temperature "Température mesurée (°C)"
        decimal humidity "Humidité mesurée (%)"
        timestamp timestamp "Horodatage de la mesure"
        boolean conforme "TRUE si dans les seuils"
    }

    ALERTES {
        int id PK "ID de l'alerte"
        varchar lot_id FK "Lot concerné"
        varchar type "Type: TEMPERATURE / HUMIDITE / PERIME"
        timestamp date_alerte "Date de création"
        boolean validation "Alerte traitée ?"
    }
```
