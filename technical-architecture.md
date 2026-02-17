# Multilingual Mandi - Technical Architecture

## System Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    FRONTEND LAYER                           │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────────┐ │
│  │   Web App   │ │ Mobile PWA  │ │    Admin Dashboard      │ │
│  │  (React)    │ │  (React)    │ │      (React)            │ │
│  └─────────────┘ └─────────────┘ └─────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│                     API GATEWAY                             │
│              (Express.js + Rate Limiting)                   │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│                  MICROSERVICES LAYER                        │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────────┐ │
│  │ Translation │ │ Price       │ │   Negotiation AI        │ │
│  │  Service    │ │ Discovery   │ │     Service             │ │
│  │             │ │  Service    │ │                         │ │
│  └─────────────┘ └─────────────┘ └─────────────────────────┘ │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────────┐ │
│  │ User        │ │ Analytics   │ │   Notification          │ │
│  │ Management  │ │  Service    │ │     Service             │ │
│  │             │ │             │ │                         │ │
│  └─────────────┘ └─────────────┘ └─────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│                    DATA LAYER                               │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────────┐ │
│  │ PostgreSQL  │ │    Redis    │ │    MongoDB              │ │
│  │ (User Data) │ │  (Cache)    │ │ (Conversations)         │ │
│  └─────────────┘ └─────────────┘ └─────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

## Core Technology Stack

### Frontend
- **Framework**: React 18 with TypeScript
- **State Management**: Zustand (lightweight, simple)
- **UI Library**: Tailwind CSS + Headless UI
- **Voice Processing**: Web Speech API + MediaRecorder
- **PWA**: Workbox for offline functionality
- **Build Tool**: Vite for fast development

### Backend
- **Runtime**: Node.js 18+
- **Framework**: Express.js with TypeScript
- **API Documentation**: OpenAPI 3.0 + Swagger
- **Authentication**: JWT + Refresh Tokens
- **Rate Limiting**: Express Rate Limit
- **Validation**: Zod for schema validation
### AI & ML Services
- **Speech-to-Text**: Google Cloud Speech-to-Text API
- **Translation**: Google Translate API + Custom models
- **Text-to-Speech**: Google Cloud Text-to-Speech
- **Price Prediction**: TensorFlow.js + Historical data
- **Negotiation AI**: OpenAI GPT-4 + Custom prompts

### Database & Storage
- **Primary DB**: PostgreSQL 14+ (User data, transactions)
- **Cache**: Redis 7+ (Session data, frequent queries)
- **Document Store**: MongoDB (Conversation logs)
- **File Storage**: AWS S3 (Audio files, images)
- **Search**: Elasticsearch (Price data, market intelligence)

### Infrastructure
- **Cloud Provider**: AWS (Primary) + Google Cloud (AI services)
- **Container**: Docker + Docker Compose
- **Orchestration**: AWS ECS or Kubernetes
- **CDN**: CloudFlare for global content delivery
- **Monitoring**: DataDog + Sentry for error tracking

## Detailed Service Architecture

### 1. Translation Service
```javascript
// Core translation pipeline
class TranslationService {
  async translateSpeech(audioBuffer, sourceLanguage, targetLanguage) {
    // Step 1: Speech to Text
    const transcript = await this.speechToText(audioBuffer, sourceLanguage);
    
    // Step 2: Text Translation
    const translated = await this.translateText(transcript, targetLanguage);
    
    // Step 3: Text to Speech
    const audioResponse = await this.textToSpeech(translated, targetLanguage);
    
    return {
      originalText: transcript,
      translatedText: translated,
      audioUrl: audioResponse.url,
      confidence: transcript.confidence
    };
  }
}
```

### 2. Price Discovery Service
```javascript
class PriceDiscoveryService {
  async getPriceRecommendation(product, location, quantity) {
    const factors = await this.gatherPriceFactors(product, location);
    
    return {
      costPrice: factors.averageCost,
      minimumPrice: factors.averageCost * 1.15, // 15% minimum profit
      fairPrice: factors.marketAverage,
      premiumPrice: factors.marketAverage * 1.2,
      confidence: factors.dataQuality,
      reasoning: this.generatePriceJustification(factors)
    };
  }
  
  async gatherPriceFactors(product, location) {
    return {
      marketAverage: await this.getMarketAverage(product, location),
      seasonality: await this.getSeasonalTrends(product),
      demand: await this.getDemandLevel(product, location),
      supply: await this.getSupplyLevel(product, location),
      weather: await this.getWeatherImpact(product, location),
      transport: await this.getTransportCosts(location)
    };
  }
}
```

### 3. Negotiation AI Service
```javascript
class NegotiationAIService {
  async generateResponse(context, buyerMessage, vendorProfile) {
    const prompt = this.buildNegotiationPrompt(context, buyerMessage, vendorProfile);
    
    const aiResponse = await this.callOpenAI(prompt);
    
    return {
      suggestedResponse: aiResponse.response,
      strategy: aiResponse.strategy,
      confidence: aiResponse.confidence,
      alternatives: aiResponse.alternatives,
      warnings: this.checkForRedFlags(buyerMessage)
    };
  }
  
  buildNegotiationPrompt(context, buyerMessage, vendorProfile) {
    return `
      You are an AI negotiation coach for Indian market vendors.
      
      Context:
      - Product: ${context.product}
      - Vendor's cost: ₹${context.cost}/kg
      - Fair market price: ₹${context.fairPrice}/kg
      - Buyer's message: "${buyerMessage}"
      
      Generate a polite but firm response that:
      1. Maintains respect for the buyer
      2. Justifies the price with quality/freshness
      3. Shows flexibility for bulk orders
      4. Prevents selling below cost
      
      Respond in Hindi with English translation.
    `;
  }
}
```

