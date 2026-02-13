# Design Document: My_restro - Restaurant Intelligence Platform

## Overview

The  My_restro a Restaurant Intelligence Platform is an AI-powered system that transforms restaurant operations through data-driven insights, automation, and customer engagement. The platform addresses critical challenges in the restaurant industry including food waste, inventory management, customer experience, and operational efficiency.

### Core Value Propositions

1. **Customer Experience**: QR-based digital menus with AI chat assistance, multi-language support, and visual previews eliminate waiting times and language barriers
2. **Operational Intelligence**: Real-time analytics dashboard with revenue tracking, peak hours analysis, and customer behavior insights
3. **Waste Reduction**: Smart surplus management system that tracks waste patterns and enables selling discounted food or donating to charity
4. **Demand Optimization**: ML-based forecasting predicts demand to optimize inventory and reduce overproduction
5. **Direct Marketing**: Real-time push notifications for targeted offers and surplus food deals

### System Boundaries

**In Scope**:
- Digital menu presentation and QR code generation
- AI-powered chat interface for menu assistance
- Order data collection and storage
- Demand forecasting using historical data
- Food waste tracking and surplus management
- Inventory optimization recommendations
- Push notification delivery system
- Analytics dashboard with real-time metrics
- Role-based access control (Owner, Manager, Chef)
- Multi-language support for menus

**Out of Scope**:
- Payment processing (integration only)
- Table reservation system
- Kitchen display system (KDS) hardware
- Point of Sale (POS) hardware
- Delivery logistics and routing
- Accounting and payroll systems

## Architecture

### High-Level Architecture


The system follows a modern three-tier architecture with clear separation of concerns:

```
┌─────────────────────────────────────────────────────────────┐
│                     Presentation Layer                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │   Customer   │  │  Restaurant  │  │    chef      │       │
│  │   Web App    │  │   Dashboard  │  │    Portal    │       │
│  │  (QR Menu)   │  │  (  admin  ) │  │  ( order )   │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                      API Gateway Layer                      │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  REST API + WebSocket (Real-time updates)            │   │
│  │  - Authentication & Authorization                    │   │
│  │  - Rate Limiting                                     │   │
│  │  - Request Validation                                │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    Application Layer                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │    Menu      │  │    Order     │  │  Analytics   │      │
│  │   Service    │  │   Service    │  │   Service    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Forecasting │  │   Wastage    │  │  Inventory   │      │
│  │   Service    │  │   Service    │  │   Service    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Notification │  │   AI Chat    │  │    Auth      │      │
│  │   Service    │  │   Service    │  │   Service    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                       Data Layer                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  PostgreSQL  │  │    Redis     │  │   S3/Blob    │      │
│  │ (Relational) │  │   (Cache)    │  │   (Images)   │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│  ┌──────────────┐  ┌──────────────┐                         │
│  │  TimescaleDB │  │   Message    │                         │
│  │ (Time-series)│  │    Queue     │                         │
│  └──────────────┘  └──────────────┘                         │
└─────────────────────────────────────────────────────────────┘
```

### Architecture Patterns

1. **Microservices Architecture**: Each service is independently deployable and scalable
2. **Event-Driven**: Services communicate via message queue for async operations
3. **CQRS Pattern**: Separate read and write models for analytics queries
4. **Cache-Aside**: Redis caching for frequently accessed data (menus, forecasts)
5. **API Gateway**: Single entry point with authentication, rate limiting, and routing



## Components and Interfaces

### 1. Menu Service

**Responsibilities**:
- Manage menu items (CRUD operations)
- Handle QR code generation for tables
- Serve digital menus with availability status
- Support multi-language translations
- Manage menu item images


### 2. AI Chat Service

**Responsibilities**:
- Answer customer questions about menu items
- Provide ingredient information and dietary details
- Suggest dishes based on preferences
- Handle multi-language conversations


### 3. Order Service

**Responsibilities**:
- Record customer orders
- Track order status and completion
- Store order history
- Provide order data for analytics



### 4. Forecasting Service

**Responsibilities**:
- Generate demand predictions for menu items
- Train and update ML models with historical data
- Provide confidence intervals for forecasts
- Account for seasonality and trends


### 5. Wastage Service

**Responsibilities**:
- Track food waste entries
- Analyze wastage patterns
- Manage surplus food (discounts/donations)
- Generate waste reduction recommendations


### 6. Inventory Service

**Responsibilities**:
- Track ingredient stock levels
- Generate reorder recommendations
- Calculate optimal order quantities
- Account for shelf life and lead times


### 7. Notification Service

**Responsibilities**:
- Send push notifications to customers
- Manage notification preferences and opt-ins
- Track notification delivery and engagement
- Enforce rate limits and scheduling


### 8. Analytics Service

**Responsibilities**:
- Aggregate and compute performance metrics
- Provide real-time dashboard data
- Generate insights and trends
- Support custom date range queries


### 9. Auth Service

**Responsibilities**:
- Authenticate users (staff and customers)
- Manage user sessions
- Enforce role-based access control
- Audit access attempts


## Data Models

### Core Entities

#### Restaurant

```python
model Restaurant {
  id: string
  name: string
  address: Address
  contactInfo: ContactInfo
  settings: RestaurantSettings
  operatingHours: OperatingHours[]
  createdAt: Date
  updatedAt: Date
}

model RestaurantSettings {
  defaultLanguage: string
  supportedLanguages: string[]
  currency: string
  timezone: string
  wastageThreshold: number // percentage
  notificationDailyLimit: number
}
```

#### User

```python
model User {
  id: string
  email: string
  passwordHash: string
  role: UserRole
  restaurantId: string
  profile: UserProfile
  createdAt: Date
  lastLogin: Date
}

model UserProfile {
  firstName: string
  lastName: string
  phone?: string
  preferences: Record<string, any>
}
```

