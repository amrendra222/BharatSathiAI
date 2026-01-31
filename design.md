# BharatSathi AI - System Design Document

## System Architecture Overview

BharatSathi AI follows a microservices architecture pattern with cloud-native design principles, ensuring scalability, maintainability, and high availability.

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        User Interface Layer                      │
├─────────────────┬─────────────────┬─────────────────────────────┤
│   Web App       │   Mobile App    │      Voice Interface        │
│   (React PWA)   │  (React Native) │    (Alexa/Google/IVR)      │
└─────────────────┴─────────────────┴─────────────────────────────┘
                              │
                    ┌─────────────────┐
                    │   API Gateway   │
                    │   (Kong/AWS)    │
                    └─────────────────┘
                              │
┌─────────────────────────────────────────────────────────────────┐
│                    Microservices Layer                          │
├─────────────┬─────────────┬─────────────┬─────────────────────┤
│   User      │    AI       │  Government │    Notification     │
│  Service    │  Service    │   Service   │     Service         │
└─────────────┴─────────────┴─────────────┴─────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────────┐
│                      Data Layer                                 │
├─────────────┬─────────────┬─────────────┬─────────────────────┤
│  PostgreSQL │   MongoDB   │    Redis    │    Elasticsearch    │
│ (User Data) │(AI Models)  │  (Cache)    │   (Search Index)    │
└─────────────┴─────────────┴─────────────┴─────────────────────┘
```

## Component Design

### Frontend Components

#### 1. Web Application (React PWA)

**Technology Stack:**
- React 18 with TypeScript
- Material-UI for component library
- Redux Toolkit for state management
- React Query for API state management
- Workbox for PWA capabilities

**Key Components:**
```
src/
├── components/
│   ├── Chat/
│   │   ├── ChatInterface.tsx
│   │   ├── MessageBubble.tsx
│   │   └── VoiceInput.tsx
│   ├── Services/
│   │   ├── ServiceCard.tsx
│   │   ├── ServiceSearch.tsx
│   │   └── ServiceTracker.tsx
│   └── Common/
│       ├── LanguageSelector.tsx
│       ├── Header.tsx
│       └── Navigation.tsx
├── pages/
│   ├── Dashboard.tsx
│   ├── Services.tsx
│   ├── Profile.tsx
│   └── Help.tsx
├── hooks/
│   ├── useChat.ts
│   ├── useVoice.ts
│   └── useLocation.ts
└── utils/
    ├── api.ts
    ├── i18n.ts
    └── storage.ts
```

#### 2. Mobile Application (React Native)

**Key Features:**
- Cross-platform iOS/Android support
- Offline-first architecture
- Voice recognition integration
- Camera for document scanning
- Push notifications

**Architecture:**
```
├── src/
│   ├── screens/
│   ├── components/
│   ├── navigation/
│   ├── services/
│   └── utils/
├── android/
├── ios/
└── assets/
```

### Backend Components

#### 1. API Gateway

**Technology:** Kong API Gateway / AWS API Gateway

**Responsibilities:**
- Request routing and load balancing
- Authentication and authorization
- Rate limiting and throttling
- API versioning
- Request/response transformation

**Configuration:**
```yaml
services:
  - name: user-service
    url: http://user-service:3001
    routes:
      - name: user-routes
        paths: ["/api/v1/users"]
  - name: ai-service
    url: http://ai-service:3002
    routes:
      - name: ai-routes
        paths: ["/api/v1/chat", "/api/v1/voice"]
```

#### 2. User Service

**Technology:** Node.js with Express/Fastify

**Responsibilities:**
- User authentication and authorization
- Profile management
- Preference settings
- Service history tracking

**API Endpoints:**
```typescript
// User Management
POST   /api/v1/users/register
POST   /api/v1/users/login
GET    /api/v1/users/profile
PUT    /api/v1/users/profile
DELETE /api/v1/users/account

