# وثيقة التصميم - منصة Stackly

## نظرة عامة

هذه الوثيقة تحتوي على التصميم المعماري الكامل لمنصة Stackly، بما في ذلك:
- التصميم عالي المستوى (High-Level Design): المعمارية العامة، المكونات الرئيسية، التدفقات
- التصميم منخفض المستوى (Low-Level Design): التفاصيل التقنية، الخوارزميات، واجهات API

---

# الجزء الأول: التصميم عالي المستوى (High-Level Design)

## 1. المعمارية العامة (System Architecture)

### 1.1 نظرة عامة

منصة Stackly تتبع معمارية Microservices مع Monorepo، حيث يتم تقسيم النظام إلى خدمات مستقلة تتواصل عبر Event Bus و API Gateway.

```
┌─────────────────────────────────────────────────────────────────┐
│                         User Layer                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │   Browser    │  │  Mobile App  │  │   CLI Tool   │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      Gateway Layer                               │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  API Gateway (Gateway API + Cilium)                      │   │
│  │  - Authentication                                         │   │
│  │  - Rate Limiting                                          │   │
│  │  - Load Balancing                                         │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Application Layer                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐       │
│  │Dashboard │  │ Editor   │  │Core API  │  │ Workers  │       │
│  │(Next.js) │  │(React)   │  │(Node.js) │  │  (Go)    │       │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     Service Layer                                │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐  │
│  │Identity │ │Template │ │Deploy   │ │Email    │ │Payment  │  │
│  │Service  │ │Service  │ │Service  │ │Service  │ │Service  │  │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘  │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐              │
│  │Domain   │ │Storage  │ │Analytics│ │Automation│             │
│  │Service  │ │Service  │ │Service  │ │Service  │              │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘              │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                  Infrastructure Layer                           │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐         │
│  │PostgreSQL│  │  Valkey  │  │  NATS    │  │ Temporal │         │
│  │    18    │  │  (Cache) │  │(Events)  │  │(Workflow)│         │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘         │
│  ┌──────────┐  ┌──────────┐   ┌──────────┐  ┌──────────┐        │
│  │  MinIO   │  │ClickHouse│   │ ZITADEL  │  │Postfix/  │        │
│  │(Storage) │  │(Analytics)│  │  (Auth)  │  │Dovecot   │        │
│  └──────────┘  └──────────┘   └──────────┘  └──────────┘        │
└─────────────────────────────────────────────────────────────────┘
```


### 1.2 الطبقات الست (Six Layers)

#### Layer 1: Identity & Organization Layer
```
┌─────────────────────────────────────────────────────────────┐
│  Identity & Organization Layer                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐     │
│  │  Users   │  │Workspaces│  │ Projects │  │  Roles   │     │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘     │
│                                                             │
│  - User Management                                          │
│  - Workspace Management                                     │
│  - Project Management                                       │
│  - Role-Based Access Control (RBAC)                         │
│  - Team Collaboration                                       │
└─────────────────────────────────────────────────────────────┘
```

#### Layer 2: Template Editor Layer
```
┌─────────────────────────────────────────────────────────────┐
│  Template Editor Layer                                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │ Visual   │  │Components│  │  Blocks  │  │  Pages   │   │
│  │ Editor   │  │          │  │          │  │          │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                 │
│  │  Theme   │  │ Variants │  │  Slots   │                 │
│  └──────────┘  └──────────┘  └──────────┘                 │
│                                                             │
│  - Drag & Drop Interface                                    │
│  - Live Preview                                             │
│  - Component Registry                                       │
│  - Variant System                                           │
│  - Slot System                                              │
└─────────────────────────────────────────────────────────────┘
```

#### Layer 3: Application Modeling Layer
```
┌─────────────────────────────────────────────────────────────┐
│  Application Modeling Layer                                 │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │   Data   │  │  Forms   │  │Workflows │  │Integration│  │
│  │  Models  │  │          │  │          │  │           │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
│                                                             │
│  - Entity Definition                                        │
│  - Relationship Mapping                                     │
│  - Form Builder                                             │
│  - Workflow Engine                                          │
│  - API Integration                                          │
└─────────────────────────────────────────────────────────────┘
```

#### Layer 4: Services Layer
```
┌─────────────────────────────────────────────────────────────┐
│  Services Layer                                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │  Email   │  │ Mailbox  │  │  Domain  │  │ Payment  │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │Automation│  │Analytics │  │ Storage  │  │    AI    │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
│                                                             │
│  - Email Sending & Receiving                                │
│  - Domain & DNS Management                                  │
│  - Payment Processing                                       │
│  - Workflow Automation                                      │
│  - Analytics & Reporting                                    │
│  - File Storage                                             │
│  - AI Features                                              │
└─────────────────────────────────────────────────────────────┘
```


#### Layer 5: Build & Deployment Layer
```
┌─────────────────────────────────────────────────────────────┐
│  Build & Deployment Layer                                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │  Build   │  │ Preview  │  │  Deploy  │  │ Runtime  │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                 │
│  │   CDN    │  │   SSL    │  │   DNS    │                 │
│  └──────────┘  └──────────┘  └──────────┘                 │
│                                                             │
│  - Build Pipeline                                           │
│  - Preview Environments                                     │
│  - Deployment Strategies                                    │
│  - Runtime Management                                       │
│  - CDN Distribution                                         │
│  - SSL Certificate Management                               │
│  - DNS Configuration                                        │
└─────────────────────────────────────────────────────────────┘
```

#### Layer 6: Infrastructure Layer
```
┌─────────────────────────────────────────────────────────────┐
│  Infrastructure Layer                                       │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │ Database │  │ Storage  │  │  Queue   │  │  Cache   │   │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                 │
│  │Observ-   │  │ Compute  │  │ Network  │                 │
│  │ability   │  │          │  │          │                 │
│  └──────────┘  └──────────┘  └──────────┘                 │
│                                                             │
│  - PostgreSQL (Platform DB + App Data DB)                   │
│  - MinIO/S3 (Object Storage)                                │
│  - NATS JetStream (Event Bus)                               │
│  - Valkey (Cache & Session)                                 │
│  - OpenTelemetry (Observability)                            │
│  - Docker/Kubernetes (Compute)                              │
│  - Gateway API + Cilium (Network)                           │
└─────────────────────────────────────────────────────────────┘
```

## 2. المكونات الرئيسية (Core Components)

### 2.1 Dashboard Application (Next.js)

**المسؤوليات:**
- واجهة المستخدم الرئيسية
- إدارة المشاريع والمساحات
- لوحة التحكم والإحصائيات
- إعدادات المستخدم والفريق

**التقنيات:**
- Next.js 15 App Router
- React 19
- TypeScript 5.7
- Tailwind CSS + shadcn/ui
- React Query (TanStack Query)
- Zustand (State Management)

**الصفحات الرئيسية:**
```
/                          → Landing Page
/login                     → Login Page
/register                  → Register Page
/dashboard                 → Main Dashboard
/dashboard/workspaces      → Workspaces List
/dashboard/projects        → Projects List
/dashboard/projects/:id    → Project Details
/dashboard/settings        → User Settings
/dashboard/billing         → Billing & Plans
```


### 2.2 Visual Editor (React)

**المسؤوليات:**
- محرر مرئي للقوالب
- السحب والإفلات
- المعاينة الفورية
- تخصيص المكونات

**التقنيات:**
- React 19
- TypeScript 5.7
- React Canvas (للمحرر)
- Lexical (Rich Text Editor)
- Monaco Editor (Code Editor)
- React DnD (Drag & Drop)

**المكونات الرئيسية:**
```
EditorCanvas          → لوحة الرسم الرئيسية
ComponentPanel        → لوحة المكونات
PropertiesPanel       → لوحة الخصائص
LayersPanel           → لوحة الطبقات
PreviewPanel          → لوحة المعاينة
ToolbarPanel          → شريط الأدوات
```

**تدفق العمل:**
```
1. User selects template
2. Editor loads template structure
3. User drags component to slot
4. Editor validates slot rules
5. User customizes component properties
6. Editor updates live preview
7. User saves changes
8. Editor syncs to backend
```

### 2.3 Core API (Node.js + Fastify)

**المسؤوليات:**
- API الرئيسي للمنصة
- إدارة المستخدمين والمشاريع
- إدارة القوالب والمكونات
- التواصل مع الخدمات الأخرى

**التقنيات:**
- Node.js 24 LTS
- TypeScript 5.7
- Fastify (Web Framework)
- Prisma (ORM)
- Zod (Validation)
- JWT (Authentication)

**API Endpoints:**
```
# Authentication
POST   /api/auth/register
POST   /api/auth/login
POST   /api/auth/logout
POST   /api/auth/refresh
GET    /api/auth/me

# Workspaces
GET    /api/workspaces
POST   /api/workspaces
GET    /api/workspaces/:id
PATCH  /api/workspaces/:id
DELETE /api/workspaces/:id

# Projects
GET    /api/projects
POST   /api/projects
GET    /api/projects/:id
PATCH  /api/projects/:id
DELETE /api/projects/:id

# Templates
GET    /api/templates
GET    /api/templates/:id
GET    /api/templates/:id/preview

# Components
GET    /api/components
GET    /api/components/:id
GET    /api/components/:id/variants

# Deployments
POST   /api/projects/:id/deploy
GET    /api/projects/:id/deployments
GET    /api/deployments/:id
POST   /api/deployments/:id/rollback
```


### 2.4 Workers (Go)

**المسؤوليات:**
- عمليات البناء والنشر
- معالجة الصور
- إرسال البريد الجماعي
- المهام الثقيلة

**التقنيات:**
- Go 1.26
- PostgreSQL Driver
- NATS Client
- Temporal SDK
- MinIO Client

**Workers:**
```
BuildWorker           → بناء المشاريع
DeployWorker          → نشر المشاريع
ImageWorker           → معالجة الصور
EmailWorker           → إرسال البريد
AnalyticsWorker       → معالجة التحليلات
BackupWorker          → النسخ الاحتياطي
```

## 3. قاعدة البيانات (Database Design)

### 3.1 Platform Database

**الجداول الرئيسية:**

