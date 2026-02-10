# projet-stage
# AÏDAA - Diagrammes Architecturaux

## Diagramme d'Architecture Système

```mermaid
graph TB
    subgraph "Couche Présentation"
        WebApp["React Web App<br/>(TypeScript + TailwindCSS)"]
        MobileApp["React Native App<br/>(Expo + TypeScript)"]
    end
    
    subgraph "API Gateway"
        Gateway["Nginx/Traefik<br/>Rate Limiting + SSL"]
    end
    
    subgraph "Couche Métier - Backend Services"
        NestAPI["NestJS API (TypeScript)"]
        
        subgraph "NestJS Modules"
            AuthModule["Auth Module<br/>(JWT + Firebase)"]
            ContentModule["Content Module<br/>(Upload + Stream)"]
            UserModule["User Module<br/>(Profiles + Consent)"]
            GamifModule["Gamification Module<br/>(Badges + Progress)"]
            TeleModule["Teleconsult Module<br/>(WebRTC)"]
            NotifModule["Notification Module<br/>(Push + Email)"]
        end
        
        AIService["AI Microservice<br/>(Python FastAPI)<br/>Chatbot + Recommendations"]
    end
    
    subgraph "Couche Données"
        PostgreSQL[("PostgreSQL<br/>Users + Content<br/>+ Progress")]
        Redis[("Redis<br/>Cache + Sessions<br/>+ Pub/Sub")]
        S3["Cloud Storage<br/>(S3/R2)<br/>Videos + Assets"]
    end
    
    subgraph "Services Externes"
        Firebase["Firebase Auth<br/>(Parental Consent)"]
        GoogleSpeech["Google Cloud<br/>Speech API<br/>(ASR/TTS)"]
        SendGrid["SendGrid/SES<br/>(Email Service)"]
    end
    
    WebApp --> Gateway
    MobileApp --> Gateway
    Gateway --> NestAPI
    
    NestAPI --> AuthModule
    NestAPI --> ContentModule
    NestAPI --> UserModule
    NestAPI --> GamifModule
    NestAPI --> TeleModule
    NestAPI --> NotifModule
    
    NestAPI --> AIService
    
    AuthModule --> PostgreSQL
    ContentModule --> PostgreSQL
    ContentModule --> S3
    UserModule --> PostgreSQL
    GamifModule --> PostgreSQL
    TeleModule --> PostgreSQL
    NotifModule --> PostgreSQL
    
    NestAPI --> Redis
    
    AuthModule --> Firebase
    AIService --> GoogleSpeech
    NotifModule --> SendGrid
    
    AIService --> PostgreSQL
    AIService --> Redis
    
    style WebApp fill:#61DAFB,stroke:#333,stroke-width:2px
    style MobileApp fill:#61DAFB,stroke:#333,stroke-width:2px
    style NestAPI fill:#E0234E,stroke:#333,stroke-width:2px
    style AIService fill:#FFD43B,stroke:#333,stroke-width:2px
    style PostgreSQL fill:#336791,stroke:#333,stroke-width:2px,color:#fff
    style Redis fill:#DC382D,stroke:#333,stroke-width:2px,color:#fff
    style S3 fill:#569A31,stroke:#333,stroke-width:2px,color:#fff
```

## Diagramme de Classes Simplifié