// Preferences
GET    /api/v1/users/preferences
PUT    /api/v1/users/preferences
GET    /api/v1/users/history
```

#### 3. AI Service

**Technology:** Python with FastAPI

**Responsibilities:**
- Natural language processing
- Intent recognition and entity extraction
- Response generation
- Multilingual support
- Voice processing

**Service Architecture:**
```python
# AI Service Structure
ai_service/
├── app/
│   ├── models/
│   │   ├── nlp_model.py
│   │   ├── intent_classifier.py
│   │   └── entity_extractor.py
│   ├── services/
│   │   ├── chat_service.py
│   │   ├── voice_service.py
│   │   └── translation_service.py
│   ├── api/
│   │   ├── chat_routes.py
│   │   └── voice_routes.py
│   └── utils/
│       ├── preprocessing.py
│       └── postprocessing.py
```

#### 4. Government Service

**Technology:** Node.js with Express

**Responsibilities:**
- Government API integrations
- Service information management
- Document processing
- Status tracking

**Integration Points:**
```typescript
// Government APIs
interface GovernmentAPIs {
  aadhaar: AadhaarAPI;
  pan: PANServiceAPI;
  passport: PassportAPI;
  driving_license: DLServiceAPI;
  voter_id: VoterServiceAPI;
  ration_card: RationCardAPI;
}
```

#### 5. Notification Service

**Technology:** Node.js with Bull Queue

**Responsibilities:**
- Push notifications
- SMS notifications
- Email notifications
- Reminder scheduling

## AI Engine Design

### Natural Language Processing Pipeline

```
User Input → Preprocessing → Language Detection → Translation → 
Intent Classification → Entity Extraction → Context Management → 
Response Generation → Translation → Postprocessing → User Output
```

### AI Components

#### 1. Language Models

**Primary Model:** Custom fine-tuned multilingual BERT
- Base: google/muril-base-cased
- Fine-tuned on Indian government service data
- Support for 22 Indian languages

**Specialized Models:**
```python
models = {
    "intent_classifier": "bharatsathi/intent-classifier-v1",
    "entity_extractor": "bharatsathi/entity-extractor-v1", 
    "response_generator": "bharatsathi/response-generator-v1",
    "document_classifier": "bharatsathi/document-classifier-v1"
}
```

#### 2. Knowledge Graph

**Structure:**
```
Government Services
├── Central Government
│   ├── Ministries
│   ├── Departments
│   └── Schemes
├── State Government
│   ├── Departments
│   ├── Districts
│   └── Local Services
└── Documents
    ├── Required Documents
    ├── Procedures
    └── Fees
```

**Implementation:** Neo4j Graph Database

#### 3. Voice Processing

**Components:**
- Speech-to-Text: Google Cloud Speech API / Azure Speech
- Text-to-Speech: Amazon Polly with Indian voices
- Voice Activity Detection: WebRTC VAD
- Noise Reduction: Spectral Subtraction

## Data Flow Architecture

### Chat Interaction Flow

```
User Message → API Gateway → User Service (Auth) → AI Service → 
NLP Pipeline → Knowledge Graph Query → Government Service API → 
Response Generation → Translation → User Interface
```

### Voice Interaction Flow

```
Voice Input → STT Service → Text Processing → AI Service → 
Response Generation → TTS Service → Audio Output
```

### Document Processing Flow

```
Document Upload → OCR Service → Document Classification → 
Validation Service → Government API → Status Update → 
Notification Service → User Notification
```

## API Design

### RESTful API Structure

#### Chat API
```typescript
// Chat Endpoints
POST /api/v1/chat/message
{
  "message": "string",
  "language": "string",
  "session_id": "string",
  "user_id": "string"
}

Response:
{
  "response": "string",
  "intent": "string",
  "entities": [],
  "suggestions": [],
  "session_id": "string"
}
```

#### Voice API
```typescript
// Voice Endpoints
POST /api/v1/voice/process
Content-Type: multipart/form-data
{
  "audio": "file",
  "language": "string",
  "user_id": "string"
}

Response:
{
  "transcription": "string",
  "response": "string",
  "audio_url": "string"
}
```

#### Services API
```typescript
// Government Services
GET /api/v1/services/search?query=passport&location=delhi
GET /api/v1/services/{service_id}/requirements
POST /api/v1/services/{service_id}/apply
GET /api/v1/services/applications/{app_id}/status
```

### WebSocket API for Real-time Chat

```typescript
// WebSocket Events
interface ChatEvents {
  'message': (data: ChatMessage) => void;
  'typing': (data: TypingIndicator) => void;
  'voice_start': () => void;
  'voice_end': (data: VoiceMessage) => void;
}
```

## Database Design

### PostgreSQL Schema (User Data)

```sql
-- Users Table
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    phone_number VARCHAR(15) UNIQUE NOT NULL,
    email VARCHAR(255),
    name VARCHAR(255),
    preferred_language VARCHAR(10) DEFAULT 'en',
    location JSONB,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- User Sessions