```sql
-- Users Table
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  name VARCHAR(255) NOT NULL,
  avatar_url TEXT,
  email_verified BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Workspaces Table
CREATE TABLE workspaces (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  slug VARCHAR(255) UNIQUE NOT NULL,
  owner_id UUID REFERENCES users(id) ON DELETE CASCADE,
  plan_id UUID REFERENCES plans(id),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Workspace Members Table
CREATE TABLE workspace_members (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  workspace_id UUID REFERENCES workspaces(id) ON DELETE CASCADE,
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  role VARCHAR(50) NOT NULL, -- owner, admin, member, viewer
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(workspace_id, user_id)
);

-- Projects Table
CREATE TABLE projects (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  workspace_id UUID REFERENCES workspaces(id) ON DELETE CASCADE,
  name VARCHAR(255) NOT NULL,
  slug VARCHAR(255) NOT NULL,
  template_id UUID REFERENCES templates(id),
  status VARCHAR(50) DEFAULT 'draft', -- draft, building, deployed, archived
  config JSONB NOT NULL DEFAULT '{}',
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(workspace_id, slug)
);

-- Environments Table
CREATE TABLE environments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id UUID REFERENCES projects(id) ON DELETE CASCADE,
  name VARCHAR(50) NOT NULL, -- dev, preview, production
  url TEXT,
  status VARCHAR(50) DEFAULT 'inactive', -- active, inactive, building
  config JSONB NOT NULL DEFAULT '{}',
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(project_id, name)
);
```


```sql
-- Templates Table
CREATE TABLE templates (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  slug VARCHAR(255) UNIQUE NOT NULL,
  category VARCHAR(100) NOT NULL,
  description TEXT,
  thumbnail_url TEXT,
  metadata JSONB NOT NULL DEFAULT '{}',
  ui_schema JSONB NOT NULL DEFAULT '{}',
  theme_schema JSONB NOT NULL DEFAULT '{}',
  data_model_schema JSONB NOT NULL DEFAULT '{}',
  forms_schema JSONB NOT NULL DEFAULT '{}',
  workflow_schema JSONB NOT NULL DEFAULT '{}',
  service_requirements JSONB NOT NULL DEFAULT '{}',
  deployment_preset JSONB NOT NULL DEFAULT '{}',
  version VARCHAR(50) NOT NULL DEFAULT '1.0.0',
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Blocks Table
CREATE TABLE blocks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  slug VARCHAR(255) UNIQUE NOT NULL,
  category VARCHAR(100) NOT NULL,
  description TEXT,
  thumbnail_url TEXT,
  schema JSONB NOT NULL DEFAULT '{}',
  slots JSONB NOT NULL DEFAULT '[]',
  version VARCHAR(50) NOT NULL DEFAULT '1.0.0',
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Components Table
CREATE TABLE components (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  slug VARCHAR(255) UNIQUE NOT NULL,
  category VARCHAR(100) NOT NULL,
  description TEXT,
  thumbnail_url TEXT,
  schema JSONB NOT NULL DEFAULT '{}',
  props_schema JSONB NOT NULL DEFAULT '{}',
  version VARCHAR(50) NOT NULL DEFAULT '1.0.0',
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Variants Table
CREATE TABLE variants (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  element_type VARCHAR(50) NOT NULL, -- template, block, component
  element_id UUID NOT NULL,
  name VARCHAR(255) NOT NULL,
  slug VARCHAR(255) NOT NULL,
  description TEXT,
  thumbnail_url TEXT,
  schema JSONB NOT NULL DEFAULT '{}',
  is_default BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Deployments Table
CREATE TABLE deployments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  environment_id UUID REFERENCES environments(id) ON DELETE CASCADE,
  version VARCHAR(50) NOT NULL,
  status VARCHAR(50) DEFAULT 'pending', -- pending, building, deploying, deployed, failed, rolled_back
  build_log TEXT,
  deploy_log TEXT,
  started_at TIMESTAMP,
  completed_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Plans Table
CREATE TABLE plans (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(100) NOT NULL,
  slug VARCHAR(100) UNIQUE NOT NULL,
  price_monthly DECIMAL(10, 2) NOT NULL,
  price_yearly DECIMAL(10, 2) NOT NULL,
  limits JSONB NOT NULL DEFAULT '{}',
  features JSONB NOT NULL DEFAULT '[]',
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```


### 3.2 App Data Database

**مستويات التوفير:**

**Level 1: Static-Only**
- لا توجد قاعدة بيانات
- ملفات ثابتة فقط

**Level 2: Shared Schema**
```sql
-- Products Table (Shared)
CREATE TABLE products (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL, -- Project ID
  name VARCHAR(255) NOT NULL,
  description TEXT,
  price DECIMAL(10, 2) NOT NULL,
  stock INTEGER DEFAULT 0,
  category_id UUID,
  images JSONB DEFAULT '[]',
  metadata JSONB DEFAULT '{}',
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_products_tenant ON products(tenant_id);

-- Orders Table (Shared)
CREATE TABLE orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL,
  user_id UUID,
  status VARCHAR(50) DEFAULT 'pending',
  total DECIMAL(10, 2) NOT NULL,
  items JSONB NOT NULL DEFAULT '[]',
  shipping_address JSONB,
  billing_address JSONB,
  metadata JSONB DEFAULT '{}',
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_orders_tenant ON orders(tenant_id);
```

**Level 3: Dedicated Schema**
```sql
-- Each project gets its own schema
CREATE SCHEMA project_abc123;

-- Products Table (Dedicated)
CREATE TABLE project_abc123.products (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  description TEXT,
  price DECIMAL(10, 2) NOT NULL,
  stock INTEGER DEFAULT 0,
  category_id UUID,
  images JSONB DEFAULT '[]',
  metadata JSONB DEFAULT '{}',
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

**Level 4: Isolated DB**
- قاعدة بيانات PostgreSQL منفصلة لكل مشروع
- نفس البنية ولكن في قاعدة بيانات مستقلة

## 4. التدفقات الرئيسية (Main Flows)

### 4.1 تدفق إنشاء مشروع جديد

```
┌──────────┐
│  User    │
└────┬─────┘
     │
     │ 1. Click "New Project"
     ▼
┌──────────────────┐
│   Dashboard      │
│   (Next.js)      │
└────┬─────────────┘
     │
     │ 2. GET /api/templates
     ▼
┌──────────────────┐
│   Core API       │
│   (Node.js)      │
└────┬─────────────┘
     │
     │ 3. Query templates
     ▼
┌──────────────────┐
│   PostgreSQL     │
│   (Platform DB)  │
└────┬─────────────┘
     │
     │ 4. Return templates
     ▼
┌──────────────────┐
│   Dashboard      │
│   Show templates │
└────┬─────────────┘
     │
     │ 5. User selects template
     │ 6. POST /api/projects
     ▼
┌──────────────────┐
│   Core API       │
│   Validate &     │
│   Create Project │
└────┬─────────────┘
     │
     │ 7. Insert project
     ▼
┌──────────────────┐
│   PostgreSQL     │
└────┬─────────────┘
     │
     │ 8. Publish event
     ▼
┌──────────────────┐
│   NATS           │
│   project.created│
└────┬─────────────┘
     │
     │ 9. Subscribe
     ▼
┌──────────────────┐
│   Provisioning   │
│   Service        │
│   Analyze &      │
│   Provision      │
└────┬─────────────┘
     │
     │ 10. Create resources
     ▼
┌──────────────────┐
│   Infrastructure │
│   - DB Schema    │
│   - Storage      │
│   - Services     │
└──────────────────┘
```


### 4.2 تدفق التحرير والمعاينة

```
┌──────────┐
│  User    │
└────┬─────┘
     │
     │ 1. Open Editor
     ▼
┌──────────────────┐
│   Editor         │
│   (React)        │
└────┬─────────────┘
     │
     │ 2. GET /api/projects/:id
     ▼
┌──────────────────┐
│   Core API       │
└────┬─────────────┘
     │
     │ 3. Load project data
     ▼
┌──────────────────┐
│   PostgreSQL     │
└────┬─────────────┘
     │
     │ 4. Return project + template
     ▼
┌──────────────────┐
│   Editor         │
│   Render Canvas  │
└────┬─────────────┘
     │
     │ 5. User drags component
     │ 6. Validate slot rules
     │ 7. Update canvas
     │ 8. Update live preview
     │
     │ 9. User saves (debounced)
     │ 10. PATCH /api/projects/:id
     ▼
┌──────────────────┐
│   Core API       │
│   Update config  │
└────┬─────────────┘
     │
     │ 11. Update project
     ▼
┌──────────────────┐
│   PostgreSQL     │
└────┬─────────────┘
     │
     │ 12. Publish event
     ▼
┌──────────────────┐
│   NATS           │
│   project.updated│
└────┬─────────────┘
     │
     │ 13. Subscribe
     ▼
┌──────────────────┐
│   Preview        │
│   Service        │
│   Rebuild preview│
└──────────────────┘
```

### 4.3 تدفق النشر

```
┌──────────┐
│  User    │
└────┬─────┘
     │
     │ 1. Click "Deploy"
     ▼
┌──────────────────┐
│   Dashboard      │
└────┬─────────────┘
     │
     │ 2. POST /api/projects/:id/deploy
     ▼
┌──────────────────┐
│   Core API       │
│   Validate       │
└────┬─────────────┘
     │
     │ 3. Check plan limits
     ▼
┌──────────────────┐
│   PostgreSQL     │
└────┬─────────────┘
     │
     │ 4. Create deployment record
     │ 5. Publish event
     ▼
┌──────────────────┐
│   NATS           │
│   deployment.    │
│   requested      │
└────┬─────────────┘
     │
     │ 6. Subscribe
     ▼
┌──────────────────┐
│   Temporal       │
│   Start workflow │
└────┬─────────────┘
     │
     │ 7. Execute activities
     ▼
┌──────────────────────────────────────┐
│   Deployment Workflow                │
│                                      │
│   Activity 1: Build                  │
│   ┌────────────────────────────┐    │
│   │ BuildWorker (Go)           │    │
│   │ - Fetch project config     │    │
│   │ - Generate code            │    │
│   │ - Build assets             │    │
│   │ - Optimize images          │    │
│   │ - Bundle JS/CSS            │    │
│   └────────────────────────────┘    │
│                                      │
│   Activity 2: Provision              │
│   ┌────────────────────────────┐    │
│   │ ProvisioningService        │    │
│   │ - Analyze requirements     │    │
│   │ - Create DB schema         │    │
│   │ - Allocate storage         │    │
│   │ - Configure services       │    │
│   └────────────────────────────┘    │
│                                      │
│   Activity 3: Deploy                 │
│   ┌────────────────────────────┐    │
│   │ DeployWorker (Go)          │    │
│   │ - Upload to CDN            │    │
│   │ - Configure DNS            │    │
│   │ - Generate SSL cert        │    │
│   │ - Update routing           │    │
│   └────────────────────────────┘    │
│                                      │
│   Activity 4: Verify                 │
│   ┌────────────────────────────┐    │
│   │ VerificationService        │    │
│   │ - Health check             │    │
│   │ - Smoke tests              │    │
│   │ - Update status            │    │
│   └────────────────────────────┘    │
└──────────────────────────────────────┘
     │
     │ 8. Workflow complete
     │ 9. Publish event
     ▼
┌──────────────────┐
│   NATS           │
│   deployment.    │
│   completed      │
└────┬─────────────┘
     │
     │ 10. Update UI
     ▼
