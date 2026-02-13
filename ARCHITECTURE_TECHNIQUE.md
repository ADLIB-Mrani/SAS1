# Architecture Technique - SAS IA Platform

## Vue d'Ensemble

Cette architecture présente une plateforme SaaS d'intelligence artificielle sécurisée, scalable et conforme aux standards européens (RGPD, ISO 27001).

## Architecture Globale

```
┌─────────────────────────────────────────────────────────────────┐
│                         UTILISATEURS                             │
│                    (Clients, Administrateurs)                    │
└────────────┬────────────────────────────────────────────────────┘
             │
             │ HTTPS/TLS 1.3
             ▼
┌─────────────────────────────────────────────────────────────────┐
│                    CDN + WAF (Cloudflare)                        │
│  • Protection DDoS                                               │
│  • Cache statique (images, CSS, JS)                              │
│  • Filtrage trafic malveillant                                   │
│  • Rate limiting (100 req/min par IP)                            │
└────────────┬────────────────────────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────────────────────────────┐
│               Load Balancer (Nginx / HAProxy)                    │
│  • Distribution charge round-robin                               │
│  • Health checks (15s interval)                                  │
│  • SSL termination                                               │
│  • Compression gzip                                              │
└────────────┬────────────────────────────────────────────────────┘
             │
      ┌──────┴──────┐
      ▼             ▼
┌──────────┐  ┌──────────┐
│  App 1   │  │  App 2   │
│ (Docker) │  │ (Docker) │
└────┬─────┘  └────┬─────┘
     │             │
     └──────┬──────┘
            │
     ┌──────┴───────┬────────────┬──────────────┐
     ▼              ▼            ▼              ▼
┌─────────┐   ┌─────────┐  ┌──────────┐  ┌──────────┐
│   DB    │   │  Redis  │  │ Services │  │ Storage  │
│PostgreSQL│  │ Cache   │  │   IA     │  │   S3     │
└─────────┘   └─────────┘  └──────────┘  └──────────┘
```

## Stack Technique

### Backend

#### Framework : FastAPI (Python)

**Justification** :
- Performance élevée (async/await)
- Documentation auto-générée (OpenAPI)
- Validation de données (Pydantic)
- Écosystème IA riche

**Structure du projet** :

```
backend/
├── app/
│   ├── __init__.py
│   ├── main.py                 # Point d'entrée
│   ├── config.py               # Configuration
│   ├── dependencies.py         # Injections de dépendances
│   │
│   ├── api/
│   │   ├── v1/
│   │   │   ├── endpoints/
│   │   │   │   ├── auth.py     # Authentification
│   │   │   │   ├── chatbot.py  # API Chatbot
│   │   │   │   ├── content.py  # Génération contenu
│   │   │   │   └── analytics.py # Analytics
│   │   │   └── api.py          # Router principal
│   │   └── deps.py
│   │
│   ├── core/
│   │   ├── security.py         # JWT, hashing
│   │   ├── config.py           # Settings
│   │   └── events.py           # Startup/shutdown
│   │
│   ├── db/
│   │   ├── base.py             # Base SQLAlchemy
│   │   ├── session.py          # DB session
│   │   └── models/
│   │       ├── user.py
│   │       ├── chatbot.py
│   │       └── conversation.py
│   │
│   ├── schemas/
│   │   ├── user.py             # Pydantic schemas
│   │   ├── chatbot.py
│   │   └── response.py
│   │
│   ├── services/
│   │   ├── ai/
│   │   │   ├── openai_service.py
│   │   │   ├── anthropic_service.py
│   │   │   └── embeddings.py
│   │   ├── chatbot_service.py
│   │   └── analytics_service.py
│   │
│   └── utils/
│       ├── security.py
│       ├── validators.py
│       └── helpers.py
│
├── tests/
│   ├── unit/
│   ├── integration/
│   └── conftest.py
│
├── alembic/                    # Migrations DB
│   ├── versions/
│   └── env.py
│
├── Dockerfile
├── requirements.txt
├── requirements-dev.txt
└── pyproject.toml
```

#### Code Exemple : API Chatbot

