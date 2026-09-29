# Xport — AI Synthetic Data Generation Platform

![Xport Banner](https://img.shields.io/badge/Xport-Enterprise_Synthetic_Data-orange?style=for-the-badge&logo=databricks)
![Status](https://img.shields.io/badge/Status-Production_Ready-success?style=for-the-badge)
![Supabase](https://img.shields.io/badge/Supabase-PostgreSQL_%26_Storage-3ecf8e?style=for-the-badge&logo=supabase)
![Gemini AI](https://img.shields.io/badge/Google_AI_Studio-Gemini_Flash_API-blue?style=for-the-badge&logo=google)

> **Export your data into any format — on demand, privacy-safe, AI-synthesized.**

---

## 📋 Table of Contents
1. [Executive Summary & Value Proposition](#1-executive-summary--value-proposition)
2. [System Architecture & Tech Stack](#2-system-architecture--tech-stack)
3. [Core Features & Modules](#3-core-features--modules)
4. [Step-by-Step Local Installation Guide](#4-step-by-step-local-installation-guide)
5. [Supabase Database & Storage Setup (Complete SQL)](#5-supabase-database--storage-setup-complete-sql)
6. [TSTR Evaluation Methodology](#6-tstr-evaluation-methodology)

---

## 1. Executive Summary & Value Proposition

Modern software engineering and machine learning teams routinely face severe production data bottlenecks due to stringent global privacy compliance laws (**GDPR, HIPAA, CCPA, PCI-DSS**) and slow security procurement cycles. 

Conventional mock data solutions present critical flaws:
* **Flaky Test Suites:** Uncontrolled randomness causing non-reproducible test failures.
* **The "Fake Data Oasis":** Failure to simulate complex edge cases and real-world distributions.
* **Relational Disconnection:** Broken foreign-key dependencies across multi-table databases.
* **ML Invalidation:** Inability to preserve statistical correlations, covariance, and label variance.

**Xport** eliminates procurement delays by ingesting unstructured or structured metadata and leveraging **Google AI Studio (Gemini 1.5/3.8 Flash)** with active web grounding to synthesize privacy-compliant, statistically faithful datasets across Tabular, Relational, Document, and Machine Learning formats.

---

## 2. System Architecture & Tech Stack

* **Frontend & Web Hosting:** React, Vite, Tailwind CSS (Soft Extruded Neumorphic Design System fused with Solar Accent Illumination, Poppins & Outfit typography).
* **Database & Persistence:** Supabase (PostgreSQL DB + Storage Buckets for user uploads and generated exports).
* **Intelligence Layer:** Google AI Studio API (`gemini-3.8-flash` with structured outputs and Search Grounding).
* **Deployment Pipeline:** Vercel continuous production hosting connected via GitHub.

---

## 3. Core Features & Modules

1. **Tabular Data Module (.csv)**:
   - Generates flat row-column structures optimized for Excel, Pandas, and BigQuery.
   - Enforces strict seed determinism so identical random seed values produce identical CSV outputs.
   - Integrated **TSTR (Train Synthetic, Test Real)** quality scoring.

2. **Relational Database Module (.sql)**:
   - Generates complete, executable SQL dumps with valid `CREATE TABLE` DDL statements (`customers`, `orders`, `order_items`).
   - Enforces strict foreign key referential integrity with zero orphan records.

3. **Document Generator Engine**:
   - Supports sub-format generation for **PDF (.pdf)**, **Word (.docx)**, **PowerPoint (.pptx)**, **Plain Text (.txt)**, **Markdown (.md)**, **Rich Text (.rtf)**, and **JSON (.json)**.

4. **Machine Learning Training Exporter**:
   - Delivers clean numeric features stripped of currency symbols and commas (e.g., `annual_revenue: 791746`).
   - Separates continuous probabilities from class labels (`churn_risk_score: 0.51`, `churn_risk_label: "Medium"`).
   - Provides pre-split `.jsonl` options (`train.jsonl` / `test.jsonl` with an 80/20 split toggle).

---

## 4. Step-by-Step Local Installation Guide

### Prerequisites
* Node.js v18 or higher installed on your system.
* A Supabase project (for PostgreSQL database and storage buckets).
* A Google AI Studio API key (`GEMINI_XPORT_API_KEY`).

### 1. Clone Repository & Install Dependencies
\`\`\`bash
git clone https://github.com/your-username/xport-platform.git
cd xport-platform
npm install
\`\`\`

### 2. Environment Variable Setup
Create a `.env` or `.env.local` file in the root directory:
\`\`\`env
GEMINI_XPORT_API_KEY="your_google_ai_studio_api_key_here"
SUPABASE_URL="https://your-supabase-project.supabase.co"
SUPABASE_ANON_KEY="your_supabase_anon_key_here"
APP_URL="http://localhost:3000"
\`\`\`

### 3. Run Local Development Server
\`\`\`bash
npm run dev
\`\`\`
Open your browser and navigate to `http://localhost:3000`.

---

## 5. Supabase Database Setup (Complete SQL)

Run the following complete SQL script in your **Supabase SQL Editor** to set up all required database tables, audit columns, indexes, and storage bucket policies.

\`\`\`sql
-- ========================================================
-- XPORT ENTERPRISE PLATFORM - SUPABASE DDL SETUP SCRIPT
-- ========================================================

-- Enable UUID extension if not already enabled
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- 1. USER UPLOADS TABLE
-- Stores metadata for files uploaded during Ingestion Stage 2
CREATE TABLE IF NOT EXISTS public.user_uploads (
    id VARCHAR(64) PRIMARY KEY,
    file_name VARCHAR(255) NOT NULL,
    file_path TEXT NOT NULL,
    file_type VARCHAR(32) NOT NULL,
    raw_metadata JSONB DEFAULT '{}'::jsonb,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- 2. GENERATED EXPORTS TABLE
-- Stores audit records for generated synthetic datasets and exports
CREATE TABLE IF NOT EXISTS public.generated_exports (
    id VARCHAR(64) PRIMARY KEY,
    upload_id VARCHAR(64) REFERENCES public.user_uploads(id) ON DELETE SET NULL,
    format_type VARCHAR(32) NOT NULL,
    sub_format VARCHAR(32) DEFAULT 'csv',
    row_count INTEGER NOT NULL DEFAULT 500,
    random_seed INTEGER NOT NULL DEFAULT 42,
    tstr_score NUMERIC(3, 2) NOT NULL DEFAULT 4.80,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- 3. EXPORT CHECKPOINTS TABLE
-- Stores state checkpoints for reproducibility and time-travel restore
CREATE TABLE IF NOT EXISTS public.export_checkpoints (
    checkpoint_id VARCHAR(64) PRIMARY KEY,
    upload_id VARCHAR(64) REFERENCES public.user_uploads(id) ON DELETE CASCADE,
    format VARCHAR(32) NOT NULL,
    sub_format VARCHAR(32),
    transformation_rules JSONB DEFAULT '{}'::jsonb,
    tstr_score NUMERIC(3, 2),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Indexes for performance optimization
CREATE INDEX IF NOT EXISTS idx_user_uploads_created ON public.user_uploads(created_at DESC);
CREATE INDEX IF NOT EXISTS idx_generated_exports_upload ON public.generated_exports(upload_id);
CREATE INDEX IF NOT EXISTS idx_export_checkpoints_chk ON public.export_checkpoints(checkpoint_id);

-- ========================================================
-- STORAGE BUCKETS SETUP
-- ========================================================

-- Insert public/private storage buckets into storage.buckets
INSERT INTO storage.buckets (id, name, public, file_size_limit, allowed_mime_types)
VALUES 
    ('user-uploads', 'user-uploads', true, 524288000, ARRAY['text/csv', 'application/json', 'application/pdf', 'application/vnd.openxmlformats-officedocument.wordprocessingml.document']),
    ('generated-exports', 'generated-exports', true, 524288000, ARRAY['text/csv', 'application/sql', 'application/x-jsonlines', 'application/json', 'text/plain'])
ON CONFLICT (id) DO UPDATE SET public = EXCLUDED.public;

-- Row Level Security (RLS) Policies for Storage
CREATE POLICY "Public Access User Uploads" ON storage.objects 
    FOR ALL USING (bucket_id = 'user-uploads') WITH CHECK (bucket_id = 'user-uploads');

CREATE POLICY "Public Access Generated Exports" ON storage.objects 
    FOR ALL USING (bucket_id = 'generated-exports') WITH CHECK (bucket_id = 'generated-exports');

-- Enable RLS on tables (optional permissive development policies)
ALTER TABLE public.user_uploads ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.generated_exports ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.export_checkpoints ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Allow all operations on user_uploads" ON public.user_uploads FOR ALL USING (true) WITH CHECK (true);
CREATE POLICY "Allow all operations on generated_exports" ON public.generated_exports FOR ALL USING (true) WITH CHECK (true);
CREATE POLICY "Allow all operations on export_checkpoints" ON public.export_checkpoints FOR ALL USING (true) WITH CHECK (true);
\`\`G`

---

## 6. TSTR Evaluation Methodology

To eliminate data scientist skepticism regarding synthetic data utility, Xport incorporates an automated **TSTR (Train Synthetic, Test Real)** evaluation module:

1. **Synthetic Utility Index Formula:**
   $$\text{Utility Index} = 1 + 4 \times (0.4 \times \text{JS\_Fidelity} + 0.3 \times \text{KS\_Fidelity} + 0.3 \times \text{Correlation\_Fidelity})$$
2. **Jensen-Shannon Divergence ($JSD$):** Evaluates categorical distribution overlap ($JSD \le 0.02$).
3. **Kolmogorov-Smirnov Test ($KS$):** Measures continuous feature distribution alignment.
4. **Target Threshold:** Requires $\ge 85\%$ baseline utility retention (normalized to a 5.0 score scale) to guarantee production-grade ML readiness.