## Database Schema Design

### Users Table (PostgreSQL)
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  phone_number VARCHAR(15) UNIQUE NOT NULL,
  name VARCHAR(100) NOT NULL,
  preferred_language VARCHAR(10) DEFAULT 'hi',
  location JSONB, -- {city, state, market_name}
  products TEXT[], -- Array of products they sell
  cost_data JSONB, -- Product costs and margins
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

### Negotiations Table (PostgreSQL)
```sql
CREATE TABLE negotiations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  vendor_id UUID REFERENCES users(id),
  product VARCHAR(50) NOT NULL,
  quantity DECIMAL(10,2),
  initial_price DECIMAL(10,2),
  final_price DECIMAL(10,2),
  buyer_language VARCHAR(10),
  vendor_language VARCHAR(10),
  status VARCHAR(20), -- 'active', 'completed', 'cancelled'
  created_at TIMESTAMP DEFAULT NOW(),
  completed_at TIMESTAMP
);
```

### Price History (PostgreSQL)
```sql
CREATE TABLE price_history (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  product VARCHAR(50) NOT NULL,
  location VARCHAR(100) NOT NULL,
  price DECIMAL(10,2) NOT NULL,
  quantity DECIMAL(10,2),
  source VARCHAR(50), -- 'user_reported', 'market_data', 'api'
  recorded_at TIMESTAMP DEFAULT NOW(),
  
  INDEX idx_product_location_date (product, location, recorded_at)
);
```

### Conversation Logs (MongoDB)
```javascript
{
  _id: ObjectId,
  negotiationId: "uuid",
  messages: [
    {
      timestamp: ISODate,
      speaker: "buyer" | "vendor" | "ai",
      originalText: "string",
      translatedText: "string",
      language: "string",
      audioUrl: "string",
      confidence: 0.95
    }
  ],
  aiSuggestions: [
    {
      timestamp: ISODate,
      suggestion: "string",
      strategy: "string",
      used: boolean
    }
  ]
}
```

## API Endpoints Design

### Authentication Endpoints
```
POST /api/auth/login
POST /api/auth/refresh
POST /api/auth/logout
```

### User Management
```
GET /api/users/profile
PUT /api/users/profile
POST /api/users/products
GET /api/users/earnings
```

### Translation & Communication
```
POST /api/translate/speech
POST /api/translate/text
GET /api/translate/languages
```

### Price Discovery
```
GET /api/prices/:product
GET /api/prices/:product/history
POST /api/prices/recommendation
GET /api/market/intelligence
```

### Negotiation
```
POST /api/negotiations/start
PUT /api/negotiations/:id/message
GET /api/negotiations/:id/suggestions
POST /api/negotiations/:id/complete
```

## Security Implementation

### Authentication & Authorization
```javascript
// JWT Token Structure
{
  "sub": "user-uuid",
  "phone": "+91XXXXXXXXXX",
  "role": "vendor",
  "lang": "hi",
  "exp": 1640995200,
  "iat": 1640908800
}

// Rate Limiting Configuration
const rateLimiter = {
  translation: { windowMs: 60000, max: 100 }, // 100 requests per minute
  priceDiscovery: { windowMs: 60000, max: 200 },
  negotiation: { windowMs: 60000, max: 50 }
};
```

### Data Privacy & Compliance
- **Audio Data**: Encrypted at rest, deleted after 30 days
- **Personal Information**: Minimal collection, encrypted storage
- **Conversation Logs**: Anonymized for AI training
- **GDPR Compliance**: Right to deletion, data portability

## Performance Optimization

### Caching Strategy
```javascript
// Redis Cache Keys
const cacheKeys = {
  priceData: `prices:${product}:${location}:${date}`, // TTL: 1 hour
  marketIntel: `market:${location}:${date}`, // TTL: 6 hours
  userProfile: `user:${userId}`, // TTL: 24 hours
  translations: `translate:${hash}`, // TTL: 7 days
};
```

### Database Optimization
- **Read Replicas**: For price queries and analytics
- **Partitioning**: Price history by date ranges
- **Indexing**: Optimized for common query patterns
- **Connection Pooling**: Efficient database connections

### CDN & Asset Optimization
- **Audio Files**: Compressed and cached globally
- **Static Assets**: Minified and gzipped
- **Image Optimization**: WebP format with fallbacks
- **API Responses**: Compressed with gzip

## Deployment Architecture

### Development Environment
```yaml
# docker-compose.dev.yml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
    volumes:
      - .:/app
  
  postgres:
    image: postgres:14
    environment:
      POSTGRES_DB: multilingual_mandi
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: dev123
    ports:
      - "5432:5432"
  
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
```

### Production Deployment (AWS)
```yaml
# AWS ECS Task Definition
{
  "family": "multilingual-mandi",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "1024",
  "memory": "2048",
  "containerDefinitions": [
    {
      "name": "app",
      "image": "multilingual-mandi:latest",
      "portMappings": [{"containerPort": 3000}],
      "environment": [
        {"name": "NODE_ENV", "value": "production"},
        {"name": "DATABASE_URL", "valueFrom": "arn:aws:ssm:..."}
      ]
    }
  ]
}
```

## Monitoring & Analytics

### Application Metrics
- **Response Times**: API endpoint performance
- **Error Rates**: 4xx/5xx error tracking
- **Translation Accuracy**: User feedback scores
- **Negotiation Success**: Completion rates
- **User Engagement**: Daily/monthly active users

### Business Metrics
- **Revenue Impact**: Average price improvement
- **User Satisfaction**: Net Promoter Score
- **Market Penetration**: Geographic coverage
- **Feature Usage**: Most used AI suggestions

This technical architecture ensures the Multilingual Mandi platform can scale efficiently while maintaining high performance and reliability for vendors across India.