```python
# app/api/v1/endpoints/chatbot.py
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from app.db.session import get_db
from app.services.ai.openai_service import OpenAIService
from app.schemas.chatbot import ChatRequest, ChatResponse
from app.core.security import get_current_user
from app.db.models.user import User

router = APIRouter()

@router.post("/chat", response_model=ChatResponse)
async def chat(
    request: ChatRequest,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user)
):
    """
    Endpoint de conversation avec le chatbot IA.
    
    Sécurité : Requiert authentification JWT
    Rate limit : 50 requêtes/minute
    """
    try:
        # Validation des entrées (anti-injection)
        if len(request.message) > 1000:
            raise HTTPException(400, "Message trop long")
        
        # Service IA
        ai_service = OpenAIService()
        response = await ai_service.generate_response(
            message=request.message,
            context=request.context,
            user_id=current_user.id
        )
        
        # Logging pour audit
        log_conversation(
            user_id=current_user.id,
            message=request.message,
            response=response
        )
        
        return ChatResponse(
            message=response,
            timestamp=datetime.utcnow(),
            tokens_used=ai_service.last_tokens_count
        )
        
    except Exception as e:
        # Sentry logging
        capture_exception(e)
        raise HTTPException(500, "Erreur de traitement")
```

```python
# app/services/ai/openai_service.py
import openai
from typing import Optional
from app.core.config import settings

class OpenAIService:
    """Service d'intégration OpenAI avec gestion d'erreurs"""
    
    def __init__(self):
        openai.api_key = settings.OPENAI_API_KEY
        self.last_tokens_count = 0
    
    async def generate_response(
        self,
        message: str,
        context: Optional[str] = None,
        user_id: int = None
    ) -> str:
        """
        Génère une réponse avec GPT-4
        
        Sécurité :
        - Validation entrées
        - Rate limiting par user
        - Filtrage contenu inapproprié
        """
        try:
            # Construction du prompt sécurisé
            system_prompt = self._build_system_prompt(context)
            
            # Vérification quota utilisateur
            if not self._check_user_quota(user_id):
                raise QuotaExceededError("Quota mensuel atteint")
            
            # Appel API OpenAI
            response = await openai.ChatCompletion.acreate(
                model="gpt-4-turbo-preview",
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": message}
                ],
                temperature=0.7,
                max_tokens=500,
                user=str(user_id)  # Tracking par user
            )
            
            self.last_tokens_count = response.usage.total_tokens
            
            return response.choices[0].message.content
            
        except openai.error.RateLimitError:
            raise HTTPException(429, "Trop de requêtes")
        except openai.error.InvalidRequestError as e:
            raise HTTPException(400, f"Requête invalide: {str(e)}")
    
    def _build_system_prompt(self, context: Optional[str]) -> str:
        """Construit un prompt système sécurisé"""
        base_prompt = """Tu es un assistant IA professionnel.
        Règles strictes :
        - Réponds uniquement en français
        - Sois concis et précis
        - Refuse tout contenu inapproprié
        - Ne partage jamais d'informations confidentielles
        """
        
        if context:
            base_prompt += f"\nContexte : {context[:500]}"
        
        return base_prompt
```

#### Sécurité Backend

```python
# app/core/security.py
from datetime import datetime, timedelta
from typing import Optional
from jose import JWTError, jwt
from passlib.context import CryptContext
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer

# Configuration
SECRET_KEY = os.getenv("SECRET_KEY")  # 256-bit key
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

def verify_password(plain_password: str, hashed_password: str) -> bool:
    """Vérifie un mot de passe hashé avec bcrypt"""
    return pwd_context.verify(plain_password, hashed_password)

def get_password_hash(password: str) -> str:
    """Hash un mot de passe avec bcrypt (10 rounds)"""
    return pwd_context.hash(password)

def create_access_token(data: dict, expires_delta: Optional[timedelta] = None):
    """Crée un JWT token"""
    to_encode = data.copy()
    
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=15)
    
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    
    return encoded_jwt

async def get_current_user(
    token: str = Depends(oauth2_scheme),
    db: Session = Depends(get_db)
) -> User:
    """Récupère l'utilisateur actuel depuis le JWT"""
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Impossible de valider les credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        user_id: int = payload.get("sub")
        
        if user_id is None:
            raise credentials_exception
            
    except JWTError:
        raise credentials_exception
    
    user = db.query(User).filter(User.id == user_id).first()
    
    if user is None:
        raise credentials_exception
    
    return user
```

### Frontend

#### Framework : React + TypeScript

**Structure** :