#### Customer

```python
model Customer {
  id: string
  phone?: string
  email?: string
  preferences: CustomerPreferences
  notificationOptIn: boolean
  location?: GeoLocation
  createdAt: Date
  lastVisit?: Date
}

model CustomerPreferences {
  language: string
  dietaryRestrictions: string[]
  favoriteItems: string[]
  notificationTypes: string[]
}
```

#### MenuItem

```python
model MenuItem {
  id: string
  restaurantId: string
  name: string
  description: string
  category: string
  price: number
  cost: number // ingredient cost
  imageUrl: string
  available: boolean
  preparationTime: number // minutes
  allergens: string[]
  dietaryInfo: string[]
  ingredients: IngredientUsage[]
  translations: Record<string, MenuItemTranslation>
  createdAt: Date
  updatedAt: Date
}

model IngredientUsage {
  ingredientId: string
  quantity: number
  unit: string
}

model MenuItemTranslation {
  name: string
  description: string
  language: string
}
```

#### Order

```python
model Order {
  id: string
  restaurantId: string
  tableId: string
  customerId?: string
  items: OrderItem[]
  subtotal: number
  tax: number
  totalAmount: number
  status: OrderStatus
  paymentStatus: PaymentStatus
  timestamp: Date
  completedAt?: Date
  metadata: OrderMetadata
}

model OrderMetadata {
  source: 'qr_menu' | 'waiter' | 'phone'
  sessionId?: string
  specialRequests?: string
}

### enumchoices
type OrderStatus = 'pending' | 'confirmed' | 'preparing' | 'ready' | 'served' | 'completed' | 'cancelled'
type PaymentStatus = 'pending' | 'paid' | 'failed' | 'refunded'
```



#### Wastage

```python
model WastageEntry {
  id: string
  restaurantId: string
  menuItemId: string
  quantity: number
  unit: string
  reason: WastageReason
  cost: number
  timestamp: Date
  recordedBy: string
  notes?: string
}

Enumchoices WastageReason = 'spoilage' | 'over_preparation' | 'customer_return' | 'cooking_error' | 'quality_issue' | 'other'

model SurplusFood {
  id: string
  restaurantId: string
  menuItemId: string
  quantity: number
  originalPrice: number
  discountedPrice: number
  discountPercent: number
  availableFrom: Date
  availableUntil: Date
  status: SurplusStatus
  disposition?: DispositionRecord
  createdAt: Date
}

EnumChoices SurplusStatus = 'available' | 'sold' | 'donated' | 'expired' | 'wasted'

model DispositionRecord {
  type: 'sold' | 'donated' | 'wasted'
  quantity: number
  timestamp: Date
  charityId?: string
  revenue?: number
}
```

#### Inventory

```python
model Ingredient {
  id: string
  restaurantId: string
  name: string
  category: string
  unit: string
  currentStock: number
  safetyThreshold: number
  reorderPoint: number
  optimalStock: number
  costPerUnit: number
  supplierId?: string
  shelfLifeDays: number
  leadTimeDays: number
  lastRestocked: Date
  updatedAt: Date
}

model StockTransaction {
  id: string
  ingredientId: string
  type: 'purchase' | 'usage' | 'wastage' | 'adjustment'
  quantity: number
  costPerUnit: number
  totalCost: number
  timestamp: Date
  reference?: string // order ID or wastage ID
  notes?: string
}
```

#### Forecast

```python
model DemandForecast {
  id: string
  restaurantId: string
  menuItemId: string
  forecastDate: Date
  predictedQuantity: number
  confidenceLower: number
  confidenceUpper: number
  actualQuantity?: number // filled after the day
  accuracy?: number // calculated after actual data available
  modelVersion: string
  generatedAt: Date
}

model ForecastModel {
  id: string
  restaurantId: string
  modelType: 'time_series' | 'regression' | 'ensemble'
  version: string
  parameters: Record<string, any>
  trainingDataPeriod: DateRange
  metrics: ModelMetrics
  trainedAt: Date
  active: boolean
}

model ModelMetrics {
  mae: number // Mean Absolute Error
  rmse: number // Root Mean Square Error
  mape: number // Mean Absolute Percentage Error
  r2Score: number
}
```

#### Notification

```python
model Notification {
  id: string
  restaurantId: string
  type: NotificationType
  title: string
  body: string
  imageUrl?: string
  actionUrl?: string
  targetSegment?: string
  targetCustomers?: string[]
  location?: GeoLocation
  radius?: number // meters
  scheduledFor: Date
  sentAt?: Date
  status: NotificationStatus
  metrics: NotificationMetrics
  createdBy: string
  createdAt: Date
}

EnumChoices NotificationType = 'marketing' | 'surplus_food' | 'special_offer' | 'new_menu_item' | 'loyalty_reward'
EnumChoices NotificationStatus = 'draft' | 'scheduled' | 'sending' | 'sent' | 'failed'

model NotificationDelivery {
  id: string
  notificationId: string
  customerId: string
  deliveredAt: Date
  openedAt?: Date
  clickedAt?: Date
  deviceToken: string
}
```



### Database Schema Design

**PostgreSQL Tables** (Relational Data):
- restaurants, users, customers
- menu_items, menu_categories, menu_translations
- ingredients, stock_transactions
- surplus_food, donations
- notifications, notification_deliveries
- audit_logs

**TimescaleDB Hypertables** (Time-Series Data):
- orders (partitioned by timestamp)
- wastage_entries (partitioned by timestamp)
- demand_forecasts (partitioned by forecast_date)
- analytics_snapshots (partitioned by timestamp)