┌──────────────────┐
│   Dashboard      │
│   Show success   │
└──────────────────┘
```


---

# الجزء الثاني: التصميم منخفض المستوى (Low-Level Design)

## 5. التفاصيل التقنية (Technical Details)

### 5.1 Registry System Implementation

**Data Structure:**
```typescript
// Registry Entry Interface
interface RegistryEntry {
  id: string;
  type: 'template' | 'block' | 'component' | 'variant' | 'addon';
  name: string;
  slug: string;
  category: string;
  description: string;
  thumbnailUrl: string;
  schema: Record<string, any>;
  metadata: {
    version: string;
    author: string;
    tags: string[];
    dependencies: string[];
  };
  isActive: boolean;
  createdAt: Date;
  updatedAt: Date;
}

// Registry Service
class RegistryService {
  private cache: Map<string, RegistryEntry>;
  
  constructor(
    private db: PrismaClient,
    private redis: Redis
  ) {
    this.cache = new Map();
  }
  
  async getEntry(id: string): Promise<RegistryEntry | null> {
    // Check memory cache first
    if (this.cache.has(id)) {
      return this.cache.get(id)!;
    }
    
    // Check Redis cache
    const cached = await this.redis.get(`registry:${id}`);
    if (cached) {
      const entry = JSON.parse(cached);
      this.cache.set(id, entry);
      return entry;
    }
    
    // Query database
    const entry = await this.db.registryEntry.findUnique({
      where: { id }
    });
    
    if (entry) {
      // Cache in Redis (1 hour TTL)
      await this.redis.setex(
        `registry:${id}`,
        3600,
        JSON.stringify(entry)
      );
      this.cache.set(id, entry);
    }
    
    return entry;
  }
  
  async search(query: RegistrySearchQuery): Promise<RegistryEntry[]> {
    const { type, category, tags, search } = query;
    
    // Build search query
    const where: any = { isActive: true };
    
    if (type) where.type = type;
    if (category) where.category = category;
    if (tags?.length) {
      where.metadata = {
        path: ['tags'],
        array_contains: tags
      };
    }
    if (search) {
      where.OR = [
        { name: { contains: search, mode: 'insensitive' } },
        { description: { contains: search, mode: 'insensitive' } }
      ];
    }
    
    return this.db.registryEntry.findMany({
      where,
      orderBy: { createdAt: 'desc' }
    });
  }
}
```


### 5.2 Slot System Implementation

**Data Structure:**
```typescript
// Slot Definition
interface SlotDefinition {
  id: string;
  name: string;
  allowedTypes: ('block' | 'component')[];
  allowedElements: string[]; // IDs or slugs
  minItems: number;
  maxItems: number;
  rules: SlotRule[];
}

interface SlotRule {
  type: 'order' | 'dependency' | 'exclusion' | 'required';
  config: Record<string, any>;
}

// Slot Validator
class SlotValidator {
  validate(
    slot: SlotDefinition,
    elements: RegistryEntry[]
  ): ValidationResult {
    const errors: string[] = [];
    
    // Check count
    if (elements.length < slot.minItems) {
      errors.push(`Minimum ${slot.minItems} items required`);
    }
    if (elements.length > slot.maxItems) {
      errors.push(`Maximum ${slot.maxItems} items allowed`);
    }
    
    // Check types
    for (const element of elements) {
      if (!slot.allowedTypes.includes(element.type as any)) {
        errors.push(`Type ${element.type} not allowed in this slot`);
      }
      
      if (
        slot.allowedElements.length > 0 &&
        !slot.allowedElements.includes(element.id) &&
        !slot.allowedElements.includes(element.slug)
      ) {
        errors.push(`Element ${element.name} not allowed in this slot`);
      }
    }
    
    // Check rules
    for (const rule of slot.rules) {
      const ruleErrors = this.validateRule(rule, elements);
      errors.push(...ruleErrors);
    }
    
    return {
      valid: errors.length === 0,
      errors
    };
  }
  
  private validateRule(
    rule: SlotRule,
    elements: RegistryEntry[]
  ): string[] {
    switch (rule.type) {
      case 'order':
        return this.validateOrder(rule.config, elements);
      case 'dependency':
        return this.validateDependency(rule.config, elements);
      case 'exclusion':
        return this.validateExclusion(rule.config, elements);
      case 'required':
        return this.validateRequired(rule.config, elements);
      default:
        return [];
    }
  }
  
  private validateOrder(
    config: any,
    elements: RegistryEntry[]
  ): string[] {
    // Validate element order
    const { order } = config;
    const errors: string[] = [];
    
    for (let i = 0; i < order.length; i++) {
      const expectedId = order[i];
      const actualId = elements[i]?.id;
      
      if (expectedId !== actualId) {
        errors.push(`Element at position ${i} should be ${expectedId}`);
      }
    }
    
    return errors;
  }
  
  private validateDependency(
    config: any,
    elements: RegistryEntry[]
  ): string[] {
    // Validate dependencies between elements
    const { dependencies } = config;
    const errors: string[] = [];
    const elementIds = elements.map(e => e.id);
    
    for (const [elementId, requiredIds] of Object.entries(dependencies)) {
      if (elementIds.includes(elementId)) {
        for (const requiredId of requiredIds as string[]) {
          if (!elementIds.includes(requiredId)) {
            errors.push(
              `Element ${elementId} requires ${requiredId}`
            );
          }
        }
      }
    }
    
    return errors;
  }
}
```


### 5.3 Variant Engine Implementation

**Data Structure:**
```typescript
// Variant Definition
interface Variant {
  id: string;
  elementType: 'template' | 'block' | 'component';
  elementId: string;
  name: string;
  slug: string;
  description: string;
  thumbnailUrl: string;
  schema: Record<string, any>;
  isDefault: boolean;
}

// Variant Engine
class VariantEngine {
  constructor(
    private db: PrismaClient,
    private registry: RegistryService
  ) {}
  
  async getVariants(
    elementType: string,
    elementId: string
  ): Promise<Variant[]> {
    return this.db.variant.findMany({
      where: {
        elementType,
        elementId
      },
      orderBy: [
        { isDefault: 'desc' },
        { name: 'asc' }
      ]
    });
  }
  
  async applyVariant(
    projectId: string,
    elementPath: string,
    variantId: string
  ): Promise<void> {
    // Get project config
    const project = await this.db.project.findUnique({
      where: { id: projectId }
    });
    
    if (!project) {
      throw new Error('Project not found');
    }
    
    // Get variant
    const variant = await this.db.variant.findUnique({
      where: { id: variantId }
    });
    
    if (!variant) {
      throw new Error('Variant not found');
    }
    
    // Update project config
    const config = project.config as any;
    const element = this.getElementByPath(config, elementPath);
    
    if (!element) {
      throw new Error('Element not found');
    }
    
    // Preserve data while changing variant
    const preservedData = this.extractPreservableData(
      element,
      variant.schema
    );
    
    // Apply variant schema
    Object.assign(element, variant.schema, preservedData);
    element.variantId = variantId;
    
    // Save project
    await this.db.project.update({
      where: { id: projectId },
      data: { config }
    });
  }
  
  private getElementByPath(
    config: any,
    path: string
  ): any {
    const parts = path.split('.');
    let current = config;
    
    for (const part of parts) {
      if (current[part] === undefined) {
        return null;
      }
      current = current[part];
    }
    
    return current;
  }
  
  private extractPreservableData(
    element: any,
    newSchema: any
  ): Record<string, any> {
    const preserved: Record<string, any> = {};
    
    // Preserve common fields
    const preservableFields = [
      'id',
      'title',
      'description',
      'content',
      'images',
      'links',
      'customData'
    ];
    
    for (const field of preservableFields) {
      if (element[field] !== undefined && newSchema[field] !== undefined) {
        preserved[field] = element[field];
      }
    }
    
    return preserved;
  }
}
```


### 5.4 Capability Rules Engine Implementation

**Data Structure:**
```typescript
// Plan Definition
interface Plan {
  id: string;
  name: string;
  slug: string;
  priceMonthly: number;
  priceYearly: number;
  limits: PlanLimits;
  features: string[];
}

interface PlanLimits {
  pages: number | 'unlimited';
  templates: number | 'unlimited';
  blocks: number | 'unlimited';
  domains: number | 'unlimited';
  storage: string; // e.g., "1GB", "10GB"
  bandwidth: string; // e.g., "10GB", "100GB"
  products: number | 'unlimited';
  orders: number | 'unlimited';
  emails: number | 'unlimited';
  mailboxes: number | 'unlimited';
  database: 'shared-schema' | 'dedicated-schema' | 'isolated-db';
}

// Capability Rules Engine
class CapabilityRulesEngine {
  constructor(
    private db: PrismaClient
  ) {}
  
  async checkCapability(
    workspaceId: string,
    capability: string,
    requestedAmount: number = 1
  ): Promise<CapabilityCheckResult> {
    // Get workspace with plan
    const workspace = await this.db.workspace.findUnique({
      where: { id: workspaceId },
      include: { plan: true }
    });
    
    if (!workspace || !workspace.plan) {
      return {
        allowed: false,
        reason: 'No active plan'
      };
    }
    
    const plan = workspace.plan;
    const limit = plan.limits[capability];
    
    // Check if unlimited
    if (limit === 'unlimited') {
      return { allowed: true };
    }
    
    // Get current usage
    const usage = await this.getCurrentUsage(workspaceId, capability);
    
    // Check if within limits
    const newUsage = usage + requestedAmount;
    if (newUsage <= limit) {
      return {
        allowed: true,
        usage,
        limit,
        remaining: limit - newUsage
      };
    }
    
    return {
      allowed: false,
      reason: `Limit exceeded: ${usage}/${limit}`,
      usage,
      limit,
      remaining: 0
    };
  }
  
  private async getCurrentUsage(
    workspaceId: string,
    capability: string
  ): Promise<number> {
    switch (capability) {
      case 'pages':
        return this.db.page.count({
          where: {
            project: {
              workspaceId
            }
          }
        });
      
      case 'projects':
        return this.db.project.count({
          where: { workspaceId }
        });
      
      case 'domains':
        return this.db.domain.count({
          where: {
            project: {
              workspaceId
            }
          }
        });
      
      case 'storage':
        // Calculate total storage used
        const result = await this.db.$queryRaw`
          SELECT SUM(size) as total
          FROM files
          WHERE workspace_id = ${workspaceId}
        `;
        return result[0]?.total || 0;
      
      default:
        return 0;
    }
  }
  