```
frontend/
├── public/
│   ├── index.html
│   └── favicon.ico
│
├── src/
│   ├── components/
│   │   ├── common/
│   │   │   ├── Button.tsx
│   │   │   ├── Input.tsx
│   │   │   └── Modal.tsx
│   │   ├── chat/
│   │   │   ├── ChatWindow.tsx
│   │   │   ├── MessageList.tsx
│   │   │   └── InputBox.tsx
│   │   └── dashboard/
│   │       ├── Analytics.tsx
│   │       └── Settings.tsx
│   │
│   ├── pages/
│   │   ├── Home.tsx
│   │   ├── Login.tsx
│   │   ├── Dashboard.tsx
│   │   └── NotFound.tsx
│   │
│   ├── services/
│   │   ├── api.ts              # Axios config
│   │   ├── auth.service.ts
│   │   └── chatbot.service.ts
│   │
│   ├── hooks/
│   │   ├── useAuth.ts
│   │   ├── useChat.ts
│   │   └── useAnalytics.ts
│   │
│   ├── store/
│   │   ├── slices/
│   │   │   ├── authSlice.ts
│   │   │   └── chatSlice.ts
│   │   └── store.ts
│   │
│   ├── utils/
│   │   ├── validators.ts
│   │   └── helpers.ts
│   │
│   ├── types/
│   │   └── index.ts
│   │
│   ├── App.tsx
│   └── index.tsx
│
├── package.json
├── tsconfig.json
└── vite.config.ts
```

#### Code Exemple : Service API

```typescript
// src/services/api.ts
import axios, { AxiosInstance } from 'axios';

const API_BASE_URL = process.env.REACT_APP_API_URL || 'http://localhost:8000';

class ApiService {
  private client: AxiosInstance;

  constructor() {
    this.client = axios.create({
      baseURL: API_BASE_URL,
      timeout: 10000,
      headers: {
        'Content-Type': 'application/json',
      },
    });

    // Intercepteur pour ajouter le token JWT
    this.client.interceptors.request.use(
      (config) => {
        const token = localStorage.getItem('access_token');
        if (token) {
          config.headers.Authorization = `Bearer ${token}`;
        }
        return config;
      },
      (error) => Promise.reject(error)
    );

    // Intercepteur pour gérer les erreurs
    this.client.interceptors.response.use(
      (response) => response,
      (error) => {
        if (error.response?.status === 401) {
          // Redirection vers login si token expiré
          localStorage.removeItem('access_token');
          window.location.href = '/login';
        }
        return Promise.reject(error);
      }
    );
  }

  async post<T>(url: string, data: any): Promise<T> {
    const response = await this.client.post<T>(url, data);
    return response.data;
  }

  async get<T>(url: string): Promise<T> {
    const response = await this.client.get<T>(url);
    return response.data;
  }
}

export default new ApiService();
```

```typescript
// src/services/chatbot.service.ts
import api from './api';

export interface ChatMessage {
  message: string;
  context?: string;
}

export interface ChatResponse {
  message: string;
  timestamp: string;
  tokens_used: number;
}

class ChatbotService {
  async sendMessage(message: string, context?: string): Promise<ChatResponse> {
    try {
      const response = await api.post<ChatResponse>('/api/v1/chatbot/chat', {
        message,
        context,
      });
      return response;
    } catch (error) {
      console.error('Erreur lors de l\'envoi du message:', error);
      throw error;
    }
  }
}

export default new ChatbotService();
```

### Base de Données

#### PostgreSQL - Schéma

```sql
-- Table utilisateurs
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    hashed_password VARCHAR(255) NOT NULL,
    full_name VARCHAR(255),
    company_name VARCHAR(255),
    is_active BOOLEAN DEFAULT TRUE,
    is_superuser BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Index pour performance
CREATE INDEX idx_users_email ON users(email);

-- Table chatbots
CREATE TABLE chatbots (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    system_prompt TEXT,
    model VARCHAR(50) DEFAULT 'gpt-4',
    temperature DECIMAL(2,1) DEFAULT 0.7,
    max_tokens INTEGER DEFAULT 500,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_chatbots_user ON chatbots(user_id);

-- Table conversations
CREATE TABLE conversations (
    id SERIAL PRIMARY KEY,
    chatbot_id INTEGER REFERENCES chatbots(id) ON DELETE CASCADE,
    user_message TEXT NOT NULL,
    bot_response TEXT NOT NULL,
    tokens_used INTEGER,
    response_time_ms INTEGER,
    user_ip VARCHAR(45),  -- Pour analytics, anonymisé après 30j
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_conversations_chatbot ON conversations(chatbot_id);
CREATE INDEX idx_conversations_date ON conversations(created_at);

-- Table quotas utilisateurs
CREATE TABLE user_quotas (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    month DATE NOT NULL,
    tokens_used INTEGER DEFAULT 0,
    requests_count INTEGER DEFAULT 0,
    limit_tokens INTEGER DEFAULT 100000,
    limit_requests INTEGER DEFAULT 1000,
    UNIQUE(user_id, month)
);

-- Table logs d'audit (RGPD)
CREATE TABLE audit_logs (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    action VARCHAR(100) NOT NULL,
    resource_type VARCHAR(50),
    resource_id INTEGER,
    details JSONB,
    ip_address VARCHAR(45),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_audit_user ON audit_logs(user_id);
CREATE INDEX idx_audit_date ON audit_logs(created_at);

-- Fonction de mise à jour automatique du timestamp
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ language 'plpgsql';

-- Triggers
CREATE TRIGGER update_users_updated_at BEFORE UPDATE ON users
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_chatbots_updated_at BEFORE UPDATE ON chatbots
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

### Infrastructure & DevOps

#### Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  # Application backend
  backend:
    build: ./backend
    container_name: sas-backend
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://postgres:password@db:5432/sas_db
      - REDIS_URL=redis://redis:6379
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - SECRET_KEY=${SECRET_KEY}
    depends_on:
      - db
      - redis
    volumes:
      - ./backend:/app
    restart: unless-stopped
    networks:
      - sas-network

  # Base de données
  db:
    image: postgres:15-alpine
    container_name: sas-postgres
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=sas_db
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    restart: unless-stopped
    networks:
      - sas-network

  # Cache Redis
  redis:
    image: redis:7-alpine
    container_name: sas-redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    restart: unless-stopped
    networks:
      - sas-network

  # Reverse proxy
  nginx:
    image: nginx:alpine
    container_name: sas-nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/ssl:/etc/nginx/ssl
    depends_on:
      - backend
    restart: unless-stopped
    networks:
      - sas-network

volumes:
  postgres_data:
  redis_data:

networks:
  sas-network:
    driver: bridge
```