**Redis Cache Structure**:
- `menu:{restaurantId}:{language}` - Cached menu data (TTL: 5 minutes)
- `forecast:{restaurantId}:{date}` - Daily forecasts (TTL: 24 hours)
- `dashboard:{restaurantId}` - Real-time metrics (TTL: 30 seconds)
- `session:{sessionId}` - User sessions (TTL: 24 hours)

**S3/Blob Storage**:
- Menu item images: `images/menu/{restaurantId}/{itemId}.jpg`
- Notification images: `images/notifications/{notificationId}.jpg`
- Analytics reports: `reports/{restaurantId}/{year}/{month}/`



## API Design

### REST API Endpoints

#### Menu Endpoints

```
GET    /api/v1/restaurants/{restaurantId}/menu
  Query: ?language=en&qrCode=xyz
  Response: Menu with items and categories

GET    /api/v1/menu-items/{itemId}
  Query: ?language=en
  Response: Detailed menu item information

PUT    /api/v1/menu-items/{itemId}/availability
  Body: { available: boolean }
  Response: Updated menu item

POST   /api/v1/qr-codes
  Body: { restaurantId, tableId }
  Response: QR code data and image URL
```

#### Order Endpoints

```
POST   /api/v1/orders
  Body: { restaurantId, tableId, items[], customerId? }
  Response: Created order with ID

GET    /api/v1/orders/{orderId}
  Response: Order details

GET    /api/v1/restaurants/{restaurantId}/orders
  Query: ?startDate=&endDate=&status=
  Response: List of orders with pagination

PUT    /api/v1/orders/{orderId}/status
  Body: { status: OrderStatus }
  Response: Updated order
```

#### Analytics Endpoints

```
GET    /api/v1/restaurants/{restaurantId}/analytics/dashboard
  Query: ?startDate=&endDate=
  Response: Dashboard metrics

GET    /api/v1/restaurants/{restaurantId}/analytics/peak-hours
  Query: ?startDate=&endDate=
  Response: Peak hours analysis

GET    /api/v1/restaurants/{restaurantId}/analytics/popular-dishes
  Query: ?startDate=&endDate=&limit=10
  Response: Popular dishes ranking

GET    /api/v1/restaurants/{restaurantId}/analytics/customer-behavior
  Response: Customer behavior metrics
```

#### Forecasting Endpoints

```
GET    /api/v1/restaurants/{restaurantId}/forecasts
  Query: ?date=&itemId=
  Response: Demand forecasts

POST   /api/v1/restaurants/{restaurantId}/forecasts/generate
  Body: { date }
  Response: Generated forecast

POST   /api/v1/restaurants/{restaurantId}/forecasts/train
  Response: Model training status
```

#### Wastage Endpoints

```
POST   /api/v1/wastage
  Body: { restaurantId, menuItemId, quantity, reason, cost }
  Response: Created wastage entry

GET    /api/v1/restaurants/{restaurantId}/wastage/report
  Query: ?startDate=&endDate=
  Response: Wastage report with patterns

GET    /api/v1/restaurants/{restaurantId}/wastage/patterns
  Response: Analyzed wastage patterns

POST   /api/v1/surplus-food
  Body: { menuItemId, quantity, discountPercent }
  Response: Created surplus listing

GET    /api/v1/restaurants/{restaurantId}/surplus-food
  Query: ?status=available
  Response: List of surplus food items
```

#### Inventory Endpoints

```
GET    /api/v1/restaurants/{restaurantId}/inventory
  Response: Current stock levels

PUT    /api/v1/inventory/{ingredientId}/stock
  Body: { quantity, operation: 'add' | 'subtract' }
  Response: Updated stock level

GET    /api/v1/restaurants/{restaurantId}/inventory/recommendations
  Response: Stock adjustment recommendations

GET    /api/v1/restaurants/{restaurantId}/inventory/alerts
  Response: Reorder alerts
```

#### Notification Endpoints

```
POST   /api/v1/notifications
  Body: { restaurantId, type, title, body, targetCustomers, scheduledFor? }
  Response: Created notification

GET    /api/v1/notifications/{notificationId}/metrics
  Response: Notification performance metrics

POST   /api/v1/notifications/location-based
  Body: { location, radius, content }
  Response: Sent notification result

PUT    /api/v1/customers/{customerId}/notification-preferences
  Body: { optIn, types[] }
  Response: Updated preferences
```

#### AI Chat Endpoints

```
POST   /api/v1/chat/sessions
  Body: { restaurantId, customerId?, language }
  Response: Created chat session

POST   /api/v1/chat/sessions/{sessionId}/messages
  Body: { message, language }
  Response: AI response with suggestions

GET    /api/v1/chat/sessions/{sessionId}
  Response: Chat session history
```

#### Auth Endpoints

```
POST   /api/v1/auth/login
  Body: { email, password }
  Response: Auth tokens and user info

POST   /api/v1/auth/logout
  Header: Authorization: Bearer {token}
  Response: Success

POST   /api/v1/auth/refresh
  Body: { refreshToken }
  Response: New access token

GET    /api/v1/auth/me
  Header: Authorization: Bearer {token}
  Response: Current user info
```

### WebSocket Events

**Real-time Dashboard Updates**:
```
Channel: dashboard:{restaurantId}
Events:
  - order.created
  - order.completed
  - metrics.updated
  - alert.triggered
```

**Order Status Updates**:
```
Channel: orders:{restaurantId}
Events:
  - order.status_changed
  - order.assigned
```



## Technology Stack

### Backend

**Primary Language**: Django (python)
- Type safety for complex business logic
- Rich ecosystem for web services
- Excellent async/await support for I/O operations