  async enforceCapability(
    workspaceId: string,
    capability: string,
    requestedAmount: number = 1
  ): Promise<void> {
    const check = await this.checkCapability(
      workspaceId,
      capability,
      requestedAmount
    );
    
    if (!check.allowed) {
      throw new CapabilityError(
        check.reason || 'Capability not allowed',
        {
          capability,
          usage: check.usage,
          limit: check.limit
        }
      );
    }
  }
}
```


### 5.5 Provisioning Decision Engine Implementation

**Data Structure:**
```typescript
// Provisioning Requirements
interface ProvisioningRequirements {
  database: 'none' | 'shared-schema' | 'dedicated-schema' | 'isolated-db';
  storage: number; // bytes
  bandwidth: number; // bytes/month
  services: {
    email: boolean;
    mailbox: boolean;
    domain: boolean;
    payment: boolean;
    automation: boolean;
    analytics: boolean;
    ai: boolean;
  };
  compute: {
    cpu: number; // cores
    memory: number; // MB
  };
}

// Provisioning Decision Engine
class ProvisioningDecisionEngine {
  constructor(
    private db: PrismaClient,
    private capabilityEngine: CapabilityRulesEngine
  ) {}
  
  async analyzeRequirements(
    projectId: string
  ): Promise<ProvisioningRequirements> {
    // Get project with template
    const project = await this.db.project.findUnique({
      where: { id: projectId },
      include: {
        template: true,
        workspace: {
          include: { plan: true }
        }
      }
    });
    
    if (!project) {
      throw new Error('Project not found');
    }
    
    const template = project.template;
    const plan = project.workspace.plan;
    
    // Analyze template requirements
    const templateReqs = template.serviceRequirements as any;
    
    // Determine database level
    const databaseLevel = this.determineDatabaseLevel(
      templateReqs,
      plan.limits
    );
    
    // Calculate storage needs
    const storage = this.calculateStorageNeeds(
      project.config as any,
      templateReqs
    );
    
    // Calculate bandwidth needs
    const bandwidth = this.calculateBandwidthNeeds(
      project.config as any,
      templateReqs
    );
    
    // Determine required services
    const services = this.determineServices(templateReqs);
    
    // Calculate compute needs
    const compute = this.calculateComputeNeeds(
      databaseLevel,
      services
    );
    
    return {
      database: databaseLevel,
      storage,
      bandwidth,
      services,
      compute
    };
  }
  
  private determineDatabaseLevel(
    templateReqs: any,
    planLimits: PlanLimits
  ): ProvisioningRequirements['database'] {
    // Check if database is needed
    if (!templateReqs.database || templateReqs.database === 'none') {
      return 'none';
    }
    
    // Use plan's database level
    return planLimits.database;
  }
  
  private calculateStorageNeeds(
    config: any,
    templateReqs: any
  ): number {
    let storage = 0;
    
    // Base storage for template
    storage += templateReqs.storage?.base || 100 * 1024 * 1024; // 100MB
    
    // Storage per image
    const imageCount = this.countImages(config);
    storage += imageCount * 2 * 1024 * 1024; // 2MB per image
    
    // Storage per product
    const productCount = config.products?.length || 0;
    storage += productCount * 1 * 1024 * 1024; // 1MB per product
    
    return storage;
  }
  
  private calculateBandwidthNeeds(
    config: any,
    templateReqs: any
  ): number {
    // Estimate based on page count and expected traffic
    const pageCount = config.pages?.length || 1;
    const avgPageSize = 2 * 1024 * 1024; // 2MB
    const estimatedVisits = 1000; // per month
    
    return pageCount * avgPageSize * estimatedVisits;
  }
  
  private determineServices(
    templateReqs: any
  ): ProvisioningRequirements['services'] {
    return {
      email: templateReqs.email !== false,
      mailbox: templateReqs.mailbox === true,
      domain: templateReqs.domain !== false,
      payment: templateReqs.payment === true,
      automation: templateReqs.automation === true,
      analytics: templateReqs.analytics !== false,
      ai: templateReqs.ai === true
    };
  }
  
  private calculateComputeNeeds(
    databaseLevel: string,
    services: ProvisioningRequirements['services']
  ): ProvisioningRequirements['compute'] {
    let cpu = 0.5; // 0.5 cores base
    let memory = 512; // 512MB base
    
    // Add for database
    if (databaseLevel === 'dedicated-schema') {
      cpu += 0.5;
      memory += 512;
    } else if (databaseLevel === 'isolated-db') {
      cpu += 2;
      memory += 2048;
    }
    
    // Add for services
    const activeServices = Object.values(services).filter(Boolean).length;
    cpu += activeServices * 0.25;
    memory += activeServices * 256;
    
    return { cpu, memory };
  }
  
  async provision(
    projectId: string,
    requirements: ProvisioningRequirements
  ): Promise<void> {
    // Create database resources
    if (requirements.database !== 'none') {
      await this.provisionDatabase(projectId, requirements.database);
    }
    
    // Allocate storage
    await this.provisionStorage(projectId, requirements.storage);
    
    // Configure services
    for (const [service, enabled] of Object.entries(requirements.services)) {
      if (enabled) {
        await this.provisionService(projectId, service);
      }
    }
    
    // Allocate compute resources
    await this.provisionCompute(projectId, requirements.compute);
  }
}
```


### 5.6 Build & Deployment Pipeline

**Build Worker (Go):**
```go
package workers

import (
    "context"
    "encoding/json"
    "fmt"
    "os"
    "os/exec"
    "path/filepath"
)

type BuildWorker struct {
    db        *sql.DB
    storage   *minio.Client
    nats      *nats.Conn
}

type BuildRequest struct {
    ProjectID    string                 `json:"project_id"`
    EnvironmentID string                `json:"environment_id"`
    Config       map[string]interface{} `json:"config"`
}

type BuildResult struct {
    Success   bool     `json:"success"`
    BuildPath string   `json:"build_path"`
    Assets    []string `json:"assets"`
    Errors    []string `json:"errors"`
}

func (w *BuildWorker) Build(ctx context.Context, req BuildRequest) (*BuildResult, error) {
    // Create build directory
    buildDir := filepath.Join("/tmp/builds", req.ProjectID)
    if err := os.MkdirAll(buildDir, 0755); err != nil {
        return nil, fmt.Errorf("failed to create build dir: %w", err)
    }
    defer os.RemoveAll(buildDir)
    
    // Generate project files
    if err := w.generateProjectFiles(buildDir, req.Config); err != nil {
        return nil, fmt.Errorf("failed to generate files: %w", err)
    }
    
    // Install dependencies
    if err := w.installDependencies(buildDir); err != nil {
        return nil, fmt.Errorf("failed to install deps: %w", err)
    }
    
    // Build project
    if err := w.buildProject(buildDir); err != nil {
        return nil, fmt.Errorf("failed to build: %w", err)
    }
    
    // Optimize assets
    if err := w.optimizeAssets(buildDir); err != nil {
        return nil, fmt.Errorf("failed to optimize: %w", err)
    }
    
    // Upload to storage
    assets, err := w.uploadAssets(ctx, buildDir, req.ProjectID)
    if err != nil {
        return nil, fmt.Errorf("failed to upload: %w", err)
    }
    
    return &BuildResult{
        Success:   true,
        BuildPath: buildDir,
        Assets:    assets,
    }, nil
}

func (w *BuildWorker) generateProjectFiles(buildDir string, config map[string]interface{}) error {
    // Generate package.json
    packageJSON := map[string]interface{}{
        "name":    "stackly-project",
        "version": "1.0.0",
        "scripts": map[string]string{
            "build": "next build",
        },
        "dependencies": map[string]string{
            "next":  "15.0.0",
            "react": "19.0.0",
        },
    }
    
    data, err := json.MarshalIndent(packageJSON, "", "  ")
    if err != nil {
        return err
    }
    
    if err := os.WriteFile(
        filepath.Join(buildDir, "package.json"),
        data,
        0644,
    ); err != nil {
        return err
    }
    
    // Generate pages from config
    pages := config["pages"].([]interface{})
    for _, page := range pages {
        pageData := page.(map[string]interface{})
        if err := w.generatePage(buildDir, pageData); err != nil {
            return err
        }
    }
    
    return nil
}

func (w *BuildWorker) generatePage(buildDir string, pageData map[string]interface{}) error {
    pageName := pageData["slug"].(string)
    pageContent := w.renderPageContent(pageData)
    
    pagesDir := filepath.Join(buildDir, "app", pageName)
    if err := os.MkdirAll(pagesDir, 0755); err != nil {
        return err
    }
    
    return os.WriteFile(
        filepath.Join(pagesDir, "page.tsx"),
        []byte(pageContent),
        0644,
    )
}

func (w *BuildWorker) renderPageContent(pageData map[string]interface{}) string {
    // Generate React component from page data
    template := `
import React from 'react';

export default function Page() {
  return (
    <div>
      %s
    </div>
  );
}
`
    
    sections := pageData["sections"].([]interface{})
    content := ""
    
    for _, section := range sections {
        sectionData := section.(map[string]interface{})
        content += w.renderSection(sectionData)
    }
    
    return fmt.Sprintf(template, content)
}

func (w *BuildWorker) installDependencies(buildDir string) error {
    cmd := exec.Command("pnpm", "install")
    cmd.Dir = buildDir
    return cmd.Run()
}

func (w *BuildWorker) buildProject(buildDir string) error {
    cmd := exec.Command("pnpm", "build")
    cmd.Dir = buildDir
    return cmd.Run()
}

func (w *BuildWorker) optimizeAssets(buildDir string) error {
    // Optimize images
    imagesDir := filepath.Join(buildDir, ".next", "static", "images")
    if err := w.optimizeImages(imagesDir); err != nil {
        return err
    }
    
    // Minify CSS/JS (already done by Next.js)
    
    return nil
}

func (w *BuildWorker) uploadAssets(ctx context.Context, buildDir, projectID string) ([]string, error) {
    assets := []string{}
    
    // Upload .next/static directory
    staticDir := filepath.Join(buildDir, ".next", "static")
    
    err := filepath.Walk(staticDir, func(path string, info os.FileInfo, err error) error {
        if err != nil {
            return err
        }
        
        if info.IsDir() {
            return nil
        }
        
        relPath, _ := filepath.Rel(staticDir, path)
        objectName := fmt.Sprintf("projects/%s/static/%s", projectID, relPath)
        
        _, err = w.storage.FPutObject(
            ctx,
            "stackly-builds",
            objectName,
            path,
            minio.PutObjectOptions{},
        )
        
        if err != nil {
            return err
        }
        
        assets = append(assets, objectName)
        return nil
    })
    
    return assets, err
}
```


### 5.7 Deployment Workflow (Temporal)

**Workflow Definition:**
```typescript
// Deployment Workflow
import { proxyActivities, sleep } from '@temporalio/workflow';
import type * as activities from './activities';

const {
  buildProject,
  provisionResources,
  deployToProduction,
  verifyDeployment,
  rollbackDeployment,
  notifyUser
} = proxyActivities<typeof activities>({
  startToCloseTimeout: '10 minutes',
  retry: {
    initialInterval: '1s',
    maximumInterval: '1m',
    maximumAttempts: 3
  }
});

