# RICE WORLD ERP - COMPLETE TECHNICAL DOCUMENTATION

## 24-MONTH ENTERPRISE DEVELOPMENT PLAN

---

## TABLE OF CONTENTS

1. [Executive Summary & Vision](#1-executive-summary--vision)
2. [Complete System Architecture](#2-complete-system-architecture)
3. [Database Design & Schema](#3-database-design--schema)
4. [Frontend Architecture](#4-frontend-architecture)
5. [Backend Services & APIs](#5-backend-services--apis)
6. [Security & Access Control](#6-security--access-control)
7. [Detailed 24-Month Development Timeline](#7-detailed-24-month-development-timeline)
8. [Technical Implementation Details by Module](#8-technical-implementation-details-by-module)
9. [Integration Strategy](#9-integration-strategy)
10. [Performance Optimization](#10-performance-optimization)
11. [Testing Strategy](#11-testing-strategy)
12. [Deployment & DevOps](#12-deployment--devops)
13. [Monitoring & Maintenance](#13-monitoring--maintenance)
14. [Risk Mitigation](#14-risk-mitigation)
15. [Resource Requirements](#15-resource-requirements)
16. [Success Metrics & KPIs](#16-success-metrics--kpis)

---

## 1. EXECUTIVE SUMMARY & VISION

### 1.1 Project Overview

**Project Name:** Rice World ERP System  
**Duration:** 24 Months  
**Development Model:** Solo Developer (Amaan) + Recommended Support  
**Technology Stack:** Next.js 16+, TypeScript, Supabase, Tailwind CSS v4  

### 1.2 System Vision

To build a comprehensive, enterprise-grade ERP system specifically designed for the rice industry that will:

1. **Integrate All Operations:** From paddy procurement to export sales, with seamless data flow between departments
2. **Enable Digital Traceability:** Track every transaction, decision, and document with complete audit trails
3. **Empower Workforce:** Provide structured task management, request systems, and performance tracking
4. **Drive Data-Driven Decisions:** Advanced analytics and business intelligence for strategic planning
5. **Ensure Compliance:** Full compliance with Pakistani tax regulations and international trade requirements

### 1.3 Core Modules

| Module | Description | Priority |
|--------|-------------|----------|
| Master Data Management | Products, parties, chart of accounts, departments | Critical |
| Procurement Management | Paddy deals, GRN, supplier management | Critical |
| Quality Management | Lab testing, certifications, instrument integration | Critical |
| Production Management | Production planning, BOM, costing | Critical |
| Inventory & Warehouse | Multi-location WMS with bin management | Critical |
| Sales & Distribution | Local sales, export management, logistics | Critical |
| **Task Management** | **Task assignment, tracking, comments, scoring** | **High (New)** |
| **Request Management** | **Departmental requests, approvals workflow** | **High (New)** |
| **Document Management** | **Centralized document storage and linking** | **High (New)** |
| Finance & Accounting | GL, AP/AR, multi-currency, consolidation | Critical |
| Tax Management | Sales tax, WHT, FBR integration | Critical |
| HR & Payroll | Employee management, attendance, payroll | High |
| Fixed Assets | Asset lifecycle, depreciation | Medium |
| Business Intelligence | Analytics, predictive models, custom reports | High |
| Mobile Applications | iOS/Android for field operations | Medium |

---

## 2. COMPLETE SYSTEM ARCHITECTURE

### 2.1 High-Level Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           CLIENT LAYER                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│  Web Application    │    Mobile App (iOS/Android)   │   Admin Console      │
│  (Next.js 16+)     │    (React Native/Flutter)     │   (Next.js)          │
└────────┬────────────────────────────────────────────────┬──────────────────┘
         │                                                │
         ▼                                                ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                        API GATEWAY & SERVICE LAYER                          │
├─────────────────────────────────────────────────────────────────────────────┤
│  Authentication   │   Authorization   │   Rate Limiting   │   Caching       │
│  (JWT, SSO)       │   (RBAC)          │   (Upstash)       │   (Redis)       │
└────────┬────────────────────────────────────────────────┬──────────────────┘
         │                                                │
         ▼                                                ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                       BUSINESS LOGIC LAYER                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐    │
│  │  Core Modules   │  │  Task/Request   │  │  Integration Services   │    │
│  │  - Procurement  │  │  - Assignment   │  │  - QuickBooks           │    │
│  │  - Production   │  │  - Approval     │  │  - Banking APIs         │    │
│  │  - Sales        │  │  - Scoring      │  │  - FBR (Pakistan Tax)   │    │
│  │  - Inventory    │  │  - Comments     │  │  - SMS/Email            │    │
│  │  - Finance      │  │  - Documents    │  │  - GPS Tracking         │    │
│  └─────────────────┘  └─────────────────┘  └─────────────────────────┘    │
└────────┬────────────────────────────────────────────────┬──────────────────┘
         │                                                │
         ▼                                                ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         DATABASE LAYER                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │  Supabase PostgreSQL                                                │  │
│  │  - Tables (180+)           - Views (60+)                            │  │
│  │  - Functions (120+)        - Triggers (60+)                         │  │
│  │  - RLS Policies (250+)     - Materialized Views (25+)              │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │  Supabase Storage                                                   │  │
│  │  - Documents Bucket        - Images Bucket                          │  │
│  │  - Attachments Bucket      - Reports Bucket                         │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │  Supabase Edge Functions                                             │  │
│  │  - Payment Processing      - Report Generation                     │  │
│  │  - Email/SMS Services      - Webhook Handlers                      │  │
│  │  - Scheduled Jobs          - Data Export                           │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Technology Stack Details

#### Frontend Technologies
```typescript
{
  "framework": "Next.js 16+ with App Router",
  "language": "TypeScript (Strict Mode)",
  "styling": "Tailwind CSS v4 with Custom Design System",
  "stateManagement": "Zustand + React Query",
  "forms": "React Hook Form + Zod Validation",
  "tables": "TanStack Table v8",
  "charts": "Recharts + D3.js",
  "dragAndDrop": "react-beautiful-dnd",
  "fileUpload": "react-dropzone",
  "richText": "react-quill / TipTap",
  "pdfGeneration": "react-pdf / @react-pdf/renderer",
  "dateHandling": "date-fns / dayjs",
  "notifications": "react-hot-toast / sonner"
}
```

#### Backend Technologies
```typescript
{
  "database": "Supabase PostgreSQL 15+",
  "auth": "Supabase Auth + JWT + SSO",
  "storage": "Supabase Storage",
  "serverless": "Supabase Edge Functions (Deno)",
  "caching": "Redis (via Upstash)",
  "queue": "BullMQ (for background jobs)",
  "search": "PostgreSQL Full-Text Search + pg_trgm",
  "analytics": "Mixpanel / Plausible"
}
```

#### Third-Party Integrations
```typescript
{
  "financial": {
    "quickbooks": "QuickBooks Online/Desktop API",
    "tally": "Tally ERP Integration",
    "xero": "Xero Accounting API"
  },
  "banking": {
    "paymentGateways": ["Stripe", "JazzCash", "EasyPaisa"],
    "bankAPIs": ["HBL", "UBL", "MCB", "ABL"]
  },
  "tax": {
    "fbr": "FBR Integration (Sales Tax, WHT)"
  },
  "communication": {
    "email": "SendGrid / Resend",
    "sms": "Twilio",
    "whatsapp": "WhatsApp Business API"
  },
  "logistics": {
    "gps": "GPS Tracking Providers",
    "fleet": "Fleet Management Systems"
  },
  "hr": {
    "biometric": "Biometric Device Integration",
    "payroll": "Payroll Systems"
  },
  "analytics": {
    "google": "Google Analytics 4",
    "mixpanel": "Mixpanel"
  }
}
```

---

## 3. DATABASE DESIGN & SCHEMA

### 3.1 Entity Relationship Diagram (ERD) - Complete Schema

The database consists of **180+ tables** organized into logical groups. Below is the comprehensive schema structure.

#### 3.1.1 Master Data Tables

```sql
-- Users & Authentication
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  first_name VARCHAR(100) NOT NULL,
  last_name VARCHAR(100) NOT NULL,
  phone VARCHAR(20),
  department_id UUID REFERENCES departments(id),
  designation VARCHAR(100),
  employee_code VARCHAR(50) UNIQUE,
  expertise TEXT[],
  performance_score DECIMAL(5,2) DEFAULT 0,
  profile_image_url TEXT,
  is_active BOOLEAN DEFAULT true,
  last_login_at TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Departments
CREATE TABLE departments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(100) NOT NULL,
  code VARCHAR(20) UNIQUE NOT NULL,
  head_user_id UUID REFERENCES users(id),
  parent_department_id UUID REFERENCES departments(id),
  cost_center_id UUID REFERENCES cost_centers(id),
  description TEXT,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Roles & Permissions
CREATE TABLE roles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(50) NOT NULL,
  slug VARCHAR(50) UNIQUE NOT NULL,
  description TEXT,
  level INTEGER NOT NULL, -- 1-7 for hierarchy
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE TABLE permissions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  resource VARCHAR(100) NOT NULL,
  action VARCHAR(50) NOT NULL,
  description TEXT,
  UNIQUE(resource, action)
);

CREATE TABLE role_permissions (
  role_id UUID REFERENCES roles(id) ON DELETE CASCADE,
  permission_id UUID REFERENCES permissions(id) ON DELETE CASCADE,
  PRIMARY KEY (role_id, permission_id)
);

CREATE TABLE user_roles (
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  role_id UUID REFERENCES roles(id) ON DELETE CASCADE,
  assigned_by UUID REFERENCES users(id),
  assigned_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  PRIMARY KEY (user_id, role_id)
);

-- Chart of Accounts
CREATE TABLE chart_of_accounts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  account_code VARCHAR(50) UNIQUE NOT NULL,
  account_name VARCHAR(255) NOT NULL,
  account_type VARCHAR(50) NOT NULL, -- Asset, Liability, Equity, Revenue, Expense
  parent_account_id UUID REFERENCES chart_of_accounts(id),
  level INTEGER NOT NULL, -- 1-5 hierarchy
  account_group VARCHAR(100),
  is_contra BOOLEAN DEFAULT false,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Cost Centers
CREATE TABLE cost_centers (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  code VARCHAR(50) UNIQUE NOT NULL,
  name VARCHAR(255) NOT NULL,
  type VARCHAR(50) NOT NULL, -- Department, Project, Location
  parent_cost_center_id UUID REFERENCES cost_centers(id),
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Product Master
CREATE TABLE product_categories (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(100) NOT NULL,
  code VARCHAR(50) UNIQUE NOT NULL,
  parent_category_id UUID REFERENCES product_categories(id),
  description TEXT,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE TABLE products (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  product_code VARCHAR(50) UNIQUE NOT NULL,
  product_name VARCHAR(255) NOT NULL,
  category_id UUID REFERENCES product_categories(id),
  product_type VARCHAR(50) NOT NULL, -- Raw Material, FG, By-product, Store
  uom VARCHAR(20) NOT NULL, -- KG, Bags, Tons
  conversion_factor DECIMAL(10,4) DEFAULT 1,
  specification JSONB, -- Custom attributes for rice/paddy
  standard_cost DECIMAL(15,2),
  selling_price DECIMAL(15,2),
  purchase_price DECIMAL(15,2),
  image_url TEXT,
  barcode VARCHAR(100),
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Rice/Paddy Specific Attributes
CREATE TABLE rice_specifications (
  product_id UUID PRIMARY KEY REFERENCES products(id),
  variety VARCHAR(100), -- Basmati, Irri, Super
  crop_year INTEGER,
  grade VARCHAR(50), -- Grade 1, Grade 2, etc.
  moisture_max DECIMAL(5,2),
  broken_percentage_max DECIMAL(5,2),
  foreign_matter_max DECIMAL(5,2),
  chalky_grain_max DECIMAL(5,2),
  milling_degree VARCHAR(50),
  type VARCHAR(50), -- White, Brown, Parboiled
  length_avg DECIMAL(6,2),
  width_avg DECIMAL(6,2),
  color VARCHAR(50)
);

-- Party Management
CREATE TABLE party_types (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(50) NOT NULL, -- Supplier, Customer, Farmer, Agent
  code VARCHAR(20) UNIQUE NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE TABLE parties (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  party_code VARCHAR(50) UNIQUE NOT NULL,
  party_type_id UUID REFERENCES party_types(id),
  legal_name VARCHAR(255) NOT NULL,
  trading_name VARCHAR(255),
  cnic VARCHAR(20),
  ntn VARCHAR(20),
  strn VARCHAR(20),
  secp_reg_no VARCHAR(50),
  address TEXT,
  city VARCHAR(100),
  district VARCHAR(100),
  province VARCHAR(50),
  country VARCHAR(50) DEFAULT 'Pakistan',
  phone VARCHAR(20),
  email VARCHAR(255),
  website VARCHAR(255),
  credit_limit DECIMAL(15,2),
  credit_terms INTEGER, -- Days
  rating INTEGER DEFAULT 3, -- 1-5 stars
  is_blacklisted BOOLEAN DEFAULT false,
  blacklist_reason TEXT,
  documents JSONB,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE TABLE party_contacts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  party_id UUID REFERENCES parties(id) ON DELETE CASCADE,
  contact_name VARCHAR(255) NOT NULL,
  designation VARCHAR(100),
  phone VARCHAR(20),
  email VARCHAR(255),
  is_primary BOOLEAN DEFAULT false,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE TABLE party_bank_accounts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  party_id UUID REFERENCES parties(id) ON DELETE CASCADE,
  bank_name VARCHAR(255) NOT NULL,
  account_title VARCHAR(255) NOT NULL,
  account_number VARCHAR(50) NOT NULL,
  iban VARCHAR(50),
  branch_code VARCHAR(20),
  swift_code VARCHAR(20),
  is_primary BOOLEAN DEFAULT false,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Warehouse & Inventory
CREATE TABLE warehouses (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  warehouse_code VARCHAR(50) UNIQUE NOT NULL,
  warehouse_name VARCHAR(255) NOT NULL,
  address TEXT,
  city VARCHAR(100),
  province VARCHAR(50),
  contact_person VARCHAR(255),
  contact_phone VARCHAR(20),
  capacity DECIMAL(15,2), -- Total capacity in KG
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE TABLE warehouse_zones (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  warehouse_id UUID REFERENCES warehouses(id) ON DELETE CASCADE,
  zone_code VARCHAR(50) NOT NULL,
  zone_name VARCHAR(255) NOT NULL,
  description TEXT,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  UNIQUE(warehouse_id, zone_code)
);

CREATE TABLE warehouse_bins (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  zone_id UUID REFERENCES warehouse_zones(id) ON DELETE CASCADE,
  bin_code VARCHAR(50) NOT NULL,
  bin_name VARCHAR(255) NOT NULL,
  capacity DECIMAL(15,2), -- Capacity in KG
  current_stock DECIMAL(15,2) DEFAULT 0,
  storage_conditions JSONB,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  UNIQUE(zone_id, bin_code)
);

-- Currency & Exchange Rates
CREATE TABLE currencies (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  code VARCHAR(10) UNIQUE NOT NULL, -- PKR, USD, EUR
  name VARCHAR(100) NOT NULL,
  symbol VARCHAR(10),
  decimal_places INTEGER DEFAULT 2,
  is_base_currency BOOLEAN DEFAULT false,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE TABLE exchange_rates (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  from_currency_id UUID REFERENCES currencies(id),
  to_currency_id UUID REFERENCES currencies(id),
  rate DECIMAL(15,6) NOT NULL,
  effective_date DATE NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  UNIQUE(from_currency_id, to_currency_id, effective_date)
);

-- Fiscal Years & Periods
CREATE TABLE fiscal_years (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  fiscal_year_code VARCHAR(20) UNIQUE NOT NULL,
  fiscal_year_name VARCHAR(255) NOT NULL,
  start_date DATE NOT NULL,
  end_date DATE NOT NULL,
  is_closed BOOLEAN DEFAULT false,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE TABLE fiscal_periods (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  fiscal_year_id UUID REFERENCES fiscal_years(id),
  period_number INTEGER NOT NULL,
  start_date DATE NOT NULL,
  end_date DATE NOT NULL,
  is_closed BOOLEAN DEFAULT false,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  UNIQUE(fiscal_year_id, period_number)
);
```

#### 3.1.2 Task Management Tables (NEW)

```sql
-- Task Categories (Flexible Hierarchy)
CREATE TABLE task_categories (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  category_code VARCHAR(50) UNIQUE NOT NULL,
  category_name VARCHAR(255) NOT NULL,
  parent_category_id UUID REFERENCES task_categories(id),
  category_type VARCHAR(50) NOT NULL, -- Maintenance, Production, Quality, HR, Admin
  icon VARCHAR(50),
  color VARCHAR(20),
  default_priority VARCHAR(20) DEFAULT 'Medium',
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Job Locations (Factory Areas)
CREATE TABLE job_locations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  location_code VARCHAR(50) UNIQUE NOT NULL,
  location_name VARCHAR(255) NOT NULL,
  description TEXT,
  department_id UUID REFERENCES departments(id),
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Main Tasks Table
CREATE TABLE tasks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  task_number VARCHAR(50) UNIQUE NOT NULL, -- Auto-generated (TASK-YYYY-XXXX)
  title VARCHAR(500) NOT NULL,
  description TEXT,
  
  -- Hierarchy
  parent_task_id UUID REFERENCES tasks(id),
  task_level INTEGER DEFAULT 0,
  
  -- Categorization
  category_id UUID REFERENCES task_categories(id),
  job_location_id UUID REFERENCES job_locations(id),
  
  -- Assignment
  assigned_to_id UUID REFERENCES users(id),
  created_by_id UUID REFERENCES users(id) NOT NULL,
  requested_by_id UUID REFERENCES users(id), -- For requests/needs
  
  -- Priorities & Scheduling
  priority VARCHAR(20) NOT NULL CHECK (priority IN ('Low', 'Medium', 'High', 'Urgent/Overdue')),
  due_date DATE,
  pending_days INTEGER DEFAULT 0,
  estimated_hours DECIMAL(8,2),
  actual_hours DECIMAL(8,2),
  
  -- Status Management
  status VARCHAR(30) NOT NULL DEFAULT 'Pending' CHECK (status IN (
    'Pending', 
    'In Progress', 
    'Work in Progress',
    'Completed', 
    'Done / Completed',
    'Cancelled / On Hold',
    'Not Started',
    'Urgent / Overdue'
  )),
  completion_percentage INTEGER DEFAULT 0,
  
  -- Buckets (0-5 days, 5-10 days, etc.)
  bucket VARCHAR(50),
  
  -- Scoring System
  task_score DECIMAL(10,2) DEFAULT 0,
  complexity_score DECIMAL(5,2) DEFAULT 0, -- Pre-assigned
  quality_score DECIMAL(5,2) DEFAULT 0, -- Assigned on completion
  contribution_score DECIMAL(10,2) DEFAULT 0, -- Auto-calculated
  
  -- Metadata
  approved_by_id UUID REFERENCES users(id),
  approval_status VARCHAR(20) DEFAULT 'Pending',
  highlighted_by VARCHAR(255), -- Who highlighted the task
  is_overdue_days INTEGER DEFAULT 0,
  
  -- Traceability
  source_module VARCHAR(50), -- Procurement, Production, Sales, etc.
  source_record_id UUID, -- Link to specific deal, order, etc.
  
  -- Timestamps
  assigned_date DATE,
  start_date TIMESTAMP WITH TIME ZONE,
  completed_at TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  -- Indexes for performance
  INDEX idx_tasks_status (status),
  INDEX idx_tasks_assigned_to (assigned_to_id),
  INDEX idx_tasks_due_date (due_date),
  INDEX idx_tasks_category (category_id),
  INDEX idx_tasks_parent (parent_task_id),
  INDEX idx_tasks_source (source_module, source_record_id)
);

-- Task Dependencies
CREATE TABLE task_dependencies (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  task_id UUID REFERENCES tasks(id) ON DELETE CASCADE,
  predecessor_task_id UUID REFERENCES tasks(id) ON DELETE CASCADE,
  dependency_type VARCHAR(20) DEFAULT 'Finish-to-Start',
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  UNIQUE(task_id, predecessor_task_id)
);

-- Task Multiple Assignees
CREATE TABLE task_assignments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  task_id UUID REFERENCES tasks(id) ON DELETE CASCADE,
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  assigned_by UUID REFERENCES users(id),
  assigned_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  role VARCHAR(50) DEFAULT 'Assignee', -- Assignee, Reviewer, Observer
  UNIQUE(task_id, user_id)
);

-- Task Comments & Remarks
CREATE TABLE task_comments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  task_id UUID REFERENCES tasks(id) ON DELETE CASCADE,
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  comment TEXT NOT NULL,
  comment_type VARCHAR(20) DEFAULT 'Regular', -- Regular, Concern, Highlight
  is_concern BOOLEAN DEFAULT false,
  is_highlight BOOLEAN DEFAULT false,
  parent_comment_id UUID REFERENCES task_comments(id), -- For threaded comments
  sentiment_score DECIMAL(3,2), -- For future NLP analysis
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  INDEX idx_task_comments_task (task_id),
  INDEX idx_task_comments_user (user_id)
);

-- Task Attachments
CREATE TABLE task_attachments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  task_id UUID REFERENCES tasks(id) ON DELETE CASCADE,
  uploaded_by_id UUID REFERENCES users(id),
  file_name VARCHAR(255) NOT NULL,
  file_url TEXT NOT NULL,
  file_type VARCHAR(100),
  file_size INTEGER, -- In bytes
  mime_type VARCHAR(100),
  uploaded_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  INDEX idx_task_attachments_task (task_id)
);

-- Task Status History (Audit Trail)
CREATE TABLE task_status_history (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  task_id UUID REFERENCES tasks(id) ON DELETE CASCADE,
  previous_status VARCHAR(30),
  new_status VARCHAR(30) NOT NULL,
  changed_by_id UUID REFERENCES users(id),
  comment TEXT,
  changed_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  INDEX idx_task_status_history_task (task_id),
  INDEX idx_task_status_history_changed_at (changed_at)
);
```

#### 3.1.3 Requests & Approvals Tables (NEW)

```sql
-- Request Types
CREATE TABLE request_types (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  type_code VARCHAR(50) UNIQUE NOT NULL,
  type_name VARCHAR(255) NOT NULL,
  category VARCHAR(50) NOT NULL, -- HR, Procurement, Finance, Admin
  description TEXT,
  default_priority VARCHAR(20) DEFAULT 'Medium',
  requires_approval BOOLEAN DEFAULT true,
  approval_workflow_id UUID REFERENCES approval_workflows(id),
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Requests Table
CREATE TABLE requests (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  request_number VARCHAR(50) UNIQUE NOT NULL,
  task_id UUID REFERENCES tasks(id) ON DELETE CASCADE, -- Link to task
  request_type_id UUID REFERENCES request_types(id),
  
  -- Request Data (JSONB for flexible structure)
  request_data JSONB NOT NULL, -- Stores type-specific data
  
  -- Department making request
  department_id UUID REFERENCES departments(id),
  requested_by_id UUID REFERENCES users(id) NOT NULL,
  
  -- Status
  status VARCHAR(30) DEFAULT 'Draft' CHECK (status IN (
    'Draft', 
    'Submitted', 
    'Under Review', 
    'Approved', 
    'Rejected', 
    'Cancelled'
  )),
  
  -- Approvals
  current_approval_level INTEGER DEFAULT 0,
  approved_by_id UUID REFERENCES users(id),
  approval_date TIMESTAMP WITH TIME ZONE,
  rejection_reason TEXT,
  
  -- Timestamps
  submitted_at TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  INDEX idx_requests_status (status),
  INDEX idx_requests_department (department_id),
  INDEX idx_requests_task (task_id)
);

-- Approval Workflows
CREATE TABLE approval_workflows (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  workflow_code VARCHAR(50) UNIQUE NOT NULL,
  workflow_name VARCHAR(255) NOT NULL,
  description TEXT,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Approval Workflow Steps
CREATE TABLE approval_workflow_steps (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  workflow_id UUID REFERENCES approval_workflows(id) ON DELETE CASCADE,
  step_order INTEGER NOT NULL,
  approver_role_id UUID REFERENCES roles(id),
  approver_user_id UUID REFERENCES users(id),
  is_parallel BOOLEAN DEFAULT false,
  escalation_days INTEGER DEFAULT 3,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  UNIQUE(workflow_id, step_order)
);

-- Approvals Table (Individual Approval Records)
CREATE TABLE approvals (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  request_id UUID REFERENCES requests(id) ON DELETE CASCADE,
  workflow_step_id UUID REFERENCES approval_workflow_steps(id),
  approver_id UUID REFERENCES users(id),
  status VARCHAR(20) DEFAULT 'Pending' CHECK (status IN ('Pending', 'Approved', 'Rejected')),
  comments TEXT,
  approved_at TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  INDEX idx_approvals_request (request_id),
  INDEX idx_approvals_approver (approver_id)
);
```

#### 3.1.4 Document Management Tables (NEW)

```sql
-- Document Types
CREATE TABLE document_types (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  type_code VARCHAR(50) UNIQUE NOT NULL,
  type_name VARCHAR(255) NOT NULL,
  description TEXT,
  required_for_modules TEXT[], -- Which modules use this doc type
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Documents Central Repository
CREATE TABLE documents (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  document_number VARCHAR(50) UNIQUE NOT NULL,
  document_type_id UUID REFERENCES document_types(id),
  document_name VARCHAR(500) NOT NULL,
  description TEXT,
  
  -- Linking to source records
  source_module VARCHAR(50) NOT NULL, -- Deal, Task, Request, etc.
  source_record_id UUID NOT NULL,
  
  -- File Information
  file_name VARCHAR(255) NOT NULL,
  file_url TEXT NOT NULL,
  file_type VARCHAR(100),
  file_size INTEGER,
  mime_type VARCHAR(100),
  file_hash VARCHAR(255), -- SHA-256 for integrity
  
  -- Metadata
  version INTEGER DEFAULT 1,
  is_latest_version BOOLEAN DEFAULT true,
  uploaded_by_id UUID REFERENCES users(id),
  uploaded_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  -- Approval/Status
  status VARCHAR(30) DEFAULT 'Draft' CHECK (status IN (
    'Draft', 
    'Submitted', 
    'Reviewed', 
    'Approved', 
    'Rejected'
  )),
  approved_by_id UUID REFERENCES users(id),
  approved_at TIMESTAMP WITH TIME ZONE,
  
  -- Timestamps
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  INDEX idx_documents_source (source_module, source_record_id),
  INDEX idx_documents_type (document_type_id),
  INDEX idx_documents_uploaded_by (uploaded_by_id),
  INDEX idx_documents_status (status)
);

-- Document Versions
CREATE TABLE document_versions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  document_id UUID REFERENCES documents(id) ON DELETE CASCADE,
  version_number INTEGER NOT NULL,
  file_url TEXT NOT NULL,
  file_name VARCHAR(255) NOT NULL,
  file_size INTEGER,
  uploaded_by_id UUID REFERENCES users(id),
  uploaded_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  change_notes TEXT,
  
  INDEX idx_doc_versions_document (document_id)
);

-- Document Categories (For Organization)
CREATE TABLE document_categories (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  category_code VARCHAR(50) UNIQUE NOT NULL,
  category_name VARCHAR(255) NOT NULL,
  parent_category_id UUID REFERENCES document_categories(id),
  description TEXT,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

#### 3.1.5 Procurement Tables

```sql
-- Paddy Deals
CREATE TABLE paddy_deals (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  deal_number VARCHAR(50) UNIQUE NOT NULL,
  supplier_id UUID REFERENCES parties(id),
  deal_date DATE NOT NULL,
  
  -- Deal Details
  product_id UUID REFERENCES products(id),
  quantity DECIMAL(15,2) NOT NULL,
  unit_price DECIMAL(15,2) NOT NULL,
  total_amount DECIMAL(15,2) NOT NULL,
  
  -- Specifications
  variety VARCHAR(100),
  crop_year INTEGER,
  moisture_max DECIMAL(5,2),
  foreign_matter_max DECIMAL(5,2),
  broken_percentage_max DECIMAL(5,2),
  
  -- Logistics
  delivery_terms VARCHAR(100),
  delivery_date DATE,
  warehouse_id UUID REFERENCES warehouses(id),
  
  -- Status
  status VARCHAR(30) DEFAULT 'Draft' CHECK (status IN (
    'Draft', 
    'Confirmed', 
    'In Transit', 
    'Completed', 
    'Cancelled'
  )),
  
  -- Approval
  approved_by_id UUID REFERENCES users(id),
  approved_at TIMESTAMP WITH TIME ZONE,
  
  -- Financial
  payment_terms VARCHAR(100),
  tax_rate DECIMAL(5,2),
  withholding_tax DECIMAL(5,2),
  
  -- Traceability
  created_by_id UUID REFERENCES users(id),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  INDEX idx_deals_supplier (supplier_id),
  INDEX idx_deals_status (status),
  INDEX idx_deals_date (deal_date)
);

-- Goods Receipt Notes (GRN)
CREATE TABLE grn (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  grn_number VARCHAR(50) UNIQUE NOT NULL,
  deal_id UUID REFERENCES paddy_deals(id),
  supplier_id UUID REFERENCES parties(id),
  
  -- Receiving Details
  vehicle_number VARCHAR(50),
  transporter_name VARCHAR(255),
  driver_name VARCHAR(255),
  driver_contact VARCHAR(20),
  weight_bridge_ticket VARCHAR(50),
  
  -- Weighing
  gross_weight DECIMAL(15,2),
  tare_weight DECIMAL(15,2),
  net_weight DECIMAL(15,2),
  unit_price DECIMAL(15,2),
  total_amount DECIMAL(15,2),
  
  -- Quality Check
  quality_checked_by_id UUID REFERENCES users(id),
  quality_check_date TIMESTAMP WITH TIME ZONE,
  quality_status VARCHAR(30) DEFAULT 'Pending',
  
  -- Status
  status VARCHAR(30) DEFAULT 'Pending',
  received_by_id UUID REFERENCES users(id),
  received_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  -- Inventory
  warehouse_id UUID REFERENCES warehouses(id),
  bin_id UUID REFERENCES warehouse_bins(id),
  batch_lot_number VARCHAR(50),
  
  -- Timestamps
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  INDEX idx_grn_deal (deal_id),
  INDEX idx_grn_status (status),
  INDEX idx_grn_supplier (supplier_id)
);

-- Inventory Transactions
CREATE TABLE inventory_transactions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  transaction_number VARCHAR(50) UNIQUE NOT NULL,
  transaction_type VARCHAR(30) NOT NULL CHECK (transaction_type IN (
    'GRN', 
    'Transfer', 
    'Adjustment', 
    'Production', 
    'Sales', 
    'Return'
  )),
  
  -- Links
  source_module VARCHAR(50) NOT NULL,
  source_record_id UUID NOT NULL,
  
  -- Inventory Details
  product_id UUID REFERENCES products(id),
  warehouse_id UUID REFERENCES warehouses(id),
  bin_id UUID REFERENCES warehouse_bins(id),
  
  quantity DECIMAL(15,2) NOT NULL,
  unit_price DECIMAL(15,2),
  total_amount DECIMAL(15,2),
  
  -- Batch Info
  batch_number VARCHAR(50),
  expiry_date DATE,
  
  -- Accounting
  accounting_entry_id UUID, -- Link to journal entry
  
  -- Timestamps
  transaction_date TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  created_by_id UUID REFERENCES users(id),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  INDEX idx_inv_trans_source (source_module, source_record_id),
  INDEX idx_inv_trans_product (product_id),
  INDEX idx_inv_trans_warehouse (warehouse_id),
  INDEX idx_inv_trans_date (transaction_date)
);
```

#### 3.1.6 Production Tables

```sql
-- Bill of Materials (BOM)
CREATE TABLE bom (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  bom_code VARCHAR(50) UNIQUE NOT NULL,
  product_id UUID REFERENCES products(id),
  version VARCHAR(10) DEFAULT '1.0',
  
  -- Yield Information
  input_product_id UUID REFERENCES products(id), -- Paddy
  input_quantity DECIMAL(15,2) NOT NULL,
  output_quantity DECIMAL(15,2) NOT NULL,
  
  -- By-products
  by_product_1_id UUID REFERENCES products(id), -- Husks
  by_product_1_quantity DECIMAL(15,2),
  by_product_2_id UUID REFERENCES products(id), -- Bran
  by_product_2_quantity DECIMAL(15,2),
  by_product_3_id UUID REFERENCES products(id), -- Broken
  by_product_3_quantity DECIMAL(15,2),
  
  -- Process Loss
  process_loss_percentage DECIMAL(5,2) DEFAULT 0,
  
  -- Status
  is_active BOOLEAN DEFAULT true,
  created_by_id UUID REFERENCES users(id),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Production Orders
CREATE TABLE production_orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_number VARCHAR(50) UNIQUE NOT NULL,
  bom_id UUID REFERENCES bom(id),
  
  -- Planning
  planned_quantity DECIMAL(15,2) NOT NULL,
  planned_start_date DATE NOT NULL,
  planned_end_date DATE NOT NULL,
  
  -- Resources
  machine_id UUID, -- Link to equipment
  assigned_to_team_id UUID, -- Link to team
  
  -- Actuals
  actual_quantity DECIMAL(15,2),
  actual_start_date DATE,
  actual_end_date DATE,
  
  -- Quality
  quality_checked BOOLEAN DEFAULT false,
  quality_score DECIMAL(5,2),
  
  -- Status
  status VARCHAR(30) DEFAULT 'Planned' CHECK (status IN (
    'Planned', 
    'In Progress', 
    'Paused', 
    'Completed', 
    'Cancelled'
  )),
  
  -- Tracking
  created_by_id UUID REFERENCES users(id),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  INDEX idx_prod_orders_status (status),
  INDEX idx_prod_orders_date (planned_start_date)
);
```

#### 3.1.7 Sales Tables

```sql
-- Sales Orders
CREATE TABLE sales_orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_number VARCHAR(50) UNIQUE NOT NULL,
  order_date DATE NOT NULL,
  customer_id UUID REFERENCES parties(id),
  
  -- Order Details
  product_id UUID REFERENCES products(id),
  quantity DECIMAL(15,2) NOT NULL,
  unit_price DECIMAL(15,2) NOT NULL,
  total_amount DECIMAL(15,2) NOT NULL,
  
  -- Discounts & Taxes
  discount_percentage DECIMAL(5,2) DEFAULT 0,
  tax_rate DECIMAL(5,2) DEFAULT 0,
  withholding_tax DECIMAL(5,2) DEFAULT 0,
  net_amount DECIMAL(15,2),
  
  -- Delivery
  delivery_date DATE,
  delivery_address TEXT,
  shipping_method VARCHAR(100),
  
  -- Status
  status VARCHAR(30) DEFAULT 'Draft' CHECK (status IN (
    'Draft', 
    'Confirmed', 
    'In Production', 
    'Ready for Dispatch', 
    'Dispatched', 
    'Delivered', 
    'Cancelled'
  )),
  
  -- Accounting
  invoice_id UUID, -- Link to invoice
  
  -- Timestamps
  created_by_id UUID REFERENCES users(id),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  INDEX idx_sales_customer (customer_id),
  INDEX idx_sales_status (status),
  INDEX idx_sales_date (order_date)
);

-- Export Orders
CREATE TABLE export_orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  export_order_number VARCHAR(50) UNIQUE NOT NULL,
  order_id UUID REFERENCES sales_orders(id),
  
  -- Export Details
  buyer_name VARCHAR(255) NOT NULL,
  buyer_country VARCHAR(100) NOT NULL,
  buyer_address TEXT,
  
  -- Shipping
  port_of_loading VARCHAR(255),
  port_of_destination VARCHAR(255),
  shipping_terms VARCHAR(50), -- FOB, CIF, etc.
  shipment_date DATE,
  container_number VARCHAR(50),
  vessel_name VARCHAR(255),
  
  -- Documentation
  letter_of_credit_number VARCHAR(50),
  lc_issue_date DATE,
  lc_expiry_date DATE,
  
  -- Financial
  foreign_currency_id UUID REFERENCES currencies(id),
  foreign_currency_amount DECIMAL(15,2),
  exchange_rate DECIMAL(15,6),
  local_currency_amount DECIMAL(15,2),
  
  -- Documentation Status
  certificate_of_origin_generated BOOLEAN DEFAULT false,
  bill_of_lading_generated BOOLEAN DEFAULT false,
  commercial_invoice_generated BOOLEAN DEFAULT false,
  packing_list_generated BOOLEAN DEFAULT false,
  
  -- Status
  status VARCHAR(30) DEFAULT 'Draft',
  
  -- Timestamps
  created_by_id UUID REFERENCES users(id),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  INDEX idx_export_order (order_id),
  INDEX idx_export_status (status)
);
```

#### 3.1.8 Finance & Accounting Tables

```sql
-- Journal Entries
CREATE TABLE journal_entries (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  entry_number VARCHAR(50) UNIQUE NOT NULL,
  entry_date DATE NOT NULL,
  entry_type VARCHAR(50) NOT NULL,
  description TEXT,
  total_debit DECIMAL(15,2) NOT NULL,
  total_credit DECIMAL(15,2) NOT NULL,
  
  -- Links
  source_module VARCHAR(50),
  source_record_id UUID,
  
  -- Status
  is_posted BOOLEAN DEFAULT false,
  posted_at TIMESTAMP WITH TIME ZONE,
  posted_by_id UUID REFERENCES users(id),
  
  -- Reversal
  is_reversal BOOLEAN DEFAULT false,
  reversal_of_entry_id UUID REFERENCES journal_entries(id),
  
  -- Timestamps
  created_by_id UUID REFERENCES users(id),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  INDEX idx_journal_source (source_module, source_record_id),
  INDEX idx_journal_date (entry_date),
  INDEX idx_journal_posted (is_posted)
);

-- Journal Entry Lines
CREATE TABLE journal_entry_lines (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  journal_entry_id UUID REFERENCES journal_entries(id) ON DELETE CASCADE,
  account_id UUID REFERENCES chart_of_accounts(id),
  cost_center_id UUID REFERENCES cost_centers(id),
  debit DECIMAL(15,2) DEFAULT 0,
  credit DECIMAL(15,2) DEFAULT 0,
  description TEXT,
  reference VARCHAR(255),
  
  INDEX idx_jel_journal (journal_entry_id),
  INDEX idx_jel_account (account_id)
);

-- Customer Invoices
CREATE TABLE customer_invoices (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  invoice_number VARCHAR(50) UNIQUE NOT NULL,
  invoice_date DATE NOT NULL,
  due_date DATE NOT NULL,
  
  -- Links
  order_id UUID REFERENCES sales_orders(id),
  customer_id UUID REFERENCES parties(id),
  
  -- Amounts
  gross_amount DECIMAL(15,2) NOT NULL,
  discount_amount DECIMAL(15,2) DEFAULT 0,
  tax_amount DECIMAL(15,2) DEFAULT 0,
  net_amount DECIMAL(15,2) NOT NULL,
  
  -- Payment
  payment_status VARCHAR(30) DEFAULT 'Unpaid' CHECK (payment_status IN (
    'Unpaid', 'Partially Paid', 'Paid', 'Overdue'
  )),
  paid_amount DECIMAL(15,2) DEFAULT 0,
  payment_date DATE,
  
  -- Accounting
  journal_entry_id UUID REFERENCES journal_entries(id),
  
  -- Timestamps
  created_by_id UUID REFERENCES users(id),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  INDEX idx_invoice_customer (customer_id),
  INDEX idx_invoice_status (payment_status),
  INDEX idx_invoice_due_date (due_date)
);

-- Vendor Invoices
CREATE TABLE vendor_invoices (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  invoice_number VARCHAR(50) NOT NULL,
  invoice_date DATE NOT NULL,
  due_date DATE NOT NULL,
  
  -- Links
  deal_id UUID REFERENCES paddy_deals(id),
  supplier_id UUID REFERENCES parties(id),
  
  -- Amounts
  gross_amount DECIMAL(15,2) NOT NULL,
  tax_amount DECIMAL(15,2) DEFAULT 0,
  withholding_tax DECIMAL(15,2) DEFAULT 0,
  net_amount DECIMAL(15,2) NOT NULL,
  
  -- Payment
  payment_status VARCHAR(30) DEFAULT 'Unpaid' CHECK (payment_status IN (
    'Unpaid', 'Partially Paid', 'Paid', 'Overdue'
  )),
  paid_amount DECIMAL(15,2) DEFAULT 0,
  payment_date DATE,
  
  -- Accounting
  journal_entry_id UUID REFERENCES journal_entries(id),
  
  -- Timestamps
  created_by_id UUID REFERENCES users(id),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  INDEX idx_vendor_invoice_supplier (supplier_id),
  INDEX idx_vendor_invoice_status (payment_status)
);
```

#### 3.1.9 HR Tables

```sql
-- Employees
CREATE TABLE employees (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  employee_code VARCHAR(50) UNIQUE NOT NULL,
  date_of_birth DATE,
  gender VARCHAR(20),
  marital_status VARCHAR(20),
  cnic VARCHAR(20),
  cnic_expiry DATE,
  
  -- Contact
  permanent_address TEXT,
  current_address TEXT,
  emergency_contact_name VARCHAR(255),
  emergency_contact_phone VARCHAR(20),
  
  -- Employment
  joining_date DATE NOT NULL,
  employment_type VARCHAR(30) CHECK (employment_type IN (
    'Permanent', 'Contract', 'Intern', 'Consultant'
  )),
  contract_end_date DATE,
  department_id UUID REFERENCES departments(id),
  designation VARCHAR(100),
  reporting_to_id UUID REFERENCES users(id),
  
  -- Salary
  salary_type VARCHAR(20) DEFAULT 'Monthly' CHECK (salary_type IN (
    'Monthly', 'Daily', 'Hourly'
  )),
  basic_salary DECIMAL(15,2),
  bank_account_id UUID REFERENCES party_bank_accounts(id),
  
  -- Documents
  documents JSONB,
  
  -- Status
  employment_status VARCHAR(30) DEFAULT 'Active' CHECK (employment_status IN (
    'Active', 'On Leave', 'Terminated', 'Resigned'
  )),
  leaving_date DATE,
  leaving_reason TEXT,
  
  -- Timestamps
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  INDEX idx_employee_code (employee_code),
  INDEX idx_employee_department (department_id),
  INDEX idx_employee_status (employment_status)
);

-- Attendance
CREATE TABLE attendance (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  employee_id UUID REFERENCES employees(id) ON DELETE CASCADE,
  attendance_date DATE NOT NULL,
  check_in_time TIMESTAMP WITH TIME ZONE,
  check_out_time TIMESTAMP WITH TIME ZONE,
  check_in_location VARCHAR(255),
  check_out_location VARCHAR(255),
  
  -- Status
  status VARCHAR(30) DEFAULT 'Present' CHECK (status IN (
    'Present', 'Absent', 'Late', 'Half Day', 'Leave', 'Holiday'
  )),
  
  -- Overtime
  overtime_hours DECIMAL(8,2) DEFAULT 0,
  
  -- Approval
  approved_by_id UUID REFERENCES users(id),
  approved_at TIMESTAMP WITH TIME ZONE,
  
  -- Notes
  notes TEXT,
  
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  INDEX idx_attendance_employee (employee_id),
  INDEX idx_attendance_date (attendance_date),
  UNIQUE(employee_id, attendance_date)
);

-- Leave Requests
CREATE TABLE leave_requests (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  employee_id UUID REFERENCES employees(id) ON DELETE CASCADE,
  leave_type VARCHAR(50) NOT NULL CHECK (leave_type IN (
    'Annual', 'Sick', 'Casual', 'Maternity', 'Paternity', 'Unpaid'
  )),
  start_date DATE NOT NULL,
  end_date DATE NOT NULL,
  total_days INTEGER NOT NULL,
  reason TEXT,
  
  -- Approval
  status VARCHAR(30) DEFAULT 'Pending' CHECK (status IN (
    'Pending', 'Approved', 'Rejected', 'Cancelled'
  )),
  approved_by_id UUID REFERENCES users(id),
  approved_at TIMESTAMP WITH TIME ZONE,
  rejection_reason TEXT,
  
  -- Timestamps
  requested_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  INDEX idx_leave_employee (employee_id),
  INDEX idx_leave_status (status),
  INDEX idx_leave_dates (start_date, end_date)
);

-- Payroll
CREATE TABLE payroll (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  payroll_period VARCHAR(50) NOT NULL, -- e.g., '2024-01'
  employee_id UUID REFERENCES employees(id) ON DELETE CASCADE,
  
  -- Earnings
  basic_salary DECIMAL(15,2) NOT NULL,
  allowances JSONB, -- House rent, Utilities, etc.
  overtime_amount DECIMAL(15,2) DEFAULT 0,
  bonus DECIMAL(15,2) DEFAULT 0,
  commission DECIMAL(15,2) DEFAULT 0,
  
  -- Deductions
  income_tax DECIMAL(15,2) DEFAULT 0,
  employee_pf DECIMAL(15,2) DEFAULT 0,
  employee_eobi DECIMAL(15,2) DEFAULT 0,
  advances_deduction DECIMAL(15,2) DEFAULT 0,
  loan_deduction DECIMAL(15,2) DEFAULT 0,
  other_deductions JSONB,
  
  -- Net
  gross_salary DECIMAL(15,2) GENERATED ALWAYS AS (
    basic_salary + COALESCE(allowances->>'total', '0')::DECIMAL + 
    COALESCE(overtime_amount, 0) + COALESCE(bonus, 0) + COALESCE(commission, 0)
  ) STORED,
  total_deductions DECIMAL(15,2) GENERATED ALWAYS AS (
    COALESCE(income_tax, 0) + COALESCE(employee_pf, 0) + 
    COALESCE(employee_eobi, 0) + COALESCE(advances_deduction, 0) + 
    COALESCE(loan_deduction, 0) + COALESCE(other_deductions->>'total', '0')::DECIMAL
  ) STORED,
  net_salary DECIMAL(15,2) GENERATED ALWAYS AS (
    (basic_salary + COALESCE(allowances->>'total', '0')::DECIMAL + 
     COALESCE(overtime_amount, 0) + COALESCE(bonus, 0) + COALESCE(commission, 0)) -
    (COALESCE(income_tax, 0) + COALESCE(employee_pf, 0) + 
     COALESCE(employee_eobi, 0) + COALESCE(advances_deduction, 0) + 
     COALESCE(loan_deduction, 0) + COALESCE(other_deductions->>'total', '0')::DECIMAL)
  ) STORED,
  
  -- Payment
  payment_status VARCHAR(30) DEFAULT 'Unpaid' CHECK (payment_status IN (
    'Unpaid', 'Processing', 'Paid', 'Failed'
  )),
  payment_date DATE,
  transaction_reference VARCHAR(100),
  
  -- Timestamps
  created_by_id UUID REFERENCES users(id),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  INDEX idx_payroll_period (payroll_period),
  INDEX idx_payroll_employee (employee_id),
  UNIQUE(payroll_period, employee_id)
);
```

#### 3.1.10 Analytics & Scoring Tables (NEW)

```sql
-- User Performance Scores
CREATE TABLE user_performance_scores (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  
  -- Score Components
  task_completion_score DECIMAL(5,2) DEFAULT 0,
  task_quality_score DECIMAL(5,2) DEFAULT 0,
  concern_value_score DECIMAL(5,2) DEFAULT 0,
  remark_insight_score DECIMAL(5,2) DEFAULT 0,
  collaboration_score DECIMAL(5,2) DEFAULT 0,
  
  -- Composite
  overall_score DECIMAL(5,2) GENERATED ALWAYS AS (
    (task_completion_score * 0.3 + task_quality_score * 0.3 + 
     concern_value_score * 0.15 + remark_insight_score * 0.15 + 
     collaboration_score * 0.1)
  ) STORED,
  
  -- Ranking
  percentile_rank INTEGER, -- 1-100
  
  -- Metadata
  evaluation_period VARCHAR(20) NOT NULL, -- e.g., '2024-Q1'
  calculated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  INDEX idx_perf_user (user_id),
  INDEX idx_perf_period (evaluation_period)
);

-- Concern Analysis (For Future NLP)
CREATE TABLE concern_analysis (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  comment_id UUID REFERENCES task_comments(id) ON DELETE CASCADE,
  sentiment_score DECIMAL(3,2),
  concern_category VARCHAR(50), -- Process, Resource, Management, etc.
  concern_severity VARCHAR(20) CHECK (concern_severity IN ('Low', 'Medium', 'High', 'Critical')),
  keywords TEXT[],
  analyzed_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  analyzed_by VARCHAR(50) DEFAULT 'AI' -- 'AI' or 'Manual'
);

-- Activity Audit Log
CREATE TABLE audit_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  action VARCHAR(50) NOT NULL,
  resource_type VARCHAR(50) NOT NULL,
  resource_id UUID NOT NULL,
  change_data JSONB,
  ip_address INET,
  user_agent TEXT,
  timestamp TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  INDEX idx_audit_user (user_id),
  INDEX idx_audit_resource (resource_type, resource_id),
  INDEX idx_audit_timestamp (timestamp)
);
```

### 3.2 Database Functions (120+)

```sql
-- Function to Calculate Task Score
CREATE OR REPLACE FUNCTION calculate_task_score(
  p_task_id UUID
)
RETURNS DECIMAL
LANGUAGE plpgsql
AS $$
DECLARE
  v_score DECIMAL;
  v_complexity DECIMAL;
  v_quality DECIMAL;
  v_timeliness DECIMAL;
  v_concerns DECIMAL;
BEGIN
  -- Get task details
  SELECT 
    complexity_score,
    quality_score,
    CASE 
      WHEN completed_at <= due_date THEN 1.0
      WHEN completed_at <= due_date + INTERVAL '3 days' THEN 0.7
      WHEN completed_at <= due_date + INTERVAL '7 days' THEN 0.4
      ELSE 0.1
    END as timeliness,
    (SELECT COUNT(*) FROM task_comments WHERE task_id = p_task_id AND is_concern = true) as concern_count
  INTO v_complexity, v_quality, v_timeliness, v_concerns
  FROM tasks 
  WHERE id = p_task_id;
  
  -- Calculate score
  v_score := (v_complexity * 0.3) + (v_quality * 0.4) + (v_timeliness * 0.2) + 
              (GREATEST(0, 1 - (v_concerns * 0.1)) * 0.1);
  
  -- Update task
  UPDATE tasks 
  SET task_score = v_score,
      contribution_score = v_score * (1 + (v_concerns * 0.05)) -- Bonus for concerns
  WHERE id = p_task_id;
  
  RETURN v_score;
END;
$$;

-- Function to Update User Performance Score
CREATE OR REPLACE FUNCTION update_user_performance_score(
  p_user_id UUID,
  p_period VARCHAR(20)
)
RETURNS VOID
LANGUAGE plpgsql
AS $$
DECLARE
  v_completion DECIMAL;
  v_quality DECIMAL;
  v_concern_value DECIMAL;
  v_remark_insight DECIMAL;
  v_collaboration DECIMAL;
BEGIN
  -- Calculate scores from tasks
  SELECT 
    AVG(CASE WHEN status = 'Done / Completed' THEN 1 ELSE 0 END) * 100,
    AVG(quality_score),
    COUNT(CASE WHEN is_concern = true THEN 1 END) / NULLIF(COUNT(*), 0) * 100,
    AVG(CASE WHEN LENGTH(comment) > 50 THEN 1 ELSE 0 END) * 100,
    COUNT(DISTINCT assigned_to_id) / NULLIF(COUNT(*), 0) * 100
  INTO v_completion, v_quality, v_concern_value, v_remark_insight, v_collaboration
  FROM tasks t
  LEFT JOIN task_comments tc ON tc.task_id = t.id
  WHERE (assigned_to_id = p_user_id OR created_by_id = p_user_id)
    AND created_at >= (CURRENT_DATE - INTERVAL '3 months');
  
  -- Insert or update performance record
  INSERT INTO user_performance_scores (
    user_id,
    task_completion_score,
    task_quality_score,
    concern_value_score,
    remark_insight_score,
    collaboration_score,
    evaluation_period,
    calculated_at
  ) VALUES (
    p_user_id,
    v_completion,
    v_quality,
    v_concern_value,
    v_remark_insight,
    v_collaboration,
    p_period,
    NOW()
  ) ON CONFLICT (user_id, evaluation_period) DO UPDATE SET
    task_completion_score = EXCLUDED.task_completion_score,
    task_quality_score = EXCLUDED.task_quality_score,
    concern_value_score = EXCLUDED.concern_value_score,
    remark_insight_score = EXCLUDED.remark_insight_score,
    collaboration_score = EXCLUDED.collaboration_score,
    calculated_at = EXCLUDED.calculated_at;
END;
$$;

-- Function to Auto-assign Task Number
CREATE OR REPLACE FUNCTION generate_task_number()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
  IF NEW.task_number IS NULL THEN
    NEW.task_number := 'TASK-' || TO_CHAR(NEW.created_at, 'YYYY') || '-' || 
                        LPAD(CAST(EXTRACT(YEAR FROM NEW.created_at) AS TEXT) || 
                             CAST(CEIL(RANDOM() * 10000) AS TEXT), 6, '0');
  END IF;
  RETURN NEW;
END;
$$;

CREATE TRIGGER trigger_generate_task_number
BEFORE INSERT ON tasks
FOR EACH ROW
EXECUTE FUNCTION generate_task_number();
```

### 3.3 Database Views (60+)

```sql
-- View for Task Dashboard
CREATE VIEW task_dashboard_view AS
SELECT 
  t.id,
  t.task_number,
  t.title,
  t.status,
  t.priority,
  t.due_date,
  t.pending_days,
  t.bucket,
  t.completion_percentage,
  t.task_score,
  u.first_name || ' ' || u.last_name AS assigned_to,
  d.name AS department,
  c.category_name AS category,
  jl.location_name AS job_location,
  EXTRACT(DAY FROM (NOW() - t.created_at)) AS age_days,
  COUNT(tc.id) AS comment_count
FROM tasks t
LEFT JOIN users u ON t.assigned_to_id = u.id
LEFT JOIN departments d ON u.department_id = d.id
LEFT JOIN task_categories c ON t.category_id = c.id
LEFT JOIN job_locations jl ON t.job_location_id = jl.id
LEFT JOIN task_comments tc ON tc.task_id = t.id
WHERE t.parent_task_id IS NULL
GROUP BY t.id, u.first_name, u.last_name, d.name, c.category_name, jl.location_name;

-- View for User Performance Summary
CREATE VIEW user_performance_summary AS
SELECT 
  u.id AS user_id,
  u.first_name || ' ' || u.last_name AS user_name,
  d.name AS department,
  COUNT(t.id) AS total_tasks,
  COUNT(CASE WHEN t.status = 'Done / Completed' THEN 1 END) AS completed_tasks,
  ROUND(AVG(t.task_score), 2) AS avg_task_score,
  ROUND(AVG(ups.overall_score), 2) AS current_performance_score,
  RANK() OVER (ORDER BY AVG(ups.overall_score) DESC) AS rank
FROM users u
LEFT JOIN tasks t ON t.assigned_to_id = u.id
LEFT JOIN user_performance_scores ups ON ups.user_id = u.id
LEFT JOIN departments d ON u.department_id = d.id
WHERE t.created_at >= (CURRENT_DATE - INTERVAL '90 days')
GROUP BY u.id, d.name;
```

### 3.4 RLS Policies (250+)

```sql
-- Example RLS Policies for Tasks

-- Users can view only their assigned tasks or tasks they created
CREATE POLICY "Users can view their own tasks"
ON tasks FOR SELECT
USING (
  auth.uid() = assigned_to_id 
  OR auth.uid() = created_by_id 
  OR auth.uid() IN (SELECT user_id FROM task_assignments WHERE task_id = tasks.id)
);

-- Department heads can view tasks in their department
CREATE POLICY "Department heads can view department tasks"
ON tasks FOR SELECT
USING (
  EXISTS (
    SELECT 1 FROM users u
    WHERE u.id = auth.uid()
    AND u.department_id = (
      SELECT department_id FROM users WHERE id = tasks.assigned_to_id
    )
    AND EXISTS (
      SELECT 1 FROM user_roles ur
      JOIN roles r ON ur.role_id = r.id
      WHERE ur.user_id = auth.uid()
      AND r.slug IN ('department_head', 'owner', 'super_admin')
    )
  )
);

-- Managers can insert tasks
CREATE POLICY "Managers can create tasks"
ON tasks FOR INSERT
WITH CHECK (
  EXISTS (
    SELECT 1 FROM user_roles ur
    JOIN roles r ON ur.role_id = r.id
    WHERE ur.user_id = auth.uid()
    AND r.slug IN ('manager', 'department_head', 'owner', 'super_admin')
  )
);

-- Only assigned users or managers can update tasks
CREATE POLICY "Assigned users and managers can update tasks"
ON tasks FOR UPDATE
USING (
  auth.uid() = assigned_to_id 
  OR EXISTS (
    SELECT 1 FROM user_roles ur
    JOIN roles r ON ur.role_id = r.id
    WHERE ur.user_id = auth.uid()
    AND r.slug IN ('manager', 'department_head', 'owner', 'super_admin')
  )
);
```

### 3.5 Materialized Views for Analytics

```sql
-- Materialized View for Monthly Performance
CREATE MATERIALIZED VIEW monthly_performance_mv AS
SELECT 
  DATE_TRUNC('month', created_at) AS month,
  COUNT(id) AS total_tasks,
  COUNT(CASE WHEN status = 'Done / Completed' THEN 1 END) AS completed_tasks,
  AVG(task_score) AS avg_score,
  COUNT(CASE WHEN priority = 'Urgent/Overdue' THEN 1 END) AS urgent_tasks,
  AVG(pending_days) AS avg_pending_days
FROM tasks
GROUP BY DATE_TRUNC('month', created_at)
WITH DATA;

CREATE UNIQUE INDEX idx_monthly_performance_month ON monthly_performance_mv(month);
REFRESH MATERIALIZED VIEW CONCURRENTLY monthly_performance_mv;
```

---

## 4. FRONTEND ARCHITECTURE

### 4.1 Project Structure

```
src/
├── app/                                   # Next.js App Router
│   ├── (auth)/                           # Authentication routes
│   │   ├── login/
│   │   ├── register/
│   │   ├── forgot-password/
│   │   └── reset-password/
│   ├── (dashboard)/                      # Protected dashboard routes
│   │   ├── layout.tsx                   # Dashboard layout with sidebar
│   │   ├── page.tsx                     # Main dashboard
│   │   ├── procurement/
│   │   ├── production/
│   │   ├── sales/
│   │   ├── inventory/
│   │   ├── finance/
│   │   ├── hr/
│   │   ├── tasks/                       # NEW: Task Management Module
│   │   │   ├── board/
│   │   │   │   └── page.tsx
│   │   │   ├── list/
│   │   │   │   └── page.tsx
│   │   │   ├── create/
│   │   │   │   └── page.tsx
│   │   │   └── [id]/
│   │   │       └── page.tsx
│   │   ├── requests/                    # NEW: Request Management
│   │   ├── documents/                   # NEW: Document Management
│   │   ├── analytics/
│   │   └── settings/
│   ├── api/                             # API routes
│   │   ├── auth/
│   │   ├── tasks/
│   │   ├── requests/
│   │   └── ...
│   ├── layout.tsx                       # Root layout
│   └── providers.tsx                    # Providers wrapper
├── components/
│   ├── ui/                              # 60+ reusable UI components
│   │   ├── Button/
│   │   ├── Card/
│   │   ├── Table/
│   │   ├── Modal/
│   │   ├── Form/
│   │   ├── Input/
│   │   ├── Select/
│   │   ├── DatePicker/
│   │   ├── FileUpload/
│   │   ├── Avatar/
│   │   ├── Badge/
│   │   ├── Progress/
│   │   ├── Tabs/
│   │   ├── Alert/
│   │   ├── Toast/
│   │   └── ...
│   ├── forms/                           # 40+ form components
│   │   ├── TaskForm/
│   │   ├── RequestForm/
│   │   ├── DealForm/
│   │   └── ...
│   ├── tables/                          # 20+ table components
│   │   ├── TaskTable/
│   │   ├── EmployeeTable/
│   │   └── ...
│   ├── charts/                          # 15+ chart types
│   │   ├── BarChart/
│   │   ├── LineChart/
│   │   ├── PieChart/
│   │   ├── GanttChart/                  # For project tasks
│   │   └── ...
│   ├── layouts/                         # Layout components
│   │   ├── Sidebar/
│   │   ├── Header/
│   │   ├── Footer/
│   │   └── ...
│   ├── tasks/                           # Task-specific components
│   │   ├── TaskBoard/
│   │   │   ├── TaskBoardView.tsx
│   │   │   ├── TaskColumn.tsx
│   │   │   ├── TaskCard.tsx
│   │   │   └── TaskDropzone.tsx
│   │   ├── TaskList/
│   │   │   ├── TaskListView.tsx
│   │   │   ├── TaskFilters.tsx
│   │   │   └── TaskSort.tsx
│   │   ├── TaskDetail/
│   │   │   ├── TaskHeader.tsx
│   │   │   ├── TaskInfo.tsx
│   │   │   ├── TaskComments.tsx
│   │   │   ├── TaskAttachments.tsx
│   │   │   ├── TaskTimeline.tsx
│   │   │   └── TaskActions.tsx
│   │   └── TaskForm/
│   │       ├── TaskFormWizard.tsx
│   │       ├── TaskBasicInfo.tsx
│   │       ├── TaskAssignment.tsx
│   │       ├── TaskDependencies.tsx
│   │       └── TaskReview.tsx
│   ├── requests/                        # Request-specific components
│   │   ├── RequestForm/
│   │   ├── ApprovalFlow/
│   │   └── ApprovalQueue/
│   └── documents/                       # Document-specific components
│       ├── DocumentUploader/
│       ├── DocumentViewer/
│       └── DocumentManager/
├── lib/
│   ├── supabase/                        # Supabase client
│   │   ├── client.ts
│   │   ├── server.ts
│   │   └── admin.ts
│   ├── validation/                      # Zod schemas
│   │   ├── task.schema.ts
│   │   ├── request.schema.ts
│   │   ├── deal.schema.ts
│   │   └── ...
│   ├── hooks/                           # Custom React hooks
│   │   ├── useAuth/
│   │   ├── useTasks/
│   │   ├── useRequests/
│   │   ├── useDocuments/
│   │   ├── useDebounce/
│   │   ├── usePagination/
│   │   ├── useSearch/
│   │   └── ...
│   ├── utils/                           # Helper functions
│   │   ├── date.ts
│   │   ├── format.ts
│   │   ├── validation.ts
│   │   ├── constants.ts
│   │   └── ...
│   ├── constants/                       # App constants
│   │   ├── roles.ts
│   │   ├── statuses.ts
│   │   ├── priorities.ts
│   │   └── ...
│   └── config/                          # Configuration
│       ├── site.ts
│       ├── auth.ts
│       └── navigation.ts
├── services/
│   ├── api/                             # API service layer
│   │   ├── task.service.ts
│   │   ├── request.service.ts
│   │   ├── document.service.ts
│   │   ├── auth.service.ts
│   │   ├── user.service.ts
│   │   ├── procurement.service.ts
│   │   ├── production.service.ts
│   │   ├── sales.service.ts
│   │   ├── inventory.service.ts
│   │   ├── finance.service.ts
│   │   └── ...
│   ├── auth/                            # Auth services
│   │   ├── auth.service.ts
│   │   ├── rbac.service.ts
│   │   └── session.service.ts
│   ├── reporting/                       # Report generation
│   │   ├── report.service.ts
│   │   ├── pdf.service.ts
│   │   └── excel.service.ts
│   └── integration/                     # Third-party integrations
│       ├── quickbooks.service.ts
│       ├── banking.service.ts
│       ├── fbr.service.ts
│       ├── email.service.ts
│       └── sms.service.ts
├── stores/                              # Zustand state stores
│   ├── auth.store.ts
│   ├── task.store.ts
│   ├── request.store.ts
│   ├── notification.store.ts
│   ├── theme.store.ts
│   └── ...
├── types/                               # TypeScript type definitions
│   ├── user.types.ts
│   ├── task.types.ts
│   ├── request.types.ts
│   ├── document.types.ts
│   ├── deal.types.ts
│   └── ...
└── styles/                              # Tailwind configuration
    ├── globals.css
    ├── theme.css
    └── components.css
```

### 4.2 Core Frontend Components

#### 4.2.1 Task Board Component

```typescript
// TaskBoardView.tsx
import { DragDropContext, Droppable, Draggable } from 'react-beautiful-dnd';
import { TaskColumn } from './TaskColumn';
import { useTaskStore } from '@/stores/task.store';

export const TaskBoardView = () => {
  const { tasks, updateTaskStatus } = useTaskStore();
  
  // Group tasks by status
  const columns = {
    'Pending': tasks.filter(t => t.status === 'Pending'),
    'In Progress': tasks.filter(t => t.status === 'In Progress'),
    'Completed': tasks.filter(t => t.status === 'Done / Completed'),
    'On Hold': tasks.filter(t => t.status === 'Cancelled / On Hold'),
  };

  const handleDragEnd = (result) => {
    if (!result.destination) return;
    
    const { draggableId, destination } = result;
    const newStatus = destination.droppableId;
    
    updateTaskStatus(draggableId, newStatus);
  };

  return (
    <DragDropContext onDragEnd={handleDragEnd}>
      <div className="flex gap-4 overflow-x-auto p-4">
        {Object.entries(columns).map(([status, tasks]) => (
          <div key={status} className="w-80 shrink-0">
            <TaskColumn 
              status={status} 
              tasks={tasks} 
            />
          </div>
        ))}
      </div>
    </DragDropContext>
  );
};

// TaskColumn.tsx
export const TaskColumn = ({ status, tasks }) => {
  return (
    <div className="bg-gray-50 rounded-lg p-3">
      <div className="flex items-center justify-between mb-3">
        <h3 className="font-medium text-gray-700">
          {status}
          <span className="ml-2 text-sm text-gray-500">{tasks.length}</span>
        </h3>
      </div>
      <Droppable droppableId={status}>
        {(provided) => (
          <div
            ref={provided.innerRef}
            {...provided.droppableProps}
            className="space-y-2 min-h-[200px]"
          >
            {tasks.map((task, index) => (
              <Draggable key={task.id} draggableId={task.id} index={index}>
                {(provided, snapshot) => (
                  <div
                    ref={provided.innerRef}
                    {...provided.draggableProps}
                    {...provided.dragHandleProps}
                    className={`p-4 bg-white rounded-md shadow-sm hover:shadow-md transition-shadow ${
                      snapshot.isDragging ? 'shadow-lg' : ''
                    }`}
                  >
                    <TaskCard task={task} />
                  </div>
                )}
              </Draggable>
            ))}
            {provided.placeholder}
          </div>
        )}
      </Droppable>
    </div>
  );
};

// TaskCard.tsx
export const TaskCard = ({ task }) => {
  const getPriorityColor = (priority) => {
    const colors = {
      'Low': 'bg-blue-100 text-blue-800',
      'Medium': 'bg-yellow-100 text-yellow-800',
      'High': 'bg-orange-100 text-orange-800',
      'Urgent/Overdue': 'bg-red-100 text-red-800',
    };
    return colors[priority] || 'bg-gray-100 text-gray-800';
  };

  return (
    <div className="cursor-pointer">
      <div className="flex items-start justify-between">
        <h4 className="text-sm font-medium text-gray-900">{task.title}</h4>
        <span className={`text-xs px-2 py-1 rounded-full ${getPriorityColor(task.priority)}`}>
          {task.priority}
        </span>
      </div>
      <p className="text-sm text-gray-500 mt-1 line-clamp-2">{task.description}</p>
      <div className="flex items-center justify-between mt-2">
        <div className="flex items-center gap-2">
          <Avatar user={task.assigned_to} size="sm" />
          <span className="text-xs text-gray-600">{task.assigned_to?.name}</span>
        </div>
        <div className="flex items-center gap-2 text-xs text-gray-500">
          <CalendarIcon className="w-3 h-3" />
          <span>{formatDate(task.due_date)}</span>
        </div>
      </div>
      {task.completion_percentage > 0 && (
        <Progress value={task.completion_percentage} className="mt-2" />
      )}
    </div>
  );
};
```

#### 4.2.2 Task List View with Advanced Filters

```typescript
// TaskListView.tsx
import { useTaskFilters } from '@/hooks/useTaskFilters';
import { TaskTable } from './TaskTable';
import { TaskFilters } from './TaskFilters';

export const TaskListView = () => {
  const { filters, setFilters, pagination, setPagination } = useTaskFilters();
  const { data: tasks, isLoading } = useTasks({ filters, pagination });

  return (
    <div className="space-y-4">
      <TaskFilters filters={filters} onFilterChange={setFilters} />
      <TaskTable 
        tasks={tasks} 
        pagination={pagination}
        onPaginationChange={setPagination}
        isLoading={isLoading}
      />
    </div>
  );
};

// TaskFilters.tsx
export const TaskFilters = ({ filters, onFilterChange }) => {
  return (
    <div className="bg-white p-4 rounded-lg shadow space-y-4">
      <div className="grid grid-cols-1 md:grid-cols-4 gap-4">
        <Input
          placeholder="Search tasks..."
          value={filters.search}
          onChange={(e) => onFilterChange({ search: e.target.value })}
          leftIcon={<SearchIcon />}
        />
        <Select
          placeholder="Category"
          value={filters.category}
          onChange={(value) => onFilterChange({ category: value })}
          options={taskCategories}
        />
        <Select
          placeholder="Status"
          value={filters.status}
          onChange={(value) => onFilterChange({ status: value })}
          options={taskStatuses}
        />
        <Select
          placeholder="Priority"
          value={filters.priority}
          onChange={(value) => onFilterChange({ priority: value })}
          options={priorities}
        />
        <DatePicker
          value={filters.dueDate}
          onChange={(date) => onFilterChange({ dueDate: date })}
          placeholder="Due Date"
        />
        <Select
          placeholder="Assigned To"
          value={filters.assignedTo}
          onChange={(value) => onFilterChange({ assignedTo: value })}
          options={users.map(u => ({ label: u.name, value: u.id }))}
        />
      </div>
      <div className="flex items-center justify-between">
        <div className="flex items-center gap-2">
          <Button variant="outline" size="sm">Reset Filters</Button>
          <Button variant="outline" size="sm">Save Filter</Button>
        </div>
        <div className="flex items-center gap-2">
          <Button variant="primary" size="sm">
            <PlusIcon className="w-4 h-4 mr-2" />
            New Task
          </Button>
        </div>
      </div>
    </div>
  );
};
```

#### 4.2.3 Task Detail Component with Comments

```typescript
// TaskDetail.tsx
export const TaskDetail = ({ taskId }) => {
  const { task, isLoading } = useTask(taskId);
  const { comments, addComment } = useTaskComments(taskId);
  const [newComment, setNewComment] = useState('');
  const [isConcern, setIsConcern] = useState(false);

  if (isLoading) return <LoadingSpinner />;

  return (
    <div className="max-w-6xl mx-auto p-6">
      <div className="grid grid-cols-3 gap-6">
        {/* Main Content */}
        <div className="col-span-2 space-y-6">
          <TaskHeader task={task} />
          <TaskInfo task={task} />
          
          {/* Comments Section */}
          <div className="bg-white rounded-lg shadow p-6">
            <h3 className="text-lg font-medium mb-4">
              Comments & Remarks
              <span className="ml-2 text-sm text-gray-500">
                ({comments.length})
              </span>
            </h3>
            
            <div className="space-y-4 max-h-96 overflow-y-auto">
              {comments.map((comment) => (
                <CommentItem key={comment.id} comment={comment} />
              ))}
            </div>
            
            <div className="mt-4 border-t pt-4">
              <div className="flex items-start gap-3">
                <Avatar user={currentUser} />
                <div className="flex-1">
                  <textarea
                    className="w-full p-2 border rounded-lg resize-none"
                    rows={3}
                    placeholder="Add your comment or raise a concern..."
                    value={newComment}
                    onChange={(e) => setNewComment(e.target.value)}
                  />
                  <div className="flex items-center justify-between mt-2">
                    <label className="flex items-center gap-2 text-sm text-gray-600">
                      <input
                        type="checkbox"
                        checked={isConcern}
                        onChange={(e) => setIsConcern(e.target.checked)}
                      />
                      This is a concern
                    </label>
                    <div className="flex gap-2">
                      <Button
                        variant="outline"
                        onClick={() => setNewComment('')}
                      >
                        Cancel
                      </Button>
                      <Button
                        variant={isConcern ? 'danger' : 'primary'}
                        onClick={() => {
                          addComment({ content: newComment, isConcern });
                          setNewComment('');
                          setIsConcern(false);
                        }}
                      >
                        {isConcern ? 'Raise Concern' : 'Add Comment'}
                      </Button>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
        
        {/* Sidebar */}
        <div className="col-span-1 space-y-6">
          <TaskStatusActions task={task} />
          <TaskAttachments task={task} />
          <TaskMetadata task={task} />
          <TaskDependencies task={task} />
        </div>
      </div>
    </div>
  );
};

// CommentItem.tsx
export const CommentItem = ({ comment }) => {
  return (
    <div className={`flex gap-3 ${comment.is_concern ? 'bg-red-50 p-3 rounded-lg' : ''}`}>
      <Avatar user={comment.user} size="sm" />
      <div className="flex-1">
        <div className="flex items-center gap-2">
          <span className="font-medium">{comment.user.name}</span>
          <span className="text-xs text-gray-500">{formatRelative(comment.created_at)}</span>
          {comment.is_concern && (
            <Badge variant="danger" className="text-xs">Concern</Badge>
          )}
        </div>
        <p className="text-sm text-gray-700 mt-1">{comment.content}</p>
        {comment.is_concern && comment.sentiment_score && (
          <div className="mt-1">
            <Progress value={comment.sentiment_score * 100} size="sm" />
          </div>
        )}
      </div>
    </div>
  );
};
```

#### 4.2.4 Request Form with Approval Workflow

```typescript
// RequestFormWizard.tsx
export const RequestFormWizard = ({ onComplete }) => {
  const [step, setStep] = useState(1);
  const [formData, setFormData] = useState({
    type: '',
    title: '',
    description: '',
    priority: 'Medium',
    department: '',
    // Type-specific data
    requestData: {}
  });

  const requestTypes = [
    { id: 'new_hire', label: 'New Hire Request', icon: UserPlusIcon },
    { id: 'equipment', label: 'Equipment Purchase', icon: ToolIcon },
    { id: 'budget', label: 'Budget Approval', icon: DollarIcon },
    { id: 'maintenance', label: 'Maintenance Request', icon: WrenchIcon },
  ];

  const renderStep = () => {
    switch(step) {
      case 1:
        return <RequestTypeSelector 
          types={requestTypes} 
          selected={formData.type}
          onSelect={(type) => updateForm('type', type)}
        />;
      case 2:
        return <RequestBasicInfo 
          data={formData}
          onChange={updateForm}
        />;
      case 3:
        return <RequestDetails 
          type={formData.type}
          data={formData.requestData}
          onChange={(data) => updateForm('requestData', data)}
        />;
      case 4:
        return <RequestReview data={formData} />;
      default:
        return null;
    }
  };

  const handleSubmit = async () => {
    try {
      await createRequest(formData);
      toast.success('Request submitted for approval');
      onComplete();
    } catch (error) {
      toast.error('Failed to submit request');
    }
  };

  return (
    <div className="bg-white rounded-lg shadow p-6">
      <div className="mb-8">
        <Stepper currentStep={step} totalSteps={4} />
      </div>
      
      <div className="min-h-[400px]">
        {renderStep()}
      </div>
      
      <div className="flex justify-between mt-8 border-t pt-4">
        <Button 
          variant="outline" 
          onClick={() => setStep(step - 1)}
          disabled={step === 1}
        >
          Back
        </Button>
        <div className="flex gap-3">
          {step === 4 ? (
            <>
              <Button variant="outline" onClick={() => setStep(1)}>
                Edit
              </Button>
              <Button variant="primary" onClick={handleSubmit}>
                Submit Request
              </Button>
            </>
          ) : (
            <Button 
              variant="primary" 
              onClick={() => setStep(step + 1)}
            >
              Next
            </Button>
          )}
        </div>
      </div>
    </div>
  );
};
```

---

## 5. BACKEND SERVICES & APIS

### 5.1 API Service Layer

```typescript
// services/api/task.service.ts
export class TaskService {
  private readonly supabase = createSupabaseClient();
  private readonly cache = new RedisCache();

  async getTasks(filters: TaskFilters, pagination: Pagination) {
    const cacheKey = `tasks:${JSON.stringify({ filters, pagination })}`;
    const cached = await this.cache.get(cacheKey);
    
    if (cached) return cached;

    let query = this.supabase
      .from('tasks')
      .select(`
        *,
        assigned_to:assigned_to_id(*),
        created_by:created_by_id(*),
        category:category_id(*),
        job_location:job_location_id(*),
        comments:task_comments(count)
      `, { count: 'exact' });

    // Apply filters
    if (filters.search) {
      query = query.or(`title.ilike.%${filters.search}%,description.ilike.%${filters.search}%`);
    }
    if (filters.status) {
      query = query.eq('status', filters.status);
    }
    if (filters.priority) {
      query = query.eq('priority', filters.priority);
    }
    if (filters.assignedTo) {
      query = query.eq('assigned_to_id', filters.assignedTo);
    }
    if (filters.category) {
      query = query.eq('category_id', filters.category);
    }
    if (filters.dueDate) {
      query = query.lte('due_date', filters.dueDate);
    }
    if (filters.department) {
      query = query.eq('departments.id', filters.department);
    }

    // Apply sorting
    const sortField = filters.sortBy || 'created_at';
    const sortOrder = filters.sortOrder || 'desc';
    query = query.order(sortField, { ascending: sortOrder === 'asc' });

    // Apply pagination
    const from = (pagination.page - 1) * pagination.limit;
    const to = from + pagination.limit - 1;
    query = query.range(from, to);

    const { data, error, count } = await query;

    if (error) throw new Error(`Failed to fetch tasks: ${error.message}`);

    const result = { tasks: data, total: count };
    await this.cache.set(cacheKey, result, { ttl: 60 }); // Cache for 1 minute

    return result;
  }

  async createTask(taskData: CreateTaskInput): Promise<Task> {
    // Validate input
    const validated = taskSchema.parse(taskData);

    // Check if task should be auto-assigned
    let assignedTo = validated.assignedTo;
    if (!assignedTo) {
      assignedTo = await this.autoAssignTask(validated);
    }

    const { data, error } = await this.supabase
      .from('tasks')
      .insert({
        ...validated,
        assigned_to_id: assignedTo,
        task_number: await this.generateTaskNumber(),
        created_by_id: getCurrentUser().id,
        created_at: new Date().toISOString(),
        pending_days: this.calculatePendingDays(validated.dueDate),
        bucket: this.calculateBucket(validated.dueDate)
      })
      .select()
      .single();

    if (error) throw new Error(`Failed to create task: ${error.message}`);

    // Create audit log
    await this.createAuditLog(data.id, 'CREATE', data);

    // Send notifications
    await this.sendTaskNotifications(data);

    // Invalidate cache
    await this.cache.deletePattern('tasks:*');

    return data;
  }

  async updateTaskStatus(taskId: string, newStatus: string, comment?: string) {
    const { data, error } = await this.supabase
      .from('tasks')
      .update({
        status: newStatus,
        completed_at: newStatus === 'Done / Completed' ? new Date().toISOString() : null,
        completion_percentage: newStatus === 'Done / Completed' ? 100 : 
                               newStatus === 'In Progress' ? 50 : 0,
        updated_at: new Date().toISOString()
      })
      .eq('id', taskId)
      .select()
      .single();

    if (error) throw new Error(`Failed to update task status: ${error.message}`);

    // Record status history
    await this.supabase
      .from('task_status_history')
      .insert({
        task_id: taskId,
        new_status: newStatus,
        changed_by_id: getCurrentUser().id,
        comment: comment
      });

    // If completed, calculate score
    if (newStatus === 'Done / Completed') {
      await this.calculateTaskScore(taskId);
    }

    // Invalidate cache
    await this.cache.delete(`task:${taskId}`);
    await this.cache.deletePattern('tasks:*');

    return data;
  }

  async addComment(taskId: string, content: string, isConcern: boolean) {
    const { data, error } = await this.supabase
      .from('task_comments')
      .insert({
        task_id: taskId,
        user_id: getCurrentUser().id,
        comment: content,
        is_concern: isConcern,
        comment_type: isConcern ? 'Concern' : 'Regular',
        created_at: new Date().toISOString()
      })
      .select()
      .single();

    if (error) throw new Error(`Failed to add comment: ${error.message}`);

    // If concern, analyze and update user score
    if (isConcern) {
      await this.analyzeConcern(data);
      await this.updateUserScore(getCurrentUser().id);
    }

    return data;
  }

  async calculateTaskScore(taskId: string): Promise<number> {
    const { data, error } = await this.supabase
      .rpc('calculate_task_score', { p_task_id: taskId });

    if (error) throw new Error(`Failed to calculate task score: ${error.message}`);

    return data;
  }

  private async autoAssignTask(taskData: CreateTaskInput): Promise<string> {
    // Logic to automatically assign task based on:
    // 1. Category/Type
    // 2. Current workload
    // 3. Expertise
    // 4. Performance score
    
    const { data } = await this.supabase
      .from('users')
      .select(`
        id,
        expertise,
        performance_score,
        tasks:assigned_to_id(count)
      `)
      .eq('department_id', taskData.department)
      .eq('is_active', true)
      .order('performance_score', { ascending: false })
      .limit(5);

    // Find best fit based on expertise match and workload
    // ... complex matching logic

    return bestFitUser.id;
  }

  private async generateTaskNumber(): Promise<string> {
    const year = new Date().getFullYear();
    const { count } = await this.supabase
      .from('tasks')
      .select('*', { count: 'exact', head: true })
      .gte('created_at', `${year}-01-01`)
      .lte('created_at', `${year}-12-31`);

    return `TASK-${year}-${String((count || 0) + 1).padStart(6, '0')}`;
  }

  private calculatePendingDays(dueDate: string): number {
    const due = new Date(dueDate);
    const now = new Date();
    const diff = Math.ceil((due.getTime() - now.getTime()) / (1000 * 60 * 60 * 24));
    return Math.max(0, diff);
  }

  private calculateBucket(dueDate: string): string {
    const days = this.calculatePendingDays(dueDate);
    if (days <= 5) return '0 to 5 days';
    if (days <= 10) return '5 to 10 days';
    if (days <= 15) return '10 to 15 days';
    if (days <= 20) return '15 to 20 days';
    if (days <= 25) return '20 to 25 days';
    if (days <= 30) return '25 to 30 days';
    return 'Above 30 days';
  }

  private async sendTaskNotifications(task: Task) {
    // Email to assignee
    await sendEmail({
      to: task.assigned_to.email,
      subject: `New Task Assigned: ${task.task_number}`,
      template: 'task-assignment',
      data: { task }
    });

    // In-app notification
    await this.createNotification({
      user_id: task.assigned_to_id,
      title: 'New Task Assigned',
      message: `${task.title} has been assigned to you`,
      type: 'task_assignment',
      link: `/tasks/${task.id}`
    });

    // SMS if urgent
    if (task.priority === 'Urgent/Overdue') {
      await sendSMS({
        to: task.assigned_to.phone,
        message: `URGENT: New task assigned - ${task.title}. Due: ${task.due_date}`
      });
    }
  }

  private async createAuditLog(taskId: string, action: string, data: any) {
    await this.supabase
      .from('audit_logs')
      .insert({
        user_id: getCurrentUser().id,
        action: action,
        resource_type: 'task',
        resource_id: taskId,
        change_data: data,
        timestamp: new Date().toISOString()
      });
  }
}
```

### 5.2 Request Management Service

```typescript
// services/api/request.service.ts
export class RequestService {
  private readonly supabase = createSupabaseClient();

  async createRequest(input: CreateRequestInput): Promise<Request> {
    // Create task first
    const taskService = new TaskService();
    const task = await taskService.createTask({
      title: input.title,
      description: input.description,
      category: 'request',
      priority: input.priority,
      department: input.department,
      // ... other fields
    });

    // Create request
    const { data, error } = await this.supabase
      .from('requests')
      .insert({
        task_id: task.id,
        request_type_id: input.typeId,
        request_data: input.requestData,
        department_id: input.department,
        requested_by_id: getCurrentUser().id,
        status: 'Submitted',
        submitted_at: new Date().toISOString()
      })
      .select()
      .single();

    if (error) throw new Error(`Failed to create request: ${error.message}`);

    // Start approval workflow
    await this.startApprovalWorkflow(data.id);

    return data;
  }

  async startApprovalWorkflow(requestId: string) {
    const { data: request } = await this.supabase
      .from('requests')
      .select('request_type_id, department_id')
      .eq('id', requestId)
      .single();

    // Get workflow steps
    const { data: steps } = await this.supabase
      .from('approval_workflow_steps')
      .select(`
        *,
        approver:approver_user_id(*),
        role:approver_role_id(*)
      `)
      .eq('workflow_id', request.request_type.approval_workflow_id)
      .order('step_order', { ascending: true });

    // Create approval records
    for (const step of steps) {
      const approverId = step.approver_user_id || 
                         await this.getDefaultApprover(step.approver_role_id, request.department_id);

      await this.supabase
        .from('approvals')
        .insert({
          request_id: requestId,
          workflow_step_id: step.id,
          approver_id: approverId,
          status: 'Pending'
        });

      // Send approval notification
      await this.sendApprovalNotification(requestId, approverId);
    }
  }

  async approveRequest(approvalId: string, comments?: string) {
    const { data, error } = await this.supabase
      .from('approvals')
      .update({
        status: 'Approved',
        comments: comments,
        approved_at: new Date().toISOString()
      })
      .eq('id', approvalId)
      .select()
      .single();

    if (error) throw new Error(`Failed to approve request: ${error.message}`);

    // Check if all approvals are done
    const { data: allApprovals } = await this.supabase
      .from('approvals')
      .select('status')
      .eq('request_id', data.request_id);

    const allApproved = allApprovals.every(a => a.status === 'Approved');

    if (allApproved) {
      await this.completeRequest(data.request_id);
    }

    return data;
  }

  async completeRequest(requestId: string) {
    // Update request status
    await this.supabase
      .from('requests')
      .update({
        status: 'Approved',
        approved_by_id: getCurrentUser().id,
        approval_date: new Date().toISOString()
      })
      .eq('id', requestId);

    // Update linked task
    await this.supabase
      .from('tasks')
      .update({
        status: 'Done / Completed',
        approval_status: 'Approved',
        approved_by_id: getCurrentUser().id
      })
      .eq('id', (await this.getRequestTaskId(requestId)));

    // If new hire request, create employee record
    const { data: request } = await this.supabase
      .from('requests')
      .select('request_type_id, request_data')
      .eq('id', requestId)
      .single();

    if (request.request_type_id === 'new_hire') {
      await this.createEmployeeFromRequest(requestId, request.request_data);
    }
  }

  private async createEmployeeFromRequest(requestId: string, data: any) {
    // Create employee record from request data
    // ... implementation
  }
}
```

### 5.3 Document Management Service

```typescript
// services/api/document.service.ts
export class DocumentService {
  private readonly supabase = createSupabaseClient();
  private readonly storage = createStorageClient();

  async uploadDocument(
    file: File,
    metadata: DocumentMetadata
  ): Promise<Document> {
    // Generate unique filename
    const fileExt = file.name.split('.').pop();
    const fileName = `${uuidv4()}.${fileExt}`;
    const filePath = `${metadata.source_module}/${metadata.source_record_id}/${fileName}`;

    // Upload to storage
    const { data: storageData, error: uploadError } = await this.storage
      .from('documents')
      .upload(filePath, file, {
        cacheControl: '3600',
        upsert: false,
        contentType: file.type
      });

    if (uploadError) throw new Error(`Failed to upload file: ${uploadError.message}`);

    // Get public URL
    const { data: urlData } = this.storage
      .from('documents')
      .getPublicUrl(filePath);

    // Create document record
    const { data, error } = await this.supabase
      .from('documents')
      .insert({
        document_number: await this.generateDocumentNumber(),
        document_type_id: metadata.documentTypeId,
        document_name: metadata.documentName || file.name,
        description: metadata.description,
        source_module: metadata.sourceModule,
        source_record_id: metadata.sourceRecordId,
        file_name: file.name,
        file_url: urlData.publicUrl,
        file_type: file.type,
        file_size: file.size,
        mime_type: file.type,
        file_hash: await this.calculateFileHash(file),
        uploaded_by_id: getCurrentUser().id,
        uploaded_at: new Date().toISOString(),
        status: metadata.status || 'Draft'
      })
      .select()
      .single();

    if (error) throw new Error(`Failed to create document record: ${error.message}`);

    return data;
  }

  async getDocumentsBySource(sourceModule: string, sourceRecordId: string) {
    const { data, error } = await this.supabase
      .from('documents')
      .select(`
        *,
        uploaded_by:uploaded_by_id(*),
        document_type:document_type_id(*)
      `)
      .eq('source_module', sourceModule)
      .eq('source_record_id', sourceRecordId)
      .order('uploaded_at', { ascending: false });

    if (error) throw new Error(`Failed to fetch documents: ${error.message}`);
    return data;
  }

  async linkDocumentToTask(documentId: string, taskId: string) {
    const { data, error } = await this.supabase
      .from('documents')
      .update({
        source_module: 'task',
        source_record_id: taskId
      })
      .eq('id', documentId)
      .select()
      .single();

    if (error) throw new Error(`Failed to link document: ${error.message}`);
    return data;
  }

  async getDocumentVersions(documentId: string) {
    const { data, error } = await this.supabase
      .from('document_versions')
      .select('*')
      .eq('document_id', documentId)
      .order('version_number', { ascending: false });

    if (error) throw new Error(`Failed to fetch document versions: ${error.message}`);
    return data;
  }

  async createNewVersion(documentId: string, file: File, changeNotes: string) {
    // Get current version
    const { data: currentDoc } = await this.supabase
      .from('documents')
      .select('version, file_url, file_name')
      .eq('id', documentId)
      .single();

    // Upload new version
    const newVersion = await this.uploadDocument(file, {
      sourceModule: currentDoc.source_module,
      sourceRecordId: currentDoc.source_record_id,
      documentName: currentDoc.document_name,
      documentTypeId: currentDoc.document_type_id
    });

    // Create version record
    await this.supabase
      .from('document_versions')
      .insert({
        document_id: documentId,
        version_number: currentDoc.version + 1,
        file_url: newVersion.file_url,
        file_name: newVersion.file_name,
        file_size: newVersion.file_size,
        uploaded_by_id: getCurrentUser().id,
        uploaded_at: new Date().toISOString(),
        change_notes: changeNotes
      });

    // Update document with new version
    await this.supabase
      .from('documents')
      .update({
        version: currentDoc.version + 1,
        file_url: newVersion.file_url,
        file_name: newVersion.file_name,
        file_size: newVersion.file_size,
        updated_at: new Date().toISOString()
      })
      .eq('id', documentId);
  }

  private async calculateFileHash(file: File): Promise<string> {
    const buffer = await file.arrayBuffer();
    const hashBuffer = await crypto.subtle.digest('SHA-256', buffer);
    const hashArray = Array.from(new Uint8Array(hashBuffer));
    return hashArray.map(b => b.toString(16).padStart(2, '0')).join('');
  }

  private async generateDocumentNumber(): Promise<string> {
    const year = new Date().getFullYear();
    const { count } = await this.supabase
      .from('documents')
      .select('*', { count: 'exact', head: true })
      .gte('uploaded_at', `${year}-01-01`)
      .lte('uploaded_at', `${year}-12-31`);

    return `DOC-${year}-${String((count || 0) + 1).padStart(6, '0')}`;
  }
}
```

---

## 6. SECURITY & ACCESS CONTROL

### 6.1 RBAC Implementation

```typescript
// lib/auth/rbac.service.ts
export class RBACService {
  private readonly permissions: Map<string, string[]> = new Map();

  constructor() {
    this.initializePermissions();
  }

  private initializePermissions() {
    // Define all permissions
    const allPermissions = [
      // Task Management
      'task:create', 'task:read', 'task:update', 'task:delete',
      'task:assign', 'task:approve', 'task:comment',
      
      // Request Management
      'request:create', 'request:read', 'request:update', 'request:delete',
      'request:approve', 'request:reject',
      
      // Document Management
      'document:upload', 'document:read', 'document:delete',
      'document:share', 'document:version',
      
      // Master Data
      'master:create', 'master:read', 'master:update', 'master:delete',
      
      // Procurement
      'procurement:create', 'procurement:read', 'procurement:update', 'procurement:delete',
      'procurement:approve',
      
      // Production
      'production:create', 'production:read', 'production:update', 'production:delete',
      
      // Sales
      'sales:create', 'sales:read', 'sales:update', 'sales:delete',
      'sales:export', 'sales:approve',
      
      // Finance
      'finance:read', 'finance:create', 'finance:update', 'finance:delete',
      'finance:approve', 'finance:reconcile',
      
      // HR
      'hr:read', 'hr:create', 'hr:update', 'hr:delete',
      'hr:approve', 'hr:payroll',
      
      // Analytics
      'analytics:read', 'analytics:create', 'analytics:export',
      
      // Admin
      'admin:users', 'admin:roles', 'admin:system', 'admin:audit'
    ];

    // Assign permissions to roles
    this.permissions.set('super_admin', allPermissions);
    
    this.permissions.set('owner', [
      ...allPermissions.filter(p => p.startsWith('finance:') || 
                                   p.startsWith('analytics:') || 
                                   p.startsWith('admin:') ||
                                   p.startsWith('procurement:approve') ||
                                   p.startsWith('sales:approve') ||
                                   p.startsWith('request:approve'))
    ]);
    
    this.permissions.set('hr_admin', [
      'hr:read', 'hr:create', 'hr:update', 'hr:delete',
      'hr:approve', 'hr:payroll',
      'task:read', 'task:comment',
      'document:upload', 'document:read'
    ]);
    
    this.permissions.set('department_head', [
      'task:create', 'task:read', 'task:update', 'task:assign',
      'task:approve', 'task:comment',
      'request:create', 'request:read', 'request:approve',
      'document:upload', 'document:read', 'document:share',
      'procurement:read', 'production:read', 'sales:read',
      'analytics:read'
    ]);
    
    this.permissions.set('manager', [
      'task:create', 'task:read', 'task:update', 'task:assign',
      'task:comment',
      'request:create', 'request:read',
      'document:upload', 'document:read',
      'procurement:create', 'procurement:read',
      'production:create', 'production:read',
      'sales:create', 'sales:read'
    ]);
    
    this.permissions.set('employee', [
      'task:read', 'task:update', 'task:comment',
      'request:create', 'request:read',
      'document:upload', 'document:read'
    ]);
    
    this.permissions.set('viewer', [
      'task:read', 'request:read', 'document:read',
      'analytics:read'
    ]);
  }

  async hasPermission(userId: string, permission: string): Promise<boolean> {
    // Get user roles
    const { data: userRoles } = await supabase
      .from('user_roles')
      .select('role:roles(slug)')
      .eq('user_id', userId);

    if (!userRoles || userRoles.length === 0) return false;

    // Check if any role has the permission
    for (const { role } of userRoles) {
      const rolePermissions = this.permissions.get(role.slug) || [];
      if (rolePermissions.includes(permission)) {
        return true;
      }
      
      // Check if role has wildcard permission
      const permissionParts = permission.split(':');
      if (rolePermissions.includes(`${permissionParts[0]}:*`)) {
        return true;
      }
    }

    return false;
  }

  async canAccessResource(userId: string, resourceType: string, resourceId: string): Promise<boolean> {
    // Check specific resource access based on RLS policies
    // This is a lightweight check before the actual database query
    
    const user = await this.getUser(userId);
    if (!user) return false;

    // Super admin has access to everything
    if (await this.hasPermission(userId, 'admin:system')) {
      return true;
    }

    // Different logic for different resource types
    switch(resourceType) {
      case 'task':
        return this.canAccessTask(user, resourceId);
      case 'request':
        return this.canAccessRequest(user, resourceId);
      case 'document':
        return this.canAccessDocument(user, resourceId);
      default:
        return false;
    }
  }

  private async canAccessTask(user: any, taskId: string): Promise<boolean> {
    const { data: task } = await supabase
      .from('tasks')
      .select('assigned_to_id, created_by_id, department_id')
      .eq('id', taskId)
      .single();

    if (!task) return false;

    // User is assignee or creator
    if (task.assigned_to_id === user.id || task.created_by_id === user.id) {
      return true;
    }

    // Department head has access to department tasks
    if (user.department_id === task.department_id) {
      const isDeptHead = await this.hasPermission(user.id, 'task:approve');
      if (isDeptHead) return true;
    }

    return false;
  }

  private async canAccessRequest(user: any, requestId: string): Promise<boolean> {
    const { data: request } = await supabase
      .from('requests')
      .select('requested_by_id, department_id')
      .eq('id', requestId)
      .single();

    if (!request) return false;

    if (request.requested_by_id === user.id) return true;

    // Department head has access
    if (user.department_id === request.department_id) {
      const isDeptHead = await this.hasPermission(user.id, 'request:approve');
      if (isDeptHead) return true;
    }

    return false;
  }

  private async canAccessDocument(user: any, documentId: string): Promise<boolean> {
    const { data: document } = await supabase
      .from('documents')
      .select('uploaded_by_id, source_module, source_record_id')
      .eq('id', documentId)
      .single();

    if (!document) return false;

    if (document.uploaded_by_id === user.id) return true;

    // Check if user has access to the source record
    if (document.source_module === 'task') {
      return this.canAccessTask(user, document.source_record_id);
    }

    return false;
  }

  private async getUser(userId: string) {
    const { data } = await supabase
      .from('users')
      .select('id, department_id, roles:user_roles(role:roles(slug))')
      .eq('id', userId)
      .single();
    return data;
  }
}

// Middleware for checking permissions
export const withPermission = (permission: string) => {
  return async (req: Request, res: Response, next: NextFunction) => {
    const userId = getCurrentUserId(req);
    const rbac = new RBACService();
    
    const hasPermission = await rbac.hasPermission(userId, permission);
    if (!hasPermission) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }
    
    next();
  };
};
```

### 6.2 Row Level Security (RLS) Policies

```sql
-- Complete RLS Policies for Tasks Table

-- Enable RLS
ALTER TABLE tasks ENABLE ROW LEVEL SECURITY;

-- 1. Users can view tasks they are assigned to, created, or are part of
CREATE POLICY "Users can view their own tasks"
ON tasks FOR SELECT
USING (
  auth.uid() = assigned_to_id 
  OR auth.uid() = created_by_id 
  OR auth.uid() IN (SELECT user_id FROM task_assignments WHERE task_id = tasks.id)
  OR auth.uid() IN (SELECT user_id FROM task_comments WHERE task_id = tasks.id)
);

-- 2. Department heads can view tasks in their department
CREATE POLICY "Department heads can view department tasks"
ON tasks FOR SELECT
USING (
  EXISTS (
    SELECT 1 FROM users u
    WHERE u.id = auth.uid()
    AND u.department_id = (
      SELECT department_id FROM users WHERE id = tasks.assigned_to_id
    )
    AND EXISTS (
      SELECT 1 FROM user_roles ur
      JOIN roles r ON ur.role_id = r.id
      WHERE ur.user_id = auth.uid()
      AND r.slug IN ('department_head', 'owner', 'super_admin')
    )
  )
);

-- 3. Users can create tasks
CREATE POLICY "Users can create tasks"
ON tasks FOR INSERT
WITH CHECK (
  auth.uid() IS NOT NULL
);

-- 4. Users can update tasks they are assigned to or created
CREATE POLICY "Users can update their own tasks"
ON tasks FOR UPDATE
USING (
  auth.uid() = assigned_to_id 
  OR auth.uid() = created_by_id
)
WITH CHECK (
  auth.uid() = assigned_to_id 
  OR auth.uid() = created_by_id
);

-- 5. Managers can update any task in their department
CREATE POLICY "Managers can update department tasks"
ON tasks FOR UPDATE
USING (
  EXISTS (
    SELECT 1 FROM users u
    JOIN user_roles ur ON ur.user_id = u.id
    JOIN roles r ON r.id = ur.role_id
    WHERE u.id = auth.uid()
    AND r.slug IN ('manager', 'department_head', 'owner', 'super_admin')
    AND u.department_id = (
      SELECT department_id FROM users WHERE id = tasks.assigned_to_id
    )
  )
);

-- 6. Only super_admin and owner can delete tasks
CREATE POLICY "Only admins can delete tasks"
ON tasks FOR DELETE
USING (
  EXISTS (
    SELECT 1 FROM user_roles ur
    JOIN roles r ON ur.role_id = r.id
    WHERE ur.user_id = auth.uid()
    AND r.slug IN ('super_admin', 'owner')
  )
);

-- RLS for Task Comments
CREATE POLICY "Users can view comments on accessible tasks"
ON task_comments FOR SELECT
USING (
  EXISTS (
    SELECT 1 FROM tasks t
    WHERE t.id = task_comments.task_id
    AND (
      t.assigned_to_id = auth.uid()
      OR t.created_by_id = auth.uid()
      OR EXISTS (
        SELECT 1 FROM task_assignments ta
        WHERE ta.task_id = t.id
        AND ta.user_id = auth.uid()
      )
    )
  )
);

CREATE POLICY "Users can comment on accessible tasks"
ON task_comments FOR INSERT
WITH CHECK (
  EXISTS (
    SELECT 1 FROM tasks t
    WHERE t.id = task_comments.task_id
    AND (
      t.assigned_to_id = auth.uid()
      OR t.created_by_id = auth.uid()
      OR EXISTS (
        SELECT 1 FROM task_assignments ta
        WHERE ta.task_id = t.id
        AND ta.user_id = auth.uid()
      )
    )
  )
);
```

---

## 7. DETAILED 24-MONTH DEVELOPMENT TIMELINE

### 7.1 Phase 1: Foundation & Infrastructure (Months 1-6)

| Sprint | Week | Priority | Module | Task | Deliverable | Dependencies | Status |
|--------|------|----------|--------|------|-------------|--------------|--------|
| S1 | W1 | Critical | Foundation | Project Setup & Architecture | Next.js 16+ with App Router, TypeScript, Tailwind CSS v4 | None | Planning |
| S1 | W1-2 | Critical | Foundation | Database Schema Design | 180+ tables with relationships, RLS policies | None | Planning |
| S1 | W1-2 | Critical | Foundation | Authentication & RBAC | Complete 7-tier authority system, SSO support | Auth Ready | Planning |
| S1 | W2-3 | Critical | Foundation | CI/CD Pipeline | GitHub Actions, staging/production deployment | Project Setup | Planning |
| S2 | W3-4 | Critical | Master Data | Chart of Accounts | 5-level COA with aggregation logic | Schema Complete | Planning |
| S2 | W4-5 | Critical | Master Data | Financial Configuration | Cost centers, fiscal year, multi-currency | COA Complete | Planning |
| S2 | W5-6 | Critical | Master Data | Party Management | Suppliers, customers, farmers, agents | Financial Config | Planning |
| S3 | W6-7 | High | Master Data | Product & Item Master | Paddy, rice, by-products specifications | Party Complete | Planning |
| S3 | W7-8 | High | Master Data | Department & User Setup | Departments, roles, employee profiles | Product Master | Planning |
| S3 | W8-9 | High | Master Data | Inventory Configuration | Warehouses, zones, bins | User Setup | Planning |
| S4 | W9-10 | High | Foundation | Workflow & Approval Engine | Configurable approval workflows | All Masters | Planning |
| S4 | W10-11 | High | Foundation | Notification System | Email, SMS, in-app notifications | Workflow Engine | Planning |
| S4 | W11-12 | High | Foundation | Document Management | Templates, PDF generation, storage | Notification System | Planning |
| S5 | W12-13 | High | Foundation | Localization & Compliance | Pakistan-specific tax, regulatory | All Configuration | Planning |
| S5 | W13-14 | Medium | Foundation | Accounting Integration | QuickBooks/Tally integration | Financial Setup | Planning |
| S5 | W14-15 | Medium | Foundation | Banking Integration | Payment gateways, bank APIs | Accounting Integration | Planning |
| S6 | W15-16 | High | UI/UX | Design System | 60+ component library | All Configurations | Planning |
| S6 | W16-17 | High | UI/UX | Responsive Dashboard | Role-specific dashboards, mobile-first | Design System | Planning |
| S6 | W17-18 | High | UI/UX | User Personalization | Preferences, themes, shortcuts | Dashboard Complete | Planning |

### 7.2 Phase 2: Core Operational Modules (Months 7-14)

| Sprint | Week | Priority | Module | Task | Deliverable | Dependencies | Status |
|--------|------|----------|--------|------|-------------|--------------|--------|
| S7 | W19-22 | Critical | Procurement | Paddy Deal Management | Full deal lifecycle with automation | Phase 1 Complete | Planning |
| S7 | W22-24 | Critical | Procurement | Arrival & GRN | Inbound logistics, weighbridge integration | Deal Complete | Planning |
| S8 | W25-28 | Critical | Quality | Complete Lab Management | LIMS, instrument integration, CoA | GRN Complete | Planning |
| S9 | W29-32 | Critical | Production | Production Management | Planning, BOM, costing, OEE tracking | Quality Complete | Planning |
| S10 | W33-36 | Critical | Inventory | Advanced WMS | Multi-location, RF scanning, forecasting | Production Complete | Planning |
| S11 | W37-40 | Critical | Sales | Local Sales Management | Orders, pricing, delivery, returns | Inventory Complete | Planning |
| S11 | W40-42 | High | Sales | Export Management | International sales, LC, shipping | Local Sales | Planning |
| S12 | W43-46 | High | Logistics | Logistics Management | Fleet, routing, GPS tracking | Sales Complete | Planning |

### 7.3 Phase 3: Task Management & Analytics Module (Months 15-18) - **NEW**

| Sprint | Week | Priority | Module | Task | Deliverable | Dependencies | Status |
|--------|------|----------|--------|------|-------------|--------------|--------|
| S13 | W47-50 | Critical | Task Management | Task Database Schema | tasks, task_comments, task_attachments, task_dependencies | Phase 2 Complete | Planning |
| S13 | W47-50 | Critical | Task Management | Task Service Layer | taskService, CRUD operations, scoring | Schema Complete | Planning |
| S13 | W48-49 | Critical | Task Management | Task Board UI | Kanban board, drag-and-drop | Service Layer | Planning |
| S13 | W49-50 | High | Task Management | Task List UI | Filterable table, advanced search | Board UI | Planning |
| S13 | W49-50 | High | Task Management | Task Detail UI | Comments, attachments, timeline | List UI | Planning |
| S14 | W50-52 | Critical | Request Management | Request Database Schema | requests, approvals, workflows | Task Module | Planning |
| S14 | W51-53 | Critical | Request Management | Request Service Layer | requestService, approval engine | Schema Complete | Planning |
| S14 | W51-53 | High | Request Management | Request UI | Request forms, approval dashboard | Service Layer | Planning |
| S14 | W52-54 | High | Scoring | Scoring Engine | calculateTaskScore, updateUserScore | Task Module | Planning |
| S14 | W53-54 | Medium | Scoring | User Performance UI | Performance dashboard, ranking | Scoring Engine | Planning |
| S15 | W54-56 | Critical | Document Management | Document Database Schema | documents, document_versions | Task Module | Planning |
| S15 | W54-56 | Critical | Document Management | Document Service Layer | upload, versioning, linking | Schema Complete | Planning |
| S15 | W55-57 | High | Document Management | Document UI | Uploader, viewer, manager | Service Layer | Planning |
| S15 | W56-58 | High | Traceability | Audit Log System | audit_logs, activity feed | All Modules | Planning |
| S16 | W57-60 | High | Integration | Task Integration | Integrate tasks into all modules | Task Module | Planning |
| S16 | W58-60 | High | Analytics | Task Analytics | Task reports, performance metrics | Analytics Module | Planning |
| S16 | W59-62 | Medium | Mobile | Task Mobile UI | Mobile task management | Task Module | Planning |

### 7.4 Phase 4: Finance & Advanced Modules (Months 19-22)

| Sprint | Week | Priority | Module | Task | Deliverable | Dependencies | Status |
|--------|------|----------|--------|------|-------------|--------------|--------|
| S17 | W63-66 | Critical | Finance | Core Accounting | Multi-company GL, consolidation | All Operational | Planning |
| S18 | W67-70 | Critical | Finance | Advanced Finance | AP/AR, banking, portals | Core Accounting | Planning |
| S19 | W71-74 | Critical | Finance | Tax Management | Complete tax, FBR integration | Advanced Finance | Planning |
| S19 | W73-75 | High | Finance | Management Accounting | Costing, profitability | Tax Complete | Planning |
| S20 | W75-78 | High | HR | HR Management | Employee, attendance, leave | Finance Complete | Planning |
| S20 | W78-80 | High | HR | Payroll Management | Complete payroll with auto-calc | HR Complete | Planning |
| S20 | W79-82 | Medium | Assets | Fixed Assets Management | Asset lifecycle, depreciation | Finance Complete | Planning |

### 7.5 Phase 5: Analytics & Deployment (Months 23-24)

| Sprint | Week | Priority | Module | Task | Deliverable | Dependencies | Status |
|--------|------|----------|--------|------|-------------|--------------|--------|
| S21 | W83-86 | Very High | Analytics | Business Intelligence | BI, ML predictions, custom reports | All Modules | Planning |
| S21 | W85-88 | Very High | Analytics | Data Warehouse | ETL pipeline, Snowflake/BigQuery | BI Setup | Planning |
| S22 | W87-90 | High | Mobile | Mobile Applications | iOS/Android apps | All Modules | Planning |
| S22 | W89-92 | High | Mobile | Mobile API | GraphQL endpoint for mobile | Mobile Apps | Planning |
| S23 | W91-94 | Critical | Deployment | Enterprise Deployment | Production, training, go-live | All Modules | Planning |
| S23 | W93-96 | Critical | Deployment | Data Migration | Legacy data migration | Deployment Ready | Planning |
| S24 | W95-98 | Critical | Support | Post-Implementation | Stabilization, handover | Deployment | Planning |
| S24 | W97-100 | Critical | Documentation | System Documentation | Complete technical docs | All Modules | Planning |

---

## 8. TECHNICAL IMPLEMENTATION DETAILS BY MODULE

### 8.1 Core Module Implementation Matrix

| Module | Tables | Services | Edge Functions | UI Components | RLS Policies |
|--------|--------|----------|---------------|---------------|--------------|
| Task Management | 12 | task.service.ts | task-management/* | TaskBoard, TaskList | 25+ |
| Request Management | 8 | request.service.ts | request-management/* | RequestForm, ApprovalFlow | 20+ |
| Document Management | 5 | document.service.ts | document-management/* | DocumentUploader, Viewer | 15+ |
| Procurement | 12 | procurement.service.ts | procurement/* | DealForm, GRNForm | 30+ |
| Production | 8 | production.service.ts | production/* | ProductionOrder, BOM | 15+ |
| Inventory | 10 | inventory.service.ts | inventory/* | WMSDashboard, Transfer | 25+ |
| Sales | 10 | sales.service.ts | sales/* | SalesOrder, ExportForm | 25+ |
| Finance | 18 | finance.service.ts | finance/* | JournalEntry, Reports | 40+ |
| HR | 10 | hr.service.ts | hr/* | EmployeeProfile, Payroll | 25+ |
| Analytics | 8 | analytics.service.ts | analytics/* | Dashboards, Reports | 15+ |

### 8.2 Edge Functions Implementation

```typescript
// supabase/functions/task-management/index.ts
import { serve } from "https://deno.land/std@0.177.0/http/server.ts";
import { createClient } from "https://esm.sh/@supabase/supabase-js@2.39.7";

const supabase = createClient(
  Deno.env.get("SUPABASE_URL")!,
  Deno.env.get("SUPABASE_SERVICE_ROLE_KEY")!
);

serve(async (req) => {
  const url = new URL(req.url);
  const path = url.pathname;
  const method = req.method;

  // Parse request body
  let body = null;
  if (method !== "GET") {
    try {
      body = await req.json();
    } catch {
      // No body
    }
  }

  // Authentication
  const authHeader = req.headers.get("Authorization");
  if (!authHeader) {
    return new Response(JSON.stringify({ error: "Unauthorized" }), {
      status: 401,
      headers: { "Content-Type": "application/json" },
    });
  }

  const token = authHeader.replace("Bearer ", "");
  const { data: { user }, error: authError } = await supabase.auth.getUser(token);
  
  if (authError || !user) {
    return new Response(JSON.stringify({ error: "Invalid token" }), {
      status: 401,
      headers: { "Content-Type": "application/json" },
    });
  }

  // Route handling
  try {
    let result;
    
    // GET /tasks
    if (path === "/tasks" && method === "GET") {
      const { searchParams } = url;
      const filters = {
        status: searchParams.get("status"),
        priority: searchParams.get("priority"),
        assignedTo: searchParams.get("assignedTo"),
        search: searchParams.get("search"),
        page: parseInt(searchParams.get("page") || "1"),
        limit: parseInt(searchParams.get("limit") || "20"),
      };
      
      result = await handleGetTasks(user.id, filters);
    }
    
    // POST /tasks
    else if (path === "/tasks" && method === "POST") {
      result = await handleCreateTask(user.id, body);
    }
    
    // GET /tasks/:id
    else if (path.match(/^\/tasks\/[^\/]+$/) && method === "GET") {
      const taskId = path.split("/")[2];
      result = await handleGetTask(user.id, taskId);
    }
    
    // PATCH /tasks/:id
    else if (path.match(/^\/tasks\/[^\/]+$/) && method === "PATCH") {
      const taskId = path.split("/")[2];
      result = await handleUpdateTask(user.id, taskId, body);
    }
    
    // POST /tasks/:id/comments
    else if (path.match(/^\/tasks\/[^\/]+\/comments$/) && method === "POST") {
      const taskId = path.split("/")[2];
      result = await handleAddComment(user.id, taskId, body);
    }
    
    // POST /tasks/:id/status
    else if (path.match(/^\/tasks\/[^\/]+\/status$/) && method === "POST") {
      const taskId = path.split("/")[2];
      result = await handleUpdateStatus(user.id, taskId, body);
    }
    
    else {
      return new Response(JSON.stringify({ error: "Not found" }), {
        status: 404,
        headers: { "Content-Type": "application/json" },
      });
    }

    return new Response(JSON.stringify(result), {
      status: 200,
      headers: { "Content-Type": "application/json" },
    });
  } catch (error) {
    console.error("Error:", error);
    return new Response(JSON.stringify({ error: error.message }), {
      status: 500,
      headers: { "Content-Type": "application/json" },
    });
  }
});

async function handleGetTasks(userId: string, filters: any) {
  let query = supabase
    .from("tasks")
    .select(`
      *,
      assigned_to:assigned_to_id(*),
      created_by:created_by_id(*),
      category:category_id(*),
      comments:task_comments(count)
    `, { count: "exact" });

  // Apply filters
  if (filters.status) query = query.eq("status", filters.status);
  if (filters.priority) query = query.eq("priority", filters.priority);
  if (filters.assignedTo) query = query.eq("assigned_to_id", filters.assignedTo);
  if (filters.search) {
    query = query.or(`title.ilike.%${filters.search}%,description.ilike.%${filters.search}%`);
  }

  // Apply RLS (user can only see accessible tasks)
  query = query.eq("assigned_to_id", userId);

  // Apply pagination
  const from = (filters.page - 1) * filters.limit;
  const to = from + filters.limit - 1;
  query = query.range(from, to);

  const { data, error, count } = await query;
  
  if (error) throw new Error(`Database error: ${error.message}`);
  
  return { tasks: data, total: count, page: filters.page, limit: filters.limit };
}

async function handleCreateTask(userId: string, data: any) {
  // Validate input
  const required = ["title", "description", "priority", "due_date"];
  for (const field of required) {
    if (!data[field]) {
      throw new Error(`Missing required field: ${field}`);
    }
  }

  // Insert task
  const { data: task, error } = await supabase
    .from("tasks")
    .insert({
      title: data.title,
      description: data.description,
      priority: data.priority,
      due_date: data.due_date,
      category_id: data.categoryId,
      job_location_id: data.jobLocationId,
      assigned_to_id: data.assignedToId || null,
      created_by_id: userId,
      status: "Pending",
      pending_days: calculatePendingDays(data.due_date),
      bucket: calculateBucket(data.due_date),
      created_at: new Date().toISOString(),
    })
    .select()
    .single();

  if (error) throw new Error(`Database error: ${error.message}`);

  // Send notifications
  if (data.assignedToId) {
    await sendTaskAssignmentNotification(task, data.assignedToId);
  }

  return task;
}

function calculatePendingDays(dueDate: string): number {
  const due = new Date(dueDate);
  const now = new Date();
  const diff = Math.ceil((due.getTime() - now.getTime()) / (1000 * 60 * 60 * 24));
  return Math.max(0, diff);
}

function calculateBucket(dueDate: string): string {
  const days = calculatePendingDays(dueDate);
  if (days <= 5) return "0 to 5 days";
  if (days <= 10) return "5 to 10 days";
  if (days <= 15) return "10 to 15 days";
  if (days <= 20) return "15 to 20 days";
  if (days <= 25) return "20 to 25 days";
  if (days <= 30) return "25 to 30 days";
  return "Above 30 days";
}

async function sendTaskAssignmentNotification(task: any, assigneeId: string) {
  // Send email, SMS, and in-app notification
  // ... implementation
}
```

---

## 9. INTEGRATION STRATEGY

### 9.1 External System Integration Architecture

```typescript
// services/integration/integration.manager.ts
export class IntegrationManager {
  private integrations: Map<string, BaseIntegration> = new Map();

  constructor() {
    this.registerIntegrations();
  }

  private registerIntegrations() {
    this.integrations.set('quickbooks', new QuickBooksIntegration());
    this.integrations.set('banking', new BankingIntegration());
    this.integrations.set('fbr', new FBRIntegration());
    this.integrations.set('sms', new SMSIntegration());
    this.integrations.set('email', new EmailIntegration());
    this.integrations.set('gps', new GPSIntegration());
  }

  async executeIntegration<T>(
    integrationName: string,
    action: string,
    data: any
  ): Promise<T> {
    const integration = this.integrations.get(integrationName);
    if (!integration) {
      throw new Error(`Integration ${integrationName} not found`);
    }

    // Log integration attempt
    await this.logIntegrationAttempt(integrationName, action, data);

    try {
      const result = await integration.execute(action, data);
      
      // Log success
      await this.logIntegrationSuccess(integrationName, action, data, result);
      
      return result;
    } catch (error) {
      // Log failure
      await this.logIntegrationFailure(integrationName, action, data, error);
      throw error;
    }
  }

  private async logIntegrationAttempt(integration: string, action: string, data: any) {
    // Create audit log entry
    // ... implementation
  }
}

// QuickBooks Integration
export class QuickBooksIntegration extends BaseIntegration {
  private oauth2Client: any;
  
  constructor() {
    super();
    this.initializeOAuth();
  }

  async execute(action: string, data: any): Promise<any> {
    switch(action) {
      case 'create_invoice':
        return this.createInvoice(data);
      case 'sync_accounts':
        return this.syncAccounts(data);
      case 'sync_transactions':
        return this.syncTransactions(data);
      default:
        throw new Error(`Unknown action: ${action}`);
    }
  }

  private async createInvoice(data: any) {
    const invoice = {
      Line: data.items.map((item: any) => ({
        Amount: item.amount,
        DetailType: 'SalesItemLineDetail',
        SalesItemLineDetail: {
          ItemRef: { value: item.itemId },
          Qty: item.quantity,
          UnitPrice: item.unitPrice
        }
      })),
      CustomerRef: { value: data.customerId },
      BillEmail: { Address: data.email },
      DueDate: data.dueDate
    };

    // Make API call to QuickBooks
    // ... implementation

    return { invoiceId: 'QB-12345', status: 'created' };
  }

  private async syncAccounts(data: any) {
    // Sync chart of accounts
    // ... implementation
  }

  private async syncTransactions(data: any) {
    // Sync transactions
    // ... implementation
  }
}

// FBR Integration (Pakistan Tax)
export class FBRIntegration extends BaseIntegration {
  async execute(action: string, data: any): Promise<any> {
    switch(action) {
      case 'verify_ntn':
        return this.verifyNTN(data.ntn);
      case 'validate_strn':
        return this.validateSTRN(data.strn);
      case 'submit_sales_tax':
        return this.submitSalesTax(data);
      case 'submit_wht':
        return this.submitWithholdingTax(data);
      default:
        throw new Error(`Unknown action: ${action}`);
    }
  }

  private async verifyNTN(ntn: string) {
    // Call FBR API to verify NTN
    // ... implementation
    return { isValid: true, status: 'active' };
  }

  private async validateSTRN(strn: string) {
    // Call FBR API to validate STRN
    // ... implementation
    return { isValid: true, registrationDate: '2023-01-01' };
  }

  private async submitSalesTax(data: any) {
    // Submit sales tax return to FBR
    // ... implementation
    return { submissionId: 'FBR-2024-001', status: 'submitted' };
  }
}
```

### 9.2 Webhook System

```typescript
// services/api/webhook.service.ts
export class WebhookService {
  private readonly supabase = createSupabaseClient();

  async registerWebhook(config: WebhookConfig) {
    const { data, error } = await this.supabase
      .from('webhooks')
      .insert({
        name: config.name,
        url: config.url,
        events: config.events,
        headers: config.headers,
        secret: config.secret || this.generateSecret(),
        is_active: true,
        created_by_id: getCurrentUser().id
      })
      .select()
      .single();

    if (error) throw new Error(`Failed to register webhook: ${error.message}`);
    return data;
  }

  async triggerWebhook(event: string, payload: any) {
    const { data: webhooks } = await this.supabase
      .from('webhooks')
      .select('*')
      .contains('events', [event])
      .eq('is_active', true);

    if (!webhooks || webhooks.length === 0) return;

    // Trigger all matching webhooks
    for (const webhook of webhooks) {
      await this.sendWebhook(webhook, event, payload);
    }
  }

  private async sendWebhook(webhook: any, event: string, payload: any) {
    try {
      const signature = this.generateSignature(payload, webhook.secret);
      
      const response = await fetch(webhook.url, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          ...webhook.headers,
          'X-Webhook-Signature': signature,
          'X-Webhook-Event': event
        },
        body: JSON.stringify({
          event: event,
          timestamp: new Date().toISOString(),
          data: payload
        })
      });

      // Log webhook attempt
      await this.logWebhookAttempt(webhook.id, event, response.status);

      if (!response.ok) {
        throw new Error(`Webhook failed with status: ${response.status}`);
      }
    } catch (error) {
      // Log failure and potentially retry
      await this.logWebhookFailure(webhook.id, event, error);
    }
  }

  private generateSignature(payload: any, secret: string): string {
    const encoder = new TextEncoder();
    const data = encoder.encode(JSON.stringify(payload));
    const secretKey = encoder.encode(secret);
    
    // HMAC-SHA256
    // ... implementation using Web Crypto API
    
    return 'signature';
  }
}
```

---

## 10. PERFORMANCE OPTIMIZATION

### 10.1 Caching Strategy

```typescript
// lib/cache/cache.service.ts
import Redis from 'ioredis';

export class CacheService {
  private redis: Redis;
  private defaultTTL = 300; // 5 minutes

  constructor() {
    this.redis = new Redis({
      host: process.env.REDIS_HOST || 'localhost',
      port: parseInt(process.env.REDIS_PORT || '6379'),
      password: process.env.REDIS_PASSWORD,
      retryStrategy: (times) => {
        const delay = Math.min(times * 50, 2000);
        return delay;
      }
    });
  }

  async get<T>(key: string): Promise<T | null> {
    try {
      const data = await this.redis.get(key);
      if (!data) return null;
      return JSON.parse(data) as T;
    } catch (error) {
      console.error('Cache get error:', error);
      return null;
    }
  }

  async set(key: string, value: any, ttl?: number): Promise<void> {
    try {
      const serialized = JSON.stringify(value);
      if (ttl) {
        await this.redis.setex(key, ttl, serialized);
      } else {
        await this.redis.setex(key, this.defaultTTL, serialized);
      }
    } catch (error) {
      console.error('Cache set error:', error);
    }
  }

  async delete(key: string): Promise<void> {
    try {
      await this.redis.del(key);
    } catch (error) {
      console.error('Cache delete error:', error);
    }
  }

  async deletePattern(pattern: string): Promise<void> {
    try {
      const keys = await this.redis.keys(pattern);
      if (keys.length > 0) {
        await this.redis.del(...keys);
      }
    } catch (error) {
      console.error('Cache delete pattern error:', error);
    }
  }

  async remember<T>(
    key: string,
    fetchFn: () => Promise<T>,
    ttl?: number
  ): Promise<T> {
    const cached = await this.get<T>(key);
    if (cached !== null) {
      return cached;
    }

    const fresh = await fetchFn();
    await this.set(key, fresh, ttl);
    return fresh;
  }
}

// Usage example in service
export class TaskService {
  private cache = new CacheService();

  async getTask(taskId: string): Promise<Task> {
    const cacheKey = `task:${taskId}`;
    
    return this.cache.remember(
      cacheKey,
      async () => {
        const { data, error } = await this.supabase
          .from('tasks')
          .select('*')
          .eq('id', taskId)
          .single();
        
        if (error) throw error;
        return data;
      },
      600 // 10 minutes
    );
  }
}
```

### 10.2 Database Indexing Strategy

```sql
-- Performance-critical indexes for Task Management

-- Task table indexes
CREATE INDEX idx_tasks_assigned_to_id ON tasks(assigned_to_id);
CREATE INDEX idx_tasks_status ON tasks(status);
CREATE INDEX idx_tasks_priority ON tasks(priority);
CREATE INDEX idx_tasks_due_date ON tasks(due_date);
CREATE INDEX idx_tasks_category_id ON tasks(category_id);
CREATE INDEX idx_tasks_created_at ON tasks(created_at DESC);
CREATE INDEX idx_tasks_completion_percentage ON tasks(completion_percentage);

-- Composite indexes for common queries
CREATE INDEX idx_tasks_assigned_status ON tasks(assigned_to_id, status);
CREATE INDEX idx_tasks_status_due ON tasks(status, due_date);
CREATE INDEX idx_tasks_category_status ON tasks(category_id, status);

-- Full-text search index
CREATE INDEX idx_tasks_search ON tasks 
  USING GIN (to_tsvector('english', title || ' ' || COALESCE(description, '')));

-- Task comments indexes
CREATE INDEX idx_task_comments_task_id ON task_comments(task_id);
CREATE INDEX idx_task_comments_user_id ON task_comments(user_id);
CREATE INDEX idx_task_comments_created_at ON task_comments(created_at DESC);
CREATE INDEX idx_task_comments_is_concern ON task_comments(is_concern) WHERE is_concern = true;

-- Request indexes
CREATE INDEX idx_requests_status ON requests(status);
CREATE INDEX idx_requests_department_id ON requests(department_id);
CREATE INDEX idx_requests_requested_by_id ON requests(requested_by_id);
CREATE INDEX idx_requests_created_at ON requests(created_at DESC);

-- Approval indexes
CREATE INDEX idx_approvals_request_id ON approvals(request_id);
CREATE INDEX idx_approvals_approver_id ON approvals(approver_id);
CREATE INDEX idx_approvals_status ON approvals(status);

-- Document indexes
CREATE INDEX idx_documents_source ON documents(source_module, source_record_id);
CREATE INDEX idx_documents_uploaded_by_id ON documents(uploaded_by_id);
CREATE INDEX idx_documents_created_at ON documents(created_at DESC);
CREATE INDEX idx_documents_status ON documents(status);

-- User performance indexes
CREATE INDEX idx_performance_user_id ON user_performance_scores(user_id);
CREATE INDEX idx_performance_evaluation_period ON user_performance_scores(evaluation_period);
CREATE INDEX idx_performance_overall_score ON user_performance_scores(overall_score DESC);
```

### 10.3 Query Optimization

```typescript
// Optimized query with pagination and eager loading
async function getTasksWithFilters(
  filters: TaskFilters,
  pagination: Pagination
) {
  let query = supabase
    .from('tasks')
    .select(`
      *,
      assigned_to:assigned_to_id(
        id,
        first_name,
        last_name,
        email,
        department:department_id(name)
      ),
      created_by:created_by_id(
        id,
        first_name,
        last_name
      ),
      category:category_id(
        id,
        category_name,
        color
      ),
      job_location:job_location_id(
        id,
        location_name
      ),
      comments_count:task_comments(count)
    `, { count: 'exact' });

  // Apply filters with proper indexing
  if (filters.status) {
    query = query.eq('status', filters.status);
  }
  
  if (filters.priority) {
    query = query.eq('priority', filters.priority);
  }
  
  if (filters.assignedTo) {
    query = query.eq('assigned_to_id', filters.assignedTo);
  }
  
  if (filters.search) {
    // Use full-text search index
    query = query.textSearch('title_description', filters.search, {
      config: 'english'
    });
  }

  // Date range filter
  if (filters.startDate && filters.endDate) {
    query = query
      .gte('created_at', filters.startDate)
      .lte('created_at', filters.endDate);
  }

  // Sorting
  const sortField = filters.sortBy || 'created_at';
  const sortOrder = filters.sortOrder || 'desc';
  query = query.order(sortField, { ascending: sortOrder === 'asc' });

  // Pagination
  const from = (pagination.page - 1) * pagination.limit;
  const to = from + pagination.limit - 1;
  query = query.range(from, to);

  // Use explain to analyze query
  if (process.env.NODE_ENV === 'development') {
    const { data: explain } = await query.explain();
    console.log('Query plan:', explain);
  }

  const { data, error, count } = await query;
  
  if (error) throw error;
  
  return {
    tasks: data,
    total: count,
    page: pagination.page,
    limit: pagination.limit,
    totalPages: Math.ceil(count / pagination.limit)
  };
}
```

---

## 11. TESTING STRATEGY

### 11.1 Unit Testing

```typescript
// __tests__/services/task.service.test.ts
import { TaskService } from '@/services/api/task.service';

describe('TaskService', () => {
  let taskService: TaskService;

  beforeEach(() => {
    taskService = new TaskService();
  });

  describe('createTask', () => {
    it('should create a task with valid data', async () => {
      const taskData = {
        title: 'Test Task',
        description: 'This is a test task',
        priority: 'High',
        dueDate: '2024-12-31',
        categoryId: 'cat-123',
        assignedToId: 'user-456'
      };

      const result = await taskService.createTask(taskData);
      
      expect(result).toHaveProperty('id');
      expect(result.title).toBe(taskData.title);
      expect(result.status).toBe('Pending');
      expect(result.task_number).toMatch(/^TASK-2024-/);
    });

    it('should throw error for invalid data', async () => {
      const invalidData = {
        title: '', // Empty title
        description: 'Test'
      };

      await expect(taskService.createTask(invalidData))
        .rejects
        .toThrow('Title is required');
    });
  });

  describe('updateTaskStatus', () => {
    it('should update task status and calculate score on completion', async () => {
      const taskId = 'task-123';
      const newStatus = 'Done / Completed';

      const result = await taskService.updateTaskStatus(taskId, newStatus);
      
      expect(result.status).toBe('Done / Completed');
      expect(result.completion_percentage).toBe(100);
      expect(result.task_score).toBeDefined();
    });

    it('should record status history', async () => {
      const taskId = 'task-123';
      
      await taskService.updateTaskStatus(taskId, 'In Progress');
      
      const history = await getTaskStatusHistory(taskId);
      expect(history).toContainEqual(
        expect.objectContaining({
          task_id: taskId,
          new_status: 'In Progress'
        })
      );
    });
  });

  describe('addComment', () => {
    it('should add a regular comment', async () => {
      const taskId = 'task-123';
      const content = 'This is a test comment';

      const result = await taskService.addComment(taskId, content, false);
      
      expect(result.comment).toBe(content);
      expect(result.is_concern).toBe(false);
      expect(result.comment_type).toBe('Regular');
    });

    it('should add a concern and update user score', async () => {
      const taskId = 'task-123';
      const content = 'I have a serious concern about this task';

      const result = await taskService.addComment(taskId, content, true);
      
      expect(result.is_concern).toBe(true);
      expect(result.comment_type).toBe('Concern');
      
      // Verify user score was updated
      const userScore = await getUserPerformanceScore(result.user_id);
      expect(userScore.concern_value_score).toBeGreaterThan(0);
    });
  });
});
```

### 11.2 Integration Testing

```typescript
// __tests__/integration/task-management.test.ts
describe('Task Management Integration', () => {
  let testUser: any;
  let testTask: any;

  beforeAll(async () => {
    // Setup test database
    await setupTestDatabase();
    
    // Create test user
    testUser = await createTestUser({
      email: 'test@example.com',
      role: 'manager'
    });
  });

  afterAll(async () => {
    await cleanupTestDatabase();
  });

  test('Full task lifecycle', async () => {
    // 1. Create task
    const taskData = {
      title: 'Integration Test Task',
      description: 'Testing full lifecycle',
      priority: 'High',
      dueDate: '2024-12-31',
      assignedToId: testUser.id
    };

    const created = await createTask(testUser.id, taskData);
    expect(created).toBeDefined();
    expect(created.status).toBe('Pending');
    testTask = created;

    // 2. Assign and start task
    await updateTaskStatus(created.id, 'In Progress');
    const inProgress = await getTask(created.id);
    expect(inProgress.status).toBe('In Progress');
    expect(inProgress.completion_percentage).toBe(50);

    // 3. Add comments
    await addComment(created.id, 'Working on this task', false);
    await addComment(created.id, 'Found an issue with the approach', true);

    const comments = await getTaskComments(created.id);
    expect(comments).toHaveLength(2);
    expect(comments.some(c => c.is_concern)).toBe(true);

    // 4. Complete task
    await updateTaskStatus(created.id, 'Done / Completed');
    const completed = await getTask(created.id);
    expect(completed.status).toBe('Done / Completed');
    expect(completed.completion_percentage).toBe(100);
    expect(completed.task_score).toBeGreaterThan(0);

    // 5. Verify user score updated
    const score = await getUserPerformanceScore(testUser.id);
    expect(score.task_completion_score).toBeGreaterThan(0);
  });

  test('Approval workflow', async () => {
    // 1. Create request
    const requestData = {
      typeId: 'new_hire',
      title: 'Request for new QA Engineer',
      description: 'We need a new QA engineer for the production line',
      priority: 'High',
      department: 'production',
      requestData: {
        position: 'QA Engineer',
        salary: '150000',
        experience: '3-5 years'
      }
    };

    const request = await createRequest(testUser.id, requestData);
    expect(request.status).toBe('Submitted');

    // 2. Approve workflow
    const approvals = await getPendingApprovals('dept_head');
    expect(approvals).toContainEqual(
      expect.objectContaining({ request_id: request.id })
    );

    // 3. Approve
    const approval = approvals.find(a => a.request_id === request.id);
    await approveRequest(approval.id, 'Approved. Good to proceed.');

    // 4. Verify completion
    const completedRequest = await getRequest(request.id);
    expect(completedRequest.status).toBe('Approved');

    // 5. Verify linked task
    const linkedTask = await getTaskBySource('request', request.id);
    expect(linkedTask.status).toBe('Done / Completed');
  });
});
```

### 11.3 Performance Testing

```typescript
// __tests__/performance/task-api.test.ts
import { performance } from 'perf_hooks';

describe('Task API Performance', () => {
  const BATCH_SIZE = 100;
  const CONCURRENT_REQUESTS = 50;

  test('Should handle concurrent task creation', async () => {
    const tasks = Array(BATCH_SIZE).fill(null).map((_, i) => ({
      title: `Performance Task ${i}`,
      description: 'Testing performance',
      priority: 'Medium',
      dueDate: '2024-12-31'
    }));

    const startTime = performance.now();
    
    // Create tasks concurrently
    const promises = tasks.map(task => createTask('test-user', task));
    const results = await Promise.all(promises);
    
    const endTime = performance.now();
    const duration = endTime - startTime;
    
    console.log(`Created ${BATCH_SIZE} tasks in ${duration}ms`);
    console.log(`Average: ${duration / BATCH_SIZE}ms per task`);
    
    expect(results).toHaveLength(BATCH_SIZE);
    expect(duration).toBeLessThan(10000); // Should take less than 10 seconds
  });

  test('Should handle complex queries efficiently', async () => {
    const filters = {
      status: 'Pending',
      priority: 'High',
      assignedTo: 'user-123',
      search: 'urgent',
      startDate: '2024-01-01',
      endDate: '2024-12-31',
      sortBy: 'due_date',
      sortOrder: 'asc'
    };

    const pagination = {
      page: 1,
      limit: 50
    };

    const startTime = performance.now();
    
    const result = await getTasksWithFilters(filters, pagination);
    
    const endTime = performance.now();
    const duration = endTime - startTime;
    
    console.log(`Complex query executed in ${duration}ms`);
    console.log(`Found ${result.total} tasks`);
    
    expect(duration).toBeLessThan(500); // Should take less than 500ms
  });

  test('Should handle bulk status updates', async () => {
    // Create test tasks
    const taskIds = await createBulkTasks(200);
    
    const startTime = performance.now();
    
    // Update all tasks in parallel
    const promises = taskIds.map(id => 
      updateTaskStatus(id, 'In Progress')
    );
    await Promise.all(promises);
    
    const endTime = performance.now();
    const duration = endTime - startTime;
    
    console.log(`Updated ${taskIds.length} tasks in ${duration}ms`);
    console.log(`Average: ${duration / taskIds.length}ms per update`);
    
    expect(duration).toBeLessThan(5000); // Should take less than 5 seconds
  });
});
```

---

## 12. DEPLOYMENT & DEVOPS

### 12.1 CI/CD Pipeline

```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linting
        run: npm run lint
      
      - name: Run type checking
        run: npm run type-check
      
      - name: Run tests
        run: npm run test
        env:
          SUPABASE_URL: ${{ secrets.SUPABASE_URL }}
          SUPABASE_ANON_KEY: ${{ secrets.SUPABASE_ANON_KEY }}
      
      - name: Run integration tests
        run: npm run test:integration
        env:
          SUPABASE_URL: ${{ secrets.SUPABASE_URL }}
          SUPABASE_ANON_KEY: ${{ secrets.SUPABASE_ANON_KEY }}
  
  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build application
        run: npm run build
        env:
          NEXT_PUBLIC_SUPABASE_URL: ${{ secrets.SUPABASE_URL }}
          NEXT_PUBLIC_SUPABASE_ANON_KEY: ${{ secrets.SUPABASE_ANON_KEY }}
      
      - name: Upload build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: build
          path: .next
  
  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v3
      
      - name: Download build artifacts
        uses: actions/download-artifact@v3
        with:
          name: build
          path: .next
      
      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v20
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: '--prod'
      
      - name: Deploy Edge Functions to Supabase
        run: |
          npm install -g supabase
          supabase functions deploy --project-ref ${{ secrets.SUPABASE_PROJECT_REF }}
        env:
          SUPABASE_ACCESS_TOKEN: ${{ secrets.SUPABASE_ACCESS_TOKEN }}
      
      - name: Run database migrations
        run: |
          supabase db push --project-ref ${{ secrets.SUPABASE_PROJECT_REF }}
        env:
          SUPABASE_ACCESS_TOKEN: ${{ secrets.SUPABASE_ACCESS_TOKEN }}
      
      - name: Notify deployment
        run: |
          curl -X POST -H "Content-Type: application/json" \
            -d '{"text":"✅ Deployment successful: ${{ github.sha }}"}' \
            ${{ secrets.SLACK_WEBHOOK_URL }}
```

### 12.2 Environment Configuration

```typescript
// config/env.config.ts
export const env = {
  // Next.js environment
  NODE_ENV: process.env.NODE_ENV || 'development',
  
  // Supabase
  supabase: {
    url: process.env.NEXT_PUBLIC_SUPABASE_URL!,
    anonKey: process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    serviceRoleKey: process.env.SUPABASE_SERVICE_ROLE_KEY!,
  },
  
  // Redis
  redis: {
    host: process.env.REDIS_HOST || 'localhost',
    port: parseInt(process.env.REDIS_PORT || '6379'),
    password: process.env.REDIS_PASSWORD,
  },
  
  // Integrations
  integrations: {
    quickbooks: {
      clientId: process.env.QUICKBOOKS_CLIENT_ID,
      clientSecret: process.env.QUICKBOOKS_CLIENT_SECRET,
      redirectUri: process.env.QUICKBOOKS_REDIRECT_URI,
    },
    fbr: {
      apiKey: process.env.FBR_API_KEY,
      apiUrl: process.env.FBR_API_URL,
    },
    email: {
      sendgrid: {
        apiKey: process.env.SENDGRID_API_KEY,
        fromEmail: process.env.SENDGRID_FROM_EMAIL,
      },
    },
    sms: {
      twilio: {
        accountSid: process.env.TWILIO_ACCOUNT_SID,
        authToken: process.env.TWILIO_AUTH_TOKEN,
        fromNumber: process.env.TWILIO_FROM_NUMBER,
      },
    },
  },
  
  // Payment gateways
  payment: {
    stripe: {
      secretKey: process.env.STRIPE_SECRET_KEY,
      webhookSecret: process.env.STRIPE_WEBHOOK_SECRET,
    },
    jazzcash: {
      merchantId: process.env.JAZZCASH_MERCHANT_ID,
      password: process.env.JAZZCASH_PASSWORD,
      integritySalt: process.env.JAZZCASH_INTEGRITY_SALT,
    },
  },
  
  // Analytics
  analytics: {
    google: {
      measurementId: process.env.NEXT_PUBLIC_GA_MEASUREMENT_ID,
    },
    mixpanel: {
      token: process.env.NEXT_PUBLIC_MIXPANEL_TOKEN,
    },
  },
  
  // Features
  features: {
    enableExport: process.env.NEXT_PUBLIC_ENABLE_EXPORT === 'true',
    enableMobile: process.env.NEXT_PUBLIC_ENABLE_MOBILE === 'true',
    enableAI: process.env.NEXT_PUBLIC_ENABLE_AI === 'true',
  },
};

// Validation
if (!env.supabase.url || !env.supabase.anonKey) {
  throw new Error('Missing Supabase environment variables');
}
```

### 12.3 Monitoring Setup

```typescript
// lib/monitoring/monitoring.service.ts
import * as Sentry from '@sentry/nextjs';
import { Logtail } from '@logtail/node';

export class MonitoringService {
  private logtail: Logtail;
  private metrics: Map<string, number> = new Map();

  constructor() {
    this.logtail = new Logtail(process.env.LOGTAIL_TOKEN!);
    
    if (process.env.NODE_ENV === 'production') {
      Sentry.init({
        dsn: process.env.SENTRY_DSN,
        tracesSampleRate: 0.1,
        environment: process.env.NODE_ENV,
      });
    }
  }

  // Error tracking
  captureError(error: Error, context?: any) {
    console.error('Error:', error, context);
    
    if (process.env.NODE_ENV === 'production') {
      Sentry.captureException(error, {
        extra: context,
      });
    }
    
    this.logtail.error(error.message, {
      context,
      stack: error.stack,
    });
  }

  // Performance monitoring
  measurePerformance(name: string, startTime: number) {
    const duration = Date.now() - startTime;
    
    if (process.env.NODE_ENV === 'production') {
      Sentry.addBreadcrumb({
        message: `Performance: ${name}`,
        data: { duration },
        level: 'info',
      });
    }
    
    this.metrics.set(name, duration);
    this.logtail.info(`Performance metric: ${name}`, { duration });
    
    return duration;
  }

  // Custom metrics
  incrementMetric(name: string, increment: number = 1) {
    const current = this.metrics.get(name) || 0;
    this.metrics.set(name, current + increment);
  }

  // Logging
  logInfo(message: string, context?: any) {
    console.log(message, context);
    this.logtail.info(message, context);
  }

  logWarning(message: string, context?: any) {
    console.warn(message, context);
    this.logtail.warn(message, context);
  }

  logError(message: string, context?: any) {
    console.error(message, context);
    this.logtail.error(message, context);
  }

  // Health checks
  async healthCheck(): Promise<boolean> {
    try {
      // Check database connection
      await supabase.from('tasks').select('count', { count: 'exact', head: true });
      
      // Check Redis
      await this.redis.ping();
      
      return true;
    } catch (error) {
      this.captureError(error, { service: 'health-check' });
      return false;
    }
  }

  // Report generation
  async generateReport(period: string) {
    // Generate performance report
    const report = {
      period,
      metrics: Object.fromEntries(this.metrics),
      errors: await this.getErrorCount(period),
      uptime: await this.getUptime(period),
      performance: await this.getPerformanceMetrics(period),
    };
    
    return report;
  }
}

// Usage example
export class TaskService {
  private monitor = new MonitoringService();

  async createTask(data: any) {
    const startTime = Date.now();
    
    try {
      const result = await this.supabase.from('tasks').insert(data);
      
      this.monitor.measurePerformance('task.create', startTime);
      this.monitor.incrementMetric('tasks.created');
      
      return result;
    } catch (error) {
      this.monitor.captureError(error, { data });
      throw error;
    }
  }
}
```

---

## 13. MONITORING & MAINTENANCE

### 13.1 Daily Maintenance Checklist

```yaml
# .github/ops/daily-maintenance.yml
Daily Maintenance Checklist:
  Database:
    - Check connection pool
    - Verify backups completed
    - Check for slow queries
    - Monitor table sizes
    - Check RLS policies
    
  Caching:
    - Verify Redis connectivity
    - Check cache hit ratio
    - Clear expired caches
    - Monitor memory usage
    
  Performance:
    - Review API response times
    - Check error rates
    - Monitor CPU/Memory usage
    - Verify CDN performance
    
  Security:
    - Review login attempts
    - Check for suspicious activity
    - Verify SSL certificates
    - Update security headers
    
  Integrations:
    - Verify external API connections
    - Check webhook delivery
    - Monitor queue processing
    - Verify payment gateways
    
  User Activity:
    - Monitor active users
    - Check task completion rates
    - Review approval backlogs
    - Verify notifications delivery
```

### 13.2 Database Maintenance

```sql
-- Regular maintenance scripts

-- 1. Vacuum and analyze
VACUUM ANALYZE tasks;
VACUUM ANALYZE task_comments;
VACUUM ANALYZE users;
VACUUM ANALYZE requests;
VACUUM ANALYZE approvals;

-- 2. Reindex if needed
REINDEX INDEX idx_tasks_assigned_to_id;
REINDEX INDEX idx_tasks_status;
REINDEX INDEX idx_tasks_created_at;
REINDEX INDEX idx_task_comments_task_id;
REINDEX INDEX idx_requests_status;

-- 3. Refresh materialized views
REFRESH MATERIALIZED VIEW CONCURRENTLY monthly_performance_mv;
REFRESH MATERIALIZED VIEW CONCURRENTLY user_performance_summary_mv;

-- 4. Check table bloat
SELECT 
  schemaname,
  tablename,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS total_size,
  pg_size_pretty(pg_relation_size(schemaname||'.'||tablename)) AS table_size,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename) - pg_relation_size(schemaname||'.'||tablename)) AS index_size,
  (pg_total_relation_size(schemaname||'.'||tablename) - pg_relation_size(schemaname||'.'||tablename)) * 100 / pg_total_relation_size(schemaname||'.'||tablename) AS index_percentage
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

-- 5. Check for long-running queries
SELECT 
  pid,
  usename,
  application_name,
  client_addr,
  state,
  query,
  now() - query_start AS duration
FROM pg_stat_activity
WHERE state != 'idle'
AND now() - query_start > interval '5 minutes'
ORDER BY duration DESC;
```

### 13.3 Backup Strategy

```yaml
# backup-config.yaml
Backup Strategy:
  Frequency:
    - Full Database Backup: Daily at 2 AM
    - Incremental Backup: Every 6 hours
    - WAL Archiving: Continuous
    
  Retention:
    - Daily Backups: 30 days
    - Weekly Backups: 12 weeks
    - Monthly Backups: 12 months
    
  Storage:
    - Primary: AWS S3 (us-east-1)
    - Secondary: Google Cloud Storage
    - Disaster Recovery: Another region
    
  Verification:
    - Automated restore test: Weekly
    - Integrity check: Daily
    - Checksum validation: On each backup
    
  Recovery:
    - Point-in-Time Recovery: Available
    - Estimated RTO: 30 minutes
    - Estimated RPO: 15 minutes
```

---

## 14. RISK MITIGATION

### 14.1 Risk Register

| Risk | Impact | Probability | Mitigation Strategy | Contingency Plan |
|------|--------|-------------|-------------------|------------------|
| Solo Developer Burnout | Very High | Medium | - Maintain 40-hour work week<br>- Take regular breaks<br>- Use AI tools for efficiency<br>- Consider part-time help | - Hire additional developer<br>- Reduce scope<br>- Extend timeline |
| Technology Changes | Medium | Medium | - Use stable versions<br>- Follow LTS releases<br>- Regular dependency updates | - Backward compatibility<br>- Rollback procedure<br>- Phase upgrades |
| Scope Creep | Very High | High | - Strict change control<br>- Monthly priority reviews<br>- Feature freeze periods | - Defer features to Phase 2<br>- Minimum viable product<br>- Clear prioritization |
| Data Migration Issues | High | Medium | - Build robust migration tools<br>- Test with sample data<br>- Migration rehearsals | - Rollback capability<br>- Data validation scripts<br>- Extended hypercare |
| User Adoption | High | Medium | - Early user involvement<br>- Comprehensive training<br>- User champions program | - Extra training sessions<br>- 24/7 support period<br>- Feedback collection |
| Security Breach | Very High | Low | - Regular security audits<br>- Penetration testing<br>- RLS implementation | - Incident response plan<br>- Data breach notification<br>- System isolation |
| Performance Issues | High | Medium | - Performance testing<br>- Optimization from start<br>- Monitoring early | - Horizontal scaling<br>- Cache optimization<br>- Query tuning |
| Integration Failures | High | Medium | - Thorough testing<br>- Circuit breakers<br>- Retry mechanisms | - Fallback procedures<br>- Manual processing<br>- Alternative providers |
| Regulatory Changes | Medium | Medium | - Flexible tax/compliance engine<br>- Regular updates<br>- Industry monitoring | - Quick configuration updates<br>- Regulatory consulting<br>- Grace period handling |

### 14.2 Disaster Recovery Plan

```yaml
Disaster Recovery Plan:
  Infrastructure Failure:
    - Detection: Monitoring alerts (5 minutes)
    - Response: Identify affected components
    - Recovery: 
      - Database: Restore from latest backup
      - Application: Redeploy from last stable build
      - Cache: Rebuild from database
    - Testing: Quarterly DR drills
    
  Data Corruption:
    - Detection: Data validation checks (1 hour)
    - Response: Stop all write operations
    - Recovery: 
      - Point-in-time restore
      - Verify data integrity
      - Replay transactions
    - Testing: Monthly backup verification
    
  Service Outage:
    - Detection: Uptime monitoring (1 minute)
    - Response: Auto-scaling/self-healing
    - Recovery: 
      - Failover to backup region
      - Load balancing adjustments
      - Cache warming
    - Testing: Monthly failover tests
    
  Cyber Attack:
    - Detection: Security monitoring (5 minutes)
    - Response: Isolate affected systems
    - Recovery: 
      - Clean from known good state
      - Security patches
      - Rotate credentials
    - Testing: Quarterly penetration tests
```

---

## 15. RESOURCE REQUIREMENTS

### 15.1 Development Team

```yaml
Core Team:
  Lead Developer (Amaan):
    - Role: Full-stack Architect/Developer
    - Hours: 40 hours/week
    - Responsibilities: 
      - System architecture
      - Database design
      - Core module development
      - Code review
      - Deployment
  
  Recommended Additional Resources:
    Junior Developer (Part-time):
      - Role: Frontend/UI Developer
      - Hours: 20 hours/week
      - Rate: $15-25/hour
      - Responsibilities:
        - UI component implementation
        - Bug fixes
        - Unit testing
        - Documentation
    
    UI/UX Designer (Part-time):
      - Role: Design System Specialist
      - Hours: 10 hours/week (first 6 months)
      - Rate: $30-50/hour
      - Responsibilities:
        - Design system
        - Prototypes
        - User testing
        - Accessibility
    
    QA Engineer (Part-time):
      - Role: Quality Assurance Specialist
      - Hours: 15 hours/week
      - Rate: $20-35/hour
      - Responsibilities:
        - Test case creation
        - Automation testing
        - Regression testing
        - Performance testing
    
    DevOps Engineer (Part-time):
      - Role: Infrastructure Specialist
      - Hours: 5-10 hours/week
      - Rate: $40-60/hour
      - Responsibilities:
        - CI/CD pipeline
        - Monitoring setup
        - Security audits
        - Performance optimization

Total Monthly Cost (Part-time team): $8,000-12,000
```

### 15.2 Infrastructure Costs

```yaml
Infrastructure Costs (Monthly):
  Hosting:
    Vercel (Production): $50-200
      - Premium plan with team features
      - Edge functions included
      - Analytics included
    
    Supabase:
      - Database: $25-100
        - Depending on size and connections
      - Storage: $0.021/GB
        - Estimated 100GB initial
      - Edge Functions: $2/million invocations
    
  Services:
    Redis (Upstash): $20-50
      - 5GB persistent storage
      - Daily backups
    
    Sentry (Monitoring): $29-99
      - Error tracking
      - Performance monitoring
    
    Logtail (Logging): $0-50
      - 1GB free, then $0.60/GB
    
    Email (SendGrid): $15-100
      - 10,000 free emails/month
      - Additional $0.60/1000
    
    SMS (Twilio): $10-100
      - $0.0075/SMS
    
    Total: $150-700/month
```

---

## 16. SUCCESS METRICS & KPIS

### 16.1 System Performance KPIs

| Metric | Target | Measurement |
|--------|--------|-------------|
| API Response Time | <200ms | P95 latency |
| Page Load Time | <3s | Core Web Vitals |
| Database Query Time | <100ms | P95 query duration |
| Task Board Load | <500ms | Time to interactive |
| Cache Hit Ratio | >80% | Cache hits/total requests |
| Error Rate | <0.1% | 5xx errors/total requests |
| Uptime | >99.9% | Monitoring tools |
| Daily Active Users | >80% of employees | Auth events |
| Task Completion Time | <80% of due | Average days to complete |

### 16.2 Business KPIs

| Metric | Target | Measurement |
|--------|--------|-------------|
| User Adoption | >90% | Active users/Total employees |
| Task Completion Rate | >95% | Completed tasks/Total tasks |
| Time Savings | 30% reduction | Manual vs automated processes |
| Cost Savings | 20% reduction | Before/after implementation |
| User Satisfaction | >4.0/5.0 | Quarterly surveys |
| ROI Achieved | >200% | Cost savings/Investment |
| Revenue Impact | +10% | Efficiency gains |

### 16.3 Module-Specific KPIs

```typescript
// lib/metrics/kpi.service.ts
export class KPIService {
  async getTaskManagementKPIs() {
    const metrics = {
      tasks: {
        total: 0,
        completed: 0,
        inProgress: 0,
        overdue: 0,
        completionRate: 0,
        avgCompletionTime: 0,
      },
      comments: {
        total: 0,
        concerns: 0,
        concernRate: 0,
      },
      scoring: {
        avgScore: 0,
        topPerformers: [],
      },
      requests: {
        total: 0,
        approved: 0,
        rejected: 0,
        avgApprovalTime: 0,
      }
    };
    
    // Query database for real metrics
    // ... implementation
    
    return metrics;
  }

  async generateExecutiveDashboard() {
    const taskKPIs = await this.getTaskManagementKPIs();
    const productionKPIs = await this.getProductionKPIs();
    const financialKPIs = await this.getFinancialKPIs();
    
    return {
      overview: {
        health: 'Good',
        alerts: 2,
        updates: 15,
      },
      tasks: taskKPIs,
      production: productionKPIs,
      finance: financialKPIs,
      charts: {
        taskTrends: await this.getTaskTrends(),
        performanceDistribution: await this.getPerformanceDistribution(),
        requestFlow: await this.getRequestFlow(),
      }
    };
  }
}
```

---

## 17. APPENDICES

### 17.1 Glossary of Terms

| Term | Definition |
|------|------------|
| **BOM** | Bill of Materials - List of raw materials and components required for production |
| **COA** | Chart of Accounts - List of all financial accounts in the system |
| **ERP** | Enterprise Resource Planning - Integrated management of business processes |
| **FBR** | Federal Board of Revenue - Pakistan's tax authority |
| **GRN** | Goods Receipt Note - Document acknowledging receipt of goods |
| **LC** | Letter of Credit - Payment guarantee for international trade |
| **LIMS** | Laboratory Information Management System |
| **NTN** | National Tax Number - Pakistan's tax identification number |
| **OEE** | Overall Equipment Effectiveness - Manufacturing productivity metric |
| **RBAC** | Role-Based Access Control - Security model based on user roles |
| **RLS** | Row Level Security - Database security at row level |
| **STRN** | Sales Tax Registration Number - Pakistan's sales tax identification |
| **WHT** | Withholding Tax - Tax deducted at source |
| **WMS** | Warehouse Management System |

### 17.2 References

1. **Next.js Documentation** - https://nextjs.org/docs
2. **Supabase Documentation** - https://supabase.com/docs
3. **TypeScript Handbook** - https://www.typescriptlang.org/docs
4. **Tailwind CSS Documentation** - https://tailwindcss.com/docs
5. **PostgreSQL Documentation** - https://www.postgresql.org/docs
6. **React Documentation** - https://react.dev/learn
7. **FBR Integration Guide** - https://www.fbr.gov.pk
8. **QuickBooks API Documentation** - https://developer.intuit.com
9. **Stripe API Documentation** - https://stripe.com/docs/api

---

## CONCLUSION

This comprehensive technical plan provides a complete roadmap for building an enterprise-grade ERP system for Rice World. The plan covers all aspects of the system, including:

1. **Complete System Architecture** - Modern, scalable architecture using Next.js and Supabase
2. **Detailed Database Design** - 180+ tables with comprehensive relationships and RLS policies
3. **Full-Stack Implementation** - Complete frontend and backend services
4. **Task Management Integration** - Comprehensive task, request, and document management
5. **Security & Compliance** - Enterprise-level security with RBAC and RLS
6. **Performance Optimization** - Caching, indexing, and query optimization strategies
7. **Testing & Quality** - Comprehensive testing at all levels
8. **Deployment & DevOps** - CI/CD, monitoring, and maintenance procedures
9. **Risk Management** - Risk mitigation and disaster recovery plans
10. **Resource Planning** - Clear resource requirements and cost estimates

The system will transform the rice factory operations by providing:
- **Full Business Process Integration** - From procurement to sales and finance
- **Advanced Task Management** - Complete tracking of all work activities
- **Data-Driven Decision Making** - Rich analytics and business intelligence
- **Regulatory Compliance** - Full compliance with Pakistani tax laws
- **Mobile Accessibility** - iOS/Android apps for field operations
- **Scalability** - Architecture designed for 1000+ concurrent users
- **ROI Achievement** - Significant efficiency gains and cost savings

With this comprehensive technical plan, the development team has a clear roadmap to build a world-class ERP system for the rice industry.