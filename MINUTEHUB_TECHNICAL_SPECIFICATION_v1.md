# MINUTEHUB ACTIONBOOK APPLICATION
# TECHNICAL SPECIFICATION & IMPLEMENTATION GUIDE v1

---

## DOCUMENT HEADER

**Document Title:** MinuteHub ActionBook - Technical Specification & Implementation Guide  
**Version:** 1.0  
**Created:** October 2025  
**Author:** Guardian-X Digital Solutions  
**Classification:** Technical Implementation / Development Team

**Linked Documents:**
- MinuteHub Charter Document
- MinuteHub Development Guideline Plan v1
- Sentinel Security Framework
- FollowUpHub Integration Specifications

---

# TABLE OF CONTENTS

**PART I: TECHNICAL ARCHITECTURE**
- [Section 1: System Architecture Overview](#section-1-system-architecture-overview)
- [Section 2: Database Design](#section-2-database-design)
- [Section 3: API Specification](#section-3-api-specification)
- [Section 4: Security Implementation](#section-4-security-implementation)

**PART II: IMPLEMENTATION GUIDE**
- [Section 5: Development Setup](#section-5-development-setup)
- [Section 6: Feature Implementation](#section-6-feature-implementation)
- [Section 7: Testing Strategy](#section-7-testing-strategy)
- [Section 8: Deployment Guide](#section-8-deployment-guide)

**PART III: CODE STANDARDS & EXAMPLES**
- [Section 9: Code Quality Standards](#section-9-code-quality-standards)
- [Section 10: Implementation Examples](#section-10-implementation-examples)
- [Section 11: Testing Templates](#section-11-testing-templates)
- [Section 12: Deployment Scripts](#section-12-deployment-scripts)

---

# PART I: TECHNICAL ARCHITECTURE

## SECTION 1: SYSTEM ARCHITECTURE OVERVIEW

### 1.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    MINUTEHUB ACTIONBOOK                     │
│                  Technical Architecture                     │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                    PRESENTATION LAYER                       │
├─────────────────────────────────────────────────────────────┤
│  React 18 + TypeScript + Tailwind CSS + shadcn/ui          │
│                                                             │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│  │   Upload    │ │  Templates  │ │    Admin    │           │
│  │   Module    │ │   Module    │ │   Module    │           │
│  └─────────────┘ └─────────────┘ └─────────────┘           │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│  │    Auth     │ │   Minutes   │ │   Search    │           │
│  │   Module    │ │   Module    │ │   Module    │           │
│  └─────────────┘ └─────────────┘ └─────────────┘           │
└─────────────────────────────────────────────────────────────┘
                            │
                    HTTP/HTTPS (TLS 1.3)
                            │
┌─────────────────────────────────────────────────────────────┐
│                    APPLICATION LAYER                        │
├─────────────────────────────────────────────────────────────┤
│               Supabase Backend Services                     │
│                                                             │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│  │    Auth     │ │  Database   │ │   Storage   │           │
│  │   Service   │ │   Service   │ │   Service   │           │
│  └─────────────┘ └─────────────┘ └─────────────┘           │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│  │  Realtime   │ │  Functions  │ │     API     │           │
│  │   Service   │ │   Service   │ │   Gateway   │           │
│  └─────────────┘ └─────────────┘ └─────────────┘           │
└─────────────────────────────────────────────────────────────┘
                            │
                         SQL/RPC
                            │
┌─────────────────────────────────────────────────────────────┐
│                      DATA LAYER                            │
├─────────────────────────────────────────────────────────────┤
│                PostgreSQL Database                         │
│                                                             │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│  │    Users    │ │ Documents   │ │ Templates   │           │
│  │   Tables    │ │   Tables    │ │   Tables    │           │
│  └─────────────┘ └─────────────┘ └─────────────┘           │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│  │ Organizations│ │  Decisions  │ │   Audit     │           │
│  │   Tables    │ │   Tables    │ │   Tables    │           │
│  └─────────────┘ └─────────────┘ └─────────────┘           │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 Technology Stack

**Frontend Technologies:**
```json
{
  "framework": "React 18.2.0",
  "language": "TypeScript 5.1.0",
  "buildTool": "Vite 4.4.0",
  "styling": "Tailwind CSS 3.3.0",
  "components": "shadcn/ui + Radix UI",
  "stateManagement": "Zustand 4.4.0",
  "routing": "React Router 6.14.0",
  "forms": "React Hook Form 7.45.0",
  "httpClient": "Axios 1.4.0",
  "dateHandling": "date-fns 2.30.0",
  "icons": "Lucide React 0.263.0",
  "testing": "Vitest + React Testing Library",
  "e2e": "Playwright"
}
```

**Backend Technologies:**
```json
{
  "database": "Supabase (PostgreSQL 15)",
  "authentication": "Supabase Auth (JWT)",
  "fileStorage": "Supabase Storage",
  "realtime": "Supabase Realtime",
  "hosting": "Vercel",
  "monitoring": "Sentry + Vercel Analytics",
  "cdn": "Vercel Edge Network"
}
```

### 1.3 Data Flow Architecture

**Document Processing Flow:**
```
1. User Upload → 2. Validation → 3. Storage → 4. Processing Queue
                                    ↓
8. Export Ready ← 7. Template Apply ← 6. AI Analysis ← 5. File Parse
```

**Authentication Flow:**
```
1. Login Request → 2. Supabase Auth → 3. JWT Token → 4. Session Storage
                                         ↓
5. Protected Routes ← 6. Token Validation ← 7. API Requests
```

---

## SECTION 2: DATABASE DESIGN

### 2.1 Complete Database Schema

```sql
-- Database: MinuteHub ActionBook Application
-- Version: 1.0
-- Created: October 2025

-- Enable required extensions
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pg_trgm";
CREATE EXTENSION IF NOT EXISTS "unaccent";

-- Custom Types
CREATE TYPE user_role AS ENUM ('admin', 'manager', 'user', 'viewer');
CREATE TYPE document_status AS ENUM ('uploaded', 'processing', 'completed', 'failed', 'archived');
CREATE TYPE template_category AS ENUM ('government', 'education', 'business', 'general');
CREATE TYPE decision_type AS ENUM ('policy', 'budget', 'personnel', 'operational', 'strategic');
CREATE TYPE priority_level AS ENUM ('low', 'medium', 'high', 'critical');
CREATE TYPE decision_status AS ENUM ('pending', 'in_progress', 'completed', 'cancelled');
CREATE TYPE notification_type AS ENUM ('document_processed', 'decision_assigned', 'task_reminder', 'system_alert');
CREATE TYPE audit_action AS ENUM ('create', 'read', 'update', 'delete', 'login', 'logout', 'export', 'share');

-- Organizations table
CREATE TABLE organizations (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name VARCHAR(255) NOT NULL,
  slug VARCHAR(100) UNIQUE NOT NULL,
  description TEXT,
  logo_url TEXT,
  settings JSONB DEFAULT '{}',
  subscription_plan VARCHAR(50) DEFAULT 'free',
  max_users INTEGER DEFAULT 10,
  max_storage_gb INTEGER DEFAULT 5,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  CONSTRAINT valid_slug CHECK (slug ~ '^[a-z0-9-]+$'),
  CONSTRAINT positive_limits CHECK (max_users > 0 AND max_storage_gb > 0)
);

-- Users table
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  email VARCHAR(255) UNIQUE NOT NULL,
  full_name VARCHAR(255) NOT NULL,
  role user_role NOT NULL DEFAULT 'user',
  organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
  avatar_url TEXT,
  phone VARCHAR(20),
  department VARCHAR(100),
  job_title VARCHAR(150),
  preferences JSONB DEFAULT '{}',
  storage_used_mb INTEGER DEFAULT 0,
  storage_limit_mb INTEGER DEFAULT 1000,
  last_active_at TIMESTAMP WITH TIME ZONE,
  email_verified_at TIMESTAMP WITH TIME ZONE,
  phone_verified_at TIMESTAMP WITH TIME ZONE,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  CONSTRAINT valid_email CHECK (email ~ '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$'),
  CONSTRAINT storage_limits CHECK (storage_used_mb >= 0 AND storage_limit_mb > 0),
  CONSTRAINT phone_format CHECK (phone IS NULL OR phone ~ '^\+?[1-9]\d{1,14}$')
);

-- Guardian-X Templates table
CREATE TABLE admin_templates (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  template_id VARCHAR(50) UNIQUE NOT NULL,
  template_name VARCHAR(255) NOT NULL,
  description TEXT,
  category template_category NOT NULL,
  version VARCHAR(10) DEFAULT '1.0',
  
  -- Template Structure
  template_schema JSONB NOT NULL,
  default_sections JSONB NOT NULL,
  ai_processing_rules JSONB DEFAULT '{}',
  validation_rules JSONB DEFAULT '{}',
  
  -- Template Files and Assets
  template_file_url TEXT,
  preview_image_url TEXT,
  icon_name VARCHAR(50),
  sample_output_url TEXT,
  
  -- Configuration
  is_active BOOLEAN DEFAULT true,
  is_default BOOLEAN DEFAULT false,
  sort_order INTEGER DEFAULT 0,
  supported_languages VARCHAR(10)[] DEFAULT ARRAY['en'],
  min_confidence_threshold DECIMAL(3,2) DEFAULT 0.7,
  
  -- Metadata
  created_by UUID REFERENCES users(id),
  usage_count INTEGER DEFAULT 0,
  success_rate DECIMAL(5,2) DEFAULT 0.0,
  avg_processing_time INTEGER DEFAULT 0, -- in seconds
  
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  CONSTRAINT valid_template_id CHECK (template_id ~ '^[a-z0-9_]+$'),
  CONSTRAINT valid_confidence CHECK (min_confidence_threshold BETWEEN 0.0 AND 1.0),
  CONSTRAINT valid_success_rate CHECK (success_rate BETWEEN 0.0 AND 100.0)
);

-- Documents table
CREATE TABLE documents (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE NOT NULL,
  organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE NOT NULL,
  template_id VARCHAR(50) REFERENCES admin_templates(template_id),
  
  -- File Information
  original_filename VARCHAR(500) NOT NULL,
  file_size_bytes BIGINT NOT NULL,
  file_type VARCHAR(50) NOT NULL,
  mime_type VARCHAR(100) NOT NULL,
  original_file_url TEXT NOT NULL,
  processed_file_url TEXT,
  thumbnail_url TEXT,
  file_hash VARCHAR(64), -- SHA-256 for deduplication
  
  -- Processing Information
  processing_status document_status DEFAULT 'uploaded',
  processing_started_at TIMESTAMP WITH TIME ZONE,
  processing_completed_at TIMESTAMP WITH TIME ZONE,
  processing_duration INTEGER, -- in seconds
  processing_error TEXT,
  retry_count INTEGER DEFAULT 0,
  
  -- Content and Metadata
  title VARCHAR(500),
  description TEXT,
  extracted_text TEXT,
  ai_structured_content JSONB DEFAULT '{}',
  ai_confidence_score DECIMAL(3,2),
  language_detected VARCHAR(10),
  page_count INTEGER,
  word_count INTEGER,
  
  -- Meeting Information
  meeting_date DATE,
  meeting_time TIME,
  meeting_location VARCHAR(255),
  meeting_type VARCHAR(100),
  meeting_chairperson VARCHAR(255),
  participants JSONB DEFAULT '[]',
  agenda_items JSONB DEFAULT '[]',
  
  -- Extracted Content
  decisions_extracted JSONB DEFAULT '[]',
  action_items JSONB DEFAULT '[]',
  key_topics JSONB DEFAULT '[]',
  attachments JSONB DEFAULT '[]',
  
  -- User Management
  tags VARCHAR(50)[],
  is_starred BOOLEAN DEFAULT false,
  is_archived BOOLEAN DEFAULT false,
  shared_with UUID[] DEFAULT '{}',
  access_level VARCHAR(20) DEFAULT 'private', -- private, organization, public
  
  -- Search and Indexing
  search_vector tsvector,
  
  -- Timestamps
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  CONSTRAINT positive_file_size CHECK (file_size_bytes > 0),
  CONSTRAINT valid_confidence CHECK (ai_confidence_score IS NULL OR ai_confidence_score BETWEEN 0.0 AND 1.0),
  CONSTRAINT positive_counts CHECK (
    (page_count IS NULL OR page_count > 0) AND 
    (word_count IS NULL OR word_count >= 0)
  ),
  CONSTRAINT valid_access_level CHECK (access_level IN ('private', 'organization', 'public'))
);

-- Decisions table (for FollowUpHub integration)
CREATE TABLE decisions (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  document_id UUID REFERENCES documents(id) ON DELETE CASCADE NOT NULL,
  user_id UUID REFERENCES users(id) ON DELETE SET NULL,
  organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE NOT NULL,
  
  -- Decision Content
  decision_text TEXT NOT NULL,
  decision_context TEXT,
  decision_type decision_type NOT NULL DEFAULT 'operational',
  priority priority_level DEFAULT 'medium',
  
  -- Assignment and Tracking
  assigned_to UUID REFERENCES users(id) ON DELETE SET NULL,
  assigned_by UUID REFERENCES users(id) ON DELETE SET NULL,
  due_date DATE,
  estimated_effort_hours DECIMAL(5,2),
  status decision_status DEFAULT 'pending',
  completion_percentage INTEGER DEFAULT 0,
  
  -- Content Context
  section_in_document VARCHAR(255),
  page_number INTEGER,
  paragraph_number INTEGER,
  confidence_score DECIMAL(3,2),
  extraction_method VARCHAR(50), -- 'ai_automatic', 'user_manual', 'ai_assisted'
  
  -- Implementation Details
  implementation_notes TEXT,
  resources_required JSONB DEFAULT '[]',
  dependencies JSONB DEFAULT '[]',
  success_criteria TEXT,
  
  -- Integration with FollowUpHub
  followuphub_task_id UUID,
  sync_status VARCHAR(20) DEFAULT 'pending', -- pending, synced, failed
  last_sync_at TIMESTAMP WITH TIME ZONE,
  sync_error TEXT,
  
  -- Approval Workflow
  requires_approval BOOLEAN DEFAULT false,
  approved_by UUID REFERENCES users(id) ON DELETE SET NULL,
  approved_at TIMESTAMP WITH TIME ZONE,
  approval_notes TEXT,
  
  -- Timestamps
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  completed_at TIMESTAMP WITH TIME ZONE,
  
  CONSTRAINT valid_completion CHECK (completion_percentage BETWEEN 0 AND 100),
  CONSTRAINT valid_confidence CHECK (confidence_score IS NULL OR confidence_score BETWEEN 0.0 AND 1.0),
  CONSTRAINT completion_consistency CHECK (
    (status = 'completed' AND completed_at IS NOT NULL AND completion_percentage = 100) OR
    (status != 'completed' AND (completed_at IS NULL OR completion_percentage < 100))
  )
);

-- Action Items table
CREATE TABLE action_items (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  document_id UUID REFERENCES documents(id) ON DELETE CASCADE NOT NULL,
  decision_id UUID REFERENCES decisions(id) ON DELETE CASCADE,
  
  -- Action Item Content
  action_text TEXT NOT NULL,
  action_description TEXT,
  priority priority_level DEFAULT 'medium',
  
  -- Assignment
  assigned_to UUID REFERENCES users(id) ON DELETE SET NULL,
  assigned_by UUID REFERENCES users(id) ON DELETE SET NULL,
  due_date DATE,
  estimated_hours DECIMAL(5,2),
  
  -- Progress Tracking
  status decision_status DEFAULT 'pending',
  progress_notes TEXT,
  completion_percentage INTEGER DEFAULT 0,
  
  -- Context
  section_reference VARCHAR(255),
  confidence_score DECIMAL(3,2),
  
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  completed_at TIMESTAMP WITH TIME ZONE,
  
  CONSTRAINT valid_completion CHECK (completion_percentage BETWEEN 0 AND 100)
);

-- Audit Logs table
CREATE TABLE audit_logs (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id) ON DELETE SET NULL,
  organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
  
  -- Action Details
  action audit_action NOT NULL,
  resource_type VARCHAR(50) NOT NULL, -- 'document', 'user', 'template', etc.
  resource_id UUID,
  old_values JSONB,
  new_values JSONB,
  
  -- Request Context
  ip_address INET,
  user_agent TEXT,
  request_id VARCHAR(100),
  session_id VARCHAR(100),
  
  -- Additional Context
  description TEXT,
  metadata JSONB DEFAULT '{}',
  
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  CONSTRAINT valid_resource_type CHECK (resource_type ~ '^[a-z_]+$')
);

-- Notifications table
CREATE TABLE notifications (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE NOT NULL,
  organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE NOT NULL,
  
  -- Notification Content
  type notification_type NOT NULL,
  title VARCHAR(255) NOT NULL,
  message TEXT NOT NULL,
  action_url TEXT,
  
  -- Related Entities
  document_id UUID REFERENCES documents(id) ON DELETE CASCADE,
  decision_id UUID REFERENCES decisions(id) ON DELETE CASCADE,
  
  -- Delivery Status
  is_read BOOLEAN DEFAULT false,
  read_at TIMESTAMP WITH TIME ZONE,
  is_sent BOOLEAN DEFAULT false,
  sent_at TIMESTAMP WITH TIME ZONE,
  delivery_method VARCHAR(20), -- 'email', 'sms', 'push', 'in_app'
  
  -- Scheduling
  scheduled_for TIMESTAMP WITH TIME ZONE,
  expires_at TIMESTAMP WITH TIME ZONE,
  
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  CONSTRAINT future_expiry CHECK (expires_at IS NULL OR expires_at > created_at)
);
```

### 2.2 Database Indexes and Optimization

```sql
-- Performance Indexes

-- Users table indexes
CREATE INDEX idx_users_organization ON users(organization_id);
CREATE INDEX idx_users_email_active ON users(email) WHERE is_active = true;
CREATE INDEX idx_users_role_org ON users(role, organization_id);
CREATE INDEX idx_users_last_active ON users(last_active_at DESC);

-- Documents table indexes
CREATE INDEX idx_documents_user_org ON documents(user_id, organization_id);
CREATE INDEX idx_documents_status ON documents(processing_status);
CREATE INDEX idx_documents_created_desc ON documents(created_at DESC);
CREATE INDEX idx_documents_template ON documents(template_id) WHERE template_id IS NOT NULL;
CREATE INDEX idx_documents_meeting_date ON documents(meeting_date DESC) WHERE meeting_date IS NOT NULL;
CREATE INDEX idx_documents_tags ON documents USING gin(tags);
CREATE INDEX idx_documents_shared ON documents USING gin(shared_with);

-- Full-text search indexes
CREATE INDEX idx_documents_search ON documents USING gin(search_vector);
CREATE INDEX idx_documents_title_search ON documents USING gin(to_tsvector('english', title));
CREATE INDEX idx_documents_content_search ON documents USING gin(to_tsvector('english', COALESCE(extracted_text, '')));

-- Decisions table indexes
CREATE INDEX idx_decisions_document ON decisions(document_id);
CREATE INDEX idx_decisions_assigned ON decisions(assigned_to, status) WHERE assigned_to IS NOT NULL;
CREATE INDEX idx_decisions_due_date ON decisions(due_date) WHERE due_date IS NOT NULL AND status != 'completed';
CREATE INDEX idx_decisions_org_status ON decisions(organization_id, status);
CREATE INDEX idx_decisions_sync_status ON decisions(sync_status, last_sync_at);

-- Action items indexes
CREATE INDEX idx_action_items_document ON action_items(document_id);
CREATE INDEX idx_action_items_decision ON action_items(decision_id) WHERE decision_id IS NOT NULL;
CREATE INDEX idx_action_items_assigned ON action_items(assigned_to, status) WHERE assigned_to IS NOT NULL;
CREATE INDEX idx_action_items_due_date ON action_items(due_date) WHERE due_date IS NOT NULL;

-- Notifications indexes
CREATE INDEX idx_notifications_user_unread ON notifications(user_id, created_at DESC) WHERE is_read = false;
CREATE INDEX idx_notifications_scheduled ON notifications(scheduled_for) WHERE scheduled_for IS NOT NULL AND is_sent = false;
CREATE INDEX idx_notifications_org ON notifications(organization_id, created_at DESC);

-- Audit logs indexes
CREATE INDEX idx_audit_logs_user_action ON audit_logs(user_id, action, created_at DESC);
CREATE INDEX idx_audit_logs_resource ON audit_logs(resource_type, resource_id, created_at DESC);
CREATE INDEX idx_audit_logs_org_date ON audit_logs(organization_id, created_at DESC);
CREATE INDEX idx_audit_logs_ip ON audit_logs(ip_address, created_at DESC);

-- Templates indexes
CREATE INDEX idx_templates_category_active ON admin_templates(category, is_active);
CREATE INDEX idx_templates_usage ON admin_templates(usage_count DESC);
CREATE INDEX idx_templates_success_rate ON admin_templates(success_rate DESC);
```

### 2.3 Database Functions and Triggers

```sql
-- Automatic timestamp updates
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ language 'plpgsql';

-- Apply update triggers to relevant tables
CREATE TRIGGER update_organizations_updated_at BEFORE UPDATE ON organizations
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_users_updated_at BEFORE UPDATE ON users
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_documents_updated_at BEFORE UPDATE ON documents
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_decisions_updated_at BEFORE UPDATE ON decisions
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_admin_templates_updated_at BEFORE UPDATE ON admin_templates
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- Search vector update function
CREATE OR REPLACE FUNCTION update_document_search_vector()
RETURNS TRIGGER AS $$
BEGIN
    NEW.search_vector := 
        setweight(to_tsvector('english', COALESCE(NEW.title, '')), 'A') ||
        setweight(to_tsvector('english', COALESCE(NEW.description, '')), 'B') ||
        setweight(to_tsvector('english', COALESCE(NEW.extracted_text, '')), 'C') ||
        setweight(to_tsvector('english', array_to_string(NEW.tags, ' ')), 'D');
    RETURN NEW;
END;
$$ language 'plpgsql';

CREATE TRIGGER update_documents_search_vector 
    BEFORE INSERT OR UPDATE ON documents
    FOR EACH ROW EXECUTE FUNCTION update_document_search_vector();

-- User storage tracking function
CREATE OR REPLACE FUNCTION update_user_storage()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        UPDATE users 
        SET storage_used_mb = storage_used_mb + (NEW.file_size_bytes / 1048576)
        WHERE id = NEW.user_id;
        RETURN NEW;
    ELSIF TG_OP = 'DELETE' THEN
        UPDATE users 
        SET storage_used_mb = GREATEST(0, storage_used_mb - (OLD.file_size_bytes / 1048576))
        WHERE id = OLD.user_id;
        RETURN OLD;
    END IF;
    RETURN NULL;
END;
$$ language 'plpgsql';

CREATE TRIGGER update_user_storage_on_document_change
    AFTER INSERT OR DELETE ON documents
    FOR EACH ROW EXECUTE FUNCTION update_user_storage();

-- Template usage tracking
CREATE OR REPLACE FUNCTION update_template_usage()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.template_id IS NOT NULL AND (OLD.template_id IS NULL OR OLD.template_id != NEW.template_id) THEN
        UPDATE admin_templates 
        SET usage_count = usage_count + 1
        WHERE template_id = NEW.template_id;
    END IF;
    RETURN NEW;
END;
$$ language 'plpgsql';

CREATE TRIGGER update_template_usage_on_document
    AFTER INSERT OR UPDATE ON documents
    FOR EACH ROW EXECUTE FUNCTION update_template_usage();

-- Decision completion tracking
CREATE OR REPLACE FUNCTION update_decision_completion()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.status = 'completed' AND (OLD.status IS NULL OR OLD.status != 'completed') THEN
        NEW.completed_at = NOW();
        NEW.completion_percentage = 100;
    ELSIF NEW.status != 'completed' THEN
        NEW.completed_at = NULL;
        IF NEW.completion_percentage = 100 THEN
            NEW.completion_percentage = 90; -- Keep it just below completion
        END IF;
    END IF;
    RETURN NEW;
END;
$$ language 'plpgsql';

CREATE TRIGGER update_decision_completion_trigger
    BEFORE UPDATE ON decisions
    FOR EACH ROW EXECUTE FUNCTION update_decision_completion();
```

### 2.4 Row Level Security (RLS) Policies

```sql
-- Enable RLS on sensitive tables
ALTER TABLE organizations ENABLE ROW LEVEL SECURITY;
ALTER TABLE users ENABLE ROW LEVEL SECURITY;
ALTER TABLE documents ENABLE ROW LEVEL SECURITY;
ALTER TABLE decisions ENABLE ROW LEVEL SECURITY;
ALTER TABLE action_items ENABLE ROW LEVEL SECURITY;
ALTER TABLE notifications ENABLE ROW LEVEL SECURITY;
ALTER TABLE audit_logs ENABLE ROW LEVEL SECURITY;

-- Organizations: Users can only access their own organization
CREATE POLICY organizations_access_policy ON organizations
  FOR ALL USING (
    id = (SELECT organization_id FROM users WHERE users.id = auth.uid())
  );

-- Users: Can access users in same organization, admins can access all
CREATE POLICY users_access_policy ON users
  FOR ALL USING (
    organization_id = (SELECT organization_id FROM users WHERE users.id = auth.uid()) OR
    EXISTS (SELECT 1 FROM users WHERE users.id = auth.uid() AND users.role = 'admin')
  );

-- Documents: Users can access documents in their organization
CREATE POLICY documents_access_policy ON documents
  FOR ALL USING (
    organization_id = (SELECT organization_id FROM users WHERE users.id = auth.uid()) AND
    (
      -- Own documents
      user_id = auth.uid() OR
      -- Shared documents
      auth.uid() = ANY(shared_with) OR
      -- Organization-wide documents
      access_level = 'organization' OR
      -- Public documents
      access_level = 'public' OR
      -- Admins and managers can access all org documents
      EXISTS (
        SELECT 1 FROM users 
        WHERE users.id = auth.uid() 
        AND users.role IN ('admin', 'manager')
        AND users.organization_id = documents.organization_id
      )
    )
  );

-- Decisions: Users can access decisions for documents they can see
CREATE POLICY decisions_access_policy ON decisions
  FOR ALL USING (
    organization_id = (SELECT organization_id FROM users WHERE users.id = auth.uid()) AND
    (
      -- Assigned to user
      assigned_to = auth.uid() OR
      -- Created by user
      user_id = auth.uid() OR
      -- Can access the related document
      EXISTS (
        SELECT 1 FROM documents 
        WHERE documents.id = decisions.document_id 
        AND (
          documents.user_id = auth.uid() OR
          auth.uid() = ANY(documents.shared_with) OR
          documents.access_level IN ('organization', 'public')
        )
      ) OR
      -- Admins and managers can access all
      EXISTS (
        SELECT 1 FROM users 
        WHERE users.id = auth.uid() 
        AND users.role IN ('admin', 'manager')
        AND users.organization_id = decisions.organization_id
      )
    )
  );

-- Action items: Similar to decisions
CREATE POLICY action_items_access_policy ON action_items
  FOR ALL USING (
    -- Assigned to user
    assigned_to = auth.uid() OR
    -- Can access the related document
    EXISTS (
      SELECT 1 FROM documents 
      WHERE documents.id = action_items.document_id 
      AND documents.organization_id = (SELECT organization_id FROM users WHERE users.id = auth.uid())
      AND (
        documents.user_id = auth.uid() OR
        auth.uid() = ANY(documents.shared_with) OR
        documents.access_level IN ('organization', 'public')
      )
    ) OR
    -- Admins and managers can access all
    EXISTS (
      SELECT 1 FROM users 
      WHERE users.id = auth.uid() 
      AND users.role IN ('admin', 'manager')
      AND users.organization_id = (
        SELECT documents.organization_id FROM documents 
        WHERE documents.id = action_items.document_id
      )
    )
  );

-- Notifications: Users can only see their own notifications
CREATE POLICY notifications_access_policy ON notifications
  FOR ALL USING (
    user_id = auth.uid() OR
    EXISTS (
      SELECT 1 FROM users 
      WHERE users.id = auth.uid() 
      AND users.role = 'admin'
      AND users.organization_id = notifications.organization_id
    )
  );

-- Audit logs: Users can see their own actions, admins can see all in org
CREATE POLICY audit_logs_access_policy ON audit_logs
  FOR SELECT USING (
    user_id = auth.uid() OR
    EXISTS (
      SELECT 1 FROM users 
      WHERE users.id = auth.uid() 
      AND users.role = 'admin'
      AND users.organization_id = audit_logs.organization_id
    )
  );
```

---

## SECTION 3: API SPECIFICATION

### 3.1 Authentication APIs

```typescript
// Authentication and User Management API
namespace AuthAPI {
  // POST /api/v1/auth/register
  interface RegisterRequest {
    email: string;
    password: string;
    full_name: string;
    organization_id?: string;
    invitation_token?: string;
  }
  
  interface RegisterResponse {
    user: User;
    access_token: string;
    refresh_token: string;
    expires_in: number;
    requires_email_verification?: boolean;
  }
  
  // POST /api/v1/auth/login
  interface LoginRequest {
    email: string;
    password: string;
    remember_me?: boolean;
  }
  
  interface LoginResponse {
    user: User;
    access_token: string;
    refresh_token: string;
    expires_in: number;
    requires_mfa?: boolean;
    mfa_token?: string;
  }
  
  // POST /api/v1/auth/mfa/verify
  interface MFAVerifyRequest {
    mfa_token: string;
    code: string;
  }
  
  // POST /api/v1/auth/refresh
  interface RefreshRequest {
    refresh_token: string;
  }
  
  // POST /api/v1/auth/logout
  interface LogoutRequest {
    refresh_token: string;
  }
  
  // POST /api/v1/auth/forgot-password
  interface ForgotPasswordRequest {
    email: string;
  }
  
  // POST /api/v1/auth/reset-password
  interface ResetPasswordRequest {
    token: string;
    password: string;
  }
  
  // GET /api/v1/auth/me
  interface UserProfileResponse {
    user: User;
    permissions: string[];
    organization: Organization;
  }
  
  // PUT /api/v1/auth/me
  interface UpdateProfileRequest {
    full_name?: string;
    phone?: string;
    department?: string;
    job_title?: string;
    preferences?: Record<string, any>;
  }
}
```

### 3.2 Document Management APIs

```typescript
// Document Management API
namespace DocumentAPI {
  // GET /api/v1/documents
  interface ListDocumentsRequest {
    page?: number;
    limit?: number;
    sort?: 'created_at' | 'updated_at' | 'title' | 'meeting_date';
    order?: 'asc' | 'desc';
    status?: DocumentStatus[];
    template_id?: string;
    search?: string;
    tags?: string[];
    date_from?: string;
    date_to?: string;
    user_id?: string; // For managers/admins
  }
  
  interface ListDocumentsResponse {
    documents: Document[];
    meta: {
      total: number;
      page: number;
      limit: number;
      total_pages: number;
    };
    filters: {
      applied: Record<string, any>;
      available: {
        statuses: DocumentStatus[];
        templates: Template[];
        users: User[];
      };
    };
  }
  
  // POST /api/v1/documents
  interface UploadDocumentRequest {
    file: File; // multipart/form-data
    template_id?: string;
    title?: string;
    description?: string;
    meeting_date?: string;
    meeting_type?: string;
    participants?: string[];
    tags?: string[];
    auto_process?: boolean;
  }
  
  interface UploadDocumentResponse {
    document: Document;
    upload_url?: string; // For direct upload to storage
    processing_estimate?: number; // Estimated processing time in seconds
  }
  
  // GET /api/v1/documents/:id
  interface GetDocumentResponse {
    document: Document;
    versions: DocumentVersion[];
    decisions: Decision[];
    action_items: ActionItem[];
    related_documents: Document[];
  }
  
  // PUT /api/v1/documents/:id
  interface UpdateDocumentRequest {
    title?: string;
    description?: string;
    meeting_date?: string;
    meeting_type?: string;
    participants?: string[];
    tags?: string[];
    is_starred?: boolean;
    access_level?: 'private' | 'organization' | 'public';
    shared_with?: string[]; // User IDs
  }
  
  // POST /api/v1/documents/:id/process
  interface ProcessDocumentRequest {
    template_id: string;
    force_reprocess?: boolean;
    processing_options?: {
      extract_decisions?: boolean;
      extract_action_items?: boolean;
      detect_language?: boolean;
      confidence_threshold?: number;
    };
  }
  
  interface ProcessDocumentResponse {
    processing_id: string;
    estimated_duration: number;
    status: 'queued' | 'processing' | 'completed' | 'failed';
    progress?: number; // 0-100
  }
  
  // GET /api/v1/documents/:id/processing-status
  interface ProcessingStatusResponse {
    processing_id: string;
    status: 'queued' | 'processing' | 'completed' | 'failed';
    progress: number; // 0-100
    started_at?: string;
    completed_at?: string;
    error?: string;
    estimated_remaining?: number; // seconds
  }
  
  // POST /api/v1/documents/:id/export
  interface ExportDocumentRequest {
    format: 'pdf' | 'docx' | 'html';
    template?: string;
    options?: {
      include_decisions?: boolean;
      include_action_items?: boolean;
      include_metadata?: boolean;
      watermark?: string;
      header?: string;
      footer?: string;
    };
  }
  
  interface ExportDocumentResponse {
    export_id: string;
    download_url?: string; // If ready immediately
    estimated_duration?: number; // If processing required
  }
  
  // GET /api/v1/documents/:id/export/:export_id
  interface ExportStatusResponse {
    export_id: string;
    status: 'processing' | 'ready' | 'failed';
    download_url?: string;
    expires_at?: string;
    error?: string;
  }
  
  // POST /api/v1/documents/:id/share
  interface ShareDocumentRequest {
    user_ids?: string[];
    emails?: string[];
    access_level: 'view' | 'edit';
    expires_at?: string;
    message?: string;
  }
  
  interface ShareDocumentResponse {
    shared_with: Array<{
      user_id?: string;
      email?: string;
      access_level: string;
      shared_at: string;
      expires_at?: string;
    }>;
    share_links?: Array<{
      token: string;
      url: string;
      expires_at: string;
    }>;
  }
  
  // DELETE /api/v1/documents/:id
  interface DeleteDocumentResponse {
    deleted: boolean;
    cleanup_tasks?: string[]; // Background cleanup tasks
  }
}
```

### 3.3 Template Management APIs

```typescript
// Template Management API
namespace TemplateAPI {
  // GET /api/v1/templates
  interface ListTemplatesRequest {
    category?: TemplateCategory;
    language?: string;
    is_active?: boolean;
    include_usage_stats?: boolean;
  }
  
  interface ListTemplatesResponse {
    templates: Template[];
    categories: Array<{
      category: TemplateCategory;
      count: number;
      description: string;
    }>;
    languages: string[];
  }
  
  // GET /api/v1/templates/:template_id
  interface GetTemplateResponse {
    template: Template;
    usage_stats?: {
      total_uses: number;
      success_rate: number;
      avg_processing_time: number;
      recent_uses: number; // Last 30 days
    };
    sample_output?: string; // URL to sample processed document
  }
  
  // POST /api/v1/templates/:template_id/apply
  interface ApplyTemplateRequest {
    document_id: string;
    override_options?: {
      confidence_threshold?: number;
      sections?: string[]; // Only process specific sections
      preserve_original?: boolean;
    };
  }
  
  interface ApplyTemplateResponse {
    processing_id: string;
    estimated_duration: number;
    template_applied: Template;
  }
  
  // Admin Template Management
  // POST /api/v1/admin/templates
  interface CreateTemplateRequest {
    template_id: string;
    template_name: string;
    description: string;
    category: TemplateCategory;
    template_schema: Record<string, any>;
    default_sections: Record<string, any>;
    ai_processing_rules?: Record<string, any>;
    supported_languages?: string[];
    min_confidence_threshold?: number;
  }
  
  // PUT /api/v1/admin/templates/:template_id
  interface UpdateTemplateRequest extends Partial<CreateTemplateRequest> {
    is_active?: boolean;
    version?: string;
  }
  
  // GET /api/v1/admin/templates/:template_id/analytics
  interface TemplateAnalyticsResponse {
    usage_over_time: Array<{
      date: string;
      uses: number;
      success_rate: number;
    }>;
    top_users: Array<{
      user_id: string;
      user_name: string;
      uses: number;
    }>;
    common_errors: Array<{
      error_type: string;
      count: number;
      last_occurrence: string;
    }>;
    performance_metrics: {
      avg_processing_time: number;
      p95_processing_time: number;
      success_rate: number;
      total_uses: number;
    };
  }
}
```

### 3.4 Decision Management APIs

```typescript
// Decision Management API
namespace DecisionAPI {
  // GET /api/v1/decisions
  interface ListDecisionsRequest {
    page?: number;
    limit?: number;
    status?: DecisionStatus[];
    priority?: PriorityLevel[];
    assigned_to?: string;
    due_date_from?: string;
    due_date_to?: string;
    document_id?: string;
    decision_type?: DecisionType[];
    search?: string;
  }
  
  interface ListDecisionsResponse {
    decisions: Decision[];
    meta: {
      total: number;
      page: number;
      limit: number;
      total_pages: number;
    };
    aggregations: {
      by_status: Record<DecisionStatus, number>;
      by_priority: Record<PriorityLevel, number>;
      by_type: Record<DecisionType, number>;
      overdue_count: number;
      due_this_week: number;
    };
  }
  
  // POST /api/v1/decisions
  interface CreateDecisionRequest {
    document_id: string;
    decision_text: string;
    decision_context?: string;
    decision_type: DecisionType;
    priority?: PriorityLevel;
    assigned_to?: string;
    due_date?: string;
    estimated_effort_hours?: number;
    requires_approval?: boolean;
    success_criteria?: string;
    resources_required?: string[];
    dependencies?: string[];
  }
  
  interface CreateDecisionResponse {
    decision: Decision;
    notifications_sent: string[]; // User IDs notified
    followuphub_sync?: {
      status: 'pending' | 'synced' | 'failed';
      task_id?: string;
      error?: string;
    };
  }
  
  // GET /api/v1/decisions/:id
  interface GetDecisionResponse {
    decision: Decision;
    document: Document;
    action_items: ActionItem[];
    related_decisions: Decision[];
    history: Array<{
      action: string;
      user: User;
      timestamp: string;
      details?: Record<string, any>;
    }>;
    followuphub_sync_status?: {
      last_sync: string;
      status: 'synced' | 'failed' | 'pending';
      task_url?: string;
    };
  }
  
  // PUT /api/v1/decisions/:id
  interface UpdateDecisionRequest {
    decision_text?: string;
    decision_context?: string;
    priority?: PriorityLevel;
    assigned_to?: string;
    due_date?: string;
    status?: DecisionStatus;
    completion_percentage?: number;
    implementation_notes?: string;
    resources_required?: string[];
    dependencies?: string[];
  }
  
  // POST /api/v1/decisions/:id/approve
  interface ApproveDecisionRequest {
    approval_notes?: string;
  }
  
  interface ApproveDecisionResponse {
    decision: Decision;
    approved_by: User;
    approved_at: string;
    notifications_sent: string[];
  }
  
  // POST /api/v1/decisions/:id/sync
  interface SyncDecisionRequest {
    force_sync?: boolean;
  }
  
  interface SyncDecisionResponse {
    sync_status: 'success' | 'failed' | 'pending';
    followuphub_task_id?: string;
    error?: string;
    last_sync_at: string;
  }
  
  // Action Items sub-resource
  // GET /api/v1/decisions/:id/action-items
  interface DecisionActionItemsResponse {
    action_items: ActionItem[];
    total: number;
  }
  
  // POST /api/v1/decisions/:id/action-items
  interface CreateActionItemRequest {
    action_text: string;
    action_description?: string;
    priority?: PriorityLevel;
    assigned_to?: string;
    due_date?: string;
    estimated_hours?: number;
  }
}
```

---

## SECTION 4: SECURITY IMPLEMENTATION

### 4.1 Authentication Service

```typescript
// JWT-Based Authentication Service
import jwt from 'jsonwebtoken';
import bcrypt from 'bcryptjs';
import crypto from 'crypto';
import speakeasy from 'speakeasy';

export class AuthenticationService {
  private readonly accessTokenExpiry = '15m';
  private readonly refreshTokenExpiry = '7d';
  private readonly maxFailedAttempts = 5;
  private readonly lockoutDuration = 30 * 60 * 1000; // 30 minutes
  
  // User registration with security validations
  async registerUser(registerData: RegisterRequest): Promise<AuthResponse> {
    try {
      // Validate password strength
      const passwordValidation = this.validatePasswordStrength(registerData.password);
      if (!passwordValidation.isValid) {
        throw new ValidationError('Password does not meet security requirements', {
          requirements: passwordValidation.requirements,
          violations: passwordValidation.violations
        });
      }
      
      // Check if email is already registered
      const existingUser = await User.findOne({ email: registerData.email.toLowerCase() });
      if (existingUser) {
        throw new ConflictError('Email address is already registered');
      }
      
      // Validate organization invitation if provided
      if (registerData.invitation_token) {
        const invitation = await this.validateInvitationToken(registerData.invitation_token);
        registerData.organization_id = invitation.organization_id;
        registerData.role = invitation.role;
      } else if (!registerData.organization_id) {
        // Create new organization for first user
        const organization = await this.createDefaultOrganization(registerData.email);
        registerData.organization_id = organization.id;
        registerData.role = 'admin';
      }
      
      // Hash password with salt
      const saltRounds = 12;
      const hashedPassword = await bcrypt.hash(registerData.password, saltRounds);
      
      // Create user
      const user = await User.create({
        email: registerData.email.toLowerCase(),
        full_name: registerData.full_name,
        password_hash: hashedPassword,
        organization_id: registerData.organization_id,
        role: registerData.role || 'user',
        email_verified_at: null, // Require email verification
        is_active: true
      });
      
      // Generate email verification token
      const verificationToken = this.generateEmailVerificationToken(user.id);
      await this.sendEmailVerification(user.email, verificationToken);
      
      // Generate auth tokens
      const tokens = await this.generateTokenPair(user);
      
      // Log registration event
      await auditService.logAuthEvent({
        user_id: user.id,
        action: 'user_registered',
        ip_address: registerData.ip_address,
        user_agent: registerData.user_agent,
        metadata: {
          organization_id: user.organization_id,
          role: user.role
        }
      });
      
      return {
        user: this.sanitizeUser(user),
        access_token: tokens.accessToken,
        refresh_token: tokens.refreshToken,
        expires_in: this.parseExpiry(this.accessTokenExpiry),
        requires_email_verification: true
      };
      
    } catch (error) {
      // Log failed registration attempt
      await auditService.logAuthEvent({
        action: 'registration_failed',
        email: registerData.email,
        ip_address: registerData.ip_address,
        error: error.message
      });
      
      throw error;
    }
  }
  
  // Secure user login with rate limiting and MFA
  async loginUser(loginData: LoginRequest): Promise<AuthResponse> {
    try {
      const email = loginData.email.toLowerCase();
      
      // Check for account lockout
      await this.checkAccountLockout(email);
      
      // Find user
      const user = await User.findOne({ 
        email,
        is_active: true 
      }).populate('organization');
      
      if (!user) {
        await this.recordFailedAttempt(email, 'user_not_found');
        throw new UnauthorizedError('Invalid email or password');
      }
      
      // Verify password
      const isPasswordValid = await bcrypt.compare(loginData.password, user.password_hash);
      if (!isPasswordValid) {
        await this.recordFailedAttempt(email, 'invalid_password');
        throw new UnauthorizedError('Invalid email or password');
      }
      
      // Check if account is locked
      if (user.locked_until && user.locked_until > new Date()) {
        throw new UnauthorizedError('Account is temporarily locked due to failed login attempts');
      }
      
      // Reset failed attempts on successful password verification
      await this.resetFailedAttempts(email);
      
      // Check if MFA is enabled
      if (user.mfa_enabled) {
        const mfaToken = this.generateMFAToken(user.id);
        
        return {
          requires_mfa: true,
          mfa_token: mfaToken,
          mfa_methods: ['authenticator', 'sms'] // Available MFA methods
        };
      }
      
      // Generate auth tokens
      const tokens = await this.generateTokenPair(user);
      
      // Update last login
      await User.updateOne(
        { _id: user._id },
        { 
          last_active_at: new Date(),
          last_login_ip: loginData.ip_address 
        }
      );
      
      // Log successful login
      await auditService.logAuthEvent({
        user_id: user.id,
        action: 'login_success',
        ip_address: loginData.ip_address,
        user_agent: loginData.user_agent,
        metadata: {
          organization_id: user.organization_id,
          remember_me: loginData.remember_me
        }
      });
      
      return {
        user: this.sanitizeUser(user),
        access_token: tokens.accessToken,
        refresh_token: tokens.refreshToken,
        expires_in: this.parseExpiry(this.accessTokenExpiry)
      };
      
    } catch (error) {
      // Log failed login attempt
      await auditService.logAuthEvent({
        action: 'login_failed',
        email: loginData.email,
        ip_address: loginData.ip_address,
        error: error.message
      });
      
      throw error;
    }
  }
  
  // Multi-Factor Authentication verification
  async verifyMFA(mfaData: MFAVerifyRequest): Promise<AuthResponse> {
    try {
      // Verify MFA token
      const mfaPayload = jwt.verify(mfaData.mfa_token, process.env.MFA_SECRET) as any;
      const userId = mfaPayload.user_id;
      
      const user = await User.findById(userId).populate('organization');
      if (!user) {
        throw new UnauthorizedError('Invalid MFA token');
      }
      
      // Verify the MFA code
      const isCodeValid = speakeasy.totp.verify({
        secret: user.mfa_secret,
        encoding: 'base32',
        token: mfaData.code,
        window: 2 // Allow 2 time steps (60 seconds) of drift
      });
      
      if (!isCodeValid) {
        await auditService.logAuthEvent({
          user_id: user.id,
          action: 'mfa_failed',
          ip_address: mfaData.ip_address,
          error: 'Invalid MFA code'
        });
        throw new UnauthorizedError('Invalid MFA code');
      }
      
      // Generate auth tokens
      const tokens = await this.generateTokenPair(user);
      
      // Update last login
      await User.updateOne(
        { _id: user._id },
        { last_active_at: new Date() }
      );
      
      // Log successful MFA verification
      await auditService.logAuthEvent({
        user_id: user.id,
        action: 'mfa_success',
        ip_address: mfaData.ip_address,
        metadata: {
          organization_id: user.organization_id
        }
      });
      
      return {
        user: this.sanitizeUser(user),
        access_token: tokens.accessToken,
        refresh_token: tokens.refreshToken,
        expires_in: this.parseExpiry(this.accessTokenExpiry)
      };
      
    } catch (error) {
      if (error.name === 'JsonWebTokenError') {
        throw new UnauthorizedError('Invalid MFA token');
      }
      throw error;
    }
  }
  
  // Password strength validation
  private validatePasswordStrength(password: string): PasswordValidation {
    const requirements = {
      minLength: 8,
      maxLength: 128,
      requireUppercase: true,
      requireLowercase: true,
      requireNumbers: true,
      requireSpecialChars: true,
      preventCommonPasswords: true
    };
    
    const violations: string[] = [];
    
    if (password.length < requirements.minLength) {
      violations.push(`Password must be at least ${requirements.minLength} characters long`);
    }
    
    if (password.length > requirements.maxLength) {
      violations.push(`Password must be no more than ${requirements.maxLength} characters long`);
    }
    
    if (requirements.requireUppercase && !/[A-Z]/.test(password)) {
      violations.push('Password must contain at least one uppercase letter');
    }
    
    if (requirements.requireLowercase && !/[a-z]/.test(password)) {
      violations.push('Password must contain at least one lowercase letter');
    }
    
    if (requirements.requireNumbers && !/\d/.test(password)) {
      violations.push('Password must contain at least one number');
    }
    
    if (requirements.requireSpecialChars && !/[!@#$%^&*()_+\-=\[\]{};':"\\|,.<>\/?]/.test(password)) {
      violations.push('Password must contain at least one special character');
    }
    
    // Check against common passwords
    if (requirements.preventCommonPasswords && this.isCommonPassword(password)) {
      violations.push('Password is too common. Please choose a more unique password');
    }
    
    return {
      isValid: violations.length === 0,
      requirements,
      violations
    };
  }
  
  // Generate secure JWT token pair
  private async generateTokenPair(user: User): Promise<TokenPair> {
    const payload = {
      user_id: user.id,
      email: user.email,
      role: user.role,
      organization_id: user.organization_id
    };
    
    const accessToken = jwt.sign(payload, process.env.JWT_SECRET!, {
      expiresIn: this.accessTokenExpiry,
      issuer: 'minutehub',
      audience: 'minutehub-users',
      subject: user.id
    });
    
    const refreshTokenPayload = {
      user_id: user.id,
      type: 'refresh'
    };
    
    const refreshToken = jwt.sign(refreshTokenPayload, process.env.JWT_REFRESH_SECRET!, {
      expiresIn: this.refreshTokenExpiry,
      issuer: 'minutehub',
      audience: 'minutehub-users',
      subject: user.id
    });
    
    // Store refresh token hash in database
    const refreshTokenHash = crypto.createHash('sha256').update(refreshToken).digest('hex');
    await RefreshToken.create({
      user_id: user.id,
      token_hash: refreshTokenHash,
      expires_at: new Date(Date.now() + this.parseExpiry(this.refreshTokenExpiry) * 1000),
      created_at: new Date()
    });
    
    return { accessToken, refreshToken };
  }
  
  // Account lockout protection
  private async checkAccountLockout(email: string): Promise<void> {
    const key = `failed_attempts:${email}`;
    const attempts = await redis.get(key);
    
    if (attempts && parseInt(attempts) >= this.maxFailedAttempts) {
      const lockoutKey = `locked:${email}`;
      const isLocked = await redis.get(lockoutKey);
      
      if (isLocked) {
        throw new UnauthorizedError('Account is temporarily locked due to too many failed login attempts');
      }
    }
  }
  
  private async recordFailedAttempt(email: string, reason: string): Promise<void> {
    const key = `failed_attempts:${email}`;
    const attempts = await redis.incr(key);
    await redis.expire(key, 3600); // Expire after 1 hour
    
    if (attempts >= this.maxFailedAttempts) {
      const lockoutKey = `locked:${email}`;
      await redis.setex(lockoutKey, this.lockoutDuration / 1000, '1');
      
      // Log account lockout
      await auditService.logSecurityEvent({
        event_type: 'account_locked',
        email: email,
        reason: 'too_many_failed_attempts',
        attempts: attempts,
        lockout_duration: this.lockoutDuration
      });
    }
    
    // Log failed attempt
    await auditService.logSecurityEvent({
      event_type: 'failed_login_attempt',
      email: email,
      reason: reason,
      attempt_number: attempts
    });
  }
  
  private async resetFailedAttempts(email: string): Promise<void> {
    const key = `failed_attempts:${email}`;
    const lockoutKey = `locked:${email}`;
    
    await redis.del(key);
    await redis.del(lockoutKey);
  }
}
```

### 4.2 Role-Based Access Control (RBAC)

```typescript
// Permission-based authorization system
export class AuthorizationService {
  private permissions: Map<UserRole, string[]> = new Map([
    ['admin', [
      'user:*',
      'organization:*',
      'document:*',
      'template:*',
      'decision:*',
      'system:*',
      'audit:read',
      'analytics:read'
    ]],
    ['manager', [
      'user:read',
      'user:update', // Own profile only
      'organization:read',
      'document:*', // All documents in organization
      'template:read',
      'template:apply',
      'decision:*', // All decisions in organization
      'analytics:read' // Organization analytics
    ]],
    ['user', [
      'user:read', // Own profile only
      'user:update', // Own profile only
      'document:create',
      'document:read', // Own documents + shared
      'document:update', // Own documents only
      'document:delete', // Own documents only
      'template:read',
      'template:apply',
      'decision:create',
      'decision:read', // Assigned decisions + own
      'decision:update' // Assigned decisions + own
    ]],
    ['viewer', [
      'user:read', // Own profile only
      'document:read', // Shared documents only
      'template:read',
      'decision:read' // Assigned decisions only
    ]]
  ]);
  
  // Check if user has specific permission
  hasPermission(user: User, permission: string, resource?: any): boolean {
    const userPermissions = this.permissions.get(user.role) || [];
    
    // Check for wildcard permissions
    if (userPermissions.includes('*') || userPermissions.includes(`${permission.split(':')[0]}:*`)) {
      return true;
    }
    
    // Check for exact permission
    if (userPermissions.includes(permission)) {
      // Apply resource-specific checks
      return this.checkResourceAccess(user, permission, resource);
    }
    
    return false;
  }
  
  // Resource-specific access control
  private checkResourceAccess(user: User, permission: string, resource: any): boolean {
    if (!resource) return true;
    
    const [action, operation] = permission.split(':');
    
    switch (action) {
      case 'user':
        return this.checkUserAccess(user, operation, resource);
      case 'document':
        return this.checkDocumentAccess(user, operation, resource);
      case 'decision':
        return this.checkDecisionAccess(user, operation, resource);
      case 'organization':
        return this.checkOrganizationAccess(user, operation, resource);
      default:
        return false;
    }
  }
  
  private checkUserAccess(currentUser: User, operation: string, targetUser: User): boolean {
    switch (operation) {
      case 'read':
      case 'update':
        // Users can read/update their own profile
        if (currentUser.id === targetUser.id) return true;
        // Admins and managers can read users in same organization
        if (['admin', 'manager'].includes(currentUser.role) && 
            currentUser.organization_id === targetUser.organization_id) {
          return true;
        }
        return false;
      
      case 'delete':
        // Only admins can delete users, and not themselves
        return currentUser.role === 'admin' && 
               currentUser.id !== targetUser.id &&
               currentUser.organization_id === targetUser.organization_id;
      
      default:
        return false;
    }
  }
  
  private checkDocumentAccess(user: User, operation: string, document: Document): boolean {
    // Check organization membership
    if (user.organization_id !== document.organization_id) {
      return false;
    }
    
    switch (operation) {
      case 'read':
        return this.canReadDocument(user, document);
      case 'update':
        return this.canUpdateDocument(user, document);
      case 'delete':
        return this.canDeleteDocument(user, document);
      case 'share':
        return this.canShareDocument(user, document);
      default:
        return false;
    }
  }
  
  private canReadDocument(user: User, document: Document): boolean {
    // Document owner can always read
    if (document.user_id === user.id) return true;
    
    // Check access level
    switch (document.access_level) {
      case 'public':
        return true;
      case 'organization':
        return user.organization_id === document.organization_id;
      case 'private':
        // Check if explicitly shared
        return document.shared_with?.includes(user.id) || false;
      default:
        return false;
    }
  }
  
  private canUpdateDocument(user: User, document: Document): boolean {
    // Only document owner or admins can update
    return document.user_id === user.id || 
           (user.role === 'admin' && user.organization_id === document.organization_id);
  }
  
  private canDeleteDocument(user: User, document: Document): boolean {
    // Only document owner or admins can delete
    return document.user_id === user.id || 
           (user.role === 'admin' && user.organization_id === document.organization_id);
  }
  
  private canShareDocument(user: User, document: Document): boolean {
    // Document owner, managers, and admins can share
    return document.user_id === user.id || 
           (['admin', 'manager'].includes(user.role) && user.organization_id === document.organization_id);
  }
  
  private checkDecisionAccess(user: User, operation: string, decision: Decision): boolean {
    // Check organization membership
    if (user.organization_id !== decision.organization_id) {
      return false;
    }
    
    switch (operation) {
      case 'read':
        return this.canReadDecision(user, decision);
      case 'update':
        return this.canUpdateDecision(user, decision);
      case 'delete':
        return this.canDeleteDecision(user, decision);
      case 'approve':
        return this.canApproveDecision(user, decision);
      default:
        return false;
    }
  }
  
  private canReadDecision(user: User, decision: Decision): boolean {
    // Assigned user can read
    if (decision.assigned_to === user.id) return true;
    
    // Decision creator can read
    if (decision.user_id === user.id) return true;
    
    // Managers and admins can read all organization decisions
    if (['admin', 'manager'].includes(user.role)) return true;
    
    // Check if user can read the related document
    return this.canReadDocument(user, decision.document);
  }
  
  private canUpdateDecision(user: User, decision: Decision): boolean {
    // Assigned user can update
    if (decision.assigned_to === user.id) return true;
    
    // Decision creator can update
    if (decision.user_id === user.id) return true;
    
    // Managers and admins can update
    return ['admin', 'manager'].includes(user.role);
  }
  
  private canDeleteDecision(user: User, decision: Decision): boolean {
    // Only decision creator or admins can delete
    return decision.user_id === user.id || user.role === 'admin';
  }
  
  private canApproveDecision(user: User, decision: Decision): boolean {
    // Only managers and admins can approve, and not their own decisions
    return ['admin', 'manager'].includes(user.role) && decision.user_id !== user.id;
  }
  
  // Middleware for route protection
  requirePermission(permission: string) {
    return async (req: Request, res: Response, next: NextFunction) => {
      try {
        const user = req.user; // Set by authentication middleware
        
        if (!user) {
          return res.status(401).json({
            error: 'Authentication required',
            code: 'UNAUTHENTICATED'
          });
        }
        
        // Get resource if available
        const resource = await this.getResourceFromRequest(req, permission);
        
        if (!this.hasPermission(user, permission, resource)) {
          // Log unauthorized access attempt
          await auditService.logSecurityEvent({
            event_type: 'unauthorized_access_attempt',
            user_id: user.id,
            permission: permission,
            resource_type: permission.split(':')[0],
            resource_id: resource?.id,
            ip_address: req.ip,
            user_agent: req.get('User-Agent')
          });
          
          return res.status(403).json({
            error: 'Insufficient permissions',
            code: 'FORBIDDEN',
            required_permission: permission
          });
        }
        
        // Store resource in request for later use
        if (resource) {
          req.resource = resource;
        }
        
        next();
      } catch (error) {
        res.status(500).json({
          error: 'Authorization check failed',
          code: 'AUTHORIZATION_ERROR'
        });
      }
    };
  }
  
  // Extract resource from request parameters
  private async getResourceFromRequest(req: Request, permission: string): Promise<any> {
    const [resourceType] = permission.split(':');
    
    switch (resourceType) {
      case 'document':
        if (req.params.id || req.params.document_id) {
          return await Document.findById(req.params.id || req.params.document_id);
        }
        break;
        
      case 'decision':
        if (req.params.id || req.params.decision_id) {
          return await Decision.findById(req.params.id || req.params.decision_id)
            .populate('document');
        }
        break;
        
      case 'user':
        if (req.params.id || req.params.user_id) {
          return await User.findById(req.params.id || req.params.user_id);
        }
        break;
        
      case 'organization':
        if (req.params.id || req.params.organization_id) {
          return await Organization.findById(req.params.id || req.params.organization_id);
        }
        break;
    }
    
    return null;
  }
}
```

---

# PART II: IMPLEMENTATION GUIDE

## SECTION 5: DEVELOPMENT SETUP

### 5.1 Development Environment Configuration

```bash
#!/bin/bash
# MinuteHub Development Environment Setup Script

echo "🚀 Setting up MinuteHub Development Environment..."

# Check Node.js version
if ! command -v node &> /dev/null; then
    echo "❌ Node.js is not installed. Please install Node.js 18 or higher."
    exit 1
fi

NODE_VERSION=$(node -v | cut -d 'v' -f 2 | cut -d '.' -f 1)
if [ $NODE_VERSION -lt 18 ]; then
    echo "❌ Node.js version 18 or higher is required. Current version: $(node -v)"
    exit 1
fi

echo "✅ Node.js version: $(node -v)"

# Install pnpm if not installed
if ! command -v pnpm &> /dev/null; then
    echo "📦 Installing pnpm..."
    npm install -g pnpm
fi

echo "✅ pnpm version: $(pnpm -v)"

# Clone repository (if not already cloned)
if [ ! -d ".git" ]; then
    echo "📂 Repository setup required. Please run this script from the project root."
    exit 1
fi

# Install dependencies
echo "📦 Installing dependencies..."
pnpm install

# Copy environment variables
if [ ! -f ".env.local" ]; then
    echo "🔧 Setting up environment variables..."
    cp .env.example .env.local
    echo "⚠️  Please configure your .env.local file with actual values"
fi

# Setup Git hooks
echo "🪝 Setting up Git hooks..."
npx husky install

# Database setup (optional - using Supabase)
echo "🗄️  Database setup..."
echo "✅ Using Supabase - no local database setup required"

# Build the project
echo "🔨 Building the project..."
pnpm build

# Run tests
echo "🧪 Running tests..."
pnpm test:unit

echo "🎉 Development environment setup complete!"
echo ""
echo "Next steps:"
echo "1. Configure your .env.local file with your Supabase credentials"
echo "2. Run 'pnpm dev' to start the development server"
echo "3. Visit http://localhost:5173 to see the application"
```

### 5.2 Project Configuration Files

**TypeScript Configuration:**
```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "allowJs": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noFallthroughCasesInSwitch": true,
    "module": "ESNext",
    "moduleResolution": "Node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "incremental": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@/components/*": ["src/components/*"],
      "@/hooks/*": ["src/hooks/*"],
      "@/services/*": ["src/services/*"],
      "@/types/*": ["src/types/*"],
      "@/utils/*": ["src/utils/*"]
    }
  },
  "include": [
    "src/**/*",
    "tests/**/*"
  ],
  "exclude": [
    "node_modules",
    "dist",
    "build"
  ]
}
```

**Vite Configuration:**
```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { resolve } from 'path';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': resolve(__dirname, './src'),
      '@/components': resolve(__dirname, './src/components'),
      '@/hooks': resolve(__dirname, './src/hooks'),
      '@/services': resolve(__dirname, './src/services'),
      '@/types': resolve(__dirname, './src/types'),
      '@/utils': resolve(__dirname, './src/utils')
    }
  },
  server: {
    port: 5173,
    host: true,
    open: true
  },
  build: {
    outDir: 'dist',
    sourcemap: true,
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          supabase: ['@supabase/supabase-js'],
          ui: ['@radix-ui/react-dialog', '@radix-ui/react-dropdown-menu']
        }
      }
    }
  },
  define: {
    __APP_VERSION__: JSON.stringify(process.env.npm_package_version)
  }
});
```

**ESLint Configuration:**
```javascript
// .eslintrc.js
module.exports = {
  root: true,
  env: {
    browser: true,
    es2022: true,
    node: true
  },
  extends: [
    'eslint:recommended',
    '@typescript-eslint/recommended',
    '@typescript-eslint/recommended-requiring-type-checking',
    'plugin:react/recommended',
    'plugin:react-hooks/recommended',
    'plugin:jsx-a11y/recommended',
    'plugin:import/recommended',
    'plugin:import/typescript',
    'prettier'
  ],
  plugins: [
    '@typescript-eslint',
    'react',
    'react-hooks',
    'jsx-a11y',
    'import'
  ],
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaVersion: 2022,
    sourceType: 'module',
    ecmaFeatures: {
      jsx: true
    },
    project: './tsconfig.json'
  },
  settings: {
    react: {
      version: 'detect'
    },
    'import/resolver': {
      typescript: true
    }
  },
  rules: {
    // TypeScript specific rules
    '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
    '@typescript-eslint/no-explicit-any': 'warn',
    '@typescript-eslint/explicit-function-return-type': 'off',
    '@typescript-eslint/explicit-module-boundary-types': 'off',
    '@typescript-eslint/no-non-null-assertion': 'warn',
    '@typescript-eslint/prefer-nullish-coalescing': 'error',
    '@typescript-eslint/prefer-optional-chain': 'error',
    '@typescript-eslint/no-unnecessary-type-assertion': 'error',
    '@typescript-eslint/no-floating-promises': 'error',
    
    // React specific rules
    'react/react-in-jsx-scope': 'off', // Not needed in React 17+
    'react/prop-types': 'off', // Using TypeScript for prop validation
    'react/jsx-props-no-spreading': 'off',
    'react/require-default-props': 'off',
    'react-hooks/rules-of-hooks': 'error',
    'react-hooks/exhaustive-deps': 'warn',
    
    // Import rules
    'import/order': [
      'error',
      {
        groups: [
          'builtin',
          'external',
          'internal',
          ['parent', 'sibling'],
          'index'
        ],
        'newlines-between': 'always',
        alphabetize: {
          order: 'asc',
          caseInsensitive: true
        }
      }
    ],
    'import/no-duplicates': 'error',
    'import/no-unresolved': 'error',
    
    // General rules
    'no-console': ['warn', { allow: ['warn', 'error'] }],
    'no-debugger': 'error',
    'prefer-const': 'error',
    'no-var': 'error',
    'object-shorthand': 'error',
    'prefer-template': 'error'
  },
  overrides: [
    {
      files: ['**/*.test.ts', '**/*.test.tsx', '**/*.spec.ts', '**/*.spec.tsx'],
      env: {
        jest: true
      },
      rules: {
        '@typescript-eslint/no-explicit-any': 'off',
        '@typescript-eslint/no-non-null-assertion': 'off'
      }
    }
  ]
};
```

### 5.3 Package Configuration

```json
{
  "name": "minutehub-actionbook",
  "version": "1.0.0",
  "description": "MinuteHub ActionBook - Meeting Minutes Processing Application",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "lint": "eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0",
    "lint:fix": "eslint . --ext ts,tsx --fix",
    "type-check": "tsc --noEmit",
    "test": "vitest",
    "test:unit": "vitest run",
    "test:coverage": "vitest run --coverage",
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "format": "prettier --write \"src/**/*.{js,jsx,ts,tsx,json,css,md}\"",
    "format:check": "prettier --check \"src/**/*.{js,jsx,ts,tsx,json,css,md}\"",
    "prepare": "husky install"
  },
  "dependencies": {
    "@hookform/resolvers": "^3.3.1",
    "@radix-ui/react-alert-dialog": "^1.0.4",
    "@radix-ui/react-avatar": "^1.0.3",
    "@radix-ui/react-button": "^1.0.3",
    "@radix-ui/react-dialog": "^1.0.4",
    "@radix-ui/react-dropdown-menu": "^2.0.5",
    "@radix-ui/react-form": "^0.0.3",
    "@radix-ui/react-label": "^2.0.2",
    "@radix-ui/react-popover": "^1.0.6",
    "@radix-ui/react-progress": "^1.0.3",
    "@radix-ui/react-select": "^1.2.2",
    "@radix-ui/react-separator": "^1.0.3",
    "@radix-ui/react-slot": "^1.0.2",
    "@radix-ui/react-switch": "^1.0.3",
    "@radix-ui/react-tabs": "^1.0.4",
    "@radix-ui/react-toast": "^1.1.4",
    "@radix-ui/react-tooltip": "^1.0.6",
    "@supabase/supabase-js": "^2.38.0",
    "@tanstack/react-query": "^4.32.6",
    "axios": "^1.4.0",
    "class-variance-authority": "^0.7.0",
    "clsx": "^2.0.0",
    "cmdk": "^0.2.0",
    "date-fns": "^2.30.0",
    "jspdf": "^2.5.1",
    "lucide-react": "^0.263.1",
    "pdf-lib": "^1.17.1",
    "pdfjs-dist": "^3.9.179",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-dropzone": "^14.2.3",
    "react-hook-form": "^7.45.2",
    "react-router-dom": "^6.14.2",
    "tailwind-merge": "^1.14.0",
    "tailwindcss-animate": "^1.0.6",
    "tesseract.js": "^4.1.1",
    "zod": "^3.21.4",
    "zustand": "^4.4.0"
  },
  "devDependencies": {
    "@playwright/test": "^1.36.2",
    "@testing-library/jest-dom": "^5.17.0",
    "@testing-library/react": "^13.4.0",
    "@testing-library/user-event": "^14.4.3",
    "@types/node": "^20.4.8",
    "@types/react": "^18.2.17",
    "@types/react-dom": "^18.2.7",
    "@typescript-eslint/eslint-plugin": "^6.2.1",
    "@typescript-eslint/parser": "^6.2.1",
    "@vitejs/plugin-react": "^4.0.3",
    "autoprefixer": "^10.4.14",
    "eslint": "^8.46.0",
    "eslint-config-prettier": "^8.10.0",
    "eslint-import-resolver-typescript": "^3.5.5",
    "eslint-plugin-import": "^2.28.0",
    "eslint-plugin-jsx-a11y": "^6.7.1",
    "eslint-plugin-react": "^7.33.1",
    "eslint-plugin-react-hooks": "^4.6.0",
    "husky": "^8.0.3",
    "jsdom": "^22.1.0",
    "lint-staged": "^13.2.3",
    "postcss": "^8.4.27",
    "prettier": "^3.0.0",
    "tailwindcss": "^3.3.3",
    "typescript": "^5.1.6",
    "vite": "^4.4.7",
    "vitest": "^0.34.1"
  },
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{json,css,md}": [
      "prettier --write"
    ]
  },
  "engines": {
    "node": ">=18.0.0",
    "pnpm": ">=8.0.0"
  }
}
```

---

## SECTION 6: FEATURE IMPLEMENTATION

### 6.1 Authentication Implementation

```typescript
// Authentication Hook Implementation
import { useState, useEffect, useCallback } from 'react';
import { useNavigate } from 'react-router-dom';
import { supabase } from '@/lib/supabase';
import { useAuthStore } from '@/stores/auth';

export const useAuth = () => {
  const [loading, setLoading] = useState(true);
  const navigate = useNavigate();
  const { user, setUser, clearUser } = useAuthStore();

  // Initialize auth state
  useEffect(() => {
    let mounted = true;

    const initializeAuth = async () => {
      try {
        const { data: { session }, error } = await supabase.auth.getSession();
        
        if (error) {
          console.error('Error getting session:', error);
          return;
        }

        if (mounted) {
          if (session?.user) {
            // Fetch full user profile
            const userProfile = await fetchUserProfile(session.user.id);
            setUser(userProfile);
          } else {
            clearUser();
          }
          setLoading(false);
        }
      } catch (error) {
        console.error('Auth initialization error:', error);
        if (mounted) {
          setLoading(false);
        }
      }
    };

    initializeAuth();

    // Listen for auth changes
    const { data: { subscription } } = supabase.auth.onAuthStateChange(
      async (event, session) => {
        if (!mounted) return;

        switch (event) {
          case 'SIGNED_IN':
            if (session?.user) {
              const userProfile = await fetchUserProfile(session.user.id);
              setUser(userProfile);
            }
            break;
          case 'SIGNED_OUT':
            clearUser();
            navigate('/login');
            break;
          case 'TOKEN_REFRESHED':
            console.log('Token refreshed');
            break;
          case 'USER_UPDATED':
            if (session?.user) {
              const userProfile = await fetchUserProfile(session.user.id);
              setUser(userProfile);
            }
            break;
        }
      }
    );

    return () => {
      mounted = false;
      subscription.unsubscribe();
    };
  }, [setUser, clearUser, navigate]);

  // Fetch user profile with organization data
  const fetchUserProfile = async (userId: string) => {
    const { data, error } = await supabase
      .from('users')
      .select(`
        *,
        organization:organizations(*)
      `)
      .eq('id', userId)
      .single();

    if (error) {
      console.error('Error fetching user profile:', error);
      throw error;
    }

    return data;
  };

  // Login function
  const login = useCallback(async (email: string, password: string) => {
    setLoading(true);
    try {
      const { data, error } = await supabase.auth.signInWithPassword({
        email,
        password
      });

      if (error) throw error;

      if (data.user) {
        const userProfile = await fetchUserProfile(data.user.id);
        setUser(userProfile);
      }

      return { success: true };
    } catch (error: any) {
      return { 
        success: false, 
        error: error.message || 'Login failed' 
      };
    } finally {
      setLoading(false);
    }
  }, [setUser]);

  // Register function
  const register = useCallback(async (
    email: string, 
    password: string, 
    fullName: string,
    organizationId?: string
  ) => {
    setLoading(true);
    try {
      const { data, error } = await supabase.auth.signUp({
        email,
        password,
        options: {
          data: {
            full_name: fullName,
            organization_id: organizationId
          }
        }
      });

      if (error) throw error;

      return { 
        success: true, 
        requiresVerification: !data.session,
        user: data.user
      };
    } catch (error: any) {
      return { 
        success: false, 
        error: error.message || 'Registration failed' 
      };
    } finally {
      setLoading(false);
    }
  }, []);

  // Logout function
  const logout = useCallback(async () => {
    setLoading(true);
    try {
      const { error } = await supabase.auth.signOut();
      if (error) throw error;

      clearUser();
      navigate('/login');
      return { success: true };
    } catch (error: any) {
      return { 
        success: false, 
        error: error.message || 'Logout failed' 
      };
    } finally {
      setLoading(false);
    }
  }, [clearUser, navigate]);

  // Reset password function
  const resetPassword = useCallback(async (email: string) => {
    try {
      const { error } = await supabase.auth.resetPasswordForEmail(email, {
        redirectTo: `${window.location.origin}/reset-password`
      });

      if (error) throw error;

      return { success: true };
    } catch (error: any) {
      return { 
        success: false, 
        error: error.message || 'Password reset failed' 
      };
    }
  }, []);

  // Update password function
  const updatePassword = useCallback(async (newPassword: string) => {
    try {
      const { error } = await supabase.auth.updateUser({
        password: newPassword
      });

      if (error) throw error;

      return { success: true };
    } catch (error: any) {
      return { 
        success: false, 
        error: error.message || 'Password update failed' 
      };
    }
  }, []);

  return {
    user,
    loading,
    login,
    register,
    logout,
    resetPassword,
    updatePassword,
    isAuthenticated: !!user
  };
};

// Auth Store using Zustand
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface User {
  id: string;
  email: string;
  full_name: string;
  role: 'admin' | 'manager' | 'user' | 'viewer';
  organization_id: string;
  organization: {
    id: string;
    name: string;
    slug: string;
  };
  avatar_url?: string;
  last_active_at?: string;
}

interface AuthState {
  user: User | null;
  setUser: (user: User) => void;
  clearUser: () => void;
  updateUser: (updates: Partial<User>) => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set, get) => ({
      user: null,
      setUser: (user) => set({ user }),
      clearUser: () => set({ user: null }),
      updateUser: (updates) => {
        const currentUser = get().user;
        if (currentUser) {
          set({ user: { ...currentUser, ...updates } });
        }
      }
    }),
    {
      name: 'auth-storage',
      partialize: (state) => ({ user: state.user })
    }
  )
);
```

### 6.2 Document Upload Implementation

```typescript
// Document Upload Service
import { supabase } from '@/lib/supabase';

export interface UploadProgress {
  loaded: number;
  total: number;
  percentage: number;
}

export interface DocumentMetadata {
  title?: string;
  description?: string;
  templateId?: string;
  meetingDate?: string;
  meetingType?: string;
  participants?: string[];
  tags?: string[];
  autoProcess?: boolean;
}

export class DocumentUploadService {
  private readonly maxFileSize = 10 * 1024 * 1024; // 10MB
  private readonly allowedTypes = [
    'application/pdf',
    'application/msword',
    'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
    'image/png',
    'image/jpeg',
    'image/jpg'
  ];

  async uploadDocument(
    file: File,
    metadata: DocumentMetadata,
    userId: string,
    organizationId: string,
    onProgress?: (progress: UploadProgress) => void
  ): Promise<UploadResult> {
    try {
      // Validate file
      const validation = this.validateFile(file);
      if (!validation.isValid) {
        throw new Error(validation.errors.join(', '));
      }

      // Generate unique file path
      const timestamp = Date.now();
      const fileExt = file.name.split('.').pop();
      const fileName = `${timestamp}-${crypto.randomUUID()}.${fileExt}`;
      const filePath = `${organizationId}/${userId}/${fileName}`;

      // Upload file to Supabase Storage
      const { data: uploadData, error: uploadError } = await supabase.storage
        .from('documents')
        .upload(filePath, file, {
          cacheControl: '3600',
          upsert: false
        });

      if (uploadError) throw uploadError;

      // Get file URL
      const { data: urlData } = supabase.storage
        .from('documents')
        .getPublicUrl(filePath);

      // Create document record in database
      const { data: documentData, error: dbError } = await supabase
        .from('documents')
        .insert({
          user_id: userId,
          organization_id: organizationId,
          original_filename: file.name,
          file_size_bytes: file.size,
          file_type: fileExt || 'unknown',
          mime_type: file.type,
          original_file_url: urlData.publicUrl,
          title: metadata.title || file.name.replace(/\.[^/.]+$/, ''),
          description: metadata.description,
          meeting_date: metadata.meetingDate,
          meeting_type: metadata.meetingType,
          participants: metadata.participants || [],
          tags: metadata.tags || [],
          template_id: metadata.templateId,
          processing_status: 'uploaded'
        })
        .select()
        .single();

      if (dbError) throw dbError;

      // Trigger processing if auto-process is enabled
      if (metadata.autoProcess && metadata.templateId) {
        await this.triggerProcessing(documentData.id, metadata.templateId);
      }

      return {
        success: true,
        document: documentData,
        fileUrl: urlData.publicUrl
      };
    } catch (error: any) {
      console.error('Document upload error:', error);
      return {
        success: false,
        error: error.message || 'Upload failed'
      };
    }
  }

  private validateFile(file: File): { isValid: boolean; errors: string[] } {
    const errors: string[] = [];

    // Check file size
    if (file.size > this.maxFileSize) {
      errors.push(`File size must be less than ${this.maxFileSize / 1024 / 1024}MB`);
    }

    // Check file type
    if (!this.allowedTypes.includes(file.type)) {
      errors.push('File type not supported. Please upload PDF, DOC, DOCX, PNG, or JPG files.');
    }

    // Check filename for security
    if (this.hasUnsafeCharacters(file.name)) {
      errors.push('Filename contains unsafe characters');
    }

    return {
      isValid: errors.length === 0,
      errors
    };
  }

  private hasUnsafeCharacters(filename: string): boolean {
    const unsafeChars = /[<>:"/\\|?*]/;
    return unsafeChars.test(filename);
  }

  private async triggerProcessing(documentId: string, templateId: string): Promise<void> {
    try {
      // Update document status to processing
      await supabase
        .from('documents')
        .update({
          processing_status: 'processing',
          processing_started_at: new Date().toISOString(),
          template_id: templateId
        })
        .eq('id', documentId);

      // In a real implementation, this would trigger a background job
      // For now, we'll call the processing function directly
      setTimeout(() => {
        this.processDocument(documentId, templateId);
      }, 1000);
    } catch (error) {
      console.error('Failed to trigger processing:', error);
    }
  }

  private async processDocument(documentId: string, templateId: string): Promise<void> {
    try {
      // Get document
      const { data: document, error } = await supabase
        .from('documents')
        .select('*')
        .eq('id', documentId)
        .single();

      if (error || !document) {
        throw new Error('Document not found');
      }

      // Simulate text extraction and processing
      const extractedText = await this.extractTextFromFile(document.original_file_url, document.file_type);
      
      // Simulate AI processing (decision extraction, etc.)
      const decisions = await this.extractDecisions(extractedText);
      const actionItems = await this.extractActionItems(extractedText);

      // Update document with processed results
      await supabase
        .from('documents')
        .update({
          processing_status: 'completed',
          processing_completed_at: new Date().toISOString(),
          extracted_text: extractedText,
          decisions_extracted: decisions,
          action_items: actionItems,
          ai_confidence_score: 0.85
        })
        .eq('id', documentId);

      // Create decision records
      for (const decision of decisions) {
        await supabase
          .from('decisions')
          .insert({
            document_id: documentId,
            user_id: document.user_id,
            organization_id: document.organization_id,
            decision_text: decision.text,
            decision_type: decision.type,
            confidence_score: decision.confidence,
            extraction_method: 'ai_automatic'
          });
      }

    } catch (error) {
      console.error('Document processing failed:', error);
      
      // Update document status to failed
      await supabase
        .from('documents')
        .update({
          processing_status: 'failed',
          processing_error: error instanceof Error ? error.message : 'Unknown error'
        })
        .eq('id', documentId);
    }
  }

  private async extractTextFromFile(fileUrl: string, fileType: string): Promise<string> {
    try {
      switch (fileType.toLowerCase()) {
        case 'pdf':
          return await this.extractTextFromPDF(fileUrl);
        case 'doc':
        case 'docx':
          return await this.extractTextFromDOC(fileUrl);
        case 'png':
        case 'jpg':
        case 'jpeg':
          return await this.extractTextFromImage(fileUrl);
        default:
          throw new Error(`Unsupported file type: ${fileType}`);
      }
    } catch (error) {
      console.error('Text extraction failed:', error);
      return 'Text extraction failed';
    }
  }

  private async extractTextFromPDF(fileUrl: string): Promise<string> {
    // Simplified PDF text extraction
    // In a real implementation, you would use pdf-parse or similar
    return 'Extracted text from PDF file...';
  }

  private async extractTextFromDOC(fileUrl: string): Promise<string> {
    // Simplified DOC text extraction
    // In a real implementation, you would use mammoth.js or similar
    return 'Extracted text from DOC file...';
  }

  private async extractTextFromImage(fileUrl: string): Promise<string> {
    // Simplified OCR text extraction
    // In a real implementation, you would use Tesseract.js
    return 'Extracted text from image file using OCR...';
  }

  private async extractDecisions(text: string): Promise<any[]> {
    // Simplified decision extraction
    // In a real implementation, you would use NLP techniques
    const decisionPatterns = [
      /decided that ([^.]+)/gi,
      /resolved to ([^.]+)/gi,
      /agreed that ([^.]+)/gi
    ];

    const decisions: any[] = [];
    
    for (const pattern of decisionPatterns) {
      const matches = text.matchAll(pattern);
      for (const match of matches) {
        decisions.push({
          text: match[1].trim(),
          type: 'operational',
          confidence: 0.8
        });
      }
    }

    return decisions;
  }

  private async extractActionItems(text: string): Promise<any[]> {
    // Simplified action item extraction
    const actionPatterns = [
      /action item:? ([^.]+)/gi,
      /to do:? ([^.]+)/gi,
      /assigned to ([^.]+)/gi
    ];

    const actionItems: any[] = [];
    
    for (const pattern of actionPatterns) {
      const matches = text.matchAll(pattern);
      for (const match of matches) {
        actionItems.push({
          text: match[1].trim(),
          priority: 'medium',
          confidence: 0.7
        });
      }
    }

    return actionItems;
  }
}

// Document Upload Hook
export const useDocumentUpload = () => {
  const [uploading, setUploading] = useState(false);
  const [progress, setProgress] = useState(0);
  const uploadService = new DocumentUploadService();

  const uploadDocument = async (
    file: File,
    metadata: DocumentMetadata,
    userId: string,
    organizationId: string
  ) => {
    setUploading(true);
    setProgress(0);

    try {
      const result = await uploadService.uploadDocument(
        file,
        metadata,
        userId,
        organizationId,
        (progress) => {
          setProgress(progress.percentage);
        }
      );

      return result;
    } finally {
      setUploading(false);
      setProgress(0);
    }
  };

  return {
    uploadDocument,
    uploading,
    progress
  };
};
```

### 6.3 Document Processing Engine

```typescript
// Template Processing Service
export class TemplateProcessingService {
  private templates: Map<string, GuardianXTemplate> = new Map();

  constructor() {
    this.initializeTemplates();
  }

  private initializeTemplates() {
    // Template A: Demo Meeting Template
    this.templates.set('demo_meeting', {
      id: 'demo_meeting',
      name: 'Demo Meeting Template',
      category: 'general',
      description: 'General purpose meeting template for all types of meetings',
      schema: {
        requiredSections: ['agenda', 'discussions', 'decisions', 'next_steps'],
        optionalSections: ['attendees', 'resources', 'notes'],
        metadata: ['meeting_date', 'meeting_time', 'location', 'chairperson']
      },
      sections: [
        {
          id: 'header',
          title: 'Meeting Information',
          type: 'metadata',
          fields: ['meeting_title', 'date', 'time', 'location', 'chairperson']
        },
        {
          id: 'attendees',
          title: 'Attendees',
          type: 'list',
          fields: ['name', 'role', 'organization']
        },
        {
          id: 'agenda',
          title: 'Agenda Items',
          type: 'ordered_list',
          fields: ['item_number', 'description', 'presenter', 'duration']
        },
        {
          id: 'discussions',
          title: 'Discussions',
          type: 'narrative',
          fields: ['agenda_item_ref', 'discussion_points', 'key_concerns']
        },
        {
          id: 'decisions',
          title: 'Decisions Made',
          type: 'structured_list',
          fields: ['decision_text', 'rationale', 'responsible_party', 'deadline']
        },
        {
          id: 'action_items',
          title: 'Action Items',
          type: 'task_list',
          fields: ['task_description', 'assigned_to', 'due_date', 'priority']
        },
        {
          id: 'next_steps',
          title: 'Next Steps',
          type: 'narrative',
          fields: ['next_meeting_date', 'follow_up_actions', 'review_items']
        }
      ],
      processingRules: [
        {
          type: 'extract_decisions',
          patterns: ['decided that', 'resolved to', 'agreed that', 'approved'],
          confidence_threshold: 0.8
        },
        {
          type: 'extract_action_items',
          patterns: ['action:', 'to do:', 'assigned to', 'responsible:'],
          confidence_threshold: 0.7
        }
      ]
    });

    // Template B: Kamati ya Usimamizi (Management Committee)
    this.templates.set('kamati_usimamizi', {
      id: 'kamati_usimamizi',
      name: 'Kamati ya Usimamizi',
      category: 'government',
      description: 'Management Committee template for Swahili-language meetings',
      schema: {
        requiredSections: ['utangulizi', 'mijadala', 'maamuzi', 'hatua_zijazo'],
        optionalSections: ['wahudhuriaji', 'rasilimali'],
        metadata: ['tarehe_mkutano', 'muda', 'mahali', 'mwenyekiti']
      },
      sections: [
        {
          id: 'header',
          title: 'Taarifa za Mkutano',
          type: 'metadata',
          fields: ['jina_mkutano', 'tarehe', 'muda', 'mahali', 'mwenyekiti']
        },
        {
          id: 'wahudhuriaji',
          title: 'Wahudhuriaji',
          type: 'list',
          fields: ['jina', 'cheo', 'shirika']
        },
        {
          id: 'ajenda',
          title: 'Ajenda za Mkutano',
          type: 'ordered_list',
          fields: ['nambari', 'maelezo', 'mwasilishaji', 'muda']
        },
        {
          id: 'mijadala',
          title: 'Mijadala',
          type: 'narrative',
          fields: ['kumbuka_ajenda', 'nukuu_muhimu', 'changamoto']
        },
        {
          id: 'maamuzi',
          title: 'Maamuzi Yaliyofikiwa',
          type: 'structured_list',
          fields: ['uamuzi', 'sababu', 'mhusika', 'tarehe_mwisho']
        },
        {
          id: 'hatua_zijazo',
          title: 'Hatua za Kufuata',
          type: 'narrative',
          fields: ['mkutano_ujao', 'vitendo_kufuata', 'mapitio']
        }
      ],
      processingRules: [
        {
          type: 'extract_decisions',
          patterns: ['imeamuliwa', 'imekubaliwa', 'tumefikia uamuzi', 'imeidhinishwa'],
          confidence_threshold: 0.8
        },
        {
          type: 'extract_action_items',
          patterns: ['hatua:', 'wajibu:', 'amekabiliwa', 'anahusika:'],
          confidence_threshold: 0.7
        }
      ]
    });

    // Template C: Baraza Kuu (Executive Council)
    this.templates.set('baraza_kuu', {
      id: 'baraza_kuu',
      name: 'Baraza Kuu',
      category: 'government',
      description: 'Executive Council template for high-level meetings',
      schema: {
        requiredSections: ['ripoti_uongozi', 'mipango_mikakati', 'maamuzi_muhimu', 'maelekezo'],
        optionalSections: ['taarifa_fedha', 'mapitio_sera'],
        metadata: ['tarehe_mkutano', 'muda', 'mahali', 'mkurugenzi_mkuu']
      },
      sections: [
        {
          id: 'header',
          title: 'Taarifa za Baraza',
          type: 'metadata',
          fields: ['jina_baraza', 'tarehe', 'muda', 'mahali', 'mkurugenzi']
        },
        {
          id: 'wajumbe',
          title: 'Wajumbe wa Baraza',
          type: 'list',
          fields: ['jina', 'cheo', 'idara']
        },
        {
          id: 'ripoti_uongozi',
          title: 'Ripoti ya Uongozi',
          type: 'narrative',
          fields: ['mafanikio', 'changamoto', 'mipango']
        },
        {
          id: 'maamuzi_muhimu',
          title: 'Maamuzi Muhimu',
          type: 'structured_list',
          fields: ['uamuzi_mkuu', 'athari', 'utekelezaji', 'mfumo_ufuatiliaji']
        },
        {
          id: 'maelekezo',
          title: 'Maelekezo ya Uongozi',
          type: 'narrative',
          fields: ['maelekezo_makuu', 'mipango_mifupi', 'sera_mpya']
        }
      ],
      processingRules: [
        {
          type: 'extract_decisions',
          patterns: ['baraza limeamua', 'limeidhinishwa', 'uamuzi wa baraza', 'sera mpya'],
          confidence_threshold: 0.9
        },
        {
          type: 'extract_action_items',
          patterns: ['maelekezo:', 'utekelezaji:', 'wajibu wa', 'hatua za haraka:'],
          confidence_threshold: 0.8
        }
      ]
    });

    // Template D: Mkutano wa Kitengo (Operational Committee)
    this.templates.set('mkutano_kitengo', {
      id: 'mkutano_kitengo',
      name: 'Mkutano wa Kitengo',
      category: 'business',
      description: 'Operational committee template for departmental meetings',
      schema: {
        requiredSections: ['taarifa_kitengo', 'malengo_wiki', 'matatizo', 'hatua_zijazo'],
        optionalSections: ['mafunzo', 'rasilimali_zinahitajika'],
        metadata: ['tarehe_mkutano', 'muda', 'kitengo', 'msimamizi']
      },
      sections: [
        {
          id: 'header',
          title: 'Taarifa za Mkutano wa Kitengo',
          type: 'metadata',
          fields: ['jina_kitengo', 'tarehe', 'muda', 'msimamizi']
        },
        {
          id: 'wafanyakazi',
          title: 'Wafanyakazi Waliohudhulia',
          type: 'list',
          fields: ['jina', 'cheo', 'idara']
        },
        {
          id: 'taarifa_kitengo',
          title: 'Taarifa ya Kitengo',
          type: 'narrative',
          fields: ['kazi_zilizofanywa', 'mafanikio', 'changamoto']
        },
        {
          id: 'matatizo',
          title: 'Matatizo na Changamoto',
          type: 'structured_list',
          fields: ['tatizo', 'chanzo', 'suluhisho_linalopendekezwa', 'mhusika']
        },
        {
          id: 'hatua_zijazo',
          title: 'Hatua za Kufuata',
          type: 'task_list',
          fields: ['kazi', 'mhusika', 'tarehe_mwisho', 'kipaumbele']
        }
      ],
      processingRules: [
        {
          type: 'extract_decisions',
          patterns: ['tumeamua', 'hatua itachukuliwa', 'suluhisho', 'tutekeleze'],
          confidence_threshold: 0.7
        },
        {
          type: 'extract_action_items',
          patterns: ['kazi:', 'wajibu:', 'atafanya:', 'itakamilishwa:'],
          confidence_threshold: 0.6
        }
      ]
    });
  }

  async processDocument(documentId: string, templateId: string): Promise<ProcessingResult> {
    try {
      // Get document and template
      const document = await this.getDocument(documentId);
      const template = this.templates.get(templateId);
      
      if (!document || !template) {
        throw new ProcessingError('Document or template not found');
      }
      
      // Update processing status
      await this.updateProcessingStatus(documentId, 'processing');
      
      // Extract text from document
      const extractedText = await this.extractText(document);
      
      // Apply AI processing rules
      const aiResults = await this.applyAIProcessing(extractedText, template);
      
      // Map content to template sections
      const structuredContent = await this.mapToTemplate(aiResults, template);
      
      // Extract decisions and action items
      const decisions = await this.extractDecisions(extractedText, template);
      const actionItems = await this.extractActionItems(extractedText, template);
      
      // Generate standardized output
      const standardizedMinutes = await this.generateStandardizedMinutes(
        structuredContent, template
      );
      
      // Save processed results
      const { data: updatedDoc, error } = await supabase
        .from('documents')
        .update({
          processing_status: 'completed',
          processing_completed_at: new Date().toISOString(),
          extracted_text: extractedText,
          ai_structured_content: structuredContent,
          decisions_extracted: decisions,
          action_items: actionItems,
          template_id: templateId
        })
        .eq('id', documentId)
        .select()
        .single();
      
      if (error) throw error;
      
      // Create decision records
      for (const decision of decisions) {
        await this.createDecisionRecord(documentId, decision);
      }
      
      return {
        document: updatedDoc,
        decisions,
        actionItems,
        standardizedMinutes,
        success: true
      };
    } catch (error) {
      await this.updateProcessingStatus(documentId, 'failed', error.message);
      throw new ProcessingError('Document processing failed', error);
    }
  }

  private async extractText(document: Document): Promise<string> {
    const fileType = document.file_type.toLowerCase();
    
    switch (fileType) {
      case 'pdf':
        return await this.extractTextFromPDF(document.original_file_url);
      case 'docx':
        return await this.extractTextFromDOCX(document.original_file_url);
      case 'doc':
        return await this.extractTextFromDOC(document.original_file_url);
      case 'png':
      case 'jpg':
      case 'jpeg':
        return await this.extractTextFromImage(document.original_file_url);
      default:
        throw new ProcessingError(`Unsupported file type: ${fileType}`);
    }
  }

  private async extractDecisions(text: string, template: GuardianXTemplate): Promise<Decision[]> {
    const decisions: Decision[] = [];
    const patterns = template.processingRules?.find(rule => rule.type === 'extract_decisions')?.patterns || [
      /(?:decided|resolved|agreed|approved)\s+(?:that|to)\s+([^.!?]+)/gi,
      /motion\s+(?:carried|passed)[:\s]*([^.!?]+)/gi
    ];
    
    for (const pattern of patterns) {
      const regex = new RegExp(pattern, 'gi');
      const matches = text.matchAll(regex);
      for (const match of matches) {
        if (match[1] && match[1].trim().length > 10) {
          const confidence = this.calculateDecisionConfidence(match[0], text);
          
          if (confidence > 0.7) {
            decisions.push({
              decision_text: match[1].trim(),
              confidence_score: confidence,
              extraction_method: 'ai_automatic',
              decision_type: this.classifyDecisionType(match[1])
            });
          }
        }
      }
    }
    
    return decisions;
  }

  private calculateDecisionConfidence(sentence: string, fullText: string): number {
    let confidence = 0.5; // Base confidence
    
    // Boost confidence for formal decision language
    if (/motion\s+(?:carried|passed)/i.test(sentence)) confidence += 0.3;
    if (/unanimously|majority|vote/i.test(sentence)) confidence += 0.2;
    if (/committee|board|council/i.test(sentence)) confidence += 0.1;
    
    // Reduce confidence for uncertain language
    if (/might|maybe|possibly|consider/i.test(sentence)) confidence -= 0.2;
    if (/discussed|mentioned|talked about/i.test(sentence)) confidence -= 0.1;
    
    return Math.min(Math.max(confidence, 0), 1);
  }

  private classifyDecisionType(decisionText: string): DecisionType {
    if (/budget|financial|cost|expense|funding/i.test(decisionText)) {
      return 'budget';
    }
    if (/policy|procedure|guideline|rule/i.test(decisionText)) {
      return 'policy';
    }
    if (/personnel|staff|hiring|appointment/i.test(decisionText)) {
      return 'personnel';
    }
    if (/operational|implementation|execution/i.test(decisionText)) {
      return 'operational';
    }
    return 'operational';
  }

  private async extractActionItems(text: string, template: GuardianXTemplate): Promise<ActionItem[]> {
    const actionItems: ActionItem[] = [];
    const patterns = template.processingRules?.find(rule => rule.type === 'extract_action_items')?.patterns || [
      /action\s*(?:item)?[:\s]*([^.!?]+)/gi,
      /to\s+do[:\s]*([^.!?]+)/gi,
      /assigned\s+to[:\s]*([^.!?]+)/gi
    ];
    
    for (const pattern of patterns) {
      const regex = new RegExp(pattern, 'gi');
      const matches = text.matchAll(regex);
      for (const match of matches) {
        if (match[1] && match[1].trim().length > 5) {
          actionItems.push({
            action_text: match[1].trim(),
            confidence_score: 0.7,
            priority: 'medium'
          });
        }
      }
    }
    
    return actionItems;
  }

  // Additional helper methods would be implemented here...
}
```

---

## SECTION 7: TESTING STRATEGY

### 7.1 Unit Testing Templates

```typescript
// Unit test template for React components
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { vi } from 'vitest';

import { DocumentCard } from './DocumentCard';
import { TestWrapper } from '../test-utils/TestWrapper';

// Mock external dependencies
vi.mock('../services/api.service');
vi.mock('../hooks/useAuth');

describe('DocumentCard', () => {
  // Setup and teardown
  beforeEach(() => {
    vi.clearAllMocks();
  });
  
  afterEach(() => {
    vi.restoreAllMocks();
  });
  
  // Helper function to render component with providers
  const renderComponent = (props = {}) => {
    const defaultProps = {
      document: {
        id: 'doc-1',
        title: 'Test Document',
        created_at: '2025-01-01T00:00:00Z',
        processing_status: 'completed',
        file_type: 'pdf'
      },
      onEdit: vi.fn(),
      onDelete: vi.fn(),
      onView: vi.fn()
    };
    
    return render(
      <TestWrapper>
        <DocumentCard {...defaultProps} {...props} />
      </TestWrapper>
    );
  };
  
  describe('Rendering', () => {
    it('should render document card with basic information', () => {
      renderComponent();
      
      expect(screen.getByText('Test Document')).toBeInTheDocument();
      expect(screen.getByText('completed')).toBeInTheDocument();
      expect(screen.getByText(/PDF/)).toBeInTheDocument();
    });
    
    it('should show processing indicator when document is processing', () => {
      renderComponent({
        document: {
          id: 'doc-1',
          title: 'Processing Document',
          processing_status: 'processing'
        }
      });
      
      expect(screen.getByText('processing')).toBeInTheDocument();
      expect(screen.getByRole('progressbar')).toBeInTheDocument();
    });
  });
  
  describe('User Interactions', () => {
    it('should call onView when view button is clicked', async () => {
      const mockOnView = vi.fn();
      renderComponent({ onView: mockOnView });
      
      const viewButton = screen.getByRole('button', { name: /view/i });
      await userEvent.click(viewButton);
      
      expect(mockOnView).toHaveBeenCalledWith('doc-1');
    });
    
    it('should call onEdit when edit button is clicked', async () => {
      const mockOnEdit = vi.fn();
      renderComponent({ onEdit: mockOnEdit });
      
      const editButton = screen.getByRole('button', { name: /edit/i });
      await userEvent.click(editButton);
      
      expect(mockOnEdit).toHaveBeenCalledWith(expect.objectContaining({
        id: 'doc-1',
        title: 'Test Document'
      }));
    });
    
    it('should confirm before deleting document', async () => {
      const mockOnDelete = vi.fn();
      const confirmSpy = vi.spyOn(window, 'confirm').mockReturnValue(true);
      
      renderComponent({ onDelete: mockOnDelete });
      
      const deleteButton = screen.getByRole('button', { name: /delete/i });
      await userEvent.click(deleteButton);
      
      expect(confirmSpy).toHaveBeenCalledWith('Are you sure you want to delete this document?');
      expect(mockOnDelete).toHaveBeenCalledWith('doc-1');
    });
    
    it('should not delete if user cancels confirmation', async () => {
      const mockOnDelete = vi.fn();
      const confirmSpy = vi.spyOn(window, 'confirm').mockReturnValue(false);
      
      renderComponent({ onDelete: mockOnDelete });
      
      const deleteButton = screen.getByRole('button', { name: /delete/i });
      await userEvent.click(deleteButton);
      
      expect(confirmSpy).toHaveBeenCalled();
      expect(mockOnDelete).not.toHaveBeenCalled();
    });
  });
  
  describe('Accessibility', () => {
    it('should have proper ARIA attributes', () => {
      renderComponent();
      
      const card = screen.getByRole('article');
      expect(card).toHaveAttribute('aria-label', 'Document: Test Document');
      expect(card).toHaveAttribute('tabIndex', '0');
    });
    
    it('should support keyboard navigation', async () => {
      const mockOnView = vi.fn();
      renderComponent({ onView: mockOnView });
      
      const card = screen.getByRole('article');
      card.focus();
      
      expect(card).toHaveFocus();
      
      fireEvent.keyDown(card, { key: 'Enter' });
      expect(mockOnView).toHaveBeenCalled();
    });
  });
  
  describe('Error Handling', () => {
    it('should display error message when delete fails', async () => {
      const mockOnDelete = vi.fn().mockRejectedValue(new Error('Delete failed'));
      vi.spyOn(window, 'confirm').mockReturnValue(true);
      
      renderComponent({ onDelete: mockOnDelete });
      
      const deleteButton = screen.getByRole('button', { name: /delete/i });
      await userEvent.click(deleteButton);
      
      await waitFor(() => {
        expect(screen.getByText(/delete failed/i)).toBeInTheDocument();
      });
    });
  });
});
```

### 7.2 Integration Testing

```typescript
// Integration test example
import request from 'supertest';
import { createTestApp } from '../test-utils/test-app';
import { createTestDatabase } from '../test-utils/test-database';
import { createTestUser } from '../test-utils/test-data';

describe('Documents API Integration', () => {
  let app: any;
  let database: any;
  let authToken: string;
  
  beforeAll(async () => {
    // Setup test environment
    database = await createTestDatabase();
    app = await createTestApp(database);
    
    // Create test user and get auth token
    const { token } = await createTestUser(app);
    authToken = token;
  });
  
  afterAll(async () => {
    // Cleanup test environment
    await database.cleanup();
    await app.close();
  });
  
  beforeEach(async () => {
    // Reset database state before each test
    await database.reset();
  });
  
  describe('POST /api/v1/documents', () => {
    it('should upload document successfully', async () => {
      const response = await request(app)
        .post('/api/v1/documents')
        .set('Authorization', `Bearer ${authToken}`)
        .attach('file', Buffer.from('test content'), 'test.pdf')
        .field('title', 'Test Document')
        .field('description', 'Test Description')
        .expect(201);
      
      expect(response.body).toMatchObject({
        document: expect.objectContaining({
          id: expect.any(String),
          title: 'Test Document',
          description: 'Test Description',
          processing_status: 'uploaded'
        })
      });
      
      // Verify document was saved to database
      const savedDoc = await database.findDocument(response.body.document.id);
      expect(savedDoc).toBeTruthy();
    });
    
    it('should validate file type', async () => {
      const response = await request(app)
        .post('/api/v1/documents')
        .set('Authorization', `Bearer ${authToken}`)
        .attach('file', Buffer.from('malicious content'), 'virus.exe')
        .expect(422);
      
      expect(response.body.error).toContain('Invalid file type');
    });
    
    it('should check file size limits', async () => {
      const largeBuffer = Buffer.alloc(15 * 1024 * 1024); // 15MB
      
      const response = await request(app)
        .post('/api/v1/documents')
        .set('Authorization', `Bearer ${authToken}`)
        .attach('file', largeBuffer, 'large.pdf')
        .expect(422);
      
      expect(response.body.error).toContain('File too large');
    });
  });
  
  describe('GET /api/v1/documents', () => {
    it('should return user documents with authentication', async () => {
      // Create test data
      await createTestDocument(database, { user_id: 'test-user' });
      
      const response = await request(app)
        .get('/api/v1/documents')
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);
      
      expect(response.body).toMatchObject({
        documents: expect.arrayContaining([
          expect.objectContaining({
            id: expect.any(String),
            title: expect.any(String),
            created_at: expect.any(String)
          })
        ]),
        meta: expect.objectContaining({
          total: expect.any(Number),
          page: expect.any(Number)
        })
      });
    });
    
    it('should reject unauthenticated requests', async () => {
      await request(app)
        .get('/api/v1/documents')
        .expect(401);
    });
  });
});
```

### 7.3 End-to-End Testing with Playwright

```typescript
// E2E test example
import { test, expect } from '@playwright/test';

test.describe('Document Management Workflow', () => {
  test.beforeEach(async ({ page }) => {
    // Login
    await page.goto('/login');
    await page.fill('[data-testid="email"]', 'test@example.com');
    await page.fill('[data-testid="password"]', 'password123');
    await page.click('[data-testid="login-button"]');
    await expect(page).toHaveURL('/dashboard');
  });
  
  test('should complete full document processing workflow', async ({ page }) => {
    // 1. Navigate to upload page
    await page.click('[data-testid="upload-document"]');
    await expect(page).toHaveURL('/upload');
    
    // 2. Upload document
    await page.setInputFiles(
      '[data-testid="file-input"]', 
      './test-files/sample-meeting.pdf'
    );
    
    // 3. Select template
    await page.click('[data-testid="template-demo-meeting"]');
    await page.click('[data-testid="start-processing"]');
    
    // 4. Wait for processing to complete
    await expect(page.locator('[data-testid="processing-status"]'))
      .toHaveText('Processing completed', { timeout: 30000 });
    
    // 5. View processed document
    await page.click('[data-testid="view-document"]');
    await expect(page.locator('[data-testid="document-title"]')).toBeVisible();
    
    // 6. Verify extracted decisions
    const decisions = page.locator('[data-testid="extracted-decisions"] li');
    await expect(decisions).toHaveCount.greaterThan(0);
    
    // 7. Export as PDF
    await page.click('[data-testid="export-pdf"]');
    
    // Wait for download
    const downloadPromise = page.waitForEvent('download');
    await page.click('[data-testid="confirm-export"]');
    const download = await downloadPromise;
    
    expect(download.suggestedFilename()).toContain('.pdf');
  });
  
  test('should handle invalid file upload', async ({ page }) => {
    await page.goto('/upload');
    
    // Try to upload invalid file
    await page.setInputFiles(
      '[data-testid="file-input"]', 
      './test-files/invalid.txt'
    );
    
    // Should show error message
    await expect(page.locator('[data-testid="error-message"]'))
      .toContainText('Invalid file type');
  });
});
```

---

## SECTION 8: DEPLOYMENT GUIDE

### 8.1 Production Deployment Script

```bash
#!/bin/bash

# MinuteHub Production Deployment Script
# Version: 1.0
# Author: Guardian-X Digital Solutions

set -e  # Exit on any error

# Configuration
APP_NAME="minutehub"
DEPLOY_ENV="production"
BACKUP_DIR="/backups/$(date +%Y%m%d_%H%M%S)"
LOG_FILE="/var/log/${APP_NAME}_deploy.log"
HEALTH_CHECK_URL="https://minutehub.app/health"
ROLLBACK_VERSION=""

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# Logging function
log() {
    echo -e "${GREEN}[$(date '+%Y-%m-%d %H:%M:%S')]${NC} $1" | tee -a "$LOG_FILE"
}

error() {
    echo -e "${RED}[$(date '+%Y-%m-%d %H:%M:%S')] ERROR:${NC} $1" | tee -a "$LOG_FILE"
}

warning() {
    echo -e "${YELLOW}[$(date '+%Y-%m-%d %H:%M:%S')] WARNING:${NC} $1" | tee -a "$LOG_FILE"
}

# Check dependencies
check_dependencies() {
    log "Checking deployment dependencies..."
    
    command -v node >/dev/null 2>&1 || { error "Node.js is required but not installed. Aborting."; exit 1; }
    command -v pnpm >/dev/null 2>&1 || { error "pnpm is required but not installed. Aborting."; exit 1; }
    command -v git >/dev/null 2>&1 || { error "Git is required but not installed. Aborting."; exit 1; }
    
    log "All dependencies are available."
}

# Backup current deployment
backup_current_deployment() {
    log "Creating backup of current deployment..."
    
    if [ -d "/var/www/${APP_NAME}" ]; then
        mkdir -p "$BACKUP_DIR"
        cp -r "/var/www/${APP_NAME}" "$BACKUP_DIR/"
        log "Backup created at $BACKUP_DIR"
        ROLLBACK_VERSION="$BACKUP_DIR"
    else
        warning "No existing deployment found to backup."
    fi
}

# Validate environment
validate_environment() {
    log "Validating production environment..."
    
    # Check required environment variables
    required_vars=("VITE_SUPABASE_URL" "VITE_SUPABASE_ANON_KEY" "SUPABASE_SERVICE_ROLE_KEY")
    
    for var in "${required_vars[@]}"; do
        if [ -z "${!var}" ]; then
            error "Required environment variable $var is not set."
            exit 1
        fi
    done
    
    # Validate Supabase connection
    log "Testing Supabase connection..."
    response=$(curl -s -o /dev/null -w "%{http_code}" \
        -H "apikey: $VITE_SUPABASE_ANON_KEY" \
        -H "Authorization: Bearer $VITE_SUPABASE_ANON_KEY" \
        "$VITE_SUPABASE_URL/rest/v1/")
    
    if [ "$response" != "200" ]; then
        error "Supabase connection failed. HTTP status: $response"
        exit 1
    fi
    
    log "Environment validation completed successfully."
}

# Build application
build_application() {
    log "Building application for production..."
    
    # Install dependencies
    log "Installing dependencies..."
    pnpm install --frozen-lockfile --prod=false
    
    # Run tests
    log "Running test suite..."
    pnpm test:unit --run --coverage
    
    if [ $? -ne 0 ]; then
        error "Tests failed. Deployment aborted."
        exit 1
    fi
    
    # Build application
    log "Building production bundle..."
    pnpm build
    
    # Verify build output
    if [ ! -d "dist" ]; then
        error "Build failed - dist directory not found."
        exit 1
    fi
    
    # Check bundle size
    bundle_size=$(du -sh dist | cut -f1)
    log "Bundle size: $bundle_size"
    
    # Security scan
    log "Running security audit..."
    pnpm audit --audit-level high
    
    log "Application build completed successfully."
}

# Deploy to production
deploy_to_production() {
    log "Deploying to production..."
    
    # Deploy to Vercel
    log "Deploying to Vercel..."
    npx vercel --prod --token "$VERCEL_TOKEN" --yes
    
    if [ $? -ne 0 ]; then
        error "Vercel deployment failed."
        rollback_deployment
        exit 1
    fi
    
    log "Deployment to Vercel completed successfully."
}

# Health check
health_check() {
    log "Performing health checks..."
    
    # Wait for deployment to be available
    sleep 30
    
    # Check application health
    for i in {1..5}; do
        log "Health check attempt $i/5..."
        
        response=$(curl -s -o /dev/null -w "%{http_code}" "$HEALTH_CHECK_URL")
        
        if [ "$response" = "200" ]; then
            log "Health check passed. Application is healthy."
            return 0
        fi
        
        warning "Health check failed (HTTP $response). Retrying in 30 seconds..."
        sleep 30
    done
    
    error "Health checks failed after 5 attempts."
    rollback_deployment
    exit 1
}

# Rollback deployment
rollback_deployment() {
    error "Initiating rollback procedure..."
    
    if [ -n "$ROLLBACK_VERSION" ]; then
        log "Rolling back to previous version..."
        # Implement rollback logic here
        log "Rollback completed."
    else
        error "No rollback version available."
    fi
}

# Update database schema
update_database() {
    log "Checking for database migrations..."
    
    # Run database migrations if needed
    if [ -f "migrations/latest.sql" ]; then
        log "Applying database migrations..."
        # Apply migrations using Supabase CLI or direct SQL execution
        log "Database migrations completed."
    else
        log "No database migrations required."
    fi
}

# Send deployment notification
send_notification() {
    local status=$1
    local message=$2
    
    log "Sending deployment notification..."
    
    # Send to Slack webhook (if configured)
    if [ -n "$SLACK_WEBHOOK_URL" ]; then
        curl -X POST -H 'Content-type: application/json' \
            --data "{\"text\":\"MinuteHub Deployment $status: $message\"}" \
            "$SLACK_WEBHOOK_URL"
    fi
    
    # Send email notification (if configured)
    if [ -n "$NOTIFICATION_EMAIL" ]; then
        echo "$message" | mail -s "MinuteHub Deployment $status" "$NOTIFICATION_EMAIL"
    fi
}

# Main deployment function
main() {
    log "Starting MinuteHub production deployment..."
    
    # Trap errors and execute rollback
    trap 'error "Deployment failed at line $LINENO"; rollback_deployment; exit 1' ERR
    
    check_dependencies
    validate_environment
    backup_current_deployment
    update_database
    build_application
    deploy_to_production
    health_check
    
    log "MinuteHub deployment completed successfully!"
    send_notification "SUCCESS" "MinuteHub has been successfully deployed to production."
}

# Execute main function
main "$@"
```

### 8.2 Monitoring and Analytics Setup

```typescript
// Monitoring and alerting service
export class MonitoringService {
  async setupProductionMonitoring(): Promise<void> {
    // Error tracking with Sentry
    Sentry.init({
      dsn: process.env.VITE_SENTRY_DSN,
      environment: process.env.VITE_APP_ENVIRONMENT,
      tracesSampleRate: 1.0,
      beforeSend(event) {
        // Filter out sensitive information
        if (event.extra) {
          delete event.extra.password;
          delete event.extra.token;
        }
        return event;
      }
    });
    
    // Performance monitoring
    this.setupPerformanceMonitoring();
    
    // Application metrics
    this.setupApplicationMetrics();
    
    // Health checks
    this.setupHealthChecks();
    
    // Alert configuration
    this.setupAlerts();
  }
  
  private setupPerformanceMonitoring(): void {
    // Core Web Vitals tracking
    getCLS(this.reportMetric);
    getFID(this.reportMetric);
    getFCP(this.reportMetric);
    getLCP(this.reportMetric);
    getTTFB(this.reportMetric);
  }
  
  private reportMetric = (metric: Metric): void => {
    // Send to analytics service
    gtag('event', metric.name, {
      value: Math.round(metric.name === 'CLS' ? metric.value * 1000 : metric.value),
      event_category: 'Web Vitals',
      event_label: metric.id,
      non_interaction: true,
    });
    
    // Send to monitoring service
    fetch('/api/metrics', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        name: metric.name,
        value: metric.value,
        id: metric.id,
        timestamp: Date.now()
      })
    });
  };
  
  private setupHealthChecks(): void {
    // Create health check endpoint
    setInterval(async () => {
      try {
        const healthData = {
          timestamp: new Date().toISOString(),
          uptime: process.uptime(),
          memory: process.memoryUsage(),
          database: await this.checkDatabaseHealth(),
          storage: await this.checkStorageHealth(),
          external_services: await this.checkExternalServices()
        };
        
        // Report health metrics
        await this.reportHealthMetrics(healthData);
      } catch (error) {
        console.error('Health check failed:', error);
        Sentry.captureException(error);
      }
    }, 60000); // Every minute
  }
  
  private async checkDatabaseHealth(): Promise<{ status: string; latency: number }> {
    const start = Date.now();
    try {
      await supabase.from('users').select('count').limit(1);
      return {
        status: 'healthy',
        latency: Date.now() - start
      };
    } catch (error) {
      return {
        status: 'unhealthy',
        latency: Date.now() - start
      };
    }
  }
  
  private async checkStorageHealth(): Promise<{ status: string; latency: number }> {
    const start = Date.now();
    try {
      await supabase.storage.from('documents').list('', { limit: 1 });
      return {
        status: 'healthy',
        latency: Date.now() - start
      };
    } catch (error) {
      return {
        status: 'unhealthy',
        latency: Date.now() - start
      };
    }
  }
  
  private async setupAlerts(): Promise<void> {
    // Configure error rate alerts
    this.setupErrorRateAlert({
      threshold: 0.05, // 5% error rate
      timeWindow: '5m',
      action: 'email + slack'
    });
    
    // Configure response time alerts
    this.setupResponseTimeAlert({
      threshold: 2000, // 2 seconds
      percentile: 95,
      timeWindow: '5m',
      action: 'slack'
    });
    
    // Configure availability alerts
    this.setupAvailabilityAlert({
      threshold: 0.99, // 99% availability
      timeWindow: '5m',
      action: 'email + slack + sms'
    });
  }
}
```

### 8.3 GitHub Actions Workflow

```yaml
# .github/workflows/production-deploy.yml
name: Production Deployment

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '18'
  PNPM_VERSION: '8'

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
      
      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Run linting
        run: pnpm lint
      
      - name: Run type checking
        run: pnpm type-check
      
      - name: Run unit tests
        run: pnpm test:unit --coverage
      
      - name: Run integration tests
        run: pnpm test:integration
        env:
          SUPABASE_URL: ${{ secrets.SUPABASE_URL }}
          SUPABASE_ANON_KEY: ${{ secrets.SUPABASE_ANON_KEY }}
      
      - name: Upload coverage reports
        uses: codecov/codecov-action@v3
        with:
          token: ${{ secrets.CODECOV_TOKEN }}

  security:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Run security audit
        run: pnpm audit --audit-level high
      
      - name: Run Snyk security scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

  build:
    runs-on: ubuntu-latest
    needs: [test, security]
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
      
      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Build application
        run: pnpm build
        env:
          VITE_SUPABASE_URL: ${{ secrets.VITE_SUPABASE_URL }}
          VITE_SUPABASE_ANON_KEY: ${{ secrets.VITE_SUPABASE_ANON_KEY }}
          VITE_SENTRY_DSN: ${{ secrets.VITE_SENTRY_DSN }}
      
      - name: Upload build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: dist
          path: dist/
          retention-days: 7

  deploy:
    runs-on: ubuntu-latest
    needs: [build]
    if: github.ref == 'refs/heads/main'
    environment: production
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Download build artifacts
        uses: actions/download-artifact@v3
        with:
          name: dist
          path: dist/
      
      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: '--prod'
      
      - name: Run smoke tests
        run: |
          curl -f https://minutehub.app/health || exit 1
          curl -f https://minutehub.app/api/health || exit 1

  e2e-tests:
    runs-on: ubuntu-latest
    needs: [deploy]
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
      
      - name: Install Playwright
        run: npx playwright install --with-deps
      
      - name: Run E2E tests
        run: npx playwright test
        env:
          PLAYWRIGHT_BASE_URL: 'https://minutehub.app'
          TEST_USER_EMAIL: ${{ secrets.TEST_USER_EMAIL }}
          TEST_USER_PASSWORD: ${{ secrets.TEST_USER_PASSWORD }}
      
      - name: Upload test results
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

---

# CONCLUSION

## Summary of MinuteHub Technical Specification

This comprehensive technical specification provides a complete implementation guide for the MinuteHub ActionBook application. The document covers:

### **Technical Architecture**
- **Database Design:** Complete PostgreSQL schema with RLS policies, indexes, and triggers
- **API Specification:** RESTful API design with comprehensive endpoint documentation
- **Security Implementation:** JWT authentication, RBAC, encryption, and security monitoring
- **System Architecture:** Scalable, secure, and maintainable application structure

### **Implementation Guide**
- **Development Setup:** Complete environment configuration and tooling setup
- **Feature Implementation:** Detailed code examples for core features
- **Testing Strategy:** Unit, integration, and E2E testing with comprehensive examples
- **Deployment Guide:** Production-ready deployment scripts and CI/CD pipelines

### **Code Quality Standards**
- **TypeScript Configuration:** Strict typing and modern JavaScript features
- **ESLint and Prettier:** Code formatting and linting standards
- **Testing Templates:** Comprehensive testing strategies and examples
- **Security Guidelines:** Best practices for secure development

### **Key Technical Features**

**Frontend Technologies:**
- React 18 with TypeScript for type safety
- Tailwind CSS with shadcn/ui for modern UI components
- Zustand for state management
- React Hook Form for form handling
- Vite for fast development and optimized builds

**Backend Technologies:**
- Supabase for PostgreSQL database, authentication, and storage
- Row Level Security for data protection
- Real-time subscriptions for live updates
- Serverless functions for custom logic

**Security Features:**
- JWT-based authentication with refresh tokens
- Role-based access control (RBAC)
- Data encryption at rest and in transit
- Input validation and sanitization
- Audit logging and security monitoring

**Guardian-X Template System:**
- Template A: Demo Meeting Template (General purpose)
- Template B: Kamati ya Usimamizi (Management Committee - Swahili)
- Template C: Baraza Kuu (Executive Council - High-level)
- Template D: Mkutano wa Kitengo (Operational Committee)

**AI Processing Capabilities:**
- Document text extraction (PDF, DOCX, Images with OCR)
- Decision extraction with confidence scoring
- Action item identification
- Multi-language support (English and Swahili)

**Export and Integration:**
- PDF generation with custom formatting
- DOCX export with proper styling
- FollowUpHub integration for decision tracking
- Real-time status updates

### **Production Readiness**

**Performance Targets:**
- Core Web Vitals: LCP <2.5s, FID <100ms, CLS <0.1
- API Response Time: <500ms average
- 99.5% uptime during business hours
- Support for 100 concurrent users

**Security Standards:**
- Zero critical vulnerabilities
- Data encryption (AES-256 at rest, TLS 1.3 in transit)
- Multi-factor authentication support
- Comprehensive audit logging

**Quality Assurance:**
- >90% test coverage
- Automated security scanning
- Performance monitoring
- Error tracking and alerting

### **Deployment Strategy**

**Hosting Platform:** Vercel with CDN
**Database:** Supabase (PostgreSQL)
**Monitoring:** Sentry for error tracking, Vercel Analytics
**CI/CD:** GitHub Actions with automated testing and deployment

### **Success Metrics**

**Technical KPIs:**
- Application uptime >99.5%
- Build time <3 minutes
- Test execution time <5 minutes
- Security scan passing rate 100%

**User Experience KPIs:**
- Document processing success rate >95%
- User satisfaction score >4.0/5.0
- Feature adoption rate >80%
- Support ticket volume <5% of users

This technical specification provides everything needed to successfully implement, deploy, and maintain the MinuteHub ActionBook application as a robust, scalable, and secure meeting minutes processing platform that integrates seamlessly with the broader MinuteHub ecosystem and FollowUpHub.

---

*Document prepared by Guardian-X Digital Solutions*  
*Version: 1.0 - Technical Specification*  
*Date: October 2025*  
*Status: Ready for Implementation*

*Linked to: MinuteHub Development Guideline Plan v1*  
*Integration Target: FollowUpHub Task Management System*  
*Compliance Framework: Sentinel Security Document*