**API Framework**: reactjs.js 
- Mature, well-documented framework
- Extensive middleware ecosystem
- Easy integration with authentication and validation libraries

**Alternative Consideration**: NestJS
- Built-in dependency injection
- Better structure for microservices
- TypeScript-first design

**ML/Forecasting**: Python
- Separate microservice for forecasting
- Libraries: scikit-learn, statsmodels, Prophet (Facebook's time series library)
- REST API to communicate with main backend

### Databases

**Primary Database**: PostgreSQL 14+
- ACID compliance for transactional data
- JSON support for flexible schemas
- Strong consistency guarantees
- Excellent query performance

**Time-Series Data**: TimescaleDB (PostgreSQL extension)
- Optimized for time-series queries
- Automatic partitioning by time
- Continuous aggregates for analytics
- Compatible with PostgreSQL tools

**Cache Layer**: Redis 7+
- In-memory caching for frequently accessed data
- Pub/sub for real-time updates
- Session storage
- Rate limiting implementation

**Object Storage**: AWS S3 or Azure Blob Storage
- Menu item images
- Generated reports
- Backup storage

### Frontend

**Customer Web App** (QR Menu):
- React with vite
- Tailwind CSS for styling
- React Query for data fetching
- i18next for multi-language support
- Progressive Web App (PWA) for offline capability

**Restaurant Dashboard**:
- React with vite
- Recharts or Chart.js for analytics visualizations
- WebSocket client for real-time updates
- Responsive design for tablet/desktop

### AI/ML Services

**AI Chat**: OpenAI GPT-4 API or Anthropic Claude
- Natural language understanding
- Context-aware responses
- Multi-language support
- Fallback to rule-based system for common queries

**Demand Forecasting**: Python-based service
- Prophet for time series forecasting
- scikit-learn for feature engineering
- Scheduled daily training jobs
- REST API for predictions

### Infrastructure

**Containerization**: Docker
- Consistent deployment environments
- Easy local development setup
- Microservices isolation

**Orchestration**: Kubernetes or AWS ECS
- Service scaling
- Load balancing
- Health checks and auto-recovery

**Message Queue**: RabbitMQ or AWS SQS
- Async task processing
- Event-driven architecture
- Notification delivery queue

**Push Notifications**: Firebase Cloud Messaging (FCM)
- Cross-platform support (iOS, Android, Web)
- Reliable delivery
- Rich notification support

**Monitoring**: 
- Application: Datadog or New Relic
- Logs: ELK Stack (Elasticsearch, Logstash, Kibana)
- Errors: Sentry
- or Grafana

### Development Tools

**API Documentation**: OpenAPI/Swagger
**Testing**: Jest (unit), Supertest (integration), Playwright (E2E), Postman
**CI/CD**: GitHub Actions or GitLab CI
**Code Quality**: ESLint, Prettier, Husky (pre-commit hooks), Sonarqube



## Implementation Approach

### Phase 1: Core Menu and Order System (Requirements 1, 2)

**Objective**: Enable customers to view digital menus and place orders

**Implementation Steps**:
1. Set up PostgreSQL database with core tables (restaurants, menu_items, orders)
2. Implement Menu Service with CRUD operations
3. Create QR code generation endpoint
4. Build customer web app for menu viewing
5. Implement Order Service for order creation and tracking
6. Add basic multi-language support using i18next
7. Integrate image storage for menu items

**Key Decisions**:
- Use PostgreSQL JSONB for menu translations (flexible schema)
- Generate QR codes server-side using `qrcode` npm package
- Store QR code mappings in database (qr_code_id -> restaurant_id + table_id)

### Phase 2: AI Chat Assistant (Requirement 1.5-1.8)

**Objective**: Provide AI-powered menu assistance

**Implementation Steps**:
1. Create AI Chat Service with session management
2. Integrate OpenAI GPT-4 API with menu context
3. Build chat UI component in customer web app
4. Implement conversation context tracking
5. Add multi-language chat support
6. Create fallback responses for common questions

**Key Decisions**:
- Use GPT-4 with function calling for structured responses
- Maintain conversation context in Redis (TTL: 1 hour)
- Pre-load menu context into system prompt for better responses
- Implement rate limiting (10 messages per session per minute)

### Phase 3: Analytics and Dashboard (Requirements 7, 9)

**Objective**: Provide real-time insights to restaurant staff

**Implementation Steps**:
1. Set up TimescaleDB for time-series order data
2. Implement Analytics Service with aggregation queries
3. Create continuous aggregates for common metrics
4. Build dashboard UI with real-time updates
5. Implement WebSocket connection for live metrics
6. Add peak hours and popular dishes analysis

**Key Decisions**:
- Use TimescaleDB continuous aggregates for performance
- Cache dashboard metrics in Redis (TTL: 30 seconds)
- Use WebSocket for real-time updates instead of polling
- Pre-calculate common aggregations in background jobs

### Phase 4: Demand Forecasting (Requirement 3)

**Objective**: Predict future demand for menu items

**Implementation Steps**:
1. Create Python microservice for ML models
2. Implement data pipeline to extract historical order data
3. Train initial models using Prophet (Facebook's time series library)
4. Create REST API for forecast requests
5. Implement daily forecast generation job
6. Add forecast accuracy tracking

**Key Decisions**:
- Use Prophet for time series forecasting (handles seasonality well)
- Require minimum 7 days of data before generating forecasts
- Store forecasts in TimescaleDB for historical tracking
- Run daily training job at 2 AM (low traffic period)
- Calculate confidence intervals using Prophet's built-in uncertainty

**Forecasting Algorithm**:
```python
# Simplified forecasting approach
def generate_forecast(menu_item_id, historical_orders):
    # Prepare data: aggregate daily quantities
    df = prepare_time_series_data(historical_orders)
    
    # Train Prophet model
    model = Prophet(
        yearly_seasonality=True,
        weekly_seasonality=True,
        daily_seasonality=False
    )
    model.fit(df)
    
    # Generate forecast for next 7 days
    future = model.make_future_dataframe(periods=7)
    forecast = model.predict(future)
    
    return forecast[['ds', 'yhat', 'yhat_lower', 'yhat_upper']]
```



### Phase 5: Wastage Tracking and Surplus Management (Requirements 4, 10)

**Objective**: Track food waste and enable surplus food management

**Implementation Steps**:
1. Implement Wastage Service with entry recording
2. Create wastage pattern analysis algorithms
3. Build surplus food management features
4. Add donation tracking functionality
5. Implement wastage alerts and thresholds
6. Create wastage reports and visualizations

**Key Decisions**:
- Store wastage entries in TimescaleDB for trend analysis
- Calculate wastage rates as percentage of total ingredient usage
- Trigger alerts when wastage exceeds configurable threshold (default: 10%)
- Allow marking surplus food with discount percentage (10-70%)
- Track disposition: sold, donated, or wasted


### Phase 6: Inventory Optimization (Requirement 5)

**Objective**: Provide automated stock recommendations

**Implementation Steps**:
1. Implement Inventory Service with stock tracking
2. Create ingredient-to-menu-item mapping
3. Build stock recommendation algorithm
4. Implement reorder alerts
5. Add shelf life and lead time tracking
6. Create inventory dashboard

**Key Decisions**:
- Calculate recommended order quantity using Economic Order Quantity (EOQ) formula
- Account for lead time and safety stock
- Consider shelf life when recommending quantities
- Trigger reorder alerts at safety threshold (default: 20% of optimal stock)



### Phase 7: Push Notifications (Requirement 6)

**Objective**: Enable targeted marketing and surplus food notifications

**Implementation Steps**:
1. Integrate Firebase Cloud Messaging (FCM)
2. Implement Notification Service with targeting logic
3. Add customer preference management
4. Build notification scheduling system
5. Implement rate limiting (3 per customer per day)
6. Add notification analytics tracking
7. Create location-based notification support

**Key Decisions**:
- Use FCM for cross-platform push notifications
- Store device tokens in customer records
- Implement daily rate limit using Redis counters
- Support notification templates for common scenarios
- Track metrics: sent, delivered, opened, clicked
- Use geohashing for efficient location-based queries


### Phase 8: Role-Based Access Control (Requirement 8)

**Objective**: Implement secure authentication and authorization

**Implementation Steps**:
1. Implement Auth Service with JWT tokens
2. Define role permissions matrix
3. Add middleware for permission checking
4. Implement audit logging
5. Add session management
6. Create user management UI for owners

**Key Decisions**:
- Use JWT for stateless authentication
- Store refresh tokens in database for revocation
- Implement role-based permissions using middleware
- Log all access attempts for security auditing
- Use bcrypt for password hashing (cost factor: 12)

**Permission Matrix**:
```typescript
const PERMISSIONS = {
  owner: [
    'users:*',
    'menu:*',
    'orders:*',
    'analytics:*',
    'forecasts:*',
    'inventory:*',
    'wastage:*',
    'notifications:*',
    'settings:*'
  ],
  manager(optional): [
    'menu:read',
    'menu:update',
    'orders:*',
    'analytics:*',
    'inventory:*',
    'wastage:read',
    'notifications:create',
    'notifications:read'
  ],
  chef: [
    'menu:read',
    'menu:update_availability',
    'orders:read',
    'orders:update_status',
    'wastage:*',
    'analytics:read:kitchen'
  ],
  customer: [
    'menu:read',
    'orders:create',
    'orders:read:own'
  ]
}
```

### Phase 9: Customer Behavior Tracking (Requirement 11)

**Objective**: Track and analyze customer preferences

**Implementation Steps**:
1. Implement customer identification (phone/email)
2. Track order history per customer
3. Build preference analysis algorithms
4. Implement customer segmentation
5. Add privacy controls and data deletion
6. Create customer insights dashboard

**Key Decisions**:
- Use phone number or email as customer identifier
- Allow anonymous orders (no customer tracking)
- Implement GDPR-compliant data deletion
- Segment customers by: frequency, spending, preferences
- Store minimal PII, encrypt sensitive data



### Phase 10: Security and Privacy (Requirement 12)

**Objective**: Ensure data security and regulatory compliance

**Implementation Steps**:
1. Implement encryption at rest (database level)
2. Enable TLS/HTTPS for all API endpoints
3. Add rate limiting to prevent abuse
4. Implement audit logging for all data access
5. Add GDPR compliance features (data export, deletion)
6. Set up security monitoring and alerts

**Key Decisions**:
- Use AES-256 encryption for sensitive data at rest
- Enforce TLS 1.3 for all connections
- Implement rate limiting using Redis (sliding window)
- Store audit logs in separate database for immutability
- Provide customer data export in JSON format
- Implement data retention policies (12 months for orders)


## Correctness Properties

A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.

### Property Reflection

After analyzing all 60 acceptance criteria, several opportunities for consolidation were identified:

- **Menu data completeness**: Multiple criteria (1.2, 1.6) test that menu items contain required fields - consolidated into one comprehensive property
- **Data persistence round-trips**: Multiple criteria (2.5, 11.1) test data integrity through storage and retrieval - consolidated into general round-trip properties
- **Permission checking**: Multiple criteria (8.1, 8.2, 8.3, 8.4) test role-based access - consolidated into one property per role plus denial
- **Notification tracking**: Multiple criteria (6.3, 6.9) test tracking metrics - consolidated into comprehensive tracking property
- **Dashboard metrics completeness**: Multiple criteria (7.1, 7.8) test metric presence - consolidated into one property
- **Wastage analysis**: Multiple criteria (4.2, 10.1, 10.2) test pattern analysis - consolidated into comprehensive pattern property

### Menu and QR Code Properties

**Property 1: QR Code to Menu Mapping**
*For any* valid QR code associated with a restaurant, scanning it should return a menu containing at least one menu item for that restaurant.
**Validates: Requirements 1.1**

**Property 2: Menu Item Completeness**
*For any* menu item returned by the system, it should contain all required fields: name, description, price, imageUrl, category, and available status.
**Validates: Requirements 1.2, 1.6**

**Property 3: Availability Marking**
*For any* menu item marked as unavailable, the menu response should include available=false for that item.
**Validates: Requirements 1.3**

**Property 4: Unique QR Codes per Table**
*For any* restaurant with multiple tables, generating QR codes for different tables should produce unique QR code identifiers that map to the correct table.
**Validates: Requirements 1.4**

**Property 5: Multi-language Menu Translation**
*For any* menu requested in a supported language, all translatable fields (item names, descriptions, categories) should be returned in the requested language.
**Validates: Requirements 1.7, 1.8**

### AI Chat Properties

**Property 6: Chat Response Generation**
*For any* valid chat message sent to an active session, the system should return a response with non-empty message text and a confidence score.
**Validates: Requirements 1.5**

### Order Properties

**Property 7: Order Data Completeness**
*For any* created order, the stored order should contain all required fields: items, quantities, timestamp (accurate to the minute), table identifier, and total amount.
**Validates: Requirements 2.1, 2.3**

**Property 8: Order Completion Data**
*For any* order marked as completed, the order record should contain a non-null completedAt timestamp and a payment status.
**Validates: Requirements 2.2**

**Property 9: Order Round-Trip Integrity**
*For any* order created in the system, retrieving that order by ID should return an order with identical items, quantities, and total amount.
**Validates: Requirements 2.5**



### Forecasting Properties

**Property 10: Forecast Generation with Sufficient Data**
*For any* restaurant with at least 7 days of historical order data, requesting a forecast should return predictions for each menu item.
**Validates: Requirements 3.1, 3.2**

**Property 11: Forecast Confidence Intervals**
*For any* generated forecast, it should include predicted quantity along with lower and upper confidence bounds where lower ≤ predicted ≤ upper.
**Validates: Requirements 3.5**

**Property 12: Day-of-Week Pattern Recognition**
*For any* menu item with historical data showing different demand on different days of the week, forecasts for those days should reflect the pattern (not be identical).
**Validates: Requirements 3.3**

**Property 13: Forecast Updates with New Data**
*For any* restaurant, generating a forecast after adding new order data should produce a different forecast than before the data was added (when patterns change).
**Validates: Requirements 3.4**

### Wastage and Surplus Properties

**Property 14: Wastage Entry Completeness**
*For any* recorded wastage entry, it should contain all required fields: menuItemId, quantity, reason, cost, and timestamp.
**Validates: Requirements 4.1**

**Property 15: Wastage Pattern Identification**
*For any* menu item with wastage entries exceeding 10% of total orders over a period, the pattern analysis should identify it as a high-waste item.
**Validates: Requirements 4.2, 10.1, 10.2**

**Property 16: Wastage Alert Generation**
*For any* menu item where wastage exceeds the configured threshold, an alert should be generated for chef and manager roles.
**Validates: Requirements 4.3**

**Property 17: Wastage Financial Impact Calculation**
*For any* set of wastage entries, the calculated financial impact should equal the sum of (quantity × cost) for all entries.
**Validates: Requirements 4.4, 10.4**

**Property 18: Surplus Food Creation**
*For any* menu item marked as surplus with a discount percentage, the system should create a surplus listing with discountedPrice = originalPrice × (1 - discountPercent/100).
**Validates: Requirements 4.6, 4.7**

**Property 19: Surplus Disposition Tracking**
*For any* surplus food item, updating its disposition (sold/donated/wasted) should be tracked and the status should reflect the disposition type.
**Validates: Requirements 4.10**

**Property 20: Surplus Notification Triggering**
*For any* surplus food item created with status 'available', a notification should be automatically generated with type 'surplus_food'.
**Validates: Requirements 4.9**

### Inventory Properties

**Property 21: Stock Recommendation Generation**
*For any* ingredient with current stock below its safety threshold, the system should generate a stock recommendation with positive recommended quantity.
**Validates: Requirements 5.1, 5.3**

**Property 22: Stock Recommendation Calculation**
*For any* stock recommendation, the recommended order quantity should account for current stock, lead time, and predicted demand (quantity > 0 when stock is low).
**Validates: Requirements 5.2**

**Property 23: Shelf Life Constraint**
*For any* stock recommendation for an ingredient with shelf life, the recommended quantity should not exceed the amount that can be used within the shelf life period.
**Validates: Requirements 5.4**

**Property 24: Recommendation Justification**
*For any* generated stock recommendation, it should include a non-empty justification string explaining why the recommendation was made.
**Validates: Requirements 5.5**



### Notification Properties

**Property 25: Notification Targeting Options**
*For any* notification creation request, the system should accept either a list of specific customer IDs or 'all' as valid targeting options.
**Validates: Requirements 6.1**

**Property 26: Opt-In Filtering**
*For any* notification sent to multiple customers, only customers with notificationOptIn=true and the notification type in their preferences should receive it.
**Validates: Requirements 6.2**

**Property 27: Notification Rate Limiting**
*For any* customer, attempting to send more than 3 notifications in a 24-hour period should result in the 4th and subsequent notifications being blocked for that customer.
**Validates: Requirements 6.4**

**Property 28: Notification Scheduling**
*For any* notification created with a future scheduledFor timestamp, the notification should be stored with status='scheduled' and not sent immediately.
**Validates: Requirements 6.5**

**Property 29: Location-Based Targeting**
*For any* location-based notification with a specified radius, only customers whose location is within the radius should be targeted.
**Validates: Requirements 6.7**

**Property 30: Notification Metrics Tracking**
*For any* sent notification, the system should track and calculate metrics including sent count, delivered count, opened count, open rate, and click-through rate.
**Validates: Requirements 6.3, 6.9**

### Analytics Properties

**Property 31: Dashboard Metrics Completeness**
*For any* dashboard metrics request, the response should include all required fields: daily revenue, yesterday's revenue, monthly revenue, order count, average order value, table turnover rate, and average service time.
**Validates: Requirements 7.1, 7.8**

**Property 32: Peak Hours Data Structure**
*For any* peak hours analysis, the response should include hourly distribution data for all operating hours with order counts and revenue per hour.
**Validates: Requirements 7.2, 9.3**

**Property 33: Popular Dishes Ranking**
*For any* popular dishes query, the returned list should be sorted in descending order by order frequency (most popular first).
**Validates: Requirements 7.3, 9.2**

**Property 34: Dashboard Filtering**
*For any* dashboard request with a date range filter, all returned metrics should only include data from orders within that date range.
**Validates: Requirements 7.5**

**Property 35: Real-Time Metrics Update**
*For any* restaurant, creating a new completed order should cause the dashboard metrics (when refreshed) to reflect the new order in revenue and order count.
**Validates: Requirements 7.6**

**Property 36: Trending Items Identification**
*For any* menu item with increasing order frequency over consecutive time periods, the system should identify it as a trending item with trend='increasing'.
**Validates: Requirements 9.4**



### Role-Based Access Control Properties

**Property 37: Owner Full Access**
*For any* API endpoint in the system, an authenticated user with role='owner' should be granted access.
**Validates: Requirements 8.1**

**Property 38: Manager Limited Access**
*For any* manager-permitted endpoint (analytics, inventory, orders, customers), an authenticated user with role='manager' should be granted access, but user administration endpoints should be denied.
**Validates: Requirements 8.2**

**Property 39: Chef Kitchen Access**
*For any* chef-permitted endpoint (menu items, orders, wastage, kitchen analytics), an authenticated user with role='chef' should be granted access, but inventory and forecasting endpoints should be denied.
**Validates: Requirements 8.3**

**Property 40: Permission Denial with Message**
*For any* user attempting to access an endpoint outside their role permissions, the system should return a 403 status code with an error message explaining the denial.
**Validates: Requirements 8.4**

**Property 41: Access Audit Logging**
*For any* API request to a protected endpoint, an audit log entry should be created containing userId, resource, action, granted status, and timestamp.
**Validates: Requirements 8.5, 12.4**

### Customer Behavior Properties

**Property 42: Order History Tracking**
*For any* customer who places multiple orders, retrieving their order history should return all orders associated with their customer ID in chronological order.
**Validates: Requirements 11.1**

**Property 43: Repeat Customer Identification**
*For any* customer with more than one order, the system should calculate their visit frequency as the number of orders divided by the time span between first and last order.
**Validates: Requirements 11.2**

**Property 44: Customer Preference Analysis**
*For any* customer with order history, their customer profile should include calculated preferences: favorite items (most frequently ordered) and average spending per order.
**Validates: Requirements 11.3**

**Property 45: Customer Segmentation**
*For any* restaurant with sufficient customer data, customers should be segmented into groups based on behavior patterns (frequency, spending, preferences).
**Validates: Requirements 11.4**

### Security and Privacy Properties

**Property 46: Password Hashing**
*For any* user account created or updated, the stored password should be hashed (not plaintext) and verifiable using bcrypt or equivalent secure hashing.
**Validates: Requirements 12.2**

**Property 47: Authentication Rate Limiting**
*For any* user attempting to login, making more than 5 failed login attempts within 5 minutes should result in subsequent attempts being rate limited.
**Validates: Requirements 12.3**

**Property 48: Customer Data Deletion**
*For any* customer requesting data deletion, all personal data (name, email, phone, order history) should be removed or anonymized from the system.
**Validates: Requirements 11.5, 12.5**

**Property 49: Sensitive Data Encryption**
*For any* sensitive customer data (email, phone, payment details) stored in the database, it should be encrypted at rest using AES-256 or equivalent.
**Validates: Requirements 12.1**



## Error Handling

### Error Response Format

All API errors follow a consistent format:

```typescript
interface ErrorResponse {
  error: {
    code: string
    message: string
    details?: Record<string, any>
    timestamp: Date
    requestId: string
  }
}
```

### Error Categories

**1. Validation Errors (400 Bad Request)**
- Invalid input data (missing required fields, wrong data types)
- Business rule violations (discount > 100%, negative quantities)
- Example: `{ code: 'INVALID_INPUT', message: 'Discount percentage must be between 0 and 100' }`

**2. Authentication Errors (401 Unauthorized)**
- Missing or invalid authentication token
- Expired token
- Example: `{ code: 'AUTH_REQUIRED', message: 'Valid authentication token required' }`

**3. Authorization Errors (403 Forbidden)**
- Insufficient permissions for requested action
- Role-based access denial
- Example: `{ code: 'INSUFFICIENT_PERMISSIONS', message: 'Manager role cannot access user administration' }`

**4. Not Found Errors (404 Not Found)**
- Resource doesn't exist
- Invalid IDs
- Example: `{ code: 'RESOURCE_NOT_FOUND', message: 'Menu item with ID xyz not found' }`

**5. Rate Limit Errors (429 Too Many Requests)**
- Exceeded rate limits
- Too many notifications sent
- Example: `{ code: 'RATE_LIMIT_EXCEEDED', message: 'Maximum 3 notifications per customer per day exceeded' }`

**6. Server Errors (500 Internal Server Error)**
- Unexpected errors
- Database connection failures
- External service failures (AI API, FCM)
- Example: `{ code: 'INTERNAL_ERROR', message: 'An unexpected error occurred' }`

### Error Handling Strategies

**Database Errors**:
- Retry transient failures (connection timeouts) up to 3 times with exponential backoff
- Log all database errors with full context
- Return generic error to client, detailed error in logs

**External Service Failures**:
- AI Chat: Fallback to rule-based responses if API fails
- FCM: Queue failed notifications for retry (max 3 attempts)
- Image Storage: Return placeholder image if upload fails

**Data Validation**:
- Validate all inputs at API boundary using validation middleware
- Return specific error messages for each validation failure
- Sanitize user inputs to prevent injection attacks

**Forecasting Errors**:
- Return cached forecast if model training fails
- Log model errors for investigation
- Gracefully degrade to simple moving average if ML model unavailable

**Concurrent Access**:
- Use optimistic locking for inventory updates
- Retry on conflict with exponential backoff
- Return conflict error if retries exhausted



## Testing Strategy

### Dual Testing Approach

The system requires both unit tests and property-based tests for comprehensive coverage:

**Unit Tests**: Verify specific examples, edge cases, and error conditions
- Specific scenarios with known inputs and outputs
- Integration points between services
- Error handling paths
- Edge cases (empty data, boundary values)

**Property-Based Tests**: Verify universal properties across all inputs
- Universal correctness properties that hold for all valid inputs
- Comprehensive input coverage through randomization
- Catch unexpected edge cases through fuzzing
- Validate business rules across the input space

Both approaches are complementary and necessary. Unit tests catch concrete bugs in specific scenarios, while property tests verify general correctness across many inputs.

### Property-Based Testing Configuration

**Library Selection**: 
- **TypeScript/JavaScript**: fast-check
- **Python** (forecasting service): Hypothesis

**Test Configuration**:
- Minimum 100 iterations per property test (due to randomization)
- Each property test must reference its design document property
- Tag format: `// Feature: restaurant-intelligence-platform, Property {number}: {property_text}`


### Unit Testing Strategy

**Focus Areas for Unit Tests**:
1. Specific examples demonstrating correct behavior
2. Edge cases: empty arrays, null values, boundary conditions
3. Error conditions: invalid inputs, missing data, constraint violations
4. Integration between components: service-to-service calls
5. Complex calculations: wastage rates, stock recommendations, forecast accuracy

**Avoid Over-Testing**:
- Don't write many unit tests for scenarios covered by property tests
- Property tests handle comprehensive input coverage
- Unit tests should focus on specific important cases

### Test Coverage by Component

**Menu Service**:
- Unit: QR code generation, translation fallbacks, image upload errors
- Property: Menu completeness, availability marking, multi-language support

**Order Service**:
- Unit: Order total calculation, payment status transitions, invalid order items
- Property: Order data completeness, round-trip integrity, timestamp accuracy

**Forecasting Service**:
- Unit: Insufficient data handling, model training failures, specific forecast scenarios
- Property: Forecast generation with sufficient data, confidence intervals, pattern recognition

**Wastage Service**:
- Unit: Specific wastage calculations, alert threshold edge cases, disposition updates
- Property: Wastage entry completeness, pattern identification, financial impact calculation

**Inventory Service**:
- Unit: EOQ calculation edge cases, shelf life expiration, specific reorder scenarios
- Property: Stock recommendation generation, shelf life constraints, justification presence

**Notification Service**:
- Unit: FCM integration errors, template rendering, specific targeting scenarios
- Property: Opt-in filtering, rate limiting, location-based targeting, metrics tracking

**Analytics Service**:
- Unit: Specific metric calculations, date range edge cases, empty data handling
- Property: Dashboard completeness, filtering, real-time updates, ranking correctness

**Auth Service**:
- Unit: Token expiration, refresh token flow, specific permission checks
- Property: Role-based access, permission denial, audit logging, password hashing

### Integration Testing

**API Integration Tests**:
- Test complete request/response flows
- Verify authentication and authorization
- Test error responses
- Validate WebSocket connections

**Database Integration Tests**:
- Test TimescaleDB continuous aggregates
- Verify data retention policies
- Test concurrent access scenarios
- Validate encryption at rest

**External Service Integration Tests**:
- Mock AI API responses
- Mock FCM notification delivery
- Test fallback behaviors
- Verify retry logic

### Performance Testing

**Load Testing**:
- Dashboard metrics under high query load
- Order creation throughput
- Notification delivery at scale
- Forecast generation performance

**Stress Testing**:
- Database connection pool exhaustion
- Redis cache eviction under memory pressure
- API rate limiting effectiveness
- WebSocket connection limits

### End-to-End Testing

**Critical User Flows**:
1. Customer scans QR → views menu → places order → order recorded
2. Chef take order → prepair food → Update status
2.1 waiter see status update → take plat → surve to table 
2.2 waiter records wastage → pattern identified → alert generated
3. Owner creates notification → customers receive → metrics tracked
4. Manager views dashboard → filters by date → sees updated metrics
5. Forecast generated → inventory recommendations created → alerts sent

**Testing Tools**:
- E2E Framework: Playwright or Cypress
- API Testing: Supertest, postman
- Load Testing: k6 or Artillery
- Database Testing: Testcontainers for isolated environments