export async function deploymentWorkflow(
  projectId: string,
  environmentId: string
): Promise<DeploymentResult> {
  let buildResult: BuildResult;
  let provisionResult: ProvisionResult;
  let deployResult: DeployResult;
  
  try {
    // Step 1: Build project
    buildResult = await buildProject(projectId, environmentId);
    
    if (!buildResult.success) {
      throw new Error('Build failed');
    }
    
    // Step 2: Provision resources
    provisionResult = await provisionResources(projectId);
    
    if (!provisionResult.success) {
      throw new Error('Provisioning failed');
    }
    
    // Step 3: Deploy to production
    deployResult = await deployToProduction(
      projectId,
      environmentId,
      buildResult.assets
    );
    
    if (!deployResult.success) {
      throw new Error('Deployment failed');
    }
    
    // Step 4: Wait for propagation
    await sleep('30s');
    
    // Step 5: Verify deployment
    const verifyResult = await verifyDeployment(
      projectId,
      environmentId
    );
    
    if (!verifyResult.success) {
      // Rollback on verification failure
      await rollbackDeployment(projectId, environmentId);
      throw new Error('Verification failed, rolled back');
    }
    
    // Step 6: Notify user
    await notifyUser(projectId, {
      type: 'deployment_success',
      url: deployResult.url
    });
    
    return {
      success: true,
      url: deployResult.url,
      version: deployResult.version
    };
    
  } catch (error) {
    // Notify user of failure
    await notifyUser(projectId, {
      type: 'deployment_failed',
      error: error.message
    });
    
    throw error;
  }
}

// Canary Deployment Workflow
export async function canaryDeploymentWorkflow(
  projectId: string,
  environmentId: string
): Promise<DeploymentResult> {
  // Build and deploy to canary
  const canaryResult = await deployToProduction(
    projectId,
    environmentId,
    buildResult.assets,
    { strategy: 'canary', percentage: 10 }
  );
  
  // Monitor for 5 minutes
  await sleep('5m');
  
  // Check metrics
  const metrics = await getDeploymentMetrics(projectId, environmentId);
  
  if (metrics.errorRate > 0.01) {
    // Error rate too high, rollback
    await rollbackDeployment(projectId, environmentId);
    throw new Error('Canary deployment failed');
  }
  
  // Gradually increase traffic
  for (const percentage of [25, 50, 75, 100]) {
    await updateTrafficSplit(projectId, environmentId, percentage);
    await sleep('2m');
    
    const metrics = await getDeploymentMetrics(projectId, environmentId);
    if (metrics.errorRate > 0.01) {
      await rollbackDeployment(projectId, environmentId);
      throw new Error(`Canary deployment failed at ${percentage}%`);
    }
  }
  
  return {
    success: true,
    url: canaryResult.url,
    version: canaryResult.version
  };
}
```


### 5.8 Editor State Management

**State Structure:**
```typescript
// Editor State (Zustand)
interface EditorState {
  // Project data
  project: Project | null;
  template: Template | null;
  
  // Canvas state
  canvas: {
    zoom: number;
    pan: { x: number; y: number };
    selectedElement: string | null;
    hoveredElement: string | null;
    mode: 'select' | 'drag' | 'resize';
  };
  
  // History
  history: {
    past: ProjectConfig[];
    present: ProjectConfig;
    future: ProjectConfig[];
  };
  
  // UI state
  ui: {
    leftPanelOpen: boolean;
    rightPanelOpen: boolean;
    bottomPanelOpen: boolean;
    activeTab: 'components' | 'layers' | 'assets';
  };
  
  // Actions
  loadProject: (projectId: string) => Promise<void>;
  updateElement: (path: string, data: any) => void;
  addElement: (slotPath: string, elementId: string) => void;
  removeElement: (path: string) => void;
  undo: () => void;
  redo: () => void;
  save: () => Promise<void>;
}

// Create store
const useEditorStore = create<EditorState>((set, get) => ({
  project: null,
  template: null,
  
  canvas: {
    zoom: 1,
    pan: { x: 0, y: 0 },
    selectedElement: null,
    hoveredElement: null,
    mode: 'select'
  },
  
  history: {
    past: [],
    present: {},
    future: []
  },
  
  ui: {
    leftPanelOpen: true,
    rightPanelOpen: true,
    bottomPanelOpen: false,
    activeTab: 'components'
  },
  
  loadProject: async (projectId: string) => {
    const response = await fetch(`/api/projects/${projectId}`);
    const project = await response.json();
    
    const templateResponse = await fetch(
      `/api/templates/${project.templateId}`
    );
    const template = await templateResponse.json();
    
    set({
      project,
      template,
      history: {
        past: [],
        present: project.config,
        future: []
      }
    });
  },
  
  updateElement: (path: string, data: any) => {
    const { history } = get();
    const newConfig = { ...history.present };
    
    // Update element at path
    const parts = path.split('.');
    let current = newConfig;
    for (let i = 0; i < parts.length - 1; i++) {
      current = current[parts[i]];
    }
    current[parts[parts.length - 1]] = {
      ...current[parts[parts.length - 1]],
      ...data
    };
    
    // Update history
    set({
      history: {
        past: [...history.past, history.present],
        present: newConfig,
        future: []
      }
    });
    
    // Debounced save
    debouncedSave();
  },
  
  undo: () => {
    const { history } = get();
    if (history.past.length === 0) return;
    
    const previous = history.past[history.past.length - 1];
    const newPast = history.past.slice(0, -1);
    
    set({
      history: {
        past: newPast,
        present: previous,
        future: [history.present, ...history.future]
      }
    });
  },
  
  redo: () => {
    const { history } = get();
    if (history.future.length === 0) return;
    
    const next = history.future[0];
    const newFuture = history.future.slice(1);
    
    set({
      history: {
        past: [...history.past, history.present],
        present: next,
        future: newFuture
      }
    });
  },
  
  save: async () => {
    const { project, history } = get();
    if (!project) return;
    
    await fetch(`/api/projects/${project.id}`, {
      method: 'PATCH',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        config: history.present
      })
    });
  }
}));

// Debounced save
const debouncedSave = debounce(() => {
  useEditorStore.getState().save();
}, 1000);
```


### 5.9 Real-time Collaboration

**WebSocket Server:**
```typescript
// Collaboration Server (Node.js + Socket.IO)
import { Server } from 'socket.io';
import { createAdapter } from '@socket.io/redis-adapter';
import { Redis } from 'ioredis';

interface CollaborationSession {
  projectId: string;
  users: Map<string, CollaborationUser>;
  cursors: Map<string, CursorPosition>;
  selections: Map<string, ElementSelection>;
}

interface CollaborationUser {
  id: string;
  name: string;
  avatar: string;
  color: string;
}

interface CursorPosition {
  x: number;
  y: number;
  timestamp: number;
}

interface ElementSelection {
  elementPath: string;
  timestamp: number;
}

class CollaborationServer {
  private io: Server;
  private sessions: Map<string, CollaborationSession>;
  
  constructor() {
    this.io = new Server({
      cors: { origin: '*' }
    });
    
    this.sessions = new Map();
    
    // Setup Redis adapter for multi-server support
    const pubClient = new Redis();
    const subClient = pubClient.duplicate();
    this.io.adapter(createAdapter(pubClient, subClient));
    
    this.setupEventHandlers();
  }
  
  private setupEventHandlers() {
    this.io.on('connection', (socket) => {
      console.log('User connected:', socket.id);
      
      // Join project room
      socket.on('join-project', async (data) => {
        const { projectId, user } = data;
        
        // Verify user has access
        const hasAccess = await this.verifyAccess(user.id, projectId);
        if (!hasAccess) {
          socket.emit('error', { message: 'Access denied' });
          return;
        }
        
        // Join room
        socket.join(`project:${projectId}`);
        
        // Get or create session
        let session = this.sessions.get(projectId);
        if (!session) {
          session = {
            projectId,
            users: new Map(),
            cursors: new Map(),
            selections: new Map()
          };
          this.sessions.set(projectId, session);
        }
        
        // Add user to session
        session.users.set(socket.id, {
          ...user,
          color: this.generateUserColor()
        });
        
        // Notify others
        socket.to(`project:${projectId}`).emit('user-joined', {
          socketId: socket.id,
          user: session.users.get(socket.id)
        });
        
        // Send current session state
        socket.emit('session-state', {
          users: Array.from(session.users.entries()),
          cursors: Array.from(session.cursors.entries()),
          selections: Array.from(session.selections.entries())
        });
      });
      
      // Handle cursor movement
      socket.on('cursor-move', (data) => {
        const { projectId, position } = data;
        const session = this.sessions.get(projectId);
        
        if (session) {
          session.cursors.set(socket.id, {
            ...position,
            timestamp: Date.now()
          });
          
          // Broadcast to others
          socket.to(`project:${projectId}`).emit('cursor-update', {
            socketId: socket.id,
            position
          });
        }
      });
      
      // Handle element selection
      socket.on('element-select', (data) => {
        const { projectId, elementPath } = data;
        const session = this.sessions.get(projectId);
        
        if (session) {
          session.selections.set(socket.id, {
            elementPath,
            timestamp: Date.now()
          });
          
          // Broadcast to others
          socket.to(`project:${projectId}`).emit('element-selected', {
            socketId: socket.id,
            elementPath
          });
        }
      });
      
      // Handle element update
      socket.on('element-update', async (data) => {
        const { projectId, elementPath, changes } = data;
        
        // Apply operational transformation
        const transformedChanges = await this.transformChanges(
          projectId,
          elementPath,
          changes
        );
        
        // Broadcast to others
        socket.to(`project:${projectId}`).emit('element-updated', {
          socketId: socket.id,
          elementPath,
          changes: transformedChanges
        });
        
        // Save to database (debounced)
        this.debouncedSave(projectId, elementPath, transformedChanges);
      });
      
      // Handle disconnect
      socket.on('disconnect', () => {
        // Remove user from all sessions
        for (const [projectId, session] of this.sessions.entries()) {
          if (session.users.has(socket.id)) {
            session.users.delete(socket.id);
            session.cursors.delete(socket.id);
            session.selections.delete(socket.id);
            
            // Notify others
            this.io.to(`project:${projectId}`).emit('user-left', {
              socketId: socket.id
            });
            
            // Clean up empty sessions
            if (session.users.size === 0) {
              this.sessions.delete(projectId);
            }
          }
        }
      });
    });
  }
  
  private async transformChanges(
    projectId: string,
    elementPath: string,
    changes: any
  ): Promise<any> {
    // Implement Operational Transformation (OT)
    // to handle concurrent edits
    
    // Get current state from database
    const currentState = await this.getCurrentState(projectId, elementPath);
    
    // Transform changes based on current state
    // This is a simplified version
    return changes;
  }
  
