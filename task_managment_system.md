# RICE WORLD ERP - TASK MANAGEMENT SYSTEM
## COMPLETE TECHNICAL DOCUMENTATION

---

## TABLE OF CONTENTS

1. [System Overview](#1-system-overview)
2. [Core Architecture](#2-core-architecture)
3. [Database Schema](#3-database-schema)
4. [API Services](#4-api-services)
5. [Frontend Components](#5-frontend-components)
6. [Scoring & Analytics Engine](#6-scoring--analytics-engine)
7. [Notification System](#7-notification-system)
8. [Mobile Integration](#8-mobile-integration)
9. [Security Implementation](#9-security-implementation)
10. [Performance Optimization](#10-performance-optimization)
11. [Testing Strategy](#11-testing-strategy)
12. [Deployment Guide](#12-deployment-guide)

---

## 1. SYSTEM OVERVIEW

### 1.1 Purpose & Scope

The Task Management System (TMS) is a comprehensive module within the Rice World ERP that enables:

1. **Task Assignment & Tracking**: Create, assign, and monitor tasks across departments
2. **Request Management**: Submit and approve departmental requests (new hires, equipment, etc.)
3. **Document Management**: Attach and link documents to tasks for traceability
4. **Collaboration**: Comment, raise concerns, and discuss tasks
5. **Performance Scoring**: Track individual contributions and performance
6. **Analytics**: Generate insights on task completion, efficiency, and workload

### 1.2 Key Features

| Feature | Description | Technical Implementation |
|---------|-------------|--------------------------|
| Task Board | Kanban-style task visualization | react-beautiful-dnd + Zustand |
| Task List | Filterable, sortable task table | TanStack Table + React Query |
| Comments | Threaded discussions with concern flagging | Supabase + Real-time subscriptions |
| Requests | Structured approval workflows | Edge Functions + Workflow Engine |
| Scoring | Automated performance calculation | PostgreSQL Functions + Triggers |
| Documents | Centralized file management | Supabase Storage + Versioning |
| Notifications | Multi-channel alerts | SendGrid + Twilio + In-app |

### 1.3 User Roles & Permissions

```typescript
// Roles hierarchy for Task Management
const TaskRoles = {
  SUPER_ADMIN: 'super_admin',     // Full system access
  OWNER: 'owner',                 // Strategic oversight
  HR_ADMIN: 'hr_admin',           // HR operations
  DEPARTMENT_HEAD: 'department_head', // Department oversight
  MANAGER: 'manager',             // Team management
  EMPLOYEE: 'employee',           // Task execution
  VIEWER: 'viewer'                // Read-only access
};

// Permission matrix
const TaskPermissions = {
  CREATE_TASK: ['super_admin', 'owner', 'department_head', 'manager'],
  ASSIGN_TASK: ['super_admin', 'owner', 'department_head', 'manager'],
  VIEW_TASK: ['super_admin', 'owner', 'department_head', 'manager', 'employee', 'viewer'],
  UPDATE_TASK: ['super_admin', 'owner', 'department_head', 'manager'],
  DELETE_TASK: ['super_admin', 'owner'],
  COMMENT_TASK: ['super_admin', 'owner', 'department_head', 'manager', 'employee'],
  APPROVE_TASK: ['super_admin', 'owner', 'department_head'],
  CREATE_REQUEST: ['super_admin', 'owner', 'department_head', 'manager', 'employee'],
  APPROVE_REQUEST: ['super_admin', 'owner', 'department_head'],
  VIEW_ANALYTICS: ['super_admin', 'owner', 'department_head', 'manager'],
  MANAGE_DOCUMENTS: ['super_admin', 'owner', 'department_head', 'manager', 'employee']
};
```

---

## 2. CORE ARCHITECTURE

### 2.1 Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           PRESENTATION LAYER                                │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐      │
│  │ Task Board  │  │ Task List   │  │ Task Detail │  │ Dashboard   │      │
│  │ (Kanban)    │  │ (Table)     │  │ (Detail)    │  │ (Analytics) │      │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘      │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐      │
│  │ Request Form│  │ Approval    │  │ Comment     │  │ Document    │      │
│  │ (Wizard)    │  │ Dashboard   │  │ Thread      │  │ Manager     │      │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘      │
└─────────────────────────────┬───────────────────────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────────────────────┐
│                            STATE MANAGEMENT                                │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │  Zustand Stores                                                      │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐│ │
│  │  │ Task Store  │  │ Request     │  │ Document    │  │ Notification││ │
│  │  │             │  │ Store       │  │ Store       │  │ Store       ││ │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘│ │
│  └───────────────────────────────────────────────────────────────────────┘ │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │  React Query Cache                                                    │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                  │ │
│  │  │ useTasks    │  │ useRequests │  │ useComments │                  │ │
│  │  │ Query       │  │ Query       │  │ Query       │                  │ │
│  │  └─────────────┘  └─────────────┘  └─────────────┘                  │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────┬───────────────────────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────────────────────┐
│                            SERVICE LAYER                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │  Core Services                                                       │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐│ │
│  │  │ TaskService │  │ Request     │  │ Document    │  │ Analytics   ││ │
│  │  │             │  │ Service     │  │ Service     │  │ Service     ││ │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘│ │
│  └───────────────────────────────────────────────────────────────────────┘ │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │  Support Services                                                     │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐│ │
│  │  │ Notification│  │ Scoring     │  │ Workflow    │  │ Audit       ││ │
│  │  │ Service     │  │ Service     │  │ Engine      │  │ Service     ││ │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘│ │
│  └───────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────┬───────────────────────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────────────────────┐
│                          DATABASE LAYER                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │  PostgreSQL (Supabase)                                               │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐│ │
│  │  │ Tasks       │  │ Requests    │  │ Documents   │  │ Comments    ││ │
│  │  │ Table       │  │ Table       │  │ Table       │  │ Table       ││ │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘│ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐│ │
│  │  │ Approvals   │  │ Workflows   │  │ Scores      │  │ Audit Logs  ││ │
│  │  │ Table       │  │ Table       │  │ Table       │  │ Table       ││ │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘│ │
│  └───────────────────────────────────────────────────────────────────────┘ │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │  Storage (Supabase)                                                   │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                  │ │
│  │  │ Task        │  │ Request     │  │ Profile     │                  │ │
│  │  │ Attachments │  │ Documents   │  │ Images      │                  │ │
│  │  └─────────────┘  └─────────────┘  └─────────────┘                  │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Technology Stack - Task Management Specific

```typescript
{
  "frontend": {
    "uiComponents": {
      "dragAndDrop": "react-beautiful-dnd",
      "tables": "TanStack Table v8",
      "forms": "React Hook Form",
      "validation": "Zod",
      "modals": "Headless UI / Radix UI",
      "datePicker": "React DayPicker",
      "fileUpload": "react-dropzone",
      "richText": "react-quill"
    },
    "state": {
      "global": "Zustand",
      "server": "React Query (TanStack Query)",
      "forms": "React Hook Form",
      "auth": "Supabase Auth"
    }
  },
  "backend": {
    "database": "Supabase PostgreSQL 15+",
    "serverless": "Supabase Edge Functions (Deno)",
    "caching": "Redis (Upstash)",
    "queue": "BullMQ",
    "search": "PostgreSQL Full-Text Search"
  },
  "notifications": {
    "email": "SendGrid / Resend",
    "sms": "Twilio",
    "inApp": "Supabase Realtime / WebSockets"
  }
}
```

---

## 3. DATABASE SCHEMA

### 3.1 Task Management Core Tables

```sql
-- ============================================
-- TASK MANAGEMENT DATABASE SCHEMA
-- ============================================

-- 1. Task Categories (Hierarchical)
CREATE TABLE task_categories (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  category_code VARCHAR(50) UNIQUE NOT NULL,
  category_name VARCHAR(255) NOT NULL,
  parent_category_id UUID REFERENCES task_categories(id),
  category_type VARCHAR(50) NOT NULL CHECK (category_type IN (
    'Maintenance', 'Production', 'Quality', 'HR', 'Admin', 
    'Supply Chain', 'Commercial', 'Electrical', 'Civil', 
    'Project', 'Study', 'Review', 'Implementation', 'Execution'
  )),
  icon VARCHAR(50),
  color VARCHAR(20),
  default_priority VARCHAR(20) DEFAULT 'Medium' CHECK (default_priority IN (
    'Low', 'Medium', 'High', 'Urgent/Overdue'
  )),
  approval_required BOOLEAN DEFAULT false,
  scoring_weight DECIMAL(5,2) DEFAULT 1.0,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 2. Job Locations
CREATE TABLE job_locations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  location_code VARCHAR(50) UNIQUE NOT NULL,
  location_name VARCHAR(255) NOT NULL,
  description TEXT,
  department_id UUID REFERENCES departments(id),
  parent_location_id UUID REFERENCES job_locations(id),
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 3. Main Tasks Table
CREATE TABLE tasks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  task_number VARCHAR(50) UNIQUE NOT NULL,
  title VARCHAR(500) NOT NULL,
  description TEXT,
  
  -- Hierarchy
  parent_task_id UUID REFERENCES tasks(id) ON DELETE CASCADE,
  task_level INTEGER DEFAULT 0,
  
  -- Categorization
  category_id UUID REFERENCES task_categories(id),
  job_location_id UUID REFERENCES job_locations(id),
  
  -- Assignment
  assigned_to_id UUID REFERENCES users(id),
  created_by_id UUID REFERENCES users(id) NOT NULL,
  requested_by_id UUID REFERENCES users(id),
  
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
  bucket VARCHAR(50) GENERATED ALWAYS AS (
    CASE 
      WHEN pending_days <= 5 THEN '0 to 5 days'
      WHEN pending_days <= 10 THEN '5 to 10 days'
      WHEN pending_days <= 15 THEN '10 to 15 days'
      WHEN pending_days <= 20 THEN '15 to 20 days'
      WHEN pending_days <= 25 THEN '20 to 25 days'
      WHEN pending_days <= 30 THEN '25 to 30 days'
      ELSE 'Above 30 days'
    END
  ) STORED,
  
  -- Scoring System
  complexity_score DECIMAL(5,2) DEFAULT 0,
  quality_score DECIMAL(5,2) DEFAULT 0,
  timeliness_score DECIMAL(5,2) DEFAULT 0,
  concern_score DECIMAL(5,2) DEFAULT 0,
  task_score DECIMAL(10,2) GENERATED ALWAYS AS (
    (complexity_score * 0.3) + (quality_score * 0.4) + 
    (timeliness_score * 0.2) + (concern_score * 0.1)
  ) STORED,
  contribution_score DECIMAL(10,2) DEFAULT 0,
  
  -- Metadata
  approved_by_id UUID REFERENCES users(id),
  approval_status VARCHAR(20) DEFAULT 'Pending' CHECK (approval_status IN (
    'Pending', 'Approved', 'Rejected', 'N/A'
  )),
  highlighted_by VARCHAR(255),
  is_overdue_days INTEGER DEFAULT 0,
  
  -- Traceability
  source_module VARCHAR(50), -- Procurement, Production, Sales, etc.
  source_record_id UUID,
  
  -- Timestamps
  assigned_date DATE,
  start_date TIMESTAMP WITH TIME ZONE,
  completed_at TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  -- Constraints
  CONSTRAINT fk_assigned_to FOREIGN KEY (assigned_to_id) REFERENCES users(id) ON DELETE SET NULL,
  CONSTRAINT fk_created_by FOREIGN KEY (created_by_id) REFERENCES users(id) ON DELETE CASCADE,
  CONSTRAINT fk_category FOREIGN KEY (category_id) REFERENCES task_categories(id) ON DELETE SET NULL,
  CONSTRAINT fk_location FOREIGN KEY (job_location_id) REFERENCES job_locations(id) ON DELETE SET NULL
);

-- Indexes for Performance
CREATE INDEX idx_tasks_assigned_to_id ON tasks(assigned_to_id);
CREATE INDEX idx_tasks_created_by_id ON tasks(created_by_id);
CREATE INDEX idx_tasks_status ON tasks(status);
CREATE INDEX idx_tasks_priority ON tasks(priority);
CREATE INDEX idx_tasks_due_date ON tasks(due_date);
CREATE INDEX idx_tasks_category_id ON tasks(category_id);
CREATE INDEX idx_tasks_source ON tasks(source_module, source_record_id);
CREATE INDEX idx_tasks_parent ON tasks(parent_task_id);
CREATE INDEX idx_tasks_created_at ON tasks(created_at DESC);
CREATE INDEX idx_tasks_completion ON tasks(completion_percentage);
CREATE INDEX idx_tasks_pending_days ON tasks(pending_days);

-- Composite Indexes for Common Queries
CREATE INDEX idx_tasks_assigned_status ON tasks(assigned_to_id, status);
CREATE INDEX idx_tasks_status_due ON tasks(status, due_date);
CREATE INDEX idx_tasks_category_status ON tasks(category_id, status);
CREATE INDEX idx_tasks_priority_status ON tasks(priority, status);

-- Full-Text Search Index
CREATE INDEX idx_tasks_search ON tasks 
  USING GIN (to_tsvector('english', title || ' ' || COALESCE(description, '')));

-- 4. Task Dependencies
CREATE TABLE task_dependencies (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  task_id UUID REFERENCES tasks(id) ON DELETE CASCADE,
  predecessor_task_id UUID REFERENCES tasks(id) ON DELETE CASCADE,
  dependency_type VARCHAR(20) DEFAULT 'Finish-to-Start' CHECK (
    dependency_type IN ('Finish-to-Start', 'Start-to-Start', 'Finish-to-Finish', 'Start-to-Finish')
  ),
  lag_days INTEGER DEFAULT 0,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  UNIQUE(task_id, predecessor_task_id)
);

-- 5. Task Multiple Assignees
CREATE TABLE task_assignments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  task_id UUID REFERENCES tasks(id) ON DELETE CASCADE,
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  assigned_by UUID REFERENCES users(id),
  assigned_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  role VARCHAR(50) DEFAULT 'Assignee' CHECK (role IN ('Assignee', 'Reviewer', 'Observer')),
  UNIQUE(task_id, user_id)
);

-- 6. Task Comments
CREATE TABLE task_comments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  task_id UUID REFERENCES tasks(id) ON DELETE CASCADE,
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  comment TEXT NOT NULL,
  comment_type VARCHAR(20) DEFAULT 'Regular' CHECK (comment_type IN (
    'Regular', 'Concern', 'Highlight', 'Update'
  )),
  is_concern BOOLEAN DEFAULT false,
  is_highlight BOOLEAN DEFAULT false,
  parent_comment_id UUID REFERENCES task_comments(id) ON DELETE CASCADE,
  sentiment_score DECIMAL(3,2),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_task_comments_task ON task_comments(task_id);
CREATE INDEX idx_task_comments_user ON task_comments(user_id);
CREATE INDEX idx_task_comments_created_at ON task_comments(created_at DESC);
CREATE INDEX idx_task_comments_parent ON task_comments(parent_comment_id);
CREATE INDEX idx_task_comments_concern ON task_comments(is_concern) WHERE is_concern = true;

-- 7. Task Attachments
CREATE TABLE task_attachments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  task_id UUID REFERENCES tasks(id) ON DELETE CASCADE,
  uploaded_by_id UUID REFERENCES users(id),
  file_name VARCHAR(255) NOT NULL,
  file_url TEXT NOT NULL,
  file_type VARCHAR(100),
  file_size INTEGER,
  mime_type VARCHAR(100),
  file_hash VARCHAR(255),
  uploaded_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_task_attachments_task ON task_attachments(task_id);

-- 8. Task Status History
CREATE TABLE task_status_history (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  task_id UUID REFERENCES tasks(id) ON DELETE CASCADE,
  previous_status VARCHAR(30),
  new_status VARCHAR(30) NOT NULL,
  changed_by_id UUID REFERENCES users(id),
  comment TEXT,
  changed_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_task_status_history_task ON task_status_history(task_id);
CREATE INDEX idx_task_status_history_changed_at ON task_status_history(changed_at DESC);
```

### 3.2 Requests & Approvals Schema

```sql
-- 9. Request Types
CREATE TABLE request_types (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  type_code VARCHAR(50) UNIQUE NOT NULL,
  type_name VARCHAR(255) NOT NULL,
  category VARCHAR(50) NOT NULL CHECK (category IN (
    'HR', 'Procurement', 'Finance', 'Admin', 'Maintenance'
  )),
  description TEXT,
  default_priority VARCHAR(20) DEFAULT 'Medium' CHECK (default_priority IN (
    'Low', 'Medium', 'High', 'Urgent/Overdue'
  )),
  requires_approval BOOLEAN DEFAULT true,
  approval_workflow_id UUID REFERENCES approval_workflows(id),
  form_schema JSONB, -- Dynamic form configuration
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 10. Requests Table
CREATE TABLE requests (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  request_number VARCHAR(50) UNIQUE NOT NULL,
  task_id UUID REFERENCES tasks(id) ON DELETE CASCADE,
  request_type_id UUID REFERENCES request_types(id),
  
  -- Request Data (JSONB for flexible structure)
  request_data JSONB NOT NULL,
  
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
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_requests_status ON requests(status);
CREATE INDEX idx_requests_department ON requests(department_id);
CREATE INDEX idx_requests_requested_by ON requests(requested_by_id);
CREATE INDEX idx_requests_task ON requests(task_id);
CREATE INDEX idx_requests_type ON requests(request_type_id);

-- 11. Approval Workflows
CREATE TABLE approval_workflows (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  workflow_code VARCHAR(50) UNIQUE NOT NULL,
  workflow_name VARCHAR(255) NOT NULL,
  description TEXT,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 12. Approval Workflow Steps
CREATE TABLE approval_workflow_steps (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  workflow_id UUID REFERENCES approval_workflows(id) ON DELETE CASCADE,
  step_order INTEGER NOT NULL,
  approver_role_id UUID REFERENCES roles(id),
  approver_user_id UUID REFERENCES users(id),
  is_parallel BOOLEAN DEFAULT false,
  escalation_days INTEGER DEFAULT 3,
  required_approvals INTEGER DEFAULT 1,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  UNIQUE(workflow_id, step_order)
);

-- 13. Approvals Table
CREATE TABLE approvals (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  request_id UUID REFERENCES requests(id) ON DELETE CASCADE,
  workflow_step_id UUID REFERENCES approval_workflow_steps(id),
  approver_id UUID REFERENCES users(id),
  status VARCHAR(20) DEFAULT 'Pending' CHECK (status IN ('Pending', 'Approved', 'Rejected')),
  comments TEXT,
  approved_at TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_approvals_request ON approvals(request_id);
CREATE INDEX idx_approvals_approver ON approvals(approver_id);
CREATE INDEX idx_approvals_status ON approvals(status);
```

### 3.3 Document Management Schema

```sql
-- 14. Document Types
CREATE TABLE document_types (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  type_code VARCHAR(50) UNIQUE NOT NULL,
  type_name VARCHAR(255) NOT NULL,
  description TEXT,
  required_for_modules TEXT[],
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 15. Documents Repository
CREATE TABLE documents (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  document_number VARCHAR(50) UNIQUE NOT NULL,
  document_type_id UUID REFERENCES document_types(id),
  document_name VARCHAR(500) NOT NULL,
  description TEXT,
  
  -- Linking
  source_module VARCHAR(50) NOT NULL, -- Task, Request, Deal, etc.
  source_record_id UUID NOT NULL,
  
  -- File Information
  file_name VARCHAR(255) NOT NULL,
  file_url TEXT NOT NULL,
  file_type VARCHAR(100),
  file_size INTEGER,
  mime_type VARCHAR(100),
  file_hash VARCHAR(255),
  
  -- Versioning
  version INTEGER DEFAULT 1,
  is_latest_version BOOLEAN DEFAULT true,
  previous_version_id UUID REFERENCES documents(id),
  
  -- Metadata
  uploaded_by_id UUID REFERENCES users(id),
  uploaded_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  -- Approval
  status VARCHAR(30) DEFAULT 'Draft' CHECK (status IN (
    'Draft', 'Submitted', 'Reviewed', 'Approved', 'Rejected'
  )),
  approved_by_id UUID REFERENCES users(id),
  approved_at TIMESTAMP WITH TIME ZONE,
  
  -- Tags
  tags TEXT[],
  
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_documents_source ON documents(source_module, source_record_id);
CREATE INDEX idx_documents_type ON documents(document_type_id);
CREATE INDEX idx_documents_uploaded_by ON documents(uploaded_by_id);
CREATE INDEX idx_documents_status ON documents(status);
CREATE INDEX idx_documents_version ON documents(version, is_latest_version);

-- 16. Document Versions
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
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_doc_versions_document ON document_versions(document_id);
```

### 3.4 Scoring & Analytics Schema

```sql
-- 17. User Performance Scores
CREATE TABLE user_performance_scores (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  
  -- Score Components
  task_completion_score DECIMAL(5,2) DEFAULT 0,
  task_quality_score DECIMAL(5,2) DEFAULT 0,
  concern_value_score DECIMAL(5,2) DEFAULT 0,
  remark_insight_score DECIMAL(5,2) DEFAULT 0,
  collaboration_score DECIMAL(5,2) DEFAULT 0,
  leadership_score DECIMAL(5,2) DEFAULT 0,
  
  -- Composite Score
  overall_score DECIMAL(5,2) GENERATED ALWAYS AS (
    (task_completion_score * 0.25 + 
     task_quality_score * 0.25 + 
     concern_value_score * 0.15 + 
     remark_insight_score * 0.15 + 
     collaboration_score * 0.10 + 
     leadership_score * 0.10)
  ) STORED,
  
  -- Ranking
  percentile_rank INTEGER,
  performance_band VARCHAR(20) GENERATED ALWAYS AS (
    CASE 
      WHEN overall_score >= 90 THEN 'Top Performer'
      WHEN overall_score >= 75 THEN 'High Performer'
      WHEN overall_score >= 60 THEN 'Solid Performer'
      WHEN overall_score >= 40 THEN 'Needs Improvement'
      ELSE 'Underperformer'
    END
  ) STORED,
  
  -- Period
  evaluation_period VARCHAR(20) NOT NULL, -- e.g., '2024-Q1'
  calculated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  UNIQUE(user_id, evaluation_period)
);

CREATE INDEX idx_perf_user ON user_performance_scores(user_id);
CREATE INDEX idx_perf_period ON user_performance_scores(evaluation_period);
CREATE INDEX idx_perf_overall ON user_performance_scores(overall_score DESC);

-- 18. Concern Analysis
CREATE TABLE concern_analysis (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  comment_id UUID REFERENCES task_comments(id) ON DELETE CASCADE,
  sentiment_score DECIMAL(3,2),
  concern_category VARCHAR(50) CHECK (concern_category IN (
    'Process', 'Resource', 'Management', 'Technical', 'Quality', 'Safety', 'Other'
  )),
  concern_severity VARCHAR(20) CHECK (concern_severity IN ('Low', 'Medium', 'High', 'Critical')),
  keywords TEXT[],
  analyzed_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  analyzed_by VARCHAR(50) DEFAULT 'AI'
);

-- 19. Task Analytics Summary (Materialized View)
CREATE MATERIALIZED VIEW task_analytics_summary AS
SELECT 
  DATE_TRUNC('month', created_at) AS month,
  department_id,
  COUNT(*) AS total_tasks,
  COUNT(CASE WHEN status = 'Done / Completed' THEN 1 END) AS completed_tasks,
  COUNT(CASE WHEN status = 'Urgent / Overdue' THEN 1 END) AS overdue_tasks,
  AVG(task_score) AS avg_score,
  AVG(completion_percentage) AS avg_completion,
  AVG(EXTRACT(DAY FROM (completed_at - created_at))) AS avg_completion_days,
  COUNT(DISTINCT assigned_to_id) AS active_workers
FROM tasks t
LEFT JOIN users u ON t.assigned_to_id = u.id
WHERE created_at >= CURRENT_DATE - INTERVAL '12 months'
GROUP BY DATE_TRUNC('month', created_at), department_id;

CREATE UNIQUE INDEX idx_task_analytics_month ON task_analytics_summary(month, department_id);
REFRESH MATERIALIZED VIEW CONCURRENTLY task_analytics_summary;

-- 20. Audit Logs
CREATE TABLE audit_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  action VARCHAR(50) NOT NULL,
  resource_type VARCHAR(50) NOT NULL,
  resource_id UUID NOT NULL,
  change_data JSONB,
  ip_address INET,
  user_agent TEXT,
  timestamp TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_audit_user ON audit_logs(user_id);
CREATE INDEX idx_audit_resource ON audit_logs(resource_type, resource_id);
CREATE INDEX idx_audit_timestamp ON audit_logs(timestamp DESC);
```

### 3.5 Database Functions

```sql
-- ============================================
-- TASK MANAGEMENT DATABASE FUNCTIONS
-- ============================================

-- 1. Generate Task Number
CREATE OR REPLACE FUNCTION generate_task_number()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
DECLARE
  year_string TEXT;
  sequence_number TEXT;
BEGIN
  year_string := TO_CHAR(NEW.created_at, 'YYYY');
  
  -- Get next sequence number for the year
  SELECT LPAD(CAST(COALESCE(MAX(CAST(SUBSTRING(task_number FROM '\d{6}$') AS INTEGER)), 0) + 1 AS TEXT), 6, '0')
  INTO sequence_number
  FROM tasks
  WHERE task_number LIKE 'TASK-' || year_string || '-%';
  
  NEW.task_number := 'TASK-' || year_string || '-' || sequence_number;
  RETURN NEW;
END;
$$;

CREATE TRIGGER trigger_generate_task_number
BEFORE INSERT ON tasks
FOR EACH ROW
WHEN (NEW.task_number IS NULL)
EXECUTE FUNCTION generate_task_number();

-- 2. Calculate Task Score
CREATE OR REPLACE FUNCTION calculate_task_score(
  p_task_id UUID
)
RETURNS DECIMAL
LANGUAGE plpgsql
AS $$
DECLARE
  v_task RECORD;
  v_complexity DECIMAL;
  v_quality DECIMAL;
  v_timeliness DECIMAL;
  v_concern DECIMAL;
  v_concern_count INTEGER;
  v_total_score DECIMAL;
BEGIN
  -- Get task details
  SELECT 
    t.complexity_score,
    t.quality_score,
    t.completed_at,
    t.due_date,
    t.assigned_to_id,
    t.created_at
  INTO v_task
  FROM tasks t
  WHERE t.id = p_task_id;
  
  -- Calculate timeliness score
  IF v_task.completed_at IS NOT NULL AND v_task.due_date IS NOT NULL THEN
    IF v_task.completed_at <= v_task.due_date THEN
      v_timeliness := 100;
    ELSIF v_task.completed_at <= v_task.due_date + INTERVAL '3 days' THEN
      v_timeliness := 70;
    ELSIF v_task.completed_at <= v_task.due_date + INTERVAL '7 days' THEN
      v_timeliness := 40;
    ELSE
      v_timeliness := 10;
    END IF;
  ELSE
    v_timeliness := 0;
  END IF;
  
  -- Count concerns
  SELECT COUNT(*) INTO v_concern_count
  FROM task_comments
  WHERE task_id = p_task_id AND is_concern = true;
  
  -- Calculate concern score (10% deduction per concern, max 50%)
  v_concern := GREATEST(0, 100 - (v_concern_count * 10));
  
  -- Calculate total score
  v_total_score := (v_task.complexity_score * 0.3) + 
                   (v_task.quality_score * 0.4) + 
                   (v_timeliness * 0.2) + 
                   (v_concern * 0.1);
  
  -- Update task
  UPDATE tasks 
  SET 
    timeliness_score = v_timeliness,
    concern_score = v_concern,
    task_score = v_total_score,
    contribution_score = v_total_score * (1 + (LEAST(v_concern_count, 5) * 0.05))
  WHERE id = p_task_id;
  
  -- Update user performance score
  PERFORM update_user_performance(v_task.assigned_to_id, 'quarterly');
  
  RETURN v_total_score;
END;
$$;

-- 3. Update User Performance
CREATE OR REPLACE FUNCTION update_user_performance(
  p_user_id UUID,
  p_period_type VARCHAR(20) DEFAULT 'quarterly'
)
RETURNS VOID
LANGUAGE plpgsql
AS $$
DECLARE
  v_period VARCHAR(20);
  v_completion DECIMAL;
  v_quality DECIMAL;
  v_concern_value DECIMAL;
  v_remark_insight DECIMAL;
  v_collaboration DECIMAL;
  v_leadership DECIMAL;
BEGIN
  -- Determine period
  IF p_period_type = 'quarterly' THEN
    v_period := TO_CHAR(NOW(), 'YYYY') || '-Q' || 
                CASE EXTRACT(QUARTER FROM NOW())
                  WHEN 1 THEN '1'
                  WHEN 2 THEN '2'
                  WHEN 3 THEN '3'
                  WHEN 4 THEN '4'
                END;
  ELSE
    v_period := TO_CHAR(NOW(), 'YYYY-MM');
  END IF;
  
  -- Calculate metrics
  SELECT 
    COALESCE(ROUND(AVG(CASE WHEN status = 'Done / Completed' THEN 100 ELSE 0 END), 2), 0),
    COALESCE(ROUND(AVG(quality_score), 2), 0),
    COALESCE(ROUND(COUNT(CASE WHEN tc.is_concern = true THEN 1 END) * 100.0 / NULLIF(COUNT(tc.id), 0), 2), 0),
    COALESCE(ROUND(AVG(CASE WHEN LENGTH(tc.comment) > 50 THEN 1 ELSE 0 END) * 100, 2), 0),
    COALESCE(ROUND(COUNT(DISTINCT ta.user_id) * 100.0 / NULLIF(COUNT(DISTINCT t.id), 0), 2), 0),
    COALESCE(ROUND(COUNT(DISTINCT t.created_by_id) * 100.0 / NULLIF(COUNT(DISTINCT t.id), 0), 2), 0)
  INTO v_completion, v_quality, v_concern_value, v_remark_insight, v_collaboration, v_leadership
  FROM tasks t
  LEFT JOIN task_comments tc ON tc.task_id = t.id
  LEFT JOIN task_assignments ta ON ta.task_id = t.id
  WHERE (t.assigned_to_id = p_user_id OR t.created_by_id = p_user_id)
    AND t.created_at >= DATE_TRUNC('quarter', NOW() - INTERVAL '3 months');
  
  -- Insert or update performance record
  INSERT INTO user_performance_scores (
    user_id,
    task_completion_score,
    task_quality_score,
    concern_value_score,
    remark_insight_score,
    collaboration_score,
    leadership_score,
    evaluation_period,
    calculated_at
  ) VALUES (
    p_user_id,
    v_completion,
    v_quality,
    v_concern_value,
    v_remark_insight,
    v_collaboration,
    v_leadership,
    v_period,
    NOW()
  ) ON CONFLICT (user_id, evaluation_period) DO UPDATE SET
    task_completion_score = EXCLUDED.task_completion_score,
    task_quality_score = EXCLUDED.task_quality_score,
    concern_value_score = EXCLUDED.concern_value_score,
    remark_insight_score = EXCLUDED.remark_insight_score,
    collaboration_score = EXCLUDED.collaboration_score,
    leadership_score = EXCLUDED.leadership_score,
    calculated_at = EXCLUDED.calculated_at;
END;
$$;

-- 4. Calculate Pending Days
CREATE OR REPLACE FUNCTION update_task_pending_days()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
  -- Update pending days if not completed
  IF NEW.status NOT IN ('Done / Completed', 'Cancelled / On Hold') THEN
    NEW.pending_days := GREATEST(0, EXTRACT(DAY FROM (NEW.due_date - CURRENT_DATE)));
    NEW.is_overdue_days := GREATEST(0, EXTRACT(DAY FROM (CURRENT_DATE - NEW.due_date)));
  ELSE
    NEW.pending_days := 0;
    NEW.is_overdue_days := 0;
  END IF;
  
  -- Update bucket
  NEW.bucket := CASE 
    WHEN NEW.pending_days <= 5 THEN '0 to 5 days'
    WHEN NEW.pending_days <= 10 THEN '5 to 10 days'
    WHEN NEW.pending_days <= 15 THEN '10 to 15 days'
    WHEN NEW.pending_days <= 20 THEN '15 to 20 days'
    WHEN NEW.pending_days <= 25 THEN '20 to 25 days'
    WHEN NEW.pending_days <= 30 THEN '25 to 30 days'
    ELSE 'Above 30 days'
  END;
  
  RETURN NEW;
END;
$$;

CREATE TRIGGER trigger_update_pending_days
BEFORE INSERT OR UPDATE OF due_date, status ON tasks
FOR EACH ROW
EXECUTE FUNCTION update_task_pending_days();

-- 5. Auto-update Completion Percentage
CREATE OR REPLACE FUNCTION auto_update_completion()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
  IF NEW.status = 'Done / Completed' THEN
    NEW.completion_percentage := 100;
    NEW.completed_at := NOW();
    -- Calculate score on completion
    PERFORM calculate_task_score(NEW.id);
  ELSIF NEW.status = 'In Progress' OR NEW.status = 'Work in Progress' THEN
    NEW.completion_percentage := GREATEST(NEW.completion_percentage, 50);
  ELSIF NEW.status = 'Pending' THEN
    NEW.completion_percentage := LEAST(NEW.completion_percentage, 10);
  END IF;
  
  RETURN NEW;
END;
$$;

CREATE TRIGGER trigger_auto_completion
BEFORE UPDATE OF status ON tasks
FOR EACH ROW
WHEN (OLD.status IS DISTINCT FROM NEW.status)
EXECUTE FUNCTION auto_update_completion();
```

---

## 4. API SERVICES

### 4.1 Task Service

```typescript
// services/api/task.service.ts

import { createSupabaseClient } from '@/lib/supabase/client';
import { Task, CreateTaskInput, UpdateTaskInput, TaskFilters, Pagination } from '@/types/task.types';
import { CacheService } from '@/lib/cache/cache.service';
import { NotificationService } from './notification.service';
import { ScoringService } from './scoring.service';

export class TaskService {
  private supabase = createSupabaseClient();
  private cache = new CacheService();
  private notification = new NotificationService();
  private scoring = new ScoringService();

  /**
   * Get tasks with filters and pagination
   */
  async getTasks(filters: TaskFilters, pagination: Pagination): Promise<{
    tasks: Task[];
    total: number;
    page: number;
    limit: number;
  }> {
    const cacheKey = `tasks:${JSON.stringify({ filters, pagination })}`;
    const cached = await this.cache.get(cacheKey);
    
    if (cached) {
      return cached;
    }

    let query = this.supabase
      .from('tasks')
      .select(`
        *,
        assigned_to:assigned_to_id(
          id, first_name, last_name, email, department:department_id(name)
        ),
        created_by:created_by_id(id, first_name, last_name),
        category:category_id(*),
        job_location:job_location_id(*),
        comments:task_comments(count),
        attachments:task_attachments(count)
      `, { count: 'exact' });

    // Apply filters
    if (filters.search) {
      query = query.textSearch('title_description', filters.search, {
        config: 'english'
      });
    }

    if (filters.status && filters.status.length > 0) {
      query = query.in('status', filters.status);
    }

    if (filters.priority && filters.priority.length > 0) {
      query = query.in('priority', filters.priority);
    }

    if (filters.assignedTo) {
      query = query.eq('assigned_to_id', filters.assignedTo);
    }

    if (filters.category) {
      query = query.eq('category_id', filters.category);
    }

    if (filters.department) {
      query = query.eq('departments.id', filters.department);
    }

    if (filters.startDate) {
      query = query.gte('created_at', filters.startDate);
    }

    if (filters.endDate) {
      query = query.lte('created_at', filters.endDate);
    }

    if (filters.dueDateStart) {
      query = query.gte('due_date', filters.dueDateStart);
    }

    if (filters.dueDateEnd) {
      query = query.lte('due_date', filters.dueDateEnd);
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

    if (error) {
      throw new Error(`Failed to fetch tasks: ${error.message}`);
    }

    const result = {
      tasks: data,
      total: count || 0,
      page: pagination.page,
      limit: pagination.limit
    };

    // Cache the result
    await this.cache.set(cacheKey, result, 60); // Cache for 1 minute

    return result;
  }

  /**
   * Get task by ID with all details
   */
  async getTask(taskId: string): Promise<Task> {
    const cacheKey = `task:${taskId}`;
    const cached = await this.cache.get<Task>(cacheKey);
    
    if (cached) {
      return cached;
    }

    const { data, error } = await this.supabase
      .from('tasks')
      .select(`
        *,
        assigned_to:assigned_to_id(
          id, first_name, last_name, email, phone, department:department_id(*)
        ),
        created_by:created_by_id(id, first_name, last_name),
        category:category_id(*),
        job_location:job_location_id(*),
        parent_task:parent_task_id(*),
        dependencies:task_dependencies(
          predecessor_task_id,
          predecessor:predecessor_task_id(*)
        ),
        comments:task_comments(
          *,
          user:user_id(id, first_name, last_name, avatar_url)
        ),
        attachments:task_attachments(*)
      `)
      .eq('id', taskId)
      .single();

    if (error) {
      throw new Error(`Failed to fetch task: ${error.message}`);
    }

    // Cache the result
    await this.cache.set(cacheKey, data, 300); // Cache for 5 minutes

    return data;
  }

  /**
   * Create a new task
   */
  async createTask(input: CreateTaskInput): Promise<Task> {
    // Validate input
    this.validateTaskInput(input);

    // Get current user
    const user = await this.getCurrentUser();

    // Insert task
    const { data: task, error } = await this.supabase
      .from('tasks')
      .insert({
        title: input.title,
        description: input.description,
        category_id: input.categoryId,
        job_location_id: input.jobLocationId,
        assigned_to_id: input.assignedToId || null,
        created_by_id: user.id,
        priority: input.priority || 'Medium',
        due_date: input.dueDate,
        estimated_hours: input.estimatedHours,
        complexity_score: input.complexityScore || 50,
        source_module: input.sourceModule,
        source_record_id: input.sourceRecordId,
        status: 'Pending',
        approval_status: input.approvalRequired ? 'Pending' : 'N/A',
        created_at: new Date().toISOString()
      })
      .select()
      .single();

    if (error) {
      throw new Error(`Failed to create task: ${error.message}`);
    }

    // Handle multiple assignees
    if (input.assignees && input.assignees.length > 0) {
      await this.assignMultipleUsers(task.id, input.assignees);
    }

    // Handle dependencies
    if (input.dependencies && input.dependencies.length > 0) {
      await this.addDependencies(task.id, input.dependencies);
    }

    // Handle attachments
    if (input.attachments && input.attachments.length > 0) {
      await this.linkAttachments(task.id, input.attachments);
    }

    // Send notifications
    await this.notification.taskAssigned(task);

    // Invalidate cache
    await this.cache.deletePattern('tasks:*');

    // Create audit log
    await this.createAuditLog(task.id, 'CREATE', task);

    return task;
  }

  /**
   * Update task
   */
  async updateTask(taskId: string, input: UpdateTaskInput): Promise<Task> {
    // Get current task
    const currentTask = await this.getTask(taskId);

    // Build update object
    const updateData: any = {
      updated_at: new Date().toISOString()
    };

    if (input.title) updateData.title = input.title;
    if (input.description) updateData.description = input.description;
    if (input.categoryId) updateData.category_id = input.categoryId;
    if (input.jobLocationId) updateData.job_location_id = input.jobLocationId;
    if (input.priority) updateData.priority = input.priority;
    if (input.dueDate) updateData.due_date = input.dueDate;
    if (input.estimatedHours) updateData.estimated_hours = input.estimatedHours;
    if (input.complexityScore) updateData.complexity_score = input.complexityScore;

    // Handle status change
    if (input.status && input.status !== currentTask.status) {
      updateData.status = input.status;
      
      // Record status history
      await this.recordStatusHistory(taskId, currentTask.status, input.status);
      
      // If completed, calculate score
      if (input.status === 'Done / Completed') {
        await this.scoring.calculateTaskScore(taskId);
      }
    }

    // Handle assignee change
    if (input.assignedToId && input.assignedToId !== currentTask.assigned_to_id) {
      updateData.assigned_to_id = input.assignedToId;
      
      // Send notification to new assignee
      const newAssignee = await this.getUser(input.assignedToId);
      await this.notification.taskAssigned({
        ...currentTask,
        ...updateData,
        assigned_to: newAssignee
      });
    }

    // Update task
    const { data: task, error } = await this.supabase
      .from('tasks')
      .update(updateData)
      .eq('id', taskId)
      .select()
      .single();

    if (error) {
      throw new Error(`Failed to update task: ${error.message}`);
    }

    // Invalidate cache
    await this.cache.delete(`task:${taskId}`);
    await this.cache.deletePattern('tasks:*');

    // Create audit log
    await this.createAuditLog(taskId, 'UPDATE', { old: currentTask, new: task });

    return task;
  }

  /**
   * Add comment to task
   */
  async addComment(taskId: string, content: string, isConcern: boolean = false): Promise<any> {
    const user = await this.getCurrentUser();

    const { data: comment, error } = await this.supabase
      .from('task_comments')
      .insert({
        task_id: taskId,
        user_id: user.id,
        comment: content,
        is_concern: isConcern,
        comment_type: isConcern ? 'Concern' : 'Regular',
        created_at: new Date().toISOString()
      })
      .select()
      .single();

    if (error) {
      throw new Error(`Failed to add comment: ${error.message}`);
    }

    // If concern, update scoring
    if (isConcern) {
      await this.scoring.handleConcern(comment);
    }

    // Send notifications to assignee and creator
    const task = await this.getTask(taskId);
    await this.notification.newComment(task, comment);

    // Invalidate cache
    await this.cache.delete(`task:${taskId}`);

    return comment;
  }

  /**
   * Upload attachment to task
   */
  async uploadAttachment(taskId: string, file: File): Promise<any> {
    const user = await this.getCurrentUser();
    
    // Upload to storage
    const fileExt = file.name.split('.').pop();
    const fileName = `${uuidv4()}.${fileExt}`;
    const filePath = `tasks/${taskId}/${fileName}`;

    const { data: storageData, error: uploadError } = await this.supabase.storage
      .from('task-attachments')
      .upload(filePath, file, {
        cacheControl: '3600',
        upsert: false
      });

    if (uploadError) {
      throw new Error(`Failed to upload file: ${uploadError.message}`);
    }

    // Get public URL
    const { data: urlData } = this.supabase.storage
      .from('task-attachments')
      .getPublicUrl(filePath);

    // Create attachment record
    const { data: attachment, error } = await this.supabase
      .from('task_attachments')
      .insert({
        task_id: taskId,
        uploaded_by_id: user.id,
        file_name: file.name,
        file_url: urlData.publicUrl,
        file_type: file.type,
        file_size: file.size,
        mime_type: file.type,
        uploaded_at: new Date().toISOString()
      })
      .select()
      .single();

    if (error) {
      // Clean up storage if DB insert fails
      await this.supabase.storage
        .from('task-attachments')
        .remove([filePath]);
      
      throw new Error(`Failed to create attachment record: ${error.message}`);
    }

    // Invalidate cache
    await this.cache.delete(`task:${taskId}`);

    return attachment;
  }

  /**
   * Get task analytics
   */
  async getTaskAnalytics(period: string = 'monthly'): Promise<any> {
    const cacheKey = `analytics:tasks:${period}`;
    const cached = await this.cache.get(cacheKey);
    
    if (cached) {
      return cached;
    }

    let query = this.supabase
      .from('task_analytics_summary')
      .select('*')
      .order('month', { ascending: false });

    if (period === 'monthly') {
      query = query.limit(12);
    } else if (period === 'quarterly') {
      query = query.limit(4);
    }

    const { data, error } = await query;

    if (error) {
      throw new Error(`Failed to fetch analytics: ${error.message}`);
    }

    // Calculate additional metrics
    const analytics = {
      summary: data,
      metrics: await this.calculateMetrics(data)
    };

    // Cache for 1 hour
    await this.cache.set(cacheKey, analytics, 3600);

    return analytics;
  }

  /**
   * Get user performance scores
   */
  async getUserPerformance(userId: string): Promise<any> {
    const cacheKey = `performance:${userId}`;
    const cached = await this.cache.get(cacheKey);
    
    if (cached) {
      return cached;
    }

    const { data, error } = await this.supabase
      .from('user_performance_scores')
      .select('*')
      .eq('user_id', userId)
      .order('evaluation_period', { ascending: false })
      .limit(1)
      .single();

    if (error && error.code !== 'PGRST116') { // PGRST116 = Not found
      throw new Error(`Failed to fetch performance: ${error.message}`);
    }

    const result = data || {
      user_id: userId,
      overall_score: 0,
      performance_band: 'Not Evaluated'
    };

    // Cache for 1 hour
    await this.cache.set(cacheKey, result, 3600);

    return result;
  }

  // ============================================
  // PRIVATE HELPER METHODS
  // ============================================

  private validateTaskInput(input: CreateTaskInput): void {
    if (!input.title || input.title.trim().length === 0) {
      throw new Error('Task title is required');
    }

    if (!input.dueDate) {
      throw new Error('Task due date is required');
    }

    const dueDate = new Date(input.dueDate);
    if (dueDate < new Date()) {
      throw new Error('Due date cannot be in the past');
    }
  }

  private async assignMultipleUsers(taskId: string, userIds: string[]): Promise<void> {
    const user = await this.getCurrentUser();
    
    const assignments = userIds.map(userId => ({
      task_id: taskId,
      user_id: userId,
      assigned_by: user.id,
      assigned_at: new Date().toISOString()
    }));

    const { error } = await this.supabase
      .from('task_assignments')
      .insert(assignments);

    if (error) {
      throw new Error(`Failed to assign users: ${error.message}`);
    }
  }

  private async addDependencies(taskId: string, dependencies: any[]): Promise<void> {
    const deps = dependencies.map(dep => ({
      task_id: taskId,
      predecessor_task_id: dep.taskId,
      dependency_type: dep.type || 'Finish-to-Start',
      lag_days: dep.lagDays || 0
    }));

    const { error } = await this.supabase
      .from('task_dependencies')
      .insert(deps);

    if (error) {
      throw new Error(`Failed to add dependencies: ${error.message}`);
    }
  }

  private async linkAttachments(taskId: string, attachmentIds: string[]): Promise<void> {
    const { error } = await this.supabase
      .from('task_attachments')
      .update({ task_id: taskId })
      .in('id', attachmentIds)
      .eq('task_id', null);

    if (error) {
      throw new Error(`Failed to link attachments: ${error.message}`);
    }
  }

  private async recordStatusHistory(
    taskId: string,
    previousStatus: string,
    newStatus: string
  ): Promise<void> {
    const user = await this.getCurrentUser();

    const { error } = await this.supabase
      .from('task_status_history')
      .insert({
        task_id: taskId,
        previous_status: previousStatus,
        new_status: newStatus,
        changed_by_id: user.id,
        changed_at: new Date().toISOString()
      });

    if (error) {
      console.error('Failed to record status history:', error);
    }
  }

  private async createAuditLog(
    taskId: string,
    action: string,
    data: any
  ): Promise<void> {
    const user = await this.getCurrentUser();

    const { error } = await this.supabase
      .from('audit_logs')
      .insert({
        user_id: user.id,
        action: action,
        resource_type: 'task',
        resource_id: taskId,
        change_data: data,
        timestamp: new Date().toISOString()
      });

    if (error) {
      console.error('Failed to create audit log:', error);
    }
  }

  private async getCurrentUser(): Promise<any> {
    const { data: { user }, error } = await this.supabase.auth.getUser();
    
    if (error || !user) {
      throw new Error('User not authenticated');
    }

    return user;
  }

  private async getUser(userId: string): Promise<any> {
    const { data, error } = await this.supabase
      .from('users')
      .select('*')
      .eq('id', userId)
      .single();

    if (error) {
      throw new Error(`Failed to get user: ${error.message}`);
    }

    return data;
  }

  private async calculateMetrics(data: any[]): Promise<any> {
    if (!data || data.length === 0) {
      return {
        avgCompletionRate: 0,
        avgScore: 0,
        totalTasks: 0,
        overdueRate: 0
      };
    }

    const totalTasks = data.reduce((sum, d) => sum + d.total_tasks, 0);
    const totalCompleted = data.reduce((sum, d) => sum + d.completed_tasks, 0);
    const totalOverdue = data.reduce((sum, d) => sum + d.overdue_tasks, 0);
    const avgScore = data.reduce((sum, d) => sum + d.avg_score, 0) / data.length;

    return {
      avgCompletionRate: totalTasks > 0 ? (totalCompleted / totalTasks) * 100 : 0,
      avgScore: avgScore || 0,
      totalTasks: totalTasks,
      overdueRate: totalTasks > 0 ? (totalOverdue / totalTasks) * 100 : 0
    };
  }
}
```

### 4.2 Request Service

```typescript
// services/api/request.service.ts

import { createSupabaseClient } from '@/lib/supabase/client';
import { Request, CreateRequestInput, ApprovalInput } from '@/types/request.types';
import { TaskService } from './task.service';
import { NotificationService } from './notification.service';

export class RequestService {
  private supabase = createSupabaseClient();
  private taskService = new TaskService();
  private notification = new NotificationService();

  /**
   * Create a new request
   */
  async createRequest(input: CreateRequestInput): Promise<Request> {
    // Validate input
    this.validateRequestInput(input);

    const user = await this.getCurrentUser();

    // First, create a task for the request
    const task = await this.taskService.createTask({
      title: input.title,
      description: input.description,
      categoryId: input.categoryId || 'request',
      priority: input.priority || 'Medium',
      dueDate: input.dueDate || this.calculateDueDate(input.priority),
      assignedToId: input.assignedToId,
      sourceModule: 'request',
      approvalRequired: true
    });

    // Create request
    const { data: request, error } = await this.supabase
      .from('requests')
      .insert({
        request_number: await this.generateRequestNumber(),
        task_id: task.id,
        request_type_id: input.typeId,
        request_data: input.requestData,
        department_id: input.departmentId,
        requested_by_id: user.id,
        status: 'Submitted',
        submitted_at: new Date().toISOString()
      })
      .select()
      .single();

    if (error) {
      // Clean up task if request fails
      await this.taskService.deleteTask(task.id);
      throw new Error(`Failed to create request: ${error.message}`);
    }

    // Start approval workflow
    await this.startApprovalWorkflow(request.id);

    // Send notifications
    await this.notification.requestSubmitted(request);

    return request;
  }

  /**
   * Start approval workflow for request
   */
  async startApprovalWorkflow(requestId: string): Promise<void> {
    // Get request details
    const { data: request } = await this.supabase
      .from('requests')
      .select(`
        *,
        request_type:request_type_id(
          *,
          approval_workflow:approval_workflow_id(*)
        )
      `)
      .eq('id', requestId)
      .single();

    if (!request) {
      throw new Error('Request not found');
    }

    // Get workflow steps
    const { data: steps } = await this.supabase
      .from('approval_workflow_steps')
      .select(`
        *,
        approver:approver_user_id(id, first_name, last_name, email),
        role:approver_role_id(id, name, slug)
      `)
      .eq('workflow_id', request.request_type.approval_workflow_id)
      .order('step_order', { ascending: true });

    if (!steps || steps.length === 0) {
      // No approval required
      await this.approveRequestDirect(requestId);
      return;
    }

    // Create approvals for each step
    for (const step of steps) {
      let approverId = step.approver_user_id;
      
      // If approver is role-based, find user with that role in the department
      if (!approverId && step.role) {
        approverId = await this.findApproverByRole(step.role.slug, request.department_id);
      }

      if (!approverId) {
        throw new Error(`No approver found for step ${step.step_order}`);
      }

      const { error } = await this.supabase
        .from('approvals')
        .insert({
          request_id: requestId,
          workflow_step_id: step.id,
          approver_id: approverId,
          status: 'Pending'
        });

      if (error) {
        throw new Error(`Failed to create approval: ${error.message}`);
      }

      // Send notification to approver
      await this.notification.approvalRequired(requestId, approverId);
    }
  }

  /**
   * Approve a request
   */
  async approveRequest(approvalId: string, comments?: string): Promise<void> {
    const user = await this.getCurrentUser();

    // Update approval
    const { data: approval, error } = await this.supabase
      .from('approvals')
      .update({
        status: 'Approved',
        comments: comments,
        approved_at: new Date().toISOString()
      })
      .eq('id', approvalId)
      .select()
      .single();

    if (error) {
      throw new Error(`Failed to approve request: ${error.message}`);
    }

    // Check if all approvals are complete
    const { data: allApprovals } = await this.supabase
      .from('approvals')
      .select('status')
      .eq('request_id', approval.request_id);

    const allApproved = allApprovals.every(a => a.status === 'Approved');
    const anyRejected = allApprovals.some(a => a.status === 'Rejected');

    if (anyRejected) {
      await this.rejectRequest(approval.request_id, 'One or more approvals were rejected');
      return;
    }

    if (allApproved) {
      await this.completeRequest(approval.request_id);
    }
  }

  /**
   * Reject a request
   */
  async rejectRequest(requestId: string, reason: string): Promise<void> {
    const user = await this.getCurrentUser();

    // Update request
    const { error } = await this.supabase
      .from('requests')
      .update({
        status: 'Rejected',
        rejection_reason: reason,
        approved_by_id: user.id,
        updated_at: new Date().toISOString()
      })
      .eq('id', requestId);

    if (error) {
      throw new Error(`Failed to reject request: ${error.message}`);
    }

    // Update linked task
    await this.supabase
      .from('tasks')
      .update({
        status: 'Cancelled / On Hold',
        approval_status: 'Rejected'
      })
      .eq('source_module', 'request')
      .eq('source_record_id', requestId);

    // Notify requester
    await this.notification.requestRejected(requestId, reason);
  }

  /**
   * Complete request (all approvals done)
   */
  async completeRequest(requestId: string): Promise<void> {
    const user = await this.getCurrentUser();

    // Get request details
    const { data: request } = await this.supabase
      .from('requests')
      .select('*')
      .eq('id', requestId)
      .single();

    if (!request) {
      throw new Error('Request not found');
    }

    // Update request
    await this.supabase
      .from('requests')
      .update({
        status: 'Approved',
        approved_by_id: user.id,
        approval_date: new Date().toISOString(),
        updated_at: new Date().toISOString()
      })
      .eq('id', requestId);

    // Update linked task
    await this.supabase
      .from('tasks')
      .update({
        status: 'Done / Completed',
        approval_status: 'Approved',
        approved_by_id: user.id,
        completed_at: new Date().toISOString()
      })
      .eq('source_module', 'request')
      .eq('source_record_id', requestId);

    // Execute request-specific actions based on type
    await this.executeRequestActions(request);

    // Notify requester
    await this.notification.requestApproved(requestId);
  }

  /**
   * Execute actions based on request type
   */
  private async executeRequestActions(request: any): Promise<void> {
    const type = request.request_type_id;
    const data = request.request_data;

    switch(type) {
      case 'new_hire':
        await this.createEmployeeFromRequest(request);
        break;
      case 'equipment':
        await this.createEquipmentRequest(request);
        break;
      case 'budget':
        await this.createBudgetRequest(request);
        break;
      default:
        console.log(`No specific actions for request type: ${type}`);
    }
  }

  /**
   * Create employee from new hire request
   */
  private async createEmployeeFromRequest(request: any): Promise<void> {
    const data = request.request_data;
    
    // Create employee record
    const { error } = await this.supabase
      .from('employees')
      .insert({
        user_id: data.userId || null,
        employee_code: await this.generateEmployeeCode(),
        first_name: data.firstName,
        last_name: data.lastName,
        date_of_birth: data.dateOfBirth,
        gender: data.gender,
        cnic: data.cnic,
        joining_date: data.joiningDate || new Date().toISOString(),
        employment_type: data.employmentType || 'Permanent',
        department_id: data.departmentId,
        designation: data.designation,
        basic_salary: data.salary,
        employment_status: 'Active'
      });

    if (error) {
      console.error('Failed to create employee:', error);
    }
  }

  // ============================================
  // PRIVATE HELPER METHODS
  // ============================================

  private validateRequestInput(input: CreateRequestInput): void {
    if (!input.title || input.title.trim().length === 0) {
      throw new Error('Request title is required');
    }

    if (!input.typeId) {
      throw new Error('Request type is required');
    }

    if (!input.requestData || Object.keys(input.requestData).length === 0) {
      throw new Error('Request data is required');
    }
  }

  private async generateRequestNumber(): Promise<string> {
    const year = new Date().getFullYear();
    const { count } = await this.supabase
      .from('requests')
      .select('*', { count: 'exact', head: true })
      .gte('created_at', `${year}-01-01`)
      .lte('created_at', `${year}-12-31`);

    return `REQ-${year}-${String((count || 0) + 1).padStart(6, '0')}`;
  }

  private async generateEmployeeCode(): Promise<string> {
    const { count } = await this.supabase
      .from('employees')
      .select('*', { count: 'exact', head: true });

    return `EMP-${String((count || 0) + 1).padStart(6, '0')}`;
  }

  private calculateDueDate(priority: string): string {
    const date = new Date();
    const days = {
      'Low': 14,
      'Medium': 7,
      'High': 3,
      'Urgent/Overdue': 1
    };
    date.setDate(date.getDate() + (days[priority] || 7));
    return date.toISOString().split('T')[0];
  }

  private async findApproverByRole(roleSlug: string, departmentId: string): Promise<string> {
    const { data } = await this.supabase
      .from('users')
      .select('id')
      .eq('department_id', departmentId)
      .eq('is_active', true)
      .order('created_at', { ascending: true })
      .limit(1);

    if (data && data.length > 0) {
      return data[0].id;
    }

    // Fallback to any user with the role
    const { data: fallback } = await this.supabase
      .from('users')
      .select('id')
      .eq('is_active', true)
      .order('created_at', { ascending: true })
      .limit(1);

    return fallback?.[0]?.id || null;
  }

  private async approveRequestDirect(requestId: string): Promise<void> {
    const user = await this.getCurrentUser();
    
    await this.supabase
      .from('requests')
      .update({
        status: 'Approved',
        approved_by_id: user.id,
        approval_date: new Date().toISOString()
      })
      .eq('id', requestId);

    await this.supabase
      .from('tasks')
      .update({
        status: 'Done / Completed',
        approval_status: 'Approved'
      })
      .eq('source_module', 'request')
      .eq('source_record_id', requestId);
  }

  private async getCurrentUser(): Promise<any> {
    const { data: { user }, error } = await this.supabase.auth.getUser();
    
    if (error || !user) {
      throw new Error('User not authenticated');
    }

    return user;
  }
}
```

### 4.3 Document Service

```typescript
// services/api/document.service.ts

import { createSupabaseClient } from '@/lib/supabase/client';
import { Document, DocumentMetadata } from '@/types/document.types';

export class DocumentService {
  private supabase = createSupabaseClient();

  /**
   * Upload document
   */
  async uploadDocument(
    file: File,
    metadata: DocumentMetadata
  ): Promise<Document> {
    const user = await this.getCurrentUser();

    // Generate unique filename
    const fileExt = file.name.split('.').pop();
    const fileName = `${uuidv4()}.${fileExt}`;
    const filePath = `${metadata.sourceModule}/${metadata.sourceRecordId}/${fileName}`;

    // Upload to storage
    const { data: storageData, error: uploadError } = await this.supabase.storage
      .from('documents')
      .upload(filePath, file, {
        cacheControl: '3600',
        upsert: false
      });

    if (uploadError) {
      throw new Error(`Failed to upload file: ${uploadError.message}`);
    }

    // Get public URL
    const { data: urlData } = this.supabase.storage
      .from('documents')
      .getPublicUrl(filePath);

    // Calculate file hash
    const fileHash = await this.calculateFileHash(file);

    // Create document record
    const { data: document, error } = await this.supabase
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
        file_hash: fileHash,
        uploaded_by_id: user.id,
        uploaded_at: new Date().toISOString(),
        status: metadata.status || 'Draft',
        tags: metadata.tags || []
      })
      .select()
      .single();

    if (error) {
      // Clean up storage if DB insert fails
      await this.supabase.storage
        .from('documents')
        .remove([filePath]);
      
      throw new Error(`Failed to create document record: ${error.message}`);
    }

    return document;
  }

  /**
   * Get documents by source
   */
  async getDocumentsBySource(
    sourceModule: string,
    sourceRecordId: string
  ): Promise<Document[]> {
    const { data, error } = await this.supabase
      .from('documents')
      .select(`
        *,
        uploaded_by:uploaded_by_id(id, first_name, last_name),
        document_type:document_type_id(*)
      `)
      .eq('source_module', sourceModule)
      .eq('source_record_id', sourceRecordId)
      .eq('is_latest_version', true)
      .order('uploaded_at', { ascending: false });

    if (error) {
      throw new Error(`Failed to fetch documents: ${error.message}`);
    }

    return data;
  }

  /**
   * Get document with version history
   */
  async getDocumentWithHistory(documentId: string): Promise<any> {
    const { data: document, error } = await this.supabase
      .from('documents')
      .select(`
        *,
        uploaded_by:uploaded_by_id(*),
        document_type:document_type_id(*),
        versions:document_versions(*)
      `)
      .eq('id', documentId)
      .single();

    if (error) {
      throw new Error(`Failed to fetch document: ${error.message}`);
    }

    return document;
  }

  /**
   * Create new version of document
   */
  async createNewVersion(
    documentId: string,
    file: File,
    changeNotes: string
  ): Promise<Document> {
    // Get current document
    const { data: currentDoc } = await this.supabase
      .from('documents')
      .select('*')
      .eq('id', documentId)
      .single();

    if (!currentDoc) {
      throw new Error('Document not found');
    }

    // Upload new file
    const fileExt = file.name.split('.').pop();
    const fileName = `${uuidv4()}.${fileExt}`;
    const filePath = `${currentDoc.source_module}/${currentDoc.source_record_id}/${fileName}`;

    const { error: uploadError } = await this.supabase.storage
      .from('documents')
      .upload(filePath, file, {
        cacheControl: '3600',
        upsert: false
      });

    if (uploadError) {
      throw new Error(`Failed to upload file: ${uploadError.message}`);
    }

    const { data: urlData } = this.supabase.storage
      .from('documents')
      .getPublicUrl(filePath);

    // Create version record
    const user = await this.getCurrentUser();
    
    const { data: version, error } = await this.supabase
      .from('document_versions')
      .insert({
        document_id: documentId,
        version_number: currentDoc.version + 1,
        file_url: urlData.publicUrl,
        file_name: file.name,
        file_size: file.size,
        uploaded_by_id: user.id,
        uploaded_at: new Date().toISOString(),
        change_notes: changeNotes
      })
      .select()
      .single();

    if (error) {
      throw new Error(`Failed to create version: ${error.message}`);
    }

    // Update document with new version info
    const { data: updatedDoc } = await this.supabase
      .from('documents')
      .update({
        version: currentDoc.version + 1,
        file_url: urlData.publicUrl,
        file_name: file.name,
        file_size: file.size,
        updated_at: new Date().toISOString()
      })
      .eq('id', documentId)
      .select()
      .single();

    return updatedDoc;
  }

  /**
   * Delete document
   */
  async deleteDocument(documentId: string): Promise<void> {
    // Get document
    const { data: document } = await this.supabase
      .from('documents')
      .select('file_url, source_module, source_record_id')
      .eq('id', documentId)
      .single();

    if (!document) {
      throw new Error('Document not found');
    }

    // Delete from storage
    const filePath = document.file_url.split('/').slice(-3).join('/');
    await this.supabase.storage
      .from('documents')
      .remove([filePath]);

    // Delete from database
    const { error } = await this.supabase
      .from('documents')
      .delete()
      .eq('id', documentId);

    if (error) {
      throw new Error(`Failed to delete document: ${error.message}`);
    }
  }

  /**
   * Generate document preview URL (with expiration)
   */
  async getDocumentPreview(documentId: string): Promise<string> {
    const { data: document } = await this.supabase
      .from('documents')
      .select('file_url, file_type')
      .eq('id', documentId)
      .single();

    if (!document) {
      throw new Error('Document not found');
    }

    // For images and PDFs, return direct URL with expiration
    if (document.file_type?.startsWith('image/') || document.file_type === 'application/pdf') {
      const { data } = await this.supabase.storage
        .from('documents')
        .createSignedUrl(
          document.file_url.split('/').slice(-3).join('/'),
          300 // 5 minutes expiration
        );
      
      return data.signedUrl;
    }

    // For other files, return download URL
    return document.file_url;
  }

  /**
   * Search documents
   */
  async searchDocuments(query: string, filters?: any): Promise<Document[]> {
    let searchQuery = this.supabase
      .from('documents')
      .select(`
        *,
        uploaded_by:uploaded_by_id(id, first_name, last_name),
        document_type:document_type_id(*)
      `)
      .textSearch('document_name', query, {
        config: 'english'
      });

    if (filters?.documentType) {
      searchQuery = searchQuery.eq('document_type_id', filters.documentType);
    }

    if (filters?.sourceModule) {
      searchQuery = searchQuery.eq('source_module', filters.sourceModule);
    }

    if (filters?.status) {
      searchQuery = searchQuery.eq('status', filters.status);
    }

    const { data, error } = await searchQuery
      .order('uploaded_at', { ascending: false })
      .limit(filters?.limit || 50);

    if (error) {
      throw new Error(`Failed to search documents: ${error.message}`);
    }

    return data;
  }

  // ============================================
  // PRIVATE HELPER METHODS
  // ============================================

  private async generateDocumentNumber(): Promise<string> {
    const year = new Date().getFullYear();
    const { count } = await this.supabase
      .from('documents')
      .select('*', { count: 'exact', head: true })
      .gte('uploaded_at', `${year}-01-01`)
      .lte('uploaded_at', `${year}-12-31`);

    return `DOC-${year}-${String((count || 0) + 1).padStart(6, '0')}`;
  }

  private async calculateFileHash(file: File): Promise<string> {
    const buffer = await file.arrayBuffer();
    const hashBuffer = await crypto.subtle.digest('SHA-256', buffer);
    const hashArray = Array.from(new Uint8Array(hashBuffer));
    return hashArray.map(b => b.toString(16).padStart(2, '0')).join('');
  }

  private async getCurrentUser(): Promise<any> {
    const { data: { user }, error } = await this.supabase.auth.getUser();
    
    if (error || !user) {
      throw new Error('User not authenticated');
    }

    return user;
  }
}
```

---

## 5. FRONTEND COMPONENTS

### 5.1 Task Board Component

```typescript
// components/tasks/TaskBoard/TaskBoardView.tsx

'use client';

import { useState, useEffect } from 'react';
import { DragDropContext, Droppable, Draggable, DropResult } from 'react-beautiful-dnd';
import { useTaskStore } from '@/stores/task.store';
import { TaskColumn } from './TaskColumn';
import { TaskCard } from './TaskCard';
import { TaskFilters } from '../TaskFilters';
import { Button } from '@/components/ui/Button';
import { PlusIcon } from '@heroicons/react/24/outline';
import { Task } from '@/types/task.types';

interface TaskBoardViewProps {
  tasks: Task[];
  isLoading: boolean;
  onTaskUpdate: (taskId: string, updates: any) => void;
  onCreateTask: () => void;
}

export const TaskBoardView = ({
  tasks,
  isLoading,
  onTaskUpdate,
  onCreateTask
}: TaskBoardViewProps) => {
  const [columns, setColumns] = useState<Record<string, Task[]>>({});
  
  const statusColumns = [
    { id: 'pending', label: 'Pending', color: 'gray' },
    { id: 'in-progress', label: 'In Progress', color: 'blue' },
    { id: 'review', label: 'Review', color: 'yellow' },
    { id: 'completed', label: 'Completed', color: 'green' },
    { id: 'on-hold', label: 'On Hold', color: 'red' }
  ];

  useEffect(() => {
    // Group tasks by status
    const grouped = tasks.reduce((acc, task) => {
      const status = task.status.toLowerCase().replace(/\s+/g, '-');
      if (!acc[status]) {
        acc[status] = [];
      }
      acc[status].push(task);
      return acc;
    }, {} as Record<string, Task[]>);

    setColumns(grouped);
  }, [tasks]);

  const handleDragEnd = (result: DropResult) => {
    const { destination, source, draggableId } = result;

    // Dropped outside the list
    if (!destination) {
      return;
    }

    // Dropped in the same position
    if (
      destination.droppableId === source.droppableId &&
      destination.index === source.index
    ) {
      return;
    }

    // Find the task and update its status
    const task = tasks.find(t => t.id === draggableId);
    if (!task) return;

    const newStatus = destination.droppableId
      .split('-')
      .map(word => word.charAt(0).toUpperCase() + word.slice(1))
      .join(' ');

    // Optimistically update UI
    const newColumns = { ...columns };
    
    // Remove from source column
    const sourceColumn = [...(newColumns[source.droppableId] || [])];
    const [removed] = sourceColumn.splice(source.index, 1);
    newColumns[source.droppableId] = sourceColumn;

    // Add to destination column
    const destColumn = [...(newColumns[destination.droppableId] || [])];
    destColumn.splice(destination.index, 0, { ...removed, status: newStatus });
    newColumns[destination.droppableId] = destColumn;

    setColumns(newColumns);

    // Update in backend
    onTaskUpdate(draggableId, { status: newStatus });
  };

  if (isLoading) {
    return <TaskBoardSkeleton />;
  }

  return (
    <div className="flex flex-col h-full">
      {/* Header */}
      <div className="flex items-center justify-between mb-4 px-4">
        <h2 className="text-lg font-semibold text-gray-900">Task Board</h2>
        <Button
          variant="primary"
          size="sm"
          onClick={onCreateTask}
          className="flex items-center gap-2"
        >
          <PlusIcon className="w-4 h-4" />
          Create Task
        </Button>
      </div>

      {/* Filters */}
      <div className="px-4 mb-4">
        <TaskFilters />
      </div>

      {/* Board */}
      <DragDropContext onDragEnd={handleDragEnd}>
        <div className="flex gap-4 overflow-x-auto p-4 flex-1">
          {statusColumns.map((column) => (
            <div
              key={column.id}
              className="w-80 shrink-0"
            >
              <div className="flex items-center justify-between mb-3">
                <h3 className={`font-medium text-gray-700 flex items-center gap-2`}>
                  <span className={`w-2 h-2 rounded-full bg-${column.color}-500`} />
                  {column.label}
                  <span className="text-sm text-gray-500 bg-gray-100 px-2 py-0.5 rounded-full">
                    {columns[column.id]?.length || 0}
                  </span>
                </h3>
              </div>

              <Droppable droppableId={column.id}>
                {(provided, snapshot) => (
                  <div
                    ref={provided.innerRef}
                    {...provided.droppableProps}
                    className={`
                      space-y-2 min-h-[200px] p-2 rounded-lg
                      ${snapshot.isDraggingOver ? 'bg-blue-50' : 'bg-gray-50'}
                      transition-colors duration-200
                    `}
                  >
                    {columns[column.id]?.map((task, index) => (
                      <Draggable
                        key={task.id}
                        draggableId={task.id}
                        index={index}
                      >
                        {(provided, snapshot) => (
                          <div
                            ref={provided.innerRef}
                            {...provided.draggableProps}
                            {...provided.dragHandleProps}
                            className={`
                              transition-transform duration-150
                              ${snapshot.isDragging ? 'rotate-1 scale-105' : ''}
                            `}
                          >
                            <TaskCard task={task} />
                          </div>
                        )}
                      </Draggable>
                    ))}
                    {provided.placeholder}
                    
                    {/* Empty state */}
                    {(!columns[column.id] || columns[column.id].length === 0) && (
                      <div className="text-center py-8 text-gray-400 text-sm">
                        No tasks in {column.label}
                      </div>
                    )}
                  </div>
                )}
              </Droppable>
            </div>
          ))}
        </div>
      </DragDropContext>
    </div>
  );
};

// Task Board Skeleton Loader
const TaskBoardSkeleton = () => {
  return (
    <div className="flex gap-4 overflow-x-auto p-4">
      {[1, 2, 3, 4, 5].map((i) => (
        <div key={i} className="w-80 shrink-0">
          <div className="flex items-center justify-between mb-3">
            <div className="h-6 w-32 bg-gray-200 rounded animate-pulse" />
            <div className="h-6 w-8 bg-gray-200 rounded animate-pulse" />
          </div>
          <div className="space-y-2">
            {[1, 2, 3].map((j) => (
              <div
                key={j}
                className="h-24 bg-gray-200 rounded-lg animate-pulse"
              />
            ))}
          </div>
        </div>
      ))}
    </div>
  );
};
```

### 5.2 Task Card Component

```typescript
// components/tasks/TaskBoard/TaskCard.tsx

'use client';

import { useState } from 'react';
import Link from 'next/link';
import { Task } from '@/types/task.types';
import { Avatar } from '@/components/ui/Avatar';
import { Badge } from '@/components/ui/Badge';
import { Progress } from '@/components/ui/Progress';
import { formatDate, formatRelative } from '@/lib/utils/date';
import {
  CalendarIcon,
  ClockIcon,
  ChatBubbleLeftIcon,
  PaperClipIcon,
  UserIcon
} from '@heroicons/react/24/outline';

interface TaskCardProps {
  task: Task;
  compact?: boolean;
}

export const TaskCard = ({ task, compact = false }: TaskCardProps) => {
  const [isHovered, setIsHovered] = useState(false);

  const getPriorityConfig = (priority: string) => {
    const configs = {
      'Low': { color: 'blue', icon: '⬇️' },
      'Medium': { color: 'yellow', icon: '➡️' },
      'High': { color: 'orange', icon: '⬆️' },
      'Urgent/Overdue': { color: 'red', icon: '🔴' }
    };
    return configs[priority] || configs['Medium'];
  };

  const getStatusConfig = (status: string) => {
    const configs = {
      'Pending': { color: 'gray', variant: 'outline' },
      'In Progress': { color: 'blue', variant: 'solid' },
      'Work in Progress': { color: 'blue', variant: 'solid' },
      'Done / Completed': { color: 'green', variant: 'solid' },
      'Cancelled / On Hold': { color: 'red', variant: 'solid' },
      'Not Started': { color: 'gray', variant: 'outline' },
      'Urgent / Overdue': { color: 'red', variant: 'solid' }
    };
    return configs[status] || configs['Pending'];
  };

  const priorityConfig = getPriorityConfig(task.priority);
  const statusConfig = getStatusConfig(task.status);

  if (compact) {
    return (
      <Link href={`/tasks/${task.id}`}>
        <div
          className={`
            p-3 bg-white rounded-lg border border-gray-200 hover:shadow-md 
            transition-all duration-200 cursor-pointer
            ${isHovered ? 'shadow-md border-primary-200' : ''}
          `}
          onMouseEnter={() => setIsHovered(true)}
          onMouseLeave={() => setIsHovered(false)}
        >
          <div className="flex items-start justify-between">
            <div className="flex-1 min-w-0">
              <h4 className="text-sm font-medium text-gray-900 truncate">
                {task.title}
              </h4>
              {task.description && (
                <p className="text-xs text-gray-500 truncate mt-0.5">
                  {task.description}
                </p>
              )}
            </div>
            <Badge
              variant={statusConfig.variant as 'solid' | 'outline'}
              color={statusConfig.color}
              size="sm"
              className="ml-2 shrink-0"
            >
              {task.status}
            </Badge>
          </div>

          <div className="flex items-center justify-between mt-2 text-xs text-gray-500">
            <div className="flex items-center gap-3">
              {task.assigned_to && (
                <div className="flex items-center gap-1">
                  <Avatar
                    user={task.assigned_to}
                    size="xs"
                    fallback={task.assigned_to.first_name?.[0] || 'U'}
                  />
                  <span className="truncate max-w-[80px]">
                    {task.assigned_to.first_name} {task.assigned_to.last_name?.[0]}.
                  </span>
                </div>
              )}
              <div className="flex items-center gap-1">
                <CalendarIcon className="w-3 h-3" />
                <span>{formatDate(task.due_date)}</span>
              </div>
            </div>
            <div className="flex items-center gap-2">
              <span className={`text-${priorityConfig.color}-500`}>
                {priorityConfig.icon}
              </span>
              <span className="text-${priorityConfig.color}-600 font-medium">
                {task.priority}
              </span>
            </div>
          </div>
        </div>
      </Link>
    );
  }

  return (
    <Link href={`/tasks/${task.id}`}>
      <div
        className={`
          p-4 bg-white rounded-lg border border-gray-200 hover:shadow-md 
          transition-all duration-200 cursor-pointer
          ${isHovered ? 'shadow-md border-primary-200' : ''}
        `}
        onMouseEnter={() => setIsHovered(true)}
        onMouseLeave={() => setIsHovered(false)}
      >
        {/* Header */}
        <div className="flex items-start justify-between">
          <div className="flex-1 min-w-0">
            <div className="flex items-center gap-2">
              <span className="text-xs text-gray-400 font-mono">
                {task.task_number}
              </span>
              <Badge
                variant={statusConfig.variant as 'solid' | 'outline'}
                color={statusConfig.color}
                size="sm"
              >
                {task.status}
              </Badge>
              {task.completion_percentage > 0 && task.completion_percentage < 100 && (
                <span className="text-xs text-gray-500">
                  {task.completion_percentage}%
                </span>
              )}
            </div>
            <h4 className="text-sm font-medium text-gray-900 mt-1 line-clamp-2">
              {task.title}
            </h4>
            {task.description && (
              <p className="text-sm text-gray-500 mt-1 line-clamp-2">
                {task.description}
              </p>
            )}
          </div>
          <div className="flex items-center gap-2 ml-2 shrink-0">
            <span className={`text-${priorityConfig.color}-500`}>
              {priorityConfig.icon}
            </span>
            <span className={`text-xs font-medium text-${priorityConfig.color}-600`}>
              {task.priority}
            </span>
          </div>
        </div>

        {/* Progress */}
        {task.completion_percentage > 0 && (
          <div className="mt-3">
            <Progress
              value={task.completion_percentage}
              size="sm"
              color={task.completion_percentage === 100 ? 'green' : 'blue'}
            />
          </div>
        )}

        {/* Footer */}
        <div className="flex items-center justify-between mt-3">
          <div className="flex items-center gap-3">
            {task.assigned_to && (
              <div className="flex items-center gap-1.5">
                <Avatar
                  user={task.assigned_to}
                  size="sm"
                  fallback={task.assigned_to.first_name?.[0] || 'U'}
                />
                <span className="text-xs text-gray-600">
                  {task.assigned_to.first_name} {task.assigned_to.last_name?.[0]}.
                </span>
              </div>
            )}
            {task.category && (
              <Badge
                variant="outline"
                color={task.category.color || 'gray'}
                size="sm"
              >
                {task.category.category_name}
              </Badge>
            )}
          </div>

          <div className="flex items-center gap-3 text-xs text-gray-500">
            {task.due_date && (
              <div className={`flex items-center gap-1 ${task.is_overdue_days > 0 ? 'text-red-500' : ''}`}>
                <CalendarIcon className="w-3 h-3" />
                <span>
                  {task.is_overdue_days > 0
                    ? `${task.is_overdue_days} days overdue`
                    : formatRelative(task.due_date)}
                </span>
              </div>
            )}
            {task.comments && task.comments.length > 0 && (
              <div className="flex items-center gap-1">
                <ChatBubbleLeftIcon className="w-3 h-3" />
                <span>{task.comments.length}</span>
              </div>
            )}
            {task.attachments && task.attachments.length > 0 && (
              <div className="flex items-center gap-1">
                <PaperClipIcon className="w-3 h-3" />
                <span>{task.attachments.length}</span>
              </div>
            )}
          </div>
        </div>

        {/* Task Score */}
        {task.task_score > 0 && (
          <div className="mt-2 pt-2 border-t border-gray-100 flex items-center justify-between">
            <div className="flex items-center gap-2">
              <span className="text-xs text-gray-500">Score:</span>
              <div className="flex items-center gap-1">
                <span className="text-sm font-medium text-green-600">
                  {task.task_score.toFixed(1)}
                </span>
                <span className="text-xs text-gray-400">/ 100</span>
              </div>
            </div>
            {task.contribution_score > 0 && (
              <div className="flex items-center gap-2">
                <span className="text-xs text-gray-500">Contribution:</span>
                <span className="text-sm font-medium text-blue-600">
                  {task.contribution_score.toFixed(1)}
                </span>
              </div>
            )}
          </div>
        )}
      </div>
    </Link>
  );
};
```

### 5.3 Task Detail Component

```typescript
// components/tasks/TaskDetail/TaskDetailView.tsx

'use client';

import { useState, useEffect } from 'react';
import { useRouter } from 'next/navigation';
import { Task } from '@/types/task.types';
import { TaskHeader } from './TaskHeader';
import { TaskInfo } from './TaskInfo';
import { TaskComments } from './TaskComments';
import { TaskAttachments } from './TaskAttachments';
import { TaskTimeline } from './TaskTimeline';
import { TaskActions } from './TaskActions';
import { TaskDependencies } from './TaskDependencies';
import { Button } from '@/components/ui/Button';
import { ArrowLeftIcon } from '@heroicons/react/24/outline';
import { useTask } from '@/hooks/useTask';
import { useTaskComments } from '@/hooks/useTaskComments';
import { useTaskAttachments } from '@/hooks/useTaskAttachments';
import { LoadingSpinner } from '@/components/ui/LoadingSpinner';

interface TaskDetailViewProps {
  taskId: string;
  onBack?: () => void;
}

export const TaskDetailView = ({ taskId, onBack }: TaskDetailViewProps) => {
  const router = useRouter();
  const [activeTab, setActiveTab] = useState<'details' | 'comments' | 'attachments' | 'timeline'>('details');
  
  const { task, isLoading: taskLoading, updateTask } = useTask(taskId);
  const { comments, isLoading: commentsLoading, addComment, markConcern } = useTaskComments(taskId);
  const { attachments, isLoading: attachmentsLoading, uploadAttachment, deleteAttachment } = useTaskAttachments(taskId);

  if (taskLoading) {
    return (
      <div className="flex items-center justify-center h-96">
        <LoadingSpinner />
      </div>
    );
  }

  if (!task) {
    return (
      <div className="text-center py-12">
        <h3 className="text-lg font-medium text-gray-900">Task not found</h3>
        <p className="mt-2 text-sm text-gray-500">The task you're looking for doesn't exist or you don't have access.</p>
        <Button className="mt-4" onClick={onBack || (() => router.back())}>
          Go Back
        </Button>
      </div>
    );
  }

  return (
    <div className="max-w-7xl mx-auto">
      {/* Header */}
      <div className="flex items-center gap-4 mb-6">
        <Button
          variant="ghost"
          size="sm"
          onClick={onBack || (() => router.back())}
          className="shrink-0"
        >
          <ArrowLeftIcon className="w-4 h-4 mr-2" />
          Back
        </Button>
        <TaskHeader task={task} onUpdate={updateTask} />
      </div>

      {/* Main Content */}
      <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
        {/* Left Column - Main Content */}
        <div className="lg:col-span-2 space-y-6">
          {/* Tabs */}
          <div className="border-b border-gray-200">
            <nav className="-mb-px flex space-x-8">
              {[
                { id: 'details', label: 'Details' },
                { id: 'comments', label: 'Comments', count: comments?.length },
                { id: 'attachments', label: 'Attachments', count: attachments?.length },
                { id: 'timeline', label: 'Timeline' }
              ].map((tab) => (
                <button
                  key={tab.id}
                  onClick={() => setActiveTab(tab.id as any)}
                  className={`
                    py-2 px-1 border-b-2 font-medium text-sm
                    ${activeTab === tab.id
                      ? 'border-primary-500 text-primary-600'
                      : 'border-transparent text-gray-500 hover:text-gray-700 hover:border-gray-300'
                    }
                  `}
                >
                  {tab.label}
                  {tab.count !== undefined && (
                    <span className="ml-2 bg-gray-100 text-gray-600 px-2 py-0.5 rounded-full text-xs">
                      {tab.count}
                    </span>
                  )}
                </button>
              ))}
            </nav>
          </div>

          {/* Tab Content */}
          <div>
            {activeTab === 'details' && (
              <TaskInfo task={task} />
            )}
            
            {activeTab === 'comments' && (
              <TaskComments
                comments={comments}
                isLoading={commentsLoading}
                onAddComment={(content, isConcern) => 
                  isConcern ? markConcern(content) : addComment(content)
                }
                taskId={taskId}
              />
            )}
            
            {activeTab === 'attachments' && (
              <TaskAttachments
                attachments={attachments}
                isLoading={attachmentsLoading}
                onUpload={uploadAttachment}
                onDelete={deleteAttachment}
                taskId={taskId}
              />
            )}
            
            {activeTab === 'timeline' && (
              <TaskTimeline task={task} />
            )}
          </div>
        </div>

        {/* Right Column - Sidebar */}
        <div className="lg:col-span-1 space-y-6">
          <TaskActions
            task={task}
            onUpdate={updateTask}
          />
          
          {task.dependencies && task.dependencies.length > 0 && (
            <TaskDependencies
              task={task}
              dependencies={task.dependencies}
              onUpdate={updateTask}
            />
          )}
          
          {/* Task Metadata */}
          <div className="bg-white rounded-lg shadow p-4">
            <h4 className="text-sm font-medium text-gray-900 mb-3">Metadata</h4>
            <div className="space-y-2 text-sm">
              <div className="flex justify-between">
                <span className="text-gray-500">Created</span>
                <span className="text-gray-900">
                  {new Date(task.created_at).toLocaleString()}
                </span>
              </div>
              <div className="flex justify-between">
                <span className="text-gray-500">Updated</span>
                <span className="text-gray-900">
                  {new Date(task.updated_at).toLocaleString()}
                </span>
              </div>
              {task.created_by && (
                <div className="flex justify-between">
                  <span className="text-gray-500">Created By</span>
                  <span className="text-gray-900">
                    {task.created_by.first_name} {task.created_by.last_name}
                  </span>
                </div>
              )}
              {task.source_module && (
                <div className="flex justify-between">
                  <span className="text-gray-500">Source</span>
                  <span className="text-gray-900">
                    {task.source_module}
                    {task.source_record_id && `: ${task.source_record_id.slice(0, 8)}`}
                  </span>
                </div>
              )}
              {task.task_score > 0 && (
                <div className="flex justify-between border-t pt-2 mt-2">
                  <span className="text-gray-500">Task Score</span>
                  <span className="text-green-600 font-medium">
                    {task.task_score.toFixed(1)}
                  </span>
                </div>
              )}
              {task.contribution_score > 0 && (
                <div className="flex justify-between">
                  <span className="text-gray-500">Contribution</span>
                  <span className="text-blue-600 font-medium">
                    {task.contribution_score.toFixed(1)}
                  </span>
                </div>
              )}
            </div>
          </div>
        </div>
      </div>
    </div>
  );
};
```

### 5.4 Comment System Component

```typescript
// components/tasks/TaskDetail/TaskComments.tsx

'use client';

import { useState, useRef, useEffect } from 'react';
import { Comment } from '@/types/comment.types';
import { Avatar } from '@/components/ui/Avatar';
import { Button } from '@/components/ui/Button';
import { Badge } from '@/components/ui/Badge';
import { Textarea } from '@/components/ui/Textarea';
import { formatRelative } from '@/lib/utils/date';
import { LoadingSpinner } from '@/components/ui/LoadingSpinner';
import {
  ChatBubbleLeftIcon,
  ExclamationTriangleIcon,
  UserIcon,
  EllipsisVerticalIcon,
} from '@heroicons/react/24/outline';
import { useAuth } from '@/hooks/useAuth';

interface TaskCommentsProps {
  comments: Comment[];
  isLoading: boolean;
  onAddComment: (content: string, isConcern: boolean) => Promise<void>;
  taskId: string;
}

export const TaskComments = ({
  comments = [],
  isLoading,
  onAddComment,
  taskId
}: TaskCommentsProps) => {
  const { user } = useAuth();
  const [newComment, setNewComment] = useState('');
  const [isConcern, setIsConcern] = useState(false);
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [showConcernWarning, setShowConcernWarning] = useState(false);
  const commentInputRef = useRef<HTMLTextAreaElement>(null);

  const handleSubmit = async () => {
    if (!newComment.trim()) return;

    setIsSubmitting(true);
    try {
      await onAddComment(newComment, isConcern);
      setNewComment('');
      setIsConcern(false);
      setShowConcernWarning(false);
    } catch (error) {
      console.error('Failed to add comment:', error);
    } finally {
      setIsSubmitting(false);
    }
  };

  const handleKeyDown = (e: React.KeyboardEvent) => {
    if (e.key === 'Enter' && (e.metaKey || e.ctrlKey)) {
      handleSubmit();
    }
  };

  const toggleConcern = () => {
    if (!isConcern) {
      setShowConcernWarning(true);
    } else {
      setIsConcern(false);
      setShowConcernWarning(false);
    }
  };

  const confirmConcern = () => {
    setIsConcern(true);
    setShowConcernWarning(false);
    commentInputRef.current?.focus();
  };

  return (
    <div className="bg-white rounded-lg shadow">
      <div className="p-4 border-b border-gray-200">
        <div className="flex items-center justify-between">
          <div className="flex items-center gap-2">
            <ChatBubbleLeftIcon className="w-5 h-5 text-gray-400" />
            <h3 className="text-lg font-medium text-gray-900">Comments</h3>
            <Badge variant="outline" color="gray" size="sm">
              {comments.length}
            </Badge>
          </div>
          {isLoading && <LoadingSpinner size="sm" />}
        </div>
      </div>

      {/* Comment List */}
      <div className="max-h-96 overflow-y-auto p-4 space-y-4">
        {comments.length === 0 && !isLoading && (
          <div className="text-center py-8 text-gray-500">
            <ChatBubbleLeftIcon className="w-12 h-12 mx-auto text-gray-300" />
            <p className="mt-2 text-sm">No comments yet</p>
            <p className="text-xs">Be the first to comment on this task</p>
          </div>
        )}

        {comments.map((comment) => (
          <CommentItem
            key={comment.id}
            comment={comment}
            isCurrentUser={comment.user_id === user?.id}
          />
        ))}

        {isLoading && (
          <div className="flex justify-center py-4">
            <LoadingSpinner size="sm" />
          </div>
        )}
      </div>

      {/* New Comment Input */}
      <div className="p-4 border-t border-gray-200">
        {showConcernWarning ? (
          <div className="mb-3 p-3 bg-red-50 border border-red-200 rounded-lg">
            <div className="flex items-start gap-3">
              <ExclamationTriangleIcon className="w-5 h-5 text-red-500 mt-0.5" />
              <div className="flex-1">
                <p className="text-sm font-medium text-red-800">
                  Raising a Concern
                </p>
                <p className="text-xs text-red-600 mt-1">
                  This will flag the comment as a concern and affect the task's score.
                  Are you sure you want to proceed?
                </p>
                <div className="flex gap-2 mt-2">
                  <Button
                    variant="danger"
                    size="sm"
                    onClick={confirmConcern}
                  >
                    Yes, Raise Concern
                  </Button>
                  <Button
                    variant="outline"
                    size="sm"
                    onClick={() => setShowConcernWarning(false)}
                  >
                    Cancel
                  </Button>
                </div>
              </div>
            </div>
          </div>
        ) : (
          <div className="flex items-start gap-3">
            <Avatar
              user={user}
              size="md"
              fallback={user?.first_name?.[0] || 'U'}
            />
            <div className="flex-1 min-w-0">
              <Textarea
                ref={commentInputRef}
                placeholder="Add a comment... (Cmd+Enter to submit)"
                value={newComment}
                onChange={(e) => setNewComment(e.target.value)}
                onKeyDown={handleKeyDown}
                className="w-full"
                rows={3}
                disabled={isSubmitting}
              />
              <div className="flex items-center justify-between mt-2">
                <div className="flex items-center gap-3">
                  <button
                    type="button"
                    onClick={toggleConcern}
                    className={`
                      flex items-center gap-1 text-xs px-2 py-1 rounded
                      ${isConcern 
                        ? 'bg-red-100 text-red-700' 
                        : 'text-gray-500 hover:bg-gray-100'
                      }
                      transition-colors duration-200
                    `}
                  >
                    <ExclamationTriangleIcon className="w-4 h-4" />
                    {isConcern ? 'Concern marked' : 'Mark as concern'}
                  </button>
                </div>
                <div className="flex items-center gap-2">
                  <span className="text-xs text-gray-400">
                    Press Cmd+Enter to submit
                  </span>
                  <Button
                    variant="primary"
                    size="sm"
                    onClick={handleSubmit}
                    disabled={!newComment.trim() || isSubmitting}
                    className="shrink-0"
                  >
                    {isSubmitting ? 'Posting...' : 'Post Comment'}
                  </Button>
                </div>
              </div>
            </div>
          </div>
        )}
      </div>
    </div>
  );
};

// Individual Comment Item
const CommentItem = ({ comment, isCurrentUser }: { comment: Comment; isCurrentUser: boolean }) => {
  const [isExpanded, setIsExpanded] = useState(false);
  const [showActions, setShowActions] = useState(false);

  return (
    <div
      className={`
        flex gap-3 p-3 rounded-lg transition-colors duration-200
        ${comment.is_concern ? 'bg-red-50 border border-red-200' : 'hover:bg-gray-50'}
        ${isCurrentUser ? 'bg-primary-50 border border-primary-200' : ''}
      `}
      onMouseEnter={() => setShowActions(true)}
      onMouseLeave={() => setShowActions(false)}
    >
      <Avatar
        user={comment.user}
        size="sm"
        fallback={comment.user?.first_name?.[0] || 'U'}
        className="shrink-0"
      />
      <div className="flex-1 min-w-0">
        <div className="flex items-center gap-2 flex-wrap">
          <span className="text-sm font-medium text-gray-900">
            {comment.user?.first_name} {comment.user?.last_name}
          </span>
          <span className="text-xs text-gray-500">
            {formatRelative(comment.created_at)}
          </span>
          {comment.is_concern && (
            <Badge variant="solid" color="red" size="sm" className="shrink-0">
              Concern
            </Badge>
          )}
          {isCurrentUser && (
            <Badge variant="outline" color="primary" size="sm" className="shrink-0">
              You
            </Badge>
          )}
        </div>
        
        <p className={`
          text-sm text-gray-700 mt-1 whitespace-pre-wrap
          ${!isExpanded && comment.comment.length > 300 ? 'line-clamp-3' : ''}
        `}>
          {comment.comment}
        </p>
        
        {comment.comment.length > 300 && (
          <button
            onClick={() => setIsExpanded(!isExpanded)}
            className="text-xs text-primary-600 hover:text-primary-700 mt-1"
          >
            {isExpanded ? 'Show less' : 'Show more'}
          </button>
        )}

        {comment.is_concern && comment.sentiment_score && (
          <div className="mt-2">
            <div className="flex items-center gap-2">
              <span className="text-xs text-gray-500">Sentiment:</span>
              <div className="flex-1 max-w-40">
                <div className="w-full bg-gray-200 rounded-full h-1.5">
                  <div
                    className={`
                      h-1.5 rounded-full
                      ${comment.sentiment_score > 0.6 ? 'bg-green-500' : ''}
                      ${comment.sentiment_score > 0.3 && comment.sentiment_score <= 0.6 ? 'bg-yellow-500' : ''}
                      ${comment.sentiment_score <= 0.3 ? 'bg-red-500' : ''}
                    `}
                    style={{ width: `${(comment.sentiment_score || 0) * 100}%` }}
                  />
                </div>
              </div>
              <span className="text-xs text-gray-500">
                {(comment.sentiment_score * 100).toFixed(0)}%
              </span>
            </div>
          </div>
        )}
      </div>

      {showActions && isCurrentUser && (
        <div className="shrink-0">
          <button
            className="p-1 rounded hover:bg-gray-100"
            onClick={() => {}} // Edit/Delete actions
          >
            <EllipsisVerticalIcon className="w-4 h-4 text-gray-400" />
          </button>
        </div>
      )}
    </div>
  );
};
```

---

## 6. SCORING & ANALYTICS ENGINE

### 6.1 Scoring Service

```typescript
// services/scoring.service.ts

import { createSupabaseClient } from '@/lib/supabase/client';
import { Task, UserPerformance } from '@/types/task.types';

export class ScoringService {
  private supabase = createSupabaseClient();

  /**
   * Calculate task score on completion
   */
  async calculateTaskScore(taskId: string): Promise<number> {
    // Get task details
    const { data: task } = await this.supabase
      .from('tasks')
      .select(`
        *,
        assigned_to:assigned_to_id(id),
        comments:task_comments(
          id, is_concern, comment
        )
      `)
      .eq('id', taskId)
      .single();

    if (!task) {
      throw new Error('Task not found');
    }

    // 1. Complexity Score (pre-assigned)
    const complexity = task.complexity_score || 50;

    // 2. Quality Score (from manager review)
    const quality = task.quality_score || 0;

    // 3. Timeliness Score
    let timeliness = 0;
    if (task.completed_at && task.due_date) {
      const completed = new Date(task.completed_at);
      const due = new Date(task.due_date);
      
      if (completed <= due) {
        timeliness = 100;
      } else {
        const daysOverdue = Math.ceil((completed.getTime() - due.getTime()) / (1000 * 60 * 60 * 24));
        timeliness = Math.max(0, 100 - (daysOverdue * 10));
      }
    }

    // 4. Concern Score
    const concerns = task.comments?.filter((c: any) => c.is_concern) || [];
    const concernCount = concerns.length;
    let concernScore = 100;
    if (concernCount > 0) {
      concernScore = Math.max(0, 100 - (concernCount * 10));
    }

    // Calculate final score
    const taskScore = (
      (complexity * 0.3) +
      (quality * 0.4) +
      (timeliness * 0.2) +
      (concernScore * 0.1)
    );

    // Calculate contribution score (with bonus for concerns that led to improvements)
    const contributionScore = taskScore * (1 + (Math.min(concernCount, 5) * 0.05));

    // Update task
    await this.supabase
      .from('tasks')
      .update({
        timeliness_score: timeliness,
        concern_score: concernScore,
        task_score: taskScore,
        contribution_score: contributionScore
      })
      .eq('id', taskId);

    // Update user performance
    if (task.assigned_to) {
      await this.updateUserPerformance(task.assigned_to.id);
    }

    return taskScore;
  }

  /**
   * Update user performance score
   */
  async updateUserPerformance(userId: string, period?: string): Promise<UserPerformance> {
    const evaluationPeriod = period || this.getCurrentPeriod();

    // Get all completed tasks for user in period
    const { data: tasks } = await this.supabase
      .from('tasks')
      .select(`
        *,
        comments:task_comments(is_concern)
      `)
      .eq('assigned_to_id', userId)
      .eq('status', 'Done / Completed')
      .gte('completed_at', this.getPeriodStart(evaluationPeriod));

    if (!tasks || tasks.length === 0) {
      // No completed tasks, reset score
      const { data: score } = await this.supabase
        .from('user_performance_scores')
        .insert({
          user_id: userId,
          task_completion_score: 0,
          task_quality_score: 0,
          concern_value_score: 0,
          remark_insight_score: 0,
          collaboration_score: 0,
          leadership_score: 0,
          evaluation_period: evaluationPeriod
        })
        .select()
        .single();

      return score;
    }

    // Calculate metrics
    const totalTasks = tasks.length;
    const completedOnTime = tasks.filter(t => {
      const completed = new Date(t.completed_at);
      const due = new Date(t.due_date);
      return completed <= due;
    }).length;

    const avgQuality = tasks.reduce((sum, t) => sum + (t.quality_score || 0), 0) / totalTasks;
    const avgScore = tasks.reduce((sum, t) => sum + (t.task_score || 0), 0) / totalTasks;

    // Concern analysis
    const totalComments = tasks.reduce((sum, t) => sum + (t.comments?.length || 0), 0);
    const concernsRaised = tasks.reduce((sum, t) => 
      sum + (t.comments?.filter((c: any) => c.is_concern).length || 0), 0
    );

    // Remark insight (length of comments)
    const avgRemarkLength = tasks.reduce((sum, t) => {
      const comments = t.comments || [];
      const avgLength = comments.reduce((s, c) => s + c.comment.length, 0) / (comments.length || 1);
      return sum + avgLength;
    }, 0) / totalTasks;

    // Collaboration (number of unique collaborators)
    const collaborators = new Set();
    tasks.forEach(t => {
      t.comments?.forEach((c: any) => collaborators.add(c.user_id));
    });
    const collaborationScore = Math.min(100, (collaborators.size / totalTasks) * 100);

    // Leadership (tasks created by user that were completed by others)
    const { data: ledTasks } = await this.supabase
      .from('tasks')
      .select('id')
      .eq('created_by_id', userId)
      .eq('status', 'Done / Completed')
      .neq('assigned_to_id', userId);

    const leadershipScore = Math.min(100, ((ledTasks?.length || 0) / totalTasks) * 100);

    // Calculate component scores
    const taskCompletionScore = (completedOnTime / totalTasks) * 100;
    const taskQualityScore = avgQuality;
    const concernValueScore = Math.min(100, (concernsRaised / totalComments) * 100);
    const remarkInsightScore = Math.min(100, (avgRemarkLength / 200) * 100);

    // Calculate overall score
    const overallScore = (
      (taskCompletionScore * 0.25) +
      (taskQualityScore * 0.25) +
      (concernValueScore * 0.15) +
      (remarkInsightScore * 0.15) +
      (collaborationScore * 0.10) +
      (leadershipScore * 0.10)
    );

    // Update or insert performance record
    const { data: score } = await this.supabase
      .from('user_performance_scores')
      .upsert({
        user_id: userId,
        task_completion_score: taskCompletionScore,
        task_quality_score: taskQualityScore,
        concern_value_score: concernValueScore,
        remark_insight_score: remarkInsightScore,
        collaboration_score: collaborationScore,
        leadership_score: leadershipScore,
        evaluation_period: evaluationPeriod,
        calculated_at: new Date().toISOString()
      })
      .select()
      .single();

    return score;
  }

  /**
   * Handle concern raised on a task
   */
  async handleConcern(comment: any): Promise<void> {
    // Analyze concern for sentiment and severity
    const analysis = await this.analyzeConcern(comment.comment);

    // Store concern analysis
    await this.supabase
      .from('concern_analysis')
      .insert({
        comment_id: comment.id,
        sentiment_score: analysis.sentimentScore,
        concern_category: analysis.category,
        concern_severity: analysis.severity,
        keywords: analysis.keywords,
        analyzed_at: new Date().toISOString(),
        analyzed_by: 'AI'
      });

    // Update task score if task is completed
    const { data: task } = await this.supabase
      .from('tasks')
      .select('status')
      .eq('id', comment.task_id)
      .single();

    if (task?.status === 'Done / Completed') {
      await this.calculateTaskScore(comment.task_id);
    }
  }

  /**
   * Analyze concern using NLP
   */
  private async analyzeConcern(text: string): Promise<{
    sentimentScore: number;
    category: string;
    severity: string;
    keywords: string[];
  }> {
    // This is a simplified version. In production, use a proper NLP service.
    // For now, we'll do basic keyword analysis
    
    const keywords = ['urgent', 'critical', 'blocker', 'issue', 'problem', 'risk'];
    const foundKeywords = keywords.filter(k => text.toLowerCase().includes(k));
    
    // Calculate sentiment (simplified)
    const positiveWords = ['good', 'great', 'excellent', 'nice'];
    const negativeWords = ['bad', 'poor', 'terrible', 'awful', 'worse'];
    
    let sentimentScore = 0.5; // Neutral
    const words = text.toLowerCase().split(/\s+/);
    
    let positiveCount = words.filter(w => positiveWords.includes(w)).length;
    let negativeCount = words.filter(w => negativeWords.includes(w)).length;
    
    if (positiveCount > negativeCount) {
      sentimentScore = 0.7 + (positiveCount - negativeCount) * 0.05;
    } else if (negativeCount > positiveCount) {
      sentimentScore = 0.3 - (negativeCount - positiveCount) * 0.05;
    }
    
    sentimentScore = Math.max(0, Math.min(1, sentimentScore));
    
    // Determine category
    const categories = [
      { name: 'Process', keywords: ['process', 'procedure', 'step'] },
      { name: 'Resource', keywords: ['resource', 'staff', 'people', 'team'] },
      { name: 'Management', keywords: ['management', 'leadership', 'decisions'] },
      { name: 'Technical', keywords: ['technical', 'system', 'software', 'hardware'] },
      { name: 'Quality', keywords: ['quality', 'standard', 'specification'] },
      { name: 'Safety', keywords: ['safety', 'secure', 'danger'] }
    ];
    
    let category = 'Other';
    for (const cat of categories) {
      if (cat.keywords.some(k => text.toLowerCase().includes(k))) {
        category = cat.name;
        break;
      }
    }
    
    // Determine severity
    const severityKeywords = [
      { level: 'Critical', keywords: ['critical', 'emergency', 'fatal'] },
      { level: 'High', keywords: ['high', 'major', 'severe'] },
      { level: 'Medium', keywords: ['medium', 'moderate'] },
      { level: 'Low', keywords: ['low', 'minor'] }
    ];
    
    let severity = 'Medium';
    for (const sev of severityKeywords) {
      if (sev.keywords.some(k => text.toLowerCase().includes(k))) {
        severity = sev.level;
        break;
      }
    }
    
    return {
      sentimentScore,
      category,
      severity,
      keywords: foundKeywords
    };
  }

  /**
   * Get team performance analytics
   */
  async getTeamPerformance(departmentId: string, period?: string): Promise<any> {
    const evaluationPeriod = period || this.getCurrentPeriod();
    
    const { data: scores } = await this.supabase
      .from('user_performance_scores')
      .select(`
        *,
        user:user_id(id, first_name, last_name, department:department_id(name))
      `)
      .eq('evaluation_period', evaluationPeriod)
      .eq('user.department_id', departmentId)
      .order('overall_score', { ascending: false });

    if (!scores || scores.length === 0) {
      return {
        period: evaluationPeriod,
        average: 0,
        topPerformers: [],
        distribution: {
          top: 0,
          high: 0,
          solid: 0,
          needs_improvement: 0,
          underperformer: 0
        }
      };
    }

    // Calculate distribution
    const distribution = {
      top: 0,        // >= 90
      high: 0,       // >= 75
      solid: 0,      // >= 60
      needs_improvement: 0, // >= 40
      underperformer: 0     // < 40
    };

    scores.forEach((s: any) => {
      const score = s.overall_score || 0;
      if (score >= 90) distribution.top++;
      else if (score >= 75) distribution.high++;
      else if (score >= 60) distribution.solid++;
      else if (score >= 40) distribution.needs_improvement++;
      else distribution.underperformer++;
    });

    return {
      period: evaluationPeriod,
      total: scores.length,
      average: scores.reduce((sum: number, s: any) => sum + (s.overall_score || 0), 0) / scores.length,
      topPerformers: scores.slice(0, 5),
      distribution,
      trends: await this.getPerformanceTrends(departmentId)
    };
  }

  /**
   * Get performance trends over time
   */
  private async getPerformanceTrends(departmentId: string): Promise<any> {
    const periods = this.getLastNPeriods(4);
    const trends = [];

    for (const period of periods) {
      const { data } = await this.supabase
        .from('user_performance_scores')
        .select('overall_score')
        .eq('evaluation_period', period)
        .eq('user.department_id', departmentId);

      trends.push({
        period,
        average: data?.reduce((sum, s) => sum + (s.overall_score || 0), 0) / (data?.length || 1) || 0,
        count: data?.length || 0
      });
    }

    return trends;
  }

  /**
   * Get current period string
   */
  private getCurrentPeriod(): string {
    const now = new Date();
    const quarter = Math.floor(now.getMonth() / 3) + 1;
    return `${now.getFullYear()}-Q${quarter}`;
  }

  /**
   * Get period start date
   */
  private getPeriodStart(period: string): Date {
    const [year, quarter] = period.split('-Q');
    const quarterStart = (parseInt(quarter) - 1) * 3;
    return new Date(parseInt(year), quarterStart, 1);
  }

  /**
   * Get last N periods
   */
  private getLastNPeriods(n: number): string[] {
    const periods = [];
    const now = new Date();
    let year = now.getFullYear();
    let quarter = Math.floor(now.getMonth() / 3) + 1;

    for (let i = 0; i < n; i++) {
      periods.push(`${year}-Q${quarter}`);
      quarter--;
      if (quarter === 0) {
        quarter = 4;
        year--;
      }
    }

    return periods;
  }
}
```

### 6.2 Analytics Service

```typescript
// services/analytics.service.ts

import { createSupabaseClient } from '@/lib/supabase/client';
import { CacheService } from '@/lib/cache/cache.service';

export class AnalyticsService {
  private supabase = createSupabaseClient();
  private cache = new CacheService();

  /**
   * Get task analytics dashboard data
   */
  async getTaskAnalyticsDashboard(period: string = 'monthly'): Promise<any> {
    const cacheKey = `analytics:dashboard:${period}`;
    const cached = await this.cache.get(cacheKey);
    
    if (cached) {
      return cached;
    }

    const [overview, trends, statusDistribution, priorityDistribution, topPerformers] = await Promise.all([
      this.getOverviewMetrics(period),
      this.getTrends(period),
      this.getStatusDistribution(period),
      this.getPriorityDistribution(period),
      this.getTopPerformers()
    ]);

    const dashboard = {
      period,
      overview,
      trends,
      statusDistribution,
      priorityDistribution,
      topPerformers
    };

    await this.cache.set(cacheKey, dashboard, 300); // 5 minutes

    return dashboard;
  }

  /**
   * Get overview metrics
   */
  private async getOverviewMetrics(period: string): Promise<any> {
    let query = this.supabase
      .from('tasks')
      .select('*', { count: 'exact' });

    if (period === 'monthly') {
      query = query.gte('created_at', this.getMonthStart());
    } else if (period === 'quarterly') {
      query = query.gte('created_at', this.getQuarterStart());
    } else if (period === 'yearly') {
      query = query.gte('created_at', this.getYearStart());
    }

    const { data, count } = await query;

    const total = count || 0;
    const completed = data?.filter(t => t.status === 'Done / Completed').length || 0;
    const inProgress = data?.filter(t => t.status === 'In Progress' || t.status === 'Work in Progress').length || 0;
    const overdue = data?.filter(t => t.is_overdue_days && t.is_overdue_days > 0).length || 0;

    const averageScore = data?.reduce((sum, t) => sum + (t.task_score || 0), 0) / (data?.length || 1) || 0;
    const completionRate = total > 0 ? (completed / total) * 100 : 0;

    return {
      totalTasks: total,
      completed,
      inProgress,
      overdue,
      averageScore,
      completionRate,
      pendingTasks: total - completed - inProgress
    };
  }

  /**
   * Get task trends over time
   */
  private async getTrends(period: string): Promise<any> {
    let interval: string;
    let limit: number;

    if (period === 'monthly') {
      interval = 'day';
      limit = 30;
    } else if (period === 'quarterly') {
      interval = 'week';
      limit = 13;
    } else {
      interval = 'month';
      limit = 12;
    }

    const { data } = await this.supabase
      .from('task_analytics_summary')
      .select('*')
      .order('month', { ascending: false })
      .limit(limit);

    return data || [];
  }

  /**
   * Get task status distribution
   */
  private async getStatusDistribution(period: string): Promise<any> {
    const { data } = await this.supabase
      .from('tasks')
      .select('status, count')
      .gte('created_at', this.getPeriodStart(period))
      .group('status');

    return data?.map((d: any) => ({
      status: d.status,
      count: d.count,
      percentage: 0 // Calculate based on total
    })) || [];
  }

  /**
   * Get task priority distribution
   */
  private async getPriorityDistribution(period: string): Promise<any> {
    const { data } = await this.supabase
      .from('tasks')
      .select('priority, count')
      .gte('created_at', this.getPeriodStart(period))
      .group('priority');

    return data?.map((d: any) => ({
      priority: d.priority,
      count: d.count,
      percentage: 0 // Calculate based on total
    })) || [];
  }

  /**
   * Get top performers
   */
  private async getTopPerformers(limit: number = 10): Promise<any> {
    const { data } = await this.supabase
      .from('user_performance_scores')
      .select(`
        user_id,
        overall_score,
        performance_band,
        user:user_id(first_name, last_name, department:department_id(name))
      `)
      .order('overall_score', { ascending: false })
      .limit(limit);

    return data || [];
  }

  /**
   * Generate performance report
   */
  async generatePerformanceReport(departmentId?: string): Promise<any> {
    const scores = await this.supabase
      .from('user_performance_scores')
      .select(`
        *,
        user:user_id(
          id, first_name, last_name, 
          department:department_id(id, name)
        )
      `)
      .eq('evaluation_period', this.getCurrentPeriod());

    let data = scores.data || [];

    if (departmentId) {
      data = data.filter((s: any) => s.user?.department?.id === departmentId);
    }

    if (data.length === 0) {
      return {
        period: this.getCurrentPeriod(),
        totalEmployees: 0,
        averageScore: 0,
        distribution: {},
        topPerformers: [],
        bottomPerformers: []
      };
    }

    // Calculate distribution
    const distribution = {
      'Top Performer': 0,
      'High Performer': 0,
      'Solid Performer': 0,
      'Needs Improvement': 0,
      'Underperformer': 0
    };

    data.forEach((s: any) => {
      const band = s.performance_band || 'Needs Improvement';
      if (distribution[band] !== undefined) {
        distribution[band]++;
      }
    });

    const sorted = [...data].sort((a, b) => (b.overall_score || 0) - (a.overall_score || 0));

    return {
      period: this.getCurrentPeriod(),
      totalEmployees: data.length,
      averageScore: data.reduce((sum, s) => sum + (s.overall_score || 0), 0) / data.length,
      distribution,
      topPerformers: sorted.slice(0, 5).map((s: any) => ({
        name: `${s.user.first_name} ${s.user.last_name}`,
        score: s.overall_score,
        department: s.user?.department?.name
      })),
      bottomPerformers: sorted.slice(-5).reverse().map((s: any) => ({
        name: `${s.user.first_name} ${s.user.last_name}`,
        score: s.overall_score,
        department: s.user?.department?.name
      }))
    };
  }

  /**
   * Get department workload analysis
   */
  async getDepartmentWorkload(): Promise<any> {
    const { data } = await this.supabase
      .from('tasks')
      .select(`
        *,
        assigned_to:assigned_to_id(
          id,
          department:department_id(id, name)
        )
      `)
      .in('status', ['Pending', 'In Progress', 'Work in Progress']);

    const departmentMap = new Map();

    data?.forEach((task: any) => {
      const deptName = task.assigned_to?.department?.name || 'Unassigned';
      if (!departmentMap.has(deptName)) {
        departmentMap.set(deptName, {
          department: deptName,
          total: 0,
          byPriority: { Low: 0, Medium: 0, High: 0, 'Urgent/Overdue': 0 }
        });
      }

      const dept = departmentMap.get(deptName);
      dept.total++;
      dept.byPriority[task.priority] = (dept.byPriority[task.priority] || 0) + 1;
    });

    return Array.from(departmentMap.values());
  }

  /**
   * Get task aging report
   */
  async getTaskAgingReport(): Promise<any> {
    const { data } = await this.supabase
      .from('tasks')
      .select(`
        *,
        assigned_to:assigned_to_id(first_name, last_name),
        category:category_id(category_name)
      `)
      .not('status', 'in', '("Done / Completed", "Cancelled / On Hold")')
      .order('created_at', { ascending: true });

    const agingBuckets = {
      '0-5': [],
      '6-10': [],
      '11-15': [],
      '16-20': [],
      '21-30': [],
      '30+': []
    };

    data?.forEach((task: any) => {
      const days = Math.floor((Date.now() - new Date(task.created_at).getTime()) / (1000 * 60 * 60 * 24));
      
      let bucket = '30+';
      if (days <= 5) bucket = '0-5';
      else if (days <= 10) bucket = '6-10';
      else if (days <= 15) bucket = '11-15';
      else if (days <= 20) bucket = '16-20';
      else if (days <= 30) bucket = '21-30';

      agingBuckets[bucket].push({
        id: task.id,
        title: task.title,
        assigned_to: task.assigned_to ? 
          `${task.assigned_to.first_name} ${task.assigned_to.last_name}` : 
          'Unassigned',
        category: task.category?.category_name || 'Uncategorized',
        status: task.status,
        days
      });
    });

    return agingBuckets;
  }

  // ============================================
  // PRIVATE HELPER METHODS
  // ============================================

  private getPeriodStart(period: string): Date {
    const now = new Date();
    if (period === 'monthly') {
      return new Date(now.getFullYear(), now.getMonth(), 1);
    } else if (period === 'quarterly') {
      const quarter = Math.floor(now.getMonth() / 3);
      return new Date(now.getFullYear(), quarter * 3, 1);
    } else if (period === 'yearly') {
      return new Date(now.getFullYear(), 0, 1);
    }
    return new Date(now.getFullYear(), now.getMonth(), 1);
  }

  private getMonthStart(): Date {
    const now = new Date();
    return new Date(now.getFullYear(), now.getMonth(), 1);
  }

  private getQuarterStart(): Date {
    const now = new Date();
    const quarter = Math.floor(now.getMonth() / 3);
    return new Date(now.getFullYear(), quarter * 3, 1);
  }

  private getYearStart(): Date {
    return new Date(new Date().getFullYear(), 0, 1);
  }

  private getCurrentPeriod(): string {
    const now = new Date();
    const quarter = Math.floor(now.getMonth() / 3) + 1;
    return `${now.getFullYear()}-Q${quarter}`;
  }
}
```

---

## 7. NOTIFICATION SYSTEM

### 7.1 Notification Service

```typescript
// services/notification.service.ts

import { createSupabaseClient } from '@/lib/supabase/client';
import { sendEmail } from '@/lib/email';
import { sendSMS } from '@/lib/sms';

export class NotificationService {
  private supabase = createSupabaseClient();

  /**
   * Send task assignment notification
   */
  async taskAssigned(task: any): Promise<void> {
    const assignee = await this.getUser(task.assigned_to_id);
    if (!assignee) return;

    // Email
    await sendEmail({
      to: assignee.email,
      subject: `New Task Assigned: ${task.task_number}`,
      template: 'task-assignment',
      data: {
        taskNumber: task.task_number,
        title: task.title,
        description: task.description,
        priority: task.priority,
        dueDate: new Date(task.due_date).toLocaleDateString(),
        assignedBy: task.created_by?.first_name || 'System',
        taskUrl: `${process.env.NEXT_PUBLIC_APP_URL}/tasks/${task.id}`
      }
    });

    // SMS for urgent tasks
    if (task.priority === 'Urgent/Overdue' || task.priority === 'High') {
      await sendSMS({
        to: assignee.phone,
        message: `URGENT: New task assigned - ${task.title}. Due: ${new Date(task.due_date).toLocaleDateString()}`
      });
    }

    // In-app notification
    await this.createNotification({
      user_id: assignee.id,
      title: 'New Task Assigned',
      message: `${task.title} has been assigned to you (${task.priority})`,
      type: 'task_assignment',
      link: `/tasks/${task.id}`,
      task_id: task.id
    });

    // Push notification if mobile device registered
    if (assignee.push_token) {
      await this.sendPushNotification(assignee.push_token, {
        title: 'New Task Assigned',
        body: `${task.title} has been assigned to you`,
        data: { taskId: task.id }
      });
    }
  }

  /**
   * Send comment notification
   */
  async newComment(task: any, comment: any): Promise<void> {
    const commenter = await this.getUser(comment.user_id);
    const assignee = await this.getUser(task.assigned_to_id);
    const creator = await this.getUser(task.created_by_id);

    // Notify assignee (if different from commenter)
    if (assignee && assignee.id !== commenter.id) {
      await this.createNotification({
        user_id: assignee.id,
        title: `New comment on ${task.task_number}`,
        message: `${commenter.first_name}: ${comment.comment.substring(0, 100)}${comment.comment.length > 100 ? '...' : ''}`,
        type: comment.is_concern ? 'concern' : 'comment',
        link: `/tasks/${task.id}#comment-${comment.id}`,
        task_id: task.id
      });

      // Email for concern comments
      if (comment.is_concern) {
        await sendEmail({
          to: assignee.email,
          subject: `Concern Raised: ${task.task_number}`,
          template: 'concern-raised',
          data: {
            taskNumber: task.task_number,
            title: task.title,
            comment: comment.comment,
            commenter: `${commenter.first_name} ${commenter.last_name}`,
            taskUrl: `${process.env.NEXT_PUBLIC_APP_URL}/tasks/${task.id}`
          }
        });
      }
    }

    // Notify creator (if different from commenter and assignee)
    if (creator && creator.id !== commenter.id && creator.id !== assignee?.id) {
      await this.createNotification({
        user_id: creator.id,
        title: `New comment on ${task.task_number}`,
        message: `${commenter.first_name}: ${comment.comment.substring(0, 100)}${comment.comment.length > 100 ? '...' : ''}`,
        type: 'comment',
        link: `/tasks/${task.id}#comment-${comment.id}`,
        task_id: task.id
      });
    }
  }

  /**
   * Send approval required notification
   */
  async approvalRequired(requestId: string, approverId: string): Promise<void> {
    const request = await this.getRequest(requestId);
    const approver = await this.getUser(approverId);
    const requester = await this.getUser(request.requested_by_id);

    if (!approver) return;

    await this.createNotification({
      user_id: approver.id,
      title: 'Approval Required',
      message: `${requester.first_name} requires your approval for: ${request.request_data.title || request.task.title}`,
      type: 'approval_required',
      link: `/requests/${requestId}`,
      request_id: requestId
    });

    await sendEmail({
      to: approver.email,
      subject: `Approval Required: Request ${request.request_number}`,
      template: 'approval-required',
      data: {
        requestNumber: request.request_number,
        title: request.request_data.title || request.task.title,
        requester: `${requester.first_name} ${requester.last_name}`,
        department: request.department?.name,
        requestUrl: `${process.env.NEXT_PUBLIC_APP_URL}/requests/${requestId}`
      }
    });
  }

  /**
   * Send request submitted notification
   */
  async requestSubmitted(request: any): Promise<void> {
    const requester = await this.getUser(request.requested_by_id);
    const departmentHead = await this.getDepartmentHead(request.department_id);

    if (departmentHead) {
      await this.createNotification({
        user_id: departmentHead.id,
        title: 'New Request Submitted',
        message: `${requester.first_name} submitted a request: ${request.request_data.title}`,
        type: 'request_submitted',
        link: `/requests/${request.id}`,
        request_id: request.id
      });
    }
  }

  /**
   * Send request approved notification
   */
  async requestApproved(requestId: string): Promise<void> {
    const request = await this.getRequest(requestId);
    const requester = await this.getUser(request.requested_by_id);

    await this.createNotification({
      user_id: requester.id,
      title: 'Request Approved',
      message: `Your request "${request.request_data.title}" has been approved`,
      type: 'request_approved',
      link: `/requests/${requestId}`,
      request_id: requestId
    });

    await sendEmail({
      to: requester.email,
      subject: `Request Approved: ${request.request_number}`,
      template: 'request-approved',
      data: {
        requestNumber: request.request_number,
        title: request.request_data.title,
        requestUrl: `${process.env.NEXT_PUBLIC_APP_URL}/requests/${requestId}`
      }
    });
  }

  /**
   * Send request rejected notification
   */
  async requestRejected(requestId: string, reason: string): Promise<void> {
    const request = await this.getRequest(requestId);
    const requester = await this.getUser(request.requested_by_id);

    await this.createNotification({
      user_id: requester.id,
      title: 'Request Rejected',
      message: `Your request "${request.request_data.title}" was rejected. Reason: ${reason}`,
      type: 'request_rejected',
      link: `/requests/${requestId}`,
      request_id: requestId
    });

    await sendEmail({
      to: requester.email,
      subject: `Request Rejected: ${request.request_number}`,
      template: 'request-rejected',
      data: {
        requestNumber: request.request_number,
        title: request.request_data.title,
        reason: reason,
        requestUrl: `${process.env.NEXT_PUBLIC_APP_URL}/requests/${requestId}`
      }
    });
  }

  /**
   * Send task overdue notification
   */
  async taskOverdue(task: any): Promise<void> {
    const assignee = await this.getUser(task.assigned_to_id);
    const creator = await this.getUser(task.created_by_id);

    if (assignee) {
      await this.createNotification({
        user_id: assignee.id,
        title: 'Task Overdue',
        message: `${task.title} is overdue. Please complete as soon as possible.`,
        type: 'overdue',
        link: `/tasks/${task.id}`,
        task_id: task.id
      });

      await sendSMS({
        to: assignee.phone,
        message: `ALERT: Task "${task.title}" is overdue. Please complete immediately.`
      });
    }

    if (creator && creator.id !== assignee?.id) {
      await this.createNotification({
        user_id: creator.id,
        title: 'Task Overdue',
        message: `${task.title} assigned to ${assignee?.first_name} is overdue.`,
        type: 'overdue',
        link: `/tasks/${task.id}`,
        task_id: task.id
      });
    }
  }

  // ============================================
  // PRIVATE HELPER METHODS
  // ============================================

  private async createNotification(data: any): Promise<void> {
    await this.supabase
      .from('notifications')
      .insert({
        ...data,
        is_read: false,
        created_at: new Date().toISOString()
      });

    // Real-time broadcast
    await this.supabase.channel('notifications')
      .send({
        type: 'broadcast',
        event: 'notification',
        payload: data
      });
  }

  private async getUser(userId: string): Promise<any> {
    if (!userId) return null;
    
    const { data } = await this.supabase
      .from('users')
      .select('*')
      .eq('id', userId)
      .single();
    
    return data;
  }

  private async getRequest(requestId: string): Promise<any> {
    const { data } = await this.supabase
      .from('requests')
      .select('*, task:task_id(*), department:department_id(*)')
      .eq('id', requestId)
      .single();
    
    return data;
  }

  private async getDepartmentHead(departmentId: string): Promise<any> {
    if (!departmentId) return null;

    const { data } = await this.supabase
      .from('users')
      .select('*')
      .eq('department_id', departmentId)
      .eq('is_department_head', true)
      .limit(1)
      .single();

    return data;
  }

  private async sendPushNotification(token: string, payload: any): Promise<void> {
    // Implement push notification using Firebase Cloud Messaging
    // or other push notification service
    console.log('Sending push notification:', { token, payload });
  }
}
```

---

## 8. MOBILE INTEGRATION

### 8.1 Mobile API (GraphQL)

```typescript
// api/graphql/mobile.resolver.ts

import { TaskService } from '@/services/api/task.service';
import { RequestService } from '@/services/api/request.service';
import { NotificationService } from '@/services/notification.service';

export const MobileResolvers = {
  Query: {
    // Task queries
    tasks: async (_: any, { filters, pagination }: any, context: any) => {
      const taskService = new TaskService();
      return await taskService.getTasks(filters, pagination);
    },
    
    task: async (_: any, { id }: any, context: any) => {
      const taskService = new TaskService();
      return await taskService.getTask(id);
    },
    
    myTasks: async (_: any, { status, limit }: any, context: any) => {
      const taskService = new TaskService();
      return await taskService.getTasks(
        { assignedTo: context.user.id, status },
        { page: 1, limit: limit || 20 }
      );
    },
    
    requests: async (_: any, { status, limit }: any, context: any) => {
      const requestService = new RequestService();
      return await requestService.getRequests({
        requestedBy: context.user.id,
        status,
        limit: limit || 20
      });
    },

    notifications: async (_: any, { limit }: any, context: any) => {
      const notificationService = new NotificationService();
      return await notificationService.getUserNotifications(context.user.id, limit);
    },

    performanceScore: async (_: any, { userId }: any, context: any) => {
      const scoringService = new ScoringService();
      return await scoringService.getUserPerformance(userId || context.user.id);
    }
  },

  Mutation: {
    // Task mutations
    createTask: async (_: any, { input }: any, context: any) => {
      const taskService = new TaskService();
      return await taskService.createTask({
        ...input,
        createdBy: context.user.id
      });
    },
    
    updateTask: async (_: any, { id, input }: any, context: any) => {
      const taskService = new TaskService();
      return await taskService.updateTask(id, input);
    },
    
    updateTaskStatus: async (_: any, { id, status }: any, context: any) => {
      const taskService = new TaskService();
      return await taskService.updateTaskStatus(id, status);
    },
    
    addComment: async (_: any, { taskId, content, isConcern }: any, context: any) => {
      const taskService = new TaskService();
      return await taskService.addComment(taskId, content, isConcern);
    },
    
    uploadAttachment: async (_: any, { taskId, file }: any, context: any) => {
      const documentService = new DocumentService();
      return await documentService.uploadDocument(file, {
        sourceModule: 'task',
        sourceRecordId: taskId,
        documentName: file.name,
        documentTypeId: 'task-attachment'
      });
    },

    // Request mutations
    createRequest: async (_: any, { input }: any, context: any) => {
      const requestService = new RequestService();
      return await requestService.createRequest({
        ...input,
        requestedBy: context.user.id
      });
    },
    
    approveRequest: async (_: any, { requestId, comments }: any, context: any) => {
      const requestService = new RequestService();
      return await requestService.approveRequest(requestId, comments);
    },
    
    rejectRequest: async (_: any, { requestId, reason }: any, context: any) => {
      const requestService = new RequestService();
      return await requestService.rejectRequest(requestId, reason);
    },

    // Notification mutations
    markNotificationRead: async (_: any, { notificationId }: any, context: any) => {
      const notificationService = new NotificationService();
      return await notificationService.markAsRead(notificationId);
    },
    
    markAllNotificationsRead: async (_: any, __: any, context: any) => {
      const notificationService = new NotificationService();
      return await notificationService.markAllAsRead(context.user.id);
    }
  },

  Subscription: {
    taskUpdated: {
      subscribe: (_: any, __: any, context: any) => {
        // Implement WebSocket subscription for real-time updates
        return context.pubsub.asyncIterator('TASK_UPDATED');
      }
    },
    newNotification: {
      subscribe: (_: any, __: any, context: any) => {
        return context.pubsub.asyncIterator(`NOTIFICATION_${context.user.id}`);
      }
    }
  }
};
```

### 8.2 Mobile App SDK

```typescript
// mobile/sdk/RiceWorldSDK.ts

import { ApolloClient, InMemoryCache, gql } from '@apollo/client';
import { Platform } from 'react-native';
import AsyncStorage from '@react-native-async-storage/async-storage';
import * as FileSystem from 'expo-file-system';

export class RiceWorldSDK {
  private client: ApolloClient<any>;
  private baseURL: string;
  private token: string | null;

  constructor(config: { baseURL: string; token?: string }) {
    this.baseURL = config.baseURL;
    this.token = config.token || null;

    this.client = new ApolloClient({
      uri: `${this.baseURL}/graphql`,
      cache: new InMemoryCache(),
      headers: this.token ? {
        Authorization: `Bearer ${this.token}`
      } : {}
    });
  }

  // ============================================
  // AUTHENTICATION
  // ============================================

  async login(email: string, password: string): Promise<any> {
    const response = await fetch(`${this.baseURL}/api/auth/login`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email, password })
    });

    const data = await response.json();
    if (data.token) {
      this.token = data.token;
      await AsyncStorage.setItem('auth_token', data.token);
      await AsyncStorage.setItem('user', JSON.stringify(data.user));
    }

    return data;
  }

  async logout(): Promise<void> {
    this.token = null;
    await AsyncStorage.removeItem('auth_token');
    await AsyncStorage.removeItem('user');
  }

  async getCurrentUser(): Promise<any> {
    const userJson = await AsyncStorage.getItem('user');
    return userJson ? JSON.parse(userJson) : null;
  }

  // ============================================
  // TASK METHODS
  // ============================================

  async getTasks(filters?: any): Promise<any> {
    const query = gql`
      query GetTasks($filters: TaskFiltersInput, $pagination: PaginationInput) {
        tasks(filters: $filters, pagination: $pagination) {
          tasks {
            id
            task_number
            title
            description
            status
            priority
            due_date
            completion_percentage
            task_score
            assigned_to {
              id
              first_name
              last_name
              avatar_url
            }
            category {
              category_name
              color
            }
          }
          total
          page
          limit
        }
      }
    `;

    const result = await this.client.query({
      query,
      variables: {
        filters,
        pagination: { page: 1, limit: 50 }
      },
      fetchPolicy: 'network-only'
    });

    return result.data.tasks;
  }

  async getTask(id: string): Promise<any> {
    const query = gql`
      query GetTask($id: ID!) {
        task(id: $id) {
          id
          task_number
          title
          description
          status
          priority
          due_date
          completion_percentage
          task_score
          contribution_score
          assigned_to {
            id
            first_name
            last_name
            email
            phone
            avatar_url
          }
          created_by {
            id
            first_name
            last_name
          }
          category {
            category_name
            color
          }
          job_location {
            location_name
          }
          comments {
            id
            comment
            is_concern
            created_at
            user {
              id
              first_name
              last_name
              avatar_url
            }
          }
          attachments {
            id
            file_name
            file_url
            file_type
            uploaded_at
          }
          dependencies {
            predecessor_task_id
            predecessor {
              id
              title
            }
          }
        }
      }
    `;

    const result = await this.client.query({
      query,
      variables: { id }
    });

    return result.data.task;
  }

  async createTask(input: any): Promise<any> {
    const mutation = gql`
      mutation CreateTask($input: CreateTaskInput!) {
        createTask(input: $input) {
          id
          task_number
          title
          status
        }
      }
    `;

    const result = await this.client.mutate({
      mutation,
      variables: { input }
    });

    return result.data.createTask;
  }

  async updateTask(id: string, input: any): Promise<any> {
    const mutation = gql`
      mutation UpdateTask($id: ID!, $input: UpdateTaskInput!) {
        updateTask(id: $id, input: $input) {
          id
          status
          completion_percentage
          task_score
        }
      }
    `;

    const result = await this.client.mutate({
      mutation,
      variables: { id, input }
    });

    return result.data.updateTask;
  }

  async updateTaskStatus(id: string, status: string): Promise<any> {
    return this.updateTask(id, { status });
  }

  async addComment(taskId: string, content: string, isConcern: boolean = false): Promise<any> {
    const mutation = gql`
      mutation AddComment($taskId: ID!, $content: String!, $isConcern: Boolean!) {
        addComment(taskId: $taskId, content: $content, isConcern: $isConcern) {
          id
          comment
          is_concern
          created_at
          user {
            id
            first_name
            last_name
          }
        }
      }
    `;

    const result = await this.client.mutate({
      mutation,
      variables: { taskId, content, isConcern }
    });

    return result.data.addComment;
  }

  async uploadAttachment(taskId: string, fileUri: string, fileName: string): Promise<any> {
    const formData = new FormData();
    formData.append('file', {
      uri: fileUri,
      name: fileName,
      type: this.getMimeType(fileName)
    } as any);
    formData.append('taskId', taskId);

    const response = await fetch(`${this.baseURL}/api/upload`, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${this.token}`
      },
      body: formData
    });

    const data = await response.json();
    return data;
  }

  // ============================================
  // REQUEST METHODS
  // ============================================

  async getRequests(status?: string, limit: number = 20): Promise<any> {
    const query = gql`
      query GetRequests($status: String, $limit: Int) {
        requests(status: $status, limit: $limit) {
          id
          request_number
          status
          created_at
          request_data
          request_type {
            type_name
            category
          }
          task {
            title
            priority
          }
        }
      }
    `;

    const result = await this.client.query({
      query,
      variables: { status, limit }
    });

    return result.data.requests;
  }

  async createRequest(input: any): Promise<any> {
    const mutation = gql`
      mutation CreateRequest($input: CreateRequestInput!) {
        createRequest(input: $input) {
          id
          request_number
          status
          task {
            id
            title
          }
        }
      }
    `;

    const result = await this.client.mutate({
      mutation,
      variables: { input }
    });

    return result.data.createRequest;
  }

  async approveRequest(requestId: string, comments?: string): Promise<any> {
    const mutation = gql`
      mutation ApproveRequest($requestId: ID!, $comments: String) {
        approveRequest(requestId: $requestId, comments: $comments) {
          id
          status
          approved_at
        }
      }
    `;

    const result = await this.client.mutate({
      mutation,
      variables: { requestId, comments }
    });

    return result.data.approveRequest;
  }

  // ============================================
  // PERFORMANCE & ANALYTICS
  // ============================================

  async getMyPerformance(): Promise<any> {
    const query = gql`
      query GetMyPerformance {
        performanceScore {
          overall_score
          task_completion_score
          task_quality_score
          concern_value_score
          remark_insight_score
          collaboration_score
          leadership_score
          performance_band
          evaluation_period
        }
      }
    `;

    const result = await this.client.query({
      query,
      fetchPolicy: 'network-only'
    });

    return result.data.performanceScore;
  }

  async getTaskAnalytics(period: string = 'monthly'): Promise<any> {
    const query = gql`
      query GetTaskAnalytics($period: String!) {
        taskAnalytics(period: $period) {
          overview {
            totalTasks
            completed
            inProgress
            overdue
            averageScore
            completionRate
          }
          trends {
            month
            total_tasks
            completed_tasks
            avg_score
          }
          statusDistribution {
            status
            count
            percentage
          }
          priorityDistribution {
            priority
            count
            percentage
          }
        }
      }
    `;

    const result = await this.client.query({
      query,
      variables: { period }
    });

    return result.data.taskAnalytics;
  }

  // ============================================
  // NOTIFICATIONS
  // ============================================

  async getNotifications(limit: number = 20): Promise<any> {
    const query = gql`
      query GetNotifications($limit: Int) {
        notifications(limit: $limit) {
          id
          title
          message
          type
          is_read
          link
          created_at
        }
      }
    `;

    const result = await this.client.query({
      query,
      variables: { limit }
    });

    return result.data.notifications;
  }

  async markNotificationRead(notificationId: string): Promise<any> {
    const mutation = gql`
      mutation MarkNotificationRead($notificationId: ID!) {
        markNotificationRead(notificationId: $notificationId) {
          id
          is_read
        }
      }
    `;

    const result = await this.client.mutate({
      mutation,
      variables: { notificationId }
    });

    return result.data.markNotificationRead;
  }

  async markAllNotificationsRead(): Promise<any> {
    const mutation = gql`
      mutation MarkAllNotificationsRead {
        markAllNotificationsRead {
          success
          count
        }
      }
    `;

    const result = await this.client.mutate({
      mutation
    });

    return result.data.markAllNotificationsRead;
  }

  // ============================================
  // OFFLINE CAPABILITY
  // ============================================

  async syncOfflineTasks(): Promise<void> {
    const offlineTasks = await AsyncStorage.getItem('offline_tasks');
    if (!offlineTasks) return;

    const tasks = JSON.parse(offlineTasks);
    for (const task of tasks) {
      try {
        await this.createTask(task);
      } catch (error) {
        console.error('Failed to sync offline task:', error);
      }
    }

    await AsyncStorage.removeItem('offline_tasks');
  }

  async saveOfflineTask(task: any): Promise<void> {
    const offlineTasks = await AsyncStorage.getItem('offline_tasks');
    const tasks = offlineTasks ? JSON.parse(offlineTasks) : [];
    tasks.push(task);
    await AsyncStorage.setItem('offline_tasks', JSON.stringify(tasks));
  }

  // ============================================
  // PRIVATE METHODS
  // ============================================

  private getMimeType(fileName: string): string {
    const extension = fileName.split('.').pop()?.toLowerCase();
    const mimeTypes: { [key: string]: string } = {
      'pdf': 'application/pdf',
      'jpg': 'image/jpeg',
      'jpeg': 'image/jpeg',
      'png': 'image/png',
      'gif': 'image/gif',
      'doc': 'application/msword',
      'docx': 'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
      'xls': 'application/vnd.ms-excel',
      'xlsx': 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet',
      'txt': 'text/plain'
    };
    return mimeTypes[extension || ''] || 'application/octet-stream';
  }

  // Real-time subscriptions
  subscribeToTaskUpdates(taskId: string, callback: (data: any) => void): () => void {
    const subscription = this.client.subscribe({
      query: gql`
        subscription OnTaskUpdated($taskId: ID!) {
          taskUpdated(taskId: $taskId) {
            id
            status
            completion_percentage
            task_score
          }
        }
      `,
      variables: { taskId }
    });

    const subscriptionInstance = subscription.subscribe({
      next: ({ data }) => callback(data.taskUpdated)
    });

    return () => subscriptionInstance.unsubscribe();
  }

  subscribeToNotifications(callback: (notification: any) => void): () => void {
    const subscription = this.client.subscribe({
      query: gql`
        subscription OnNewNotification {
          newNotification {
            id
            title
            message
            type
            link
            created_at
          }
        }
      `
    });

    const subscriptionInstance = subscription.subscribe({
      next: ({ data }) => callback(data.newNotification)
    });

    return () => subscriptionInstance.unsubscribe();
  }
}
```

---

## 9. SECURITY IMPLEMENTATION

### 9.1 RLS Policies for Task Management

```sql
-- ============================================
-- TASK MANAGEMENT RLS POLICIES
-- ============================================

-- Enable RLS on all tables
ALTER TABLE tasks ENABLE ROW LEVEL SECURITY;
ALTER TABLE task_comments ENABLE ROW LEVEL SECURITY;
ALTER TABLE task_attachments ENABLE ROW LEVEL SECURITY;
ALTER TABLE task_assignments ENABLE ROW LEVEL SECURITY;
ALTER TABLE task_dependencies ENABLE ROW LEVEL SECURITY;
ALTER TABLE task_status_history ENABLE ROW LEVEL SECURITY;
ALTER TABLE requests ENABLE ROW LEVEL SECURITY;
ALTER TABLE approvals ENABLE ROW LEVEL SECURITY;
ALTER TABLE documents ENABLE ROW LEVEL SECURITY;

-- ============================================
-- TASKS POLICIES
-- ============================================

-- View tasks
CREATE POLICY "Users can view their own tasks"
ON tasks FOR SELECT
USING (
  auth.uid() = assigned_to_id 
  OR auth.uid() = created_by_id 
  OR auth.uid() IN (
    SELECT user_id FROM task_assignments WHERE task_id = tasks.id
  )
  OR auth.uid() IN (
    SELECT user_id FROM task_comments WHERE task_id = tasks.id
  )
);

-- Department heads can view all tasks in their department
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

-- Managers can view all tasks in their department
CREATE POLICY "Managers can view department tasks"
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
      AND r.slug = 'manager'
    )
  )
);

-- Create tasks
CREATE POLICY "Users can create tasks"
ON tasks FOR INSERT
WITH CHECK (
  auth.uid() IS NOT NULL
);

-- Update tasks
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

-- Managers can update any task in their department
CREATE POLICY "Managers can update department tasks"
ON tasks FOR UPDATE
USING (
  EXISTS (
    SELECT 1 FROM users u
    JOIN user_roles ur ON ur.user_id = u.id
    JOIN roles r ON r.id = ur.role_id
    WHERE u.id = auth.uid()
    AND r.slug = 'manager'
    AND u.department_id = (
      SELECT department_id FROM users WHERE id = tasks.assigned_to_id
    )
  )
);

-- Department heads can update any task in their department
CREATE POLICY "Department heads can update department tasks"
ON tasks FOR UPDATE
USING (
  EXISTS (
    SELECT 1 FROM users u
    JOIN user_roles ur ON ur.user_id = u.id
    JOIN roles r ON r.id = ur.role_id
    WHERE u.id = auth.uid()
    AND r.slug = 'department_head'
    AND u.department_id = (
      SELECT department_id FROM users WHERE id = tasks.assigned_to_id
    )
  )
);

-- Delete tasks (admin only)
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

-- ============================================
-- TASK COMMENTS POLICIES
-- ============================================

-- View comments
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
      OR EXISTS (
        SELECT 1 FROM user_roles ur
        JOIN roles r ON ur.role_id = r.id
        WHERE ur.user_id = auth.uid()
        AND r.slug IN ('super_admin', 'owner', 'department_head', 'manager')
      )
    )
  )
);

-- Create comments
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
      OR EXISTS (
        SELECT 1 FROM user_roles ur
        JOIN roles r ON ur.role_id = r.id
        WHERE ur.user_id = auth.uid()
        AND r.slug IN ('super_admin', 'owner', 'department_head', 'manager')
      )
    )
  )
);

-- Update own comments
CREATE POLICY "Users can update their own comments"
ON task_comments FOR UPDATE
USING (auth.uid() = user_id)
WITH CHECK (auth.uid() = user_id);

-- Delete own comments
CREATE POLICY "Users can delete their own comments"
ON task_comments FOR DELETE
USING (auth.uid() = user_id);

-- Admins can delete any comment
CREATE POLICY "Admins can delete any comment"
ON task_comments FOR DELETE
USING (
  EXISTS (
    SELECT 1 FROM user_roles ur
    JOIN roles r ON ur.role_id = r.id
    WHERE ur.user_id = auth.uid()
    AND r.slug IN ('super_admin', 'owner')
  )
);

-- ============================================
-- TASK ATTACHMENTS POLICIES
-- ============================================

-- View attachments
CREATE POLICY "Users can view attachments on accessible tasks"
ON task_attachments FOR SELECT
USING (
  EXISTS (
    SELECT 1 FROM tasks t
    WHERE t.id = task_attachments.task_id
    AND (
      t.assigned_to_id = auth.uid()
      OR t.created_by_id = auth.uid()
      OR EXISTS (
        SELECT 1 FROM task_assignments ta
        WHERE ta.task_id = t.id
        AND ta.user_id = auth.uid()
      )
      OR EXISTS (
        SELECT 1 FROM user_roles ur
        JOIN roles r ON ur.role_id = r.id
        WHERE ur.user_id = auth.uid()
        AND r.slug IN ('super_admin', 'owner', 'department_head', 'manager')
      )
    )
  )
);

-- Upload attachments
CREATE POLICY "Users can upload attachments to accessible tasks"
ON task_attachments FOR INSERT
WITH CHECK (
  EXISTS (
    SELECT 1 FROM tasks t
    WHERE t.id = task_attachments.task_id
    AND (
      t.assigned_to_id = auth.uid()
      OR t.created_by_id = auth.uid()
      OR EXISTS (
        SELECT 1 FROM task_assignments ta
        WHERE ta.task_id = t.id
        AND ta.user_id = auth.uid()
      )
      OR EXISTS (
        SELECT 1 FROM user_roles ur
        JOIN roles r ON ur.role_id = r.id
        WHERE ur.user_id = auth.uid()
        AND r.slug IN ('super_admin', 'owner', 'department_head', 'manager')
      )
    )
  )
);

-- Delete attachments
CREATE POLICY "Users can delete their own attachments"
ON task_attachments FOR DELETE
USING (auth.uid() = uploaded_by_id);

-- Admins can delete any attachment
CREATE POLICY "Admins can delete any attachment"
ON task_attachments FOR DELETE
USING (
  EXISTS (
    SELECT 1 FROM user_roles ur
    JOIN roles r ON ur.role_id = r.id
    WHERE ur.user_id = auth.uid()
    AND r.slug IN ('super_admin', 'owner')
  )
);

-- ============================================
-- REQUESTS POLICIES
-- ============================================

-- View requests
CREATE POLICY "Users can view their own requests"
ON requests FOR SELECT
USING (
  auth.uid() = requested_by_id
  OR EXISTS (
    SELECT 1 FROM user_roles ur
    JOIN roles r ON ur.role_id = r.id
    WHERE ur.user_id = auth.uid()
    AND r.slug IN ('super_admin', 'owner', 'department_head', 'manager')
  )
  OR EXISTS (
    SELECT 1 FROM approvals a
    WHERE a.request_id = requests.id
    AND a.approver_id = auth.uid()
  )
);

-- Create requests
CREATE POLICY "Users can create requests"
ON requests FOR INSERT
WITH CHECK (
  auth.uid() = requested_by_id
);

-- Update requests
CREATE POLICY "Users can update their own requests"
ON requests FOR UPDATE
USING (auth.uid() = requested_by_id);

-- Approvers can update requests
CREATE POLICY "Approvers can update requests"
ON requests FOR UPDATE
USING (
  EXISTS (
    SELECT 1 FROM approvals a
    WHERE a.request_id = requests.id
    AND a.approver_id = auth.uid()
    AND a.status = 'Pending'
  )
);

-- ============================================
-- APPROVALS POLICIES
-- ============================================

-- View approvals
CREATE POLICY "Users can view approvals for their requests"
ON approvals FOR SELECT
USING (
  EXISTS (
    SELECT 1 FROM requests r
    WHERE r.id = approvals.request_id
    AND (
      r.requested_by_id = auth.uid()
      OR auth.uid() = approvals.approver_id
      OR EXISTS (
        SELECT 1 FROM user_roles ur
        JOIN roles r2 ON ur.role_id = r2.id
        WHERE ur.user_id = auth.uid()
        AND r2.slug IN ('super_admin', 'owner', 'department_head')
      )
    )
  )
);

-- Create approvals (system only)
CREATE POLICY "System can create approvals"
ON approvals FOR INSERT
WITH CHECK (false);

-- Update approvals (approvers only)
CREATE POLICY "Approvers can update approvals"
ON approvals FOR UPDATE
USING (auth.uid() = approver_id)
WITH CHECK (auth.uid() = approver_id);

-- ============================================
-- DOCUMENTS POLICIES
-- ============================================

-- View documents
CREATE POLICY "Users can view accessible documents"
ON documents FOR SELECT
USING (
  -- User uploaded the document
  auth.uid() = uploaded_by_id
  -- Document is linked to a task they can access
  OR (
    source_module = 'task'
    AND EXISTS (
      SELECT 1 FROM tasks t
      WHERE t.id = documents.source_record_id::UUID
      AND (
        t.assigned_to_id = auth.uid()
        OR t.created_by_id = auth.uid()
        OR EXISTS (
          SELECT 1 FROM task_assignments ta
          WHERE ta.task_id = t.id
          AND ta.user_id = auth.uid()
        )
        OR EXISTS (
          SELECT 1 FROM user_roles ur
          JOIN roles r ON ur.role_id = r.id
          WHERE ur.user_id = auth.uid()
          AND r.slug IN ('super_admin', 'owner', 'department_head', 'manager')
        )
      )
    )
  )
  -- Document is linked to a request they can access
  OR (
    source_module = 'request'
    AND EXISTS (
      SELECT 1 FROM requests r
      WHERE r.id = documents.source_record_id::UUID
      AND (
        r.requested_by_id = auth.uid()
        OR EXISTS (
          SELECT 1 FROM approvals a
          WHERE a.request_id = r.id
          AND a.approver_id = auth.uid()
        )
        OR EXISTS (
          SELECT 1 FROM user_roles ur
          JOIN roles r2 ON ur.role_id = r2.id
          WHERE ur.user_id = auth.uid()
          AND r2.slug IN ('super_admin', 'owner', 'department_head', 'manager')
        )
      )
    )
  )
  -- Admin access
  OR EXISTS (
    SELECT 1 FROM user_roles ur
    JOIN roles r ON ur.role_id = r.id
    WHERE ur.user_id = auth.uid()
    AND r.slug IN ('super_admin', 'owner')
  )
);

-- Upload documents
CREATE POLICY "Users can upload documents to accessible records"
ON documents FOR INSERT
WITH CHECK (
  auth.uid() = uploaded_by_id
  AND (
    -- Task access check
    (source_module = 'task' AND EXISTS (
      SELECT 1 FROM tasks t
      WHERE t.id = source_record_id::UUID
      AND (
        t.assigned_to_id = auth.uid()
        OR t.created_by_id = auth.uid()
        OR EXISTS (
          SELECT 1 FROM task_assignments ta
          WHERE ta.task_id = t.id
          AND ta.user_id = auth.uid()
        )
        OR EXISTS (
          SELECT 1 FROM user_roles ur
          JOIN roles r ON ur.role_id = r.id
          WHERE ur.user_id = auth.uid()
          AND r.slug IN ('super_admin', 'owner', 'department_head', 'manager')
        )
      )
    ))
    -- Request access check
    OR (source_module = 'request' AND EXISTS (
      SELECT 1 FROM requests r
      WHERE r.id = source_record_id::UUID
      AND (
        r.requested_by_id = auth.uid()
        OR EXISTS (
          SELECT 1 FROM approvals a
          WHERE a.request_id = r.id
          AND a.approver_id = auth.uid()
        )
        OR EXISTS (
          SELECT 1 FROM user_roles ur
          JOIN roles r2 ON ur.role_id = r2.id
          WHERE ur.user_id = auth.uid()
          AND r2.slug IN ('super_admin', 'owner', 'department_head', 'manager')
        )
      )
    ))
  )
);

-- Update documents
CREATE POLICY "Users can update their own documents"
ON documents FOR UPDATE
USING (auth.uid() = uploaded_by_id)
WITH CHECK (auth.uid() = uploaded_by_id);

-- Admins can update any document
CREATE POLICY "Admins can update any document"
ON documents FOR UPDATE
USING (
  EXISTS (
    SELECT 1 FROM user_roles ur
    JOIN roles r ON ur.role_id = r.id
    WHERE ur.user_id = auth.uid()
    AND r.slug IN ('super_admin', 'owner')
  )
);

-- Delete documents
CREATE POLICY "Users can delete their own documents"
ON documents FOR DELETE
USING (auth.uid() = uploaded_by_id);

-- Admins can delete any document
CREATE POLICY "Admins can delete any document"
ON documents FOR DELETE
USING (
  EXISTS (
    SELECT 1 FROM user_roles ur
    JOIN roles r ON ur.role_id = r.id
    WHERE ur.user_id = auth.uid()
    AND r.slug IN ('super_admin', 'owner')
  )
);
```

### 9.2 Input Validation Schemas

```typescript
// lib/validation/task.schema.ts

import { z } from 'zod';

// Task validation schemas
export const taskSchema = z.object({
  title: z.string()
    .min(1, 'Title is required')
    .max(500, 'Title must be less than 500 characters'),
  description: z.string()
    .max(5000, 'Description must be less than 5000 characters')
    .optional(),
  categoryId: z.string().uuid('Invalid category ID').optional(),
  jobLocationId: z.string().uuid('Invalid location ID').optional(),
  assignedToId: z.string().uuid('Invalid user ID').optional(),
  priority: z.enum(['Low', 'Medium', 'High', 'Urgent/Overdue']).default('Medium'),
  dueDate: z.string()
    .regex(/^\d{4}-\d{2}-\d{2}$/, 'Invalid date format')
    .refine(date => new Date(date) >= new Date(), 'Due date cannot be in the past'),
  estimatedHours: z.number()
    .positive('Estimated hours must be positive')
    .max(1000, 'Estimated hours cannot exceed 1000')
    .optional(),
  complexityScore: z.number()
    .min(0, 'Complexity score must be at least 0')
    .max(100, 'Complexity score cannot exceed 100')
    .default(50),
  sourceModule: z.string().optional(),
  sourceRecordId: z.string().optional(),
  approvalRequired: z.boolean().default(false),
  assignees: z.array(z.string().uuid('Invalid user ID')).optional(),
  dependencies: z.array(z.object({
    taskId: z.string().uuid('Invalid task ID'),
    type: z.enum(['Finish-to-Start', 'Start-to-Start', 'Finish-to-Finish', 'Start-to-Finish']).default('Finish-to-Start'),
    lagDays: z.number().int().min(0).default(0)
  })).optional(),
  attachments: z.array(z.string().uuid('Invalid attachment ID')).optional()
});

export const updateTaskSchema = taskSchema.partial();

// Request validation schemas
export const requestSchema = z.object({
  title: z.string()
    .min(1, 'Title is required')
    .max(500, 'Title must be less than 500 characters'),
  description: z.string()
    .max(5000, 'Description must be less than 5000 characters')
    .optional(),
  typeId: z.string().uuid('Invalid request type ID'),
  priority: z.enum(['Low', 'Medium', 'High', 'Urgent/Overdue']).default('Medium'),
  departmentId: z.string().uuid('Invalid department ID'),
  dueDate: z.string()
    .regex(/^\d{4}-\d{2}-\d{2}$/, 'Invalid date format')
    .optional(),
  assignedToId: z.string().uuid('Invalid user ID').optional(),
  requestData: z.object({
    // Type-specific fields
    // For new hire request
    position: z.string().optional(),
    salary: z.string().optional(),
    experience: z.string().optional(),
    department: z.string().optional(),
    // For equipment request
    equipmentName: z.string().optional(),
    quantity: z.number().int().positive().optional(),
    estimatedCost: z.number().positive().optional(),
    justification: z.string().optional(),
    // For budget request
    amount: z.number().positive().optional(),
    purpose: z.string().optional(),
    // For maintenance request
    equipmentType: z.string().optional(),
    issueDescription: z.string().optional(),
    urgency: z.enum(['Low', 'Medium', 'High', 'Critical']).optional()
  }).refine(data => Object.keys(data).length > 0, 'Request data is required')
});

// Comment validation schema
export const commentSchema = z.object({
  content: z.string()
    .min(1, 'Comment is required')
    .max(5000, 'Comment must be less than 5000 characters'),
  isConcern: z.boolean().default(false),
  parentCommentId: z.string().uuid('Invalid parent comment ID').optional()
});

// Document validation schema
export const documentSchema = z.object({
  documentTypeId: z.string().uuid('Invalid document type ID'),
  documentName: z.string()
    .min(1, 'Document name is required')
    .max(500, 'Document name must be less than 500 characters'),
  description: z.string()
    .max(2000, 'Description must be less than 2000 characters')
    .optional(),
  sourceModule: z.enum(['task', 'request', 'deal', 'order', 'employee']),
  sourceRecordId: z.string().uuid('Invalid source record ID'),
  status: z.enum(['Draft', 'Submitted', 'Reviewed', 'Approved', 'Rejected']).default('Draft'),
  tags: z.array(z.string()).optional()
});

// File upload validation
export const fileUploadSchema = z.object({
  file: z.instanceof(File)
    .refine(file => file.size <= 50 * 1024 * 1024, 'File must be less than 50MB')
    .refine(
      file => ['image/jpeg', 'image/png', 'image/gif', 'application/pdf', 'application/msword', 'application/vnd.openxmlformats-officedocument.wordprocessingml.document', 'application/vnd.ms-excel', 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet', 'text/plain'].includes(file.type),
      'Unsupported file type'
    ),
  metadata: documentSchema
});
```

---

## 10. PERFORMANCE OPTIMIZATION

### 10.1 Caching Strategy

```typescript
// lib/cache/task.cache.ts

import { CacheService } from './cache.service';
import { Task } from '@/types/task.types';

export class TaskCache {
  private cache = new CacheService();
  
  // Cache TTLs (in seconds)
  private readonly TTL = {
    TASK_LIST: 60,          // 1 minute
    TASK_DETAIL: 300,       // 5 minutes
    TASK_ANALYTICS: 600,    // 10 minutes
    PERFORMANCE: 3600,      // 1 hour
    USER_PERFORMANCE: 3600  // 1 hour
  };

  async getTasks(filters: any, pagination: any): Promise<any> {
    const key = `tasks:${JSON.stringify({ filters, pagination })}`;
    return this.cache.get(key);
  }

  async setTasks(filters: any, pagination: any, data: any): Promise<void> {
    const key = `tasks:${JSON.stringify({ filters, pagination })}`;
    await this.cache.set(key, data, this.TTL.TASK_LIST);
  }

  async getTask(taskId: string): Promise<Task | null> {
    return this.cache.get(`task:${taskId}`);
  }

  async setTask(task: Task): Promise<void> {
    await this.cache.set(`task:${task.id}`, task, this.TTL.TASK_DETAIL);
  }

  async invalidateTask(taskId: string): Promise<void> {
    await this.cache.delete(`task:${taskId}`);
    await this.cache.deletePattern('tasks:*');
  }

  async getAnalytics(period: string): Promise<any> {
    return this.cache.get(`analytics:tasks:${period}`);
  }

  async setAnalytics(period: string, data: any): Promise<void> {
    await this.cache.set(`analytics:tasks:${period}`, data, this.TTL.TASK_ANALYTICS);
  }

  async invalidateAnalytics(): Promise<void> {
    await this.cache.deletePattern('analytics:tasks:*');
  }

  async getUserPerformance(userId: string): Promise<any> {
    return this.cache.get(`performance:${userId}`);
  }

  async setUserPerformance(userId: string, data: any): Promise<void> {
    await this.cache.set(`performance:${userId}`, data, this.TTL.USER_PERFORMANCE);
  }

  async invalidateUserPerformance(userId: string): Promise<void> {
    await this.cache.delete(`performance:${userId}`);
    await this.cache.deletePattern('performance:*');
  }
}
```

### 10.2 Query Optimization

```sql
-- Advanced indexes for task management

-- Composite indexes for common filter combinations
CREATE INDEX idx_tasks_status_priority ON tasks(status, priority);
CREATE INDEX idx_tasks_status_due_date ON tasks(status, due_date) WHERE status NOT IN ('Done / Completed', 'Cancelled / On Hold');
CREATE INDEX idx_tasks_assigned_status_due ON tasks(assigned_to_id, status, due_date);

-- Partial indexes for common queries
CREATE INDEX idx_tasks_active ON tasks(id, status) WHERE status NOT IN ('Done / Completed', 'Cancelled / On Hold');
CREATE INDEX idx_tasks_overdue ON tasks(id, due_date, status) WHERE due_date < NOW() AND status NOT IN ('Done / Completed', 'Cancelled / On Hold');
CREATE INDEX idx_tasks_completion ON tasks(completion_percentage, status);

-- Gin index for JSONB querying
CREATE INDEX idx_tasks_metadata ON tasks USING GIN (metadata);

-- Index for foreign key constraints
CREATE INDEX idx_tasks_category_id ON tasks(category_id);
CREATE INDEX idx_tasks_job_location_id ON tasks(job_location_id);
CREATE INDEX idx_tasks_created_by_id ON tasks(created_by_id);

-- Comment indexes
CREATE INDEX idx_task_comments_task_id ON task_comments(task_id);
CREATE INDEX idx_task_comments_user_id ON task_comments(user_id);
CREATE INDEX idx_task_comments_concern ON task_comments(is_concern) WHERE is_concern = true;

-- Request indexes
CREATE INDEX idx_requests_status_dept ON requests(status, department_id);
CREATE INDEX idx_requests_requested_by_status ON requests(requested_by_id, status);
CREATE INDEX idx_requests_created_at ON requests(created_at DESC);

-- Approval indexes
CREATE INDEX idx_approvals_request_status ON approvals(request_id, status);
CREATE INDEX idx_approvals_approver_status ON approvals(approver_id, status);
CREATE INDEX idx_approvals_created_at ON approvals(created_at DESC);
```

### 10.3 Batch Operations

```typescript
// services/batch.service.ts

import { createSupabaseClient } from '@/lib/supabase/client';

export class BatchService {
  private supabase = createSupabaseClient();
  private batchSize = 100;

  /**
   * Batch process task status updates
   */
  async batchUpdateTaskStatus(taskIds: string[], status: string): Promise<void> {
    const chunks = this.chunkArray(taskIds, this.batchSize);
    
    for (const chunk of chunks) {
      await this.supabase
        .from('tasks')
        .update({
          status,
          updated_at: new Date().toISOString()
        })
        .in('id', chunk);

      // Rate limit to avoid overwhelming the database
      await this.sleep(100);
    }
  }

  /**
   * Batch create tasks
   */
  async batchCreateTasks(tasks: any[]): Promise<void> {
    const chunks = this.chunkArray(tasks, this.batchSize);
    
    for (const chunk of chunks) {
      await this.supabase
        .from('tasks')
        .insert(chunk.map(task => ({
          ...task,
          task_number: null, // Will be auto-generated
          created_at: new Date().toISOString()
        })));

      await this.sleep(100);
    }
  }

  /**
   * Batch process notifications
   */
  async batchCreateNotifications(notifications: any[]): Promise<void> {
    const chunks = this.chunkArray(notifications, this.batchSize);
    
    for (const chunk of chunks) {
      await this.supabase
        .from('notifications')
        .insert(chunk);

      await this.sleep(50);
    }
  }

  /**
   * Bulk assign tasks to users
   */
  async bulkAssignTasks(taskIds: string[], userId: string): Promise<void> {
    await this.supabase
      .from('tasks')
      .update({
        assigned_to_id: userId,
        updated_at: new Date().toISOString()
      })
      .in('id', taskIds);

    // Create audit log entries in batch
    await this.batchCreateAuditLogs(
      taskIds.map(taskId => ({
        action: 'BULK_ASSIGN',
        resource_type: 'task',
        resource_id: taskId,
        change_data: { assigned_to: userId }
      }))
    );
  }

  /**
   * Bulk delete tasks
   */
  async bulkDeleteTasks(taskIds: string[]): Promise<void> {
    // First delete dependent data
    await this.supabase
      .from('task_comments')
      .delete()
      .in('task_id', taskIds);

    await this.supabase
      .from('task_attachments')
      .delete()
      .in('task_id', taskIds);

    await this.supabase
      .from('task_assignments')
      .delete()
      .in('task_id', taskIds);

    // Then delete tasks
    const chunks = this.chunkArray(taskIds, this.batchSize);
    for (const chunk of chunks) {
      await this.supabase
        .from('tasks')
        .delete()
        .in('id', chunk);
      
      await this.sleep(100);
    }
  }

  private chunkArray<T>(array: T[], size: number): T[][] {
    const chunks = [];
    for (let i = 0; i < array.length; i += size) {
      chunks.push(array.slice(i, i + size));
    }
    return chunks;
  }

  private sleep(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }

  private async batchCreateAuditLogs(logs: any[]): Promise<void> {
    const chunks = this.chunkArray(logs, this.batchSize);
    
    for (const chunk of chunks) {
      await this.supabase
        .from('audit_logs')
        .insert(chunk.map(log => ({
          ...log,
          timestamp: new Date().toISOString()
        })));

      await this.sleep(50);
    }
  }
}
```

---

## 11. TESTING STRATEGY

### 11.1 Unit Tests

```typescript
// __tests__/services/task.service.test.ts

import { TaskService } from '@/services/api/task.service';
import { MockSupabaseClient } from '@/__mocks__/supabase';

jest.mock('@/lib/supabase/client');

describe('TaskService', () => {
  let taskService: TaskService;
  let mockSupabase: any;

  beforeEach(() => {
    mockSupabase = new MockSupabaseClient();
    taskService = new TaskService();
    (taskService as any).supabase = mockSupabase;
  });

  describe('createTask', () => {
    it('should create a task with valid data', async () => {
      const input = {
        title: 'Test Task',
        description: 'This is a test task',
        priority: 'High',
        dueDate: '2024-12-31',
        categoryId: 'cat-123',
        assignedToId: 'user-456'
      };

      const result = await taskService.createTask(input);

      expect(result).toHaveProperty('id');
      expect(result.title).toBe(input.title);
      expect(result.status).toBe('Pending');
      expect(result.task_number).toMatch(/^TASK-\d{4}-\d{6}$/);
    });

    it('should throw error for invalid data', async () => {
      const invalidInput = {
        title: '',
        description: 'Test'
      };

      await expect(taskService.createTask(invalidInput))
        .rejects
        .toThrow('Task title is required');
    });

    it('should throw error for past due date', async () => {
      const input = {
        title: 'Test Task',
        dueDate: '2020-01-01',
        priority: 'Medium'
      };

      await expect(taskService.createTask(input))
        .rejects
        .toThrow('Due date cannot be in the past');
    });
  });

  describe('updateTaskStatus', () => {
    it('should update task status and calculate score on completion', async () => {
      const taskId = 'task-123';
      const mockTask = {
        id: taskId,
        status: 'Pending',
        assigned_to_id: 'user-456',
        due_date: '2024-12-31'
      };

      mockSupabase.from().select().single.mockResolvedValue({ data: mockTask });
      mockSupabase.from().update().eq().select().single.mockResolvedValue({
        data: { ...mockTask, status: 'Done / Completed' }
      });

      const result = await taskService.updateTaskStatus(taskId, 'Done / Completed');

      expect(result.status).toBe('Done / Completed');
      expect(result.completion_percentage).toBe(100);
    });

    it('should handle overdue tasks', async () => {
      const taskId = 'task-123';
      const mockTask = {
        id: taskId,
        status: 'Pending',
        due_date: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000).toISOString()
      };

      mockSupabase.from().select().single.mockResolvedValue({ data: mockTask });
      
      const result = await taskService.updateTaskStatus(taskId, 'In Progress');

      expect(result.is_overdue_days).toBeGreaterThan(0);
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

    it('should handle concerns properly', async () => {
      const taskId = 'task-123';
      const content = 'I have a concern about this task';
      
      const mockComment = {
        id: 'comment-789',
        task_id: taskId,
        user_id: 'user-123',
        comment: content,
        is_concern: true
      };

      mockSupabase.from().insert().select().single.mockResolvedValue({
        data: mockComment
      });

      const result = await taskService.addComment(taskId, content, true);

      expect(result.is_concern).toBe(true);
      expect(result.comment_type).toBe('Concern');
    });
  });
});
```

### 11.2 Integration Tests

```typescript
// __tests__/integration/task-workflow.test.ts

import { createClient } from '@supabase/supabase-js';
import { TaskService } from '@/services/api/task.service';
import { RequestService } from '@/services/api/request.service';

describe('Task Management Integration', () => {
  let supabase: any;
  let testUser: any;
  let testDepartment: any;
  let taskService: TaskService;
  let requestService: RequestService;

  beforeAll(async () => {
    // Setup test database
    supabase = createClient(
      process.env.TEST_SUPABASE_URL!,
      process.env.TEST_SUPABASE_ANON_KEY!
    );

    // Create test user
    const { data: user } = await supabase
      .from('users')
      .insert({
        email: 'test@example.com',
        first_name: 'Test',
        last_name: 'User',
        is_active: true
      })
      .select()
      .single();

    testUser = user;

    // Create test department
    const { data: dept } = await supabase
      .from('departments')
      .insert({
        name: 'Test Department',
        code: 'TEST'
      })
      .select()
      .single();

    testDepartment = dept;

    taskService = new TaskService();
    requestService = new RequestService();
  });

  afterAll(async () => {
    // Cleanup test data
    await supabase.from('tasks').delete().neq('id', '00000000-0000-0000-0000-000000000000');
    await supabase.from('users').delete().eq('id', testUser.id);
    await supabase.from('departments').delete().eq('id', testDepartment.id);
  });

  test('Full task lifecycle', async () => {
    // 1. Create task
    const taskInput = {
      title: 'Integration Test Task',
      description: 'Testing full lifecycle',
      priority: 'High',
      dueDate: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000).toISOString(),
      assignedToId: testUser.id,
      departmentId: testDepartment.id
    };

    const task = await taskService.createTask(taskInput);
    expect(task).toBeDefined();
    expect(task.status).toBe('Pending');

    // 2. Add comment
    const comment = await taskService.addComment(
      task.id,
      'Working on this task',
      false
    );
    expect(comment).toBeDefined();

    // 3. Update status
    const updated = await taskService.updateTaskStatus(task.id, 'In Progress');
    expect(updated.status).toBe('In Progress');
    expect(updated.completion_percentage).toBe(50);

    // 4. Add concern
    const concern = await taskService.addComment(
      task.id,
      'Found a critical issue with this approach',
      true
    );
    expect(concern.is_concern).toBe(true);

    // 5. Complete task
    const completed = await taskService.updateTaskStatus(task.id, 'Done / Completed');
    expect(completed.status).toBe('Done / Completed');
    expect(completed.completion_percentage).toBe(100);
    expect(completed.task_score).toBeGreaterThan(0);

    // 6. Verify audit log
    const { data: logs } = await supabase
      .from('audit_logs')
      .select('*')
      .eq('resource_type', 'task')
      .eq('resource_id', task.id);

    expect(logs.length).toBeGreaterThan(0);
  });

  test('Request approval workflow', async () => {
    // 1. Create request
    const requestInput = {
      title: 'New Equipment Request',
      description: 'We need a new color sorter',
      typeId: 'equipment-request',
      priority: 'Medium',
      departmentId: testDepartment.id,
      requestData: {
        equipmentName: 'Color Sorter',
        quantity: 1,
        estimatedCost: 50000,
        justification: 'Current unit is outdated'
      }
    };

    const request = await requestService.createRequest(requestInput);
    expect(request).toBeDefined();
    expect(request.status).toBe('Submitted');

    // 2. Get approvals
    const approvals = await requestService.getPendingApprovals(request.id);
    expect(approvals).toBeDefined();

    // 3. Approve
    if (approvals && approvals.length > 0) {
      const approval = approvals[0];
      const result = await requestService.approveRequest(approval.id, 'Approved');
      expect(result.status).toBe('Approved');
    }

    // 4. Verify completion
    const completedRequest = await requestService.getRequest(request.id);
    expect(completedRequest.status).toBe('Approved');
  });

  test('Performance tracking', async () => {
    // 1. Get user performance
    const performance = await taskService.getUserPerformance(testUser.id);
    expect(performance).toBeDefined();

    // 2. Verify scores
    expect(performance).toHaveProperty('task_completion_score');
    expect(performance).toHaveProperty('overall_score');
    expect(performance).toHaveProperty('performance_band');
  });
});
```

---

## 12. DEPLOYMENT GUIDE

### 12.1 Environment Configuration

```bash
# .env.local - Development
NEXT_PUBLIC_SUPABASE_URL=your-supabase-url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-supabase-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key

REDIS_HOST=localhost
REDIS_PORT=6379

SENDGRID_API_KEY=your-sendgrid-key
SENDGRID_FROM_EMAIL=noreply@riceworld.com

TWILIO_ACCOUNT_SID=your-twilio-sid
TWILIO_AUTH_TOKEN=your-twilio-token
TWILIO_FROM_NUMBER=+1234567890

NEXT_PUBLIC_APP_URL=http://localhost:3000

# .env.production - Production
NEXT_PUBLIC_SUPABASE_URL=your-prod-supabase-url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-prod-supabase-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-prod-service-role-key

REDIS_HOST=prod-redis-host
REDIS_PORT=6379
REDIS_PASSWORD=your-redis-password

SENDGRID_API_KEY=your-prod-sendgrid-key
SENDGRID_FROM_EMAIL=noreply@riceworld.com

TWILIO_ACCOUNT_SID=your-prod-twilio-sid
TWILIO_AUTH_TOKEN=your-prod-twilio-token
TWILIO_FROM_NUMBER=+1234567890

NEXT_PUBLIC_APP_URL=https://riceworld.com

SENTRY_DSN=your-sentry-dsn
LOGTAIL_TOKEN=your-logtail-token
```

### 12.2 Deployment Script

```json
// package.json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "test:integration": "jest --config jest.integration.config.js",
    "lint": "eslint . --ext .ts,.tsx",
    "type-check": "tsc --noEmit",
    "db:push": "supabase db push",
    "db:reset": "supabase db reset",
    "functions:deploy": "supabase functions deploy",
    "deploy": "npm run build && npm run db:push && npm run functions:deploy"
  }
}
```

### 12.3 Deployment Workflow

```yaml
# .github/workflows/deploy-task-manager.yml
name: Deploy Task Management System

on:
  push:
    branches: [main]
    paths:
      - 'src/**'
      - 'supabase/**'
  pull_request:
    branches: [main]
    paths:
      - 'src/**'
      - 'supabase/**'

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
      
      - name: Run unit tests
        run: npm run test
        env:
          NEXT_PUBLIC_SUPABASE_URL: ${{ secrets.TEST_SUPABASE_URL }}
          NEXT_PUBLIC_SUPABASE_ANON_KEY: ${{ secrets.TEST_SUPABASE_ANON_KEY }}
      
      - name: Run integration tests
        run: npm run test:integration
        env:
          TEST_SUPABASE_URL: ${{ secrets.TEST_SUPABASE_URL }}
          TEST_SUPABASE_ANON_KEY: ${{ secrets.TEST_SUPABASE_ANON_KEY }}
  
  deploy-staging:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: staging
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build
        run: npm run build
        env:
          NEXT_PUBLIC_SUPABASE_URL: ${{ secrets.STAGING_SUPABASE_URL }}
          NEXT_PUBLIC_SUPABASE_ANON_KEY: ${{ secrets.STAGING_SUPABASE_ANON_KEY }}
      
      - name: Deploy to Vercel (Staging)
        uses: amondnet/vercel-action@v20
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: '--preview'
      
      - name: Run database migrations
        run: npm run db:push
        env:
          SUPABASE_ACCESS_TOKEN: ${{ secrets.SUPABASE_ACCESS_TOKEN }}
          SUPABASE_PROJECT_REF: ${{ secrets.STAGING_PROJECT_REF }}
      
      - name: Deploy Edge Functions
        run: npm run functions:deploy
        env:
          SUPABASE_ACCESS_TOKEN: ${{ secrets.SUPABASE_ACCESS_TOKEN }}
          SUPABASE_PROJECT_REF: ${{ secrets.STAGING_PROJECT_REF }}

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: production
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build
        run: npm run build
        env:
          NEXT_PUBLIC_SUPABASE_URL: ${{ secrets.PROD_SUPABASE_URL }}
          NEXT_PUBLIC_SUPABASE_ANON_KEY: ${{ secrets.PROD_SUPABASE_ANON_KEY }}
      
      - name: Deploy to Vercel (Production)
        uses: amondnet/vercel-action@v20
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: '--prod'
      
      - name: Run database migrations
        run: npm run db:push
        env:
          SUPABASE_ACCESS_TOKEN: ${{ secrets.SUPABASE_ACCESS_TOKEN }}
          SUPABASE_PROJECT_REF: ${{ secrets.PROD_PROJECT_REF }}
      
      - name: Deploy Edge Functions
        run: npm run functions:deploy
        env:
          SUPABASE_ACCESS_TOKEN: ${{ secrets.SUPABASE_ACCESS_TOKEN }}
          SUPABASE_PROJECT_REF: ${{ secrets.PROD_PROJECT_REF }}
      
      - name: Notify deployment
        run: |
          curl -X POST -H "Content-Type: application/json" \
            -d '{"text":"✅ Task Management System deployed to production: ${{ github.sha }}"}' \
            ${{ secrets.SLACK_WEBHOOK_URL }}
```

---

## CONCLUSION

This comprehensive technical documentation for the Task Management System provides all the details needed for development, implementation, and maintenance. The system is designed to be:

1. **Scalable**: Handles thousands of tasks with efficient caching and indexing
2. **Secure**: Comprehensive RLS policies and input validation
3. **Performant**: Optimized queries, caching, and batch operations
4. **User-Friendly**: Intuitive interface with real-time updates
5. **Integrated**: Seamless integration with other ERP modules
6. **Mobile-Ready**: Full mobile SDK for on-the-go access
7. **Analytics-Powered**: Rich scoring and analytics capabilities
8. **Maintainable**: Well-structured code with comprehensive testing

The system will transform how the rice factory manages tasks, requests, and performance tracking, leading to improved efficiency, accountability, and decision-making.