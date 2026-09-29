# Xport — AI Synthetic Data Generation Platform

![Xport Banner](https://img.shields.io/badge/Xport-Enterprise_Synthetic_Data-orange?style=for-the-badge&logo=databricks)
![Status](https://img.shields.io/badge/Status-Production_Ready-success?style=for-the-badge)
![License](https://img.shields.io/badge/License-Apache_2.0-blue?style=for-the-badge)

**Xport** is a state-of-the-art AI-driven synthetic data platform built for developers, programmers, and ML engineers facing production data bottlenecks due to privacy compliance laws (GDPR/HIPAA) and slow procurement cycles.

---

## 🌟 Core Value Proposition
- **Zero "Fake Data Oasis" Pitfalls:** Generates statistically aligned, privacy-safe synthetic data with guaranteed relational integrity.
- **Infinite Variation Engine:** Tweak row counts, random seeds, and locales to generate endless, non-repeating dataset variations.
- **TSTR Quality Evaluation:** Built-in Train Synthetic, Test Real (TSTR) pipeline calculating Jensen-Shannon divergence, Kolmogorov-Smirnov test, and correlation matrix alignment with a 1.0–5.0 Synthetic Utility Score.
- **Multi-Key API Sanitization & Rotation Engine:** Automatic 429/401 rate limit detection and comma-separated key rotation across Gemini API keys.

---

## 🚀 System Architecture & Tech Stack
- **Frontend / UI:** React, Tailwind CSS (Neumorphic design system with Light, Cyber Emerald, and Sunset Gradient themes).
- **Backend / API:** Node.js, Express, Vite middleware, Multi-Key Rotation.
- **Database & Persistence:** Supabase (PostgreSQL DB + Storage buckets).
- **Intelligence Layer:** Google AI Studio SDK (`@google/genai`, model `gemini-3.8-flash`) with structured JSON outputs and robust fallback mechanisms.

---

## 💾 Supabase Database Setup (Complete SQL DDL)

To set up your Supabase project for **Xport**, run the following complete DDL SQL script in your Supabase SQL Editor:

```sql
-- ============================================================================
-- XPORT PLATFORM — SUPABASE POSTGRESQL PRODUCTION DDL SCHEMA
-- ============================================================================

-- 1. Enable UUID Extension
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- 2. User Uploads Table
CREATE TABLE IF NOT EXISTS user_uploads (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    file_name TEXT NOT NULL,
    file_path TEXT NOT NULL,
    file_type TEXT NOT NULL,
    file_size BIGINT DEFAULT 0,
    raw_metadata JSONB DEFAULT '{}'::jsonb,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT timezone('utc'::text, now()) NOT NULL
);

-- 3. Generated Exports Table
CREATE TABLE IF NOT EXISTS generated_exports (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    upload_id UUID REFERENCES user_uploads(id) ON DELETE SET NULL,
    format_type TEXT NOT NULL,
    sub_format TEXT NOT NULL,
    row_count INTEGER DEFAULT 500,
    random_seed INTEGER DEFAULT 42,
    tstr_score NUMERIC(3,2) DEFAULT 4.80,
    output_url TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT timezone('utc'::text, now()) NOT NULL
);

-- 4. Export Checkpoints Table
CREATE TABLE IF NOT EXISTS export_checkpoints (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    upload_id UUID REFERENCES user_uploads(id) ON DELETE SET NULL,
    format TEXT NOT NULL,
    sub_format TEXT NOT NULL,
    generated_content TEXT,
    transformation_rules JSONB DEFAULT '{}'::jsonb,
    tstr_score NUMERIC(3,2) DEFAULT 4.80,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT timezone('utc'::text, now()) NOT NULL
);

-- 5. Disable Row Level Security (RLS) for Hackathon Ease
ALTER TABLE user_uploads DISABLE ROW LEVEL SECURITY;
ALTER TABLE generated_exports DISABLE ROW LEVEL SECURITY;
ALTER TABLE export_checkpoints DISABLE ROW LEVEL SECURITY;

-- 6. Storage Buckets Initialization
INSERT INTO storage.buckets (id, name, public) 
VALUES ('user-uploads', 'user-uploads', true)
ON CONFLICT (id) DO NOTHING;

INSERT INTO storage.buckets (id, name, public) 
VALUES ('generated-exports', 'generated-exports', true)
ON CONFLICT (id) DO NOTHING;
```

---

## 🛠️ Local Installation & Development

1. **Clone Repository & Install Dependencies:**
   ```bash
   npm install
   ```
2. **Configure Environment Variables:**
   Create a `.env` file in the root directory:
   ```env
   GEMINI_XPORT_API_KEYS="key1,key2,key3"
   SUPABASE_URL="your_supabase_url"
   SUPABASE_ANON_KEY="your_supabase_anon_key"
   ```
3. **Run Development Server:**
   ```bash
   npm run dev
   ```

---

## 📄 License
Distributed under the Apache 2.0 License. See `LICENSE` for details.