  private generateUserColor(): string {
    const colors = [
      '#FF6B6B', '#4ECDC4', '#45B7D1', '#FFA07A',
      '#98D8C8', '#F7DC6F', '#BB8FCE', '#85C1E2'
    ];
    return colors[Math.floor(Math.random() * colors.length)];
  }
}
```


## 6. الأمان (Security)

### 6.1 Authentication & Authorization

**JWT Token Structure:**
```typescript
interface JWTPayload {
  sub: string; // user ID
  email: string;
  name: string;
  workspaces: string[]; // workspace IDs
  iat: number; // issued at
  exp: number; // expiration
}

// Token generation
function generateTokens(user: User): TokenPair {
  const accessToken = jwt.sign(
    {
      sub: user.id,
      email: user.email,
      name: user.name,
      workspaces: user.workspaces.map(w => w.id)
    },
    process.env.JWT_SECRET!,
    { expiresIn: '15m' }
  );
  
  const refreshToken = jwt.sign(
    { sub: user.id },
    process.env.JWT_REFRESH_SECRET!,
    { expiresIn: '7d' }
  );
  
  return { accessToken, refreshToken };
}

// Authorization middleware
function authorize(requiredPermission: string) {
  return async (req: Request, res: Response, next: NextFunction) => {
    const token = req.headers.authorization?.replace('Bearer ', '');
    
    if (!token) {
      return res.status(401).json({ error: 'Unauthorized' });
    }
    
    try {
      const payload = jwt.verify(token, process.env.JWT_SECRET!) as JWTPayload;
      req.user = payload;
      
      // Check permission
      const hasPermission = await checkPermission(
        payload.sub,
        req.params.workspaceId || req.params.projectId,
        requiredPermission
      );
      
      if (!hasPermission) {
        return res.status(403).json({ error: 'Forbidden' });
      }
      
      next();
    } catch (error) {
      return res.status(401).json({ error: 'Invalid token' });
    }
  };
}
```

### 6.2 Rate Limiting

**Implementation:**
```typescript
import rateLimit from 'express-rate-limit';
import RedisStore from 'rate-limit-redis';
import { Redis } from 'ioredis';

const redis = new Redis();

// General API rate limit
const apiLimiter = rateLimit({
  store: new RedisStore({
    client: redis,
    prefix: 'rl:api:'
  }),
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // 100 requests per window
  message: 'Too many requests, please try again later'
});

// Deployment rate limit (more restrictive)
const deployLimiter = rateLimit({
  store: new RedisStore({
    client: redis,
    prefix: 'rl:deploy:'
  }),
  windowMs: 60 * 60 * 1000, // 1 hour
  max: 10, // 10 deployments per hour
  message: 'Too many deployments, please try again later'
});

// Apply to routes
app.use('/api', apiLimiter);
app.use('/api/projects/:id/deploy', deployLimiter);
```

### 6.3 Input Validation

**Using Zod:**
```typescript
import { z } from 'zod';

// Project creation schema
const createProjectSchema = z.object({
  name: z.string().min(1).max(255),
  slug: z.string().regex(/^[a-z0-9-]+$/),
  templateId: z.string().uuid(),
  workspaceId: z.string().uuid()
});

// Validate request
app.post('/api/projects', async (req, res) => {
  try {
    const data = createProjectSchema.parse(req.body);
    
    // Create project
    const project = await createProject(data);
    
    res.json(project);
  } catch (error) {
    if (error instanceof z.ZodError) {
      return res.status(400).json({
        error: 'Validation failed',
        details: error.errors
      });
    }
    throw error;
  }
});
```

### 6.4 SQL Injection Prevention

**Using Prisma (parameterized queries):**
```typescript
// Safe - Prisma uses parameterized queries
const user = await prisma.user.findUnique({
  where: { email: userInput }
});

// Safe - Prisma escapes input
const projects = await prisma.project.findMany({
  where: {
    name: { contains: searchQuery }
  }
});
```

### 6.5 XSS Prevention

**Content Security Policy:**
```typescript
import helmet from 'helmet';

app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'", "'unsafe-inline'"],
    styleSrc: ["'self'", "'unsafe-inline'"],
    imgSrc: ["'self'", 'data:', 'https:'],
    connectSrc: ["'self'", 'wss:', 'https:'],
    fontSrc: ["'self'", 'data:'],
    objectSrc: ["'none'"],
    mediaSrc: ["'self'"],
    frameSrc: ["'none'"]
  }
}));
```

**Output Sanitization:**
```typescript
import DOMPurify from 'isomorphic-dompurify';

// Sanitize user-generated HTML
function sanitizeHTML(html: string): string {
  return DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'u', 'a', 'ul', 'ol', 'li'],
    ALLOWED_ATTR: ['href', 'target', 'rel']
  });
}
```


## 7. المراقبة والملاحظة (Observability)

### 7.1 OpenTelemetry Setup

**Instrumentation:**
```typescript
import { NodeSDK } from '@opentelemetry/sdk-node';
import { getNodeAutoInstrumentations } from '@opentelemetry/auto-instrumentations-node';
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-http';
import { OTLPMetricExporter } from '@opentelemetry/exporter-metrics-otlp-http';
import { PeriodicExportingMetricReader } from '@opentelemetry/sdk-metrics';

const sdk = new NodeSDK({
  traceExporter: new OTLPTraceExporter({
    url: 'http://otel-collector:4318/v1/traces'
  }),
  metricReader: new PeriodicExportingMetricReader({
    exporter: new OTLPMetricExporter({
      url: 'http://otel-collector:4318/v1/metrics'
    }),
    exportIntervalMillis: 60000
  }),
  instrumentations: [
    getNodeAutoInstrumentations({
      '@opentelemetry/instrumentation-fs': { enabled: false }
    })
  ]
});

sdk.start();

// Custom spans
import { trace } from '@opentelemetry/api';

const tracer = trace.getTracer('stackly-api');

async function buildProject(projectId: string) {
  const span = tracer.startSpan('build_project');
  span.setAttribute('project.id', projectId);
  
  try {
    // Build logic
    const result = await performBuild(projectId);
    
    span.setAttribute('build.success', true);
    span.setAttribute('build.duration', result.duration);
    
    return result;
  } catch (error) {
    span.recordException(error);
    span.setAttribute('build.success', false);
    throw error;
  } finally {
    span.end();
  }
}
```

### 7.2 Logging

**Structured Logging:**
```typescript
import pino from 'pino';

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  formatters: {
    level: (label) => {
      return { level: label };
    }
  },
  timestamp: pino.stdTimeFunctions.isoTime
});

// Usage
logger.info({ projectId, userId }, 'Project created');
logger.error({ error, projectId }, 'Build failed');

// Request logging middleware
app.use((req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = Date.now() - start;
    
    logger.info({
      method: req.method,
      url: req.url,
      status: res.statusCode,
      duration,
      userId: req.user?.sub
    }, 'Request completed');
  });
  
  next();
});
```

### 7.3 Metrics

**Custom Metrics:**
```typescript
import { metrics } from '@opentelemetry/api';

const meter = metrics.getMeter('stackly-api');

// Counters
const projectsCreated = meter.createCounter('projects.created', {
  description: 'Number of projects created'
});

const deploymentsStarted = meter.createCounter('deployments.started', {
  description: 'Number of deployments started'
});

const deploymentsCompleted = meter.createCounter('deployments.completed', {
  description: 'Number of deployments completed'
});

// Histograms
const buildDuration = meter.createHistogram('build.duration', {
  description: 'Build duration in milliseconds',
  unit: 'ms'
});

const deployDuration = meter.createHistogram('deploy.duration', {
  description: 'Deployment duration in milliseconds',
  unit: 'ms'
});

// Gauges
const activeUsers = meter.createObservableGauge('users.active', {
  description: 'Number of active users'
});

activeUsers.addCallback(async (result) => {
  const count = await getActiveUserCount();
  result.observe(count);
});

// Usage
projectsCreated.add(1, { workspace: workspaceId });
buildDuration.record(duration, { projectId });
```


## 8. الأداء والتحسين (Performance & Optimization)

### 8.1 Caching Strategy

**Multi-Level Caching:**
```typescript
// Level 1: Memory Cache (in-process)
const memoryCache = new Map<string, any>();

// Level 2: Redis Cache (distributed)
import { Redis } from 'ioredis';
const redis = new Redis();

// Level 3: CDN Cache (edge)
// Handled by CDN provider

class CacheService {
  async get<T>(key: string): Promise<T | null> {
    // Check memory cache
    if (memoryCache.has(key)) {
      return memoryCache.get(key);
    }
    
    // Check Redis
    const cached = await redis.get(key);
    if (cached) {
      const value = JSON.parse(cached);
      memoryCache.set(key, value);
      return value;
    }
    
    return null;
  }
  
  async set(key: string, value: any, ttl: number = 3600): Promise<void> {
    // Set in memory
    memoryCache.set(key, value);
    
    // Set in Redis
    await redis.setex(key, ttl, JSON.stringify(value));
  }
  
  async invalidate(pattern: string): Promise<void> {
    // Clear memory cache
    for (const key of memoryCache.keys()) {
      if (key.match(pattern)) {
        memoryCache.delete(key);
      }
    }
    
    // Clear Redis cache
    const keys = await redis.keys(pattern);
    if (keys.length > 0) {
      await redis.del(...keys);
    }
  }
}

// Usage
const cache = new CacheService();

async function getTemplate(id: string): Promise<Template> {
  const cacheKey = `template:${id}`;
  
  // Try cache first
  const cached = await cache.get<Template>(cacheKey);
  if (cached) {
    return cached;
  }
  
  // Query database
  const template = await db.template.findUnique({ where: { id } });
  
  // Cache result
  await cache.set(cacheKey, template, 3600); // 1 hour
  
  return template;
}
```

### 8.2 Database Optimization

**Connection Pooling:**
```typescript
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient({
  datasources: {
    db: {
      url: process.env.DATABASE_URL
    }
  },
  log: ['query', 'error', 'warn'],
  // Connection pool settings
  __internal: {
    engine: {
      connection_limit: 20,
      pool_timeout: 10
    }
  }
});
```

**Query Optimization:**
```typescript
// Bad: N+1 query problem
const projects = await prisma.project.findMany();
for (const project of projects) {
  const workspace = await prisma.workspace.findUnique({
    where: { id: project.workspaceId }
  });
}

// Good: Use include to fetch related data
const projects = await prisma.project.findMany({
  include: {
    workspace: true,
    template: true
  }
});