```mermaid
classDiagram
    class User {
        <<abstract>>
        +UUID id
        +string email
        +string passwordHash
        +UserRole role
        +DateTime createdAt
        +boolean isActive
        +login() AuthToken
        +logout() void
        +updateProfile() void
    }
    
    class AutisticPerson {
        +UUID userId
        +string firstName
        +Date dateOfBirth
        +AgeGroup ageGroup
        +DifficultyLevel level
        +string[] interests
        +watchVideo() void
        +completeActivity() void
        +earnBadge() void
    }
    
    class Parent {
        +UUID userId
        +string fullName
        +string phone
        +AutisticPerson[] children
        +addChild() void
        +viewDashboard() Dashboard
        +getNotifications() Notification[]
    }
    
    class Professional {
        +UUID userId
        +string fullName
        +Specialty specialty
        +string licenseNumber
        +Patient[] patients
        +createPatientFile() PatientFile
        +addNote() void
        +startConsultation() Teleconsultation
    }
    
    class ParentalConsent {
        +UUID id
        +UUID parentId
        +UUID childId
        +DateTime grantedAt
        +DateTime expiresAt
        +boolean isActive
        +grant() void
        +revoke() void
        +isValid() boolean
    }
    
    class EducationalContent {
        +UUID id
        +JSON title
        +ContentType type
        +AgeGroup ageGroup
        +Objective objective
        +DifficultyLevel level
        +integer durationMinutes
        +string fileUrl
        +string[] tags
        +upload() void
        +validate() void
        +getRecommendations() Content[]
    }
    
    class ContentProgress {
        +UUID id
        +UUID userId
        +UUID contentId
        +integer watchedPercentage
        +DateTime completedAt
        +markComplete() void
        +updateProgress() void
    }
    
    class Badge {
        +UUID id
        +JSON name
        +JSON criteria
        +BadgeRarity rarity
        +string iconUrl
        +award() void
        +checkEligibility() boolean
    }
    
    class PatientFile {
        +UUID id
        +UUID professionalId
        +UUID patientId
        +ParentalConsent consent
        +SessionNote[] notes
        +addNote() void
        +shareVideo() void
        +getProgress() Progress
    }
    
    class Teleconsultation {
        +UUID id
        +UUID professionalId
        +UUID patientId
        +DateTime scheduledAt
        +TeleconsultStatus status
        +string roomId
        +schedule() void
        +start() WebRTCConnection
        +end() void
    }
    
    User <|-- AutisticPerson
    User <|-- Parent
    User <|-- Professional
    
    Parent "1" --> "*" AutisticPerson : manages
    Parent "1" --> "*" ParentalConsent : grants
    ParentalConsent "*" --> "1" AutisticPerson : for
    
    AutisticPerson "1" --> "*" ContentProgress : has
    ContentProgress "*" --> "1" EducationalContent : tracks
    
    AutisticPerson "1" --> "*" Badge : earns
    
    Professional "1" --> "*" PatientFile : manages
    PatientFile "*" --> "1" AutisticPerson : for
    PatientFile "1" --> "1" ParentalConsent : requires
    
    Professional "1" --> "*" Teleconsultation : schedules
    Teleconsultation "*" --> "1" AutisticPerson : with
```

## Séquence: Création Compte Enfant avec Consentement

```mermaid
sequenceDiagram
    participant Parent
    participant WebApp
    participant API
    participant Database
    participant Email
    
    Parent->>WebApp: 1. Remplir formulaire enfant
    WebApp->>API: 2. POST /auth/create-child
    API->>Database: 3. Vérifier email parent existe
    Database-->>API: 4. OK
    API->>Database: 5. Créer AutisticPerson (isActive=false)
    API->>Database: 6. Créer ParentalConsent (pending)
    API->>Email: 7. Envoyer email confirmation
    API-->>WebApp: 8. Redirection "Vérifiez votre email"
    
    Parent->>WebApp: 9. Clic lien confirmation
    WebApp->>API: 10. GET /auth/confirm-consent?token=xxx
    API->>Database: 11. Update ParentalConsent (isActive=true)
    API->>Database: 12. Update AutisticPerson (isActive=true)
    API-->>WebApp: 13. Succès - Compte activé
    WebApp-->>Parent: 14. Confirmation visuelle
```

## Séquence: Visualisation Vidéo avec Progression

```mermaid
sequenceDiagram
    participant Autiste
    participant MobileApp
    participant API
    participant Database
    participant S3
    
    Autiste->>MobileApp: 1. Parcourir bibliothèque
    MobileApp->>API: 2. GET /content?ageGroup=CHILD
    API->>Database: 3. Query EducationalContent
    Database-->>API: 4. Liste vidéos
    API-->>MobileApp: 5. Afficher vidéos
    
    Autiste->>MobileApp: 6. Cliquer vidéo X
    MobileApp->>API: 7. GET /content/:id/stream
    API->>Database: 8. Query fileUrl
    Database-->>API: 9. fileUrl
    API->>S3: 10. Generate signed URL (15 min)
    S3-->>API: 11. Signed URL
    API-->>MobileApp: 12. Stream URL
    MobileApp->>S3: 13. Demander chunks vidéo
    S3-->>MobileApp: 14. Stream vidéo
    
    Autiste->>MobileApp: 15. Regarder (50%)
    MobileApp->>API: 16. PUT /content/:id/progress {percent:50}
    API->>Database: 17. Upsert ContentProgress
    
    Autiste->>MobileApp: 18. Finir vidéo (100%)
    MobileApp->>API: 19. PUT /content/:id/progress {percent:100}
    API->>Database: 20. Mark completed
    API->>Database: 21. Check badge eligibility
    API->>Database: 22. Award badge "10 vidéos"
    API-->>MobileApp: 23. Afficher badge gagné!
    MobileApp-->>Autiste: 24. Animation badge 🎉
```

## Séquence: Téléconsultation WebRTC