CREATE TABLE user_sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id),
    session_data JSONB,
    created_at TIMESTAMP DEFAULT NOW(),
    expires_at TIMESTAMP
);

-- Service Applications
CREATE TABLE service_applications (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id),
    service_type VARCHAR(100),
    application_data JSONB,
    status VARCHAR(50),
    government_ref_id VARCHAR(255),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Chat History
CREATE TABLE chat_messages (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id),
    session_id UUID,
    message TEXT,
    response TEXT,
    intent VARCHAR(100),
    entities JSONB,
    language VARCHAR(10),
    created_at TIMESTAMP DEFAULT NOW()
);
```

### MongoDB Schema (AI Models & Content)

```javascript
// Knowledge Base Collection
{
  _id: ObjectId,
  service_id: "passport_application",
  service_name: "Passport Application",
  category: "travel_documents",
  requirements: [
    {
      document: "birth_certificate",
      mandatory: true,
      alternatives: ["school_certificate"]
    }
  ],
  procedures: [
    {
      step: 1,
      description: "Fill online application",
      url: "https://passportindia.gov.in"
    }
  ],
  languages: ["en", "hi", "ta", "te"],
  last_updated: ISODate()
}

// AI Training Data
{
  _id: ObjectId,
  intent: "passport_application",
  examples: [
    {
      text: "I want to apply for passport",
      language: "en",
      entities: [
        {
          entity: "service_type",
          value: "passport",
          start: 19,
          end: 27
        }
      ]
    }
  ]
}
```

### Redis Cache Structure

```redis
# User Sessions
user:session:{user_id} -> {session_data}

# Chat Context
chat:context:{session_id} -> {conversation_history}

# Service Cache
service:info:{service_id} -> {service_details}

# API Rate Limiting
rate_limit:{user_id}:{endpoint} -> {request_count}
```

### Elasticsearch Index

```json
{
  "mappings": {
    "properties": {
      "service_id": {"type": "keyword"},
      "service_name": {"type": "text", "analyzer": "multilingual"},
      "description": {"type": "text", "analyzer": "multilingual"},
      "keywords": {"type": "text", "analyzer": "multilingual"},
      "category": {"type": "keyword"},
      "location": {"type": "geo_point"},
      "language": {"type": "keyword"}
    }
  }
}
```

## Security Architecture

### Authentication & Authorization

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Client App    │───▶│   API Gateway   │───▶│  Auth Service   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │                       │
                                ▼                       ▼
                       ┌─────────────────┐    ┌─────────────────┐
                       │  JWT Validation │    │  User Database  │
                       └─────────────────┘    └─────────────────┘
```

### Security Measures

#### 1. API Security
```typescript
// JWT Token Structure
interface JWTPayload {
  user_id: string;
  phone_number: string;
  roles: string[];
  permissions: string[];
  exp: number;
  iat: number;
}

// Rate Limiting
const rateLimits = {
  chat: "100 requests per minute",
  voice: "50 requests per minute",
  document_upload: "10 requests per minute"
};
```

#### 2. Data Encryption
- **In Transit:** TLS 1.3 for all API communications
- **At Rest:** AES-256 encryption for sensitive data
- **Database:** Transparent Data Encryption (TDE)
- **Files:** Client-side encryption before upload

#### 3. Privacy Controls
```typescript
// Data Anonymization
interface AnonymizedUser {
  id: string; // hashed
  location: string; // city level only
  language: string;
  service_usage: ServiceUsage[]; // aggregated
}
```

## Scalability Design

### Horizontal Scaling Strategy

```
┌─────────────────────────────────────────────────────────────────┐
│                        Load Balancer                            │
└─────────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
┌───────▼──────┐    ┌─────────▼──────┐    ┌────────▼──────┐
│   Region 1   │    │   Region 2     │    │   Region 3    │
│   (Mumbai)   │    │  (Bangalore)   │    │   (Delhi)     │
└──────────────┘    └────────────────┘    └───────────────┘
```

### Auto-scaling Configuration