// Good: Use select to fetch only needed fields
const projects = await prisma.project.findMany({
  select: {
    id: true,
    name: true,
    workspace: {
      select: {
        id: true,
        name: true
      }
    }
  }
});
```

**Indexes:**
```sql
-- Add indexes for frequently queried columns
CREATE INDEX idx_projects_workspace_id ON projects(workspace_id);
CREATE INDEX idx_projects_status ON projects(status);
CREATE INDEX idx_projects_created_at ON projects(created_at DESC);
CREATE INDEX idx_deployments_environment_id ON deployments(environment_id);
CREATE INDEX idx_deployments_status ON deployments(status);

-- Composite indexes for common queries
CREATE INDEX idx_projects_workspace_status ON projects(workspace_id, status);
CREATE INDEX idx_deployments_env_status ON deployments(environment_id, status);
```

### 8.3 Asset Optimization

**Image Optimization:**
```typescript
import sharp from 'sharp';

async function optimizeImage(
  inputPath: string,
  outputPath: string,
  options: ImageOptimizationOptions
): Promise<void> {
  const { width, height, quality = 80, format = 'webp' } = options;
  
  await sharp(inputPath)
    .resize(width, height, {
      fit: 'inside',
      withoutEnlargement: true
    })
    .toFormat(format, { quality })
    .toFile(outputPath);
}

// Generate multiple sizes
async function generateResponsiveImages(
  inputPath: string,
  outputDir: string
): Promise<ResponsiveImageSet> {
  const sizes = [320, 640, 768, 1024, 1280, 1920];
  const images: ResponsiveImageSet = {};
  
  for (const size of sizes) {
    const outputPath = `${outputDir}/${size}w.webp`;
    await optimizeImage(inputPath, outputPath, {
      width: size,
      format: 'webp',
      quality: 80
    });
    images[`${size}w`] = outputPath;
  }
  
  return images;
}
```

**Code Splitting:**
```typescript
// Next.js automatic code splitting
// Each page is automatically split

// Dynamic imports for heavy components
import dynamic from 'next/dynamic';

const Editor = dynamic(() => import('@/components/Editor'), {
  loading: () => <LoadingSpinner />,
  ssr: false // Don't render on server
});

// Route-based code splitting
const routes = [
  {
    path: '/dashboard',
    component: lazy(() => import('./pages/Dashboard'))
  },
  {
    path: '/editor',
    component: lazy(() => import('./pages/Editor'))
  }
];
```


## 9. البنية التحتية (Infrastructure)

### 9.1 Docker Compose (Development)

```yaml
version: '3.8'