```mermaid
sequenceDiagram
    participant Pro as Professionnel
    participant Patient
    participant WebApp
    participant API
    participant DB as Database
    participant WebRTC as WebRTC Server
    
    Pro->>WebApp: 1. Planifier consultation
    WebApp->>API: 2. POST /teleconsult/schedule
    API->>DB: 3. Create Teleconsultation
    API->>Patient: 4. Send notification
    
    Note over Pro,Patient: Heure du rendez-vous
    
    Pro->>WebApp: 6. Rejoindre consultation
    WebApp->>API: 7. GET /teleconsult/:id/join
    API->>DB: 8. Verify permissions
    API->>WebRTC: 9. Create room + token
    WebRTC-->>API: 10. Room token
    API-->>WebApp: 11. Room credentials
    
    Patient->>WebApp: 12. Rejoindre consultation
    WebApp->>API: 13. GET /teleconsult/:id/join
    API->>WebRTC: 14. Join room
    
    WebRTC-->>Pro: 15. P2P connection (SDP/ICE)
    WebRTC-->>Patient: 15. P2P connection (SDP/ICE)
    
    Note over Pro,Patient: 17. Visio en cours (peer-to-peer)
    
    Pro->>WebApp: 18. Prendre notes
    WebApp->>API: 19. POST /teleconsult/:id/notes
    API->>DB: 20. Save notes
    
    Pro->>WebApp: 21. Terminer session
    WebApp->>API: 22. PUT /teleconsult/:id/end
    API->>DB: 23. Update status=COMPLETED
    API->>WebRTC: 24. Close room
```

## Diagramme d'État: Cycle de vie Teleconsultation

```mermaid
stateDiagram-v2
    [*] --> SCHEDULED: Professionnel planifie
    
    SCHEDULED --> IN_PROGRESS: start()
    SCHEDULED --> CANCELLED: cancel()
    
    IN_PROGRESS --> COMPLETED: end()
    IN_PROGRESS --> CANCELLED: timeout/error
    
    COMPLETED --> [*]
    CANCELLED --> [*]
    
    note right of SCHEDULED
        - Notification envoyée
        - Room WebRTC créé
        - Timer jusqu'au rdv
    end note
    
    note right of IN_PROGRESS
        - Connexion P2P active
        - Enregistrement notes
        - Timer durée
    end note
    
    note right of COMPLETED
        - Notes sauvegardées
        - Facturation déclenchée
        - Room fermé
    end note
```

## Diagramme ER: Relations Base de Données

```mermaid
erDiagram
    USERS ||--o{ AUTISTIC_PERSONS : "has profile"
    USERS ||--o{ PARENTS : "has profile"
    USERS ||--o{ PROFESSIONALS : "has profile"
    
    PARENTS ||--o{ PARENTAL_CONSENTS : grants
    AUTISTIC_PERSONS ||--o{ PARENTAL_CONSENTS : requires
    PARENTAL_CONSENTS ||--o{ PATIENT_FILES : validates
    
    AUTISTIC_PERSONS ||--o{ CONTENT_PROGRESS : tracks
    EDUCATIONAL_CONTENTS ||--o{ CONTENT_PROGRESS : "tracked by"
    
    AUTISTIC_PERSONS ||--o{ USER_BADGES : earns
    BADGES ||--o{ USER_BADGES : "awarded as"
    
    PROFESSIONALS ||--o{ PATIENT_FILES : manages
    AUTISTIC_PERSONS ||--o{ PATIENT_FILES : "patient in"
    
    PATIENT_FILES ||--o{ SESSION_NOTES : contains
    
    PROFESSIONALS ||--o{ TELECONSULTATIONS : schedules
    AUTISTIC_PERSONS ||--o{ TELECONSULTATIONS : "participates in"
    
    USERS ||--o{ NOTIFICATIONS : receives
    USERS ||--o{ AUDIT_LOGS : "actions logged"
    
    USERS {
        uuid id PK
        string email
        string password_hash
        enum role
        boolean is_active
        timestamp created_at
    }
    
    AUTISTIC_PERSONS {
        uuid id PK
        uuid user_id FK
        string first_name
        date date_of_birth
        enum age_group
        enum level
        text_array interests
    }
    
    PARENTS {
        uuid id PK
        uuid user_id FK
        string full_name
        string phone
    }
    
    PARENTAL_CONSENTS {
        uuid id PK
        uuid parent_id FK
        uuid child_id FK
        timestamp granted_at
        timestamp expires_at
        boolean is_active
    }
    
    EDUCATIONAL_CONTENTS {
        uuid id PK
        jsonb title
        enum type
        enum age_group
        enum objective
        string file_url
        boolean is_validated
    }
    
    CONTENT_PROGRESS {
        uuid id PK
        uuid user_id FK
        uuid content_id FK
        integer watched_percentage
        timestamp completed_at
    }
    
    BADGES {
        uuid id PK
        jsonb name
        jsonb criteria
        enum rarity
        string icon_url
    }
    
    PATIENT_FILES {
        uuid id PK
        uuid professional_id FK
        uuid patient_id FK
        uuid consent_id FK
    }
    
    TELECONSULTATIONS {
        uuid id PK
        uuid professional_id FK
        uuid patient_id FK
        timestamp scheduled_at
        enum status
        string room_id
    }
```