```yaml
# Kubernetes HPA Configuration
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: bharatsathi-ai-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ai-service
  minReplicas: 3
  maxReplicas: 50
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

### Database Scaling

#### Read Replicas Strategy
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Primary DB    │───▶│  Read Replica 1 │    │  Read Replica 2 │
│   (Write)       │    │   (Mumbai)      │    │  (Bangalore)    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

#### Sharding Strategy
```sql
-- User Data Sharding by Region
shard_1: users WHERE location LIKE 'north_%'
shard_2: users WHERE location LIKE 'south_%'
shard_3: users WHERE location LIKE 'east_%'
shard_4: users WHERE location LIKE 'west_%'
```

## AWS Deployment Architecture

### Infrastructure Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                          AWS Cloud                              │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │
│  │   CloudFront    │  │      Route53    │  │       WAF       │ │
│  │      (CDN)      │  │      (DNS)      │  │  (Security)     │ │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘ │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │
│  │   API Gateway   │  │  Application    │  │   Elastic       │ │
│  │                 │  │  Load Balancer  │  │  Load Balancer  │ │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘ │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │
│  │      EKS        │  │      ECS        │  │     Lambda      │ │
│  │  (Kubernetes)   │  │  (Containers)   │  │  (Serverless)   │ │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘ │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │
│  │      RDS        │  │   ElastiCache   │  │  OpenSearch     │ │
│  │  (PostgreSQL)   │  │     (Redis)     │  │ (Elasticsearch) │ │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

### Service Deployment

#### 1. Container Orchestration (EKS)

```yaml
# Kubernetes Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: bharatsathi-ai-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: bharatsathi-ai
  template:
    metadata:
      labels:
        app: bharatsathi-ai
    spec:
      containers:
      - name: ai-service
        image: bharatsathi/ai-service:latest
        ports:
        - containerPort: 8000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: url
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
```

#### 2. Database Configuration

```yaml
# RDS Configuration
DBInstanceClass: db.r5.xlarge
Engine: postgres
EngineVersion: "13.7"
AllocatedStorage: 100
StorageType: gp2
MultiAZ: true
BackupRetentionPeriod: 7
DeletionProtection: true

# ElastiCache Configuration
CacheNodeType: cache.r5.large
Engine: redis
EngineVersion: "6.2"
NumCacheNodes: 3
ReplicationGroupDescription: "BharatSathi AI Cache"
```

#### 3. Monitoring & Logging

```yaml
# CloudWatch Configuration
LogGroups:
  - /aws/eks/bharatsathi-ai/application
  - /aws/lambda/bharatsathi-ai-functions
  - /aws/rds/bharatsathi-ai/postgresql

Metrics:
  - CPUUtilization
  - MemoryUtilization
  - RequestCount
  - ErrorRate
  - ResponseTime

Alarms:
  - HighCPUAlarm: >80% for 5 minutes
  - HighErrorRate: >5% for 2 minutes
  - DatabaseConnections: >80% of max
```

### CI/CD Pipeline

```yaml
# GitHub Actions Workflow
name: Deploy BharatSathi AI
on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run Tests
        run: |
          npm test
          python -m pytest

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Build Docker Images
        run: |
          docker build -t bharatsathi/ai-service .
          docker push bharatsathi/ai-service:latest

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to EKS
        run: |
          aws eks update-kubeconfig --name bharatsathi-cluster
          kubectl apply -f k8s/
          kubectl rollout status deployment/bharatsathi-ai-service
```

### Cost Optimization

#### 1. Resource Management
```yaml
# Spot Instances for Non-Critical Workloads
NodeGroups:
  - name: spot-nodes
    instanceTypes: [m5.large, m5.xlarge]
    capacityType: SPOT
    scalingConfig:
      minSize: 1
      maxSize: 10
      desiredSize: 3

# Reserved Instances for Databases
RDSReservedInstances:
  - InstanceClass: db.r5.xlarge
    Duration: 1year
    PaymentOption: AllUpfront
```

#### 2. Auto-scaling Policies
```yaml
# Lambda Auto-scaling
ReservedConcurrency: 100
ProvisionedConcurrency: 10

# ECS Auto-scaling
TargetCapacity: 70%
ScaleOutCooldown: 300s
ScaleInCooldown: 300s
```

---

*This design document provides a comprehensive technical blueprint for implementing BharatSathi AI with scalable, secure, and maintainable architecture suitable for serving millions of Indian citizens.*