services:
  # PostgreSQL
  postgres:
    image: postgres:18-alpine
    environment:
      POSTGRES_USER: stackly
      POSTGRES_PASSWORD: stackly
      POSTGRES_DB: stackly
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U stackly"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Valkey (Redis)
  valkey:
    image: valkey/valkey:latest
    ports:
      - "6379:6379"
    volumes:
      - valkey_data:/data
    healthcheck:
      test: ["CMD", "valkey-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  # NATS JetStream
  nats:
    image: nats:latest
    command: "-js -sd /data"
    ports:
      - "4222:4222"
      - "8222:8222"
    volumes:
      - nats_data:/data

  # MinIO
  minio:
    image: minio/minio:latest
    command: server /data --console-address ":9001"
    environment:
      MINIO_ROOT_USER: stackly
      MINIO_ROOT_PASSWORD: stackly123
    ports:
      - "9000:9000"
      - "9001:9001"
    volumes:
      - minio_data:/data

  # ClickHouse
  clickhouse:
    image: clickhouse/clickhouse-server:latest
    ports:
      - "8123:8123"
      - "9000:9000"
    volumes:
      - clickhouse_data:/var/lib/clickhouse

  # Temporal
  temporal:
    image: temporalio/auto-setup:latest
    environment:
      - DB=postgresql
      - DB_PORT=5432
      - POSTGRES_USER=stackly
      - POSTGRES_PWD=stackly
      - POSTGRES_SEEDS=postgres
    ports:
      - "7233:7233"
    depends_on:
      postgres:
        condition: service_healthy

  # ZITADEL
  zitadel:
    image: ghcr.io/zitadel/zitadel:latest
    command: 'start-from-init --masterkeyFromEnv --tlsMode disabled'
    environment:
      - ZITADEL_MASTERKEY=MasterkeyNeedsToHave32Characters
      - ZITADEL_DATABASE_POSTGRES_HOST=postgres
      - ZITADEL_DATABASE_POSTGRES_PORT=5432
      - ZITADEL_DATABASE_POSTGRES_DATABASE=zitadel
      - ZITADEL_DATABASE_POfile
    environment:
      - DATABASE_URL=postgresql://stackly:stackly@postgres:5432/stackly
      - REDIS_URL=redis://valkey:6379
      - NEXT_PUBLIC_API_URL=http://localhost:3001
    ports:
      - "3000:3000"
    depends_on:
      - postgres
      - valkey
      - api

  # Core API (Node.js)
  api:
    build:
      context: ./apps/api
      dockerfile: Dockerfile
    environment:
      - DATABASE_URL=postgresql://stackly:stackly@postgres:5432/stackly
      - REDIS_URL=redis://valkey:6379
      - NATS_URL=nats://nats:4222
      - MINIO_ENDPOINT=minio:9000
      - MINIO_ACCESS_KEY=stackly
      - MINIO_SECRET_KEY=stackly123
    ports:
      - "3001:3001"
    depends_on:
      - postgres
      - valkey
      - nats
      - minio

  # Workers (Go)
  workers:
    build:
      context: ./apps/workers
      dockerfile: Dockerfile
    environment:
      - DATABASE_URL=postgresql://stackly:stackly@postgres:5432/stackly
      - REDIS_URL=redis://valkey:6379
      - NATS_URL=nats://nats:4222
      - MINIO_ENDPOINT=minio:9000
      - MINIO_ACCESS_KEY=stackly
      - MINIO_SECRET_KEY=stackly123
      - TEMPORAL_HOST=temporal:7233
    depends_on:
      - postgres
      - valkey
      - nats
      - minio
      - temporal

volumes:
  postgres_data:
  valkey_data:
  nats_data:
  minio_data:
  clickhouse_data:
```


### 9.2 Kubernetes (Production)

**Namespace:**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: stackly
```

**PostgreSQL StatefulSet:**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
  namespace: stackly
spec:
  serviceName: postgres
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:18-alpine
        ports:
        - containerPort: 5432
        env:
        - name: POSTGRES_USER
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: username
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
        volumeMounts:
        - name: postgres-data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: postgres-data
    spec:
      accessModes: ["ReadWriteOnce"]
        name: database-secret
              key: url
        - name: REDIS_URL
          valueFrom:
            configMapKeyRef:
              name: stackly-config
              key: redis-url
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3001
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3001
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: api
  namespace: stackly
spec:
  selector:
    app: api
  ports:
  - port: 80
    targetPort: 3001
  type: ClusterIP
```

**HorizontalPodAutoscaler:**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
  namespace: stackly
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api
  minReplicas: 3
  maxReplicas: 10
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

**Gateway API:**
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: stackly-gateway
  namespace: stackly
spec:
  gatewayClassName: cilium
  listeners:
  - name: http
    protocol: HTTP
    port: 80
    hostname: "*.stackly.app"
  - name: https
    protocol: HTTPS
    port: 443
    hostname: "*.stackly.app"
    tls:
      mode: Terminate
      certificateRefs:
      - name: stackly-tls
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: api-route
  namespace: stackly
spec:
  parentRefs:
  - name: stackly-gateway
  hostnames:
  - "api.stackly.app"
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /api
    backendRefs:
    - name: api
      port: 80
```


## 10. الخلاصة والملاحظات (Summary & Notes)

### 10.1 القرارات المعمارية الرئيسية

1. **Microservices مع Monorepo**: نستخدم معمارية Microservices للمرونة والتوسع، مع الحفاظ على الكود في Monorepo لسهولة التطوير والمشاركة.

2. **Event-Driven Architecture**: نستخدم NATS JetStream لناقل الأحداث، مما يسمح بالتواصل غير المتزامن بين الخدمات.

3. **Multi-Level Caching**: نستخدم ثلاث مستويات من التخزين المؤقت (Memory, Redis, CDN) لتحسين الأداء.

4. **Flexible Provisioning**: نوفر 4 مستويات من الموارد (Static, Shared Schema, Dedicated Schema, Isolated DB) حسب الحاجة والخطة.

5. **Temporal for Workflows**: نستخدم Temporal لإدارة سير العمل المعقد مثل البناء والنشر.

6. **OpenTelemetry for Observability**: نستخدم OpenTelemetry لمراقبة شاملة للنظام.

### 10.2 الاعتبارات المستقبلية

1. **Multi-Region Deployment**: في المستقبل، يمكن نشر المنصة في مناطق جغرافية متعددة لتحسين الأداء.

2. **Edge Computing**: يمكن نقل بعض العمليات إلى Edge لتقليل الكمون.

3. **AI-Powered Features**: يمكن إضافة المزيد من ميزات الذكاء الاصطناعي مثل:
   - توليد القوالب تلقائياً
   - اقتراحات التصميم الذكية
   - تحسين SEO تلقائي
   - تحليل سلوك المستخدمين

4. **Marketplace**: يمكن إضافة سوق للقوالب والمكونات من مطورين خارجيين.

5. **White-Label Solution**: يمكن تقديم حل White-Label للشركات الكبيرة.

### 10.3 التحديات التقنية المتوقعة

1. **Scale**: التعامل مع آلاف المشاريع المتزامنة يتطلب تحسين مستمر.

2. **Real-time Collaboration**: التعاون الفوري يتطلب معالجة دقيقة للتزامن والتعارضات.

3. **Build Performance**: بناء المشاريع بسرعة يتطلب تحسين مستمر لعملية البناء.

4. **Cost Optimization**: إدارة التكاليف التشغيلية مع نمو المنصة.

### 10.4 الأولويات للتطوير

**المرحلة 1 (MVP):**
1. نظام المشاريع والمساحات
2. محرر مرئي أساسي
3. 3 قوالب جاهزة
4. نشر أساسي

**المرحلة 2 (Core Features):**
1. محرر متقدم مع Variants و Slots
2. 10 قوالب + 50 مكون
3. خدمات أساسية (Email, Domain, Payment)
4. خطط اشتراك متعددة

**المرحلة 3 (Advanced Features):**
1. خدمات متقدمة (Mailbox, Automation, Analytics)
2. نظام Provisioning متقدم
3. تعاون فوري
4. 20 قالب + 100 مكون

**المرحلة 4 (AI & Scale):**
1. ميزات الذكاء الاصطناعي
2. توسع عالمي
3. 30+ قالب + 200+ مكون
4. إطلاق عام

---

**تاريخ الإنشاء:** 2026-03-15  
**الإصدار:** 1.0.0  
**الحالة:** مسودة أولية (Draft)  
**المراجعة التالية:** بعد مراجعة المتطلبات


---

# الجزء الثالث: خطة التنفيذ (Implementation Plan)

## 11. استراتيجية التنفيذ التدريجي

### 11.1 نهج الانتقال التدريجي (Phased Migration Approach)

**القرار الاستراتيجي:**
نبدأ بـ Supabase للوصول السريع إلى MVP، ثم ننتقل تدريجياً إلى PostgreSQL + Prisma للحصول على التحكم الكامل المطلوب لمنصة Stackly.

**الأسباب:**
1. سرعة التطوير في المرحلة الأولى
2. الاستفادة من الخبرة الموجودة مع Supabase
3. تقليل المخاطر - اختبار الفكرة قبل الاستثمار الكبير
4. الانتقال عند الحاجة الفعلية (عندما نحتاج Multi-tenancy المتقدم)

### 11.2 المراحل الثلاث

#### المرحلة 1: MVP مع Supabase (الأشهر 1-3)

**التقنيات:**
```
Frontend:
- Next.js 15 App Router
- React 19
- TypeScript 5.7
- Tailwind CSS + shadcn/ui
- Supabase Client (@supabase/ssr)

Backend:
- Supabase Auth (المصادقة)
- Supabase Database (PostgreSQL مُدار)
- Supabase Storage (تخزين الملفات)
- Supabase Realtime (التعاون الفوري)
- Supabase Edge Functions (للعمليات المعقدة)

Infrastructure:
- Supabase Cloud (مُدار بالكامل)
```

**الهيكل:**
```
stackly/
├── apps/
│   └── dashboard/              # Next.js App
│       ├── app/
│       │   ├── (auth)/
│       │   │   ├── login/
│       │   │   └── register/
│       │   ├── (dashboard)/
│       │   │   ├── page.tsx
│       │   │   ├── projects/
│       │   │   └── settings/
│       │   └── layout.tsx
│       ├── components/
│       ├── lib/
│       │   ├── supabase/
│       │   │   ├── client.ts
│       │   │   ├── server.ts
│       │   │   └── middleware.ts
│       │   └── utils/
│       └── types/
├── packages/
│   ├── ui/                     # Shared UI components
│   ├── supabase/              # Supabase utilities
│   │   ├── types.ts
│   │   └── helpers.ts
│   └── config/
├── supabase/
│   ├── migrations/            # Database migrations
│   │   └── 20260315000001_initial_schema.sql
│   ├── functions/             # Edge Functions
│   │   ├── create-project/
│   │   └── deploy-project/
│   ├── seed.sql              # Initial data
│   └── config.toml
├── pnpm-workspace.yaml
├── turbo.json
└── package.json
```

**قاعدة البيانات (Supabase):**
```sql
-- Platform DB Schema

-- Workspaces
CREATE TABLE workspaces (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  slug TEXT UNIQUE NOT NULL,
  owner_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  plan_id UUID REFERENCES plans(id),
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Workspace Members
CREATE TABLE workspace_members (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  workspace_id UUID REFERENCES workspaces(id) ON DELETE CASCADE,
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  role TEXT NOT NULL CHECK (role IN ('owner', 'admin', 'member', 'viewer')),
  created_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(workspace_id, user_id)
);

-- Projects
CREATE TABLE projects (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  workspace_id UUID REFERENCES workspaces(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  slug TEXT NOT NULL,
  template_id UUID REFERENCES templates(id),
  status TEXT DEFAULT 'draft' CHECK (status IN ('draft', 'building', 'deployed', 'archived')),
  config JSONB DEFAULT '{}',
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(workspace_id, slug)
);

-- Templates
CREATE TABLE templates (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  slug TEXT UNIQUE NOT NULL,
  category TEXT NOT NULL,
  description TEXT,
  thumbnail_url TEXT,
  metadata JSONB DEFAULT '{}',
  ui_schema JSONB DEFAULT '{}',
  theme_schema JSONB DEFAULT '{}',
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Blocks
CREATE TABLE blocks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  slug TEXT UNIQUE NOT NULL,
  category TEXT NOT NULL,
  schema JSONB DEFAULT '{}',
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Components
CREATE TABLE components (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  slug TEXT UNIQUE NOT NULL,
  category TEXT NOT NULL,
  schema JSONB DEFAULT '{}',
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Plans
CREATE TABLE plans (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  slug TEXT UNIQUE NOT NULL,
  price_monthly DECIMAL(10, 2) NOT NULL,
  limits JSONB DEFAULT '{}',
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Enable Row Level Security
ALTER TABLE workspaces ENABLE ROW LEVEL SECURITY;
ALTER TABLE workspace_members ENABLE ROW LEVEL SECURITY;
ALTER TABLE projects ENABLE ROW LEVEL SECURITY;

-- RLS Policies
CREATE POLICY "Users can view their workspaces"
ON workspaces FOR SELECT
USING (
  id IN (
    SELECT workspace_id FROM workspace_members
    WHERE user_id = auth.uid()
  )
);

CREATE POLICY "Users can view their projects"
ON projects FOR SELECT
USING (
  workspace_id IN (
    SELECT workspace_id FROM workspace_members
    WHERE user_id = auth.uid()
  )
);
```

**Multi-tenancy في المرحلة 1:**
```sql
-- App Data Tables (Shared Schema مع RLS)

-- Products (لمشاريع التجارة الإلكترونية)
CREATE TABLE products (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  description TEXT,
  price DECIMAL(10,2) NOT NULL,
  stock INTEGER DEFAULT 0,
  images JSONB DEFAULT '[]',
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- RLS للعزل المنطقي
CREATE POLICY "Users can only access their project's products"
ON products FOR ALL
USING (
  project_id IN (
    SELECT p.id FROM projects p
    JOIN workspaces w ON p.workspace_id = w.id
    JOIN workspace_members wm ON w.id = wm.workspace_id
    WHERE wm.user_id = auth.uid()
  )
);

ALTER TABLE products ENABLE ROW LEVEL SECURITY;
```

**القيود في المرحلة 1:**
- ✅ Level 1 (Static-Only): مدعوم بالكامل
- ✅ Level 2 (Shared Schema): مدعوم عبر RLS
- ❌ Level 3 (Dedicated Schema): غير مدعوم (نؤجله للمرحلة 2)
- ❌ Level 4 (Isolated DB): غير مدعوم (نؤجله للمرحلة 2)


#### المرحلة 2: Hybrid Architecture (الأشهر 4-6)

**التقنيات المضافة:**
```
Backend:
+ Node.js API (Fastify)
+ PostgreSQL + Prisma (Platform DB)
+ MinIO (Storage)
+ NATS JetStream (Event Bus)
+ Temporal (Workflows)

Infrastructure:
+ Docker Compose (Development)
+ Kubernetes (Production - optional)
```

**الهيكل المحدث:**
```
stackly/
├── apps/
│   ├── dashboard/              # Next.js (يستخدم Supabase + API)
│   ├── api/                    # Node.js + Fastify
│   │   ├── src/
│   │   │   ├── routes/
│   │   │   ├── services/
│   │   │   ├── middleware/
│   │   │   └── index.ts
│   │   └── package.json
│   └── workers/                # Go Workers
│       ├── cmd/
│       ├── internal/
│       └── go.mod
├── packages/
│   ├── database/              # Prisma Schema
│   │   ├── prisma/
│   │   │   └── schema.prisma
│   │   └── package.json
│   ├── ui/
│   └── types/
├── supabase/                  # لا نزال نستخدمه للـ Auth
└── docker-compose.yml
```

**استراتيجية الانتقال:**
```typescript
// 1. نبقي Supabase Auth
import { createClient } from '@supabase/supabase-js'
const supabase = createClient(url, key)

// 2. ننقل Platform DB إلى Prisma
import { PrismaClient } from '@prisma/client'
const prisma = new PrismaClient()

// 3. App Data يبقى في Supabase (Shared Schema)
// لكن نبدأ دعم Dedicated Schema عبر Prisma

// مثال: إنشاء schema مخصص
async function provisionDedicatedSchema(projectId: string) {
  await prisma.$executeRaw`CREATE SCHEMA project_${projectId}`;
  
  // إنشاء الجداول في الـ schema الجديد
  await prisma.$executeRaw`
    CREATE TABLE project_${projectId}.products (
      id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
      name TEXT NOT NULL,
      price DECIMAL(10,2) NOT NULL,
      -- ... باقي الحقول
    )
  `;
}
```

**الميزات الجديدة:**
- ✅ Level 3 (Dedicated Schema): مدعوم الآن
- ✅ Build & Deployment Pipeline (Temporal)
- ✅ Event-Driven Architecture (NATS)
- ✅ Advanced Provisioning

#### المرحلة 3: Full Control (الأشهر 7-12)

**التقنيات المضافة:**
```
Backend:
+ ZITADEL/Keycloak (Auth)
+ Socket.IO (Realtime)
+ ClickHouse (Analytics)

Infrastructure:
+ Kubernetes (Production)
+ Argo CD (GitOps)
+ OpenTelemetry (Observability)
```

**الانتقال الكامل:**
```
❌ Supabase Auth → ✅ ZITADEL/Keycloak
❌ Supabase DB → ✅ PostgreSQL + Prisma (كل شيء)
❌ Supabase Storage → ✅ MinIO
❌ Supabase Realtime → ✅ Socket.IO
```

**الميزات الجديدة:**
- ✅ Level 4 (Isolated DB): مدعوم الآن
- ✅ Multi-Region Deployment
- ✅ Advanced Analytics (ClickHouse)
- ✅ Full Observability Stack

### 11.3 جدول الانتقال

| الميزة | المرحلة 1 (Supabase) | المرحلة 2 (Hybrid) | المرحلة 3 (Full Control) |
|--------|---------------------|-------------------|------------------------|
| Authentication | Supabase Auth | Supabase Auth | ZITADEL/Keycloak |
| Platform DB | Supabase | Prisma | Prisma |
| App Data (Shared) | Supabase + RLS | Supabase + RLS | Prisma |
| App Data (Dedicated) | ❌ | Prisma | Prisma |
| App Data (Isolated) | ❌ | ❌ | Prisma |
| Storage | Supabase Storage | MinIO | MinIO |
| Realtime | Supabase Realtime | Supabase Realtime | Socket.IO |
| Event Bus | ❌ | NATS | NATS |
| Workflows | ❌ | Temporal | Temporal |
| Analytics | ❌ | ❌ | ClickHouse |

### 11.4 معايير الانتقال بين المراحل

**من المرحلة 1 إلى المرحلة 2:**
- عدد المشاريع > 1000
- الحاجة لـ Dedicated Schema
- تكلفة Supabase تصبح عالية
- الحاجة لـ Event-Driven Architecture

**من المرحلة 2 إلى المرحلة 3:**
- عدد المشاريع > 10,000
- الحاجة لـ Isolated DB
- الحاجة لـ Multi-Region
- الحاجة لتحليلات متقدمة