## Flux de Données: Recommandation IA

```mermaid
flowchart TD
    Start([Utilisateur ouvre app]) --> GetProfile[Récupérer profil utilisateur]
    GetProfile --> GetHistory[Récupérer historique<br/>contenus vus]
    GetHistory --> BuildVector[Construire vecteur<br/>préférences]
    
    BuildVector --> CheckCache{Cache Redis<br/>existe?}
    CheckCache -->|Oui| ReturnCache[Retourner<br/>recommandations<br/>cachées]
    CheckCache -->|Non| CallAI[Appeler AI Service]
    
    CallAI --> CollabFilter[Collaborative Filtering<br/>Utilisateurs similaires]
    CallAI --> ContentBased[Content-Based<br/>Contenus similaires]
    
    CollabFilter --> Merge[Merger résultats]
    ContentBased --> Merge
    
    Merge --> Score[Scorer & Ranker<br/>par pertinence]
    Score --> Filter[Filtrer par:<br/>- Âge<br/>- Niveau<br/>- Déjà vus]
    Filter --> Top10[Top 10 contenus]
    Top10 --> Cache[Mettre en cache<br/>15 minutes]
    Cache --> Return[Retourner au client]
    ReturnCache --> Return
    Return --> End([Afficher dans app])
    
    style Start fill:#90EE90
    style End fill:#90EE90
    style CallAI fill:#FFD700
    style Cache fill:#FF6B6B
```

## Architecture Déploiement Cloud

```mermaid
graph TB
    subgraph "Internet"
        Users["👥 Utilisateurs<br/>(Web + Mobile)"]
    end
    
    subgraph "CDN Layer"
        CloudFront["CloudFront CDN<br/>(Assets + Vidéos)"]
    end
    
    subgraph "Load Balancer"
        ALB["Application Load Balancer<br/>(SSL/TLS + Health Checks)"]
    end
    
    subgraph "Auto-Scaling Group - Frontend"
        WebServer1["Web Server 1<br/>(Next.js Container)"]
        WebServer2["Web Server 2<br/>(Next.js Container)"]
        WebServer3["Web Server 3<br/>(Next.js Container)"]
    end
    
    subgraph "Auto-Scaling Group - Backend API"
        APIServer1["API Server 1<br/>(NestJS Container)"]
        APIServer2["API Server 2<br/>(NestJS Container)"]
    end
    
    subgraph "Microservices"
        AIService["AI Service<br/>(FastAPI Container)"]
    end
    
    subgraph "Managed Databases"
        RDS[("RDS PostgreSQL<br/>Master + Replica")]
        ElastiCache[("ElastiCache Redis<br/>Cluster (3 nodes)")]
    end
    
    subgraph "Storage"
        S3["S3 Bucket<br/>(Videos + Assets)"]
    end
    
    subgraph "External Services"
        Firebase["Firebase Auth"]
        SendGrid["SendGrid<br/>(Email)"]
        Sentry["Sentry<br/>(Error Tracking)"]
    end
    
    Users --> CloudFront
    Users --> ALB
    CloudFront --> S3
    
    ALB --> WebServer1
    ALB --> WebServer2
    ALB --> WebServer3
    
    WebServer1 --> APIServer1
    WebServer2 --> APIServer1
    WebServer3 --> APIServer2
    
    APIServer1 --> RDS
    APIServer2 --> RDS
    APIServer1 --> ElastiCache
    APIServer2 --> ElastiCache
    APIServer1 --> S3
    APIServer2 --> S3
    
    APIServer1 --> AIService
    APIServer2 --> AIService
    
    AIService --> RDS
    AIService --> ElastiCache
    
    APIServer1 --> Firebase
    APIServer1 --> SendGrid
    APIServer1 --> Sentry
    
    style Users fill:#61DAFB
    style RDS fill:#527FFF,color:#fff
    style ElastiCache fill:#DC382D,color:#fff
    style S3 fill:#569A31,color:#fff
    style CloudFront fill:#FF9900
```