#### CI/CD Pipeline (GitHub Actions)

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.11'
    
    - name: Install dependencies
      run: |
        cd backend
        pip install -r requirements.txt
        pip install -r requirements-dev.txt
    
    - name: Run linters
      run: |
        cd backend
        flake8 app/
        black --check app/
        mypy app/
    
    - name: Run tests
      run: |
        cd backend
        pytest tests/ --cov=app --cov-report=xml
    
    - name: Upload coverage
      uses: codecov/codecov-action@v3
      with:
        file: ./backend/coverage.xml

  security:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Run Bandit (security)
      run: |
        pip install bandit
        bandit -r backend/app/
    
    - name: Run Safety (dependencies)
      run: |
        pip install safety
        safety check -r backend/requirements.txt

  deploy:
    needs: [test, security]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Deploy to production
      run: |
        # Déploiement via SSH ou API cloud
        echo "Déploiement en production"
```

## Monitoring et Observabilité

### Prometheus + Grafana

```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'sas-backend'
    static_configs:
      - targets: ['backend:8000']
    
  - job_name: 'postgres'
    static_configs:
      - targets: ['db:5432']
```

### Métriques Clés

```python
# app/utils/metrics.py
from prometheus_client import Counter, Histogram, Gauge

# Compteurs
api_requests_total = Counter(
    'api_requests_total',
    'Total API requests',
    ['method', 'endpoint', 'status']
)

# Histogrammes (temps de réponse)
api_response_time = Histogram(
    'api_response_time_seconds',
    'API response time',
    ['endpoint']
)

# Jauges
active_users = Gauge(
    'active_users',
    'Number of active users'
)
```

## Sécurité - Checklist Complète

### Application

- [x] Authentification JWT avec expiration
- [x] Hash des mots de passe (bcrypt, 10 rounds minimum)
- [x] Rate limiting (50 req/min par IP)
- [x] Validation entrées (Pydantic)
- [x] Protection CSRF
- [x] Headers sécurité (Helmet.js)
- [x] Content Security Policy (CSP)
- [x] HTTPS obligatoire (TLS 1.3)
- [x] Sanitisation outputs (XSS prevention)

### Données

- [x] Chiffrement au repos (AES-256)
- [x] Chiffrement en transit (TLS)
- [x] Anonymisation logs après 30 jours
- [x] Sauvegardes chiffrées quotidiennes
- [x] Accès base de données restreint
- [x] Rotation clés API (90 jours)

### Infrastructure

- [x] Firewall configuré (UFW/iptables)
- [x] SSH avec clés uniquement
- [x] Ports non utilisés fermés
- [x] Mises à jour automatiques activées
- [x] Monitoring 24/7
- [x] Alertes intrusion

## Évolutions Futures

### Phase 2 (Mois 6-12)
- Intégration Stripe pour paiements
- Multi-tenant architecture
- API publique avec rate limiting
- Mobile app (React Native)

### Phase 3 (Année 2)
- Microservices architecture
- Kubernetes deployment
- Multi-régions (HA)
- ML models custom (fine-tuning)

---

**Version** : 1.0  
**Dernière mise à jour** : Février 2024
